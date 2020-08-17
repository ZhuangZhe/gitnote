# 常用注解

## Spring

| 注解 | 简介 |
| :--- | :--- |
|  |  |

## Spring Boot

| 注解 | 简介 |
| :--- | :--- |
| **`@SpringBootApplication`** | 这是Spring Boot最核心的注解，用在Spring Boot主类上，标识这是一个Spring Boot应用，用来开启Spring Boot的各项能力。其实这个注解就是 `@SpringBootConfiguration`、`@EnableAutoConfiguration`、`@ComponentScan` 这三个注解的组合。 |
| **`@EnableAutoConfiguration`** | 允许Spring Boot自动配置注解，开启这个注解之后，Spring Boot就能根据当前类路径下的包或者类来配置Spring Bean。 |
| **`@Configuration`** | 用来代替`applicationContext.xml`配置文件，所有这个配置文件里面能做到的事情都可以通过这个注解所在类来进行注册。 |
| **`@SpringBootConfiguration`** | 这个注解就是`@Configuration`注解的变体，只是用来修饰是Spring Boot配置而已，或者可利于Spring Boot后续的扩展。 |
| **`@ComponentScan`** | 用来代替配置文件中的`component-scan`配置，开启组件扫描，即自动扫描包路径下的`@Component`注解进行注册bean实例到context中。 |
| **`@Conditional`** | 用来标识一个Spring Bean或者Configuration配置文件，当满足指定的条件才开启配置。 |
| **`@ConditionalOnBean`** | 当容器中有指定的Bean才开启配置。 |
| **`@ConditionalOnMissingBean`** | 和`@ConditionalOnBean`注解相反，当容器中没有指定的Bean才开启配置。 |
| **`@ConditionalOnClass`** | 当容器中有指定的Class才开启配置。 |
| **`@ConditionalOnMissingClass`** | 和`@ConditionalOnMissingClass`注解相反，当容器中没有指定的Class才开启配置。 |
| **`@ConditionalOnWebApplication`** | 当前项目类型是WEB项目才开启配置。有3种类型：ANY、SERVLET、REACTIVE。 |
| **`@ConditionalOnNotWebApplication`** | 和`@ConditionalOnWebApplication`注解相反，当前项目类型不是WEB项目才开启配置。 |
| **`@ConditionalOnProperty`** | 当指定的属性有指定的值时才开启配置。 |
| **`@ConditionalOnExpression`** | 当SpEL表达式为true时才开启配置。 |
| **`@ConditionalOnJava`** | 当运行的Java JVM在指定的版本范围时才开启配置。 |
| **`@ConditionalOnResource`** | 当类路径下有指定的资源才开启配置。 |
| **`@ConditionalOnJndi`** | 当指定的JNDI存在时才开启配置。 |
| **`@ConditionalOnCloudPlatform`** | 当指定的云平台激活时才开启配置。 |
| **`@ConditionalOnSingleCandidate`** | 当指定的class在容器中只有一个Bean，或者同时有多个但为首选时才开启配置。 |
| **`@ConfigurationProperties`** | 用来加载额外的配置**\(**如 .properties 文件\)，可用在`@Configuration`注解类，或者`@Bean`注解方法上面。 |
| **`@EnableConfigurationProperties`** | 一般要配合`@ConfigurationProperties`注解使用，用来开启对`@ConfigurationProperties`注解配置Bean的支持。 |
| **`@AutoConfigureAfter`** | 用在自动配置类上面，表示该自动配置类需要在另外指定的自动配置类配置完之后。 |
| **`@AutoConfigureBefore`** | 这个和`@AutoConfigureAfter`注解使用相反，表示该自动配置类需要在另外指定的自动配置类配置之前。 |
| **`@Import`** | 用来导入一个或者多个`@Configuration`注解修饰的类，这在Spring Boot里面应用很多。 |
| **`@ImportResource`** | 用来导入一个或者多个Spring配置文件，这对Spring Boot兼容老项目非常有用，因为有些配置无法通过Java Config的形式来配置就只能用这个注解来导入。 |







**参考资料：**

* \*\*\*\*[**Spring Boot 最核心的 25 个注解，都是干货！**](https://zhuanlan.zhihu.com/p/57689422)\*\*\*\*

