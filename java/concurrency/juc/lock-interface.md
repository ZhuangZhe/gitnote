# Lock接口

在`Lock`接口出现之前，java程序主要是靠`synchronized`关键字实现锁功能的，而java SE5之后，并发包中增加了`Lock`接口，它提供了与`synchronized`一样的锁功能。

虽然它失去了像`synchronize`关键字隐式加锁解锁的便捷性，但是却拥有了锁获取和释放的可操作性，可中断的获取锁以及超时获取锁等多种`synchronized`关键字所不具备的同步特性。

```java
Lock lock = new ReentrantLock();
lock.lock();
try{
	// .......
}finally{
	lock.unlock();
}
```

需要注意的是`synchronized`同步块执行完成或者遇到异常是锁会自动释放，而`lock`必须调用`unlock()`方法释放锁，因此在`finally`块中释放锁。

## 源码

```java
package java.util.concurrent.locks;

import java.util.concurrent.TimeUnit;

public interface Lock {

    void lock();
    
    void lockInterruptibly() throws InterruptedException;
    
    boolean tryLock();
    
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    
    void unlock();
    
    Condition newCondition();

}
```



