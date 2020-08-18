# Java Config

## 是什么？

Spring JavaConfig是Spring社区的产品，它提供了配置Spring IoC容器的纯Java 方法。因此它有助于避免使用XML配置。

## 为什么用？

面向对象的配置。由于配置被定义为JavaConfig中的类，因此用户可以充分利用 Java 中的面向对象功能。一个配置类可以继承另一个，重写它的`@Bean`方法等。

减少或消除XML配置。基于依赖注入原则的外化配置的好处已被证明。但是，许多开发人员不希望在XML和Java之间来回切换。JavaConfig为开发人员提供了一种纯Java方法来配置与XML配置概念相似的Spring容器。从技术角度来讲，只使用JavaConfig配置类来配置容器是可行的，但实际上很多人认为将JavaConfig与XML混合匹配是理想的。

类型安全和重构友好。JavaConfig提供了一种类型安全的方法来配置Spring容器。由于Java 5.0对泛型的支持，现在可以按类型而不是按名称检索bean，不需要任何强制转换或基于字符串的查找。

## 怎么用？

### 基于Java类的配置选项

Spring 3.0引入了注解，配置文件的载体就从xml文件转换为了Java类，Java类就是一个普通的类，除了命名建议以`**Config`结尾方便识别外，Spring对其有一定的约定条件。

* 配置类不能是final类\(没法动态代理\)。
* 配置类必须是非本地的\(即不能将配置类定义在其他类的方法内部，不能是private\)。
* 配置类必须有一个无参构造函数。

### 基本使用方法

符合上述条件的类，就可以使用`@Configuration`来进行注解，表示这个类可以使用Spring IoC容器作为bean定义的来源。`@Bean`注解在该类的方法上，`AnnotationConfigApplicationContext`将配置类中标注了`@Bean`的方法的返回值识别为Spring Bean，并注册到容器中，归入IoC容器管理。

### **`@Configuration`**

`@Configuration`的作用等价于XML配置中的标签。

Spring在解析该类时，会识别出标注`@Bean`的所有方法，执行并将方法的返回值注册到IoC容器中。默认情况下，方法名即为Bean的名字。

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Configuration {
    String value() default "";
}
```

`@Configuration`注解本身定义时被`@Component`标注了，因此本质上来说`@Configuration`也是一个`@Component`，只不过我们在具体使用的过程中基本用不到它的实例化对象。

### `@Bean`

`@Bean`的作用等价于XML配置中的标签。

```java
@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Bean {

    @AliasFor("name")
    String[] value() default {};

    @AliasFor("value")
    String[] name() default {};

    Autowire autowire() default Autowire.NO;

    String initMethod() default "";

    String destroyMethod() default "(inferred)";
    
}
```

* `name`：指定一个或者多个Bean的名字。等价于XML配置中的name属性。
* `initMethod`：容器在初始化完Bean之后，会调用该属性指定的方法。等价于XML配置中的`init-method`属性。
* `destroyMethod`：该属性与`initMethod`功能相似，在容器销毁Bean之前，会调用该属性指定的方法。等价于XML配置中的`destroy-method`属性。
* `autowire`：指定Bean属性的自动装配策略，取值是Autowire类型的三个静态属性。`Autowire.BY_NAME`，`Autowire.BY_TYPE`，`Autowire.NO`。与XML配置中的`autowire`属性的取值相比，少了`constructor`，因为`constructor`在这里已经没有意义了。

### `@Scope`

`@Bean`默认是单例模式，并且没有提供指定作用域的属性，可以通过`@Scope`来实现该功能。



**参考资料：**

* \*\*\*\*[**Spring注解之@Configuration和@Bean使用详解**](https://blog.csdn.net/wo541075754/article/details/87922368)\*\*\*\*



