# 反射

反射\(Reflection\)是Java实现运行时检查和动态调用类、构造、方法、属性等等的编程语言的方式，甚至可以不需要在编译期感知类的名称、方法的名称等等。[Oracle关于Java反射的官方教程](https://docs.oracle.com/javase/tutorial/reflect/index.html)中指出反射是由应用程序使用，用于检查或修改在Java虚拟机中运行的应用程序的运行时行为，这是一个相对高级的功能，需要由掌握Java语言基础知识的开发者使用。

## 优点

* 可以检查或修改应用程序的运行时行为
* 抑制修饰符限制直接访问私有属性
* 等等

## 缺点

* 性能开销：由于反射涉及动态解析的类型，因此无法执行某些Java虚拟机优化。因此，反射操作的性能低于非反射操作，应避免在性能敏感应用程序中频繁调用反射操作代码片段。
* 安全限制：反射需要运行时权限，不能在安全管理器\(security manager\)下进行反射操作。
* 代码可移植性：反射代码打破了抽象，反射的类库有可能随着平台\(JDK\)升级发生改变，反射代码中允许执行非反射代码的逻辑例如允许访问私有字段，这些问题都有可能影响到代码的可移植性。

## 图解反射核心类的体系

JDK中对和反射相关的类库集中在`java.lang.reflect`包和`java.lang`包中。

`java.lang.reflect`包反射核心类有核心类`Class`、`Constructor`、`Method`、`Field`、`Parameter`，它们的基础体系如下：

* `java.lang.Class`类继承体系:

![](../../.gitbook/assets/image%20%2862%29.png)

* `java.lang.reflect.Constructor`类继承体系:

![](../../.gitbook/assets/image%20%2859%29.png)

* `java.lang.reflect.Method`类继承体系:

![](../../.gitbook/assets/image%20%2861%29.png)

* `java.lang.reflect.Field`类继承体系:

![](../../.gitbook/assets/image%20%2863%29.png)

* `ava.lang.reflect.Parameter`类继承体系:

![](../../.gitbook/assets/image%20%2864%29.png)







**参考资料：**

* \*\*\*\*[**深入分析Java反射\(一\)-核心类库和方法**](https://www.cnblogs.com/throwable/p/12272229.html)\*\*\*\*

