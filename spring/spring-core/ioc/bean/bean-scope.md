# Bean作用域

**Spring中的bean默认都是单例的，这些单例Bean在多线程程序下如何保证线程安全呢？** 例如对于Web应用来说，Web容器对于每个用户请求都创建一个单独的Sevlet线程来处理请求，引入Spring框架之后，每个Action都是单例的，那么对于Spring托管的单例Service Bean，如何保证其安全呢？ **Spring的单例是基于BeanFactory也就是Spring容器的，单例Bean在此容器内只有一个，Java的单例是基于JVM，每个 JVM 内只有一个实例。**

创建一个bean定义，其实质是用该bean定义对应的类来创建真正实例的“配方”。把bean定义看成一个配方很有意义，它与class很类似，只根据一张“处方”就可以创建多个实例。不仅可以控制注入到对象中的各种依赖和配置值，还可以控制该对象的作用域。这样可以灵活选择所建对象的作用域，而不必在Java Class级定义作用域。Spring支持五种作用域**：**

![](https://pic4.zhimg.com/80/v2-11939e56c5907389ca9026ac144a62fa_1440w.jpg)

五种作用域中，**request、session** 和 **global session** 三种作用域仅在基于web的应用中使用，只能用在基于web的`ApplicationContext`环境。

## singleton

**当一个bean的作用域为singleton，那么IoC容器中只会存在一个共享的bean实例，并且所有对bean的请求，只要id与该bean定义相匹配，则只会返回bean的同一实例。** singleton是单例类型\(对应于单例模式\)，就是在创建起容器时就同时自动创建了一个bean的对象，不管你是否使用，他都存在了，每次获取到的对象都是同一个对象。注意，singleton作用域是Spring中的默认作用域。要在XML中将bean定义成 singleton ，可以这样配置：

```markup
<bean id="ServiceImpl" class="cn.csdn.service.ServiceImpl" scope="singleton">
```

也可以通过`@Scope`注解的方式：

```java
@Service
@Scope("singleton")
public class ServiceImpl{ }
```

## prototype

**当一个bean的作用域为prototype，表示一个bean定义对应多个对象实例。** **prototype作用域的bean会导致在每次对该bean请求\(**将其注入到另一个bean中，或者以程序的方式调用容器的`getBean()`方法\)**时都会创建一个新的bean实例。prototype是原型类型，它在我们创建容器的时候并没有实例化，而是当我们获取bean的时候才会去创建一个对象，而且我们每次获取到的对象都不是同一个对象。根据经验，对有状态的bean应该使用 prototype 作用域，而对无状态的bean则应该使用singleton作用域。** 在XML中将bean定义成prototype，可以这样配置：

```markup
<bean id="account" class="com.foo.DefaultAccount" scope="prototype"/>  
```

或

```markup
<bean id="account" class="com.foo.DefaultAccount" singleton="false"/>
```

## request

**request只适用于Web程序，每一次HTTP请求都会产生一个新的bean，同时该bean仅在当前HTTP request内有效，当请求结束后，该对象的生命周期即告结束。**

```markup
<bean id="loginAction" class=cn.csdn.LoginAction" scope="request"/>
```

## session

**session只适用于Web程序，session作用域表示该针对每一次HTTP请求都会产生一个新的bean，同时该bean仅在当前 HTTP session内有效.与request作用域一样，可以根据需要放心的更改所创建实例的内部状态，而别的HTTP session中根据userPreferences创建的实例，将不会看到这些特定于某个HTTP session的状态变化。当HTTP session最终被废弃的时候，在该HTTP session作用域内的bean也会被废弃掉。**

```markup
<bean id="userPreferences" class="com.foo.UserPreferences" scope="session"/>
```

## globalSession

global session作用域类似于标准的HTTP session作用域，不过仅仅在基于portlet的web应用中才有意义。Portlet规范定义了全局Session的概念，它被所有构成某个portlet web应用的各种不同的portlet所共享。在global session作用域中定义的bean被限定于全局portlet Session的生命周期范围内。

```markup
<bean id="user" class="com.foo.Preferences "scope="globalSession"/>
```

