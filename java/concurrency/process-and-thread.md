# 进程与线程

### **进程**

一个程序是静态的，通常是存放在外存中的。而当程序被调入内存中运行后，就成了进程。 顾名思义，进程就是进行中的程序，它是个动态的概念。

**进程**是程序在`执行过程中分配和管理资源的基本单位`，每一个进程都有一个自己的地址空间，

至少有 5 种基本状态：`初始态、执行态、等待状态、就绪状态、终止状态`。

**进程一般有如下三个特点：**

* **独立性：**每个进程拥有自己独立的资源，拥有私有的地址空间，其大小与处理机位数有关，如Win32系统，32位地址映射4GB地址空间，其中低地址的2GB作为用户模式的虚拟地址空间（应用程序可共享，线程间独立），高地址的2GB作为内核模式的虚拟地址空间（系统使用）。
* **动态性：**这点从进程的概念可以看出，运行中的程序就是进程。进程中有时间、状态、生命周期等动态的概念。
* **并发性：**多个进程在单个处理器上并发执行。

**进程主要包含了三个部分：**

* **程序代码：**用于描述进程要完成的功能。
* **数据集合：**程序执行所需要的数据与工作区域。
* **PCB程序控制块：**包含进程的描述信息与控制信息，是进程的唯一标志，也正是因为有了PCB，进程就成了一个动态的概念。

### **线程**

* `线程是CPU调度和分派的基本单位`，它可与同属一个进程的其他的线程共享进程所拥有的全部资源。
  * `进程`和`线程`的关系：线程是进程的一部分，一个线程只能属于一个进程，而一个进程可以有多个线程，但至少有一个线程。

### 进程与线程的区别

* `根本区别`：`进程`是操作系统资源分配的基本单位，而`线程`是任务调度和执行的基本单位。
* `开销方面`：每个`进程`都有独立的代码和数据空间\(程序上下文\)，`进程`之间切换开销大；`线程`可以看做轻量级的进程，同一类`线程`共享代码和数据空间，每个`线程`都有自己独立的运行栈和程序计数器，`线程`之间切换的开销小。
* `所处环境`：在操作系统中能同时运行多个`进程`\(程序\)；而在同一个`进程`中有多个`线程`同时执行\(通过CPU调度，在每个时间片中只有一个`线程`执行\)。
* `内存分配`：系统为每个`进程`分配不同的内存空间；而对`线程`而言，除了CPU外，系统不会为`线程`分配内存\(`线程`所使用的资源来自其所属`进程`的资源\)，`线程`组之间只能共享资源。
* `包含关系`：`线程`是`进程`的一部分，所以`线程`也被称为轻权进程或者轻量级进程。

### **线程与进程关系**

一个进程内部可能包含了很多顺序执行流，每个顺序执行流就是一个线程。 现在操作系统大多使用抢占式多任务操作策略，以支持多进程的并发性，而多线程是多进程的扩展，使一个进程也能像一个处理器一样并发处理多项任务，线程就是进程中并发执行的基本单位，线程也因此被称为“轻量级进程”。一个进程可以包含多个线程，每条线程都有其父进程。

### 多线程

是指从软件或者硬件上实现**多**个**线程**并发执行的技术。 具有**多线程**能力的计算机因有硬件支持而能够在同一时间执行**多**于一个**线程**，进而提升整体处理性能。

## 相关概念

### 并发与并行

* `并发`：一个时间段中有几个程序都处于已启动运行到运行完毕之间，且这几个程序都是在同一个处理机上运行，但任一个时刻点上只有一个程序在处理机上运行。
* `并行`：两个或多个事件\(线程\)在同一时刻发生，在不同的CPU资源上，同时执行。

### 并发的优点

* 充分利用多核CPU的计算能力。
* 方便进行业务拆分，提升应用性能。

### 并发的缺点

* 频繁的上下文切换
  * 时间片是CPU分配给各个线程的时间，因为时间非常短，所以CPU不断通过切换线程，让我们觉得多个线程是同时执行的，时间片一般是几十毫秒。而每次切换时，需要保存当前的状态起来，以便能够进行恢复先前状态，而这个切换时非常损耗性能，过于频繁反而无法发挥出多线程编程的优势。
  * 通常减少上下文切换可以`采用无锁并发编程`，`CAS算法`，`使用最少的线程`和`使用协程`。
* 线程安全
  * 多线程编程中最难以把握的就是临界区线程安全问题，稍微不注意就会出现死锁的情况，一旦产生死锁就会造成系统功能不可用。
  * 避免死锁的方法：
    * 避免一个线程同时获得多个锁。
    * 避免一个线程在锁内部占有多个资源，尽量保证每个锁只占用一个资源。
    * 尝试使用定时锁，使用`lock.tryLock(timeOut)`，当超时等待时当前线程不会阻塞。
    * 对于数据库锁，加锁和解锁必须在一个数据库连接里，否则会出现解锁失败的情况。

### 同步与异步

* `同步`：按顺序执行，执行完一个再执行下一个，需要等待、协调运行。
* `异步`：彼此独立,在等待某事件的过程中继续做自己的事，不需要等待这一事件完成后再工作。
  * `异步`和`多线程`并不是一个同等关系，`异步`是最终目的，`多线程`只是我们实现异步的一种手段。

### 阻塞和非阻塞

* `阻塞`和`非阻塞`通常用来形容多线程间的相互影响，比如一个线程占有了临界区资源，那么其他线程需要这个资源就必须进行等待该资源的释放，会导致等待的线程挂起，这种情况就是`阻塞`，而`非阻塞`就恰好相反，它强调没有一个线程可以阻塞其他线程，所有的线程都会尝试地往前运行。

### 临界区

* 临界区用来表示一种公共资源或者说是共享数据，可以被多个线程使用。但是每个线程使用时，一旦临界区资源被一个线程占有，那么其他线程必须等待。

\*\*\*\*

**参考资料：**

* \*\*\*\*[**进程、线程与多线程概念详解**](https://blog.csdn.net/zhangliangzi/article/details/51236026)\*\*\*\*

