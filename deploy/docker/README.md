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

* 开发者在本地写的代码可以通过Docker分享给同时。
* Your developers write code locally and share their work with their colleagues using Docker containers.
* They use Docker to push their applications into a test environment and execute automated and manual tests.
* When developers find bugs, they can fix them in the development environment and redeploy them to the test environment for testing and validation.
* When testing is complete, getting the fix to the customer is as simple as pushing the updated image to the production environment.

### 响应式部署和扩展







**参考资料：**

* \*\*\*\*[**Docker overview**](https://docs.docker.com/get-started/overview/)\*\*\*\*



