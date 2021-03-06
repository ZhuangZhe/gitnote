# 缓存

![](../.gitbook/assets/image%20%2841%29.png)



缓存可以提高页面加载速度，并可以减少服务器和数据库的负载。在这个模型中，分发器先查看请求之前是否被响应过，如果有则将之前的结果直接返回，来省掉真正的处理。

数据库分片均匀分布的读取是最好的。但是热门数据会让读取分布不均匀，这样就会造成瓶颈，如果在数据库前加个缓存，就会抹平不均匀的负载和突发流量对数据库的影响。

## 客户端缓存

缓存可以位于客户端\(操作系统或者浏览器\)，[服务端](https://github.com/ZhuangZhe/system-design-primer/blob/master/README-zh-Hans.md#%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86web-%E6%9C%8D%E5%8A%A1%E5%99%A8)或者不同的缓存层。

## CDN 缓存

[CDN](https://github.com/ZhuangZhe/system-design-primer/blob/master/README-zh-Hans.md#%E5%86%85%E5%AE%B9%E5%88%86%E5%8F%91%E7%BD%91%E7%BB%9Ccdn)也被视为一种缓存。

## Web服务器缓存

[反向代理](https://github.com/ZhuangZhe/system-design-primer/blob/master/README-zh-Hans.md#%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86web-%E6%9C%8D%E5%8A%A1%E5%99%A8)和缓存\(比如[Varnish](https://www.varnish-cache.org/)\)可以直接提供静态和动态内容。Web服务器同样也可以缓存请求，返回相应结果而不必连接应用服务器。

## 数据库缓存

数据库的默认配置中通常包含缓存级别，针对一般用例进行了优化。调整配置，在不同情况下使用不同的模式可以进一步提高性能。

## 应用缓存

基于内存的缓存比如Memcached和Redis是应用程序和数据存储之间的一种键值存储。由于数据保存在RAM中，它比存储在磁盘上的典型数据库要快多了。RAM比磁盘限制更多，所以例如 [least recently used\(LRU\)](https://en.wikipedia.org/wiki/Cache_algorithms#Least_Recently_Used)的[缓存无效算法](https://en.wikipedia.org/wiki/Cache_algorithms)可以将热门数据放在RAM中，而对一些比较冷门的数据不做处理。

Redis有下列附加功能：

* 持久性选项
* 内置数据结构比如有序集合和列表

有多个缓存级别，分为两大类：**数据库查询**和**对象**：

* 行级别
* 查询级别
* 完整的可序列化对象
* 完全渲染的 HTML

一般来说，你应该尽量避免基于文件的缓存，因为这使得复制和自动缩放很困难。

## 数据库查询级别的缓存

当你查询数据库的时候，将查询语句的哈希值与查询结果存储到缓存中。这种方法会遇到以下问题：

* 很难用复杂的查询删除已缓存结果。
* 如果一条数据比如表中某条数据的一项被改变，则需要删除所有可能包含已更改项的缓存结果。

## 对象级别的缓存

将您的数据视为对象，就像对待你的应用代码一样。让应用程序将数据从数据库中组合到类实例或数据结构中：

* 如果对象的基础数据已经更改了，那么从缓存中删掉这个对象。
* 允许异步处理：workers 通过使用最新的缓存对象来组装对象。

建议缓存的内容：

* 用户会话
* 完全渲染的Web页面
* 活动流
* 用户图数据

## 何时更新缓存

由于你只能在缓存中存储有限的数据，所以你需要选择一个适用于你用例的缓存更新策略。

### **缓存模式**

![](https://camo.githubusercontent.com/7f5934e49a678b67f65e5ed53134bc258b007ebb/687474703a2f2f692e696d6775722e636f6d2f4f4e6a4f52716b2e706e67)

应用从存储器读写。缓存不和存储器直接交互，应用执行以下操作：

* 在缓存中查找记录，如果所需数据不在缓存中
* 从数据库中加载所需内容
* 将查找到的结果存储到缓存中
* 返回所需内容

```text
def get_user(self, user_id):
    user = cache.get("user.{0}", user_id)
    if user is None:
        user = db.query("SELECT * FROM users WHERE user_id = {0}", user_id)
        if user is not None:
            key = "user.{0}".format(user_id)
            cache.set(key, json.dumps(user))
    return user
```

[Memcached](https://memcached.org/)通常用这种方式使用。

添加到缓存中的数据读取速度很快。缓存模式也称为延迟加载。只缓存所请求的数据，这避免了没有被请求的数据占满了缓存空间。

**缓存的缺点：**

* 请求的数据如果不在缓存中就需要经过三个步骤来获取数据，这会导致明显的延迟。
* 如果数据库中的数据更新了会导致缓存中的数据过时。这个问题需要通过设置TTL强制更新缓存或者直写模式来缓解这种情况。
* 当一个节点出现故障的时候，它将会被一个新的节点替代，这增加了延迟的时间。

### **直写模式**

![](https://camo.githubusercontent.com/56b870f4d199335ccdbc98b989ef6511ed14f0e2/687474703a2f2f692e696d6775722e636f6d2f3076426330684e2e706e67)

应用使用缓存作为主要的数据存储，将数据读写到缓存中，而缓存负责从数据库中读写数据。

* 应用向缓存中添加/更新数据
* 缓存同步地写入数据存储
* 返回所需内容

应用代码：

```text
set_user(12345, {"foo":"bar"})
```

缓存代码：

```text
def set_user(user_id, values):
    user = db.query("UPDATE Users WHERE id = {0}", user_id, values)
    cache.set(user_id, user)
```

由于存写操作所以直写模式整体是一种很慢的操作，但是读取刚写入的数据很快。相比读取数据，用户通常比较能接受更新数据时速度较慢。缓存中的数据不会过时。

**直写模式的缺点：**

* 由于故障或者缩放而创建的新的节点，新的节点不会缓存，直到数据库更新为止。缓存应用直写模式可以缓解这个问题。
* 写入的大多数数据可能永远都不会被读取，用 TTL 可以最小化这种情况的出现。

### **回写模式**

![](https://camo.githubusercontent.com/8aa9f1a2f050c1422898bb5e82f1f01773334e22/687474703a2f2f692e696d6775722e636f6d2f72675372766a472e706e67)

在回写模式中，应用执行以下操作：

* 在缓存中增加或者更新条目
* 异步写入数据，提高写入性能。

**回写模式的缺点：**

* 缓存可能在其内容成功存储之前丢失数据。
* 执行直写模式比缓存或者回写模式更复杂。

### **刷新**

![](https://camo.githubusercontent.com/49dcb54307763b4f56d61a4a1369826e2e7d52e4/687474703a2f2f692e696d6775722e636f6d2f6b78746a7167452e706e67)

你可以将缓存配置成在到期之前自动刷新最近访问过的内容。

如果缓存可以准确预测将来可能请求哪些数据，那么刷新可能会导致延迟与读取时间的降低。

**刷新的缺点：**

* 不能准确预测到未来需要用到的数据可能会导致性能不如不使用刷新。

#### 缓存的缺点：

* 需要保持缓存和真实数据源之间的一致性，比如数据库根据[缓存无效](https://en.wikipedia.org/wiki/Cache_algorithms)。
* 需要改变应用程序比如增加 Redis 或者 memcached。
* 无效缓存是个难题，什么时候更新缓存是与之相关的复杂问题。

#### 相关资源和延伸阅读

* [从缓存到内存数据](http://www.slideshare.net/tmatyashovsky/from-cache-to-in-memory-data-grid-introduction-to-hazelcast)
* [可扩展系统设计模式](http://horicky.blogspot.com/2010/10/scalable-system-design-patterns.html)
* [可缩放系统构架介绍](http://lethain.com/introduction-to-architecting-systems-for-scale/)
* [可扩展性，可用性，稳定性和模式](http://www.slideshare.net/jboner/scalability-availability-stability-patterns/)
* [可扩展性](http://www.lecloud.net/post/9246290032/scalability-for-dummies-part-3-cache)
* [AWS ElastiCache 策略](http://docs.aws.amazon.com/AmazonElastiCache/latest/UserGuide/Strategies.html)
* [维基百科](https://en.wikipedia.org/wiki/Cache_%28computing%29)

