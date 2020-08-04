# AbstractQueuedSynchronizer抽象类

同步器是用来构建锁和其他同步组件的基础框架，它的实现主要依赖一个int成员变量来表示同步状态以及通过一个FIFO队列构成等待队列。它的**子类必须重写AQS的几个protected修饰的用来改变同步状态的方法**，其他方法主要是实现了排队和阻塞机制。

子类被**推荐定义为自定义同步组件的静态内部类**，同步器自身没有实现任何同步接口，它仅仅是定义了若干同步状态的获取和释放方法来供自定义同步组件的使用，同步器既支持独占式获取同步状态，也可以支持共享式获取同步状态，这样就可以方便的实现不同类型的同步组件。

同步器是实现锁（也可以是任意同步组件）的关键，在锁的实现中聚合同步器，利用同步器实现锁的语义。可以这样理解二者的关系：**锁是面向使用者，它定义了使用者与锁交互的接口，隐藏了实现细节；同步器是面向锁的实现者，它简化了锁的实现方式，屏蔽了同步状态的管理，线程的排队，等待和唤醒等底层操作**。锁和同步器很好的隔离了使用者和实现者所需关注的领域。

AQS的设计是使用模板方法设计模式，它将**一些方法开放给子类进行重写，而同步器给同步组件所提供模板方法又会重新调用被子类所重写的方法**。可以归纳总结为这么几点：

1. 同步组件\(这里不仅仅值锁，还包括CountDownLatch等\)的实现依赖于同步器AQS，在同步组件实现中，使用AQS的方式被推荐定义继承AQS的静态内存类；
2. AQS采用模板方法进行设计，AQS的protected修饰的方法需要由继承AQS的子类进行重写实现，当调用AQS的子类的方法时就会调用被重写的方法；
3. AQS负责同步状态的管理，线程的排队，等待和唤醒这些底层操作，而Lock等同步组件主要专注于实现同步语义；
4. 在重写AQS的方式时，使用AQS提供的`getState()`、`setState()`、`compareAndSetState()`方法进行修改同步状态

AQS提供的模板方法可以分为3类：

1. 独占式获取与释放同步状态；
2. 共享式获取与释放同步状态；
3. 查询同步队列中等待线程情况；

## 源码

```java
package java.util.concurrent.locks;

import java.lang.invoke.MethodHandles;
import java.lang.invoke.VarHandle;
import java.util.ArrayList;
import java.util.Collection;
import java.util.Date;
import java.util.concurrent.TimeUnit;

public abstract class AbstractQueuedSynchronizer 
    extends AbstractOwnableSynchronizer 
    implements java.io.Serializable {

    private static final long serialVersionUID = 7373984972572414691L;
    
    protected AbstractQueuedSynchronizer() { }
    
    static final class Node {
        
        static final Node SHARED = new Node();
        static final Node EXCLUSIVE = null;

        static final int CANCELLED =  1;
        static final int SIGNAL    = -1;
        static final int CONDITION = -2;
        static final int PROPAGATE = -3;
        
        volatile int waitStatus;
        volatile Node prev;
        volatile Node next;
        volatile Thread thread;
        Node nextWaiter;
        
        final boolean isShared() {
            return nextWaiter == SHARED;
        }
        
        final Node predecessor() {
            Node p = prev;
            if (p == null)
                throw new NullPointerException();
            else
                return p;
        }
        
        Node() {}

        Node(Node nextWaiter) {
            this.nextWaiter = nextWaiter;
            THREAD.set(this, Thread.currentThread());
        }

        Node(int waitStatus) {
            WAITSTATUS.set(this, waitStatus);
            THREAD.set(this, Thread.currentThread());
        }

        final boolean compareAndSetWaitStatus(int expect, int update) {
            return WAITSTATUS.compareAndSet(this, expect, update);
        }

        final boolean compareAndSetNext(Node expect, Node update) {
            return NEXT.compareAndSet(this, expect, update);
        }

        final void setPrevRelaxed(Node p) {
            PREV.set(this, p);
        }
        
        private static final VarHandle NEXT;
        private static final VarHandle PREV;
        private static final VarHandle THREAD;
        private static final VarHandle WAITSTATUS;
        
        static {
            try {
                MethodHandles.Lookup l = MethodHandles.lookup();
                NEXT = l.findVarHandle(Node.class, "next", Node.class);
                PREV = l.findVarHandle(Node.class, "prev", Node.class);
                THREAD = l.findVarHandle(Node.class, "thread", Thread.class);
                WAITSTATUS = l.findVarHandle(Node.class, "waitStatus", int.class);
            } catch (ReflectiveOperationException e) {
                throw new ExceptionInInitializerError(e);
            }
        }
    }
    
    private transient volatile Node head;
    private transient volatile Node tail;
    private volatile int state;
    
    static final long SPIN_FOR_TIMEOUT_THRESHOLD = 1000L;
    
    // ...
    
}
```

状态的更新使用`getState`，`setState`以及`compareAndSetState`这三个方法。

```java
private volatile int state;
private static final VarHandle STATE;

protected final int getState() {
    return state;
}

protected final void setState(int newState) {
    state = newState;
}

protected final boolean compareAndSetState(int expect, int update) {
    return STATE.compareAndSet(this, expect, update);
}
```

AQS可重写的方法

```java
// 独占式获取同步状态，实现该方法需要查询当前状态并判断同步状态是否符合预期，
// 然后再实行CAS设置同步状态
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}

// 独占式释放同步状态，等待获取同步状态的线程将有机会获取同步状态
protected boolean tryRelease(int arg) {
    throw new UnsupportedOperationException();
}

// 共享式获取同步状态，返回大于等于0的值，表示获取成功，反之获取失败
protected int tryAcquireShared(int arg) {
    throw new UnsupportedOperationException();
}

// 共享式释放同步状态
protected boolean tryReleaseShared(int arg) {
    throw new UnsupportedOperationException();
}    

// 当前同步器是否在独占模式下被线程占用，一般该方法表示是否被当前线程所独占
protected boolean isHeldExclusively() {
    throw new UnsupportedOperationException();
}
```

实现同步组件时AQS提供的模板方法

```java
// 独占式获取同步状态，如果当前线程获取同步状态成功，则由该方法返回，
// 否则，将进入同步队列等待，该方法将会调用重写的tryAcquire(int arg)方法
public final void acquire(int arg) {
    if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}

// 与acquire(int arg)相同，但是该方法响应中端，当前线程未获取到同步状态会进入同步状态，
// 如果当前线程被中断，则该方法会抛出InterruptedException并返回
public final void acquireInterruptibly(int arg) throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    if (!tryAcquire(arg))
        doAcquireInterruptibly(arg);
}

// 与acquireInterruptibly(int arg)基础上增加了超时限制，如果当前线程在超时前没有获取到同步状态
// 那么将返回false，如果获取到了则返回true
public final boolean tryAcquireNanos(int arg, long nanosTimeout) throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    return tryAcquire(arg) || doAcquireNanos(arg, nanosTimeout);
}

// 共享式获取同步状态，如果当前线程未获取到同步状态，将会进入同步队列等待，
// 与独占式获取的主要区别是在同一时间可以有多个线程获取同步状态
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}

// 与acquiredShared(int arg)相同，该方法响应中段
public final void acquireSharedInterruptibly(int arg) throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    if (tryAcquireShared(arg) < 0)
        doAcquireSharedInterruptibly(arg);
}

// 在acquireSharedInterruptibly(int arg)基础上增加了超市限制
public final boolean tryAcquireSharedNanos(int arg, long nanosTimeout) throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    return tryAcquireShared(arg) >= 0 || doAcquireSharedNanos(arg, nanosTimeout);
}

// 独占式的释放同步状态，该方法会在释放同步状态之后，将同步队列中第一个节点包含的线程唤醒
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}

// 共享式的释放同步状态
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}

// 获取等待在同步队列上的线程集合
public final Collection<Thread> getQueuedThreads() {
    ArrayList<Thread> list = new ArrayList<>();
    for (Node p = tail; p != null; p = p.prev) {
        Thread t = p.thread;
        if (t != null)
            list.add(t);
    }
    return list;
}
```

## 获取锁的过程

```java
// 获取起点
public final void acquire(int arg) {
    //先看同步状态是否获取成功，如果成功则方法结束返回
		//若失败则先调用addWaiter()方法再调用acquireQueued()方法
    if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}

protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}

private Node addWaiter(Node mode) {
    // 将当前线程构建成Node类型
    Node node = new Node(mode);

    for (;;) {
        Node oldTail = tail;
        // 当前尾节点是否为null
        if (oldTail != null) {
            node.setPrevRelaxed(oldTail);
            // 将当前节点尾插入的方式插入同步队列中
            if (compareAndSetTail(oldTail, node)) {
                oldTail.next = node;
                return node;
            }
        } else {
            initializeSyncQueue();
        }
    }
}

// CAS操作
private final boolean compareAndSetTail(Node expect, Node update) {
    return TAIL.compareAndSet(this, expect, update);
}

private final void initializeSyncQueue() {
    Node h;
    if (HEAD.compareAndSet(this, null, (h = new Node())))
        tail = h;
}

final boolean acquireQueued(final Node node, int arg) {
    boolean interrupted = false;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node))
                interrupted |= parkAndCheckInterrupt();
        }
    } catch (Throwable t) {
        cancelAcquire(node);
        if (interrupted)
            selfInterrupt();
        throw t;
    }
}
```

## 释放锁的过程

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}

protected boolean tryRelease(int arg) {
    throw new UnsupportedOperationException();
}

private void unparkSuccessor(Node node) {
    int ws = node.waitStatus;
    if (ws < 0)
        node.compareAndSetWaitStatus(ws, 0);
    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node p = tail; p != node && p != null; p = p.prev)
            if (p.waitStatus <= 0)
                s = p;
    }
    if (s != null)
        LockSupport.unpark(s.thread);
}
```

