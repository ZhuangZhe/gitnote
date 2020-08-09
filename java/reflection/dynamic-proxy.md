# 动态代理

Java动态代理机制的出现，使得Java开发人员不用手工编写代理类，只要简单地指定一组接口及委托类对象，便能动态地获得代理类。代理类会负责将所有的方法调用分派到委托对象上反射执行，在分派执行的过程中，开发人员还可以按需调整委托类对象及其功能，这是一套非常灵活有弹性的代理框架。Java动态代理实际上通过反射技术，把代理对象和被代理对象\(真实对象\)的代理关系建立延迟到程序运行之后，动态创建新的代理类去完成对真实对象的代理操作\(可以改变原来真实对象的方法行为\)，这一点成为了当前主流的AOP框架和延迟加载功能的基础。

## 设计模式中的代理模式

代理模式是一种常用的设计模式，其目的就是为其他对象提供一个代理以控制对某个对象的访问。代理类负责为委托类预处理消息，过滤消息并转发消息，以及进行消息被委托类执行后的后续处理。

![](../../.gitbook/assets/image%20%2864%29.png)

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



