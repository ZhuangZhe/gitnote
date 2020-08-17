# Spring Boot

## 简介

### 是什么？

Spring Boot是Spring开源组织下的子项目，是Spring组件一站式解决方案，主要是简化了使用Spring的难度，简省了繁重的配置，提供了各种启动器，开发者能快速上手。

Spring boot其实就是一个启动spring项目的一个工具而已。从最根本上来讲，Spring Boot就是一些库的集合，是spring的一个封装，它能够被任意项目的构建系统所使用。

### 为什么用？

以前在写spring项目的时候，要配置各种xml文件，还记得曾经被ssh框架支配的恐惧。随着spring3，spring4的相继推出，约定大于配置逐渐成为了开发者的共识，大家也渐渐的从写xml转为写各种注解，在spring4的项目里，你甚至可以一行xml都不写。

虽然spring4已经可以做到无xml，但写一个大项目需要茫茫多的包，maven配置要写几百行，也是一件很可怕的事。

现在，快速开发一个网站的平台层出不穷，nodejs，php等虎视眈眈，并且脚本语言渐渐流行了起来，Spring的开发模式越来越显得笨重。

在这种环境下，spring boot伴随着spring4一起出现了。

#### 优点

* **使编码变得简单：**spring boot采用java config的方式，对spring进行配置，并且提供了大量的注解，极大地提高了工作效率。
* **使配置变得简单：**spring boot提供许多默认配置，当然也提供自定义配置。但是所有spring boot的项目都只有一个配置文件：`application.properties/application.yml`。用了spring boot，再也不用担心配置出错找不到问题所在了。
* **使部署变得简单：**你只需要一个java的运行环境就可以跑spring boot的项目了。spring boot的项目可以打成一个jar包，然后通过`java -jar xxx.jar`来运行。
* **使监控变得简单：**spring boot提供了actuator包，可以使用它来对你的应用进行监控。
* 提供了一系列大型项目通用的非业务性功能，例如：内嵌服务器、安全管理、运行数据监控、运行状况检查和外部化配置等。
* 避免大量的Maven导入和各种版本冲突。



**参考资料：**

* \*\*\*\*[**Spring Boot面试题（2020最新版）**](https://blog.csdn.net/ThinkWon/article/details/104397299)\*\*\*\*
* \*\*\*\*[**【springboot 入门篇】第0篇 spring-boot是什么**](https://blog.csdn.net/qq_31655965/article/details/71258191)\*\*\*\*



