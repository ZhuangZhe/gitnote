# 微服务

**微服务** \(Microservices\) 是一种[软件架构风格](https://zh.wikipedia.org/wiki/%E8%BD%AF%E4%BB%B6%E6%9E%B6%E6%9E%84)，它是以专注于单一责任与功能的小型功能区块为基础，利用模块化的方式组合出复杂的大型应用程序，各功能区块使用与语言无关的API集相互通信。

2014年，Martin Fowler与James Lewis共同提出了微服务的概念，定义了微服务是由以单一应用程序构成的小服务，自己拥有自己的行程与轻量化处理，服务依业务功能设计，以全自动的方式部署，与其他服务使用HTTP API通信。同时服务会使用最小的规模的集中管理\(例如 Docker\)能力，服务可以用不同的编程语言与数据库等组件实现。

## 概念

微服务是一种以业务功能为主的服务设计概念，每一个服务都具有自主运行的业务功能，对外开放不受语言限制的 API \(最常用的是 HTTP\)，应用程序则是由一个或多个微服务组成。

微服务的另一个对比是[**单体式应用程序**](https://zh.wikipedia.org/wiki/%E5%96%AE%E9%AB%94%E5%BC%8F%E6%87%89%E7%94%A8%E7%A8%8B%E5%BC%8F)。单体式应用表示一个应用程序内包含了所有需要的业务功能，并且使用像[主从式架构](https://zh.wikipedia.org/wiki/%E4%B8%BB%E5%BE%9E%E5%BC%8F%E6%9E%B6%E6%A7%8B)或是[多层次架构](https://zh.wikipedia.org/w/index.php?title=%E5%A4%9A%E5%B1%A4%E6%AC%A1%E6%9E%B6%E6%A7%8B&action=edit&redlink=1)实现，虽然它也是能以分布式应用程序来实现，但是在单体式应用内，每一个业务功能是不可分割的。若要对单体式应用进行扩展则必须将**整个**应用程序都放到新的运算资源\(如：虚拟机\)内，但事实上应用程序中最吃资源、需要运算资源的仅有某个业务部分\(例如跑分析报表或是数学算法分析\)，但因为单体式应用无法分割该部分，因此无形中会有大量的资源浪费的现象。

微服务运用了以业务功能的设计概念，应用程序在设计时就能先以业务功能或流程设计先行分割，将各个业务功能都独立实现成一个能自主运行的个体服务，然后再利用相同的协议将所有应用程序需要的服务都组合起来，形成一个应用程序。若需要针对特定业务功能进行扩展时，只要对该业务功能的服务进行扩展就好，不需要整个应用程序都扩展，同时，由于微服务是以业务功能导向的实现，因此不会受到应用程序的干扰，微服务的管理员可以视运算资源的需要来配置微服务到不同的运算资源内，或是布建新的运算资源并将它配置进去。

虽然使用一般的服务器虚拟化技术就能应用于微服务的管理，但容器技术如Docker会更加地适合发展微服务的运算资源管理技术。

## 规划

微服务的规划与**单体式应用程序**十分不同，微服务中每个服务都需要避免与其他服务有所牵连，且都要能够自主，并在其他服务发生错误时不受干扰。

### 数据库

微服务理念中有数个数据库的规划方式。

* 每个服务都各有一个数据库，同属性的服务可共享同个数据库。
* 所有服务都共享同个数据库，但是不同表格，并且不会跨域访问。
* 每个服务都有自己的数据库，就算是同属性的服务也是，数据库并不会共享。

数据库并不会只存放该服务的资料，而是“**该服务所会用到的所有资料**”。更深层一点的举例：假设有个文章服务，而这个服务可能会需要判断用户的账号等。那么文章服务的数据库就可以放入用户的部分资料。此举是为了避免服务之间的相依性，避免文章服务调用用户服务。

![&#x5FAE;&#x670D;&#x52A1;&#x4E2D;&#x6BCF;&#x4E2A;&#x670D;&#x52A1;&#x90FD;&#x80FD;&#x591F;&#x6709;&#x81EA;&#x5DF1;&#x7684;&#x6570;&#x636E;&#x5E93;&#x3002;](../../.gitbook/assets/image%20%2890%29.png)

![&#x5982;&#x679C;&#x6570;&#x636E;&#x5E93;&#x90FD;&#x662F;&#x5206;&#x5F00;&#x7684;&#xFF0C;&#x90A3;&#x4E48;&#x65B0;&#x670D;&#x52A1;&#x4E0A;&#x7EBF;&#x65F6;&#x5C31;&#x4F1A;&#x9047;&#x5230;&#x6570;&#x636E;&#x5E93;&#x4E3A;&#x7A7A;&#x7684;&#x7A98;&#x5883;&#x3002;&#x6211;&#x4EEC;&#x5E76;&#x4E0D;&#x80FD;&#x4ECE;&#x53E6;&#x4E00;&#x4E2A;&#x670D;&#x52A1;&#x590D;&#x5236;&#x8D44;&#x6599;&#x8FC7;&#x6765;&#xFF0C;&#x56E0;&#x4E3A;&#x6211;&#x4EEC;&#x65E0;&#x6CD5;&#x786E;&#x5B9A;&#x8BE5;&#x670D;&#x52A1;&#x62E5;&#x6709;&#x6700;&#x65B0;&#x7684;&#x8D44;&#x6599;&#x3002; &#x6B64;&#x65F6;&#x5E94;&#x8BE5;&#x4ECE;&#x4E8B;&#x4EF6;&#x5B58;&#x50A8;&#x4E2D;&#x5FC3;&#x91CD;&#x64AD;&#x6240;&#x6709;&#x4E8B;&#x4EF6;&#xFF0C;&#x5982;&#x6B64;&#x4E00;&#x6765;&#x5C31;&#x53EF;&#x4EE5;&#x627E;&#x56DE;&#x5148;&#x524D;&#x7684;&#x6240;&#x6709;&#x3001;&#x6700;&#x65B0;&#x7684;&#x8D44;&#x6599;&#x3002;](../../.gitbook/assets/image%20%2892%29.png)

#### **数据库的可弃性**

实践微服务有许多的做法，但其中一种做法是将数据库作为**短期**的存储空间而不是存储长期的资料。这意味着数据库可以在离线时被清空。因为它们可以在上线时从事件存储中心恢复，因此也能以存储器缓存\(如：[Redis](https://zh.wikipedia.org/wiki/Redis)\) 作为数据库服务器。但这种做法需要将每个请求当作事件来进行广播。如此一来就可以从事件存储中心重播所有的事件来找回所有的资料。

### 沟通与事件广播

微服务中最重要的就是每个服务的独立与自主，因此服务与服务之间也**不应该**有所沟通。倘若真有沟通，也应采用异步沟通的方式来避免紧密的相依性问题。要达到此目的，则可用下列两种方式：

#### **事件存储中心**

这可以让你在服务集群中广播事件，并且在每个服务中监听这些事件并作处理，这令服务之间没有紧密的相依性，而这些发生的事件都会被保存在事件存储中心里。这意味着当微服务重新上线、部署时可以重播所有的事件。这也造就了微服务的数据库随时都可以被删除、摧毁，且不需要从其他服务中获取资料。

#### **消息队列**

这令你能够在服务集群中广播消息，并传递到每个服务中。具有这个功能的像是NSQ或是 [RabbitMQ](https://zh.wikipedia.org/wiki/RabbitMQ)。你能够在 A 服务上广播一个“创建新用户”的事件，这个事件可以顺便**带有**新用户的资料。而 B 服务可以**监听**这个事件并在接收到之后有所处理。这些过程都是异步处理的，这意味着 A 服务并不需要等到 B 服务处理完该事件后才能继续，而这也代表 A 服务无法获取 B 服务的处理结果。与事件存储中心近乎相似，但有所不同的是：消息队列并**不会**保存事件。一旦事件被消化\(接收**\)**后就会从队列中消失，这很适合用在像发送欢迎信件的时机。

![NSQ &#x662F;&#x4E00;&#x4E2A;&#x6D88;&#x606F;&#x961F;&#x5217;&#x7CFB;&#x7EDF;&#x3001;&#x5E73;&#x53F0;&#x3002;&#x5728;&#x5FAE;&#x670D;&#x52A1;&#x4E2D;&#x6240;&#x626E;&#x6F14;&#x7684;&#x89D2;&#x8272;&#x662F;&#x5C06;&#x6D88;&#x606F;&#x3001;&#x8D44;&#x6599;&#x4F20;&#x9012;&#x5230;&#x5176;&#x4ED6;&#x670D;&#x52A1;&#x3002; &#x6B64;&#x4E3E;&#x662F;&#x5F02;&#x6B65;&#x8FD0;&#x884C;&#xFF0C;&#x6240;&#x4EE5;&#x4E0D;&#x9700;&#x8981;&#x7B49;&#x5230;&#x5176;&#x4ED6;&#x670D;&#x52A1;&#x63A5;&#x6536;&#x5230;&#x6D88;&#x606F;&#x5C31;&#x80FD;&#x591F;&#x8FD0;&#x884C;&#x4E0B;&#x4E00;&#x6B65;&#x3002;&#x8FD9;&#x79CD;&#x65B9;&#x5F0F;&#x80FD;&#x591F;&#x907F;&#x514D;&#x670D;&#x52A1;&#x4E4B;&#x95F4;&#x6709;&#x6240;&#x7275;&#x8FDE;&#x3001;&#x8C03;&#x7528;&#x3002;](../../.gitbook/assets/image%20%2893%29.png)

### 服务探索

单个微服务在上线的时候，会向服务探索中心\(如：Consul\)注册自己的 IP 位置、服务内容，如此一来就不需要向每个微服务表明自己的IP位置，也就不用替每个微服务单独设置。当服务需要调用另一个服务的时候，会去询问服务探索中心该服务的IP位置为何，得到位置后即可直接向目标服务调用。

这么做的用意是可以统一集中所有服务的位置，就不会分散于每个微服务中，且服务探索中心可以每隔一段时间就向微服务进行健康检查\(如透过：TCP调用、HTTP调用、Ping\)，倘若该服务在时间内没有回应，则将其从服务中心移除，避免其他微服务对一个无回应的服务进行调用。

## 内容

一个微服务框架的应用程序有下列特性：

* 每个服务都容易被取代。
* 服务是以能力来组织的，例如用户界面、前端、推荐系统、账单或是物流等。
* 由于功能被拆成多个服务，因此可以由不同的编程语言、数据库实现。
* 架构是对称而非分层\(即生产者与消费者的关系\)。

一个微服务框架：

* 适用于具持续交付\(Continuous Delivery\)的软件开发流程。
* 与[服务导向架构](https://zh.wikipedia.org/wiki/%E6%9C%8D%E5%8B%99%E5%B0%8E%E5%90%91%E6%9E%B6%E6%A7%8B)\(Service-Oriented Architecture\)不同，后者是集成各种业务的应用程序，但微服务只属于一个应用程序。

## 技术

微服务可以用不同的编程语言实现，也可以使用不同的基础设施。因此，最重要的技术选择是微服务之间的通信方式（同步、异步、UI集成）以及用于通信的协议\(RESTful HTTP、消息、GraphQL等\)。在传统系统中，大多数技术选择，如编程语言，都会影响整个系统。因此，选择技术的方法是完全不同的。

### Service mesh

在服务网格中，每个服务实例都与一个反向代理服务器实例\(称为服务代理、 sidecar代理或sidecar\)配对。服务实例和sidecar代理共享一个容器，容器由一个容器编排工具\(如[Kubernetes](https://zh.wikipedia.org/wiki/Kubernetes)、[Nomad](https://zh.wikipedia.org/w/index.php?title=Nomad_%28software%29&action=edit&redlink=1)、[Docker Swarm](https://zh.wikipedia.org/wiki/Docker)、[DC/OS](https://zh.wikipedia.org/w/index.php?title=Mesosphere,_Inc.&action=edit&redlink=1)\)管理。 服务代理负责与其他服务实例的通信，并支持服务\(实例\)发现、负载平衡、身份验证和授权、安全通信等功能。

在服务网格中，服务实例及其sidecar代理被称为构成数据平面，其中不仅包括数据管理，还包括请求处理和响应。服务网格还包括一个用于管理服务之间交互的控制平面，这些交互由它们的sidecar代理协调。服务网格架构有几个选项: [Istio](https://zh.wikipedia.org/w/index.php?title=Istio&action=edit&redlink=1)、[Linkerd](https://zh.wikipedia.org/w/index.php?title=Cloud_Native_Computing_Foundation&action=edit&redlink=1)、[Consul](https://zh.wikipedia.org/w/index.php?title=Consul_%28software%29&action=edit&redlink=1)和其他许多服务网格景观。服务网格管理平面[Meshery](https://meshery.io/)提供跨服务网格部署的生命周期、配置和性能管理。

### 平台比较

实现微服务体系结构非常困难。任何微服务体系结构都需要解决许多问题。Netflix开发了一个微服务框架来支持他们的内部应用程序，然后开放了该框架的许多部分。其中许多任务具已经通过Spring框架得到推广——它们已经在Spring Cloud项目的保护伞下重新实现为基于Spring的工具。下表显示了Kubernetes生态系统中的实现功能与Spring Cloud世界中的等效功能的比较。Spring Cloud生态系统值得注意的一点是，它们都是基于Java的技术，而Kubernetes是一个多语言运行时平台。

| 微服务 | Spring Cloud与Netflix OSS | Kubernetes |
| :--- | :--- | :--- |
| 配置管理：微服务应用程序的配置需要从代码中进行外部化，并可以通过简单的服务调用进行检索。 | Spring Config Server、Netflix Archaius都支持基于Git存储库的配置位置。Archaius支持配置数据类型。 | Kubernetes ConfigMaps通过服务公开存储在etcd中的配置。 Kubernetes Secrets支持基于服务的安全部署和敏感配置信息\(例如密码，证书等的使用。 |
| 服务发现：维护微服务域中可用于工作的服务实例列表。 | Spring Cloud Eureka允许客户端向其注册，与注册的客户端保持心跳，并将服务名称映射到按服务名称查找服务的客户端的主机名。 | Kubernetes Services提供集群内部可用的服务实例的部署时注册。Ingress是一种机制，通过这种机制，服务可以向集群之外的客户端公开。 |
| 负载平衡：扩展分布式系统的关键是能够运行一个组件的多个实例。然后要通过负载均衡器将负载分配到这些实例上。 | Spring Cloud Ribbon为服务客户端提供跨服务实例负载平衡的能力。 | Kubernetes Service提供了跨服务实例对服务进行负载平衡的能力。这与Ribbon提供的功能不同。 |
| API网关：微服务提供的API的粒度通常与服务客户端需要的粒度不同。API网关实现表层，并提供其他服务，如代理、协议转换和其他管理功能。 | Spring Cloud Zuul提供基于配置的API表层 | Kubernetes Service和Ingress resources、Istio、Ambassador是提供南北（进出数据中心）和东西（跨数据中心或云或地区的通信）API网关功能的解决方案。 |
| 安全问题：许多安全问题推给API网关来实现。对于分布式微服务应用程序，不重新造安全方面的轮子，允许在所有服务共享的组件中进行策略定义和实现是有意义的。 | Spring Cloud Security通过Spring Cloud Zuul解决了许多安全问题 | Kubernetes生态系统提供了像Istio这样的服务网格，该网格能够通过其API网关机制提供安全性。 |
| 集中化日志记录：拥有一个集中化的日志收集和分析基础设施来管理大量的服务非常重要——其中许多服务是以分布式方式运行的。 | ELK技术栈\([Elasticsearch](https://zh.wikipedia.org/wiki/Elasticsearch)、LogStash、[Kibana](https://zh.wikipedia.org/w/index.php?title=Kibana&action=edit&redlink=1)\) | EFK技术栈（[Elasticsearch](https://zh.wikipedia.org/wiki/Elasticsearch)、[Fluentd](https://zh.wikipedia.org/wiki/Fluentd)、[Kibana](https://zh.wikipedia.org/w/index.php?title=Kibana&action=edit&redlink=1)） |
| 集中的度量：一个可以监控单个服务和整个系统的健康和性能的集中区域对于正确的操作是必不可少的。 | Spring Spectator & Atlas | Heapster, Prometheus, & [Grafana](https://zh.wikipedia.org/w/index.php?title=Grafana&action=edit&redlink=1) |
| 分布式跟踪：每个进程的日志记录和度量监视都有其存在的地方，但它们都不能重构事务在跨分布式系统传播时所采用的复杂路径。分布式跟踪是微服务平台必不可少的工具。 | Spring Cloud Sleuth | Hawkular, [Jaeger](https://zh.wikipedia.org/wiki/Linux%E5%9F%BA%E9%87%91%E6%9C%83) |
| 弹性和容错性：分布式系统必须能够围绕故障进行自动路由，并且能够将请求路由到提供最佳响应的服务实例。 | Spring Hystrix, Turbine, & Ribbon | Health check, [service meshes](https://zh.wikipedia.org/w/index.php?title=Service_meshes&action=edit&redlink=1) \(example: Istio\) |
| 自动伸缩和自我修复：分布式系统通过水平伸缩响应更高的负载: 平台必须检测并自动响应这些条件。此外，系统需要检测故障，并尝试自动重启，无需操作员输入。 | - | 健康检查、自我修复和自动缩放 |
| 打包、部署和调度: 大规模系统需要健壮的包管理和部署系统来管理滚动部署或蓝绿色部署，并在必要时进行回滚。调度程序帮助确定可以根据当前条件将一组新服务部署到哪个特定的执行节点。 | Spring Boot, Apache Maven. Spring Cloud系统没有真正的调度程序。 | Docker, Rkt, Kubernetes Scheduler & Deployment, Helm |
| 作业管理：控制无人值守的后台程序执行。 | Spring Batch | Kubernetes Jobs与Scheduled Jobs |
| 单例应用程序：限制特定服务作为该服务在整个系统中的唯一实例运行。 | Spring Cloud Cluster | Kubernetes Pods |

**参考资料：**

* \*\*\*\*[**微服务**](https://zh.wikipedia.org/wiki/%E5%BE%AE%E6%9C%8D%E5%8B%99)\*\*\*\*



