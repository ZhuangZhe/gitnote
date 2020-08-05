# Table of contents

* [Initial page](README.md)

## Java

* [JVM](java/untitled/README.md)
  * [JVM结构](java/untitled/jvm-structure.md)
  * [内存分配](java/untitled/memory-allocation.md)
  * [垃圾回收](java/untitled/garbage-collection.md)
  * [类实例化和类加载](java/untitled/class-loading.md)
* [反射](java/reflection.md)
* [注解](java/annotation.md)
* [集合](java/collection/README.md)
  * [Iterable接口](java/collection/iterable-interface/README.md)
    * [Collection接口](java/collection/iterable-interface/collection-interface/README.md)
      * [List接口](java/collection/iterable-interface/collection-interface/list-jie-kou/README.md)
        * [ArrayList](java/collection/iterable-interface/collection-interface/list-jie-kou/arraylist.md)
        * [Vector](java/collection/iterable-interface/collection-interface/list-jie-kou/vector.md)
        * [LinkedList](java/collection/iterable-interface/collection-interface/list-jie-kou/linkedlist.md)
      * [Queue接口](java/collection/iterable-interface/collection-interface/queue-jie-kou/README.md)
        * [PriorityQueue](java/collection/iterable-interface/collection-interface/queue-jie-kou/priorityqueue.md)
      * [Dequeue接口](java/collection/iterable-interface/collection-interface/dequeue-interface/README.md)
        * [ArrayDeque](java/collection/iterable-interface/collection-interface/dequeue-interface/arraydeque.md)
      * [Set接口](java/collection/iterable-interface/collection-interface/set-interface/README.md)
        * [HashSet](java/collection/iterable-interface/collection-interface/set-interface/hashset.md)
        * [TreeSet](java/collection/iterable-interface/collection-interface/set-interface/treeset.md)
    * [Iterator接口](java/collection/iterable-interface/iterator-interface.md)
  * [Map接口](java/collection/map-interface/README.md)
    * [HashMap](java/collection/map-interface/hashmap.md)
    * [TreeMap](java/collection/map-interface/treemap.md)
* [IO](java/io.md)
* [并发](java/concurrency/README.md)
  * [概述](java/concurrency/intro.md)
  * [进程与线程](java/concurrency/process-and-thread.md)
  * [线程状态转换以及基本操作](java/concurrency/thread-status-and-basic-operation.md)
  * [Java内存模型](java/concurrency/jmm.md)
  * [重排序](java/concurrency/reordering.md)
  * [happens-before规则](java/concurrency/happens-before.md)
  * [synchronized关键字](java/concurrency/synchronized.md)
  * [volatile关键字](java/concurrency/volatile-guan-jian-zi.md)
  * [final关键字](java/concurrency/final-guan-jian-zi.md)
  * [原子性、可见性以及有序性](java/concurrency/atomic-visibility-ordering.md)
  * [J.U.C](java/concurrency/juc/README.md)
    * [Lock接口](java/concurrency/juc/lock-interface.md)
    * [AbstractQueuedSynchronizer抽象类](java/concurrency/juc/abstractqueuedsynchronizer-abstract-class.md)
    * [ReentrantLock类](java/concurrency/juc/reentrantlock-class.md)
    * [ReentrantReadWriteLock类](java/concurrency/juc/reentrantreadwritelock-class.md)
    * [Condition接口](java/concurrency/juc/condition-interface.md)

## Spring

* [Spring Core](spring/spring-core/README.md)
  * [IoC](spring/spring-core/ioc/README.md)
    * [Spring的启动过程](spring/spring-core/ioc/spring-start-up.md)
    * [Bean](spring/spring-core/ioc/bean/README.md)
      * [Bean生命周期](spring/spring-core/ioc/bean/bean-lifecycle.md)
      * [IOC依赖注入](spring/spring-core/ioc/bean/ioc-di.md)
      * [Bean作用域](spring/spring-core/ioc/bean/bean-scope.md)
  * [MVC](spring/spring-core/untitled.md)
  * [AOP](spring/spring-core/aop.md)
* [Spring Boot](spring/spring-boot.md)
* [Spring Data](spring/spring-data.md)
* [Spring Cloud](spring/spring-cloud.md)
* [Spring Security](spring/spring-security.md)

## 数据库 <a id="database"></a>

* [SQL](database/sql.md)
* [MySQL](database/mysql.md)
* [MongoDB](database/mongodb/README.md)
  * [基本概念](database/mongodb/intro.md)
* [Redis](database/redis.md)
* [Cassandra](database/cassandra.md)

## 搜索引擎 <a id="search-engine"></a>

* [Elasticsearch](search-engine/elasticsearch.md)

## 消息队列 <a id="message-queue"></a>

* [概述](message-queue/intro.md)
* [Kafka](message-queue/kafka/README.md)
  * [概述](message-queue/kafka/intro.md)

## RPC

* [dubbo](rpc/dubbo.md)

## 网络 <a id="network"></a>

* [互联网组成](network/internet-components.md)
* [OSI模型](network/osi-model.md)

## 系统设计 <a id="system-design"></a>

* [CDN](system-design/cdn/README.md)
  * [CDN的基本工作过程](system-design/cdn/cdn-basic.md)
  * [CDN的功能架构](system-design/cdn/cdn-function-framework.md)
  * [CDN的部署架构](system-design/cdn/cdn-deploy-framework.md)
  * [使用CDN的原因](system-design/cdn/why-use-cdn.md)
* [负载均衡器](system-design/loadbalancer.md)
* [反向代理](system-design/reverse-proxy.md)
* [应用层](system-design/application-layer/README.md)
  * [微服务](system-design/application-layer/microservice.md)
* [数据库](system-design/database.md)
* [缓存](system-design/cache.md)
* [异步](system-design/asynchronous.md)
* [网络](system-design/network.md)
* [设计练习](system-design/practice.md)

## 算法和数据结构 <a id="algorithm-and-data-structure"></a>

* [树](algorithm-and-data-structure/tree.md)

