# IOC依赖注入

## Annotation的支持

在Bean实例化完成之后，会进入一段后置处理器的代码。从代码上看，过滤实现了`MergedBeanDefinitionPostProcessor`接口的类，调用其`postProcessMergedBeanDefinition()`方法。三个`类`实现了`MergedBeanDefinitionPostProcessor`

* `AutowiredAnnotationBeanPostProcessor`
* `CommonAnnotationBeanPostProcessor`
* `RequiredAnnotationBeanPostProcessor`

我们说Spring对`annotation-config`标签的支持，注册了一些特殊的Bean，正好就包含上面这三个。

从方法名字来看，它们做了相同一件事，加载注解元数据。方法内部又做了相同的两件事：

```java
ReflectionUtils.doWithLocalFields(targetClass, new ReflectionUtils.FieldCallback() 
ReflectionUtils.doWithLocalMethods(targetClass, new ReflectionUtils.MethodCallback()
```

看方法的参数，`targetClass`就是Bean的Class对象。接下来就可以获取它的字段和方法，判断是否包含了相应的注解，最后转成`InjectionMetadata`对象，下面以一段伪代码展示处理过程。

```java
public static void main(String[] args) throws ClassNotFoundException {
    Class<?> clazz = Class.forName("com.viewscenes.netsupervisor.entity.User");
    Field[] fields = clazz.getFields();
    Method[] methods = clazz.getMethods();
    
    for (int i = 0; i < fields.length; i++) {
        Field field = fields[i];
        if (field.isAnnotationPresent(Autowired.class)) {
            //转换成AutowiredFieldElement对象，加入容器
        }
    }
    for (int i = 0; i < methods.length; i++) {
        Method method = methods[i];
        if (method.isAnnotationPresent(Autowired.class)) {
            //转换成AutowiredMethodElement对象，加入容器
        }
    }
    return new InjectionMetadata(clazz, elements);
}
```

`InjectionMetadata`对象有两个重要的属性：`targetClass`，`injectedElements`，在注解式的依赖注入的时候重点就靠它们。

```java
public InjectionMetadata(Class<?> targetClass, Collection<InjectedElement> elements) {
    //targetClass是Bean的Class对象
    this.targetClass = targetClass; 
    //injectedElements是一个InjectedElement对象的集合
    this.injectedElements = elements;
}

//member是成员本身，字段或者方法
//pd是JDK中的内省机制对象，后面的注入属性值要用到
protected InjectedElement(Member member, PropertyDescriptor pd) {
    this.member = member;
    this.isField = (member instanceof Field);
    this.pd = pd;
}
```

最后再看下源码里面是什么样的，以`Autowired`为例：

```java
ReflectionUtils.doWithLocalFields(targetClass, new ReflectionUtils.FieldCallback() {
    @Override
    public void doWith(Field field) throws IllegalArgumentException, IllegalAccessException {
        AnnotationAttributes ann = findAutowiredAnnotation(field);
        if (ann != null) {
            if (Modifier.isStatic(field.getModifiers())) {
                if (logger.isWarnEnabled()) {
                    logger.warn("Autowired annotation is not supported on static fields");
                }
                return;
            }
            boolean required = determineRequiredStatus(ann);
            currElements.add(new AutowiredFieldElement(field, required));
        }
    }
});

ReflectionUtils.doWithLocalMethods(targetClass, new ReflectionUtils.MethodCallback() {
    @Override
    public void doWith(Method method) throws IllegalArgumentException, IllegalAccessException {
        Method bridgedMethod = BridgeMethodResolver.findBridgedMethod(method);
        if (!BridgeMethodResolver.isVisibilityBridgeMethodPair(method, bridgedMethod)) {
            return;
        }
        AnnotationAttributes ann = findAutowiredAnnotation(bridgedMethod);
        if (ann != null && method.equals(ClassUtils.getMostSpecificMethod(method, clazz))) {
            if (Modifier.isStatic(method.getModifiers())) {
                if (logger.isWarnEnabled()) {
                    logger.warn("Autowired annotation is not supported on static methods");
                }
                return;
            }
            if (method.getParameterTypes().length == 0) {
                if (logger.isWarnEnabled()) {
                    logger.warn("Autowired annotation should be used on methods with parameters:" );
                }
            }
            boolean required = determineRequiredStatus(ann);
            PropertyDescriptor pd = BeanUtils.findPropertyForMethod(bridgedMethod, clazz);
            currElements.add(new AutowiredMethodElement(method, required, pd));
        }
    }
});
```

## 依赖注入

完成了在`doCreateBean()`方法Bean的实例化，接下来就是依赖注入。 

Bean的依赖注入有两种方式：一种是配置文件，一种是注解式。

### **注解式的注入过程**

Spring已经过滤了Bean实例上包含`@Autowired`、`@Resource`等注解的Field和Method，并返回了包含Class对象、内省对象、成员的`InjectionMetadata`对象。还是以`@Autowired`为例，这次调用到`AutowiredAnnotationBeanPostProcessor.postProcessPropertyValues（）`。 首先拿到`InjectionMetadata`对象，再判断里面的`InjectedElement`集合是否为空，也就是说判断在Bean的字段和方法上是否包含`@Autowired`。然后调用`InjectedElement.inject()`。`InjectedElement`有两个子类`AutowiredFieldElement、AutowiredMethodElement`，很显然一个是处理Field，一个是处理Method。

#### **AutowiredFieldElement**

如果Autowired注解在字段上，它的配置是这样。

```java
public class User {	
    @Autowired
    Role role;
}
```

```java
protected void inject(Object bean, String beanName, PropertyValues pvs) throws Throwable {
    //以User类中的@Autowired Role role为例，这里的field就是
    //public com.viewscenes.netsupervisor.entity.Role 
    //com.viewscenes.netsupervisor.entity.User.role
    Field field = (Field) this.member;
    Object value;
    DependencyDescriptor desc = new DependencyDescriptor(field, this.required);
    desc.setContainingClass(bean.getClass());
    Set<String> autowiredBeanNames = new LinkedHashSet<String>(1);
    TypeConverter typeConverter = beanFactory.getTypeConverter();
    try {
        //这里的beanName因为Bean，所以会重新进入populateBean方法，先完成Role对象的注入
        //value == com.viewscenes.netsupervisor.entity.Role@7228c85c
        value = beanFactory.resolveDependency(desc, beanName, autowiredBeanNames, typeConverter);
    }
    catch (BeansException ex) {
        throw new UnsatisfiedDependencyException(null, beanName, new InjectionPoint(field), ex);
    }
    if (value != null) {
        //设置可访问，直接赋值
        ReflectionUtils.makeAccessible(field);
        field.set(bean, value);
    }
}
```

#### **AutowiredFieldElement**

如果`Autowired`注解在方法上，就得这样写。

```java
public class User {
	@Autowired
	public void setRole(Role role) {}
}
```

它的inject方法和上面类似，不过最后是method.invoke。感兴趣的小伙伴可以去翻翻源码。

```java
ReflectionUtils.makeAccessible(method);
method.invoke(bean, arguments);
```

### **配置文件的注入过程**

有一个配置文件，在User类中注入了id，name，age和Role的实例。

```markup
<bean id="user" class="com.viewscenes.netsupervisor.entity.User">
    <property name="id" value="1001"></property>
    <property name="name" value="网机动车"></property>
    <property name="age" value="24"></property>
    <property name="role" ref="role"></property>
</bean>
<bean id="role" class="com.viewscenes.netsupervisor.entity.Role">
    <property name="id" value="1002"></property>
    <property name="name" value="中心管理员"></property>
</bean>
```

bean标签的解析，我们看到在反射得到Bean的Class对象后，会设置它的property属性，也就是调用了`parsePropertyElements()`方法。在`BeanDefinition`对象里有个`MutablePropertyValues`属性。

```markup
MutablePropertyValues:
    //propertyValueList就是有几个property 节点
    List<PropertyValue> propertyValueList:
        PropertyValue:
            name 		//对应配置文件中的name    ==id
            value 	//对应配置文件中的value  ==1001 
        PropertyValue:
            name 		//对应配置文件中的name    ==name
            value 	//对应配置文件中的value  ==网机动车 
```

上图就是`BeanDefinition`对象里面`MutablePropertyValues`属性的结构。既然已经拿到了property的名称和值，注入就比较简单了。从内省对象`PropertyDescriptor`中拿到`writeMethod`对象，设置可访问，invoke即可。`PropertyDescriptor`有两个对象`readMethodRef、writeMethodRef`其实对应的就是`getter`、`setter`方法。

```java
public void setValue(final Object object, Object valueToApply) throws Exception {
    //pd 是内省对象PropertyDescriptor
    final Method writeMethod = this.pd.getWriteMethod());
    writeMethod.setAccessible(true);
    final Object value = valueToApply;
    //以id为例  writeMethod == public void com.viewscenes.netsupervisor.entity.User.setId(string)
    writeMethod.invoke(getWrappedInstance(), value);
}
```

## initializeBean

在Bean实例化和IOC依赖注入后，Spring留出了扩展，可以让我们对Bean做一些初始化的工作。

### **Aware**

`Aware`是一个空的接口，什么也没有。不过有很多`xxxAware`继承自它，下面来看源码。如果有需要，我们的Bean可以实现下面的接口拿到我们想要的。

```java
//在实例化和IOC依赖注入完成后调用
private void invokeAwareMethods(final String beanName, final Object bean) {
    if (bean instanceof Aware) {
        //让我们的Bean可以拿到自身在容器中的beanName
        if (bean instanceof BeanNameAware) {
            ((BeanNameAware) bean).setBeanName(beanName);
        }
        //可以拿到ClassLoader对象
        if (bean instanceof BeanClassLoaderAware) {
            ((BeanClassLoaderAware) bean).setBeanClassLoader(getBeanClassLoader());
        }
        //可以拿到BeanFactory对象
        if (bean instanceof BeanFactoryAware) {
            ((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);
        }
        if (bean instanceof ResourceLoaderAware) {
            ((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
        }
        if (bean instanceof MessageSourceAware) {
            ((MessageSourceAware) bean).setMessageSource(this.applicationContext);
        }
        if (bean instanceof ApplicationContextAware) {
            ((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
        }
        // ...
    }
}
```

做法如下：

```java
public class AwareTest1 implements BeanNameAware,BeanClassLoaderAware,BeanFactoryAware{
    public void setBeanName(String name) { 
        System.out.println("BeanNameAware:" + name);
    }
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
	      System.out.println("BeanFactoryAware:" + beanFactory);	
    }
    public void setBeanClassLoader(ClassLoader classLoader) {
	      System.out.println("BeanClassLoaderAware:" + classLoader);	
    }
}
//输出结果
BeanNameAware:awareTest1
BeanClassLoaderAware:WebappClassLoader
  context: /springmvc_dubbo_producer
  delegate: false
  repositories:
    /WEB-INF/classes/
----------> Parent Classloader:
java.net.URLClassLoader@2626b418
BeanFactoryAware:org.springframework.beans.factory.support.DefaultListableBean
Factory@5b4686b4: defining beans ...
```

### **初始化**

Bean的初始化方法有三种方式，按照先后顺序是：`@PostConstruct`、`afterPropertiesSet`、`init-method`。

#### **@PostConstruct**

这个注解隐藏的比较深，它是在`CommonAnnotationBeanPostProcessor`的父类`InitDestroyAnnotationBeanPostProcessor`调用到的。这个注解的初始化方法不支持带参数，会直接抛异常。

```java
if (method.getParameterTypes().length != 0) {
		throw new IllegalStateException("Lifecycle method annotation requires a no-arg method");
}
public void invoke(Object target) throws Throwable {
		ReflectionUtils.makeAccessible(this.method);
		this.method.invoke(target, (Object[]) null);
}
```

#### **afterPropertiesSet**

这个要实现`InitializingBean`接口。这个也不能有参数，因为它接口方法就没有定义参数。

```java
boolean isInitializingBean = (bean instanceof InitializingBean);
if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
    if (logger.isDebugEnabled()) {
        logger.debug("Invoking afterPropertiesSet() on bean with name '" + beanName + "'");
    }
    ((InitializingBean) bean).afterPropertiesSet();
}
```

#### **init-method**

```java
ReflectionUtils.makeAccessible(initMethod);
initMethod.invoke(bean);
```

## 注册

`registerDisposableBeanIfNecessary()`完成Bean的缓存注册工作，把Bean注册到Map中。

