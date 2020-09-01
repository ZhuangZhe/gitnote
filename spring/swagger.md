# Swagger

## 简介

Swagger是一套基于OpenAPI规范构建的开源工具，可以帮助我们设计、构建、记录以及使用Rest API。Swagger主要包含了以下三个部分：

1. `Swagger Editor`：基于浏览器的编辑器，我们可以使用它编写我们 OpenAPI 规范。
2. `Swagger UI`：它会将我们编写的 OpenAPI 规范呈现为交互式的 API 文档，后文我将使用浏览器来查看并且操作我们的 Rest API。
3. `Swagger Codegen`：它可以通过为 OpenAPI\(以前称为 Swagger\)规范定义的任何 API 生成服务器存根和客户端 SDK 来简化构建过程。

## Java配置

Springfox提供了一个Docket对象，让我们可以灵活的配置Swagger的各项属性。

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .select()
                .apis(RequestHandlerSelectors.any())
                .paths(PathSelectors.any())
                .build();
    }
}
```

`@Configuration`是告诉 Spring Boot 需要加载这个配置类，`@EnableSwagger2`是启用 Swagger2，如果没加的话自然而然也就看不到后面的验证效果了。



## 高级配置

集成swagger-ui后，浏览器中访问`http://localhost:8080/swagger-ui.html` 就可以看到如下的效果了:

![](../.gitbook/assets/image%20%28105%29.png)

可以看到虽然可读性好了一些，但对接口的表述还不是那么的清楚，接下来我们就通过一些高级配置，让这份文档变的更加的易读。

### 文档相关描述配置

通过在控制器类上增加 `@Api` 注解，可以给控制器增加描述和标签信息。

```java
@Api(tags = "用户相关接口", description = "提供用户相关的 Rest API")
public class UserController
```

通过在接口方法上增加 `@ApiOperation` 注解来展开对接口的描述，当然这个注解还可以指定很多内容，我们在下面的相关注解说明章节中详细解释。

```java
@ApiOperation("新增用户接口")
@PostMapping("/add")
public boolean addUser(@RequestBody User user) {
    return false;
}
```

实体描述，我们可以通过`@ApiModel`和`@ApiModelProperty`注解来对我们 API 中所涉及到的对象做描述。

```java
@ApiModel("用户实体")
public class User {
    @ApiModelProperty("用户 id")
    private int id;
}
```

文档信息配置，Swagger还支持设置一些文档的版本号、联系人邮箱、网站、版权、开源协议等等信息，但与上面几条不同的是这些信息不是通过注解配置，而是通过创建一个ApiInfo对象，并且使用`Docket.appInfo()`方法来设置，我们在`SwaggerConfig.java`类中新增如下内容即可。

```java
@Bean
public Docket api() {
    return new Docket(DocumentationType.SWAGGER_2)
        .select()
        .apis(RequestHandlerSelectors.any())
        .paths(PathSelectors.any())
        .build()
        .apiInfo(apiInfo());
}

private ApiInfo apiInfo() {
    return new ApiInfo("Spring Boot 项目集成 Swagger 实例文档",
                       "我的博客网站：https://itweknow.cn，欢迎大家访问。",
                       "API V1.0",
                       "Terms of service",
                       new Contact("名字想好没", 
                                   "https://itweknow.cn", 
                                   "gancy.programmer@gmail.com"),
                                   "Apache", 
                                   "http://www.apache.org/", 
                                   Collections.emptyList());
}
```

经过上面的步骤，我们的文档将会变成下图的样子，现在看起来就清楚很多了。

![](../.gitbook/assets/image%20%28103%29.png)

### 接口过滤

有些时候我们并不是希望所有的 Rest API 都呈现在文档上，这种情况下Swagger2提供给我们了两种方式配置，一种是基于`@ApiIgnore`注解，另一种是在 Docket 上增加筛选。

如果想在文档中屏蔽掉删除用户的接口\(user/delete\)，那么只需要在删除用户的方法上加上`@ApiIgnore`即可。

```java
@ApiIgnore
public boolean delete(@PathVariable("id") int id)
```

在Docket上增加筛选。Docket 类提供了`apis()`和`paths()`两个方法来帮助我们在不同级别上过滤接口。

* `apis()`：这种方式我们可以通过指定包名的方式，让Swagger只去某些包下面扫描。
* `paths()`：这种方式可以通过筛选API的 url 来进行过滤。

在集成Swagger2的章节中我们这两个方法指定的都是扫描所有，没有指定任何过滤条件。如果我们在我们修改之前定义的Docket对象的`apis()`方法和`paths()`方法为下面的内容，那么接口文档将只会展示`/user/add`和`/user/find/{id}`两个接口。

```java
.apis(RequestHandlerSelectors.basePackage("cn.itweknow.sbswagger.controller"))
.paths(Predicates.or(PathSelectors.ant("/user/add"),
                     PathSelectors.ant("/user/find/*")))
```

![](../.gitbook/assets/image%20%28104%29.png)

### 自定义响应消息

Swagger允许我们通过Docket的`globalResponseMessage()`方法全局覆盖HTTP方法的响应消息，但是首先我们得通过Docket的`useDefaultResponseMessages`方法告诉Swagger不使用默认的HTTP响应消息，假设我们现在需要覆盖所有GET方法的500和403错误的响应消息，我们只需要在`SwaggerConfig.java`类中的Docket Bean下添加如下内容：

```java
.useDefaultResponseMessages(false)
.globalResponseMessage(RequestMethod.GET, newArrayList(
    new ResponseMessageBuilder()
        .code(500)
        .message("服务器发生异常")
        .responseModel(new ModelRef("Error"))
        .build(),
    new ResponseMessageBuilder()
        .code(403)
        .message("资源不可用")
        .build()
));
```

添加如上面的代码后，如下图所示，您会发现在SwaggerUI页面展示的所有GET类型请求的403以及500错误的响应消息都变成了我们自定义的内容。

## 相关注解说明

### Controller相关注解

`@Api`: 可设置对控制器的描述。

#### `@Api`**主要属性**

| **注解属性** | **类型** | **描述** |
| :--- | :--- | :--- |
| tags | String\[\] | 控制器标签。 |
| description | String | 控制器描述\(该字段被申明为过期\)。 |

### 接口相关注解

* `@ApiOperation`: 可设置对接口的描述。

**`@ApiOperation`主要属性**

| **注解属性** | **类型** | **描述** |
| :--- | :--- | :--- |
| value | String | 接口说明。 |
| notes | String | 接口发布说明。 |
| tags | Stirng\[\] | 标签。 |
| response | Class&lt;?&gt; | 接口返回类型。 |
| httpMethod | String | 接口请求方式。 |

* `@ApiIgnore`: Swagger 文档不会显示拥有该注解的接口。 
* `@ApiImplicitParams`: 用于描述接口的非对象参数集。 
* `@ApiImplicitParam`: 用于描述接口的非对象参数，一般与`@ApiImplicitParams`组合使用。

**`@ApiImplicitParam`主要属性**

| **注解属性** | **描述** |
| :--- | :--- |
| paramType | 查询参数类型，实际上就是参数放在那里。取值：path：以地址的形式提交数据，根据 id 查询用户的接口就是这种形式传参；query：Query string 的方式传参；header：以流的形式提交；form：以 Form 表单的形式提交。 |
| dataType | 参数的数据类型。取值：Long 和 String |
| name | 参数名字。 |
| value | 参数意义的描述。 |
| required | 是否必填。取值：true：必填参数；false：非必填参数。 |

### Model 相关注解

* `@ApiModel`: 可设置接口相关实体的描述。 
* `@ApiModelProperty`: 可设置实体属性的相关描述。

**`@ApiModelProperty`主要属性**

| **注解属性** | **类型** | **描述** |
| :--- | :--- | :--- |
| value | String | 字段说明。 |
| name | String | 重写字段名称。 |
| dataType | Stirng | 重写字段类型。 |
| required | boolean | 是否必填。 |
| example | Stirng | 举例说明。 |
| hidden | boolean | 是否在文档中隐藏该字段。 |
| allowEmptyValue | boolean | 是否允许为空。 |
| allowableValues | String | 该字段允许的值，当我们 API 的某个参数为枚举类型时，使用这个属性就可以清楚地告诉 API 使用者该参数所能允许传入的值。 |

**参考资料：**

* [**在 Spring Boot 项目中使用 Swagger 文档**](https://developer.ibm.com/zh/articles/j-using-swagger-in-a-spring-boot-project/)\*\*\*\*

