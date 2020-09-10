# IoC

## 概述

**Inverse of Contro，即控制反转，**不是什么技术，而是一种**设计思想**，就是**将原本在程序中手动创建对象的控制权，交由Spring框架来管理。**

* **正控：**若要使用某个对象，需要**自己去负责对象的创建**
* **反控：**若要使用某个对象，只需要**从 Spring 容器中获取需要使用的对象，不关心对象的创建过程**，也就是把**创建对象的控制权反转给了Spring框架**

这就是一种控制反转的理念，上述的例子已经很好的说明了问题，我们再来描述一下控制反转的概念：**控制反转是一种通过描述\(在Java中可以是XML或者注解\)并通过第三方\(Spring\)去产生或获取特定对象的方式。**

* **好处：**
  * 降低对象之间的耦合。
  * 我们不需要理解一个类的具体实现，只需要知道它有什么用就好了\(直接向IoC容器拿\)。
* 缺点：
  * 生成一个对象的步骤变复杂了（其实上操作上还是挺简单的），对于不习惯这种方式的人，会觉得有些别扭和不直观。
  * 对象生成因为是使用反射编程，在效率上有些损耗。但相对于IoC提高的维护性和灵活性来说，这点损耗是微不足道的，除非某对象的生成对效率要求特别高。
  * 缺少IDE重构操作的支持，如果在Eclipse要对类改名，那么你还需要去XML文件里手工去改了，这似乎是所有XML方式的缺憾所在。

主动创建的模式中，责任归于开发者，而在被动的模式下，责任归于IoC容器，**基于这样的被动形式，我们就说对象被控制反转了。**

## Spring IoC 容器

Spring会提供**IoC容器**来管理和容纳我们所开发的各种各样的Bean，并且我们可以从中获取各种发布在Spring IoC容器里的Bean，并且**通过描述**可以得到它。

### **Spring IoC 容器的设计**

Spring IoC 容器的设计主要是基于以下两个接口：

* **BeanFactory**
* **ApplicationContext**

其中`ApplicationContext`是`BeanFactory`的子接口之一，换句话说：**`BeanFactory`是Spring IoC容器所定义的最底层接口，**而`ApplicationContext`是其最高级接口之一，并对`BeanFactory`功能做了许多的扩展，所以在**绝大部分的工作场景下**，都会使用`ApplicationContext`作为Spring IoC容器。

![](../../../.gitbook/assets/image%20%2819%29.png)

#### **BeanFactory**

从上图中我们可以几乎看到， `BeanFactory`位于设计的最底层，它提供了Spring IoC最底层的设计，为此，我们先来看看该类中提供了哪些方法。

![](../../../.gitbook/assets/image%20%2853%29.png)

* `getBean`对应了多个方法来获取配置给Spring IoC容器的Bean。
  * 按照类型获取bean
    * `bean = (Bean) factory.getBean(Bean.class);`
    * 要求在Spring中只配置了一个这种类型的实例，否则报错。
  * 按照bean的名字获取bean
    * `bean = (Bean) factory.getBean("beanName");`
    * 这种方法不太安全，IDE不会检查其安全性\(关联性\)。
  * 按照名字和类型拿bean
    * `bean = (Bean) factory.getBean("beanName", Bean.class);`
* `isSingleton`用于判断是否单例，如果判断为真，其意思是该Bean在容器中是作为一个唯一单例存在的。
* `isPrototype`如果判断为真，意思是当你从容器中获取Bean，容器就为你生成一个新的实例。
* 在默认情况下，`isSingleton`为ture，而`isPrototype`为false
* `getAliases`获取别名的方法

#### **ApplicationContext**

根据`ApplicationContext`的类继承关系图，可以看到`ApplicationContext`接口扩展了许许多多的接口，因此它的功能十分强大，所以在实际应用中常常会使用到的是`ApplicationContext`接口，因为`BeanFactory`的方法和功能较少，而`ApplicationContext`的方法和功能较多。

#### **ApplicationContext常见实现类**

* `ClassPathXmlApplicationContext`读取classpath中的资源
  * `ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");`
* `FileSystemXmlApplicationContext`读取指定路径的资源
  * `ApplicationContext ac = new FileSystemXmlApplicationContext("c:/applicationContext.xml");`
* `XmlWebApplicationContext`需要在Web的环境下才可以运行
* ```java
  XmlWebApplicationContext ac = new XmlWebApplicationContext(); // 这时并没有初始化容器
  ac.setServletContext(servletContext); // 需要指定ServletContext对象
  ac.setConfigLocation("/WEB-INF/applicationContext.xml"); // 指定配置文件路径，开头的斜线表示Web应用的根目录
  ac.refresh(); // 初始化容器
  ```

#### **BeanFactory和ApplicationContext 的区别：**

* **BeanFactory：**是Spring中最底层的接口，只提供了最简单的IoC功能，负责配置、创建和管理bean。 在应用中，一般不使用 BeanFactory，而推荐使用ApplicationContext（应用上下文），原因如下。
* **ApplicationContext：** 1.继承了 BeanFactory，拥有了基本的 IoC 功能； 2.除此之外，ApplicationContext 还提供了以下功能：
  * 支持国际化
  * 支持消息机制
  * 支持统一的资源加载
  * 支持AOP功能

**Bean 的定义分为3步：**

* `Resource`定位：Spring IoC容器先根据开发者的配置，进行资源的定位，在Spring的开发中，通过XML或者注解都是十分常见的方式，定位的内容是由开发者提供的。
* `BeanDefinition`的载入：这个时候只是将`Resource`定位到的信息，保存到 Bean 定义\(`BeanDefinition`\)中，此时并不会创建Bean的实例
* `BeanDefinition`的注册：这个过程就是将`BeanDefinition`的信息发布到Spring IoC容器中

