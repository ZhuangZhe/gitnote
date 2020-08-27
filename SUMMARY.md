# Table of contents

* [简介](README.md)
* [进度](progress.md)

## Java

* [JVM](java/untitled/README.md)
  * [JVM结构](java/untitled/jvm-structure.md)
  * [内存分配](java/untitled/memory-allocation.md)
  * [垃圾回收](java/untitled/garbage-collection.md)
  * [类实例化和类加载](java/untitled/class-loading.md)
* [反射](java/reflection/README.md)
  * [泛型](java/reflection/generic/README.md)
    * [ParameterizedType](java/reflection/generic/parameterizedtype.md)
    * [TypeVariable](java/reflection/generic/typevariable.md)
    * [WildcardType](java/reflection/generic/wildcardtype.md)
    * [GenericArrayType](java/reflection/generic/genericarraytype.md)
  * [动态代理](java/reflection/dynamic-proxy/README.md)
    * [Proxy类](java/reflection/dynamic-proxy/proxy-class.md)
    * [InvocationHandler接口](java/reflection/dynamic-proxy/invocationhandler-interface.md)
  * [反射调用的底层实现](java/reflection/reflection-underlying-implementation.md)
  * [优化反射调用性能](java/reflection/improve-reflection-performance.md)
  * [Class类](java/reflection/class-class.md)
  * [Constructor类](java/reflection/constructor-class.md)
  * [Method类](java/reflection/method-class.md)
  * [Field类](java/reflection/field-class.md)
  * [Parameter类](java/reflection/parameter-class.md)
  * [AccessibleObject类](java/reflection/accessibleobject-class.md)
  * [Modifier类](java/reflection/modifier-class.md)
  * [AnnotatedElement接口](java/reflection/annotatedelement-interface.md)
  * [GenericDeclaration接口](java/reflection/genericdeclaration-interface.md)
  * [Type接口](java/reflection/type-interface.md)
  * [Member接口](java/reflection/member-interface.md)
  * [Executable抽象类](java/reflection/executable-abstract-class.md)
* [注解](java/annotation/README.md)
  * [基本语法](java/annotation/basic.md)
  * [Annotation类](java/annotation/annotation-class.md)
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
* [IO](java/io/README.md)
  * [BIO](java/io/bio.md)
  * [NIO](java/io/nio.md)
  * [AIO](java/io/aio.md)
  * [序列化](java/io/serialization.md)
  * [网络编程](java/io/network-programming.md)
  * [工具类](java/io/tools.md)
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
  * [API](java/concurrency/api.md)

## Spring

* [Spring Core](spring/spring-core/README.md)
  * [IoC](spring/spring-core/ioc/README.md)
    * [Spring的启动过程](spring/spring-core/ioc/spring-start-up.md)
    * [Bean](spring/spring-core/ioc/bean/README.md)
      * [Bean生命周期](spring/spring-core/ioc/bean/bean-lifecycle.md)
      * [IOC依赖注入](spring/spring-core/ioc/bean/ioc-di.md)
      * [Bean作用域](spring/spring-core/ioc/bean/bean-scope.md)
      * [BeanFactory](spring/spring-core/ioc/bean/beanfactory.md)
  * [配置](spring/spring-core/config/README.md)
    * [Java Config](spring/spring-core/config/java-config.md)
* [Spring Boot](spring/spring-boot/README.md)
  * [启动过程](spring/spring-boot/initialization.md)
  * [自动装配](spring/spring-boot/auto-config.md)
  * [配置加载](spring/spring-boot/config-loading.md)
  * [监视器](spring/spring-boot/listener.md)
  * [Starter](spring/spring-boot/starter.md)
* [Spring Web](spring/spring-web.md)
* [Spring Data](spring/spring-data/README.md)
  * [DAO DTO VO](spring/spring-data/dao-dto-vo.md)
* [Spring AOP](spring/spring-aop.md)
* [Spring Test](spring/spring-test.md)
* [Spring Cloud](spring/spring-cloud.md)
* [Spring Security](spring/spring-security.md)
* [常用注解](spring/annotation.md)

## 数据库 <a id="database"></a>

* [SQL](database/sql/README.md)
  * [MySQL](database/sql/mysql/README.md)
    * [存储引擎](database/sql/mysql/storage-engine.md)
    * [索引与B+Tree](database/sql/mysql/index-and-b+tree.md)
    * [事务](database/sql/mysql/transaction.md)
    * [数据库事务隔离级别](database/sql/mysql/transaction-isolation-level.md)
    * [MVCC](database/sql/mysql/mvcc.md)
* [NoSQL](database/nosql/README.md)
  * [Redis](database/nosql/redis/README.md)
    * [基本数据类型](database/nosql/redis/basic-data-types.md)
    * [跳跃表](database/nosql/redis/skiplist.md)
    * [命令](database/nosql/redis/commands.md)
    * [配置文件](database/nosql/redis/config-file.md)
  * [Cassandra](database/nosql/cassandra.md)
  * [MongoDB](database/nosql/mongodb/README.md)
    * [基本概念](database/nosql/mongodb/intro.md)

## 搜索引擎 <a id="search-engine"></a>

* [Elasticsearch](search-engine/elasticsearch.md)

## 消息队列 <a id="message-queue"></a>

* [概述](message-queue/intro.md)
* [Kafka](message-queue/kafka/README.md)
  * [概述](message-queue/kafka/intro.md)

## RPC

* [dubbo](rpc/dubbo.md)
* [Zookeeper](rpc/zookeeper.md)

## 网络 <a id="network"></a>

* [互联网组成](network/internet-components.md)
* [OSI模型](network/osi-model.md)

## 反向代理 <a id="reverse-proxy"></a>

* [NGINX](reverse-proxy/nginx.md)

## 操作系统 <a id="operation-system"></a>

* [基础](operation-system/basic.md)
* [Linux](operation-system/linux.md)
* [命令](operation-system/commands.md)

## 部署 <a id="deploy"></a>

* [Docker](deploy/docker.md)
* [kubernetes](deploy/kubernetes.md)

## 系统设计 <a id="system-design"></a>

* [权衡与取舍](system-design/trade-off.md)
* [域名系统](system-design/dns.md)
* [CDN](system-design/cdn/README.md)
  * [CDN的基本工作过程](system-design/cdn/cdn-basic.md)
  * [CDN的功能架构](system-design/cdn/cdn-function-framework.md)
  * [CDN的部署架构](system-design/cdn/cdn-deploy-framework.md)
  * [使用CDN的原因](system-design/cdn/why-use-cdn.md)
* [负载均衡器](system-design/loadbalancer/README.md)
  * [四和七层负载均衡](system-design/loadbalancer/layer-four-and-seven-load-balancer.md)
* [反向代理](system-design/reverse-proxy.md)
* [应用层](system-design/application-layer/README.md)
  * [微服务](system-design/application-layer/microservice.md)
  * [服务发现](system-design/application-layer/fu-wu-fa-xian.md)
* [数据库](system-design/database.md)
* [缓存](system-design/cache.md)
* [异步](system-design/asynchronous.md)
* [网络](system-design/network.md)
* [设计练习](system-design/practice.md)

## 算法和数据结构 <a id="algorithm-and-data-structure"></a>

* [树](algorithm-and-data-structure/tree.md)
* [图](algorithm-and-data-structure/graph/README.md)
  * [相关算法](algorithm-and-data-structure/graph/algorithm/README.md)
    * [Prim算法](algorithm-and-data-structure/graph/algorithm/prim.md)
    * [Kruskal算法](algorithm-and-data-structure/graph/algorithm/kruskal.md)
    * [Dijkstra算法](algorithm-and-data-structure/graph/algorithm/dijkstra.md)
    * [Tarjan算法](algorithm-and-data-structure/graph/algorithm/tarjan.md)
    * [Union-Find算法](algorithm-and-data-structure/graph/algorithm/unionfind.md)
    * [Topological Sort算法](algorithm-and-data-structure/graph/algorithm/topological-sort.md)
  * [算法题](algorithm-and-data-structure/graph/algorithm-problem/README.md)
    * [Sort Integers by The Power Value](algorithm-and-data-structure/graph/algorithm-problem/leetcode-1387-sort-integers-by-the-power-value.md)
    * [Minimum Number of Vertices to Reach All Nodes](algorithm-and-data-structure/graph/algorithm-problem/leetcode1557-minimum-number-of-vertices-to-reach-all-nodes.md)
    * [Regions Cut By Slashes](algorithm-and-data-structure/graph/algorithm-problem/leetcode-959-regions-cut-by-slashes.md)
    * [Partition Array for Maximum Sum](algorithm-and-data-structure/graph/algorithm-problem/leetcode-1043-partition-array-for-maximum-sum.md)
    * [Keys and Rooms](algorithm-and-data-structure/graph/algorithm-problem/keys-and-rooms.md)
    * [Jump Game III](algorithm-and-data-structure/graph/algorithm-problem/jump-game-iii.md)

## 设计模式

* [行为型](she-ji-mo-shi/behavioral/README.md)
  * [命令模式](she-ji-mo-shi/behavioral/command-pattern.md)
  * [中介者模式](she-ji-mo-shi/behavioral/mediator-pattern.md)
  * [观察者模式](she-ji-mo-shi/behavioral/observer-pattern.md)
  * [状态模式](she-ji-mo-shi/behavioral/state-pattern.md)
  * [策略模式](she-ji-mo-shi/behavioral/strategy-pattern.md)
* [结构型](she-ji-mo-shi/structural/README.md)
  * [适配器模式](she-ji-mo-shi/structural/adapter.md)
  * [桥接模式](she-ji-mo-shi/structural/bridge-pattern.md)
  * [装饰模式](she-ji-mo-shi/structural/decorator-pattern.md)
  * [外观模式](she-ji-mo-shi/structural/facade-pattern.md)
  * [享元模式](she-ji-mo-shi/structural/flyweight-pattern.md)
  * [代理模式](she-ji-mo-shi/structural/proxy-pattern.md)
* [创建型](she-ji-mo-shi/creational/README.md)
  * [工厂模式](she-ji-mo-shi/creational/factory-pattern.md)
  * [抽象工厂模式](she-ji-mo-shi/creational/abstract-factory-pattern.md)
  * [建造者模式](she-ji-mo-shi/creational/builder-pattern.md)
  * [单例模式](she-ji-mo-shi/creational/singleton.md)

## 学习资源

* [学习资源](xue-xi-zi-yuan/learning-resource.md)

