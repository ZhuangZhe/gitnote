# 动态代理

Java动态代理机制的出现，使得Java开发人员不用手工编写代理类，只要简单地指定一组接口及委托类对象，便能动态地获得代理类。代理类会负责将所有的方法调用分派到委托对象上反射执行，在分派执行的过程中，开发人员还可以按需调整委托类对象及其功能，这是一套非常灵活有弹性的代理框架。Java动态代理实际上通过反射技术，把代理对象和被代理对象\(真实对象\)的代理关系建立延迟到程序运行之后，动态创建新的代理类去完成对真实对象的代理操作\(可以改变原来真实对象的方法行为\)，这一点成为了当前主流的AOP框架和延迟加载功能的基础。

![](../../../.gitbook/assets/image%20%2877%29.png)

## 设计模式中的代理模式

代理模式是一种常用的设计模式，其目的就是为其他对象提供一个代理以控制对某个对象的访问。代理类负责为委托类预处理消息，过滤消息并转发消息，以及进行消息被委托类执行后的后续处理。

![](../../../.gitbook/assets/image%20%2869%29.png)

代理模式主要包括三种角色：

* `Subject`抽象主题角色：一般定义为抽象类或者接口，是作为功能的定义，提供一系列抽象的功能方法。
* `RealSubject`具体\(真实\)主题角色：一般称为被委托角色或者被代理角色，它是`Subject`的一个具体实现。
* `ProxySubject`代理主题角色：一般称为委托角色或者代理角色，一般ProxySubject也实现\(或者继承\)Subject，接收一个具体的Subject实例RealSubject，在RealSubject处理前后做预定义或者后置操作，甚至可以直接忽略RealSubject原来的方法。

### 代码实现

```java
public interface Subject {

    void doSomething();
    
}

public class RealSubject implements Subject {

    @Override
    public void doSomething() {
        System.out.println("RealSubject doSomething...");
    }

}

public class ProxySubject implements Subject {

    private final Subject subject;
    
    public ProxySubject(Subject subject) {
        this.subject = subject;
    }
    
    @Override
    public void doSomething() {
        subject.doSomething();
        doOtherThing();
    }

    private void doOtherThing() {
        System.out.println("ProxySubject doOtherThing...");
    }
}

public class Client {

    public static void main(String[] args) throws Exception {
        Subject subject = new RealSubject();
        ProxySubject proxySubject = new ProxySubject(subject);
        proxySubject.doSomething();
    }
    
}
```

#### 输出

```bash
RealSubject doSomething...
ProxySubject doOtherThing...
```

代理模式在日常的场景中也经常碰到，比较常见的一个场景就是游戏代练，套进去上面的代码可以写个比较生动的例子：

```java
public interface Player {

    void playGame();

}

public class I implements Player {

    @Override
    public void playGame() {
        System.out.println("操作Throwable游戏角色打怪升级");
    }

}

public class ProxyPlayer implements Player {

    private final Player player;
    
    public ProxyPlayer(Player player) {
        this.player = player;
    }
    
    @Override
    public void playGame() {
        login();
        this.player.playGame();
        logout();
    }
    
    private void login() {
        System.out.println("登录Throwable游戏角色");
    }
    
    private void logout() {
        System.out.println("退出Throwable游戏角色");
    }
    
}
```

代理模式有几个比较大的优点：

* 职责清晰：也就是真实主题角色只需要实现具体的逻辑，不需关注代理类的职责，而代理类也只需要处理预处理和后置的逻辑，类的职责分明。
* 高扩展性：由于职责分明，也就是真实主题角色可以随时修改实现，这样就能通过更新或者替换真实主题的实现并且不改变代理主题角色的情况下改变具体功能。
* 高灵活性：主要体现在后面提到的动态代理。

## JDK动态代理的核心API

JDK动态代理提供外部使用的主要依赖两个类：

* `java.lang.reflect.Proxy`：可以理解为代理类的工厂类\(其实也是父类\)。

{% page-ref page="proxy-class.md" %}

* `java.lang.reflect.InvocationHandler`：代理实例需要实现的调用处理器接口。

{% page-ref page="invocationhandler-interface.md" %}

### Proxy

`java.lang.reflect.Proxy`是JDK动态代理的核心类，它的核心功能是提供静态方法来为一组接口动态地生成代理类并且返回代理实例对象，类似于代理类实例的工厂类。`java.lang.reflect.Proxy`主要提供四个public静态方法：

```java
// 方法 1: 该方法用于获取指定代理对象所关联的调用处理器
public static InvocationHandler getInvocationHandler(Object proxy) 
 
// 方法 2：该方法用于获取关联于指定类装载器和一组接口的动态代理类的类对象
public static Class<?> getProxyClass(ClassLoader loader, Class<?>[] interfaces) 
 
// 方法 3：该方法用于判断指定类对象是否是一个动态代理类
public static boolean isProxyClass(Class<?> cl) 
 
// 方法 4：该方法用于为指定类装载器、一组接口及调用处理器生成动态代理类实例
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)
```

* `InvocationHandler getInvocationHandler(Object proxy)`：通过制定的代理类实例查找它关联的调用处理器实例。
* `Class<?> getProxyClass(ClassLoader loader, Class<?>[] interfaces)`：用于获取关联于指定类装载器和一组接口的动态代理类的类对象，也就是获取`$ProxyXXX`的类型，此方法在JDK9以后标记为过期，原因是：在命名模块中生成的代理类是封闭的，模块外的代码无法访问这些类\(违反模块规则调用了会抛异常\)。
* `boolean isProxyClass(Class<?> cl)`：用于判断指定类是否是一个动态代理类。
* `Object newProxyInstance(ClassLoader loader, Class[] interfaces, InvocationHandler h)`：这个是JDK动态代理最核心的方法，用于为指定类装载器、一组接口及调用处理器生成动态代理类实例，也就是生成`$ProxyXXX`的实例。此方法需要指定类加载器`java.lang.ClassLoader`，`Proxy`静态方法生成动态代理类同样需要通过类装载器来进行装载才能使用，它与普通类的唯一区别就是其字节码是在运行时动态生成的而非预存在于任何一个`.class`文件中。`interfaces`是`Class`数组，也就是需要使用`InvocationHandler`进行代理访问的接口类型数组，这里的h参数就是调用处理器的实例。

### InvocationHandler

`java.lang.reflect.InvocationHandler`是调用处理器接口，它自定义了一个`invoke`方法，用于集中处理在动态代理类对象上的方法调用，通常在该方法中实现对委托类的代理访问。

```java
public interface InvocationHandler {
    Object invoke(Object proxy, Method method, Object[] args) throws Throwable;
}
```

参数说明：

* `proxy`：Object类型，此参数即是代理类实例，也就是`$ProxyXXX`的实例。
* `method`：`java.lang.reflect.Method`类型，被调用的方法的实例。
* `args`：Object\[\]类型，被调用方法的参数数组。

实现`java.lang.reflect.InvocationHandler`接口，通过实现`invoke`方法即可添加代理访问的逻辑，在这个逻辑代码块中除了可以调用委托类的方法，还可以织入额外的自定义逻辑，AOP就是这样实现的。

## JDK动态代理的流程

JDK动态代理的使用流程如下：

* 1、通过实现`java.lang.reflect.InvocationHandler`接口创建自定义的调用处理器。
* 2、通过为`java.lang.reflect.Proxy`类指定`ClassLoader`对象和一组interface来创建动态代理类。
* 3、通过反射机制获得动态代理类的构造函数，其唯一参数类型是调用处理器接口类型。
* 4、通过构造函数创建动态代理类实例，构造时调用处理器对象作为参数被传入。

伪代码如下：

```java
// InvocationHandlerImpl 实现了 InvocationHandler 接口，并能实现方法调用从代理类到委托类的分派转发
// 其内部通常包含指向委托类实例的引用，用于真正执行分派转发过来的方法调用
InvocationHandler handler = new InvocationHandlerImpl(..); 
 
// 通过Proxy为包括Interface接口在内的一组接口动态创建代理类的类对象
Class clazz = Proxy.getProxyClass(classLoader, new Class[] { Interface.class, ... }); 
 
// 通过反射从生成的类对象获得构造函数对象
Constructor constructor = clazz.getConstructor(new Class[] { InvocationHandler.class }); 
 
// 通过构造函数对象创建动态代理类实例
Interface Proxy = (Interface)constructor.newInstance(new Object[] { handler });
```

上面的过程比较复杂，可以进行精简。简化后的伪代码如下：

```java
// InvocationHandlerImpl实现了InvocationHandler 接口，并能实现方法调用从代理类到委托类的分派转发
InvocationHandler handler = new InvocationHandlerImpl(..); 
 
// 通过Proxy直接创建动态代理类实例
Interface proxy = (Interface) Proxy.newProxyInstance(classLoader, new Class[] { Interface.class }, handler);
```

## JDK动态代理的机制

首先是JDK动态代理生成的代理类本身的特点：

* **包\(或者JDK9引入的模块\)**：如果所代理的接口都是public的，那么它将被定义在包`com.sun.proxy`；如果所代理的接口中有非public的接口\(因为接口不能被定义为protect或private，所以除public之外就是默认的package访问级别，修饰符为default\)，那么它将被定义在该接口所在包\(假设代理了`throwable.club`包中的某非public接口A，那么新生成的代理类所在的包就是`throwable.club`\)，值得注意的是，**如果接口数组中存在非public的接口，那么它们必须在同一个包路径下，否则会抛异常**。这样设计的目的是为了最大程度的保证动态代理类不会因为包管理的问题而无法被成功定义并访问。
* **类修饰符**：该代理类具有final和public修饰符，**意味着它可以被所有的类访问，但是不能被再度继承**。
* **类名**：代理类名称格式是`$ProxyN`，其中N是一个逐一递增的阿拉伯数字，代表`java.lang.reflect.Proxy`类第N次生成的动态代理类，值得注意的一点是，并不是每次调用Proxy的静态方法创建动态代理类都会使得N值增加，原因是如果对同一组接口\(包括接口排列的顺序相同\)试图重复创建动态代理类，它会从缓存中获取先前已经创建好的代理类的类对象，而不会再尝试去创建一个全新的代理类，这样可以节省不必要的代码重复生成，提高了代理类的创建效率。
* **类继承关系**：代理类的继承关系图如下：

![](https://img2018.cnblogs.com/blog/1412331/202002/1412331-20200207105719279-1940030134.png)

由图可知，`java.lang.reflect.Proxy`类是代理类的父类，这个规则适用于所有由`java.lang.reflect.Proxy`创建的动态代理类。而且该类还实现了其所代理的一组接口，这就是为什么它能够被安全地类型转换到其所代理的某接口的根本原因。

### 代理类实例的特点

每个代理类实例都会关联一个调用处理器对象，可以通过`java.lang.reflect.Proxy`提供的静态方法`getInvocationHandler()`去获得代理类实例的调用处理器对象。在代理类实例上调用其代理的接口中所声明的方法时，这些方法最终都会由调用处理器的 invoke 方法执行，此外，值得注意的是，代理类的根类`java.lang.Object`中有三个方法也同样会被分派到调用处理器的`invoke`方法执行，它们是`hashCode`、`equals`和`toString`，可能的原因有：

* 一、因为这些方法为public且非final类型，能够被代理类覆盖。
* 二、因为这些方法往往呈现出一个类的某种特征属性，具有一定的区分度，所以为了保证代理类与委托类对外的一致性，这三个方法也应该被分派到委托类执行。当代理的一组接口有重复声明的方法且该方法被调用时，代理类总是从排在最前面的接口中获取方法对象并分派给调用处理器，而无论代理类实例是否正在以该接口\(或继承于该接口的某子接口\)的形式被外部引用，因为在代理类内部无法区分其当前的被引用类型。

### 被代理的一组接口的特点

首先，要注意不能有重复的接口，以避免动态代理类代码生成时的编译错误。其次，这些接口对于类装载器必须可见，否则类装载器将无法链接它们，将会导致类定义失败。再次，需被代理的所有非public的接口必须在同一个包中，否则代理类生成也会失败。最后，接口的数目不能超过65535，这是JVM设定的限制，这一点在代理类生成的时候也做了判断。

### 异常处理

从调用处理器接口声明的方法中可以看到理论上它能够抛出任何类型的异常，因为所有的异常都继承于Throwable接口，但事实是否如此呢？答案是否定的，原因是我们必须遵守一个继承原则：即子类覆盖父类或实现父接口的方法时，抛出的异常必须在原方法支持的异常列表之内。所以虽然调用处理器理论上讲能够，但实际上往往受限制，除非父接口中的方法支持抛Throwable异常。那么如果在invoke方法中的确产生了接口方法声明中不支持的异常，那将如何呢？放心，Jdk动态代理类已经为我们设计好了解决方法：它将会抛出`UndeclaredThrowableException` 异常。这个异常是一个`RuntimeException`类型，所以不会引起编译错误。通过该异常的`getCause`方法，还可以获得原来那个不受支持的异常对象，以便于错误诊断。

## JDK动态代理源码分析

因为JDK动态代理核心逻辑都在`java.lang.reflect.Proxy`类中，下面简单分析一下这个类的源码。先看`Proxy`类中的几个重要的静态变量：

```java
// 接口组中接口都为为public时候代理类创建的包路径：com.sun.proxy
private static final String PROXY_PACKAGE_PREFIX = ReflectUtil.PROXY_PACKAGE;

// 代理类的构造方法参数类型数组，可见代理类的构造参数只有InvocationHandler类型
private static final Class<?>[] constructorParams = { InvocationHandler.class };

// 缓存了所有已经调用过setAccessible(true)的代理类的构造(Constructor)实例
private static final ClassLoaderValue<Constructor<?>> proxyCache = new ClassLoaderValue<>();
```

这里注意到`ClassLoaderValue`，下文会调用到它的一个很复杂的调用链：

```java
//intf是Class<?>类型
//loader是类加载器实例
return proxyCache.sub(intf).computeIfAbsent(
    loader,
    (ld, clv) -> new ProxyBuilder(ld, clv.key()).build()
);

public V computeIfAbsent(ClassLoader cl,
                         BiFunction<? super ClassLoader,? super CLV,? extends V> mappingFunction) 
						throws IllegalStateException {
```

上面的`computeIfAbsent`中使用了函数式接口和Lambda表达式，如果Lambda表达式玩的比较熟练看起来应该没问题，它的功能可以解读为：通过接口类型和类加载器实例计算通过接口类型和类加载器实例构建`ProxyBuilder`实例并且调用`ProxyBuilder#build()`得到的结果，如果结果已经存在则直接返回缓存。其实`computeIfAbsent`在`Map`接口中也定义了同样的方法，功能是相似的。

接着看`Proxy`的构造函数：

```java
protected InvocationHandler h;

private Proxy() {
}

protected Proxy(InvocationHandler h) {
    Objects.requireNonNull(h);
    this.h = h;
}
```

到此可以明确一点，既然所有动态代理类都是`java.lang.reflect.Proxy`的子类，那么它们一定具备一个包含`InvocationHandler`参数的构造器。接着查看方法的源码：

```java
public static Object newProxyInstance(ClassLoader loader,
                                      Class<?>[] interfaces,
                                      InvocationHandler h) {
    // 空判断									  
    Objects.requireNonNull(h);
    // 当前调用类获取
    final Class<?> caller = System.getSecurityManager() == null
                                    ? null
                                    : Reflection.getCallerClass();
    // 获取代理类的构造器实例								
    Constructor<?> cons = getProxyConstructor(caller, loader, interfaces);
    // 生成代理实例
    return newProxyInstance(caller, cons, h);
}
```

先看`getProxyConstructor`方法：

```java
private static Constructor<?> getProxyConstructor(Class<?> caller,
                                                  ClassLoader loader,
                                                  Class<?>... interfaces){
    // 这里需要区分代理接口数组中只有单个接口和多个接口的逻辑				  
    // 而基本的逻辑都是先校验当前调用类的权限，后续获取Constructor实例委托到ProxyBuilder
    if (interfaces.length == 1) {
        Class<?> intf = interfaces[0];
        if (caller != null) {
            checkProxyAccess(caller, loader, intf);
        }
        return proxyCache.sub(intf).computeIfAbsent(
            loader,
            (ld, clv) -> new ProxyBuilder(ld, clv.key()).build()
        );
    } else {
        // 接口克隆
        final Class<?>[] intfsArray = interfaces.clone();
        if (caller != null) {
            checkProxyAccess(caller, loader, intfsArray);
        }
        final List<Class<?>> intfs = Arrays.asList(intfsArray);
        return proxyCache.sub(intfs).computeIfAbsent(
            loader,
            (ld, clv) -> new ProxyBuilder(ld, clv.key()).build()
        );
    }
}
```

可以明确，核心的逻辑都交给了`Proxy`的内部类`ProxyBuilder`完成，先看`ProxyBuilder`的静态成员变量：

```java
// Unsafe实例
private static final Unsafe UNSAFE = Unsafe.getUnsafe();

// 代理类的简单类名的前置字符串
private static final String proxyClassNamePrefix = "$Proxy";

// 用于生成下一个代理类的数字计数器，记住它是静态的
private static final AtomicLong nextUniqueNumber = new AtomicLong();

// 记录了已经生成的代理类-Boolean的映射，已经生成过对应代理类则记录为true
private static final ClassLoaderValue<Boolean> reverseProxyCache = new ClassLoaderValue<>();
```

```java
// 单个代理接口的情况，其实也是把接口转换为List
ProxyBuilder(ClassLoader loader, Class<?> intf) {
    this(loader, Collections.singletonList(intf));
}
// 多个代理接口的情况
ProxyBuilder(ClassLoader loader, List<Class<?>> interfaces) {
    // 通过JVM参数强制关闭动态代理功能则抛出异常
    if (!VM.isModuleSystemInited()) {
        throw new InternalError("Proxy is not supported until "
                + "module system is fully initialized");
    }
    // 代理接口数量不能超过65535，也就是最多代理65535个接口
    if (interfaces.size() > 65535) {
        throw new IllegalArgumentException("interface limit exceeded: "
                + interfaces.size());
    }
    // 收集接口数组中所有接口的非静态方法的返回值类型、共享(shared)参数类型和共享(shared)异常类型，注释说是收集代理接口的方法签名
    Set<Class<?>> refTypes = referencedTypes(loader, interfaces);

    // 确保上一步得到的代理接口方法签名的类型都是"可见(其实就是类型都存在)"的，通过遍历调用Class.forName(type.getName(), false, ld)去判断
    validateProxyInterfaces(loader, interfaces, refTypes);

    this.interfaces = interfaces;
    // 获取代理类最终生成的模块，规则如下：
    // 1、所有代理接口的修饰符都为public，接口所在模块都能公开访问，则返回unnamed模块
    // 2、如果有任意的代理接口是包私有，则返回该包所在的模块		、
    // 3、所有代理接口的修饰符都为public，有任意至少一个接口所在模块不能公开访问，则返回该不能公开访问的模块，
    this.module = mapToModule(loader, interfaces, refTypes);
    assert getLoader(module) == loader;
}
```

一个构造器处理的逻辑也是相对复杂，主要是因为引入模块管理的概念，接着看`ProxyBuilder#build()`的源码：

```java
Constructor<?> build() {
    // 定义代理类，实际上是动态生成代理类字节码和缓存它的类型的过程
    Class<?> proxyClass = defineProxyClass(module, interfaces);
    final Constructor<?> cons;
    
    try {
    // 返回代理类的构造
        cons = proxyClass.getConstructor(constructorParams);
    } catch (NoSuchMethodException e) {
        throw new InternalError(e.toString(), e);
    }
    
    AccessController.doPrivileged(new PrivilegedAction<Void>() {
        public Void run() {
            cons.setAccessible(true);
            return null;
        }
    });
    return cons;
}
```

最后到逻辑最复杂的代理类的生成过程`ProxyBuilder#defineProxyClass()`：

```java
private static Class<?> defineProxyClass(Module m, List<Class<?>> interfaces) {
    String proxyPkg = null;     // package to define proxy class in
    int accessFlags = Modifier.PUBLIC | Modifier.FINAL;
    // 这里就是定义代理类包路径的逻辑，规则如下：
    // 1、代理接口数组所有接口都是public修饰，则代理类包路径为com.sun.proxy
    // 2、代理接口数组有任意接口是包私有的，则代理类包路径为该私有包的路径			
    for (Class<?> intf : interfaces) {
        int flags = intf.getModifiers();
        if (!Modifier.isPublic(flags)) {
            accessFlags = Modifier.FINAL;  // non-public, final
            String pkg = intf.getPackageName();
            if (proxyPkg == null) {
                proxyPkg = pkg;
            } else if (!pkg.equals(proxyPkg)) {
                throw new IllegalArgumentException(
                        "non-public interfaces from different packages");
            }
        }
    }
    // 下面几个if都是包路径的合法性判断
    if (proxyPkg == null) {
        // all proxy interfaces are public
        proxyPkg = m.isNamed() ? PROXY_PACKAGE_PREFIX + "." + m.getName()
                               : PROXY_PACKAGE_PREFIX;
    } else if (proxyPkg.isEmpty() && m.isNamed()) {
        throw new IllegalArgumentException(
                "Unnamed package cannot be added to " + m);
    }
    if (m.isNamed()) {
        if (!m.getDescriptor().packages().contains(proxyPkg)) {
            throw new InternalError(proxyPkg + " not exist in " + m.getName());
        }
    }
    // 计数器加1返回新的计数值			
    long num = nextUniqueNumber.getAndIncrement();
    // 生成代理类全类名，一个常见的格式是：com.sun.proxy.$Proxy1
    String proxyName = proxyPkg.isEmpty()
                            ? proxyClassNamePrefix + num
                            : proxyPkg + "." + proxyClassNamePrefix + num;
    ClassLoader loader = getLoader(m);
    trace(proxyName, m, loader, interfaces);
    // 动态生成代理类字节码字节数组			
    byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
            proxyName, interfaces.toArray(EMPTY_CLASS_ARRAY), accessFlags);
    try {
        // 通过Unsafe定义代理类-这里是通过字节码定义新的类				
        Class<?> pc = UNSAFE.defineClass(proxyName, 
                                         proxyClassFile,
                                         0, 
                                         proxyClassFile.length,
                                         loader, 
                                         null);
        // 缓存代理类已经生成过的标记												 
        reverseProxyCache.sub(pc).putIfAbsent(loader, Boolean.TRUE);
        return pc;
    } catch (ClassFormatError e) {
        throw new IllegalArgumentException(e.toString());
    }
}
```

到这一步为止，代理类的生成过程已经大致分析完毕，`ProxyGenerator`中涉及到大量字节码操作，这里就不深入分析了。那么回到最前面的方法，得到代理类和它的构造实例，接着就可以生成代理实例：

```java
private static Object newProxyInstance(Class<?> caller, // null if no SecurityManager
                                       Constructor<?> cons,
                                       InvocationHandler h) {
    try {
        if (caller != null) {
            checkNewProxyPermission(caller, cons.getDeclaringClass());
        }
        // 这里简单反射调用Constructor#newInstance(h)
        return cons.newInstance(new Object[]{h});
    } catch (IllegalAccessException | InstantiationException e) {
        throw new InternalError(e.toString(), e);
    } catch (InvocationTargetException e) {
        Throwable t = e.getCause();
        if (t instanceof RuntimeException) {
            throw (RuntimeException) t;
        } else {
            throw new InternalError(t.toString(), t);
        }
    }
}
```

小结一下：

* **接口数组中所有接口元素的类修饰符最好一致为public。如果接口数组中存在非default修饰的接口元素，那么接口数组中的所有接口类都要放在同一个包下，并且都要使用default修饰。**
* 很少情况下我们修改接口的修饰符，默认为public，那么所有代理类的包路径都是`com.sun.proxy`，全类名是:`com.sun.proxy.$ProxyN`。
* 代理接口数量不能超过65535。

## JDK动态代理类的源代码



前面已经分析完了代理类的生成过程，这里举个简单的使用例子，并且观察生成的动态代理类的源代码。

### **例子**

```java
// 接口
public interface Simple {

    void sayHello(String name);
}
// 接口实现
public class DefaultSimple implements Simple {

    @Override
    public void sayHello(String name) {
        System.out.println(String.format("%s say hello!", name));
    }
}
// 场景类
public class Main {

    public static void main(String[] args) throws Exception {
        Simple simple = new DefaultSimple();
        Object target = Proxy.newProxyInstance(Main.class.getClassLoader(), new Class[]{Simple.class}, new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println("Before say hello...");
                method.invoke(simple, args);
                System.out.println("After say hello...");
                return null;
            }
        });
        Simple proxy = (Simple) target;
        proxy.sayHello("throwable");
    }
}
```

调用后输出：

```text
Before say hello...
throwable say hello!
After say hello...
```

可以看到，我们在被代理类`DefaultSimple`实例的方法调用前后织入了自定义的逻辑，这就是通过JDK动态代理实现AOP的底层原理。在JDK8中可以直接使用`sun.misc.ProxyGenerator`去输出代理类的class文件，但是JDK11中这个代理类生成器已经变成`java.lang.reflect.ProxyGenerator`，并且这个类是包私有的，我们无法使用，但是它提供了`jdk.proxy.ProxyGenerator.saveGeneratedFiles`这个VM参数让我们可以保存代理类的class文件：

```bash
# JVM参数
-Djdk.proxy.ProxyGenerator.saveGeneratedFiles=true
```

配置好VM参数后，再次调用mian方法就能看到在项目的顶层包路径下看到对应的类`com.sun.proxy.$Proxy0`，目前从`java.lang.reflect.ProxyGenerator`源码看无法控制代理类文件的输出路径，生成的代理类内容如下：

```java
public final class $Proxy0 extends Proxy implements Simple {
    private static Method m1;
    private static Method m3;
    private static Method m2;
    private static Method m0;

    public $Proxy0(InvocationHandler var1) throws  {
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

    public final void sayHello(String var1) throws  {
        try {
            super.h.invoke(this, m3, new Object[]{var1});
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
            m3 = Class.forName("club.throwable.jdk.sample.reflection.proxy.Simple").getMethod("sayHello", Class.forName("java.lang.String"));
            m2 = Class.forName("java.lang.Object").getMethod("toString");
            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}
```

代理类的代码比较简单，有如下几个特点：

* 1、代理类继承于`java.lang.reflect.Proxy`，实现了接口数组中的接口元素类，构造函数只有一个`InvocationHandler`类型的参数。
* 2、接口中的所有被代理方法包括`equals`、`toString`、`hashCode`都建立了一个对应的Method私有静态实例，在最后面的静态代码块中实例化。
* 3、所有代理方法都是用public final修饰，也就是代理类中的代理方法是不能覆盖的。
* 4、所有代理方法都是通过`InvocationHandler`实例的`invoke`方法进行调用的，记得第一个参数是代理类实例本身，如果用了在`InvocationHandler#invoke()`方法实现过程中使用了这个参数有可能造成死循环。

### 小结 <a id="&#x5C0F;&#x7ED3;"></a>

诚然，Proxy已经设计得非常优美，但是还是有一点点小小的遗憾之处，那就是它始终无法摆脱仅支持interface代理的桎梏，因为它的设计注定了这个遗憾。回想一下那些动态生成的代理类的继承关系图，它们已经注定有一个共同的父类叫Proxy。Java的**单继承机制**注定了这些动态代理类们无法实现对class的动态代理\(所以只能代理接口，实际上是基于反射对方法级别的逻辑进行编织\)。有很多条理由，可以否定对class代理的必要性，但是同样有一些理由，相信支持class动态代理会更美好。但是，不完美并不等于不伟大，伟大是一种本质，JDK动态代理就是佐例。



**参考资料：**

* \*\*\*\*[**Java动态代理机制分析及扩展-第1部分**](https://www.ibm.com/developerworks/cn/java/j-lo-proxy1/)\*\*\*\*
* \*\*\*\*[**Java动态代理机制分析及扩展-第2部分**](https://www.ibm.com/developerworks/cn/java/j-lo-proxy2/)\*\*\*\*
* **JDK11相关源码**
* \*\*\*\*[**深入分析Java反射\(四\)-动态代理**](https://www.cnblogs.com/throwable/p/12272262.html)\*\*\*\*

