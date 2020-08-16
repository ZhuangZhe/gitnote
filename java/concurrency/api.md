# API

## &lt;Runnable&gt;

| API | 简介 |
| :--- | :--- |
| `void run()` | When an object implementing this interface,`Runnable` is used to create a thread, starting the thread causes the object's `run` method to be called in that separately executing thread. |

## Thread

| API | 简介 |
| :--- | :--- |
| `static Thread currentThread()` | 返回当前正在执行的线程。 |
| `void interrupt()` | Interrupts this thread. |
| `void join()` | 等待当前的线程，直到其操作执行完毕，再继续执行下面的代码。 |
| `void join(long millis)` | 等待当前的线程，直到其操作执行完毕，再继续执行下面的代码。 |
| `void join(long millis, int nanos)` | 等待当前的线程，直到其操作执行完毕，再继续执行下面的代码。 |
| `void run()` | 继承自`Runnable`。 |
| `static void sleep(long millis)` | 使正在执行的线程暂停，但该线程并不会释放锁。 |
| `static void sleep(long millis, int nanos)` | 使正在执行的线程暂停，但该线程并不会释放锁。 |
| `void start()` | 以线程的形式执行。 |
| `static void yield()` | 使正在执行的线程让出CPU时间片。 |

## Object

| API | 简介 |
| :--- | :--- |
| `void notify()` | 唤醒任意一个`WAITING`状态的线程。 |
| `void notifyAll()` | 唤醒所有`WAITING`状态的线程。 |
| `void wait()` | 使持有对象锁的线程释放锁，进入`WAITING`状态，直到被唤醒。必须放在同步块或同步方法中。 |
| `void wait(long timeoutMillis)` | 使持有对象锁的线程释放锁，进入`TIMED_WAITING`状态，直到被唤醒或等待时间结束。必须放在同步块或同步方法中。 |
| `void wait(long timeoutMillis, int nanos)` | 使持有对象锁的线程释放锁，进入`TIMED_WAITING`状态，直到被唤醒或等待时间结束。必须放在同步块或同步方法中。 |

## ThreadLocal

| API | 简介 |
| :--- | :--- |


## \[AbstractQueuedSynchronizer\]

| API | 简介 |
| :--- | :--- |
| `protected boolean tryAcquire(int arg)` | 独占方式。尝试获取资源，成功则返回true，失败则返回false。 |
| `protected int tryAcquireShared(int arg)` | 共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。 |
| `protected boolean tryRelease(int arg)` | 独占方式。尝试释放资源，成功则返回true，失败则返回false。 |
| `protected boolean tryReleaseShared(int arg)` | 共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回true，否则返回false。 |
| `protected boolean isHeldExclusively()` | 该线程是否正在独占资源。只有用到`condition`才需要去实现它。 |
| `void acquire(int arg)` | 获取资源的入口，内部调用`tryAcquire(int arg)`。 |
| `void acquireShared(int arg)` | 申请共享模式的资源。 |
| `void acquireInterruptibly(int arg)` | 独占方式。申请可中断的资源。 |
| `void acquireSharedInterruptibly(int arg)` | 共享方式。申请可中断的资源。 |

### Node

| Field | 简介 |
| :--- | :--- |
| `static final Node SHRAED` | 标记一个节点\(对应的线程\)在共享模式下等待。 |
| `static final Node EXCLUSIVE` | 标记一个节点\(对应的线程\)在独占模式下等待。 |
| `static final int CANCELLED` | waitStatus，值为1，表示该节点\(对应的线程\)已被取消。 |
| `static final int SIGNAL` | waitStatus，值为-1，表示该节点\(对应的线程\)需要被唤醒。 |
| `static final int CONDITION` | waitStatus，值为-2，表示该节点\(对应的线程\)在等待某一个条件。 |
| `static final int PROPAGATE` | waitStatus，值为-3，表示有资源可用，新head结点需要继续唤醒后继结点\(共享模式下，多线程并发释放资源，而head唤醒其后继结点后，需要把多出来的资源留给后面的结点；设置新的head结点时，会继续唤醒其后继结点\)。 |
| `volatile int waitStatus` |  |
| `volatile Node prev` | 前驱节点。 |
| `volatile Node next` | 后驱节点。 |
| `volatile Thread thread` | 节点对应的线程。 |
| `Node nextWaiter` | 等待队列里的下一个等待条件的节点。 |

## LockSupport

| API | 简介 |
| :--- | :--- |
| `static void park()` | 阻塞当前线程。 |
| `static void unpark(Thread thread)` | 停止阻塞指定线程。 |

## &lt;Executor&gt;

| API | 简介 |
| :--- | :--- |
| `void execute(Runnable command)` | Executes the given command at some time in the future. |

### ThreadPoolExecutor

| 构造器参数 | 简介 | 必须提供 |
| :--- | :--- | :--- |
| `int corePoolSize` | 该线程池中**核心线程数最大值。** | 是 |
| `int maximumPoolSize` | 该线程池中**线程总数最大值** 。 | 是 |
| `long keepAliveTime` | **非核心线程闲置超时时长。** | 是 |
| `TimeUnit unit` | keepAliveTime的单位。 | 是 |
| `BlockingQueue workQueue` | 阻塞队列，维护着**等待执行的Runnable任务对象**。 | 是 |
| `ThreadFactory threadFactory` | 创建线程的工厂 ，用于批量创建线程，统一在创建线程时设置一些参数，如是否守护线程、线程的优先级等。如果不指定，会新建一个默认的线程工厂。 | 否 |
| `RejectedExecutionHandler handler` | **拒绝处理策略**，线程数量大于最大线程数就会采用拒绝处理策略。 | 否 |

## &lt;BlockingQueue&gt;

| API | 简介 |
| :--- | :--- |
| `boolean add(E e)` | 插入失败时抛出异常。 |
| `boolean offer(E e)` | 插入成功返回true，反之返回false。 |
| `void put(E e)` | 如插入失败，会一直阻塞或响应终止。 |
| `boolean offer(E e, long timeout, TimeUnit unit)` | 如插入失败，会发生阻塞直到执行，超时后返回false。 |
| `boolean remove(Object o)` | 移除失败时抛出异常。 |
| `E take()` | 移除并返回队列最前面的元素，如队列为空会阻塞直到有元素可以获取。 |
| `E poll(long timeout, TimeUnit unit)` | 移除并返回队列最前面的元素，如队列为空会阻塞直到有元素可以获取，超时后返回特定值。 |

















