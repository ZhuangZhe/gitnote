# 动态代理

Java动态代理机制的出现，使得Java开发人员不用手工编写代理类，只要简单地指定一组接口及委托类对象，便能动态地获得代理类。代理类会负责将所有的方法调用分派到委托对象上反射执行，在分派执行的过程中，开发人员还可以按需调整委托类对象及其功能，这是一套非常灵活有弹性的代理框架。Java动态代理实际上通过反射技术，把代理对象和被代理对象\(真实对象\)的代理关系建立延迟到程序运行之后，动态创建新的代理类去完成对真实对象的代理操作\(可以改变原来真实对象的方法行为\)，这一点成为了当前主流的AOP框架和延迟加载功能的基础。

## 设计模式中的代理模式

代理模式是一种常用的设计模式，其目的就是为其他对象提供一个代理以控制对某个对象的访问。代理类负责为委托类预处理消息，过滤消息并转发消息，以及进行消息被委托类执行后的后续处理。

![](../../../.gitbook/assets/image%20%2864%29.png)

代理模式主要包括三种角色：

* `Subject`抽象主题角色：一般定义为抽象类或者接口，是作为功能的定义，提供一系列抽象的功能方法。
* `RealSubject`具体\(真实\)主题角色：一般称为被委托角色或者被代理角色，它是Subject的一个具体实现。
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

{% page-ref page="invocationhandler-class.md" %}

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
* `Object newProxyInstance(ClassLoader loader, Class[] interfaces, InvocationHandler h)`：这个是JDK动态代理最核心的方法，用于为指定类装载器、一组接口及调用处理器生成动态代理类实例，也就是生成`$ProxyXXX`的实例。此方法需要指定类加载器`java.lang.ClassLoader`，Proxy静态方法生成动态代理类同样需要通过类装载器来进行装载才能使用，它与普通类的唯一区别就是其字节码是在运行时动态生成的而非预存在于任何一个.class文件中。interfaces是Class数组，也就是需要使用InvocationHandler进行代理访问的接口类型数组，这里的h参数就是调用处理器的实例。











