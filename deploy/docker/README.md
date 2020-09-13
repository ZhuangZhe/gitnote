# Docker

Docker是一个用于开发，交付和运行应用程序的开放平台。 Docker使您能够将应用程序与基础架构分开，从而可以快速交付软件。借助Docker，您可以以与管理应用程序相同的方式来管理基础架构。通过利用Docker的快速交付，测试和部署代码的方法，您可以大大减少编写代码和在生产环境中运行代码之间的延迟。

## Docker平台

Docker通过容器\(container\)实现在松散封闭环境中包装及运行程序。Docker隔离和安全的特性使得我们可以同时在一个主机上同时运行多个容器。容器是轻量级，因为他不需要额外的监管程序，但是他直接运行在机器内核的。这意味着与使用虚拟机相比，在给定的硬件组合上可以运行更多的容器。您甚至可以在实际上是虚拟机的主机中运行Docker容器。

Docker提供了工具和平台来管理容器的生命周期：

* 使用容器开发应用程序及其支持组件。
* 容器成为分发和测试您的应用程序的单元。
* 准备就绪后，可以将应用程序作为容器或协调服务部署到生产环境中。无论您的生产环境是本地数据中心，云提供商还是两者的混合，其工作原理都相同。

## Docker引擎

Docker引擎是一个client-server应用，其中包含下列部件：

* 服务器是一种长期运行的程序，称为守护进程。
* REST API，它指定程序可以用来与守护程序进行通信并指示其操作的接口。
* 命令行界面\(CLI\)

![](../../.gitbook/assets/image%20%28106%29.png)

CLI通过REST API对Docker守护线程进行交互或控制。守护线程负责创建与管理Docker对象，如image、container、network和volume。

## Docker可以用来做什么

### 持续交付程序

Docker的流线型开发生命周期使得开发者可以在不同环境中用容器发布程序与服务。容器同时也便于持续集成与持续交付。

例子场景：

* 开发者在本地写的代码可以通过Docker容器分享给同事。
* 开发者可以使用Docker将程序放到测试环境进行手动或自动测试。
* 开发者可以很方便的在自己换环境调试bug，完成后在测试环境中验证。
* 当测试通过后，可以很方便的通过容器将程序推送到生产环境。

### 响应式部署和扩展

Docker基于容器的平台实现了高度可移植的负载。Docker容器可以在开发者的电脑、数据中心的物理或虚拟设备、云服务提供者、复合环境等环境上运行。

Docker的可移植性以及轻量化特性使得可以很容易的动态管理负载，以及以近乎实时根据业务的需要扩大或缩小程序与服务的规模。

### **可以在同一个硬件上跑更多的负载**

Docker是轻量级以及快速的，它提供一个可行和低成本的方法替换基于超管理器的虚拟机，所以开发者可以获得更多算力去实现他们的业务。Docker非常适合高密度环境以及中型或小型的的部署，它可以使用有限的资源来实现更多的事情。

## Docker的架构

Docker使用client-server架构。Docker client与Docker Daemon交互，负责构建、运行、分发Docker容器。Docker client和deamon一般在同一个系统上运行，但也可以把Docker client连接到远端的Docker deamon上。Docker client和deamon之间使用REST API进行通信，通过UNIX socket或者网络端口。

![](../../.gitbook/assets/jie-ping-20200912-shang-wu-9.48.05.png)

### The Docker daemon

Docker daemon\(`dockerd`\) 负责监听Docker的API请求，以及管理Docker对象。Docker deamon还会与其他deamon通信，以达到管理Docker service的目的。

### The Docker client

Docker client\(`docker`\)  是用户与Docker交互的首选方法。当使用`docker run`等命令时，client会发送这些命令去deamon，并由deamon执行这些命令。Docker命令使用Docker API。Docker client可以和多个deamon进行通信。

### Docker注册表

Docker注册表负责储存Docker镜像。Docker Hub是最流行的一个Docker注册表，任何人都可以使用，Docker默认从Docker Hub中获取镜像。用户也可以设立自己的Docker注册表。

当使用`docker pull`或`docker run`命令时，镜像将会从默认或你指定的注册表中pull下来。同时也可以使用`docker push`命令将镜像推送到指定的注册表。

### Docker对象

当使用Docker时，需要创建和使用镜像、容器、network、volume、plugin等对象。

#### 镜像

镜像是一个用于创建Docker容器的只读模版。通常来说一个镜像是基于另外一个镜像的，在基础镜像上做做一些客制化的添加和修改。例如在一个Ubuntu的镜像上添加Apache服务器以及应用程序，以及其他需要的配置。

可以通过使用专门的syntax编辑Dockerfile来创建和运行一个镜像。Dockerfile中的每个指令都会在镜像中添加一个layer。当Dockerfile被修改并重新构建镜像时，镜像中只有那些被改动了的layer会被重建。这使得镜像和虚拟机比起来既轻量、又小、又快。

#### 容器

容器是镜像的一个可运行的实例。用户可以使用Docker API和CLI创建、启动、停止、移动、删除一个容器。可以将一个容器连接去或依附于一个或多个network。

默认情况下，一个容器是相对较好的与其他容器和主机隔离开的。同时也可以控制一个容器在网络、存储、幕后子系统等方面与其他容器和host之间的隔离程度。

容器由它使用的镜像与配置选项决定。当一个容器被移除，这个容器上的任何状态变更都不会被贮存起来。

#### **SERVICES**

Services allow you to scale containers across multiple Docker daemons, which all work together as a _swarm_ with multiple _managers_and _workers_. Each member of a swarm is a Docker daemon, and all the daemons communicate using the Docker API. A service allows you to define the desired state, such as the number of replicas of the service that must be available at any given time. By default, the service is load-balanced across all worker nodes. To the consumer, the Docker service appears to be a single application. Docker Engine supports swarm mode in Docker 1.12 and higher.

## 背后使用的技术

Docker is written in [Go](https://golang.org/) and takes advantage of several features of the Linux kernel to deliver its functionality.

### Namespaces

Docker uses a technology called `namespaces` to provide the isolated workspace called the _container_. When you run a container, Docker creates a set of _namespaces_ for that container.

These namespaces provide a layer of isolation. Each aspect of a container runs in a separate namespace and its access is limited to that namespace.

Docker Engine uses namespaces such as the following on Linux:

* **The `pid` namespace:** Process isolation \(PID: Process ID\).
* **The `net` namespace:** Managing network interfaces \(NET: Networking\).
* **The `ipc` namespace:** Managing access to IPC resources \(IPC: InterProcess Communication\).
* **The `mnt` namespace:** Managing filesystem mount points \(MNT: Mount\).
* **The `uts` namespace:** Isolating kernel and version identifiers. \(UTS: Unix Timesharing System\).

### Control groups

Docker Engine on Linux also relies on another technology called _control groups_ \(`cgroups`\). A cgroup limits an application to a specific set of resources. Control groups allow Docker Engine to share available hardware resources to containers and optionally enforce limits and constraints. For example, you can limit the memory available to a specific container.

### Union file systems

Union file systems, or UnionFS, are file systems that operate by creating layers, making them very lightweight and fast. Docker Engine uses UnionFS to provide the building blocks for containers. Docker Engine can use multiple UnionFS variants, including AUFS, BTRFS, VFS, and DeviceMapper.

### Container format

Docker Engine combines the namespaces, control groups, and UnionFS into a wrapper called a container format. The default container format is `libcontainer`. In the future, Docker may support other container formats by integrating with technologies such as BSD Jails or Solaris Zones.







**参考资料：**

* \*\*\*\*[**Docker overview**](https://docs.docker.com/get-started/overview/)\*\*\*\*



