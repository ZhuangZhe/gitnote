# 线程状态转换以及基本操作

## 新建线程

Java程序天生就是一个多线程程序，自身包含的线程：

* 分发处理发送给JVM信号的线程。
* 调用对象的`finalize`方法的线程。
* 清除Reference的线程。
* `main`线程，用户程序的入口。

新建线程的三种方式：

* 通过继承`Thread`类，重写`run`方法。

  ```java
  Thread thread = new Thread() {
    @Override
    public void run() {
        System.out.println("继承Thread");
        super.run();
    }
  };
  thread.start();
  ```

* 通过实现`Runable`接口。

  ```java
  Thread thread = new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("实现runable接口");
    }
  });
  thread.start();
  ```

* 通过实现`Callable`接口。

  ```java
  ExecutorService service = Executors.newSingleThreadExecutor();
  Future<String> future = service.submit(new Callable() {
    @Override
    public String call() throws Exception {
        return "通过实现Callable接口";
    }
  });
  ```

由于Java不能多继承可以实现多个接口，因此在创建线程的时候尽量多考虑采用实现接口的形式。

* 实现`Callable`接口，提交给`ExecutorService`返回的是异步执行的结果，通常也可以利用`FutureTask(Callable callable)`将`callable`进行包装然后`FeatureTask`提交给`ExecutorsService`。
* `FeatureTask`也可以通过实现`Runable接口`来新建线程。
  * 可以通过`Executors`将`Runable`转换成`Callable`，具体方法是：
    * `Callable callable(Runnable task, T result)`
    * `Callable callable(Runnable task)`

## 线程状态的基本操作

### `interrupted`

* 中断可以理解为线程的一个`标志位`，它表示了一个运行中的线程是否被其他线程进行了中断操作。
* 其他线程可以调用该线程的`interrupt()`方法对其进行中断操作，同时该线程可以调用`isInterrupted()`来感知其他线程对其自身的中断操作。
* 调用`Thread`的静态方法`interrupted()`对当前线程进行中断操作，该方法会清除中断标志位。
* 当抛出`InterruptedException`时候，会清除中断标志位，也就是说在调用`isInterrupted()`会返回false。
* 中断操作可以看做线程间一种简便的交互方式。
* 在结束线程时通过中断标志位或者标志位的方式可以有机会去清理资源，相对于武断而直接的结束线程，这种方式要优雅和安全。

### `join`

* 可以看做是线程间协作的一种方式。
* `线程A`执行了`threadB.join()`，其含义是：当前`线程A`会等待`线程B`终止后才会继续执行，等待期间线程A一直处于阻塞。当`线程B`退出时会调用`notifyAll()`方法通知所有的等待线程。
* 提供如下方法：
  * `public final synchronized void join(long millis)`
  * `public final synchronized void join(long millis, int nanos)`
    * `线程B`在等待的时间内还没有结束的话，`线程A`会在超时之后继续执行。
  * `public final void join() throws InterruptedException`

### `sleep`

* `public static native void sleep(long millis)`方法是`Thread`的静态方法。
* 让当前线程按照指定的时间休眠，其休眠时间的精度取决于处理器的计时器和调度器。
* 需要注意的是如果当前线程获得了锁，`sleep`方法并`不会失去锁`。
  * `sleep()`与`wait()`的区别
    * `sleep()`方法是`Thread`的静态方法，而`wait()`是`Object`实例方法。
    * `wait()`方法必须要在同步方法或者同步块中调用，也就是必须已经获得对象锁。而`sleep()`方法没有这个限制可以在任何地方种使用。
    * `wait()`方法会释放占有的对象锁，使得该线程进入等待池中，等待下一次获取资源。而`sleep()`方法只是会让出CPU并不会释放掉对象锁；
    * `sleep()`方法在休眠时间达到后如果再次获得CPU时间片就会继续执行，而`wait()`方法必须等待`Object.notify()/Object.notifyAll()`通知后，才会离开等待池，并且再次获得CPU时间片才会继续执行。

### `yield`

* `public static native void yield()`是一个静态方法。
* 执行`yield()`使当前线程让出CPU，但让出的CPU并不是代表当前线程不再运行了，如果在下一次竞争中，如又获得了CPU时间片当前线程依然会继续运行。
* 让出的时间片只会分配给当前线程相同优先级的线程。
* 现代操作系统基本采用时分的形式调度运行的线程，操作系统会分出一个个时间片，线程会分配到若干时间片，当前时间片用完后就会发生线程调度，并等待这下次分配。线程分配到的时间多少也就决定了线程使用处理器资源的多少，而线程优先级就是决定线程需要或多或少分配一些处理器资源的线程属性。
* 通过一个整型成员变量`Priority`来控制优先级，优先级的范围从`1~10`在构建线程的时候可以通过`setPriority(int)`方法进行设置，默认优先级为`5`，优先级高的线程相较于优先级低的线程优先获得处理器时间片。
* `sleep()`交出来的时间片其他所有线程都可以去竞争，而`yield()`方法只允许与当前线程具有相同优先级的线程能够获得释放出来的CPU时间片。

## 守护线程Daemon

* 一种特殊的线程，在后台默默地守护一些系统服务，比如垃圾回收线程，JIT线程。
* 用户线程就可以认为是系统的工作线程，它会完成整个系统的业务操作。
* 用户线程完全结束后就意味着整个系统的业务任务全部结束了，因此系统就没有对象需要守护的了，守护线程自然而然就会退。
* 当一个Java应用，只有守护线程的时候，虚拟机就会自然退出。
* 守护线程在退出的时候并不会执行finaly块中的代码，所以将释放资源等操作不要放在finaly块中执行。

