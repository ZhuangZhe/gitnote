# API

## Runnable

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
|  |  |

## AbstractQueuedSynchronizer

| API | 简介 |
| :--- | :--- |
| `protected boolean tryAcquire(int arg)` | 独占方式。尝试获取资源，成功则返回true，失败则返回false。 |
| `protected int tryAcquireShared(int arg)` | 共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。 |
| `protected boolean tryRelease(int arg)` | 独占方式。尝试释放资源，成功则返回true，失败则返回false。 |
| `protected boolean tryReleaseShared(int arg)` | 共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回true，否则返回false。 |
| `protected boolean isHeldExclusively()` | 该线程是否正在独占资源。只有用到`condition`才需要去实现它。 |
| `void acquire(int arg)` | 获取资源的入口，内部调用`tryAcquire(int arg)`。 |

### Node



## LockSupport

| API | 简介 |
| :--- | :--- |
| `static void park()` |  |
| `static void unpark(Thread thread)` |  |















