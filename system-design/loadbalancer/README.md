# 负载均衡器

负载均衡器将传入的请求分发到应用服务器和数据库等计算资源。无论哪种情况，负载均衡器将从计算资源来的响应返回给恰当的客户端。负载均衡器的效用在于:

* 防止请求进入不好的服务器
* 防止资源过载
* 帮助消除单一的故障点

负载均衡器可以通过硬件\(昂贵\)或HAProxy等软件来实现。 增加的好处包括:

* **SSL终结：**解密传入的请求并加密服务器响应，这样的话后端服务器就不必再执行这些潜在高消耗运算了。
  * 不需要再每台服务器上安装 [X.509 证书](https://en.wikipedia.org/wiki/X.509)。
* **Session留存：**如果 Web 应用程序不追踪会话，发出cookie并将特定客户端的请求路由到同一实例。

通常会设置采用[工作-备用](https://github.com/ZhuangZhe/system-design-primer/blob/master/README-zh-Hans.md#%E5%B7%A5%E4%BD%9C%E5%88%B0%E5%A4%87%E7%94%A8%E5%88%87%E6%8D%A2active-passive)或[双工作](https://github.com/ZhuangZhe/system-design-primer/blob/master/README-zh-Hans.md#%E5%8F%8C%E5%B7%A5%E4%BD%9C%E5%88%87%E6%8D%A2active-active)模式的多个负载均衡器，以免发生故障。

负载均衡器能基于多种方式来路由流量:

* 随机
* 最少负载
* Session/cookie
* [轮询调度或加权轮询调度算法](http://g33kinfo.com/info/archives/2657)
* [四层负载均衡](https://github.com/ZhuangZhe/system-design-primer/blob/master/README-zh-Hans.md#%E5%9B%9B%E5%B1%82%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1)
* [七层负载均衡](https://github.com/ZhuangZhe/system-design-primer/blob/master/README-zh-Hans.md#%E4%B8%83%E5%B1%82%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1)

#### 四层负载均衡

四层负载均衡根据监看[传输层](https://github.com/ZhuangZhe/system-design-primer/blob/master/README-zh-Hans.md#%E9%80%9A%E8%AE%AF)的信息来决定如何分发请求。通常，这会涉及来源，目标IP地址和请求头中的端口，但不包括数据包\(报文\)内容。四层负载均衡执行[网络地址转换\(NAT\)](https://www.nginx.com/resources/glossary/layer-4-load-balancing/)来向上游服务器转发网络数据包。

#### 七层负载均衡器

七层负载均衡器根据监控[应用层](https://github.com/ZhuangZhe/system-design-primer/blob/master/README-zh-Hans.md#%E9%80%9A%E8%AE%AF)来决定怎样分发请求。这会涉及请求头的内容，消息和cookie。七层负载均衡器终结网络流量，读取消息，做出负载均衡判定，然后传送给特定服务器。比如，一个七层负载均衡器能直接将视频流量连接到托管视频的服务器，同时将更敏感的用户账单流量引导到安全性更强的服务器。

以损失灵活性为代价，四层负载均衡比七层负载均衡花费更少时间和计算资源，虽然这对现代商用硬件的性能影响甚微。

#### 水平扩展

负载均衡器还能帮助水平扩展，提高性能和可用性。使用商业硬件的性价比更高，并且比在单台硬件上**垂直扩展**更贵的硬件具有更高的可用性。相比招聘特定企业系统人才，招聘商业硬件方面的人才更加容易。

**缺陷：水平扩展**

* 水平扩展引入了复杂度并涉及服务器复制
  * 服务器应该是无状态的：它们也不该包含像session或资料图片等与用户关联的数据。
  * session可以集中存储在数据库或持久化[缓存](https://github.com/ZhuangZhe/system-design-primer/blob/master/README-zh-Hans.md#%E7%BC%93%E5%AD%98)\(Redis、Memcached\)的数据存储区中。
* 缓存和数据库等下游服务器需要随着上游服务器进行扩展，以处理更多的并发连接。

#### 缺陷：负载均衡器

* 如果没有足够的资源配置或配置错误，负载均衡器会变成一个性能瓶颈。
* 引入负载均衡器以帮助消除单点故障但导致了额外的复杂性。
* 单个负载均衡器会导致单点故障，但配置多个负载均衡器会进一步增加复杂性。

