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

###  <a id="type&#x4F53;&#x7CFB;"></a>







