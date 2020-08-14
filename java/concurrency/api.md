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
| `void join(long millis)` | Waits at most `millis` milliseconds for this thread to die. |
| `void join(long millis, int nanos)` | Waits at most `millis` milliseconds plus `nanos` nanoseconds for this thread to die. |
| `void run()` | If this thread was constructed using a separate `Runnable` run object, then that `Runnable` object's `run` method is called; otherwise, this method does nothing and returns. |
| `static void sleep(long millis)` | Causes the currently executing thread to sleep \(temporarily cease execution\) for the specified number of milliseconds, subject to the precision and accuracy of system timers and schedulers. |
| `static void sleep(long millis, int nanos)` | Causes the currently executing thread to sleep \(temporarily cease execution\) for the specified number of milliseconds plus the specified number of nanoseconds, subject to the precision and accuracy of system timers and schedulers. |
| `void start()` | Causes this thread to begin execution; the Java Virtual Machine calls the `run` method of this thread. |
| `static void yield()` | A hint to the scheduler that the current thread is willing to yield its current use of a processor. |

## Object

| API | 简介 |
| :--- | :--- |
| `void notify()` | Wakes up a single thread that is waiting on this object's monitor. |
| `void notifyAll()` | Wakes up all threads that are waiting on this object's monitor. |
| `void wait()` | Causes the current thread to wait until it is awakened, typically by being _notified_ or _interrupted_. |
| `void wait(long timeoutMillis)` | Causes the current thread to wait until it is awakened, typically by being _notified_ or _interrupted_, or until a certain amount of real-time has elapsed. |
| `void wait(long timeoutMillis, int nanos)` | Causes the current thread to wait until it is awakened, typically by being _notified_ or _interrupted_, or until a certain amount of real-time has elapsed. |











