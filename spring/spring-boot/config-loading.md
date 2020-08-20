# 配置加载

在Spring Boot里面，可以使用以下几种方式来加载配置。

* properties文件
* YAML文件
* 系统环境变量
* 命令行参数

## YAML

YAML是一种人类可读的数据序列化语言。它通常用于配置文件。与属性文件相比，如果我们想要在配置文件中添加复杂的属性，YAML文件就更加结构化，而且更少混淆。可以看出YAML具有分层配置数据。

### 优势

* 配置有序，在一些特殊的场景下，配置有序很关键
* 支持数组，数组中的元素可以是基本数据类型也可以是对象
* 简洁

### 缺点

* 不支持`@PropertySource`注解导入自定义的 YAML 配置。

如果配置文件是基于`yml`文件类型，还可以将所有的配置放在同一个配置文件中：

```yaml
spring:
  profiles: 
    active: prod

server: 
  port: 18080
  
---
spring: 
  profiles: dev  
  
server: 
  port: 8080  
  
---
spring: 
  profiles: test  
  
server: 
  port: 8081
```

## 核心配置文件

Spring Boot 核心的两个配置文件：

* `bootstrap` \(`.yml`或者`.properties`\)：boostrap由父`ApplicationContext`加载的，比`applicaton`优先加载，配置在应用程序上下文的引导阶段生效。一般来说我们在Spring Cloud Config或者Nacos中会用到它。且boostrap里面的属性不能被覆盖；
* `application` \(`.yml`或者`. properties`\)： 由`ApplicatonContext`加载，用于spring boot项目的自动化配置。

## Profile

### Profile的作用

Profile对应中文并没有合适的翻译，它的主要作用就是让Spring Boot可以根据不同环境提供不同的配置功能支持。

我们经常遇到这样的场景：有开发、测试、生产等环境，不同的环境又有不同的配置。如果每个环境在部署时都需要修改配置文件将会非常麻烦，而通过Profile则可以轻松解决改问题。

### Profile的基本使用

比如上述环境，我们可以在Spring Boot中创建4个文件：

* `applcation.properties`：公共配置
* `application-dev.properties`：开发环境配置
* `application-test.properties`：测试环境配置
* `application-prod.properties`：生产环境配置

在`applcation.properties`中配置公共配置，然后通过如下配置激活指定环境的配置：

```yaml
spring.profiles.active = prod
```

其中“prod”对照文件名中`application-prod.properties`。Spring Boot在处理时会获取配置文件`applcation.properties`，然后通过指定的`profile`的值“prod”进行拼接，获得`application-prod.properties`文件的名称和路径。

举例说明，比如在开发环境使用服务的端口为8080，而在生产环境中需要使用18080端口。那么，在`application-prod.properties`中配置如下：

```yaml
server.port=18080
```

而在`application-prod.properties`中配置为：

```yaml
server.port=8080
```

在使用不同环境时，可以通过在`applcation.properties`中配置`spring.profiles.active`属性值来进行指定（如上面示例），也可以通过启动命令参数进行指定：

```bash
java -jar springboot.jar --spring.profiles.active=prod
```

这样打包后的程序只需通过命令行参数就可以使用不同环境的配置文件。

### 源码解析

在Spring Boot启动的过程中执行其`run`方法时，会执行如下一段代码：

```java
public ConfigurableApplicationContext run(String... args) {
    // ...
    try {
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
        ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
        // ...
    } 
    // ...
}
```

其中`prepareEnvironment`方法的相关代码如下：

```java
private ConfigurableEnvironment prepareEnvironment(SpringApplicationRunListeners listeners,
        ApplicationArguments applicationArguments) {
    // ...
    listeners.environmentPrepared(environment);
    // ...
}
```

在`prepareEnvironment`方法处理业务的过程中会调用`SpringApplicationRunListeners`的`environmentPrepared`方法发布事件。该方法会遍历注册在`spring.factories`中`SpringApplicationRunListener`实现类，然后调用其`environmentPrepared`方法。

其中`spring.factories`中`SpringApplicationRunListener`实现类注册为：

```java
org.springframework.boot.SpringApplicationRunListener=org.springframework.boot.context.event.EventPublishingRunListener
```

`SpringApplicationRunListeners`方法中的调用代码如下：

```java
void environmentPrepared(ConfigurableEnvironment environment) {
    for (SpringApplicationRunListener listener : this.listeners) {
        listener.environmentPrepared(environment);
    }
}
```

其中`listeners`便是注册的类的集合，这里默认只有`EventPublishingRunListener`。它的`environmentPrepared`方法实现为：

```java
@Override
public void environmentPrepared(ConfigurableEnvironment environment) {
    this.initialMulticaster
            .multicastEvent(new ApplicationEnvironmentPreparedEvent(this.application, this.args, environment));
}
```

其实就是发布了一个监听事件。该事件会被同样注册在`spring.factories`中的`ConfigFileApplicationListener`监听到：

```java
org.springframework.context.ApplicationListener=org.springframework.boot.context.config.ConfigFileApplicationListener
```

关于配置文件的核心处理便在`ConfigFileApplicationListener`中完成。在该类中我们可以看到很多熟悉的常量：

```java
public class ConfigFileApplicationListener implements EnvironmentPostProcessor, SmartApplicationListener, Ordered {

    private static final String DEFAULT_PROPERTIES = "defaultProperties";

    // Note the order is from least to most specific (last one wins)
    private static final String DEFAULT_SEARCH_LOCATIONS = "classpath:/,classpath:/config/,file:./,file:./config/";

    private static final String DEFAULT_NAMES = "application";
    // ...
}
```

比如Spring Boot默认寻找的配置文件的名称、默认扫描的类路径等。

不仅如此，该类还实现了监听器`SmartApplicationListener`接口和`EnvironmentPostProcessor`接口也就是拥有了监听器和环境处理的功能。

其`onApplicationEvent`方法对接收到事件进行判断，如果是`ApplicationEnvironmentPreparedEvent`事件则调用`onApplicationEnvironmentPreparedEvent`方法进行处理，代码如下：

```java
@Override
public void onApplicationEvent(ApplicationEvent event) {
    if (event instanceof ApplicationEnvironmentPreparedEvent) {
        onApplicationEnvironmentPreparedEvent((ApplicationEnvironmentPreparedEvent) event);
    }
    if (event instanceof ApplicationPreparedEvent) {
        onApplicationPreparedEvent(event);
    }
}
```

`onApplicationEnvironmentPreparedEvent`会获取到对应的`EnvironmentPostProcessor`并调用其`postProcessEnvironment`方法进行处理。而`loadPostProcessors`方法获取的`EnvironmentPostProcessor`正是在`spring.factories`中配置的当前类。

```java
private void onApplicationEnvironmentPreparedEvent(ApplicationEnvironmentPreparedEvent event) {
    List<EnvironmentPostProcessor> postProcessors = loadPostProcessors();
    postProcessors.add(this);
    AnnotationAwareOrderComparator.sort(postProcessors);
    for (EnvironmentPostProcessor postProcessor : postProcessors) {
        postProcessor.postProcessEnvironment(event.getEnvironment(), event.getSpringApplication());
    }
}
```

经过一系列的调用，最终调用到该类的如下方法：

```java
protected void addPropertySources(ConfigurableEnvironment environment, ResourceLoader resourceLoader) {
    RandomValuePropertySource.addToEnvironment(environment);
    new Loader(environment, resourceLoader).load();
}
```

其中`Loader`类为`ConfigFileApplicationListener`内部类，提供了具体处理配置文件优先级、profile、加载解析等功能。

比如，在`Loader`类的`load`方法中便有如下一段代码：

```java
for (PropertySourceLoader loader : this.propertySourceLoaders) {
    if (canLoadFileExtension(loader, location)) {
        load(loader, location, profile, filterFactory.getDocumentFilter(profile), consumer);
        return;
    }
}
```

该代码遍历`PropertySourceLoader`列表，并进行对应配置文件的解析，而这里的列表中的`PropertySourceLoader`同样配置在`spring.factories`中：

```text
org.springframework.boot.env.PropertySourceLoader=org.springframework.boot.env.PropertiesPropertySourceLoader,org.springframework.boot.env.YamlPropertySourceLoader
```

查看这两`PropertySourceLoader`的源代码，会发现SpringBoot默认支持的配置文件格式及解析方法。

```java
public class PropertiesPropertySourceLoader implements PropertySourceLoader {

    private static final String XML_FILE_EXTENSION = ".xml";

    @Override
    public String[] getFileExtensions() {
        return new String[] { "properties", "xml" };
    }

    @Override
    public List<PropertySource<?>> load(String name, Resource resource) throws IOException {
        Map<String, ?> properties = loadProperties(resource);
        // ...
    }

    @SuppressWarnings({ "unchecked", "rawtypes" })
    private Map<String, ?> loadProperties(Resource resource) throws IOException {
        String filename = resource.getFilename();
        if (filename != null && filename.endsWith(XML_FILE_EXTENSION)) {
            return (Map) PropertiesLoaderUtils.loadProperties(resource);
        }
        return new OriginTrackedPropertiesLoader(resource).load();
    }
}
```

比如`PropertiesPropertySourceLoader`中支持了xml和properties两种格式的配置文件，并分别提供了`PropertiesLoaderUtils`和`OriginTrackedPropertiesLoader`两个类进行相应的处理。

同样的`YamlPropertySourceLoader`支持yml和yaml格式的配置文件，并且采用`OriginTrackedYamlLoader`类进行解析。

```java
public class YamlPropertySourceLoader implements PropertySourceLoader {

    @Override
    public String[] getFileExtensions() {
        return new String[] { "yml", "yaml" };
    }

    @Override
    public List<PropertySource<?>> load(String name, Resource resource) throws IOException {
        // ...
        List<Map<String, Object>> loaded = new OriginTrackedYamlLoader(resource).load();
        // ...
    }
}
```

当然，在`ConfigFileApplicationListener`类中还实现了上面提到的如何拼接默认配置文件和profile的实现，相关代码如下：

```java
private void loadForFileExtension(PropertySourceLoader loader, String prefix, String fileExtension,
        Profile profile, DocumentFilterFactory filterFactory, DocumentConsumer consumer) {
    // ...
    if (profile != null) {
        String profileSpecificFile = prefix + "-" + profile + fileExtension;
        load(loader, profileSpecificFile, profile, defaultFilter, consumer);
        load(loader, profileSpecificFile, profile, profileFilter, consumer);
        // Try profile specific sections in files we've already processed
        for (Profile processedProfile : this.processedProfiles) {
            if (processedProfile != null) {
                String previouslyLoaded = prefix + "-" + processedProfile + fileExtension;
                load(loader, previouslyLoaded, profile, profileFilter, consumer);
            }
        }
    }
    // ...
}
```

`ConfigFileApplicationListener`类中还实现了其他更多的功能，大家感兴趣的话可以debug进行阅读。



**参考资料：**

* \*\*\*\*[**SpringBoot Profile使用详解及配置源码解析**](https://juejin.im/post/6844904029949001742)\*\*\*\*

