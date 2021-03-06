# IO

## 基本概念

Java中的BIO、NIO和AIO理解为是 Java语言对操作系统的各种IO模型的封装。程序员在使用这些API的时候，不需要关心操作系统层面的知识，也不需要根据不同操作系统编写不同的代码。只需要使用Java的API就可以了。

### 同步和异步

* **同步** ：两个同步任务相互依赖，并且一个任务必须以依赖于另一任务的某种方式执行。 比如在`A->B`事件模型中，你需要先完成A才能执行B。 再换句话说，同步调用种被调用者未处理完请求之前，调用不返回，调用者会一直等待结果的返回。
* **异步**： 两个异步的任务完全独立的，一方的执行不需要等待另外一方的执行。再换句话说，异步调用种一调用就返回结果不需要等待结果返回，当结果返回的时候通过回调函数或者其他方式拿着结果再做相关事情，

### **阻塞和非阻塞**

* **阻塞：** 阻塞就是发起一个请求，调用者一直等待请求结果返回，也就是当前线程会被挂起，无法从事其他任务，只有当条件就绪才能继续。
* **非阻塞：** 非阻塞就是发起一个请求，调用者不用一直等着结果返回，可以先去干其他事情。

**如何区分 “同步/异步 ”和 “阻塞/非阻塞” 呢？**

同步/异步是从行为角度描述事物的，而阻塞和非阻塞描述的当前事物的状态\(等待调用结果时的状态\)。

### BIO

`java.io`包提供了我们最熟知的一些IO功能，比如File抽象、输入输出流等。交互方式是同步、阻塞的方式，也就是说，在读取输入流或者写入输出流时，在读、写动作完成之前，线程会一直阻塞在那里，它们之间的调用是可靠的线性顺序。

`java.io`包的好处是代码比较简单、直观；缺点则是IO效率和扩展性存在局限性，容易成为应用性能的瓶颈。

很多时候，人们也把`java.net`下面提供的部分网络 API，比如`Socket`、`ServerSocket`、`HttpURLConnection`也归类到同步阻塞IO类库，因为网络通信同样是IO行为。

### NIO

`java.nio`包提供了`Channel`、`Selector`、`Buffer`等新的抽象，可以构建多路复用的、同步非阻塞IO程序，同时提供了更接近操作系统底层的高性能数据操作方式。

### AIO

在 Java 7 中，NIO有了进一步的改进，也就是NIO2，引入了异步非阻塞IO方式，也有很多人叫它AIO\(Asynchronous IO\)。异步IO操作基于事件和回调机制，可以简单理解为，应用操作直接返回，而不会阻塞在那里，当后台处理完成，操作系统会通知相应线程进行后续工作。

## BIO\(Blocking I/O\)

同步阻塞I/O模式，数据的读取写入必须阻塞在一个线程内等待其完成。

### 传统 BIO

BIO通信\(一请求一应答\)模型图如下：

![&#x4F20;&#x7EDF;BIO&#x901A;&#x4FE1;&#x6A21;&#x578B;&#x56FE;](https://camo.githubusercontent.com/5ef6de9824ae82bb0c403522a647953d1193a362/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f322e706e67)

采用**BIO通信模型**的服务端，通常由一个独立的Acceptor线程负责监听客户端的连接。我们一般通过在`while(true)`循环中服务端会调用`accept()`方法等待接收客户端的连接的方式监听请求，请求一旦接收到一个连接请求，就可以建立通信套接字在这个通信套接字上进行读写操作，此时不能再接收其他客户端连接请求，只能等待同当前连接的客户端的操作执行完成， 不过可以通过多线程来支持多个客户端的连接，如上图所示。

如果要让**BIO通信模型**能够同时处理多个客户端请求，就必须使用多线程\(主要原因是`socket.accept()`、`socket.read()`、`socket.write()` 涉及的三个主要函数都是同步阻塞的\)，也就是说它在接收到客户端连接请求之后为每个客户端创建一个新的线程进行链路处理，处理完成之后，通过输出流返回应答给客户端，线程销毁。这就是典型的**一请求一应答通信模型**。我们可以设想一下如果这个连接不做任何事情的话就会造成不必要的线程开销，不过可以通过**线程池机制**改善，线程池还可以让线程的创建和回收成本相对较低。使用`FixedThreadPool`可以有效的控制了线程的最大数量，保证了系统有限的资源的控制，实现了`N(客户端请求数量) : M(处理客户端请求的线程数量)`的伪异步I/O模型\(N可以远远大于M\)，下面一节"伪异步 BIO"中会详细介绍到。

**我们再设想一下当客户端并发访问量增加后这种模型会出现什么问题？**

在Java虚拟机中，线程是宝贵的资源，线程的创建和销毁成本很高，除此之外，线程的切换成本也是很高的。尤其在Linux这样的操作系统中，线程本质上就是一个进程，创建和销毁线程都是重量级的系统函数。如果并发访问量增加会导致线程数急剧膨胀可能会导致线程堆栈溢出、创建新线程失败等问题，最终导致进程宕机或者僵死，不能对外提供服务。

### 伪异步 IO

为了解决同步阻塞I/O面临的一个链路需要一个线程处理的问题，后来有人对它的线程模型进行了优化，即后端通过一个线程池来处理多个客户端的请求接入，形成客户端个数M：线程池最大线程数N的比例关系，其中M可以远远大于N。通过线程池可以灵活地调配线程资源，设置线程的最大值，防止由于海量并发接入导致线程耗尽。

![&#x4F2A;&#x5F02;&#x6B65;IO&#x6A21;&#x578B;&#x56FE;](https://camo.githubusercontent.com/04b258a50ca7f9762f43d64e70f4489440bae4eb/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f332e706e67)

采用线程池和任务队列可以实现一种叫做伪异步的I/O通信框架，它的模型图如上图所示。当有新的客户端接入时，将客户端的Socket封装成一个Task\(该任务实现`java.lang.Runnable`接口\)投递到后端的线程池中进行处理，JDK的线程池维护一个消息队列和N个活跃线程，对消息队列中的任务进行处理。由于线程池可以设置消息队列的大小和最大线程数，因此，它的资源占用是可控的，无论多少个客户端并发访问，都不会导致资源的耗尽和宕机。

伪异步I/O通信框架采用了线程池实现，因此避免了为每个请求都创建一个独立线程造成的线程资源耗尽问题。不过因为它的底层仍然是同步阻塞的BIO模型，因此无法从根本上解决问题。

### 代码示例

下面代码中演示了BIO通信\(一请求一应答\)模型。我们会在客户端创建多个线程依次连接服务端并向其发送"当前时间+:hello world"，服务端会为每个客户端线程创建一个线程来处理。代码示例出自闪电侠的博客，原地址如下：

[https://www.jianshu.com/p/a4e03835921a](https://www.jianshu.com/p/a4e03835921a)

**客户端**

```java
public class IOClient {

  public static void main(String[] args) {
    // TODO 创建多个线程，模拟多个客户端连接服务端
    new Thread(() -> {
      try {
        Socket socket = new Socket("127.0.0.1", 3333);
        while (true) {
          try {
            socket.getOutputStream().write((new Date() + ": hello world").getBytes());
            Thread.sleep(2000);
          } catch (Exception e) {
          }
        }
      } catch (IOException e) {
      }
    }).start();

  }

}
```

**服务端**

```java
public class IOServer {

  public static void main(String[] args) throws IOException {
    // TODO 服务端处理客户端连接请求
    ServerSocket serverSocket = new ServerSocket(3333);

    // 接收到客户端连接请求之后为每个客户端创建一个新的线程进行链路处理
    new Thread(() -> {
      while (true) {
        try {
          // 阻塞方法获取新的连接
          Socket socket = serverSocket.accept();

          // 每一个新的连接都创建一个线程，负责读取数据
          new Thread(() -> {
            try {
              int len;
              byte[] data = new byte[1024];
              InputStream inputStream = socket.getInputStream();
              // 按字节流方式读取数据
              while ((len = inputStream.read(data)) != -1) {
                System.out.println(new String(data, 0, len));
              }
            } catch (IOException e) {
            }
          }).start();

        } catch (IOException e) {
        }

      }
    }).start();

  }

}
```

#### 1.4 总结

在活动连接数不是特别高（小于单机1000）的情况下，这种模型是比较不错的，可以让每一个连接专注于自己的 I/O 并且编程模型简单，也不用过多考虑系统的过载、限流等问题。线程池本身就是一个天然的漏斗，可以缓冲一些系统处理不了的连接或请求。但是，当面对十万甚至百万级连接的时候，传统的BIO模型是无能为力的。因此，我们需要一种更高效的I/O处理模型来应对更高的并发量。

## NIO\(New I/O\)

NIO是一种同步非阻塞的I/O模型，在Java 1.4l中引入了NIO框架，对应`java.nio`包，提供了`Channel` 、`Selector`、`Buffer`等抽象。

NIO中的N可以理解为Non-blocking，不单纯是New。它支持面向缓冲的，基于通道的I/O操作方法。 NIO提供了与传统BIO模型中的`Socket`和 `ServerSocket` 相对应的`SocketChannel`和`ServerSocketChannel`两种不同的套接字通道实现，两种通道都支持阻塞和非阻塞两种模式。阻塞模式使用就像传统中的支持一样，比较简单，但是性能和可靠性都不好；非阻塞模式正好与之相反。对于低负载、低并发的应用程序，可以使用同步阻塞I/O来提升开发速率和更好的维护性；对于高负载、高并发的\(网络\)应用，应使用NIO的非阻塞模式来开发。

### NIO的特性/NIO与IO区别

如果是在面试中回答这个问题，我觉得首先肯定要从NIO流是非阻塞IO而IO流是阻塞IO说起。然后，可以从NIO的3个核心组件/特性为NIO带来的一些改进来分析。如果，你把这些都回答上了我觉得你对于NIO就有了更为深入一点的认识，面试官问到你这个问题，你也能很轻松的回答上来了。

#### **Non-blocking IO\(非阻塞IO\)**

**IO流是阻塞的，NIO流是不阻塞的。**

Java NIO使我们可以进行非阻塞IO操作。比如说，单线程中从通道读取数据到buffer，同时可以继续做别的事情，当数据读取到buffer中后，线程再继续处理数据。写数据也是一样的。另外，非阻塞写也是如此。一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。

Java IO的各种流是阻塞的。这意味着，当一个线程调用`read()`或`write()`时，该线程被阻塞，直到有一些数据被读取，或数据完全写入。该线程在此期间不能再干任何事情了

#### **Buffer\(缓冲区\)**

**IO面向流\(Stream oriented\)，而NIO面向缓冲区\(Buffer oriented\)。**

`Buffer`是一个对象，它包含一些要写入或者要读出的数据。在NIO类库中加入`Buffer`对象，体现了新库与原I/O的一个重要区别。在面向流的I/O中可以将数据直接写入或者将数据直接读到`Stream`对象中。虽然`Stream`中也有`Buffer`开头的扩展类，但只是流的包装类，还是从流读到缓冲区，而NIO却是直接读到`Buffer`中进行操作。

在NIO厍中，所有数据都是用缓冲区处理的。在读取数据时，它是直接读到缓冲区中的; 在写入数据时，写入到缓冲区中。任何时候访问NIO中的数据，都是通过缓冲区进行操作。

最常用的缓冲区是`ByteBuffer` ，一个 `ByteBuffer`提供了一组功能用于操作byte数组。除了`ByteBuffer`，有还其他的一些缓冲区，事实上，每一种Java基本类型\(除了Boolean类型\)都对应有一种缓冲区。

#### **Channel \(通道\)**

**NIO通过Channel进行读写。**

通道是双向的，可读也可写，而流的读写是单向的。无论读写，通道只能和`Buffer`交互。因为`Buffer`，通道可以异步地读写。

#### **Selector \(选择器\)**

**NIO有选择器，而IO没有。**

选择器用于使用单个线程处理多个通道。因此，它需要较少的线程来处理这些通道。线程之间的切换对于操作系统来说是昂贵的。 因此，为了提高系统效率选择器是有用的。

![&#x4E00;&#x4E2A;&#x5355;&#x7EBF;&#x7A0B;&#x4E2D;Selector&#x7EF4;&#x62A4;3&#x4E2A;Channel&#x7684;&#x793A;&#x610F;&#x56FE;](https://camo.githubusercontent.com/3a68153ce17be90275df07a47409afaea91aff83/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d322f536c6563746f722e706e67)

### NIO读数据和写数据方式

通常来说NIO中的所有IO都是从Channel\(通道\)开始的。

* 从通道进行数据读取 ：创建一个缓冲区，然后请求通道读取数据。
* 从通道进行数据写入 ：创建一个缓冲区，填充数据，并要求通道写入数据。

数据读取和写入操作图示：

[![NIO&#x8BFB;&#x5199;&#x6570;&#x636E;&#x7684;&#x65B9;&#x5F0F;](https://camo.githubusercontent.com/33ad8802c0f295f567c7432de3a58a014b27fa51/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d322f4e494f2545382541462542422545352538362539392545362539352542302545362538442541452545372539412538342545362539362542392545352542432538462e706e67)](https://camo.githubusercontent.com/33ad8802c0f295f567c7432de3a58a014b27fa51/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d322f4e494f2545382541462542422545352538362539392545362539352542302545362538442541452545372539412538342545362539362542392545352542432538462e706e67)

### NIO核心组件简单介绍

NIO包含下面几个核心的组件：

* Channel\(通道\)
* Buffer\(缓冲区\)
* Selector\(选择器\)

整个NIO体系包含的类远远不止这三个，只能说这三个是NIO体系的“核心API”。我们上面已经对这三个概念进行了基本的阐述，这里就不多做解释了。

### 代码示例

代码示例出自闪电侠的博客，原地址如下：

[https://www.jianshu.com/p/a4e03835921a](https://www.jianshu.com/p/a4e03835921a)

客户端 IOClient.java 的代码不变，我们对服务端使用 NIO 进行改造。以下代码较多而且逻辑比较复杂，大家看看就好。

```java
public class NIOServer {
    public static void main(String[] args) throws IOException {
    
        // 1. serverSelector负责轮询是否有新的连接，服务端监测到新的连接之后，不再创建一个新的线程，
        // 而是直接将新连接绑定到clientSelector上，这样就不用 IO 模型中 1w 个 while 循环在死等
        Selector serverSelector = Selector.open();
        // 2. clientSelector负责轮询连接是否有数据可读
        Selector clientSelector = Selector.open();
        
        new Thread(() -> {
            try {
                // 对应IO编程中服务端启动
                ServerSocketChannel listenerChannel = ServerSocketChannel.open();
                listenerChannel.socket().bind(new InetSocketAddress(3333));
                listenerChannel.configureBlocking(false);
                listenerChannel.register(serverSelector, SelectionKey.OP_ACCEPT);
            
                while (true) {
                    // 监测是否有新的连接，这里的1指的是阻塞的时间为 1ms
                    if (serverSelector.select(1) > 0) {
                        Set<SelectionKey> set = serverSelector.selectedKeys();
                        Iterator<SelectionKey> keyIterator = set.iterator();
            
                        while (keyIterator.hasNext()) {
                            SelectionKey key = keyIterator.next();
                
                            if (key.isAcceptable()) {
                                try {
                                    // (1) 每来一个新连接，不需要创建一个线程，而是直接注册到clientSelector
                                    SocketChannel clientChannel = ((ServerSocketChannel) key.channel()).accept();
                                    clientChannel.configureBlocking(false);
                                    clientChannel.register(clientSelector, SelectionKey.OP_READ);
                                } finally {
                                    keyIterator.remove();
                                }
                            }
                
                        }
                    }
                }
            } catch (IOException ignored) {
            }
        }).start();
        
        new Thread(() -> {
            try {
                while (true) {
                    // (2) 批量轮询是否有哪些连接有数据可读，这里的1指的是阻塞的时间为 1ms
                    if (clientSelector.select(1) > 0) {
                        Set<SelectionKey> set = clientSelector.selectedKeys();
                        Iterator<SelectionKey> keyIterator = set.iterator();
                        
                        while (keyIterator.hasNext()) {
                            SelectionKey key = keyIterator.next();
                            
                            if (key.isReadable()) {
                                try {
                                    SocketChannel clientChannel = (SocketChannel) key.channel();
                                    ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
                                    // (3) 面向 Buffer
                                    clientChannel.read(byteBuffer);
                                    byteBuffer.flip();
                                    System.out.println(
                                    Charset.defaultCharset().newDecoder().decode(byteBuffer).toString());
                                } finally {
                                    keyIterator.remove();
                                    key.interestOps(SelectionKey.OP_READ);
                                }
                            }
                        
                        }
                    }
                }
            } catch (IOException ignored) {
            }
        }).start();
    
    }
}
```

为什么大家都不愿意用JDK原生NIO进行开发呢？从上面的代码中大家都可以看出来，是真的难用！除了编程复杂、编程模型难之外，它还有以下让人诟病的问题：

* JDK的NIO底层由epoll实现，该实现饱受诟病的空轮询bug会导致cpu飙升100%
* 项目庞大之后，自行实现的NIO很容易出现各类 bug，维护成本较高，上面这一坨代码我都不能保证没有 bug

Netty的出现很大程度上改善了JDK原生NIO所存在的一些让人难以忍受的问题。

## AIO \(Asynchronous I/O\)

AIO也就是 NIO 2。在 Java 7中引入了NIO的改进版NIO 2，它是异步非阻塞的IO模型。异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。

AIO是异步IO的缩写，虽然NIO在网络操作中，提供了非阻塞的方法，但是NIO的IO行为还是同步的。对于NIO来说，我们的业务线程是在IO操作准备好时，得到通知，接着就由这个线程自行进行 IO 操作，IO操作本身是同步的。（除了 AIO 其他的 IO 类型都是同步的，这一点可以从底层IO线程模型解释，推荐一篇文章：[《漫话：如何给女朋友解释什么是Linux的五种IO模型？》](https://mp.weixin.qq.com/s?__biz=Mzg3MjA4MTExMw==&mid=2247484746&idx=1&sn=c0a7f9129d780786cabfcac0a8aa6bb7&source=41#wechat_redirect) ）

查阅网上相关资料，我发现就目前来说 AIO 的应用还不是很广泛，Netty 之前也尝试使用过 AIO，不过又放弃了。



**参考资料：**

* \*\*\*\*[**Java BIO**](https://dunwu.github.io/javacore/io/java-bio.html)\*\*\*\*

