# 网络



![](../.gitbook/assets/image%20%2813%29.png)

#### 超文本传输协议\(HTTP\)

HTTP是一种在客户端和服务器之间编码和传输数据的方法。它是一个请求/响应协议：客户端和服务端针对相关内容和完成状态信息的请求和响应。HTTP是独立的，允许请求和响应流经许多执行负载均衡，缓存，加密和压缩的中间路由器和服务器。

一个基本的HTTP请求由一个动词\(方法\)和一个资源\(端点\)组成。 以下是常见的 HTTP 动词：

| 动词 | 描述 | \*幂等 | 安全性 | 可缓存 |
| :--- | :--- | :--- | :--- | :--- |
| GET | 读取资源 | Yes | Yes | Yes |
| POST | 创建资源或触发处理数据的进程 | No | No | Yes，如果回应包含刷新信息 |
| PUT | 创建或替换资源 | Yes | No | No |
| PATCH | 部分更新资源 | No | No | Yes，如果回应包含刷新信息 |
| DELETE | 删除资源 | Yes | No | No |

**多次执行不会产生不同的结果**。

HTTP 是依赖于较低级协议\(如**TCP**和**UDP\)**的应用层协议。

**来源及延伸阅读：HTTP**

* [README](https://www.quora.com/What-is-the-difference-between-HTTP-protocol-and-TCP-protocol)
* [HTTP是什么？](https://www.nginx.com/resources/glossary/http/)
* [HTTP和TCP的区别](https://www.quora.com/What-is-the-difference-between-HTTP-protocol-and-TCP-protocol)
* [PUT和PATCH的区别](https://laracasts.com/discuss/channels/general-discussion/whats-the-differences-between-put-and-patch?page=1)

#### 传输控制协议（TCP）

![](https://camo.githubusercontent.com/821620cf6aa83566f4def561e754e5991480ca8d/687474703a2f2f692e696d6775722e636f6d2f4a6441736476472e6a7067)

TCP是通过[IP网络](https://en.wikipedia.org/wiki/Internet_Protocol)的面向连接的协议。 使用[握手](https://en.wikipedia.org/wiki/Handshaking)建立和断开连接。 发送的所有数据包保证以原始顺序到达目的地，用以下措施保证数据包不被损坏：

* 每个数据包的序列号和[校验码](https://en.wikipedia.org/wiki/Transmission_Control_Protocol#Checksum_computation)。
* [确认包](https://en.wikipedia.org/wiki/Acknowledgement_%28data_networks%29)和自动重传

如果发送者没有收到正确的响应，它将重新发送数据包。如果多次超时，连接就会断开。TCP实行[流量控制](https://en.wikipedia.org/wiki/Flow_control_%28data%29)和[拥塞控制](https://en.wikipedia.org/wiki/Network_congestion#Congestion_control)。这些确保措施会导致延迟，而且通常导致传输效率比 UDP 低。

为了确保高吞吐量，Web服务器可以保持大量的TCP连接，从而导致高内存使用。在Web服务器线程间拥有大量开放连接可能开销巨大，消耗资源过多，也就是说，一个[memcached](https://github.com/ZhuangZhe/system-design-primer/blob/master/README-zh-Hans.md#memcached)服务器。[连接池](https://en.wikipedia.org/wiki/Connection_pool)可以帮助除了在适用的情况下切换到UDP。

TCP对于需要高可靠性但时间紧迫的应用程序很有用。比如包括Web服务器，数据库信息，SMTP，FTP和SSH。

以下情况使用TCP代替UDP：

* 你需要数据完好无损。
* 你想对网络吞吐量自动进行最佳评估。

#### 用户数据报协议（UDP）

![](https://camo.githubusercontent.com/47eb14c0a2dff2166f8781a6ce8c7f33d4c33da8/687474703a2f2f692e696d6775722e636f6d2f797a44724a74412e6a7067)

UDP是无连接的。数据报\(类似于数据包\)只在数据报级别有保证。数据报可能会无序的到达目的地，也有可能会遗失。UDP不支持拥塞控制。虽然不如TCP那样有保证，但UDP通常效率更高。

UDP可以通过广播将数据报发送至子网内的所有设备。这对[DHCP](https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol)很有用，因为子网内的设备还没有分配IP地址，而IP对于TCP是必须的。

UDP可靠性更低但适合用在网络电话、视频聊天，流媒体和实时多人游戏上。

以下情况使用UDP代替TCP：

* 你需要低延迟
* 相对于数据丢失更糟的是数据延迟
* 你想实现自己的错误校正方法

**来源及延伸阅读：TCP与UDP**

* [游戏编程的网络](http://gafferongames.com/networking-for-game-programmers/udp-vs-tcp/)
* [TCP与UDP的关键区别](http://www.cyberciti.biz/faq/key-differences-between-tcp-and-udp-protocols/)
* [TCP与UDP的不同](http://stackoverflow.com/questions/5970383/difference-between-tcp-and-udp)
* [传输控制协议](https://en.wikipedia.org/wiki/Transmission_Control_Protocol)
* [用户数据报协议](https://en.wikipedia.org/wiki/User_Datagram_Protocol)
* [Memcache在Facebook的扩展](http://www.cs.bu.edu/~jappavoo/jappavoo.github.com/451/papers/memcache-fb.pdf)

#### 远程过程调用协议\(RPC\)

![](https://camo.githubusercontent.com/1a3d7771c0b0a7816d0533fffeb6eeeb442d9945/687474703a2f2f692e696d6775722e636f6d2f6946344d6b62352e706e67)

在RPC中，客户端会去调用另一个地址空间\(通常是一个远程服务器\)里的方法。调用代码看起来就像是调用的是一个本地方法，客户端和服务器交互的具体过程被抽象。远程调用相对于本地调用一般较慢而且可靠性更差，因此区分两者是有帮助的。热门的RPC框架包括 [Protobuf](https://developers.google.com/protocol-buffers/)、[Thrift](https://thrift.apache.org/) 和 [Avro](https://avro.apache.org/docs/current/)。

RPC 是一个“请求-响应”协议：

* **客户端程序：**调用客户端存根程序。就像调用本地方法一样，参数会被压入栈中。
* **客户端stub程序：**将请求过程的id和参数打包进请求信息中。
* **客户端通信模块：**将信息从客户端发送至服务端。
* **服务端通信模块：**将接受的包传给服务端存根程序。
* **服务端stub程序：**将结果解包，依据过程id调用服务端方法并将参数传递过去。

RPC调用示例：

```text
GET /someoperation?data=anId

POST /anotheroperation
{
  "data":"anId";
  "anotherdata": "another value"
}
```

RPC专注于暴露方法。RPC通常用于处理内部通讯的性能问题，这样你可以手动处理本地调用以更好的适应你的情况。

当以下情况时选择本地库\(也就是 SDK\)：

* 你知道你的目标平台。
* 你想控制如何访问你的“逻辑”。
* 你想对发生在你的库中的错误进行控制。
* 性能和终端用户体验是你最关心的事。

遵循**REST**的HTTP API往往更适用于公共API。

**缺点：RPC**

* RPC客户端与服务实现捆绑地很紧密。
* 一个新的API必须在每一个操作或者用例中定义。
* RPC很难调试。
* 你可能没办法很方便的去修改现有的技术。举个例子，如果你希望在[Squid](http://www.squid-cache.org/)这样的缓存服务器上确保 [RPC被正确缓存](http://etherealbits.com/2012/12/debunking-the-myths-of-rpc-rest/)的话可能需要一些额外的努力了。

#### 表述性状态转移\(REST\)

REST是一种强制的客户端/服务端架构设计模型，客户端基于服务端管理的一系列资源操作。服务端提供修改或获取资源的接口。所有的通信必须是无状态和可缓存的。

**RESTful接口有四条规则：**

* **标志资源\(HTTP里的URI\)：**无论什么操作都使用同一个URI。
* **表示的改变\(HTTP的动作\)：**使用动作、headers和body。
* **可自我描述的错误信息\(HTTP中的status code\)：**使用状态码，不要重新造轮子。
* [**HATEOAS**](http://restcookbook.com/Basics/hateoas/)**\(HTTP中的HTML接口\)：**你的web服务器应该能够通过浏览器访问。

REST 请求的例子：

```text
GET /someresources/anId

PUT /someresources/anId
{"anotherdata": "another value"}
```

REST关注于暴露数据。它减少了客户端／服务端的耦合程度，经常用于公共HTTP API接口设计。REST使用更通常与规范化的方法来通过 URI 暴露资源，[通过header来表述](https://github.com/for-GET/know-your-http-well/blob/master/headers.md)并通过GET、POST、PUT、DELETE和PATCH这些动作来进行操作。因为无状态的特性，REST易于横向扩展和隔离。

**缺点：REST**

* 由于REST将重点放在暴露数据，所以当资源不是自然组织的或者结构复杂的时候它可能无法很好的适应。举个例子，返回过去一小时中与特定事件集匹配的更新记录这种操作就很难表示为路径。使用REST，可能会使用URI路径，查询参数和可能的请求体来实现。
* REST一般依赖几个动作\(GET、POST、PUT、DELETE 和 PATCH\)，但有时候仅仅这些没法满足你的需要。举个例子，将过期的文档移动到归档文件夹里去，这样的操作可能没法简单的用上面这几个verbs表达。
* 为了渲染单个页面，获取被嵌套在层级结构中的复杂资源需要客户端，服务器之间多次往返通信。例如，获取博客内容及其关联评论。对于使用不确定网络环境的移动应用来说，这些多次往返通信是非常麻烦的。
* 随着时间的推移，更多的字段可能会被添加到API响应中，较旧的客户端将会接收到所有新的数据字段，即使是那些它们不需要的字段，结果它会增加负载大小并引起更大的延迟。

#### RPC与REST比较

| 操作 | RPC | REST |
| :--- | :--- | :--- |
| 注册 | **POST** /signup | **POST** /persons |
| 注销 | **POST** /resign { "personid": "1234" } | **DELETE** /persons/1234 |
| 读取用户信息 | **GET** /readPerson?personid=1234 | **GET** /persons/1234 |
| 读取用户物品列表 | **GET** /readUsersItemsList?personid=1234 | **GET** /persons/1234/items |
| 向用户物品列表添加一项 | **POST** /addItemToUsersItemsList { "personid": "1234"; "itemid": "456" } | **POST** /persons/1234/items { "itemid": "456" } |
| 更新一个物品 | **POST** /modifyItem { "itemid": "456"; "key": "value" } | **PUT** /items/456 { "key": "value" } |
| 删除一个物品 | **POST** /removeItem { "itemid": "456" } | **DELETE** /items/456 |

**来源及延伸阅读：REST与RPC**

* [你真的知道你为什么更喜欢REST而不是RPC吗](https://apihandyman.io/do-you-really-know-why-you-prefer-rest-over-rpc/)
* [什么时候RPC比REST更合适？](http://programmers.stackexchange.com/a/181186)
* [REST vs JSON-RPC](http://stackoverflow.com/questions/15056878/rest-vs-json-rpc)
* [揭开RPC和REST的神秘面纱](http://etherealbits.com/2012/12/debunking-the-myths-of-rpc-rest/)
* [使用REST的缺点是什么](https://www.quora.com/What-are-the-drawbacks-of-using-RESTful-APIs)
* [破解系统设计面试](http://www.puncsky.com/blog/2016-02-13-crack-the-system-design-interview)
* [Thrift](https://code.facebook.com/posts/1468950976659943/)
* [为什么在内部使用REST而不是RPC](http://arstechnica.com/civis/viewtopic.php?t=1190508)

