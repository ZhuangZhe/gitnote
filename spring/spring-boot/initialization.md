# 启动过程

SpringBoot的启动很简单，代码如下：

```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

从代码上可以看出，调用了`SpringApplication`的静态方法`run`。这个`run`方法会构造一个`SpringApplication`的实例，然后再调用这里实例的`run`方法就表示启动SpringBoot。

因此，想要分析SpringBoot的启动过程，我们需要熟悉`SpringApplication`的构造过程以及`SpringApplication`的`run`方法执行过程即可。

## SpringApplication的构造过程

`SpringApplication`构造的时候内部会调用一个private方法`initialize`：

```java
public SpringApplication(Object... sources) {
    initialize(sources); // sources目前是一个MyApplication的class对象
}

private void initialize(Object[] sources) {
    if (sources != null && sources.length > 0) {
        // 把sources设置到SpringApplication的sources属性中，目前只是一个MyApplication类对象。
        this.sources.addAll(Arrays.asList(sources)); 
    }
    
    // 判断是否是web程序(javax.servlet.Servlet和org.springframework.web.context.ConfigurableWebApplicationContext都必须在类加载器中存在)，并设置到webEnvironment属性中。
    this.webEnvironment = deduceWebEnvironment(); 
    
    // 从spring.factories文件中找出key为ApplicationContextInitializer的类并实例化后设置到SpringApplication的initializers属性中。
    // 这个过程也就是找出所有的应用程序初始化器。
    setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
    
    // 从spring.factories文件中找出key为ApplicationListener的类并实例化后设置到SpringApplication的listeners属性中。这个过程就是找出所有的应用程序事件监听器。
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    
    // 找出main类，这里是MyApplication类
    this.mainApplicationClass = deduceMainApplicationClass();
}
```

### **`ApplicationContextInitializer`**

**应用程序初始化器，做一些初始化的工作：**

```java
public interface ApplicationContextInitializer<C extends ConfigurableApplicationContext> {
    
    void initialize(C applicationContext);
    
}
```

默认情况下，`initialize`方法从`spring.factories`文件中找出的key为`ApplicationContextInitializer`的类有：

| 初始化器 |
| :--- |
| `org.springframework.boot.context.config.DelegatingApplicationContextInitializer` |
| `org.springframework.boot.context.ContextIdApplicationContextInitializer` |
| `org.springframework.boot.context.ConfigurationWarningsApplicationContextInitializer` |
| `org.springframework.boot.context.web.ServerPortInfoApplicationContextInitializer` |
| `org.springframework.boot.autoconfigure.logging.AutoConfigurationReportLoggingInitializer` |
| `org.springframework.boot.autoconfigure.BackgroundPreinitializer` |

### **`ApplicationListener`**

**应用程序事件\(`ApplicationEvent`\)监听器：**

```java
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
    
    void onApplicationEvent(E event);

}
```

这里的应用程序事件\(`ApplicationEvent`\)有应用程序启动事件\(`ApplicationStartedEvent`\)，失败事件\(`ApplicationFailedEvent`\)，准备事件\(`ApplicationPreparedEvent`\)等。

应用程序事件监听器跟监听事件是绑定的。如`ConfigServerBootstrapApplicationListener`只跟`ApplicationEnvironmentPreparedEvent`事件绑定，`LiquibaseServiceLocatorApplicationListener`只跟`ApplicationStartedEvent`事件绑定，`LoggingApplicationListener`跟所有事件绑定等。

key为`ApplicationListener`的有：

| 监听器 |
| :--- |
| `org.springframework.boot.context.config.ConfigFileApplicationListener` |
| `org.springframework.boot.context.config.AnsiOutputApplicationListener` |
| `org.springframework.boot.logging.LoggingApplicationListener` |
| `org.springframework.boot.logging.ClasspathLoggingApplicationListener` |
| `org.springframework.boot.context.config.DelegatingApplicationListener` |
| `org.springframework.boot.builder.ParentContextCloserApplicationListener` |
| `org.springframework.boot.context.FileEncodingApplicationListener` |
| `org.springframework.boot.liquibase.LiquibaseServiceLocatorApplicationListener` |

## SpringApplication的执行

分析run方法之前，先看一下`SpringApplication`中的一些事件和监听器概念。

首先是`SpringApplicationRunListeners`类和`SpringApplicationRunListener`类的介绍。

`SpringApplicationRunListeners`内部持有`SpringApplicationRunListener`集合和1个`Log`日志类。用于`SpringApplicationRunListener`监听器的批量执行。

`SpringApplicationRunListener`看名字也知道用于监听`SpringApplication`的`run`方法的执行。

### 5个步骤：

1. `started`\(`run`方法执行的时候立马执行；对应事件的类型是`ApplicationStartedEvent`\)
2. `environmentPrepared`\(`ApplicationContext`创建之前并且环境信息准备好的时候调用；对应事件的类型是`ApplicationEnvironmentPreparedEvent`\)
3. `contextPrepared`\(`ApplicationContext`创建好并且在`source`加载之前调用一次；没有具体的对应事件\)
4. `contextLoaded`\(`ApplicationContext`创建并加载之后并在`refresh`之前调用；对应事件的类型是`ApplicationPreparedEvent`\)
5. `finished`\(`run`方法结束之前调用；对应事件的类型是`ApplicationReadyEvent`或`ApplicationFailedEvent`\)

`SpringApplicationRunListener`目前只有一个实现类`EventPublishingRunListener`，它把监听的过程封装成了`SpringApplicationEvent`事件并让内部属性\(属性名为`multicaster`\)`ApplicationEventMulticaster`接口的实现类`SimpleApplicationEventMulticaster`广播出去，广播出去的事件对象会被`SpringApplication`中的`listeners`属性进行处理。

所以说`SpringApplicationRunListener`和`ApplicationListener`之间的关系是通过`ApplicationEventMulticaster`广播出去的`SpringApplicationEvent`所联系起来的。

![](../../.gitbook/assets/image%20%2861%29.png)

`SpringApplication`的`run`方法代码如下：

```java
public ConfigurableApplicationContext run(String... args) {
    StopWatch stopWatch = new StopWatch(); // 构造一个任务执行观察器
    stopWatch.start(); // 开始执行，记录开始时间
    ConfigurableApplicationContext context = null;
    configureHeadlessProperty();
    
    // 获取SpringApplicationRunListeners，内部只有一个EventPublishingRunListener
    SpringApplicationRunListeners listeners = getRunListeners(args);
    
    // 上面分析过，会封装成SpringApplicationEvent事件然后广播出去给SpringApplication中的listeners所监听
    // 这里接受ApplicationStartedEvent事件的listener会执行相应的操作
    listeners.started();
    
    try {
        // 构造一个应用程序参数持有类
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
        
        // 创建Spring容器
        context = createAndRefreshContext(listeners, applicationArguments);
        
        // 容器创建完成之后执行额外一些操作
        afterRefresh(context, applicationArguments);
        
        // 广播出ApplicationReadyEvent事件给相应的监听器执行
        listeners.finished(context, null);
        stopWatch.stop(); // 执行结束，记录执行时间
        
        if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
        }
        return context; // 返回Spring容器
    } catch (Throwable ex) {
        handleRunFailure(context, listeners, ex); // 这个过程报错的话会执行一些异常操作、然后广播出ApplicationFailedEvent事件给相应的监听器执行
        throw new IllegalStateException(ex);
    }
}
```

创建容器的方法`createAndRefreshContext`如下：

```java
private ConfigurableApplicationContext createAndRefreshContext(SpringApplicationRunListeners listeners, ApplicationArguments applicationArguments) {
    
    ConfigurableApplicationContext context; // 定义Spring容器
    
    // 创建应用程序的环境信息。如果是web程序，创建StandardServletEnvironment；否则，创建StandardEnvironment
    ConfigurableEnvironment environment = getOrCreateEnvironment();
    
    // 配置一些环境信息。比如profile，命令行参数
    configureEnvironment(environment, applicationArguments.getSourceArgs());
    
    // 广播出ApplicationEnvironmentPreparedEvent事件给相应的监听器执行
    listeners.environmentPrepared(environment);
    
    // 环境信息的校对
    if (isWebEnvironment(environment) && !this.webEnvironment) {
        environment = convertToStandardEnvironment(environment);
    }
    
    if (this.bannerMode != Banner.Mode.OFF) { // 是否在控制台上打印自定义的banner
        printBanner(environment);
    }
    
    // Create, load, refresh and run the ApplicationContext
    context = createApplicationContext(); // 创建Spring容器
    context.setEnvironment(environment); // 设置Spring容器的环境信息
    postProcessApplicationContext(context); // 回调方法，Spring容器创建之后做一些额外的事
    applyInitializers(context); // SpringApplication的的初始化器开始工作
    
    // 遍历调用SpringApplicationRunListener的contextPrepared方法。目前只是将这个事件广播器注册到Spring容器中
    listeners.contextPrepared(context);
    
    if (this.logStartupInfo) {
        logStartupInfo(context.getParent() == null);
        logStartupProfileInfo(context);
    }
    
    // 把应用程序参数持有类注册到Spring容器中，并且是一个单例
    context.getBeanFactory().registerSingleton("springApplicationArguments",
    applicationArguments);
    
    Set<Object> sources = getSources();
    Assert.notEmpty(sources, "Sources must not be empty");
    load(context, sources.toArray(new Object[sources.size()]));
    
    // 广播出ApplicationPreparedEvent事件给相应的监听器执行
    listeners.contextLoaded(context);
    
    // Spring容器的刷新
    refresh(context);
    
    if (this.registerShutdownHook) {
        try {
            context.registerShutdownHook();
        } catch (AccessControlException ex) {
            // Not allowed in some environments.
        }
    }
    return context;
    
}
```

Spring容器的创建`createApplicationContext`方法如下：

```java
protected ConfigurableApplicationContext createApplicationContext() {
    Class<?> contextClass = this.applicationContextClass;
    if (contextClass == null) {
        try {
            // 如果是web程序，那么构造org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext容器
            // 否则构造org.springframework.context.annotation.AnnotationConfigApplicationContext容器
            contextClass = Class.forName(this.webEnvironment ? DEFAULT_WEB_CONTEXT_CLASS : DEFAULT_CONTEXT_CLASS);
        } catch (ClassNotFoundException ex) {
            throw new IllegalStateException(
                "Unable create a default ApplicationContext, "
                + "please specify an ApplicationContextClass",
                ex);
        }
    }
    return (ConfigurableApplicationContext) BeanUtils.instantiate(contextClass);
}
```

Spring容器创建之后有个回调方法`postProcessApplicationContext`：

```java
protected void postProcessApplicationContext(ConfigurableApplicationContext context) {
    if (this.webEnvironment) { // 如果是web程序
        if (context instanceof ConfigurableWebApplicationContext) { // 并且也是Spring Web容器
            ConfigurableWebApplicationContext configurableContext = (ConfigurableWebApplicationContext) context;
            if (this.beanNameGenerator != null) { // 如果SpringApplication设置了是实例命名生成器，注册到Spring容器中
                configurableContext.getBeanFactory().registerSingleton(
                    AnnotationConfigUtils.CONFIGURATION_BEAN_NAME_GENERATOR,
                    this.beanNameGenerator);
            }
        }
    }
    
    if (this.resourceLoader != null) { // 如果SpringApplication设置了资源加载器，设置到Spring容器中
        if (context instanceof GenericApplicationContext) {
            ((GenericApplicationContext) context).setResourceLoader(this.resourceLoader);
        }
        if (context instanceof DefaultResourceLoader) {
            ((DefaultResourceLoader) context).setClassLoader(this.resourceLoader.getClassLoader());
        }
    }
}
```

初始化器做的工作，比如`ContextIdApplicationContextInitializer`会设置应用程序的id；`AutoConfigurationReportLoggingInitializer`会给应用程序添加一个条件注解解析器报告等：

```java
protected void applyInitializers(ConfigurableApplicationContext context) {
    // 遍历每个初始化器，对调用对应的initialize方法
    for (ApplicationContextInitializer initializer : getInitializers()) {
        Class<?> requiredType = GenericTypeResolver.resolveTypeArgument(
            initializer.getClass(), ApplicationContextInitializer.class);
        Assert.isInstanceOf(requiredType, context, "Unable to call initializer.");
        initializer.initialize(context);
    }
}
```

Spring容器的刷新`refresh`方法内部会做很多很多的事情：比如`BeanFactory`的设置，`BeanFactoryPostProcessor`接口的执行、`BeanPostProcessor`接口的执行、自动化配置类的解析、条件注解的解析、国际化的初始化等等。

`run`方法中的Spring容器创建完成之后会调用`afterRefresh`方法，代码如下：

```java
protected void afterRefresh(ConfigurableApplicationContext context, ApplicationArguments args) {
    afterRefresh(context, args.getSourceArgs()); // 目前是个空实现
    
    // 调用Spring容器中的ApplicationRunner和CommandLineRunner接口的实现类
    callRunners(context, args); 
}

private void callRunners(ApplicationContext context, ApplicationArguments args) {
    List<Object> runners = new ArrayList<Object>();
    
    // 找出Spring容器中ApplicationRunner接口的实现类
    runners.addAll(context.getBeansOfType(ApplicationRunner.class).values());
    
    // 找出Spring容器中CommandLineRunner接口的实现类
    runners.addAll(context.getBeansOfType(CommandLineRunner.class).values());
    
    // 对runners进行排序
    AnnotationAwareOrderComparator.sort(runners);
    
    // 遍历runners依次执行
    for (Object runner : new LinkedHashSet<Object>(runners)) {
        if (runner instanceof ApplicationRunner) { // 如果是ApplicationRunner，进行ApplicationRunner的run方法调用
            callRunner((ApplicationRunner) runner, args);
        }
        
        if (runner instanceof CommandLineRunner) { // 如果是CommandLineRunner，进行CommandLineRunner的run方法调用
            callRunner((CommandLineRunner) runner, args);
        }
    }
}
```

这样`run`方法执行完成之后。Spring容器也已经初始化完成，各种监听器和初始化器也做了相应的工作。

## 总结

SpringBoot启动的时候，不论调用什么方法，都会构造一个`SpringApplication`的实例，然后调用这个实例的`run`方法，这样就表示启动SpringBoot。

在`run`方法调用之前，也就是构造`SpringApplication`的时候会进行初始化的工作，初始化的时候会做以下几件事：

1. 把参数`sources`设置到`SpringApplication`属性中，这个`sources`可以是任何类型的参数。本文的例子中这个sources就是MyApplication的class对象。
2. 判断是否是web程序，并设置到`webEnvironment`这个boolean属性中。
3. 找出所有的初始化器，默认有5个，设置到`initializers`属性中。
4. 找出所有的应用程序监听器，默认有9个，设置到listeners属性中。
5. 找出运行的主类\(main class\)。

SpringApplication构造完成之后调用run方法，启动SpringApplication，run方法执行的时候会做以下几件事：

1. 构造一个StopWatch，观察SpringApplication的执行
2. 找出所有的SpringApplicationRunListener并封装到SpringApplicationRunListeners中，用于监听run方法的执行。监听的过程中会封装成事件并广播出去让初始化过程中产生的应用程序监听器进行监听
3. 构造Spring容器\(ApplicationContext\)，并返回 3.1 创建Spring容器的判断是否是web环境，是的话构造AnnotationConfigEmbeddedWebApplicationContext，否则构造AnnotationConfigApplicationContext 3.2 初始化过程中产生的初始化器在这个时候开始工作 3.3 Spring容器的刷新\(完成bean的解析、各种processor接口的执行、条件注解的解析等等\)
4. 从Spring容器中找出ApplicationRunner和CommandLineRunner接口的实现类并排序后依次执行



**参考资料：**

* \*\*\*\*[**SpringBoot自动配置的原理详解**](https://zhuanlan.zhihu.com/p/136469945)\*\*\*\*
* \*\*\*\*[**SpringBoot源码分析之SpringBoot的启动过程**](https://fangjian0423.github.io/2017/04/30/springboot-startup-analysis/)\*\*\*\*

