## dubbo 源码



![img](https://img-blog.csdnimg.cn/20200416211353216.png)

## 基于 spring 配置文件的扩展

NamespaceHandler：注册 BeanDefinitionParser，利用它来解析 BeanDefinitionParser：解析配置文件的元素 spring 会默认加载jar包下/META-INF/spring.handlers，找到对应的 NamespaceHandler

## Dubbo 的接入实现

Dubbo中spring扩展就是使用spring的自定义类型，所以同样也有NamespaceHandler、BeanDefinitionParser。而NamespaceHandler是DubboNamespaceHandler。

```java
public class DubboNamespaceHandler extends NamespaceHandlerSupport implements ConfigurableSourceBeanMetadataElement {

    static {
        Version.checkDuplicate(DubboNamespaceHandler.class);
    }

    @Override
    public void init() {
        registerBeanDefinitionParser("application", new DubboBeanDefinitionParser(ApplicationConfig.class, true));
        registerBeanDefinitionParser("module", new DubboBeanDefinitionParser(ModuleConfig.class, true));
        registerBeanDefinitionParser("registry", new DubboBeanDefinitionParser(RegistryConfig.class, true));
        registerBeanDefinitionParser("config-center", new DubboBeanDefinitionParser(ConfigCenterBean.class, true));
        registerBeanDefinitionParser("metadata-report", new DubboBeanDefinitionParser(MetadataReportConfig.class, true));
        registerBeanDefinitionParser("monitor", new DubboBeanDefinitionParser(MonitorConfig.class, true));
        registerBeanDefinitionParser("metrics", new DubboBeanDefinitionParser(MetricsConfig.class, true));
        registerBeanDefinitionParser("ssl", new DubboBeanDefinitionParser(SslConfig.class, true));
        registerBeanDefinitionParser("provider", new DubboBeanDefinitionParser(ProviderConfig.class, true));
        registerBeanDefinitionParser("consumer", new DubboBeanDefinitionParser(ConsumerConfig.class, true));
        registerBeanDefinitionParser("protocol", new DubboBeanDefinitionParser(ProtocolConfig.class, true));
        registerBeanDefinitionParser("service", new DubboBeanDefinitionParser(ServiceBean.class, true));
        registerBeanDefinitionParser("reference", new DubboBeanDefinitionParser(ReferenceBean.class, false));
        registerBeanDefinitionParser("annotation", new AnnotationBeanDefinitionParser());
    }

    @Override
    public BeanDefinition parse(Element element, ParserContext parserContext) {
        BeanDefinitionRegistry registry = parserContext.getRegistry();
        registerAnnotationConfigProcessors(registry);
        registerApplicationListeners(registry);
        BeanDefinition beanDefinition = super.parse(element, parserContext);
        setSource(beanDefinition);
        return beanDefinition;
    }

    private void registerApplicationListeners(BeanDefinitionRegistry registry) {
        registerBeans(registry, DubboLifecycleComponentApplicationListener.class);
        registerBeans(registry, DubboBootstrapApplicationListener.class);
    }

    private void registerAnnotationConfigProcessors(BeanDefinitionRegistry registry) {
        AnnotationConfigUtils.registerAnnotationConfigProcessors(registry);
    }
}
```

BeanDefinitionParser全部都使用了DubboBeanDefinitionParser，如果我们想看<dubbo:service>的配置，就直接看DubboBeanDefinitionParser中。这个里面主要做了一件事，把不同的配置分别转化成spring容器中的bean对象： application对应ApplicationConfig registry对应RegistryConfig monitor对应MonitorConfig provider对应ProviderConfig consumer对应ConsumerConfig …

为了在spring启动的时候，也相应的启动provider发布服务注册服务的过程，而同时为了让客户端在启动的时候自动订阅发现服务，加入了两个bean：ServiceBean、ReferenceBean，分别继承了ServiceConfig和ReferenceConfig。同时还分别实现了InitializingBean、DisposableBean, ApplicationContextAware, ApplicationListener, BeanNameAware。 InitializingBean接口为bean提供了初始化方法的方式，它只包括afterPropertiesSet方法，凡是继承该接口的类，在初始化bean的时候会执行该方法。 DisposableBean bean被销毁的时候，spring容器会自动执行destory方法，比如释放资源 ApplicationContextAware 实现了这个接口的bean，当spring容器初始化的时候，会自动的将ApplicationContext注入进来 ApplicationListener ApplicationEvent事件监听，spring容器启动后会发一个事件通知 BeanNameAware 获得自身初始化时，本身的bean的id属性

那么基本的实现思路可以整理出来了：

1. 利用spring的解析收集xml中的配置信息，然后把这些配置信息存储到serviceConfig中 
2. 调用ServiceConfig的export方法来进行服务的发布和注册

## 服务的发布

### ServiceBean

ServiceBean是服务发布的切入点，通过afterPropertiesSet方法，调用export()方法进行发布。 export为父类ServiceConfig中的方法，所以跳转到SeviceConfig类中的export方法。

### delay的使用

```java
private static final ScheduledExecutorService DELAY_EXPORT_EXECUTOR = Executors.newSingleThreadScheduledExecutor(new NamedThreadFactory("DubboServiceDelayExporter", true));

public boolean shouldDelay() {
	Integer delay = getDelay();
	return delay != null && delay > 0;
}
public synchronized void export() {
    if (!shouldExport()) {
        return;
    }
    if (bootstrap == null) {
        bootstrap = DubboBootstrap.getInstance();
        bootstrap.init();
    }
    checkAndUpdateSubConfigs();
    //init serviceMetadata
    serviceMetadata.setVersion(version);
    serviceMetadata.setGroup(group);
    serviceMetadata.setDefaultGroup(group);
    serviceMetadata.setServiceType(getInterfaceClass());
    serviceMetadata.setServiceInterfaceName(getInterface());
    serviceMetadata.setTarget(getRef());
    if (shouldDelay()) {
        DELAY_EXPORT_EXECUTOR.schedule(this::doExport, getDelay(), TimeUnit.MILLISECONDS);
    } else {
        doExport();
    }
    exported();
}
```

delay的作用就是延迟暴露，而延迟的方式也很直截了当，Thread.sleep(delay)

1. export是synchronized修饰的方法。也就是说暴露的过程是原子操作，正常情况下不会出现锁竞争的问题，毕竟初始化过程大多数情况下都是单一线程操作，这里联想到了spring的初始化流程，也进行了加锁操作，这里也给我们平时设计一个不错的启示：初始化流程的性能调优优先级应该放的比较低，但是安全的优先级应该放的比较高！ 
2. 继续看doExport()方法。同样是一堆初始化代码

### export的过程

```java
private void doExportUrls() {
    ...
    List<URL> registryURLs = ConfigValidationUtils.loadRegistries(this, true);
    for (ProtocolConfig protocolConfig : protocols) {
        ...
        doExportUrlsFor1Protocol(protocolConfig, registryURLs);
    }
}
```

最终的实现逻辑：

```java
private static final ProxyFactory PROXY_FACTORY = ExtensionLoader.getExtensionLoader(ProxyFactory.class).getAdaptiveExtension();
private static final Protocol PROTOCOL = ExtensionLoader.getExtensionLoader(Protocol.class).getAdaptiveExtension();

private void doExportUrlsFor1Protocol(ProtocolConfig protocolConfig, List<URL> registryURLs) {
	...
	Invoker<?> invoker = PROXY_FACTORY.getInvoker(ref, (Class) interfaceClass, 	registryURL.addParameterAndEncoded(EXPORT_KEY, url.toFullString()));
    DelegateProviderMetaDataInvoker wrapperInvoker = new DelegateProviderMetaDataInvoker(invoker, this);

	Exporter<?> exporter = PROTOCOL.export(wrapperInvoker);
	exporters.add(exporter);
	...
}
```

在上面这段代码中可以看到Dubbo的比较核心的抽象：Invoker， Invoker是一个代理类，从ProxyFactory中生成。这个地方可以做一个小结:

1. Invoker 执行具体的远程调用 
2. Protocol 服务地址的发布和订阅 
3. Exporter 暴露服务或取消暴露

### protocol发布服务

dubboProtocol的export方法：openServer(url），接着调用openServer， 继续createServer 创建服务，继续看其中的createServer方法：

```java
private ExchangeHandler requestHandler = new ExchangeHandlerAdapter() {
	...
}
...
ExchangeServer server;
try {
    server = Exchangers.bind(url, requestHandler);
} catch (RemotingException e) {
    throw new RpcException("Fail to start server(url: " + url + ") " + e.getMessage(), e);
}
```

发现ExchangeServer是通过Exchangers创建的，直接看Exchanger.bind方法：

```java
public static ExchangeServer bind(URL url, ExchangeHandler handler) throws RemotingException {
    if (url == null) {
        throw new IllegalArgumentException("url == null");
    }
    if (handler == null) {
        throw new IllegalArgumentException("handler == null");
    }
    url = url.addParameterIfAbsent(Constants.CODEC_KEY, "exchange");
    return getExchanger(url).bind(url, handler);
}
```

getExchanger方法实际上调用的是ExtensionLoader的相关方法，这里的ExtensionLoader是dubbo插件化的核心，我们会在后面的插件化讲解中详细讲解，这里我们只需要知道Exchanger的默认实现只有一个：HeaderExchanger。上面一段代码最终调用的是：

```java
public ExchangeServer bind(URL url, ExchangeHandler handler) throws RemotingException {
	return new HeaderExchangeServer(Transporters.bind(url, new DecodeHandler(new HeaderExchangeHandler(handler))));
}
```

可以看到Server与Client实例均是在这里创建的，HeaderExchangeServer需要一个Server类型的参数，来自Transporters.bind()：

```java
public static RemotingServer bind(URL url, ChannelHandler... handlers) throws RemotingException {
	...
    return getTransporter().bind(url, handler);
}
```

getTransporter()获取的实例来源于配置，默认返回一个NettyTransporter：

```java
public RemotingServer bind(URL url, ChannelHandler handler) throws RemotingException {
    return new NettyServer(url, handler);
}
```

## 服务消费

### ReferenceBean

和serviceBean发布一样，也是使用NamespaceHandler作为切入点，调用ReferenceBean里面的afterPropertiesSet方法 方法调用顺序afterPropertiesSet() -> getObject() -> get() -> init() -> createProxy() afterPropertiesSet方法中都是确认所有的组件是否都初始化好了，都准备好后我们进入生成Invoker的部分。这里的getObject会调用父类ReferenceConfig的init方法完成组装：

```java
public void checkAndUpdateSubConfigs() {
        // 如果interfaceName不存在
        if (StringUtils.isEmpty(interfaceName)) {
            throw new IllegalStateException("<dubbo:reference interface=\"\" /> interface not allow null!");
        }
        completeCompoundConfigs(consumer);
        if (consumer != null) {
            if (StringUtils.isEmpty(registryIds)) {
                setRegistryIds(consumer.getRegistryIds());
            }
        }
        // 获取消费者
        checkDefault();
        this.refresh();
        if (getGeneric() == null && getConsumer() != null) {
            setGeneric(getConsumer().getGeneric());
        }
        // 如果未使用泛接口并且consumer已经准备好的情况下，reference使用和consumer一样的泛接口
        if (ProtocolUtils.isGeneric(generic)) {
            interfaceClass = GenericService.class;
        } else {
            // 如果不是泛接口使用interfaceName指定的泛接口
            try {
                interfaceClass = Class.forName(interfaceName, true, Thread.currentThread()
                        .getContextClassLoader());
            } catch (ClassNotFoundException e) {
                throw new IllegalStateException(e.getMessage(), e);
            }
            // 检查接口以及接口中的方法是否配置齐全
            checkInterfaceAndMethods(interfaceClass, getMethods());
        }

        //init serivceMetadata
        serviceMetadata.setVersion(version);
        serviceMetadata.setGroup(group);
        serviceMetadata.setDefaultGroup(group);
        serviceMetadata.setServiceType(getActualInterface());
        serviceMetadata.setServiceInterfaceName(interfaceName);
        // TODO, uncomment this line once service key is unified
        serviceMetadata.setServiceKey(URL.buildKey(interfaceName, group, version));

        ServiceRepository repository = ApplicationModel.getServiceRepository();
        ServiceDescriptor serviceDescriptor = repository.registerService(interfaceClass);
        repository.registerConsumer(
                serviceMetadata.getServiceKey(),
                serviceDescriptor,
                this,
                null,
                serviceMetadata);
        // 如果服务比较多可以指定dubbo-resolve.properties文件配置service(service集中配置文件)
        resolveFile();
        ConfigValidationUtils.validateReferenceConfig(this);
        postProcessConfig();
    }
public synchronized void init() {
        // 避免重复初始化
        if (initialized) {
            return;
        }

        if (bootstrap == null) {
            bootstrap = DubboBootstrap.getInstance();
            bootstrap.init();
        }
		// 检查远程和本地服务接口真实存在（是否可load）
        checkAndUpdateSubConfigs();

        checkStubAndLocal(interfaceClass);
        ConfigValidationUtils.checkMock(interfaceClass, this);
		// 配置dubbo的端属性（是consumer还是provider），版本属性，创建时间，进程号
        Map<String, String> map = new HashMap<String, String>();
        map.put(SIDE_KEY, CONSUMER_SIDE);

        ReferenceConfigBase.appendRuntimeParameters(map);
        if (!ProtocolUtils.isGeneric(generic)) {
            String revision = Version.getVersion(interfaceClass, version);
            if (revision != null && revision.length() > 0) {
                map.put(REVISION_KEY, revision);
            }

            String[] methods = Wrapper.getWrapper(interfaceClass).getMethodNames();
            if (methods.length == 0) {
                logger.warn("No method found in service interface " + interfaceClass.getName());
                map.put(METHODS_KEY, ANY_VALUE);
            } else {
                map.put(METHODS_KEY, StringUtils.join(new HashSet<String>(Arrays.asList(methods)), COMMA_SEPARATOR));
            }
        }
        map.put(INTERFACE_KEY, interfaceName);
        // 调用application,module,consumer的get方法将属性设置到map中
        AbstractConfig.appendParameters(map, getMetrics());
        AbstractConfig.appendParameters(map, getApplication());
        AbstractConfig.appendParameters(map, getModule());
        // remove 'default.' prefix for configs from ConsumerConfig
        // appendParameters(map, consumer, Constants.DEFAULT_KEY);
        AbstractConfig.appendParameters(map, consumer);
        AbstractConfig.appendParameters(map, this);
        MetadataReportConfig metadataReportConfig = getMetadataReportConfig();
        if (metadataReportConfig != null && metadataReportConfig.isValid()) {
            map.putIfAbsent(METADATA_KEY, REMOTE_METADATA_STORAGE_TYPE);
        }
        Map<String, AsyncMethodInfo> attributes = null;
        if (CollectionUtils.isNotEmpty(getMethods())) {
            attributes = new HashMap<>();
            for (MethodConfig methodConfig : getMethods()) {
                AbstractConfig.appendParameters(map, methodConfig, methodConfig.getName());
                String retryKey = methodConfig.getName() + ".retry";
                if (map.containsKey(retryKey)) {
                    String retryValue = map.remove(retryKey);
                    if ("false".equals(retryValue)) {
                        map.put(methodConfig.getName() + ".retries", "0");
                    }
                }
                AsyncMethodInfo asyncMethodInfo = AbstractConfig.convertMethodConfig2AsyncInfo(methodConfig);
                if (asyncMethodInfo != null) {
//                    consumerModel.getMethodModel(methodConfig.getName()).addAttribute(ASYNC_KEY, asyncMethodInfo);
                    attributes.put(methodConfig.getName(), asyncMethodInfo);
                }
            }
        }

        String hostToRegistry = ConfigUtils.getSystemProperty(DUBBO_IP_TO_REGISTRY);
        if (StringUtils.isEmpty(hostToRegistry)) {
            hostToRegistry = NetUtils.getLocalHost();
        } else if (isInvalidLocalHost(hostToRegistry)) {
            throw new IllegalArgumentException("Specified invalid registry ip from property:" + DUBBO_IP_TO_REGISTRY + ", value:" + hostToRegistry);
        }
        map.put(REGISTER_IP_KEY, hostToRegistry);
 		// attributes通过系统context进行存储
        serviceMetadata.getAttachments().putAll(map);
		// 在map装载了application,module,consumer,reference的所有属性信息后创建代理
        ref = createProxy(map);

        serviceMetadata.setTarget(ref);
        serviceMetadata.addAttribute(PROXY_CLASS_REF, ref);
        ConsumerModel consumerModel = repository.lookupReferredService(serviceMetadata.getServiceKey());
        consumerModel.setProxyObject(ref);
        consumerModel.init(attributes);
        // 置为已经初始化
        initialized = true;

        // dispatch a ReferenceConfigInitializedEvent since 2.7.4
        dispatch(new ReferenceConfigInitializedEvent(this, invoker));
    }
```

#### createProxy 方法

```java
protected boolean shouldJvmRefer(Map<String, String> map) {
    // 1.创建临时URL
    URL tmpUrl = new URL("temp", "localhost", 0, map);
    boolean isJvmRefer;
    if (isInjvm() == null) {
        // if a url is specified, don't do local reference
        if (url != null && url.length() > 0) {
            isJvmRefer = false;
        } else {
            // by default, reference local service if there is
            isJvmRefer = InjvmProtocol.getInjvmProtocol().isInjvmRefer(tmpUrl);
        }
    } else {
        isJvmRefer = isInjvm();
    }
    return isJvmRefer;
}
private T createProxy(Map<String, String> map) {
    // 2.判断是否暴露本地服务
    if (shouldJvmRefer(map)) {
        URL url = new URL(LOCAL_PROTOCOL, LOCALHOST_VALUE, 0, interfaceClass.getName()).addParameters(map);
        invoker = REF_PROTOCOL.refer(interfaceClass, url);
        if (logger.isInfoEnabled()) {
            logger.info("Using injvm service " + interfaceClass.getName());
        }
    } else {
        urls.clear();
        // 3.判断用户指定URL，指定的URL可能是点对点直连地址，也可能是注册中西URL
        if (url != null && url.length() > 0) {
            String[] us = SEMICOLON_SPLIT_PATTERN.split(url);
            if (us != null && us.length > 0) {
                for (String u : us) {
                    URL url = URL.valueOf(u);
                    if (StringUtils.isEmpty(url.getPath())) {
                        url = url.setPath(interfaceName);
                    }
                    if (UrlUtils.isRegistry(url)) {
                        urls.add(url.addParameterAndEncoded(REFER_KEY, StringUtils.toQueryString(map)));
                    } else {
                        urls.add(ClusterUtils.mergeUrl(url, map));
                    }
                }
            }
        } else {
            // 4.通过注册中心配置拼装URL
            if (!LOCAL_PROTOCOL.equalsIgnoreCase(getProtocol())) {
                checkRegistry();
                List<URL> us = ConfigValidationUtils.loadRegistries(this, false);
                if (CollectionUtils.isNotEmpty(us)) {
                    for (URL u : us) {
                        URL monitorUrl = ConfigValidationUtils.loadMonitor(this, u);
                        if (monitorUrl != null) {
                            map.put(MONITOR_KEY, URL.encode(monitorUrl.toFullString()));
                        }
                        urls.add(u.addParameterAndEncoded(REFER_KEY, StringUtils.toQueryString(map)));
                    }
                }
                if (urls.isEmpty()) {
                    throw new IllegalStateException("No such any registry to reference " + interfaceName + " on the consumer " + NetUtils.getLocalHost() + " use dubbo 
                }
            }
        }
        // 5.调用refprotocol.refer()
        if (urls.size() == 1) {
            invoker = REF_PROTOCOL.refer(interfaceClass, urls.get(0));
        } else {
            List<Invoker<?>> invokers = new ArrayList<Invoker<?>>();
            URL registryURL = null;
            for (URL url : urls) {
                invokers.add(REF_PROTOCOL.refer(interfaceClass, url));
                if (UrlUtils.isRegistry(url)) {
                    registryURL = url; // 用了最后一个registry url
                }
            }
            if (registryURL != null) { // 有注册中心协议的URL
                // 对有注册中心的Cluster 只用 ZoneAwareCluster
                URL u = registryURL.addParameterIfAbsent(CLUSTER_KEY, ZoneAwareCluster.NAME);
                invoker = CLUSTER.join(new StaticDirectory(u, invokers));
            } else { // 不是注册中心URL
                invoker = CLUSTER.join(new StaticDirectory(invokers));
            }
        }
    }
    ...
    // 6.创建服务代理
    return (T) PROXY_FACTORY.getProxy(invoker, ProtocolUtils.isGeneric(generic));
}
```

#### refprotocol.refer()

refprotocol.refer()先后经过修饰类 ProtocolFilterWrapper 、ProtocolListenerWrapper，最后执行RegistryPotocol。 ProtocolFilterWrapper -&gt; ProtocolListenerWrapper：

```java
// ProtocolFilterWrapper 
public <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException {
    if (UrlUtils.isRegistry(url)) {
        return protocol.refer(type, url);
    }
    return buildInvokerChain(protocol.refer(type, url), REFERENCE_FILTER_KEY, CommonConstants.CONSUMER);
}
// ProtocolListenerWrapper
public <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException {
    if (UrlUtils.isRegistry(url)) {
        return protocol.refer(type, url);
    }
    return new ListenerInvokerWrapper<T>(protocol.refer(type, url),
            Collections.unmodifiableList(
                    ExtensionLoader.getExtensionLoader(InvokerListener.class)
                            .getActivateExtension(url, INVOKER_LISTENER_KEY)));
}
```

#### RegistryProtocol

```java
public <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException {
    url = getRegistryUrl(url);
    Registry registry = registryFactory.getRegistry(url);
    if (RegistryService.class.equals(type)) {
        return proxyFactory.getInvoker((T) registry, type, url);
    }
    // 配置Group信息
    // group="a,b" or group="*"
    Map<String, String> qs = StringUtils.parseQueryString(url.getParameterAndDecoded(REFER_KEY));
    String group = qs.get(GROUP_KEY);
    if (group != null && group.length() > 0) {
        if ((COMMA_SPLIT_PATTERN.split(group)).length > 1 || "*".equals(group)) {
            return doRefer(getMergeableCluster(), registry, type, url);
        }
    }
    return doRefer(cluster, registry, type, url);
}
```

至此Reference在关联了所有application、module、consumer、registry、monitor、service、protocol后调用对应Protocol类的refer方法生成InvokerProxy。当用户调用service时dubbo会通过InvokerProxy调用Invoker的invoke的方法向服务端发起请求。客户端就这样完成了自己的初始化。

这个代理实例中仅仅包含一个handler对象（InvokerInvocationHandler类的实例），handler中则包含了RPC调用中非常核心的一个接口Invoker<T>的实现，Invoker接口的的的定义如下：

```java
public interface Invoker<T> extends Node {     
	Class<T> getInterface();  //调用过程的具体表示形式      
	Result invoke(Invocation invocation) throws RpcException;
}
```

Invoker<T>接口的核心方法是invoke(Invocation invocation)，方法的参数Invocation是一个调用过程的抽象，也是Dubbo框架的核心接口，该接口中包含如何获取调用方法的名称、参数类型列表、参数列表以及绑定的数据，定义代码如下：

```java
public interface Invocation {
    String getTargetServiceUniqueName();
    // 调用的方法名字
    String getMethodName();
    // 调用方法的参数的类型列表
    Class<?>[] getParameterTypes();
    // 调用方法的参数列表
    Object[] getArguments();
    // 调用时附加的数据，用map存储
    Map<String, String> getAttachments();
    // 根据key来获取附加的数据
    String getAttachment(String key);
    // getAttachment扩展，支持默认值获取
    String getAttachment(String key, String defaultValue);
    // 获取真实的调用者实现
    Invoker<?> getInvoker();
}
```

代理中的handler实例中包含的Invoker<T>接口实现者是MockClusterInvoker，其中MockClusterInvoker仅仅是一个Invoker的包装，并且也实现了接口Invoker<T>，其只是用于实现Dubbo框架中的mock功能，我们可以从他的invoke方法的实现中看出。


Dubbo的插件化实现非常类似于原生的JAVA的SPI：它只是提供一种协议，并没有提供相关插件化实施的接口。用过的同学都知道，它有一种java原生的支持类：ServiceLoader，通过声明接口的实现类，在META-INF/services中注册一个实现类，然后通过ServiceLoader去生成一个接口实例，当更换插件的时候只需要把自己实现的插件替换到META-INF/services中即可。

## Dubbo的 SPI


Dubbo的SPI并非原生的SPI，Dubbo的规则是在META-INF/dubbo、META-INF/dubbo/internal或者META-INF/services下面以需要实现的接口去创建一个文件，并且在文件中以properties规则一样配置实现类的全面以及分配实现的一个名称。我们看一下dubbo-cluster模块的META-INF.dubbo.internal： ![img](https://img-blog.csdnimg.cn/20200526171640228.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

## 实现自己的扩展点

1. 在resources目录下新建META-INF/dubbo/com.alibaba.dubbo.rpc.Protocol文件，文件内容为com.***.MyProtocol 
2. 实现类的内容

```java
public class MyProtocol implements Protocol {
    @Override
    public int getDefaultPort() {
        return 99999;
    }
    ...
}
```

1. 最后在main方法中调用

```java
public class Bootstrap {
    public static void main(String[] args) {
        Protocol protocol = ExtensionLoader.getExtensionLoader(Protocol.class).getExtension("myprotocol");
        System.out.println(protocol.getDefaultPort());
    }
}
```

## 源码分析

dubbo的扩展点框架主要位于这个包下：org.alibaba.dubbo.common.extension

```java
1.	org.alibaba.dubbo.common.extension  
2.	 |  
3.	 |--factory  
4.	 |     |--AdaptiveExtensionFactory   #稍后解释  
5.	 |     |--SpiExtensionFactory        #稍后解释  
6.	 |  
7.	 |--support  
8.	 |     |--ActivateComparator  
9.	 |  
10.	 |--Activate  #自动激活加载扩展的注解  
11.	 |--Adaptive  #自适应扩展点的注解  
12.	 |--ExtensionFactory  #扩展点对象生成工厂接口  
13.	 |--ExtensionLoader   #扩展点加载器，扩展点的查找，校验，加载等核心逻辑的实现类  
14.	 |--SPI   #扩展点注解
```

其中最核心的类就是ExtensionLoader，几乎所有特性都在这个类中实现。 ExtensionLoader没有提供public的构造方法，但是提供了一个public static的getExtensionLoader，这个方法就是获取ExtensionLoader实例的工厂方法。其public成员方法中有三个比较重要的方法：

- getActivateExtension ：根据条件获取当前扩展可自动激活的实现 
- getExtension ： 根据名称获取当前扩展的指定实现 
- getAdaptiveExtension : 获取当前扩展的自适应实现

```java
private static final ConcurrentMap<Class<?>, ExtensionLoader<?>> EXTENSION_LOADERS = new ConcurrentHashMap<>(64);

    public static <T> ExtensionLoader<T> getExtensionLoader(Class<T> type) {
        if (type == null) {
            throw new IllegalArgumentException("Extension type == null");
        }
        if (!type.isInterface()) {
            throw new IllegalArgumentException("Extension type (" + type + ") is not an interface!");
        }
        // 只接受使用@SPI注解注释的接口
        if (!withExtensionAnnotation(type)) {
            throw new IllegalArgumentException("Extension type (" + type +
                    ") is not an extension, because it is NOT annotated with @" + SPI.class.getSimpleName() + "!");
        }
        // 静态缓存中获取对应的ExtensionLoader实例,如果不存在就为Extension类型创建ExtensionLoader的实例，并放入静态缓存
        return (ExtensionLoader<T>) EXTENSION_LOADERS.computeIfAbsent(type, k -> new ExtensionLoader<T>(type));
    }
```

该方法需要一个Class类型的参数，该参数表示希望加载的扩展点类型，该参数必须是接口，且该接口必须被@SPI注解注释，否则拒绝处理。检查通过之后首先会检查ExtensionLoader缓存中是否已经存在该扩展对应的ExtensionLoader，如果有则直接返回，否则创建一个新的ExtensionLoader负责加载该扩展实现，同时将其缓存起来。可以看到对于每一个扩展，dubbo中只会有一个对应的ExtensionLoader实例。

接下来看下ExtensionLoader的私有构造函数：

```java
private ExtensionLoader(Class<?> type) {
    this.type = type;
    // 如果扩展类型是ExtensionFactory，那么则设置为null
    // 这里通过getExtensionLoader方法获取一个运行时自适应的扩展类型（每个Extension只能有一个@Adaptive类型的实现，如果没有dubbo会动态生成一个类）
    objectFactory = (type == ExtensionFactory.class ? null : ExtensionLoader.getExtensionLoader(ExtensionFactory.class).getAdaptiveExtension());
}
```

这里保存了对应的扩展类型，并且设置了一个额外的objectFactory属性，他是一个ExtensionFactory类型，ExtensionFactory主要用于加载扩展的实现：

```java
@SPI
public interface ExtensionFactory {
    <T> T getExtension(Class<T> type, String name);
}
```

ExtensionFactory有@SPI注解，说明当前这个接口是一个扩展点。从extension包的结构图可以看到。Dubbo内部提供了两个实现类：SpiExtensionFactory和AdaptiveExtensionFactory。不同的实现可以以不同的方式来完成扩展点实现的加载。

默认的ExtensionFactory实现中，AdaptiveExtensionFactotry被@Adaptive注解注释，也就是它就是ExtensionFactory对应的自适应扩展实现(每个扩展点最多只能有一个自适应实现，如果所有实现中没有被@Adaptive注释的，那么dubbo会动态生成一个自适应实现类)，也就是说，所有对ExtensionFactory调用的地方，实际上调用的都是AdpativeExtensionFactory，那么我们看下他的实现代码：

```java
@Adaptive
public class AdaptiveExtensionFactory implements ExtensionFactory {

    private final List<ExtensionFactory> factories;

    public AdaptiveExtensionFactory() {
        ExtensionLoader<ExtensionFactory> loader = ExtensionLoader.getExtensionLoader(ExtensionFactory.class);
        List<ExtensionFactory> list = new ArrayList<ExtensionFactory>();
        for (String name : loader.getSupportedExtensions()) {
            // 将所有的ExtensionFactory实现保存起来
            list.add(loader.getExtension(name));
        }
        factories = Collections.unmodifiableList(list);
    }

    @Override
    public <T> T getExtension(Class<T> type, String name) {
        // 依次遍历各个ExtensionFactory实现的getExtension方法，一旦获取到Extension即返回
        // 如果遍历完所有的ExtensionFactory实现均无法找到Extension，则返回null
        for (ExtensionFactory factory : factories) {
            T extension = factory.getExtension(type, name);
            if (extension != null) {
                return extension;
            }
        }
        return null;
    }

}
```

这段代码，其实就相当于一个代理入口，它会遍历当前系统中所有的ExtensionFactory实现来获取指定的扩展实现，获取到扩展实现，遍历完所有ExtensionFactory实现，调用ExtensionLoader的getSupportedExtensions方法来获取ExtensionFactory的所有实现。

从前面ExtensionLoader的私有构造函数中可以看出，在选择ExtensionFactory的时候，并不是调用getExtension(name)来获取某个具体的实现类，而是调用getAdaptiveExtension来获取一个自适应的实现。那么首先我们就来分析一下getAdaptiveExtension这个方法的实现：

```java
public T getAdaptiveExtension() {
        // 首先判断是否已经有缓存的实例对象
        Object instance = cachedAdaptiveInstance.get();
        if (instance == null) {
            if (createAdaptiveInstanceError != null) {
                throw new IllegalStateException("Failed to create adaptive instance: " +
                        createAdaptiveInstanceError.toString(),
                        createAdaptiveInstanceError);
            }

            synchronized (cachedAdaptiveInstance) {
                instance = cachedAdaptiveInstance.get();
                if (instance == null) {
                    try {
                        // 没有缓存的实例，创建新的AdaptiveExtension实例
                        instance = createAdaptiveExtension();
                        cachedAdaptiveInstance.set(instance);
                    } catch (Throwable t) {
                        createAdaptiveInstanceError = t;
                        throw new IllegalStateException("Failed to create adaptive instance: " + t.toString(), t);
                    }
                }
            }
        }

        return (T) instance;
    }
```

首先检查缓存的adaptiveInstance是否存在，如果存在则直接使用，否则的话调用createAdaptiveExtension方法来创建新的adaptiveInstance并且缓存起来。也就是说对于某个扩展点，每次调用ExtensionLoader.getAdaptiveExtension获取到的都是同一个实例。

### createAdaptiveExtension方法

```java
private T createAdaptiveExtension() {
        try {
            // 先获取AdaptiveExtensionClass，在获取其实例，最后进行注入处理
            return injectExtension((T) getAdaptiveExtensionClass().newInstance());
        } catch (Exception e) {
            throw new IllegalStateException("Can't create adaptive extension " + type + ", cause: " + e.getMessage(), e);
        }
    }
```

在createAdaptiveExtension方法中，首先通过getAdaptiveExtensionClass方法获取到最终的自适应实现类型，然后实例化一个自适应扩展实现的实例，最后进行扩展点注入操作。

```java
private Class<?> getAdaptiveExtensionClass() {
    // 加载当前Extension的所有实现，如果有@Adaptive类型，则会赋值为cachedAdaptiveClass属性缓存起来
    getExtensionClasses();
    if (cachedAdaptiveClass != null) {
        return cachedAdaptiveClass;
    }
    // 没有找到@Adaptive类型实现，则动态创建一个AdaptiveClass
    return cachedAdaptiveClass = createAdaptiveExtensionClass();
}
```

他只是简单的调用了getExtensionClasses方法，然后在判adaptiveCalss缓存是否被设置，如果被设置那么直接返回，否则调用createAdaptiveExntesionClass方法动态生成一个自适应实现，关于动态生成自适应实现类然后编译加载并且实例化。

先看getExtensionClasses方法：

```java
private Map<String, Class<?>> getExtensionClasses() {
        Map<String, Class<?>> classes = cachedClasses.get();
        // 判断是否已经加载了当前Extension的所有实现类
        if (classes == null) {
            synchronized (cachedClasses) {
                classes = cachedClasses.get();
                if (classes == null) {
                    // 如果还没有加载Extension的实现，则进行扫描加载，完成后赋值给cachedClasses
                    classes = loadExtensionClasses();
                    cachedClasses.set(classes);
                }
            }
        }
        return classes;
    }
```

在getExtensionClasses方法中，首先检查缓存的cachedClasses，如果没有再调用loadExtensionClasses方法来加载，加载完成之后就会进行缓存。也就是说对于每个扩展点，其实现的加载只会执行一次。我们看下loadExtensionClasses方法：

```java
private static final String SERVICES_DIRECTORY = "META-INF/services/";
    private static final String DUBBO_DIRECTORY = "META-INF/dubbo/";
    private static final String DUBBO_INTERNAL_DIRECTORY = DUBBO_DIRECTORY + "internal/";
    private static LoadingStrategy DUBBO_INTERNAL_STRATEGY =  () -> DUBBO_INTERNAL_DIRECTORY;
    private static LoadingStrategy DUBBO_STRATEGY = () -> DUBBO_DIRECTORY;
    private static LoadingStrategy SERVICES_STRATEGY = () -> SERVICES_DIRECTORY;

    private static LoadingStrategy[] strategies = new LoadingStrategy[] { DUBBO_INTERNAL_STRATEGY, DUBBO_STRATEGY, SERVICES_STRATEGY };

    
    private Map<String, Class<?>> loadExtensionClasses() {
        cacheDefaultExtensionName();
        // 从配置文件中加载扩展实现类
        Map<String, Class<?>> extensionClasses = new HashMap<>();

        for (LoadingStrategy strategy : strategies) {
            loadDirectory(extensionClasses, strategy.directory(), type.getName(), strategy.preferExtensionClassLoader(), strategy.excludedPackages());
            loadDirectory(extensionClasses, strategy.directory(), type.getName().replace("org.apache", "com.alibaba"), strategy.preferExtensionClassLoader(), strategy.excludedPackages());
        }

        return extensionClasses;
    }
    private void cacheDefaultExtensionName() {
        final SPI defaultAnnotation = type.getAnnotation(SPI.class);
        if (defaultAnnotation == null) {
            return;
        }
        // 解析当前Extension配置的默认实现名，赋值给cachedDefaultName
        String value = defaultAnnotation.value();
        if ((value = value.trim()).length() > 0) {
            String[] names = NAME_SEPARATOR.split(value);
            // 每个扩展实现只能配置一个名称
            if (names.length > 1) {
                throw new IllegalStateException("More than 1 default extension name on extension " + type.getName()
                        + ": " + Arrays.toString(names));
            }
            if (names.length == 1) {
                cachedDefaultName = names[0];
            }
        }
    }
```

从代码里面可以看到，在loadExtensionClasses中首先会检测扩展点在@SPI注解中配置的默认扩展实现的名称，并将其赋值给cachedDefaultName属性进行缓存，后面想要获取该扩展点的默认实现名称就可以直接通过访问cachedDefaultName字段来完成，比如getDefaultExtensionName方法就是这么实现的。从这里的代码中又可以看到，具体的扩展实现类型，是通过调用loadFile方法来加载，分别从一下三个地方加载： META-INF/dubbo/internal/ META-INF/dubbo/ META-INF/services/

### loadFile方法

调用loadFile方法，代码比较长，主要做了几个事情，有几个变量会赋值：

- cachedAdaptiveClass : 当前Extension类型对应的AdaptiveExtension类型(只能一个) 
- cachedWrapperClasses : 当前Extension类型对应的所有Wrapper实现类型(无顺序) 
- cachedActivates : 当前Extension实现自动激活实现缓存(map,无序) 
- cachedNames : 扩展点实现类对应的名称(如配置多个名称则值为第一个)

当loadExtensionClasses方法执行完成之后，还有以下变量被赋值： cachedDefaultName : 当前扩展点的默认实现名称

当getExtensionClasses方法执行完成之后，除了上述变量被赋值之外，还有以下变量被赋值： cachedClasses : 扩展点实现名称对应的实现类(一个实现类可能有多个名称) 其实也就是说，在调用了getExtensionClasses方法之后，当前扩展点对应的实现类的一些信息就已经加载进来了并且被缓存了。后面的许多操作都可以直接通过这些缓存数据来进行处理了。

回到createAdaptiveExtension方法，他调用了getExtesionClasses方法加载扩展点实现信息完成之后，就可以直接通过判断cachedAdaptiveClass缓存字段是否被赋值盘确定当前扩展点是否有默认的AdaptiveExtension实现。如果没有，那么就调用createAdaptiveExtensionClass方法来动态生成一个。在dubbo的扩展点框架中大量的使用了缓存技术。

创建自适应扩展点实现类型和实例化就已经完成了，下面就来看下扩展点自动注入的实现injectExtension：

```java
private T injectExtension(T instance) {

        if (objectFactory == null) {
            return instance;
        }

        try {
            for (Method method : instance.getClass().getMethods()) {
                // 处理所有set方法
                if (!isSetter(method)) {
                    continue;
                }
                /**
                 * Check {@link DisableInject} to see if we need auto injection for this property
                 */
                if (method.getAnnotation(DisableInject.class) != null) {
                    continue;
                }
                // 获取set方法参数类型
                Class<?> pt = method.getParameterTypes()[0];
                if (ReflectUtils.isPrimitives(pt)) {
                    continue;
                }

                try {
                    // 获取setter对应的property名称
                    String property = getSetterProperty(method);
                    // 根据类型，名称信息从ExtensionFactory获取
                    Object object = objectFactory.getExtension(pt, property);
                    if (object != null) {
                        // 如果不为空，说set方法的参数是扩展点类型，那么进行注入
                        method.invoke(instance, object);
                    }
                } catch (Exception e) {
                    logger.error("Failed to inject via method " + method.getName()
                            + " of interface " + type.getName() + ": " + e.getMessage(), e);
                }

            }
        } catch (Exception e) {
            logger.error(e.getMessage(), e);
        }
        return instance;
    }
    private boolean isSetter(Method method) {
        return method.getName().startsWith("set")
                && method.getParameterTypes().length == 1
                && Modifier.isPublic(method.getModifiers());
    }
    private String getSetterProperty(Method method) {
        return method.getName().length() > 3 ? method.getName().substring(3, 4).toLowerCase() + method.getName().substring(4) : "";
    }
```

### getExtension

这个方法的主要作用是用来获取ExtensionLoader实例代表的扩展的指定实现。已扩展实现的名字作为参数，结合前面学习getAdaptiveExtension的代码。

```java
public T getExtension(String name) {
        if (StringUtils.isEmpty(name)) {
            throw new IllegalArgumentException("Extension name == null");
        }
        // 判断是否获取默认实现
        if ("true".equals(name)) {
            return getDefaultExtension();
        }
        // 缓存
        final Holder<Object> holder = getOrCreateHolder(name);
        Object instance = holder.get();
        if (instance == null) {
            synchronized (holder) {
                instance = holder.get();
                if (instance == null) {
                	// 没有缓存实例则创建
                    instance = createExtension(name);
                    // 缓存起来
                    holder.set(instance);
                }
            }
        }
        return (T) instance;
    }
```


![img](https://img-blog.csdnimg.cn/20200526201431521.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

## 总结

在整个过程中，最重要的两个方法getExtensionClasses和createAdaptiveExtensionClass getExtensionClasses方法主要是读取META-INF/services 、META-INF/dubbo、META-INF/internal目录下的文件内容，分析每一行，如果发现其中有哪个类的annotation是@Adaptive，就找到对应的AdaptiveClass。如果没有的话，就动态创建一个createAdaptiveExtensionClass。该方法是在getExtensionClasses方法找不到AdaptiveClass的情况下被调用，该方法主要是通过字节码的方式在内存中新生成一个类，它具有AdaptiveClass的功能，Protocol就是通过这种方式获得AdaptiveClass类的。

