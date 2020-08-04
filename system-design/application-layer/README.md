# 应用层

![](https://camo.githubusercontent.com/feeb549c5b6e94f65c613635f7166dc26e0c7de7/687474703a2f2f692e696d6775722e636f6d2f7942355359776d2e706e67)

将Web服务层与应用层\(也被称作平台层\)分离，可以独立缩放和配置这两层。添加新的API只需要添加应用服务器，而不必添加额外的web服务器。

**单一职责原则**提倡小型的，自治的服务共同合作。小团队通过提供小型的服务，可以更激进地计划增长。

应用层中的工作进程也有可以实现[异步化](https://github.com/ZhuangZhe/system-design-primer/blob/master/README-zh-Hans.md#%E5%BC%82%E6%AD%A5)。

#### 微服务 <a id="wei-fu-wu"></a>

与此讨论相关的话题是[微服务](https://en.wikipedia.org/wiki/Microservices)，可以被描述为一系列可以独立部署的小型的，模块化服务。每个服务运行在一个独立的线程中，通过明确定义的轻量级机制通讯，共同实现业务目标。[1](https://smartbear.com/learn/api-design/what-are-microservices)​

例如，Pinterest 可能有这些微服务： 用户资料、关注者、Feed 流、搜索、照片上传等。

#### 服务发现 <a id="fu-wu-fa-xian"></a>

像[Consul](https://www.consul.io/docs/index.html)，[Etcd](https://coreos.com/etcd/docs/latest)和[Zookeeper](http://www.slideshare.net/sauravhaloi/introduction-to-apache-zookeeper)这样的系统可以通过追踪注册名、地址、端口等信息来帮助服务互相发现对方。[Health checks](https://www.consul.io/intro/getting-started/checks.html)可以帮助确认服务的完整性和是否经常使用一个[HTTP](https://github.com/ZhuangZhe/system-design-primer/blob/master/README-zh-Hans.md#%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AEhttp)路径。Consul和Etcd都有一个内建的[key-value存储](https://github.com/ZhuangZhe/system-design-primer/blob/master/README-zh-Hans.md#%E9%94%AE-%E5%80%BC%E5%AD%98%E5%82%A8)用来存储配置信息和其他的共享信息。

#### 不利之处：应用层 <a id="bu-li-zhi-chu-ying-yong-ceng"></a>

* 添加由多个松耦合服务组成的应用层，从架构、运营、流程等层面来讲将非常不同\(相对于单体系统\)。
* 微服务会增加部署和运营的复杂度。

**参考资料：**

* ​[可缩放系统构架介绍](http://lethain.com/introduction-to-architecting-systems-for-scale)​
* ​[破解系统设计面试](http://www.puncsky.com/blog/2016-02-13-crack-the-system-design-interview)​
* ​[面向服务架构](https://en.wikipedia.org/wiki/Service-oriented_architecture)​
* ​[Zookeeper 介绍](http://www.slideshare.net/sauravhaloi/introduction-to-apache-zookeeper)​
* ​[构建微服务，你所需要知道的一切](https://cloudncode.wordpress.com/2016/07/22/msa-getting-started/)​



