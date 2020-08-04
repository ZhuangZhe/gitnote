# ReentrantLock类

`ReentrantLock`实现了`Lock`接口。`ReentrantLock`基本上所有的方法的实现实际上都是调用了其静态内存类`Sync`中的方法，而`Sync`类继承了`AbstractQueuedSynchronizer(AQS)`。可以看出要想理解`ReentrantLock`关键核心在于对队列同步器`AbstractQueuedSynchronizer`。

**支持重入性，表示能够对共享资源能够重复加锁，即当前线程获取该锁再次获取不会被阻塞**。

关键字synchronized隐式支持重入性，synchronized通过获取自增，释放自减的方式实现重入。

`ReentrantLock`还支持**公平锁和非公平锁**两种方式。

**何谓公平性，是针对获取锁而言的，如果一个锁是公平的，那么锁的获取顺序就应该符合请求上的绝对时间顺序，满足FIFO**。

`ReentrantLock`的构造方法无参时是构造非公平锁。另外还提供了另外一种方式，可传入一个boolean值，true时为公平锁，false时为非公平锁。

以非公平锁为例，判断当前线程能否获得锁为例，核心方法为`nonfairTryAcquire`。

* 获取锁时，如果该锁已经被线程所占有了，会继续检查占有线程是否为当前线程，如果是的话，同步状态加1返回true，表示可以再次获取成功。
* 释放锁时，重入锁的释放必须得等到同步状态为0时锁才算成功释放，否则锁仍未释放。如果锁被获取n次，释放了n-1次，该锁未完全释放返回false，只有被释放n次才算成功释放，返回true。

公平锁的核心方法为`tryAcquire`。

* 逻辑与`nonfairTryAcquire`基本上一直，唯一的不同在于增加了`hasQueuedPredecessors`的逻辑判断，方法名就可知道该方法用来判断当前节点在同步队列中是否有前驱节点的判断。
  * 如果有前驱节点说明有线程比当前线程更早的请求资源，根据公平性，当前线程请求资源失败。
  * 如果当前节点没有前驱节点的话，再才有做后面的逻辑判断的必要性。

**公平锁每次都是从同步队列中的第一个节点获取到锁，而非公平性锁则不一定，有可能刚释放锁的线程能再次获取到锁**。

我们可以多次调用`lock.newCondition()`方法创建多个`condition`对象，也就是一个lock可以持有多个等待队列。而在之前利用Object的方式实际上是指在**对象Object对象监视器上只能拥有一个同步队列和一个等待队列，而并发包中的Lock拥有一个同步队列和多个等待队列**。

**当调用condition.await\(\)方法后会使得当前获取lock的线程进入到等待队列，如果该线程能够从await\(\)方法返回的话一定是该线程获取了与condition相关联的lock**。

## 源码

```java
package java.util.concurrent.locks;

import java.util.Collection;
import java.util.concurrent.TimeUnit;
import jdk.internal.vm.annotation.ReservedStackAccess;

public class ReentrantLock implements Lock, java.io.Serializable {

    private static final long serialVersionUID = 7373984872572414699L;
    
    private final Sync sync;
    
    abstract static class Sync extends AbstractQueuedSynchronizer {
    
        private static final long serialVersionUID = -5179523762034025860L;
        
        @ReservedStackAccess
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) { // 如果该锁未被任何线程占有，该锁能被当前线程获取
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            } // 若被占有，检查占有线程是否是当前线程
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
        
        @ReservedStackAccess
        protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
        
        protected final boolean isHeldExclusively() {
            return getExclusiveOwnerThread() == Thread.currentThread();
        }
        
        final ConditionObject newCondition() {
            return new ConditionObject();
        }

        final Thread getOwner() {
            return getState() == 0 ? null : getExclusiveOwnerThread();
        }

        final int getHoldCount() {
            return isHeldExclusively() ? getState() : 0;
        }

        final boolean isLocked() {
            return getState() != 0;
        }
        
        private void readObject(java.io.ObjectInputStream s)
            throws java.io.IOException, ClassNotFoundException {
            s.defaultReadObject();
            setState(0); // reset to unlocked state
        }
    }
    
    static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;
        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }
    
    static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;
        /**
         * Fair version of tryAcquire.  Don't grant access unless
         * recursive call or no waiters or is first.
         */
        @ReservedStackAccess
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }
    
    public ReentrantLock() {
        sync = new NonfairSync();
    }
    
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
    
    public void lock() {
        sync.acquire(1);
    }
    
    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }
    
    public boolean tryLock() {
        return sync.nonfairTryAcquire(1);
    }
    
    public boolean tryLock(long timeout, TimeUnit unit)
            throws InterruptedException {
        return sync.tryAcquireNanos(1, unit.toNanos(timeout));
    }
    
    public void unlock() {
        sync.release(1);
    }
    
    public Condition newCondition() {
        return sync.newCondition();
    }
    
    public int getHoldCount() {
        return sync.getHoldCount();
    }
    
    public boolean isHeldByCurrentThread() {
        return sync.isHeldExclusively();
    }
    
    public boolean isLocked() {
        return sync.isLocked();
    }
    
    public final boolean isFair() {
        return sync instanceof FairSync;
    }
    
    protected Thread getOwner() {
        return sync.getOwner();
    }
    
    public final boolean hasQueuedThreads() {
        return sync.hasQueuedThreads();
    }
    
    public final boolean hasQueuedThread(Thread thread) {
        return sync.isQueued(thread);
    }
    
    public final int getQueueLength() {
        return sync.getQueueLength();
    }
    
    protected Collection<Thread> getQueuedThreads() {
        return sync.getQueuedThreads();
    }
    
    public boolean hasWaiters(Condition condition) {
        if (condition == null)
            throw new NullPointerException();
        if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
            throw new IllegalArgumentException("not owner");
        return sync.hasWaiters((AbstractQueuedSynchronizer.ConditionObject)condition);
    }
    
    public int getWaitQueueLength(Condition condition) {
        if (condition == null)
            throw new NullPointerException();
        if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
            throw new IllegalArgumentException("not owner");
        return sync.getWaitQueueLength((AbstractQueuedSynchronizer.ConditionObject)condition);
    }
    
    protected Collection<Thread> getWaitingThreads(Condition condition) {
        if (condition == null)
            throw new NullPointerException();
        if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
            throw new IllegalArgumentException("not owner");
        return sync.getWaitingThreads((AbstractQueuedSynchronizer.ConditionObject)condition);
    }
    
    public String toString() {
        Thread o = sync.getOwner();
        return super.toString() + ((o == null) ?
                                   "[Unlocked]" :
                                   "[Locked by thread " + o.getName() + "]");
    }
}
```

