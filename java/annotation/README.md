# 注解

## 注解的声明

```java
{InterfaceModifier} @ interface Identifier AnnotationTypeBody

接口修饰符 @interface 注解标识符 注解类型的内容
```

* 注解类型声明中的标识符指定了注解类型的名称。
* 如果注解类型与它的任何封闭类或接口具有相同的简单名称，则编译时会出现错误。
* 每个注解类型的直接父接口都是`java.lang.annotation.Annotation`。

既然所有注解类型的父接口都是`java.lang.annotation.Annotation`。

{% page-ref page="annotation-class.md" %}

## 注解实现 <a id="&#x6CE8;&#x89E3;&#x5B9E;&#x73B0;&#x63A2;&#x7A76;"></a>

我们先定义一个十分简单的Counter注解如下：

```java
package club.throwable.annotation;

import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Documented
@Target(ElementType.TYPE)
public @interface Counter {

    int count() default 0;
	
}
```

我们先从直接使用`@Counter`注解，从直观上观察`@Counter`实例的类型：

```java
@Counter(count = 1)
public class Main {

    public static void main(String[] args) throws Exception{
        Counter counter = Main.class.getAnnotation(Counter.class);
        System.out.println(counter.count());
    }

}
```

`@Counter`实例从Debug过程中观察发现是JDK的一个代理类\(并且`InvocationHandler`的实例是`sun.reflect.annotation.AnnotationInvocationHandler`，它是一个修饰符为default的sun包内可用的类\)，为了验证这一点我们使用JDK的反编译命令查看`@Counter`的字节码：

```bash
javap -c -v D:\Projects\rxjava-seed\target\classes\club\throwable\annotation\Counter.class
```

`@Counter`反编译后的字节码如下：

```java
Classfile /D:/Projects/rxjava-seed/target/classes/club/throwable/annotation/Counter.class
  Last modified 2018-10-6; size 487 bytes
  MD5 checksum 83cee23f426e5b51a096281068d8b555
  Compiled from "Counter.java"
public interface club.throwable.annotation.Counter extends java.lang.annotation.Annotation
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_INTERFACE, ACC_ABSTRACT, ACC_ANNOTATION
Constant pool:
   #1 = Class              #19            // club/throwable/annotation/Counter
   #2 = Class              #20            // java/lang/Object
   #3 = Class              #21            // java/lang/annotation/Annotation
   #4 = Utf8               count
   #5 = Utf8               ()I
   #6 = Utf8               AnnotationDefault
   #7 = Integer            0
   #8 = Utf8               SourceFile
   #9 = Utf8               Counter.java
  #10 = Utf8               RuntimeVisibleAnnotations
  #11 = Utf8               Ljava/lang/annotation/Retention;
  #12 = Utf8               value
  #13 = Utf8               Ljava/lang/annotation/RetentionPolicy;
  #14 = Utf8               RUNTIME
  #15 = Utf8               Ljava/lang/annotation/Documented;
  #16 = Utf8               Ljava/lang/annotation/Target;
  #17 = Utf8               Ljava/lang/annotation/ElementType;
  #18 = Utf8               TYPE
  #19 = Utf8               club/throwable/annotation/Counter
  #20 = Utf8               java/lang/Object
  #21 = Utf8               java/lang/annotation/Annotation
{
  public abstract int count();
    descriptor: ()I
    flags: ACC_PUBLIC, ACC_ABSTRACT
    AnnotationDefault:
      default_value: I#7}
SourceFile: "Counter.java"
RuntimeVisibleAnnotations:
  0: #11(#12=e#13.#14)
  1: #15()
  2: #16(#12=[e#17.#18])
```

可以得到下面的信息：

* 注解是一个接口，它继承自`java.lang.annotation.Annotation`父接口。
* `@Counter`对应的接口接口除了继承了`java.lang.annotation.Annotation`中的抽象方法，自身定义了一个抽象方法`public abstract int count();`。

既然注解最后转化为一个接口，注解中定义的注解成员属性会转化为抽象方法，那么最后这些注解成员属性怎么进行赋值的呢?答案就是：**为注解对应的接口生成一个实现该接口的动态代理类。**直接点说就是：**Java通过动态代理的方式生成了一个实现了"注解对应接口"的实例，该代理类实例实现了"注解成员属性对应的方法"，这个步骤类似于"注解成员属性"的赋值过程，这样子就可以在程序运行的时候通过反射获取到注解的成员属性**\(这里注解必须是运行时可见的，也就是使用了`@Retention(RetentionPolicy.RUNTIME)`\)。

## 注解对应的动态代理类实例 <a id="&#x6CE8;&#x89E3;&#x5BF9;&#x5E94;&#x7684;&#x52A8;&#x6001;&#x4EE3;&#x7406;&#x7C7B;&#x5B9E;&#x4F8B;"></a>

注解的最底层实现就是一个JDK的动态代理类。

* `Class<?>#getAnnotation(Class<A> annotationClass)`，通过类型获取注解实例。
* `Class<?>#annotationData()`，获取注解的数据。
* `Class<?>#createAnnotationData(int classRedefinedCount)`，构建注解的数据。
* `AnnotationParser#parseAnnotations(byte[] var0, ConstantPool var1, Class<?> var2)`，这里已经是sun包下的类，无法看到源码，这个方法用于解析注解，**这一步使用到字节码中常量池的索引解析，常量解析完毕会生成一个成员属性键值对作为下一个环节的入参，常量池的解析可以看`AnnotationParser#parseMemberValue`方法**。
* `AnnotationParser#annotationForMap(final Class<? extends Annotation> var0, final Map<String, Object> var1)`，同样是`sun.reflect.annotation.AnnotationParser`中的方法，用于生成注解的动态代理类。

```java
public static Annotation annotationForMap(final Class<? extends Annotation> var0, final Map<String, Object> var1) {
    return (Annotation)AccessController.doPrivileged(new PrivilegedAction<Annotation>() {
        public Annotation run() {
            return (Annotation)Proxy.newProxyInstance(var0.getClassLoader(), new Class[]{var0}, new AnnotationInvocationHandler(var0, var1));
        }
    });
}
```

熟悉JDK动态代理的这里的代码应该看起来很简单，就是生成一个标准的JDK动态代理，而`InvocationHandler`的实例是`AnnotationInvocationHandler`，可以看它的成员变量、构造方法和实现`InvocationHandler`接口的`invoke`方法：

```java
class AnnotationInvocationHandler implements InvocationHandler, Serializable {

    private static final long serialVersionUID = 6182022883658399397L;
	  //保存了当前注解的类型
    private final Class<? extends Annotation> type;
	  //保存了注解的成员属性的名称和值的映射，注解成员属性的名称实际上就对应着接口中抽象方法的名称
    private final Map<String, Object> memberValues;
    private transient volatile Method[] memberMethods = null;

    AnnotationInvocationHandler(Class<? extends Annotation> var1, Map<String, Object> var2) {
        Class[] var3 = var1.getInterfaces();
        if (var1.isAnnotation() && var3.length == 1 && var3[0] == Annotation.class) {
            this.type = var1;
            this.memberValues = var2;
        } else {
            throw new AnnotationFormatError("Attempt to create proxy for a non-annotation type.");
        }
    }

    public Object invoke(Object var1, Method var2, Object[] var3) {
		//获取当前执行的方法名称
        String var4 = var2.getName();
        Class[] var5 = var2.getParameterTypes();
        if (var4.equals("equals") && var5.length == 1 && var5[0] == Object.class) {
            return this.equalsImpl(var3[0]);
        } else if (var5.length != 0) {
            throw new AssertionError("Too many parameters for an annotation method");
        } else {
            byte var7 = -1;
            switch(var4.hashCode()) {
            case -1776922004:
                if (var4.equals("toString")) {
                    var7 = 0;
                }
                break;
            case 147696667:
                if (var4.equals("hashCode")) {
                    var7 = 1;
                }
                break;
            case 1444986633:
                if (var4.equals("annotationType")) {
                    var7 = 2;
                }
            }
            switch(var7) {
            case 0:
                return this.toStringImpl();
            case 1:
                return this.hashCodeImpl();
            case 2:
                return this.type;
            default:
			      //利用方法名称从memberValues获取成员属性的赋值
                Object var6 = this.memberValues.get(var4);
                if (var6 == null) {
                    throw new IncompleteAnnotationException(this.type, var4);
                } else if (var6 instanceof ExceptionProxy) {
                    throw ((ExceptionProxy)var6).generateException();
                } else {
    					  //这一步就是注解成员属性返回值获取的实际逻辑
    					  //需要判断是否数据，如果是数据需要克隆一个数组
    					  //不是数组直接返回
                    if (var6.getClass().isArray() && Array.getLength(var6) != 0) {
                        var6 = this.cloneArray(var6);
                    }
                    return var6;
                }
            }
        }
    }
    
    //忽略其他方法
    	
}
```

这里需要重点注意一下的是：`AnnotationInvocationHandler`的成员变量`Map<String, Object> memberValues`存放着注解的成员属性的名称和值的映射，注解成员属性的名称实际上就对应着接口中抽象方法的名称，例如上面我们定义的`@Counter`注解生成代理类后，它的`AnnotationInvocationHandler`实例中的`memberValues`属性存放着键值对`count=1`。

既然知道了注解底层使用了JDK原生的Proxy，那么我们可以直接输出代理类到指定目录去分析代理类的源码，有两种方式可以输出Proxy类的源码：

* 1、通过Java系统属性设置`System.setProperty("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");`。
* 2、通过-D参数指定，其实跟1差不多，参数是：`-Dsun.misc.ProxyGenerator.saveGeneratedFiles=true`。

这里使用方式1，修改一下上面用到的main方法：

```java
public static void main(String[] args) throws Exception {
		System.setProperty("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");
		Counter counter = Main.class.getAnnotation(Counter.class);
		System.out.println(counter.count());
}
```

执行完毕之后，项目中多了一个目录

其中`$Proxy0`是`@Retention`注解对应的动态代理类，而`$Proxy1`才是我们的`@Counter`对应的动态代理类，当然如果有更多的注解，那么有可能生成`$ProxyN`。接着我们直接看`$Proxy1`的源码：

```java
public final class $Proxy1 extends Proxy implements Counter {
    private static Method m1;
    private static Method m2;
    private static Method m3;
    private static Method m4;
    private static Method m0;

    public $Proxy1(InvocationHandler var1) throws  {
        super(var1);
    }

    public final boolean equals(Object var1) throws  {
        try {
            return (Boolean)super.h.invoke(this, m1, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final String toString() throws  {
        try {
            return (String)super.h.invoke(this, m2, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final int count() throws  {
        try {
            return (Integer)super.h.invoke(this, m3, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final Class annotationType() throws  {
        try {
            return (Class)super.h.invoke(this, m4, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final int hashCode() throws  {
        try {
            return (Integer)super.h.invoke(this, m0, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
            m2 = Class.forName("java.lang.Object").getMethod("toString");
            m3 = Class.forName("club.throwable.annotation.Counter").getMethod("count");
            m4 = Class.forName("club.throwable.annotation.Counter").getMethod("annotationType");
            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}
```

显然，`$Proxy1`实现了`Counter`接口，它在代码的最后部分使用了静态代码块实例化了成员方法的`Method`实例，在前面的代码对这些`Method`进行了缓存，在调用成员方法的时候都是直接委托到`InvocationHandler(AnnotationInvocationHandler)`实例完成调用。我们在分析`AnnotationInvocationHandler`的时候看到，它只用到了`Method`的名称从Map从匹配出成员方法的结果，因此调用过程**并不是反射调用**，反而是直接的调用，效率类似于通过Key从Map实例中获取Value一样，是十分高效的。

## 小结 <a id="&#x5C0F;&#x7ED3;"></a>

既然知道了注解的底层原理，我们可以编写一个"注解接口"和`InvocationHandler`实现来简单模拟整个过程。先定义一个接口：

```java
public interface CounterAnnotation extends Annotation {

	  int count();
	
}
```

`InvocationHandler`的简单实现：

```java
public class CounterAnnotationInvocationHandler implements InvocationHandler {
    
    private final Map<String, Object> memberValues;
    private final Class<? extends Annotation> clazz;
    
    public CounterAnnotationInvocationHandler(Map<String, Object> memberValues, Class<? extends Annotation> clazz) {
        this.memberValues = memberValues;
        this.clazz = clazz;
    }
    
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        String methodName = method.getName();
        Object value;
        switch (methodName) {
            case "toString":
                value = super.toString();
                break;
            case "hashCode":
                value = super.hashCode();
                break;
            case "equals":
                value = super.equals(args[0]);
                break;
            case "annotationType":
                value = clazz;
                break;
            default:
                value = memberValues.get(methodName);
        }
        return value;
    }
    
}
```

编写一个main方法：

```java
public class CounterAnnotationMain {

    public static void main(String[] args) throws Exception{
        //这里模拟了注解成员属性从常量池解析的过程
        Map<String,Object> values = new HashMap<>(8);
        values.put("count", 1);
        //生成代理类
        CounterAnnotation proxy = (CounterAnnotation)Proxy.newProxyInstance(CounterAnnotationMain.class.getClassLoader(),
        new Class[]{CounterAnnotation.class},
        new CounterAnnotationInvocationHandler(values, CounterAnnotation.class));
        System.out.println(proxy.count());
    }
    
}
//运行后控制台输出:1
```

**参考资料：**

* \*\*\*\*[**JDK中注解的底层实现**](https://www.cnblogs.com/throwable/p/9747595.html)\*\*\*\*

