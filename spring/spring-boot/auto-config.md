# 自动装配

## 是什么？

springboot的自动配置，会自动将一些配置类的bean注册进IOC容器，我们可以需要的地方使用`@autowired`或者`@resource`等注解来使用它。

​ “自动”的表现形式就是我们只需要引我们想用功能的包，相关的配置我们完全不用管，springboot会自动注入这些配置bean，我们直接使用这些bean即可。

## 为什么用？



## 原理

Spring Boot程序都使用`@SpringBootApplication`标记程序入口。

`@SpringBootApplication`注解中包含了`@EnableAutoConfiguration`注解。

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
    // ...
```

 而在`@EnableAutoConfiguration`中又导入了`AutoConfigurationImportSelector`这个类。

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    // ...
```



```java
/**
* 方法用于给容器中导入组件
**/
@Override
public String[] selectImports(AnnotationMetadata annotationMetadata) {
    if (!isEnabled(annotationMetadata)) {
        return NO_IMPORTS;
    }
    AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
        .loadMetadata(this.beanClassLoader);
    AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(
        autoConfigurationMetadata, annotationMetadata);  // 获取自动配置项
    return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
}

// 获取自动配置项
protected AutoConfigurationEntry getAutoConfigurationEntry(
    AutoConfigurationMetadata autoConfigurationMetadata,
    AnnotationMetadata annotationMetadata) {
    if (!isEnabled(annotationMetadata)) {
        return EMPTY_ENTRY;
    }
    AnnotationAttributes attributes = getAttributes(annotationMetadata);
    List <String> configurations = getCandidateConfigurations(annotationMetadata,
        attributes);  //  获取一个自动配置 List ，这个 List 就包含了所有自动配置的类名
    configurations = removeDuplicates(configurations);
    Set <String> exclusions = getExclusions(annotationMetadata, attributes);
    checkExcludedClasses(configurations, exclusions);
    configurations.removeAll(exclusions);
    configurations = filter(configurations, autoConfigurationMetadata);
    fireAutoConfigurationImportEvents(configurations, exclusions);
    return new AutoConfigurationEntry(configurations, exclusions);
}

//   获取一个自动配置 List ，这个 List 就包含了所有的自动配置的类名
protected List <String> getCandidateConfigurations(AnnotationMetadata metadata,
    AnnotationAttributes attributes) {
    // 通过 getSpringFactoriesLoaderFactoryClass 获取默认的 EnableAutoConfiguration.class 类名，传入 loadFactoryNames 方法
    List <String> configurations = SpringFactoriesLoader.loadFactoryNames(
        getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());
    Assert.notEmpty(configurations,
        "No auto configuration classes found in META-INF/spring.factories. If you " +
        "are using a custom packaging, make sure that file is correct.");
    return configurations;
}

// 默认的 EnableAutoConfiguration.class 类名
protected Class<?> getSpringFactoriesLoaderFactoryClass() {
	return EnableAutoConfiguration.class;
}

```

1. 首先注意到`selectImports`方法，其实从方法名就能看出，这个方法用于给容器中导入组件，然后跳到`getAutoConfigurationEntry`方法就是用于获取自动配置项的。
2. 再来进入`getCandidateConfigurations`方法就是 获取一个自动配置List ，这个List就包含了所有的自动配置的类名 。
3. 再进入`SpringFactoriesLoader`类的`loadFactoryNames`方法，跳转到`loadSpringFactories`方法发现`ClassLoader`类加载器指定了一个`FACTORIES_RESOURCE_LOCATION`常量。
4. 然后利用`PropertiesLoaderUtils`把`ClassLoader`扫描到的这些文件的内容包装成`properties`对象，从`properties`中获取到`EnableAutoConfiguration.class`对应的值，然后把他们添加在容器中。

```java
public final class SpringFactoriesLoader {
    public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";
    private static final Log logger = LogFactory.getLog(SpringFactoriesLoader.class);
    private static final Map<ClassLoader, MultiValueMap<String, String>> cache = new ConcurrentReferenceHashMap();

    private SpringFactoriesLoader() {}

    public static <T> List<T> loadFactories(Class<T> factoryType, @Nullable ClassLoader classLoader) {
        Assert.notNull(factoryType, "'factoryType' must not be null");
        ClassLoader classLoaderToUse = classLoader;
        if (classLoader == null) {
            classLoaderToUse = SpringFactoriesLoader.class.getClassLoader();
        }

        List<String> factoryImplementationNames = loadFactoryNames(factoryType, classLoaderToUse);
        if (logger.isTraceEnabled()) {
            logger.trace("Loaded [" + factoryType.getName() + "] names: " + factoryImplementationNames);
        }

        List<T> result = new ArrayList(factoryImplementationNames.size());
        Iterator var5 = factoryImplementationNames.iterator();

        while(var5.hasNext()) {
            String factoryImplementationName = (String)var5.next();
            result.add(instantiateFactory(factoryImplementationName, factoryType, classLoaderToUse));
        }

        AnnotationAwareOrderComparator.sort(result);
        return result;
    }

    public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
        String factoryTypeName = factoryType.getName();
        return (List)loadSpringFactories(classLoader).getOrDefault(factoryTypeName, Collections.emptyList());
    }

    private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
        MultiValueMap<String, String> result = (MultiValueMap)cache.get(classLoader);
        if (result != null) {
            return result;
        } else {
            try {
                Enumeration<URL> urls = classLoader != null ? classLoader.getResources("META-INF/spring.factories") : ClassLoader.getSystemResources("META-INF/spring.factories");
                LinkedMultiValueMap result = new LinkedMultiValueMap();

                while(urls.hasMoreElements()) {
                    URL url = (URL)urls.nextElement();
                    UrlResource resource = new UrlResource(url);
                    Properties properties = PropertiesLoaderUtils.loadProperties(resource);
                    Iterator var6 = properties.entrySet().iterator();

                    while(var6.hasNext()) {
                        Entry<?, ?> entry = (Entry)var6.next();
                        String factoryTypeName = ((String)entry.getKey()).trim();
                        String[] var9 = StringUtils.commaDelimitedListToStringArray((String)entry.getValue());
                        int var10 = var9.length;

                        for(int var11 = 0; var11 < var10; ++var11) {
                            String factoryImplementationName = var9[var11];
                            result.add(factoryTypeName, factoryImplementationName.trim());
                        }
                    }
                }

                cache.put(classLoader, result);
                return result;
            } catch (IOException var13) {
                throw new IllegalArgumentException("Unable to load factories from location [META-INF/spring.factories]", var13);
            }
        }
    }

    private static <T> T instantiateFactory(String factoryImplementationName, Class<T> factoryType, ClassLoader classLoader) {
        try {
            Class<?> factoryImplementationClass = ClassUtils.forName(factoryImplementationName, classLoader);
            if (!factoryType.isAssignableFrom(factoryImplementationClass)) {
                throw new IllegalArgumentException("Class [" + factoryImplementationName + "] is not assignable to factory type [" + factoryType.getName() + "]");
            } else {
                return ReflectionUtils.accessibleConstructor(factoryImplementationClass, new Class[0]).newInstance();
            }
        } catch (Throwable var4) {
            throw new IllegalArgumentException("Unable to instantiate factory class [" + factoryImplementationName + "] for factory type [" + factoryType.getName() + "]", var4);
        }
    }
}
```

* `FACTORIES_RESOURCE_LOCATION`指定的是 jar 包类路径下`META-INF/spring.factories`。
* 将类路径下`META-INF/spring.factories`里面配置的所 `EnableAutoConfiguration`的值加入到了容器中。

## 举例说明Http编码自动配置原理

```java
@Configuration 
// 表示这是一个配置类，以前编写的配置文件一样，也可以给容器中添加组件

@EnableConfigurationProperties(HttpEncodingProperties.class)  
// 启动指定类的 ConfigurationProperties 功能；将配置文件中对应的值和 HttpEncodingProperties 绑定起来；并把 HttpEncodingProperties 加入到 ioc 容器中

@ConditionalOnWebApplication 
// Spring 底层 @Conditional 注解，根据不同的条件，如果满足指定的条件，整个配置类里面的配置就会生效；判断当前应用是否是 web 应用，如果是，当前配置类生效

@ConditionalOnClass(CharacterEncodingFilter.class) 
// 判断当前项目有没有这个类 CharacterEncodingFilter；SpringMVC 中进行乱码解决的过滤器；

@ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true) 
// 判断配置文件中是否存在某个配置  spring.http.encoding.enabled；如果不存在，判断也是成立的
// 即使我们配置文件中不配置 pring.http.encoding.enabled=true，也是默认生效的；
public class HttpEncodingAutoConfiguration {

    // 已经和 SpringBoot 的配置文件建立映射关系了
    private final HttpEncodingProperties properties;

    //只有一个有参构造器的情况下，参数的值就会从容器中拿
    public HttpEncodingAutoConfiguration(HttpEncodingProperties properties) {
        this.properties = properties;
    }

    @Bean 
    // 给容器中添加一个组件，这个组件的某些值需要从 properties 中获取
    @ConditionalOnMissingBean(CharacterEncodingFilter.class) 
    public CharacterEncodingFilter characterEncodingFilter() {
        CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
        filter.setEncoding(this.properties.getCharset().name());
        filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
        filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
        return filter;
    }

    // ...
```

请见上面代码。所有在配置文件中能配置的属性都是在`xxxxProperties`类中封装的；配置文件能配置什么就可以参照某个功能对应的这个属性类，例如上述提到的`@EnableConfigurationProperties(HttpProperties.class)`。

```java
@ConfigurationProperties(prefix = "spring.http")
public class HttpProperties {

	public static class Encoding {

		public static final Charset DEFAULT_CHARSET = StandardCharsets.UTF_8;

		/**
		 * Charset of HTTP requests and responses. Added to the "Content-Type" header if
		 * not set explicitly.
		 */
		private Charset charset = DEFAULT_CHARSET;

		/**
		 * Whether to force the encoding to the configured charset on HTTP requests and
		 * responses.
		 */
		private Boolean force;

}
```

在上面可以发现里面的属性`charset`、`force`等，都是我们可以在配置文件中指定的，它的前缀就是`spring.http.encoding`。

## 总结

1. **SpringBoot 启动会加载大量的自动配置类**
2. **我们看我们需要的功能有没有SpringBoot默认写好的自动配置类；**
3. **我们再来看这个自动配置类中到底配置了哪些组件\(只要我们要用的组件有，我们就不需要再来配置，若没有，我们可能就要考虑自己写一个配置类让SpringBoot扫描了\)**
4. **给容器中自动配置类添加组件的时候，会从properties类中获取某些属性。我们就可以在配置文件中指定这些属性的值；**

`xxxxAutoConfigurartion`自动配置类的作用就是**给容器中添加组件**

`xxxxProperties`的作用就是**封装配置文件中相关属性**







**参考资料：**

* \*\*\*\*[**SpringBoot \| 自动配置原理**](https://juejin.im/post/6844903812788912141)\*\*\*\*
* \*\*\*\*[**SpringBoot自动配置原理**](https://segmentfault.com/a/1190000018011535)\*\*\*\*
* \*\*\*\*[**SpringBoot自动配置的原理详解**](https://zhuanlan.zhihu.com/p/136469945)\*\*\*\*

