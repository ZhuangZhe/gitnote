# 原子性、可见性以及有序性

## 原子性

指一个操作是不可中断的，要么全部执行成功要么全部执行失败。

```java
    int a = 10; //1
    a++; //2
    int b = a; //3
    a = a+1; //4
```

* 只有第1个语句是原子操作
* `a++`实际上包含了三个操作，无法构成原子操作：
  * 读取变量a的值
  * 对a进行加一的操作
  * 将计算后的值再赋值给变量

Java内存模型中定义了8中操作都是原子的，不可再细分

* `lock(锁定)`：作用于主内存中的变量，它把一个变量标识为一个线程独占的状态；
* `unlock(解锁)`：作用于主内存中的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定
* `read(读取)`：作用于主内存的变量，它把一个变量的值从主内存传输到线程的工作内存中，以便后面的`load`动作使用；
* `load(载入)`：作用于工作内存中的变量，它把`read`操作从主内存中得到的变量值放入工作内存中的变量副本
* `use(使用)`：作用于工作内存中的变量，它把工作内存中一个变量的值传递给执行引擎，每当虚拟机遇到一个需要使用到变量的值的字节码指令时将会执行这个操作；
* `assign(赋值)`：作用于工作内存中的变量，它把一个从执行引擎接收到的值赋给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作；
* `store(存储)`：作用于工作内存的变量，它把工作内存中一个变量的值传送给主内存中以便随后的write操作使用；
* `write(操作)`：作用于主内存的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量中。
  * 由原子性变量操作`read`、`load`、`use`、`assign`、`store`、`write`，可以大致认为基本数据类型的访问读写具备原子性\(例外就是long和double的非原子性协定\)。
  * JVM没有把`lock`和`unlock`开放给我们使用，但`JVM`以更高层次的指令`monitorenter`和`monitorexit`指令开放给我们使用，也就是说`synchronized`满足原子性。
  * `volatile`并不能保证原子性

所以`i++`线程不安全，因为，如果线程A读取`i`到工作内存后，其他线程对这个值已经做了自增操作后，那么线程A的这个值自然而然就是一个过期的值。

让volatile保证原子性，必须符合以下两条规则：

* 运算结果并不依赖于变量的当前值，或者能够确保只有一个线程修改变量的值。
* 变量不需要与其他的状态变量共同参与不变约束。

## 可见性

`synchronized`语义表示锁在同一时刻只能由一个线程进行获取，当锁被占用后，其他线程只能等待。因此，`synchronized`语义就要求线程在访问读写共享变量时只能“串行”执行，因此`synchronized`具有有序性。

* Java程序天然的有序性可以总结为：
  * 如果在本线程内观察，所有的操作都是有序的。
  * 如果在一个线程观察另一个线程，所有的操作都是无序的。

## 有序性

可见性是指当一个线程修改了共享变量后，其他线程能够立即得知这个修改。

* 通过之前对`synchronzed`内存语义进行了分析，当线程获取锁时会从主内存中获取共享变量的最新值，释放锁的时候会将共享变量同步到主内存中，从而`synchronized`具有可见性。
* 同样的在`volatile`分析中，会通过在指令中添加`lock`指令，以实现内存可见性，因此, `volatile`具有可见性

