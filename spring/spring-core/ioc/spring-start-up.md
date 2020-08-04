# Spring的启动过程

Spring有一个入口`org.springframework.web.servlet.DispatcherServlet`，最终会调用到同一个方法`org.springframework.context.support.AbstractApplicationContext.refresh()`。

## 初始化

Spring的初始化第一步就是要加载配置文件，然后解析里面的配置项。

`XmlWebApplicationContext`类的`loadBeanDefinitions`方法：

```java
protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws IOException {
    String[] configLocations = this.getConfigLocations();
    if (configLocations != null) {
        String[] var3 = configLocations;
        int var4 = configLocations.length;

        for(int var5 = 0; var5 < var4; ++var5) {
            String configLocation = var3[var5];
            reader.loadBeanDefinitions(configLocation);
        }
    }
}
```

* `configLocations`数组是获取到的配置文件。
* 通过`loadBeanDefinitions`这个方法解析这个配置文件。

## 解析XML配置

首先把一个配置文件封装成一个`Resource`对象，然后获取`Resource`对象的输入流，转换成`InputSource`对象，最后解析成`Document`对象。

```java
public int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException {
        return this.loadBeanDefinitions(new EncodedResource(resource));
    }

    public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
        Assert.notNull(encodedResource, "EncodedResource must not be null");
        
        if (this.logger.isTraceEnabled()) {
            this.logger.trace("Loading XML bean definitions from " + encodedResource);
        }

        Set<EncodedResource> currentResources = (Set)this.resourcesCurrentlyBeingLoaded.get();
        if (!currentResources.add(encodedResource)) {
            throw new BeanDefinitionStoreException("Detected cyclic loading of " + encodedResource + " - check your import definitions!");
        } else {
            int var6;
            try {
                InputStream inputStream = encodedResource.getResource().getInputStream();
                Throwable var4 = null;

                try {
                    InputSource inputSource = new InputSource(inputStream);
                    if (encodedResource.getEncoding() != null) {
                        inputSource.setEncoding(encodedResource.getEncoding());
                    }

                    var6 = this.doLoadBeanDefinitions(inputSource, encodedResource.getResource());
                } catch (Throwable var24) {
                    var4 = var24;
                    throw var24;
                } finally {
                    if (inputStream != null) {
                        if (var4 != null) {
                            try {
                                inputStream.close();
                            } catch (Throwable var23) {
                                var4.addSuppressed(var23);
                            }
                        } else {
                            inputStream.close();
                        }
                    }

                }
            } catch (IOException var26) {
                throw new BeanDefinitionStoreException("IOException parsing XML document from " + encodedResource.getResource(), var26);
            } finally {
                currentResources.remove(encodedResource);
                if (currentResources.isEmpty()) {
                    this.resourcesCurrentlyBeingLoaded.remove();
                }
            }

            return var6;
        }
    }
    
    public int loadBeanDefinitions(InputSource inputSource) throws BeanDefinitionStoreException {
        return this.loadBeanDefinitions(inputSource, "resource loaded through SAX InputSource");
    }

    public int loadBeanDefinitions(InputSource inputSource, @Nullable String resourceDescription) throws BeanDefinitionStoreException {
        return this.doLoadBeanDefinitions(inputSource, new DescriptiveResource(resourceDescription));
    }
```

* xml配置文件解析成Document对象，它的Root节点信息如下:

```text
[   
    [#text:], 
    [context:component-scan: null], 
    [#text:], 
    [bean: null], 
    [#text:], 
    [bean: null], 
    [#text:], 
    [bean: null],
    [#text:], 
    [bean: null], 
    [#text:], 
    [#comment:  指定了表现层资源的前缀和后缀
        viewClass：JstlView表示JSP模板页面需要使用JSTL标签库
        prefix 和suffix：查找视图页面的前缀和后缀，比如传进来的逻辑视图名为hello，则该该
                        jsp视图页面应该存放在“WEB-INF/jsp/hello.jsp”], 
    [#text:], 
    [bean: null], 
    [#text: ]
]
```

## 加载Bean信息

```java
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
        if (delegate.isDefaultNamespace(root)) {
            NodeList nl = root.getChildNodes();

            for(int i = 0; i < nl.getLength(); ++i) {
                Node node = nl.item(i);
                if (node instanceof Element) {
                    Element ele = (Element)node;
                    if (delegate.isDefaultNamespace(ele)) {
                        this.parseDefaultElement(ele, delegate);
                    } else {
                        delegate.parseCustomElement(ele);
                    }
                }
            }
        } else {
            delegate.parseCustomElement(root);
        }
    }
```

### component-scan的解析

* 先调用自定义解析方法`BeanDefinitionParserDelegate`的`parseCustomElement(ele)`。最终调用了`ComponentScanBeanDefinitionParser`的`parse(Element element, ParserContext parserContext)`。Spring里面的一个配置文件，`/META-INF/spring.handlers`：

```text
http\://www.springframework.org/schema/context=org.springframework.context.config.ContextNamespaceHandler
http\://www.springframework.org/schema/jee=org.springframework.ejb.config.JeeNamespaceHandler
http\://www.springframework.org/schema/lang=org.springframework.scripting.config.LangNamespaceHandler
http\://www.springframework.org/schema/task=org.springframework.scheduling.config.TaskNamespaceHandler
http\://www.springframework.org/schema/cache=org.springframework.cache.config.CacheNamespaceHandler
```

* 这里面配置了不同标签的处理类，比如`context`标签处理类就是`ContextNamespaceHandler`。

```java
// DefaultNamespaceHandlerResolver.java
@Nullable
public NamespaceHandler resolve(String namespaceUri) {
    Map<String, Object> handlerMappings = this.getHandlerMappings();
    Object handlerOrClassName = handlerMappings.get(namespaceUri);
    if (handlerOrClassName == null) {
        return null;
    } else if (handlerOrClassName instanceof NamespaceHandler) {
        return (NamespaceHandler)handlerOrClassName;
    } else {
        String className = (String)handlerOrClassName;

        try {
            Class<?> handlerClass = ClassUtils.forName(className, this.classLoader);
            if (!NamespaceHandler.class.isAssignableFrom(handlerClass)) {
                throw new FatalBeanException("Class [" + className + "] for namespace [" + namespaceUri + "] does not implement the [" + NamespaceHandler.class.getName() + "] interface");
            } else {
                NamespaceHandler namespaceHandler = (NamespaceHandler)BeanUtils.instantiateClass(handlerClass);
                namespaceHandler.init();
                handlerMappings.put(namespaceUri, namespaceHandler);
                return namespaceHandler;
            }
        } catch (ClassNotFoundException var7) {
            throw new FatalBeanException("Could not find NamespaceHandler class [" + className + "] for namespace [" + namespaceUri + "]", var7);
        } catch (LinkageError var8) {
            throw new FatalBeanException("Unresolvable class definition for NamespaceHandler class [" + className + "] for namespace [" + namespaceUri + "]", var8);
        }
    }
}
```

* 然后通过反射实例化这个处理类，调用它的`init()`方法。`init()`方法里面它又注册了一堆处理类，其中就有`component-scan`。

```java
// ContextNamespaceHandler.java
public void init() {
    this.registerBeanDefinitionParser("property-placeholder", new PropertyPlaceholderBeanDefinitionParser());
    this.registerBeanDefinitionParser("property-override", new PropertyOverrideBeanDefinitionParser());
    this.registerBeanDefinitionParser("annotation-config", new AnnotationConfigBeanDefinitionParser());
    this.registerBeanDefinitionParser("component-scan", new ComponentScanBeanDefinitionParser());
    this.registerBeanDefinitionParser("load-time-weaver", new LoadTimeWeaverBeanDefinitionParser());
    this.registerBeanDefinitionParser("spring-configured", new SpringConfiguredBeanDefinitionParser());
    this.registerBeanDefinitionParser("mbean-export", new MBeanExportBeanDefinitionParser());
    this.registerBeanDefinitionParser("mbean-server", new MBeanServerBeanDefinitionParser());
}
```

* 最终Spring就可以通过`component-scan`这个标签，拿到`ComponentScanBeanDefinitionParser`类，调用它的`parse()`方法。

```java
@Nullable
public BeanDefinition parse(Element element, ParserContext parserContext) {
    String basePackage = element.getAttribute("base-package");
    basePackage = parserContext.getReaderContext().getEnvironment().resolvePlaceholders(basePackage);
    String[] basePackages = StringUtils.tokenizeToStringArray(basePackage, ",; \t\n");
    ClassPathBeanDefinitionScanner scanner = this.configureScanner(parserContext, element);
    Set<BeanDefinitionHolder> beanDefinitions = scanner.doScan(basePackages);
    this.registerComponents(parserContext.getReaderContext(), beanDefinitions, element);
    return null;
}
```

#### configureScanner配置扫描器

* 这里面重点就是注册了默认的过滤器。`use-default-filters`，默认值是true，如果配置文件配置了此属性的值为false，有些注解就加不进来，到下一步扫描的时候就注册不了Bean。

```java
protected void registerDefaultFilters() {
    this.includeFilters.add(new AnnotationTypeFilter(Component.class));
    ClassLoader cl = ClassPathScanningCandidateComponentProvider.class.getClassLoader();

    try {
        this.includeFilters.add(new AnnotationTypeFilter(ClassUtils.forName("javax.annotation.ManagedBean", cl), false));
        this.logger.trace("JSR-250 'javax.annotation.ManagedBean' found and supported for component scanning");
    } catch (ClassNotFoundException var4) {
    }

    try {
        this.includeFilters.add(new AnnotationTypeFilter(ClassUtils.forName("javax.inject.Named", cl), false));
        this.logger.trace("JSR-330 'javax.inject.Named' annotation found and supported for component scanning");
    } catch (ClassNotFoundException var3) {
    }
}
```

#### doScan扫描

doScan分为三个步骤：

* `findCandidateComponents`扫描包路径下的所有class文件，过滤有`Component`注解的类，转换成`BeanDefinition`对象，加入一个`LinkedHashSet`中。
* 循环上一步返回的`LinkedHashSet`，设置基本属性，比如`setLazyInit`、`setScope`。
* 注册`BeanDefinition`对象，向Map容器中缓存`beanName`和`BeanDefinition`，向List中加入`beanName`。

```java
// ClassPathBeanDefinitionScanner.java
protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
        Assert.notEmpty(basePackages, "At least one base package must be specified");
        Set<BeanDefinitionHolder> beanDefinitions = new LinkedHashSet();
        String[] var3 = basePackages;
        int var4 = basePackages.length;

        for(int var5 = 0; var5 < var4; ++var5) {
            String basePackage = var3[var5];
            Set<BeanDefinition> candidates = this.findCandidateComponents(basePackage);
            Iterator var8 = candidates.iterator();

            while(var8.hasNext()) {
                BeanDefinition candidate = (BeanDefinition)var8.next();
                ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(candidate);
                candidate.setScope(scopeMetadata.getScopeName());
                String beanName = this.beanNameGenerator.generateBeanName(candidate, this.registry);
                if (candidate instanceof AbstractBeanDefinition) {
                    this.postProcessBeanDefinition((AbstractBeanDefinition)candidate, beanName);
                }

                if (candidate instanceof AnnotatedBeanDefinition) {
                    AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition)candidate);
                }

                if (this.checkCandidate(beanName, candidate)) {
                    BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
                    definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
                    beanDefinitions.add(definitionHolder);
                    this.registerBeanDefinition(definitionHolder, this.registry);
                }
            }
        }

        return beanDefinitions;
    }
```

最后整个方法返回的就是`beanDefinition`对象的Set集合，以两个Controller为例。

```java
[   
    Generic bean: class [com.viewscenes.netsupervisor.controller.IndexController]; scope=; abstract=false; lazyInit=false; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null; defined in file [D:\apache-tomcat-7.0.78\webapps\springmvc_dubbo_producer\WEB-INF\classes\com\viewscenes\netsupervisor\controller\IndexController.class], 
    Generic bean: class [com.viewscenes.netsupervisor.controller.UserController]; scope=; abstract=false; lazyInit=false; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null; defined in file [D:\apache-tomcat-7.0.78\webapps\springmvc_dubbo_producer\WEB-INF\classes\com\viewscenes\netsupervisor\controller\UserController.class]
]
```

#### annotation-config的支持

在Spring配置文件有个配置是`context:annotation-config` 但如果配置了`context:component-scan` 就不必再配置config，这是因为在解析`component-scan`的时候已经默认添加了`annotation-config`的支持，除非你手动设置了`annotation-config="false"`，不过这可不太妙，因为在IOC的时候就没办法支持`@Autowired`等注解了。

```java
// ComponentScanBeanDefinitionParser.java
protected void registerComponents(XmlReaderContext readerContext, Set<BeanDefinitionHolder> beanDefinitions, Element element) {
        Object source = readerContext.extractSource(element);
        CompositeComponentDefinition compositeDef = new CompositeComponentDefinition(element.getTagName(), source);
        Iterator var6 = beanDefinitions.iterator();

        while(var6.hasNext()) {
            BeanDefinitionHolder beanDefHolder = (BeanDefinitionHolder)var6.next();
            compositeDef.addNestedComponent(new BeanComponentDefinition(beanDefHolder));
        }

        boolean annotationConfig = true;
        if (element.hasAttribute("annotation-config")) {
            annotationConfig = Boolean.parseBoolean(element.getAttribute("annotation-config"));
        }

        if (annotationConfig) {
            Set<BeanDefinitionHolder> processorDefinitions = AnnotationConfigUtils.registerAnnotationConfigProcessors(readerContext.getRegistry(), source);
            Iterator var8 = processorDefinitions.iterator();

            while(var8.hasNext()) {
                BeanDefinitionHolder processorDefinition = (BeanDefinitionHolder)var8.next();
                compositeDef.addNestedComponent(new BeanComponentDefinition(processorDefinition));
            }
        }

        readerContext.fireComponentRegistered(compositeDef);
    }
```

### bean标签的解析

获取bean标签的id，并且把beanName赋值为id，设置别名。新建AbstractBeanDefinition对象，通过反射设置beanClass，解析property属性名称和值。

```java
// BeanDefinitionParserDelegate.java
@Nullable
public BeanDefinitionHolder parseBeanDefinitionElement(Element ele, @Nullable BeanDefinition containingBean) {
    String id = ele.getAttribute("id");
    String nameAttr = ele.getAttribute("name");
    List<String> aliases = new ArrayList();
    if (StringUtils.hasLength(nameAttr)) {
        String[] nameArr = StringUtils.tokenizeToStringArray(nameAttr, ",; ");
        aliases.addAll(Arrays.asList(nameArr));
    }

    String beanName = id;
    if (!StringUtils.hasText(id) && !aliases.isEmpty()) {
        beanName = (String)aliases.remove(0);
        if (this.logger.isTraceEnabled()) {
            this.logger.trace("No XML 'id' specified - using '" + beanName + "' as bean name and " + aliases + " as aliases");
        }
    }

    if (containingBean == null) {
        this.checkNameUniqueness(beanName, aliases, ele);
    }

    AbstractBeanDefinition beanDefinition = this.parseBeanDefinitionElement(ele, beanName, containingBean);
    if (beanDefinition != null) {
        if (!StringUtils.hasText(beanName)) {
            try {
                if (containingBean != null) {
                    beanName = BeanDefinitionReaderUtils.generateBeanName(beanDefinition, this.readerContext.getRegistry(), true);
                } else {
                    beanName = this.readerContext.generateBeanName(beanDefinition);
                    String beanClassName = beanDefinition.getBeanClassName();
                    if (beanClassName != null && beanName.startsWith(beanClassName) && beanName.length() > beanClassName.length() && !this.readerContext.getRegistry().isBeanNameInUse(beanClassName)) {
                        aliases.add(beanClassName);
                    }
                }

                if (this.logger.isTraceEnabled()) {
                    this.logger.trace("Neither XML 'id' nor 'name' specified - using generated bean name [" + beanName + "]");
                }
            } catch (Exception var9) {
                this.error(var9.getMessage(), ele);
                return null;
            }
        }

        String[] aliasesArray = StringUtils.toStringArray(aliases);
        return new BeanDefinitionHolder(beanDefinition, beanName, aliasesArray);
    } else {
        return null;
    }
}
```

* 注册`BeanDefinition`对象，和`component-scan`扫描的bean注册一样。向容器中填充对象。
* 不管是XML配置的Bean，还是通过`component-scan`扫描注册的Bean它们最后都是殊途同归的，会转换成一个`BeanDefinition`对象。记录着这个Bean对象的属性和方法，最后都注册到容器中，等待在实例化和IOC的时候遍历它们。

