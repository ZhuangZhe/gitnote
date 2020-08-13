# 泛型

泛型的设计是为了应用在Java的类型系统，**提供"用类型或者方法操作各种类型的对象从而提供编译期的类型安全功能。**但是在2016年的一些研究表明，泛型并不是在所有的情况下都能保证编译期的类型安全，例如切面\(Aspect\)编程的编译期类型安全并没有完全实现。

**泛型的一个最大的优点就是：提供编译期的类型安全**。

下面先列举出Java中泛型的一些事实：

* **Java虚拟机中不存在泛型，只有普通的类和方法，但是字节码中存放着泛型相关的信息**。
* 所有的类型参数都使用它们的限定类型替换。
* 桥方法\(Bridge Method\)由编译器合成，用于保持多态\(**Java虚拟机利用方法的参数类型、方法名称和方法返回值类型确定一个方法**\)。
* 为了保持类型的安全性，必要时需要进行类型的强制转换。

## 理解类型擦除

类型擦除的具体表现是：无论何时定义一个泛型类型，都自动提供一个相应的原始类型，原始类型的类名称就是带有泛型参数的类删去泛型参数后的类型名称，而原始类型会擦除\(Erased\)类型变量，并且把它们替换为限定类型\(如果没有指定限定类型，则擦除为Object类型\)。

### 例子

#### **擦除前**

```java
public class Pair<T>{

    private T first;
    private T second;

    public Pair(T first,T second){
        this.first = first;
        this.second = second;
    }

    public T getFirst(){
        return first;
    }

    public T getSecond(){
        return second;
    }
}
```

#### 擦除后

```java
public class Pair{

    private Object first;
    private Object second;

    public Pair(Object first,Object second){
        this.first = first;
        this.second = second;
    }

    public Object getFirst(){
        return first;
    }

    public Object getSecond(){
        return second;
    }
}
```

**如果泛型参数类型是有上限的，变量会擦除为上限的类型。**

#### **擦除前**

```java
public class Interval<T extends Comparable & Serializable> implements Serializable {

    private T lower;
    private T upper;
    
    public Interval(T lower, T upper) {
        this.lower = lower;
        this.upper = upper;
    }
    
    //省略其他方法
}
```

#### 擦除后

```java
public class Interval implements Serializable {

    private Comparable lower;
    private Comparable upper;
    
    public Interval(Comparable lower, Comparable upper) {
        this.lower = lower;
        this.upper = upper;
        }
    
    //省略其他方法
}
```

### 使用类型擦除的原因

在JDK1.5之前，也就是在泛型出现之前，所有的类型包括基本数据类型\(int、byte等\)、包装类型、其他自定义的类型等等都可以使用类文件\(`.class`\)字节码对应的`java.lang.Class`描述，也就是`java.lang.Class`类的一个具体实例对象就可以代表任意一个指定类型的原始类型。这里把泛型出现之前的所有类型暂时称为"历史原始类型"。

在JDK1.5之后，数据类型得到了扩充，出历史原始类型扩充了四种泛型类型：参数化类型\(`ParameterizedType`\)、类型变量类型\(`TypeVariable`\)、限定符类型\(`WildcardType`\)、泛型数组类型\(`GenericArrayType`\)。历史原始类型和新扩充的泛型类型都应该统一成各自的字节码文件类型对象，也就应该把泛型类型归并进去`java.lang.Class`中。但是由于JDK已经迭代了很多版本，泛型并不属于当前Java中的基本成分，如果JVM中引入真正的泛型类型，那么必须涉及到JVM指令集和字节码文件的修改\(这个修改肯定不是小的修改，因为JDK当时已经迭代了很多年，而类型是编程语言的十分基础的特性，引入泛型从项目功能迭代角度看可能需要整个JVM项目做回归测试\)，这个功能的代价十分巨大，所以Java没有在Java虚拟机层面引入泛型。

Java为了使用泛型，于是使用了类型擦除的机制引入了"泛型的使用"，并没有真正意义上引入和实现泛型。Java中的泛型实现的是编译期的类型安全，也就是泛型的类型安全检查是在编译期由编译器\(常见的是javac\)实现的，这样就能够确保数据基于类型上的安全性并且避免了强制类型转换的麻烦\(实际上，强制类型转换是由编译器完成了，只是不需要人为去完成而已\)。**一旦编译完成，所有的泛型类型都会被擦除，如果没有指定上限，就会擦除为Object类型，否则擦除为上限类型。**

既然Java虚拟机中不存在泛型，那么为什么可以从JDK中的一些类库获取泛型信息？这是因为类文件\(`.class`\)或者说字节码文件本身存储了泛型的信息，相关类库\(可以是JDK的类库，也可以是第三方的类库\)读取泛型信息的时候可以从字节码文件中提取，例如比较常用的字节码操作类库ASM就可以读取字节码中的信息甚至改造字节码动态生成类。例如前面提到的`Interval<T extends Comparable & Serializable>`类，使用`javap -c -v`命令查看其反编译得到的字节码信息，可以看到其签名如下：

```text
Signature: #22 
```

这里的签名信息实际上是保存在常量池中的，关于字节码文件的解析将来会出一个系列文章详细展开。

## Type体系

在JDK1.5中引入了四种新的泛型类型`java.lang.reflect.ParameterizedType`、`java.lang.reflect.TypeVariable`、`java.lang.reflect.WildcardType`、`java.lang.reflect.GenericArrayType`，包括原来存在的`java.lang.Class`，一共存在五种类型。为了程序的扩展性，引入了`java.lang.reflect.Type`类作为这五种类型的公共父接口，这样子就可以使用`java.lang.reflect.Type`类型参数去接收以上五种子类型的实参或者返回值，由此从逻辑上统一了泛型相关的类型和原始存在的`java.lang.Class`描述的类型。

![](../../../.gitbook/assets/image%20%2866%29.png)



* `ParameterizedType`、`TypeVariable`、`WildcardType`、`GenericArrayType`都是接口，它们位于`java.lang.reflect`包中。
* ParameterizedTypeImpl、TypeVariableImpl、WildcardTypeImpl、GenericArrayTypeImpl是四种泛型类型的实现，位于`sun.reflect.generics.reflectiveObjects`包中。

Type体系虽然看似很美好解决了泛型相关的类型和原始存在的`java.lang.Class`描述的类型的统一问题，但是引入了新的问题：如果一个方法返回值为`java.lang.reflect.Type`类型，或者一个方法的入参类型为`java.lang.reflect.Type`类型，这两种情况下，可能需要对`java.lang.reflect.Type`类型的对象做子类型判断，因为它的子类型有可能是上面提到的五种类型中的其中一种，这一点提高了编码的复杂性。

{% page-ref page="parameterizedtype.md" %}

{% page-ref page="typevariable.md" %}

{% page-ref page="wildcardtype.md" %}

{% page-ref page="genericarraytype.md" %}

## 泛型的约束

使用Java泛型的时候需要考虑一些限制，这些限制大多数是由泛型类型擦除引起的。

* 不能用基本类型实例化类型参数，也就是8种基本类型不能作为泛型参数，例如`Pair<int>`是非法的，会导致编译错误，而`Pair<Integer>`是合法的。
* 运行时的类型查询只能适用于原始类型\(非参数化类型\)。

```java
//下面的两种做法是错误的
if(a instanceof Pair<String>) //Error

if(a instanceof Pair<T>)  //Error

// 正确做法
if(a instanceof Pair)  //Right
```

* 不能创建参数化类型的数组，例如`Pair<String>[] arr = new Pair<String>[10]`是非法的。
* 不能实例化类型变量或者类型变量数组，例如`T t = new T()`或者`T[] arr = new T[10]`都是非法的。
* Varargs警告，这是因为第4点原因导致的，一般会发生在泛型类型变量作为可变参数的情况，例如`public static <T> addAll(Collection<T> con,T ... ts)`，第二个参数实际上就是泛型类型变量数组，但是这种情况是合法的，不过会受到编译器的警告，可以通过`@SuppressWarnings("unchecked")`注解或者`@SafeVarargs`注解标注该方法以消除警告。
* 不能在静态域或者方法中引用类型变量，例如`private static T singleInstance;`这样是非法的。
* 不能抛出或者抛出或者捕获泛型类型变量，但是如果在异常规范中使用泛型类型变量则是允许的，举两个例子仔细品味一下：

```java
// 反例
public static <T extends Throwable> void doWork(Class<T> t) {
    try{

    }catch(T t){  //Error

    }
}

// 正例
public static <T extends Throwable> void doWork(T t) throws T{
    try{

    }catch(Throwable e){  
       throw e;
    }
}
```

* 通过使用`@SuppressWarnings("unchecked")`注解可以消除Java类型系统的部分基本限制，一般使用在强制转换原始类型为泛型类型\(只是在编译层面告知编译器\)的情况，如：

```java
// 不加此注解会收到编译器的警告
@SuppressWarnings("unchecked")
public static <T extends Throwable> void throwAs(Throwable e){
    throw (T) e;
}
```

其实还有泛型的继承规则和通配符规则\(可以看下前面介绍的Type的子类型\)等等，这里不详细展开。



**参考资料：**

* \*\*\*\*[**深入分析Java反射\(三\)-泛型**](https://www.cnblogs.com/throwable/p/12315988.html)\*\*\*\*









