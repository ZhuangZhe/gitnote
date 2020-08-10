# 基本语法

## 声明注解与元注解

### 例子

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Test {

} 
```

我们使用了`@interface`声明了`Test`注解，并使用`@Target`注解传入`ElementType.METHOD`参数来标明`@Test`只能用于方法上，`@Retention(RetentionPolicy.RUNTIME)`则用来表示该注解生存期是运行时，从代码上看注解的定义很像接口的定义，确实如此，毕竟在编译后也会生成`Test.class`文件。对于`@Target`和`@Retention`是由Java提供的元注解，所谓元注解就是标记其他注解的注解，下面分别介绍

### @Target

用来约束注解可以应用的地方（如方法、类或字段），其中ElementType是枚举类型，其定义如下，也代表可能的取值范围。

```java
public enum ElementType {
    /**标明该注解可以用于类、接口（包括注解类型）或enum声明*/
    TYPE,

    /** 标明该注解可以用于字段(域)声明，包括enum实例 */
    FIELD,

    /** 标明该注解可以用于方法声明 */
    METHOD,

    /** 标明该注解可以用于参数声明 */
    PARAMETER,

    /** 标明注解可以用于构造函数声明 */
    CONSTRUCTOR,

    /** 标明注解可以用于局部变量声明 */
    LOCAL_VARIABLE,

    /** 标明注解可以用于注解声明(应用于另一个注解上)*/
    ANNOTATION_TYPE,

    /** 标明注解可以用于包声明 */
    PACKAGE,

    /**
     * 标明注解可以用于类型参数声明（1.8新加入）
     * @since 1.8
     */
    TYPE_PARAMETER,

    /**
     * 类型使用声明（1.8新加入)
     * @since 1.8
     */
    TYPE_USE
}
```

则此注解可以用于任何元素之上，多个值使用`{}`包含并用逗号隔开。

```java
@Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE})
```

### @Retention

用来约束注解的生命周期，分别有三个值，源码级别\(source\)，类文件级别\(class\)或者运行时级别\(runtime\)。

* `SOURCE`：注解将被编译器丢弃（该类型的注解信息只会保留在源码里，源码经过编译后，注解信息会被丢弃，不会保留在编译好的class文件里）
* `CLASS`：注解在class文件中可用，但会被VM丢弃（该类型的注解信息会保留在源码里和class文件里，在执行的时候，不会加载到虚拟机中），请注意，当注解未定义Retention值时，默认值是CLASS，如Java内置注解，`@Override`、`@Deprecated`、`@SuppressWarnning`等
* `RUNTIME`：注解信息将在运行期\(JVM\)也保留，因此可以通过反射机制读取注解的信息（源码、class文件和执行的时候都有注解的信息），如SpringMvc中的`@Controller`、`@Autowired`、`@RequestMapping`等。

## 注解元素及其数据类型

通过上述对`@Test`注解的定义，我们了解了注解定义的过程，由于`@Test`内部没有定义其他元素，所以`@Test`也称为标记注解\(marker annotation\)，但在自定义注解中，一般都会包含一些元素以表示某些值，方便处理器使用，这点在下面的例子将会看到：

```java
@Target(ElementType.TYPE)//只能应用于类上
@Retention(RetentionPolicy.RUNTIME)//保存到运行时
public @interface DBTable {

    String name() default "";
    
}
```

上述定义一个名为`DBTable`的注解，该用于主要用于数据库表与Bean类的映射（稍后会有完整案例分析），与前面`Test`注解不同的是，我们声明一个String类型的name元素，其默认值为空字符，但是必须注意到对应任何元素的声明应采用方法的声明方式，同时可选择使用default提供默认值，`@DBTable`使用方式如下：

```java
//在类上使用该注解
@DBTable(name = "MEMBER")
public class Member {
    //.......
}
```

### 支持的元素数据类型

* 所有基本类型\(int、float、boolean、byte、double、char、long、short\)
* String
* Class
* enum
* Annotation
* 上述类型的数组

倘若使用了其他数据类型，编译器将会丢出一个编译错误，注意，声明注解元素时可以使用基本类型但不允许使用任何包装类型，同时还应该注意到注解也可以作为元素的类型，也就是嵌套注解。

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@interface Reference{
    boolean next() default false;
}

public @interface AnnotationElementDemo {
    //枚举类型
    enum Status {FIXED,NORMAL};

    //声明枚举
    Status status() default Status.FIXED;

    //布尔类型
    boolean showSupport() default false;

    //String类型
    String name()default "";

    //class类型
    Class<?> testCase() default Void.class;

    //注解嵌套
    Reference reference() default @Reference(next=true);

    //数组类型
    long[] value();
}
```

## 编译器对默认值的限制

编译器对元素的默认值有些过分挑剔。首先，元素不能有不确定的值。也就是说，元素必须要么具有默认值，要么在使用注解时提供元素的值。其次，对于非基本类型的元素，无论是在源代码中声明，还是在注解接口中定义默认值，都不能以null作为值，这就是限制，没有什么利用可言，但造成一个元素的存在或缺失状态，因为每个注解的声明中，所有的元素都存在，并且都具有相应的值，为了绕开这个限制，只能定义一些特殊的值，例如空字符串或负数，表示某个元素不存在。

## 注解不支持继承

注解是不支持继承的，因此不能使用关键字extends来继承某个`@interface`，但注解在编译后，编译器会自动继承`java.lang.annotation.Annotation`接口，这里我们反编译前面定义的`DBTable`注解

```java
import java.lang.annotation.Annotation;
//反编译后的代码
public interface DBTable extends Annotation {

    public abstract String name();
    
}
```

虽然反编译后发现DBTable注解继承了`Annotation`接口，请记住，即使Java的接口可以实现多继承，但定义注解时依然无法使用extends关键字继承`@interface`。

## 快捷方式

所谓的快捷方式就是注解中定义了名为value的元素，并且在使用该注解时，如果该元素是唯一需要赋值的一个元素，那么此时无需使用`key=value`的语法，而只需在括号内给出value元素所需的值即可。这可以应用于任何合法类型的元素，记住，这限制了元素名必须为value，简单案例如下

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

//定义注解
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@interface IntegerVaule{
    int value() default 0;
    String name() default "";
}

//使用注解
public class QuicklyWay {

    //当只想给value赋值时,可以使用以下快捷方式
    @IntegerVaule(20)
    public int age;

    //当name也需要赋值时必须采用key=value的方式赋值
    @IntegerVaule(value = 10000, name = "MONEY")
    public int money;

}
```

## Java内置注解与其它元注解

接着看看Java提供的内置注解，主要有3个。

* `@Override`：用于标明此方法覆盖了父类的方法，源码如下

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```

* `@Deprecated`：用于标明已经过时的方法或类，源码如下，关于@Documented稍后分析：

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE})
public @interface Deprecated {
}
```

* `@SuppressWarnnings`：用于有选择的关闭编译器对类、方法、成员变量、变量初始化的警告，其实现源码如下：

```java
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
@Retention(RetentionPolicy.SOURCE)
public @interface SuppressWarnings {

    String[] value();

}

/*
其内部有一个String数组，主要接收值如下：
    deprecation：使用了不赞成使用的类或方法时的警告；
    unchecked：执行了未检查的转换时的警告，例如当使用集合时没有用泛型 (Generics) 来指定集合保存的类型; 
    fallthrough：当 Switch 程序块直接通往下一种情况而没有 Break 时的警告;
    path：在类路径、源文件路径等中有不存在的路径时的警告; 
    serial：当在可序列化的类上缺少 serialVersionUID 定义时的警告; 
    finally：任何 finally 子句不能正常完成时的警告; 
    all：关于以上所有情况的警告。
*/
```





**参考资料：**

* \*\*\*\*[**深入理解Java注解类型\(@Annotation\)**](https://blog.csdn.net/javazejian/article/details/71860633)\*\*\*\*

