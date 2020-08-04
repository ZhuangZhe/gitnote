# Bean生命周期

![](https://pic1.zhimg.com/80/v2-baaf7d50702f6d0935820b9415ff364c_1440w.jpg?source=1940ef5c)

![](../../../../.gitbook/assets/image%20%2823%29.png)

## 实例化Bean

在入口方法`getBean`中，首先调用了`doCreateBean`方法。第一步就是通过反射实例化一个Bean。

```java
// AbstractAutowireCapableBeanFactory.java
protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) throws BeanCreationException {
    BeanWrapper instanceWrapper = null;
    if (mbd.isSingleton()) {
        instanceWrapper = (BeanWrapper)this.factoryBeanInstanceCache.remove(beanName);
    }

    if (instanceWrapper == null) {
        instanceWrapper = this.createBeanInstance(beanName, mbd, args);
    }

    Object bean = instanceWrapper.getWrappedInstance();
    Class<?> beanType = instanceWrapper.getWrappedClass();
    if (beanType != NullBean.class) {
        mbd.resolvedTargetType = beanType;
    }

    synchronized(mbd.postProcessingLock) {
        if (!mbd.postProcessed) {
            try {
                this.applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
            } catch (Throwable var17) {
                throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Post-processing of merged bean definition failed", var17);
            }

            mbd.postProcessed = true;
        }
    }

    boolean earlySingletonExposure = mbd.isSingleton() && this.allowCircularReferences && this.isSingletonCurrentlyInCreation(beanName);
    if (earlySingletonExposure) {
        if (this.logger.isTraceEnabled()) {
            this.logger.trace("Eagerly caching bean '" + beanName + "' to allow for resolving potential circular references");
        }

        this.addSingletonFactory(beanName, () -> {
            return this.getEarlyBeanReference(beanName, mbd, bean);
        });
    }

    Object exposedObject = bean;

    try {
        this.populateBean(beanName, mbd, instanceWrapper);
        exposedObject = this.initializeBean(beanName, exposedObject, mbd);
    } catch (Throwable var18) {
        if (var18 instanceof BeanCreationException && beanName.equals(((BeanCreationException)var18).getBeanName())) {
            throw (BeanCreationException)var18;
        }

        throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Initialization of bean failed", var18);
    }

    if (earlySingletonExposure) {
        Object earlySingletonReference = this.getSingleton(beanName, false);
        if (earlySingletonReference != null) {
            if (exposedObject == bean) {
                exposedObject = earlySingletonReference;
            } else if (!this.allowRawInjectionDespiteWrapping && this.hasDependentBean(beanName)) {
                String[] dependentBeans = this.getDependentBeans(beanName);
                Set<String> actualDependentBeans = new LinkedHashSet(dependentBeans.length);
                String[] var12 = dependentBeans;
                int var13 = dependentBeans.length;

                for(int var14 = 0; var14 < var13; ++var14) {
                    String dependentBean = var12[var14];
                    if (!this.removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
                        actualDependentBeans.add(dependentBean);
                    }
                }

                if (!actualDependentBeans.isEmpty()) {
                    throw new BeanCurrentlyInCreationException(beanName, "Bean with name '" + beanName + "' has been injected into other beans [" + StringUtils.collectionToCommaDelimitedString(actualDependentBeans) + "] in its raw version as part of a circular reference, but has eventually been wrapped. This means that said other beans do not use the final version of the bean. This is often the result of over-eager type matching - consider using 'getBeanNamesForType' with the 'allowEagerInit' flag turned off, for example.");
                }
            }
        }
    }

    try {
        this.registerDisposableBeanIfNecessary(beanName, bean, mbd);
        return exposedObject;
    } catch (BeanDefinitionValidationException var16) {
        throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Invalid destruction signature", var16);
    }
}

```

对于`BeanFactory`容器，当客户向容器请求一个尚未初始化的bean时，或初始化bean的时候需要注入另一个尚未初始化的依赖时，容器就会调用`createBean`进行实例化。 

对于`ApplicationContext`容器，当容器启动结束后，便实例化所有的bean。 

容器通过获取`BeanDefinition`对象中的信息进行实例化。并且这一步仅仅是简单的实例化，并未进行依赖注入。 

实例化对象被包装在`BeanWrapper`对象中，BeanWrapper提供了设置对象属性的接口，从而避免了使用反射机制设置属性。

## 设置对象属性\(依赖注入\)

实例化后的对象被封装在`BeanWrapper`对象中，并且此时对象仍然是一个原生的状态，并没有进行依赖注入。 

紧接着，Spring根据`BeanDefinition`中的信息进行依赖注入，并且通过`BeanWrapper`提供的设置属性的接口完成依赖注入。

## 注入Aware接口

紧接着，Spring会检测该对象是否实现了`xxxAware`接口，并将相关的`xxxAware`实例注入给bean。

如果Bean实现了`BeanNameAware`接口，Spring将Bean的ID传递给`setBeanName()`方法。\(**实现`BeanNameAware`主要是为了通过Bean的引用来获得Bean的ID，一般业务中是很少有用到Bean的ID的\)**

如果Bean实现了`BeanFactoryAware`接口，Spring将调用`setBeanDactory(BeanFactory bf)`方法并把`BeanFactory`容器实例作为参数传入。**\(实现`BeanFactoryAware`主要目的是为了获取Spring容器，如Bean通过Spring容器发布事件等\)**  
  
如果Bean实现了`ApplicationContextAware`接口，Spring容器将调用`setApplicationContext(ApplicationContext ctx)`方法，把y应用上下文作为参数传入。**\(作用与`BeanFactory`类似都是为了获取Spring容器，不同的是Spring容器在调用`setApplicationContext`方法时会把它自己作为`setApplicationContext`的参数传入，而Spring容器在调用`setBeanDactory`前需要程序员自己指定\(注入\)`setBeanFactory`里的参数`BeanFactory`\)**

## BeanPostProcessor

当经过上述几个步骤后，bean对象已经被正确构造，但如果你想要对象被使用前再进行一些自定义的处理，就可以通过`BeanPostProcessor`接口实现。   
该接口提供了两个函数：

* `postProcessBeforeInitialzation( Object bean, String beanName )`
  * 当前正在初始化的bean对象会被传递进来，我们就可以对这个bean作任何处理。 
  * 这个函数会先于`InitialzationBean`执行，因此称为前置处理。 
  * 所有`Aware`接口的注入就是在这一步完成的。
* `postProcessAfterInitialzation( Object bean, String beanName )` 
  * 当前正在初始化的bean对象会被传递进来，我们就可以对这个bean作任何处理。 
  * 这个函数会在`InitialzationBean`完成后执行，因此称为后置处理。

## InitializingBean与init-method

当`BeanPostProcessor`的前置处理完成后就会进入本阶段。   
`InitializingBean`接口只有一个函数：

* `afterPropertiesSet()`
  * 这一阶段也可以在bean正式构造完成前增加我们自定义的逻辑，但它与前置处理不同，由于该函数并不会把当前bean对象传进来，因此在这一步没办法处理对象本身，只能增加一些额外的逻辑。 
  * 若要使用它，我们需要让bean实现该接口，并把要增加的逻辑写在该函数中。然后Spring会在前置处理完成后检测当前bean是否实现了该接口，并执行`afterPropertiesSet`函数。
  * Spring为了降低对客户代码的侵入性，给bean的配置提供了`init-method`属性，该属性指定了在这一阶段需要执行的函数名。Spring便会在初始化阶段执行我们设置的函数。`init-method`本质上仍然使用了`InitializingBean`接口。

## DisposableBean和destroy-method

和`init-method`一样，通过给`destroy-method`指定函数，就可以在bean销毁前执行指定的逻辑。

