# 配置加载

在Spring Boot里面，可以使用以下几种方式来加载配置。

* properties文件
* YAML文件
* 系统环境变量
* 命令行参数

## YAML

YAML是一种人类可读的数据序列化语言。它通常用于配置文件。与属性文件相比，如果我们想要在配置文件中添加复杂的属性，YAML文件就更加结构化，而且更少混淆。可以看出YAML具有分层配置数据。

### 优势

* 配置有序，在一些特殊的场景下，配置有序很关键
* 支持数组，数组中的元素可以是基本数据类型也可以是对象
* 简洁

### 缺点

* 不支持`@PropertySource`注解导入自定义的 YAML 配置。

## 核心配置文件

Spring Boot 核心的两个配置文件：

* `bootstrap` \(`.yml`或者`.properties`\)：boostrap由父`ApplicationContext`加载的，比`applicaton`优先加载，配置在应用程序上下文的引导阶段生效。一般来说我们在Spring Cloud Config或者Nacos中会用到它。且boostrap里面的属性不能被覆盖；
* `application` \(`.yml`或者`. properties`\)： 由`ApplicatonContext`加载，用于spring boot项目的自动化配置。

## Profile

### Profile的作用

Profile对应中文并没有合适的翻译，它的主要作用就是让Spring Boot可以根据不同环境提供不同的配置功能支持。

我们经常遇到这样的场景：有开发、测试、生产等环境，不同的环境又有不同的配置。如果每个环境在部署时都需要修改配置文件将会非常麻烦，而通过Profile则可以轻松解决改问题。

### Profile的基本使用

比如上述环境，我们可以在Spring Boot中创建4个文件：

* `applcation.properties`：公共配置
* `application-dev.properties`：开发环境配置
* `application-test.properties`：测试环境配置
* `application-prod.properties`：生产环境配置

在`applcation.properties`中配置公共配置，然后通过如下配置激活指定环境的配置：

```yaml
spring.profiles.active = prod
```

其中“prod”对照文件名中`application-prod.properties`。Spring Boot在处理时会获取配置文件`applcation.properties`，然后通过指定的`profile`的值“prod”进行拼接，获得`application-prod.properties`文件的名称和路径。

举例说明，比如在开发环境使用服务的端口为8080，而在生产环境中需要使用18080端口。那么，在`application-prod.properties`中配置如下：

```yaml
server.port=18080
```

而在`application-prod.properties`中配置为：

```yaml
server.port=8080
```

在使用不同环境时，可以通过在applcation.properties中配置spring.profiles.active属性值来进行指定（如上面示例），也可以通过启动命令参数进行指定：

```bash
java -jar springboot.jar --spring.profiles.active=prod
```

这样打包后的程序只需通过命令行参数就可以使用不同环境的配置文件。  


  






**参考资料：**

* \*\*\*\*[**SpringBoot Profile使用详解及配置源码解析**](https://juejin.im/post/6844904029949001742)\*\*\*\*

