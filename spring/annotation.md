# 常用注解

## Spring

| 注解 | 简介 |
| :--- | :--- |
| **`@Component`** | 表示带注释的类是“组件”。当使用基于注释的配置和类路径扫描时，这些类被视为自动检测的候选者。 |
| **`@Repository`** | 表示带注释的类是“存储库”。在注解了`@Repository`的类上如果数据库操作中抛出了异常，就能对其进行处理，转而抛出的是翻译后的spring专属数据库异常，方便我们对异常进行排查处理。这个注释也可以作为一个特化`@Component`，允许通过类路径扫描自动检测实现类。 |
| **`@Controller`** | 表示带注释的类是“控制器”。此注释用作特殊化`@Component`，允许通过类路径扫描自动检测实现类。它通常与基于`@RequestMapping`注释的带注释的处理程序方法结合使用 。 |
| **`@Service`** | 表示带注释的类是“服务”。也可能表明一个类是“Business Service Facade”或类似的东西。此注释用作特殊化`@Component`，允许通过类路径扫描自动检测实现类。 |
| **`@Autowired`** | 枚举确定自动装配状态。即，bean是否应该使用setter注入由Spring容器自动注入其依赖项。`@Autowired`默认是按照类型装配注入的。这是Spring DI的核心概念。 |
| **`@Resource`** | `@Resource`通过 `CommonAnnotationBeanPostProcessor`类实现依赖注入。`@Resource`标记应用程序所需的资源。此注释可以应用于应用程序组件类，或应用于组件类的字段或方法。当注释应用于字段或方法时，容器将在初始化组件时将所请求资源的实例注入应用程序组件。 如果注释应用于组件类，则应用程序会在运行时寻找资源。即使此注释未标记为继承，也需要部署工具检查任何组件类的所有超类，以发现此注释在所有超类中的所有用法。 所有这些注释实例都指定应用程序组件所需的资源。此注释可能出现在超类的私有字段和方法上；在这些情况下，容器也需要进行注射。 |
| **`@Inject`** | 通过`AutowiredAnnotationBeanPostProcessor`类实现的依赖注入。属于类型注入。 |
| **`@Configuration`** | `@Configuration`是把一个类变成一个配置类，即在这个类中可以用`@Bean`标识方法，并且把方法返回的对象加入到spring容器中，并且返回的是同一个实例。配置类必须以类的形式提供\(不能是工厂方法返回的实例\)，允许通过生成子类在运行时增强\(cglib 动态代理\)。配置类不能是final类\(没法动态代理\)。配置注解通常为了通过`@Bean`注解生成Spring容器管理的类。配置类必须是非本地的\(即不能在方法中声明，不能是 private\)。任何嵌套配置类都必须声明为static。`@Bean`方法可能不会反过来创建进一步的配置类\(也就是返回的bean如果带有`@Configuration`，也不会被特殊处理，只会作为普通的bean\)。 |
| **`@ComponentScan`** | `@ComponentScan`主要是定义扫描的路径并从中找出标识了需要装配的类自动装配到spring的bean容器中。 |
| **`@Bean`** | `@Bean`用于注释方法，表示该方法返回的Bean会被放入spring容器中。 |
| **`@Aspect`** | `@Aspect`的作用是把当前类标识为一个切面类供容器读取。 |
| **`@After`** | final增强，不管是抛出异常或者正常退出都会执行。 |
| **`@Before`** | 标识一个前置增强方法，相当于BeforeAdvice的功能。 |
| **`@Around`** | 环绕增强，相当于MethodInterceptor。 |
| **`@PointCut(AspectJ)`** | 声明一个切入点。 |
| **`@Transactional`** | 描述方法或类的事务属性。 这个注释类型通常可以直接与Spring的`RuleBasedTransactionAttribute`类相媲美 ，实际上它`AnnotationTransactionAttributeSource`会直接将数据转换为后一个类，因此Spring的事务支持代码不必知道注释。如果没有规则相关的例外，它会像对待`DefaultTransactionAttribute`\(回滚到`RuntimeException`和`Error`，但不会对检查的异常\)。 |
| **`@Cacheable`** | 注释指示可以缓存调用方法\(或类中的所有方法\)的结果。 每次调用一个建议的方法时，都会应用缓存行为，检查是否已经为给定的参数调用了该方法。合理的默认值只是使用方法参数来计算键，但可以通过`key()`属性提供SpEL表达式，或者自定义`KeyGenerator`实现可以替换默认值。 如果在计算键的高速缓存中未找到任何值，则将调用目标方法并将返回的值存储在关联的高速缓存中。请注意，Java8的Optional返回类型会自动处理，其内容存储在缓存中（如果存在）。 |
| **`@EnableAspectJAutoProxy`** | 支持处理使用AspectJ的`@Aspect`注释标记的组件，类似于Spring的`aop:aspectj-autoproxyXML`元素中的功能。一般和`@Configuration`一起使用。 |
| **`@Value`** | 字段或方法/构造函数参数级别的注释，指示受影响参数的默认值表达式。 通常用于表达式驱动的依赖注入。还支持动态解析处理程序方法参数，例如在Spring MVC中。 常见的用例是使用`＃{systemProperties.myProp}`样式表达式分配默认字段值。 |
| **`@PropertySource`** | 注释提供了一个方便的声明机制，用于添加一个`PropertySource`到Spring的Environment。与`@Configuration`一起使用。 |
| **`@PostConstruct`** | 被`@PostConstruct`修饰的方法会在服务器加载Servlet的时候运行，并且只会被服务器执行一次。`PostConstruct`在构造函数之后执行，`init()`方法之前执行。 |
| **`@PreDestroy`** | `PreDestroy()`方法在`destroy()`方法之后执行。 |
| **`@Profile`** | 表示当一个或多个指定的文件处于活动状态时，这个组件是有资格注册的。使用`@Profile`注解类或者方法，达到在不同情况下选择实例化不同的Bean。`@Profile(“dev”)`表示当环境为dev时实例化。 |
| **`@EnableAsync`** | 启用Spring的异步方法执行功能，类似于Spring的`task:*`XML命名空间中的功能。 要与`@Configuration`一起使用。 |
| **`@Async`** | 将方法标记为异步执行候选的注释。也可以在class级别使用，在这种情况下，所有类型的方法都被视为异步。 就目标方法签名而言，支持任何参数类型。但是，返回类型被限制为void或者`Future`。在后一种情况下，您可以声明更具体的类型`ListenableFuture`或`CompletableFuture`类型，以允许与异步任务进行更丰富的交互，并通过进一步的处理步骤立即组合。 `Future`从代理返回的句柄将是一个实际的异步 `Future`，可用于跟踪异步方法执行的结果。但是，由于目标方法需要实现相同的签名，因此必须返回一个临时`Future`句柄，该句柄只传递一个值：例如Spring `AsyncResult`，EJB 3.1 `AsyncResult`或者`CompletableFuture.completedFuture(Object)`。 |
| **`@EnableScheduling`** | 启用Spring的计划任务执行功能，类似于Spring的`task:*`XML命名空间中的功能。要在和`@Configuration`一同使用。 |
| **`@Scheduled`** | 用于标记一个需要定期执行的方法。`cron()`，`fixedDelay()`或`fixedRate()`中的至少一个属性必须指定参数。 带注释的方法必须没有参数。它通常会有一个void返回类型；如果不是，则通过调度程序调用时将忽略返回的值。 通过注册`ScheduledAnnotationBeanPostProcessor`来执行`@Scheduled`注释的处理。这可以手动完成，也可以通过元素或`@EnableScheduling`注释更方便地完成。 此注释可用作元注释，以创建具有属性覆盖的自定义组合注释。 |
| **`@Enable*`** | `@enable*`是springboot中用来启用某一个功能特性的一类注解。其中包括我们常用的`@SpringBootApplication`注解中用于开启自动注入的注解`@EnableAutoConfiguration`，开启异步方法的注解`@EnableAsync`，开启将配置文件中的属性以bean的方式注入到IOC容器的注解`@EnableConfigurationProperties`等。 |
| **`@RunWith`** | JUnit用例都是在Runner\(运行器\)来执行的。通过它，可以为这个测试类指定一个特定的Runner。 |
| **`@ContextConfiguration`** | `@ContextConfiguration`定义class级元数据，用于确定如何加载和配置`ApplicationContext`集成测试。 在Spring 3.1之前，仅支持基于路径的资源位置\(通常是XML配置文件\)。在Spring 3.1中，背景装载机可以选择支持任何基于路径或基于类的资源。在Spring 4.0.4中，上下文装载机可以选择支持基于路径 和同时基于类的资源。因此`@ContextConfiguration`可用于声明基于路径的资源位置\(通过`locations()`或`value()`属性\)或带 注释的类\(通过`classes()`属性\)。但请注意，大多数实现`SmartContextLoader`仅支持单一资源类型。从Spring 4.1开始，基于路径的资源位置可以是XML配置文件或Groovy脚本\(如果Groovy在类路径上\)。当然，第三方框架可以选择支持其他类型的基于路径的资源。 |
| **`@ActiveProfiles`** | `@ActiveProfiles`是一个class级别注释，用于声明在加载 `ApplicationContext`来测试时应使用哪些活动Bean定义配置文件。 |
| **`@EnableWebMvc`** | 将此注释添加到带有`@Configuration`的类中会从`WebMvcConfigurationSupport`中导入Spring MVC配置。 要自定义导入的配置，请实现接口`WebMvcConfigurer`并重写要自定义的方法。 |
| **`@RequestMapping`** | 使用灵活方法签名将Web请求映射到请求处理类中的方法的注释。 |
| **`@GetMapping`** | 用于将GET类型的HTTP请求映射到特定处理方法的注释。 具体来说，`@GetMapping`是一个作为快捷方式的组合注释`@RequestMapping(method = RequestMethod.GET)`。 |
| **`@PostMapping`** | 用于将POST类型的HTTP请求映射到特定处理方法的注释。 具体来说，`@PostMapping`是一个作为快捷方式的组合注释`@RequestMapping(method = RequestMethod.POST)`。 |
| **`@ResponseBody`** | 该注解用于将`Controller`的方法返回的对象，通过适当的`HttpMessageConverter`转换为指定格式后，写入到`Response`对象的body数据区。 从版本4.0开始，此注释也可以添加到类型级别，在这种情况下，它是继承的，不需要在方法级别添加。 |
| **`@RequestBody`** | 该注解用于读取`Request`请求的body部分数据，使用系统默认配置的`HttpMessageConverter`进行解析，然后把相应的数据绑定到要返回的对象上。 再把`HttpMessageConverter`返回的对象数据绑定到`controller`中方法的参数上。 |
| **`@PathVariable`** | 当使用`@RequestMapping`URI占位符映射时，Url中可以通过一个或多个`{xxxx}`占位符映射，通过`@PathVariable`可以绑定占位符参数到方法参数中。 |
| **`@RestController`** | 组合了`@Controller`和`@ResponseBody`的注释。 带有此注释的类型被视为控制器，其中@RequestMapping和@ResponseBody采用默认设置。 |
| **`@ControllerAdvice`** | `@ControllerAdvice`是一个`@Component`，用于定义`@ExceptionHandler`，`@InitBinder`和`@ModelAttribute`方法，适用于所有使用`@RequestMapping`方法。 Spring4之前，`@ControllerAdvice`在同一调度的Servlet中协助所有控制器。Spring4已经改变：`@ControllerAdvice`支持配置控制器的子集，而默认的行为仍然可以利用。 在Spring4中， `@ControllerAdvice`通过`annotations()`、`basePackageClasses()`、`basePackages()`方法定制用于选择控制器子集。 |
| **`@ExceptionHandler`** | 用于处理特定的类和/或方法中的异常的注释。 使用此注释的方法允许具有灵活的签名。 |
| **`@ModelAttribute`** | 将方法参数或方法返回值绑定到命名模型属性的注释，公开给Web视图。支持带`@RequestMapping`方法的控制器类。 可以使用特定的属性名称，通过注释`@RequestMapping`方法的相应参数，将命令对象公开给Web视图。 也可以通过使用`@RequestMapping`方法在控制器类中注释访问器方法，将引用数据公开给Web视图。允许这样的访问器方法具有`@RequestMapping`方法支持的任何参数， 将模型属性值返回并公开。 但请注意，当请求处理导致异常时，Web视图无法使用引用数据和所有其他模型内容，因为可能在任何时候引发异常，从而使模型的内容不可靠。因此，`@ExceptionHandler`方法不提供对Model参数的访问。 |
| **`@InitBinder`** | 用于标识初始化WebDataBinder的方法，该方法将用于填充带注释的方法的命令和表单对象参数。 这样的`init-binder`方法支持`RequestMapping`支持的所有参数，命令/表单对象和相应的验证结果对象除外。 `Init-binder`方法不能有返回值;它们通常被宣布为无效。 典型的参数是`WebDataBinder`与`WebRequest`或`Locale`的组合，允许注册特定于上下文的编辑器。 |
| `@WebAppConfiguration` | `@WebAppConfiguration`是一个类级别注释，用于声明为集成测试加载的`ApplicationContext`应该是`WebApplicationContext`。 测试类上存在`@WebAppConfiguration`指示应使用Web应用程序根路径的默认值为测试加载`WebApplicationContext`。要覆盖默认值，请通过`value()`属性指定显式资源路径。 请注意，`@WebAppConfiguration`必须与`@ContextConfiguration`结合使用，可以在单个测试类中，也可以在测试类层次结构中使用。 |
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
* \*\*\*\*[**Spring 注解大全与详解**](https://juejin.im/post/6844903907173335047)\*\*\*\*
