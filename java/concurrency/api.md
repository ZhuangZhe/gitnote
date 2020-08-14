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
| `void start()` |  |
| `static void yield()` | A hint to the scheduler that the current thread is willing to yield its current use of a processor. |

## Object

| API | 简介 |
| :--- | :--- |
| `void notify()` | 唤醒任意一个`WAITING`状态的线程。 |
| `void notifyAll()` | 唤醒所有`WAITING`状态的线程。 |
| `void wait()` | 使持有对象锁的线程进入`WAITING`状态，直到被唤醒。 |
| `void wait(long timeoutMillis)` | 使持有对象锁的线程进入`WAITING`状态，直到被唤醒。。 |
| `void wait(long timeoutMillis, int nanos)` | 使持有对象锁的线程进入`WAITING`状态，直到被唤醒。 |











