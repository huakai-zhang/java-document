# 1 Spring 核心之 IOC 容器

`IOC(Inversion of Control) 控制反转` 所谓控制反转，就是把原先我们代码里面需要实现的对象创建、依赖的代码，反转给容器来帮忙实现。那么必然的我们需要创建一个容器，同时需要一种描述来让容器知道需要创建的对象与对象的关系。这个描述最具体表现就是我们所看到的配置文件。 

``DI(Dependency Injection) 依赖注入`` 就是指对象是被动接受依赖类而不是自己主动去找，换句话说就是指对象不是从容器中查找它依赖的类，而是在容器实例化对象的时候主动将它依赖的类注入给它。 

先从我们自己设计这样一个视角来考虑： 

1、对象和对象的关系怎么表示？ 

可以用 xml，properties 文件等语义化配置文件表示。

2、描述对象关系的文件存放在哪里？ 

可能是 classpath，filesystem，或者是 URL 网络资源，servletContext 等。 

回到正题，有了配置文件，还需要对配置文件解析。 

3、不同的配置文件对对象的描述不一样，如标准的，自定义声明式的，如何统一？ 

在内部需要有一个统一的关于对象的定义，所有外部的描述都必须转化成统一的描述定义。 

4、如何对不同的配置文件进行解析？ 

需要对不同的配置文件语法，采用不同的解析器。

# 2 Spring IOC 体系结构

## 2.1 BeanFactory

Spring Bean 的创建是典型的工厂模式，这一系列的 Bean 工厂，即 IOC 容器为开发者管理对象间的依赖关系提供了很多便利和基础服务，在 Spring 中有许多的 IOC 的实现： 

<img src="Spring IOC 源码分析.assets/20200331100636154.png" alt="20200331100636154" style="zoom: 80%;" />

其中 BeanFactory 作为最顶层的一个接口类，它定义了 IOC 容器的基本功能规范，BeanFactory 有三个子类：`ListableBeanFactory`、`HierarchicalBeanFactory` 和 `AutowireCapableBeanFactory`。最终的默认实现类是 `DefaultListableBeanFactory`，他实现了所有的接口。

那为何要定义这么多层次的接口呢？查阅这些接口的源码和说明发现，每个接口都有他使用的场合，它主要是为了区分在 Spring 内部在操作过程中对象的传递和转化过程中，对对象的数据访问所做的限制。例如： 

``ListableBeanFactory `` 可列表化的Bean（List，Map，Set） 

``HierarchicalBeanFactory `` 有层级（继承）关系的Bean 

``AutowireCapableBeanFactory`` 配置了（隐式，上面2个是显式）接口定义 Bean 的自动装配规则

BeanFactory 源码：

```java
public interface BeanFactory {
   //对FactoryBean的转义定义，因为如果使用bean的名字检索FactoryBean得到的对象是工厂生成的对象，
   //如果需要得到工厂本身，需要转义
   String FACTORY_BEAN_PREFIX = "&";
   
   //根据bean的名字，获取在I0C容器中得到bean实例
   Object getBean(String name) throws BeansException;

   //根据bean的名字和Class类型来得到bean实例，增加了类型安全验证机制。
   <T> T getBean(String name, Class<T> requiredType) throws BeansException;

   //提供对bean的检索，看看是否在I0C容器有这个名字的bean
   <T> T getBean(Class<T> requiredType) throws BeansException;

   Object getBean(String name, Object... args) throws BeansException;

   boolean containsBean(String name);

   //根据bean名字得到bean实例，并同时判断这个bean是不是单例
   boolean isSingleton(String name) throws NoSuchBeanDefinitionException;

   boolean isPrototype(String name) throws NoSuchBeanDefinitionException;

   boolean isTypeMatch(String name, Class<?> targetType) throws NoSuchBeanDefinitionException;

   //得到bean实例的Class类型
   Class<?> getType(String name) throws NoSuchBeanDefinitionException;

   //得到bean的别名，如果根据别名检索，那么其原名也会被检索出来
   String[] getAliases(String name);
}
```

在 BeanFactory 里只对 IOC 容器的基本行为作了定义，根本不关心你的 Bean 是如何定义怎样加载的。 正如我们只关心工厂里得到什么的产品对象，至于工厂是怎么生产这些对象的，这个基本的接口不关心。

而要知道工厂是如何产生对象的，我们需要看具体的 IOC 容器实现，Spring 提供了许多 IOC 容器的实现。比如 XmlBeanFactory，ClasspathXmlApplicationContext 等。

其中 XmlBeanFactory 就是针对最基本的 IOC 容器的实现，这个 IOC 容器可以读取 XML 文件定义的 BeanDefinition（XML 文件中对 bean 的描述），如果说 XmlBeanFactory 是容器中的屌丝，ApplicationContext 应该算容器中的高帅富.

``ApplcationContext`` 是Spring提供的一个高级 IOC 容器，除了提供 IOC 容器的基本功能外，还为用户提供以下附加服务：

- 支持信息源，可以实现国际化（MessageSource接口） 
- 访问资源（实现ResourcePatternResolver接口） 
- 支持应用事件（ApplicationEventPublisher接口）

```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
		MessageSource, ApplicationEventPublisher, ResourcePatternResolver
```

## 2.2 BeanDefinition

Spring IOC 容器管理定义的各种Bean对象及其相互的关系，Bean对象在Spring实现中是以BeanDefinition（保存了Bean的所有配置信息）来描述的：

​										 ![20200331101810123](Spring IOC 源码分析.assets/20200331101810123.png) 

## 2.3 BeanDefinitionReader

Bean的解析主要就是对Spring配置文件的解析，解析过程主要通过 BeanDefinitionReader 来完成：

![20200331101843949](Spring IOC 源码分析.assets/20200331101843949.png)

# 3 基于 Xml 的 IOC 容器的初始化

IOC 容器的初始化包括 ``BeanDefinition`` 的 Resource <font color="red">定位，载入和注册</font>三个基本过程。 以ApplicationContext为例，因为Web项目中使用XmlWebApplicationContext就属于这个继承体系，还有ClasspathXmlApplicationContext等： 

![20200331102028707](Spring IOC 源码分析.assets/20200331102028707.png)

 ``ApplicationContext`` 允许上下文嵌套，通过保存父上下文可以维持一个上下文体系。对于bean的查找可以在这个上下文体系中发生，首先检查当前上下文，其次是父上下文，逐级向上，这样为不同的Spring应用提供一个共享的bean环境。

两种 IOC 容器的创建过程：

## 3.1 XmlBeanFactory

```java
public class XmlBeanFactory extends DefaultListableBeanFactory {

   private final XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(this);

   public XmlBeanFactory(Resource resource) throws BeansException {
      this(resource, null);
   }

   public XmlBeanFactory(Resource resource, BeanFactory parentBeanFactory) throws BeansException {
      super(parentBeanFactory);
      this.reader.loadBeanDefinitions(resource);
   }
}
```

调用全过程还原，定位，载入，注册：

```java
//根据Xml配置文件创建Resource资源对象，该对象中包含了BeanDefinition的信息
ClassPathResource resource =new ClassPathResource("application-context.xml");
//创建DefaultListableBeanFactory
DefaultListableBeanFactorx factory =new DefaultlistableBeanFactory();
//创建XmlBeanDefinitionReader读取器，用于载入BeanDefinition。之所以需要BeanFactory作为参数，是因为会将读取的信息回调配置给factory
XmlBeanDefinitionReader reader =new XmlBeanDefinitionReader(factory);
//XmlBeanDefinitionReader.执行载入BeanDefinition的方法，最后会完成Bean的载入和注册。完成后Bean就成功的放置到I0C容器当中，以后我们就可以从中取得Bean来使用
reader.loadBeanDefinitions(resource);
```

## 3.2 基于 FileSystemXml 的IOC容器初始化

### 3.2.1 寻找入口

```java
ApplicationContext = new FileSystemXmlApplicationContext(xmlPath);
```

先看其构造函数的调用：

```java
public FileSystemXmlApplicationContext(String... configLocations) throws BeansException {
	this(configLocations, true, null);
}
```

其实际调用的构造函数为：

```java
public FileSystemXmlApplicationContext(String[] configLocations, boolean refresh, ApplicationContext parent)
      throws BeansException {
   // 首先调用父类容器的构造方法为容器设置好Bean资源加载器
   super(parent);
   // 告诉读取器配置文件放在哪里：定位，为了加载配置文件
   setConfigLocations(configLocations);
   // 刷新
   if (refresh) {
      refresh();
   }
}
```

还有像 AnnotationConfigApplicationContext 、 ClassPathXmlApplicationContext 、XmlWebApplicationContext 等都继承自父容器 AbstractApplicationContext主要用到了装饰器模式和策略模式，最终都是调用 refresh()方法。

### 3.2.2 获得资源路径

``FileSystemXmlApplicationContext`` 首先调用父类容器的构造方法 super(parent) 为容器设置好 `Bean 资源加载器`。 然后，再调用父类``AbstractApplicationContext`` 的 setConfigLocations() 方法来设置 Bean 定义资源文件的定位路径。 追踪其父类发现：

```java
public abstract class AbstractApplicationContext extends DefaultResourceLoader
      implements ConfigurableApplicationContext, DisposableBean {

   // 静态初始化块，在整个容器创建过程中只执行一次
   static {
       //为了避免应用程序在Weblogic8.1关闭时出现类异常加载问题，加载IOC容器关闭事件（ContextColsedEvent）类
      ContextClosedEvent.class.getName();
   }
   
   public AbstractApplicationContext() {
       // 解析资源文件，动态匹配的过程
       this.resourcePatternResolver = getResourcePatternResolver();
   }

	//FileSystemXmlApplicationContext调用父类构造方法调用的就是该方法
	public AbstractApplicationContext(ApplicationContext parent) {
	   this();
	   setParent(parent);
	}

	//获取一个Spring Source的加载器用于读入Spring Bean定义资源文件
	protected ResourcePatternResolver getResourcePatternResolver() {
	   // AbstractApplicationContext继承DefaultResourceLoader因此也是一个资源加载器
	   // Spring资源加载器，其getResource(String location)方法用于载入资源
  	 	return new PathMatchingResourcePatternResolver(this);
	}
}
```

AbstracApplicationContext构造方法中调用 `PathMatchingResourcePatternResolver` 的构造方法创建Spring资源加载器：

```java
public PathMatchingResourcePatternResolver(ResourceLoader resourceLoader) {
   Assert.notNull(resourceLoader, "ResourceLoader must not be null");
   //设置Spring的资源加载器
   this.resourceLoader = resourceLoader;
}
```

在设置容器的资源加载器之后，接下来 FileSystemXmlApplicationContext 执行 setConfigLocations 方法通过调用其父类``AbstractRefreshableConfigApplicationContext`` 的方法进行对Bean定义资源文件的定义：

```java
//处理单个资源文件为一个字符串的情况
public void setConfigLocation(String location) {
   //String CONFIG_LOCATION_DELIMITERS = ",; /t/n";  
   //即多个资源文件之间用  ",; /t/n"分隔，解析成数组形式
   setConfigLocations(StringUtils.tokenizeToStringArray(location, CONFIG_LOCATION_DELIMITERS));
}

//解析Bean定义资源文件的路径，处理多个资源文件字符串数组
public void setConfigLocations(String[] locations) {
   if (locations != null) {
      Assert.noNullElements(locations, "Config locations must not be null");
      this.configLocations = new String[locations.length];
      for (int i = 0; i < locations.length; i++) {
         //resolvePath为同一个类中将字符串解析为路径的方法
         this.configLocations[i] = resolvePath(locations[i]).trim();
      }
   }
   else {
      this.configLocations = null;
   }
}
```

通过这两个方法的源码我们可以看出，我们既可以使用一个字符串来配置多个 Spring Bean 配置信息，也可以使用字符串数组。

### 3.2.3 开始启动

Spring IOC 容器对Bean定义资源的`载入是从 refresh() 函数开始的`，refresh 是一个`模版方法`，refresh 方法的作用是：在创建 IOC 容器前，如果已有容器存在，则需要把已有的容器销毁和关闭，以保证在 refresh 之后使用的是新建立起来的 IOC 容器。refresh 的作用类似于对 IOC 容器的重启，在新建立好的容器中对容器进行初始化，对 Bean 定义资源进行载入。

FileSystemXmlApplicationContext 调用其父类的 AbstractApplicationContext 的 refresh 函数启动整个 IOC 容器对 Bean 定义的载入过程：

```java
//容器初始化的过程，读入Bean定义资源，并且解析注册
public void refresh() throws BeansException, IllegalStateException {
   synchronized (this.startupShutdownMonitor) {
      // Prepare this context for refreshing.
      // 1.调用容器准备刷新的方法，获取容器的当时时间，同时给容器设置同步标识
      prepareRefresh();

      // Tell the subclass to refresh the internal bean factory.
      // 2.告诉子类启动refreshBeanFactory()方法，Bean定义资源方法的载入从子类的refreshBeanFactory()方法启动
      ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

      // Prepare the bean factory for use in this context.
      // 3.为BeanFactory配置容器特性，例如类加载器，事件处理器等
      prepareBeanFactory(beanFactory);

      try {
         // Allows post-processing of the bean factory in context subclasses.
         // 4.为容器的某些子类指定特殊的BeanPost事件处理器
         postProcessBeanFactory(beanFactory);

         // Invoke factory processors registered as beans in the context.
         // 5.调用所有注册的BeanFactoryPostProcessor的Bean
         invokeBeanFactoryPostProcessors(beanFactory);

         // Register bean processors that intercept bean creation.
         // 6.为BeanFactory注册BeanPost事件处理器 
         // BeanPostProcessor是Bean后置处理器，用于监听容器触发的事件
         registerBeanPostProcessors(beanFactory);

         // Initialize message source for this context.
         // 7.初始化信息源，国际化相关
         initMessageSource();

         // Initialize event multicaster for this context.
         // 8.初始化容器事件传播器
         initApplicationEventMulticaster();

         // Initialize other special beans in specific context subclasses.
         // 9.调用子类的某些Bean初始化方法
         onRefresh();

         // Check for listener beans and register them.
         // 10.为事件传播器注册事件监听器
         registerListeners();

         // Instantiate all remaining (non-lazy-init) singletons.
         // 11.初始化所有剩余的单例Bean
         finishBeanFactoryInitialization(beanFactory);

         // Last step: publish corresponding event.
         // 12.初始化容器的生命周期事件处理器，并发布容器的生命周期事件
         finishRefresh();
      }

      catch (BeansException ex) {
         // Destroy already created singletons to avoid dangling resources.
         // 13.销毁以创建的单例Bean
         destroyBeans();

         // Reset 'active' flag.
         // 14.取消refresh操作，重置容器的同步标识
         cancelRefresh(ex);

         // Propagate exception to caller.
         throw ex;
      }
     	finally { 
       	// Reset common introspection caches in Spring's core, since we 
       	// might not ever need metadata for singleton beans anymore... 
       	// 15.重设公共缓存 
       	resetCommonCaches(); 
     }
   }
}
```

refresh()方法主要为IOC容器Bean的生命周期管理提供条件，Spring IOC容器载入Bean定义资源文件从其子类容器的``refreshBeanFactory()``方法启动，所以整个 refresh() 中 `ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory()` 这句代码以后的都是注册容器的信息源和生命周期事件，载入过程就是从这句代码启动。

### 3.2.4 创建容器

AbstractApplicationContext 的 obtainFreshBeanFactory() 方法调用子类容器的 refreshBeanFactory 方法，`启动容器载入Bean 定义资源文件`的过程：

```java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
   // 使用了委派模式，父类定义了抽象的 refreshBeanFactory() 方法，具体实现调用子类容器的 refreshBeanFactory
   refreshBeanFactory();
   ConfigurableListableBeanFactory beanFactory = getBeanFactory();
   if (logger.isDebugEnabled()) {
      logger.debug("Bean factory for " + getDisplayName() + ": " + beanFactory);
   }
   return beanFactory;
}
```

AbstractApplicationContext 类中只抽象定义了 refreshBeanFactory()方法，容器真正调用的是其子类 `AbstractRefreshableApplicationContext` 实现的 refreshBeanFactory() 方法，方法的源码如下：

```java
@Override
protected final void refreshBeanFactory() throws BeansException {
   // 如果已有容器，销毁容器中的bean，关闭容器
   if (hasBeanFactory()) {
      destroyBeans();
      closeBeanFactory();
   }
   try {
      //创建 IOC 容器
      DefaultListableBeanFactory beanFactory = createBeanFactory();
      beanFactory.setSerializationId(getId());
      // 对IOC容器进行定制化，如设置启动参数，开启注解的自动装配等
      customizeBeanFactory(beanFactory);
      // 调用载入Bean定义的方法，主要这里又使用了一个委派模式，在当前类中只定义了抽象的loadBeanDefinitions，具体的实现调用子类容器
      loadBeanDefinitions(beanFactory);
      synchronized (this.beanFactoryMonitor) {
         this.beanFactory = beanFactory;
      }
   }
   catch (IOException ex) {
      throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
   }
}
```

这个方法中，先判断 BeanFactory 是否存在，如果存在则先销毁 beans 并关闭 beanFactory，接着创建`DefaultListableBeanFactory`，并调用 loadBeanDefinitions 装载 bean 定义。

### 3.2.5 载入配置路径

AbstractRefreshableApplicationContext 中只定义了抽象的 loadBeanDefinitions 方法，容器真正调用的是其子类 AbstractXmlApplicationContext 对该方法的实现：

```java
public abstract class AbstractXmlApplicationContext 
extends AbstractRefreshableConfigApplicationContext {
...
// 实现父类抽象的载入Bean定义方法
@Override
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
   // Create a new XmlBeanDefinitionReader for the given BeanFactory.
   // 创建XmlBeanDefinitionReader，即创建Bean读取器，并通过回调设置到容器中去，容器使用该读取器读取Bean定义资源
   XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

   // Configure the bean definition reader with this context's
   // resource loading environment.
   beanDefinitionReader.setEnvironment(this.getEnvironment());
   // 为Bean读取器设置Spring资源加载器，AbstractXmlApplicationContext的
   //祖先父类AbstractApplicationContext继承DefaultResourceLoader，因此，容器本身也是一个资源加载器
   // resourceLoader == this
   // 即是 FileSystemXmlApplicationContext
   beanDefinitionReader.setResourceLoader(this);
   //为Bean读取器设置SAX xml解析器DOM4J
   beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));

   // Allow a subclass to provide custom initialization of the reader,
   // then proceed with actually loading the bean definitions.
   //当Bean读取器读取Bean定义的Xml资源文件时，启用Xml的校验机制
   initBeanDefinitionReader(beanDefinitionReader);
   //Bean读取器真正实现加载的方法
   loadBeanDefinitions(beanDefinitionReader);
}

//Xml Bean读取器加载Bean定义资源
protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws BeansException, IOException {
   //获取Bean定义资源的定位
   //这里使用了一个委派模式，调用子类的获取Bean定义资源位的方法
   //该方法在ClassPathXmlApplicationContext中进行实现，对于我们
   //举例分析源码的FileSystemXmlApplicationContext没有使用该方法
   Resource[] configResources = getConfigResources();
   if (configResources != null) {
      //Xml Bean读取器调用器父类AbstractBeanDefinitionReader读取定位的Bean定义资源
      reader.loadBeanDefinitions(configResources);
   }
   //如果子类中获取的Bean定义资源定位为空，则获取FileSystemXmlApplicationContext构造方法中setConfigLocations方法设置的资源
   String[] configLocations = getConfigLocations();
   if (configLocations != null) {
      //Xml Bean读取器调用器父类 AbstractBeanDefinitionReader 读取定位
      reader.loadBeanDefinitions(configLocations);
   }
}

// 这里又使用了一个委派模式，调用子类的获取Bean定义资源定位的方法
// 该方法在ClassPathXmlApplicationContext中进行实现，对于我们
// 举例分析源码的FileSystemXmlApplicationContext没有使用该方法
protected Resource[] getConfigResources() {
   return null;
}
}
```

Xml Bean读取器(XmlBeanDefinitionReader)调用其父类AbstractBeanDefinitionReader的reader.loadBeanDefinitions方法读取Bean定义资源。 这里使用FileSystemXmlApplicationContext作为例子分析，因此getConfigResources的返回值为null，因此程序<font color=red>执行</font>``reader.loadBeanDefinitions(configLocations)``;

### 3.2.6 分配路径处理策略

org.springframework.beans.factory.support 的 BeanDefinitionReader 在其抽象父类 AbstractBeanDefinitionReader 中定义了载入过程。

AbstractBeanDefinitionReader 的 loadBeanDefinitions() 方法源码如下： 

```java
//重载方法，调用下面的loadBeanDefinitions(String, Set<Resource>);方法 
public int loadBeanDefinitions(String location) throws BeanDefinitionStoreException {
   return loadBeanDefinitions(location, null);
}

public int loadBeanDefinitions(String location, Set<Resource> actualResources) throws BeanDefinitionStoreException {
   //获取在IoC容器初始化过程中设置的资源加载器 
   ResourceLoader resourceLoader = getResourceLoader();
   if (resourceLoader == null) {
      throw new BeanDefinitionStoreException(
            "Cannot import bean definitions from location [" + location + "]: no ResourceLoader available");
   }
   // resourceLoader == this == FileSystemXmlApplicationContext
   // ApplicationContext extends ResourcePatternResolver
   // 条件成立
   if (resourceLoader instanceof ResourcePatternResolver) {
      // Resource pattern matching available.
      try {
         // 将指定位置的Bean定义资源文件解析为Spring IOC容器封装的资源  
         // 加载多个指定位置的Bean定义资源文件  
         Resource[] resources = ((ResourcePatternResolver) resourceLoader).getResources(location);
         //委派调用其子类XmlBeanDefinitionReader的方法，实现加载功能 
         int loadCount = loadBeanDefinitions(resources);
         if (actualResources != null) {
            for (Resource resource : resources) {
               actualResources.add(resource);
            }
         }
         if (logger.isDebugEnabled()) {
            logger.debug("Loaded " + loadCount + " bean definitions from location pattern [" + location + "]");
         }
         return loadCount;
      }
      catch (IOException ex) {
         throw new BeanDefinitionStoreException(
               "Could not resolve bean definition resource pattern [" + location + "]", ex);
      }
   }
   else {
      ...
   }
}

//重载方法，调用loadBeanDefinitions(String);
@Override
public int loadBeanDefinitions(String... locations) throws BeanDefinitionStoreException {
	Assert.notNull(locations, "Location array must not be null");
	int counter = 0;
	for (String location : locations) {
		counter += loadBeanDefinitions(location);
	}
	return counter;
}
```

loadBeanDefinitions(Resource… resources) 方法和上面分析的3个方法类似，同样也是调用 XmlBeanDefinitionReader 的loadBeanDefinitions 方法。 从对 AbstractBeanDefinitionReader 的 loadBeanDefinitions 方法源码分析可以看出该方法做了一下两件事： 

首先，调用资源加载器的获取资源方法 `resourceLoader.getResource(location)` 获取要加载的资源

其次，真正执行加载功能是其子类 XmlBeanDefinitionReader 的 loadBeanDefinitions 方法。在 loadBeanDefinitions() 方法中调用了类型为 `ResourcePatternResolver` 的 getResources() 方法，有必要来看一下 ResourcePatternResolver 的全类图：  

![image-20210609175449095](Spring IOC 源码分析.assets/image-20210609175449095.png)

结合上面的继承图，可以知道此时调用的是 `AbstractApplicationContext 的 getResource() 方法`定位 Resource：

```java
// AbstractApplicationContext.java
// resourcePatternResolver 的定义上面有解析，即是 PathMatchingResourcePatternResolver
public Resource[] getResources(String locationPattern) throws IOException {
   return this.resourcePatternResolver.getResources(locationPattern);
}
// PathMatchingResourcePatternResolver.java
public Resource[] getResources(String locationPattern) throws IOException {
	if (locationPattern.startsWith(CLASSPATH_ALL_URL_PREFIX)) {
		// a class path resource (multiple resources for same name possible)
		if (getPathMatcher().isPattern(locationPattern.substring(CLASSPATH_ALL_URL_PREFIX.length()))) {
			// a class path resource pattern
			return findPathMatchingResources(locationPattern);
		}
		else {
			// all class path resources with the given name
			return findAllClassPathResources(locationPattern.substring(CLASSPATH_ALL_URL_PREFIX.length()));
		}
	}
	else {
    	// 最终调用的是 getResourceLoader()，其返回的是 this.resourceLoader
    	// resourceLoader 为 AbstractApplication 实例化时候传入的 this
    	// 实际为 FileSystemXmlApplicationContext
    	return new Resource[] {getResourceLoader().getResource(locationPattern)};
    }
}
```

### 3.2.7 解析配置文件路径

通过调用 `FileSystemXmlApplicationContext` 父类 `DefaultResourceLoader` 的 getResource() 方法获取要加载的资源：

```java
//获取Resource的具体实现方法
public Resource getResource(String location) {
   Assert.notNull(location, "Location must not be null");
   //如果是类路径的方式，那需要使用ClassPathResource来得到bean文件的资源对象
   if (location.startsWith(CLASSPATH_URL_PREFIX)) {
      return new ClassPathResource(location.substring(CLASSPATH_URL_PREFIX.length()), getClassLoader());
   }
   else {
      try {
         // Try to parse the location as a URL...
         // 如果是URL方式，使用UrlResource作为bean文件的资源对象
         URL url = new URL(location);
         return new UrlResource(url);
      }
      catch (MalformedURLException ex) {
         // No URL -> resolve as resource path.
         // 如果既不是classpath标识，又不是URL标识的Resouce定位，则调用
         // 容器本身的getResourceByPath方式获取Resource
         return getResourceByPath(location);
      }
   }
}
```

FileSystemXmlApplicationContext 容器提供了 getResourceByPath 方法的实现，就是为了处理既不是 classpath 标识，又不是 Url 标识的 Resouce 定位这种情况。

```java
FileSystemXmlApplicationContext：
protected Resource getResourceByPath(String path) {
   if (path != null && path.startsWith("/")) {
      path = path.substring(1);
   }
   // 这里使用文件系统资源对象来定义bean文件
   return new FileSystemResource(path);
}
```

代码回到了 FileSystemXmlApplicaitonContext 中来，它提供了 `FileSystemResource` 来完成`从文件系统得到配置文件的资源定义`。 这样就可以从文件系统路径上对 IOC 配置文件进行加载——当然可以按照这个逻辑从任何地方加载，在 Spring 中提供的各种资源抽象，比如 ClassPathResource，URLResouce，FileSystemResource 等来供我们使用。

> 上面是 <font color=red>定位</font> Resource 的一个过程，而这只是加载过程的一部分。

### 3.2.8 开始读取配置内容

Bean 定义了 Resource 得到了，继续回到 XmlBeanDefinitionReader 的 loadBeanDefinitions(Resource …) 方法看到代表bean 文件的资源定义以后的 <font color=red>载入</font> 过程。

```java
//这里是载入XML形式Bean定义资源文件方法
public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
   Assert.notNull(encodedResource, "EncodedResource must not be null");
   if (logger.isInfoEnabled()) {
      logger.info("Loading XML bean definitions from " + encodedResource.getResource());
   }

   Set<EncodedResource> currentResources = this.resourcesCurrentlyBeingLoaded.get();
   if (currentResources == null) {
      currentResources = new HashSet<EncodedResource>(4);
      this.resourcesCurrentlyBeingLoaded.set(currentResources);
   }
   if (!currentResources.add(encodedResource)) {
      throw new BeanDefinitionStoreException(
            "Detected cyclic loading of " + encodedResource + " - check your import definitions!");
   }
   try {
      //将资源文件转为InputStream的IO流
      InputStream inputStream = encodedResource.getResource().getInputStream();
      try {
         //从InputStream中得到XML的解析源
         InputSource inputSource = new InputSource(inputStream);
         if (encodedResource.getEncoding() != null) {
            inputSource.setEncoding(encodedResource.getEncoding());
         }
         //这里是具体的读取过程
         return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
      }
      finally {
         //关闭从Resource中得到的IO流
         inputStream.close();
      }
   }
   catch (IOException ex) {
      throw new BeanDefinitionStoreException(
            "IOException parsing XML document from " + encodedResource.getResource(), ex);
   }
   finally {
      currentResources.remove(encodedResource);
      if (currentResources.isEmpty()) {
         this.resourcesCurrentlyBeingLoaded.remove();
      }
   }
}
...
//从特定XML文件中实际载入Bean定义资源的方法
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
      throws BeanDefinitionStoreException {
  int validationMode = getValidationModeForResource(resource);
  //将XML文件转换为DOM对象，解析过程由documentLoader实现
  Document doc = this.documentLoader.loadDocument(inputSource, getEntityResolver(), this.errorHandler, validationMode, isNamespaceAware());
  //这里是启动对Bean定义解析的详细过程，该解析过程会用到Spring的Bean配置规则
  return registerBeanDefinitions(doc, resource);
}
```

通过源码分析，载入 Bean 定义资源文件的最后一步是将 `Bean 定义资源转换为 Document 对象`，该过程有 documentLoader 实现。

### 3.2.9 准备文档对象

DocumentLoader 将 Bean 配置资源转换成 Document 对象的源码如下： 

```java
//使用标准的JAXP将载入的Bean定义资源转换成document对象
public Document loadDocument(InputSource inputSource, EntityResolver entityResolver,
      ErrorHandler errorHandler, int validationMode, boolean namespaceAware) throws Exception {
   
   //创建文件解析器工厂
   DocumentBuilderFactory factory = createDocumentBuilderFactory(validationMode, namespaceAware);
   if (logger.isDebugEnabled()) {
      logger.debug("Using JAXP provider [" + factory.getClass().getName() + "]");
   }
   //创建文档解析器
   DocumentBuilder builder = createDocumentBuilder(factory, entityResolver, errorHandler);
   //解析Spring的Bean定义资源
   return builder.parse(inputSource);
}
//创建文档解析工厂
protected DocumentBuilderFactory createDocumentBuilderFactory(int validationMode, boolean namespaceAware)
      throws ParserConfigurationException {

   DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
   factory.setNamespaceAware(namespaceAware);

   //设置解析XML的校验
   if (validationMode != XmlValidationModeDetector.VALIDATION_NONE) {
      factory.setValidating(true);

      if (validationMode == XmlValidationModeDetector.VALIDATION_XSD) {
         // Enforce namespace aware for XSD...
         factory.setNamespaceAware(true);
         try {
            factory.setAttribute(SCHEMA_LANGUAGE_ATTRIBUTE, XSD_SCHEMA_LANGUAGE);
         }
         catch (IllegalArgumentException ex) {
            ParserConfigurationException pcex = new ParserConfigurationException(
                  "Unable to validate using XSD: Your JAXP provider [" + factory +
                  "] does not support XML Schema. Are you running on Java 1.4 with Apache Crimson? " +
                  "Upgrade to Apache Xerces (or Java 1.5) for full XSD support.");
            pcex.initCause(ex);
            throw pcex;
         }
      }
   }

   return factory;
}
```

该解析过程调用 JavaEE 标准的 `JAXP 标准`进行处理。 至此 Spring IOC 容器根据定位的 Bean 定义资源文件，将其`加载读入并转化为 Document 对象`过程完成。 接下来分析Spring IOC 将载入的 Bean 定义资源文件转换为 Document 之后，是如何将其解析为 SpringIOC 管理的 Bean 对象并将其注册到容器中的。

### 3.2.10 分配解析策略

XmlBeanDefinitionReader 类中的 doLoadBeanDefinitions 方法是从特定XML文件中实际载入Bean定义资源的方法，该方法在载入Bean 资源之后将其转换为 Document 对象，接下来调用 registerBeanDefinitions 启动Spring IOC容器对Bean定义的解析过程：

```java
//按照Spring的Bean语义要求将Bean定义资源解析并转换为容器内部数据结构
public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
   //得到BeanDefinitionDocumentReader来对xml格式的BeanDefinition解析
   BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
   documentReader.setEnvironment(this.getEnvironment());
   //获得容器中注册的Bean数量
   int countBefore = getRegistry().getBeanDefinitionCount();
   //解析过程入口，这里使用了委派模式，BeanDefinitionDocumentReader只是个接口，
   //具体的解析实现过程有实现类DefaultBeanDefinitionDocumentReader完成
    // createReaderContext 返回的是 XmlReaderContext
   documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
   //统计解析的Bean数量
   return getRegistry().getBeanDefinitionCount() - countBefore;
}
//创建BeanDefinitionDocumentReader对象，解析Document对象
protected BeanDefinitionDocumentReader createBeanDefinitionDocumentReader() {
   return BeanDefinitionDocumentReader.class.cast(BeanUtils.instantiateClass(this.documentReaderClass));
}
// 创建 XmlReaderContext 其中 reader 为 this 即 XmlBeanDefinitionReader
public XmlReaderContext createReaderContext(Resource resource) {
	return new XmlReaderContext(resource, this.problemReporter, this.eventListener,
			this.sourceExtractor, this, getNamespaceHandlerResolver());
}
```

Bean定义资源的载入解析分为以下两个过程： 

首先，通过调用XML解析器将Bean定义资源转换得到Document对象，但是这些Document对象并没有按照Spring Bean规则进行操作，这一步是载入的过程。

其次，在完成通用的XML解析之后，按照Spring的Bean规则对Document对象进行解析。 按照Spring的Bean规则对Document对象解析的过程是在接口BeanDefinitionDocumentReader的实现类DefaultBeanDefinitionDocumentReader中实现的。

### 3.2.11 将配置载入内存

BeanDefinitionDocumentReader接口通过 `registerBeanDefinitions` 方法调用其实现类DefaultBeanDefinitionDocumentReader对Document对象进行解析：

```java
//根据Spring DTD对Bean的定义规则解析Bean定义Document对象
// readerContext 来自于 XmlBeanDefinitionReader 的 createReaderContext
public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
   //获得XML描述符
   this.readerContext = readerContext;
   logger.debug("Loading bean definitions");
   //获得Document的根元素
   Element root = doc.getDocumentElement();
   doRegisterBeanDefinitions(root);
}

protected void doRegisterBeanDefinitions(Element root) {
   // Any nested <beans> elements will cause recursion in this method. In
   // order to propagate and preserve <beans> default-* attributes correctly,
   // keep track of the current (parent) delegate, which may be null. Create
   // the new (child) delegate with a reference to the parent for fallback purposes,
   // then ultimately reset this.delegate back to its original (parent) reference.
   // this behavior emulates a stack of delegates without actually necessitating one.
   //具体的解析过程由BeanDefinitionParserDelegate实现，  
    //BeanDefinitionParserDelegate中定义了Spring Bean定义XML文件的各种元素 
   BeanDefinitionParserDelegate parent = this.delegate; // this.delegate == null
   this.delegate = createDelegate(this.readerContext, root, parent);
   if (this.delegate.isDefaultNamespace(root)) {
		String profileSpec = root.getAttribute(PROFILE_ATTRIBUTE);
		if (StringUtils.hasText(profileSpec)) {
			String[] specifiedProfiles = StringUtils.tokenizeToStringArray(
						profileSpec, BeanDefinitionParserDelegate.MULTI_VALUE_ATTRIBUTE_DELIMITERS);
			if (!getReaderContext().getEnvironment().acceptsProfiles(specifiedProfiles)) {
				return;
			}
		}
	}
   //在解析Bean定义之前，进行自定义的解析，增强解析过程的可扩展性
   preProcessXml(root);
   //从Document的根元素开始进行Bean定义的Document对象
   parseBeanDefinitions(root, this.delegate);
   //在解析Bean定义之后，进行自定义的解析，增加解析过程的可扩展性
   postProcessXml(root);

   this.delegate = parent;
}

//使用Spring的Bean规则从Document的根元素开始进行Bean定义的Document对象
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
   //Bean定义的Document对象使用了Spring默认的XML命名空间
   if (delegate.isDefaultNamespace(root)) {
      //获取Bean定义的Document对象根元素的所有子节点
      NodeList nl = root.getChildNodes();
      for (int i = 0; i < nl.getLength(); i++) {
         Node node = nl.item(i);
         //获得Document节点是XML元素节点
         if (node instanceof Element) {
            Element ele = (Element) node;
            //Bean定义的Document的元素节点使用的是Spring默认的XML命名空间 
            if (delegate.isDefaultNamespace(ele)) {
               //使用Spring的Bean规则解析元素节点
               parseDefaultElement(ele, delegate);
            }
            else {
               //没有使用Spring默认的XML命名空间，则使用用户自定义的解
               //析规则解析元素节点
               delegate.parseCustomElement(ele);
            }
         }
      }
   }
   else {
      //Document的根节点没有使用Spring默认的命名空间，则使用用户自定义的  
       //解析规则解析Document根节点
      delegate.parseCustomElement(root);
   }
}

//使用Spring的Bean规则解析Document元素节点
private void  parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
   //如果元素节点是<Import>导入元素，进行导入解析
   if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {
      importBeanDefinitionResource(ele);
   }
   //如果元素节点是<Alias>别名元素，进行别名解析
   else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {
      processAliasRegistration(ele);
   }
   //元素节点既不是导入元素，也不是别名元素，即普通的<Bean>元素，  
   //按照Spring的Bean规则解析元素
   else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
      processBeanDefinition(ele, delegate);
   }
   else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {
      // recurse
      doRegisterBeanDefinitions(ele);
   }
}

//解析<Import>导入元素，从给定的导入路径加载Bean定义资源到Spring IoC容器中
protected void importBeanDefinitionResource(Element ele) {
   //获取给定的导入元素的location属性
   String location = ele.getAttribute(RESOURCE_ATTRIBUTE);
   //如果导入元素的location属性值为空，则没有导入任何资源，直接返回
   if (!StringUtils.hasText(location)) {
      getReaderContext().error("Resource location must not be empty", ele);
      return;
   }

   // Resolve system properties: e.g. "${user.dir}"
   //使用系统变量值解析location属性值
   location = environment.resolveRequiredPlaceholders(location);

   Set<Resource> actualResources = new LinkedHashSet<Resource>(4);

   // Discover whether the location is an absolute or relative URI
   //标识给定的导入元素的location是否是绝对路径
   boolean absoluteLocation = false;
   try {
      absoluteLocation = ResourcePatternUtils.isUrl(location) || ResourceUtils.toURI(location).isAbsolute();
   }
   catch (URISyntaxException ex) {
      // cannot convert to an URI, considering the location relative
      // unless it is the well-known Spring prefix "classpath*:"
      //给定的导入元素的location不是绝对路径
   }

   // Absolute or relative?
   //给定的导入元素的location是绝对路径
   if (absoluteLocation) {
      try {
         //使用资源读入器加载给定路径的Bean定义资源
         int importCount = getReaderContext().getReader().loadBeanDefinitions(location, actualResources);
         if (logger.isDebugEnabled()) {
            logger.debug("Imported " + importCount + " bean definitions from URL location [" + location + "]");
         }
      }
      catch (BeanDefinitionStoreException ex) {
         getReaderContext().error(
               "Failed to import bean definitions from URL location [" + location + "]", ele, ex);
      }
   }
   else {
      // No URL -> considering resource location as relative to the current file.
      //给定的导入元素的location是相对路径
      try {
         int importCount;
         //将给定导入元素的location封装为相对路径资源
         Resource relativeResource = getReaderContext().getResource().createRelative(location);
         //封装的相对路径资源存在
         if (relativeResource.exists()) {
            //使用资源读入器加载Bean定义资源
            importCount = getReaderContext().getReader().loadBeanDefinitions(relativeResource);
            actualResources.add(relativeResource);
         }
         //封装的相对路径资源不存在
         else {
            //获取Spring IOC容器资源读入器的基本路径 
            String baseLocation = getReaderContext().getResource().getURL().toString();
            //根据Spring IoC容器资源读入器的基本路径加载给定导入路径的资源
            importCount = getReaderContext().getReader().loadBeanDefinitions(
                  StringUtils.applyRelativePath(baseLocation, location), actualResources);
         }
         if (logger.isDebugEnabled()) {
            logger.debug("Imported " + importCount + " bean definitions from relative location [" + location + "]");
         }
      }
      catch (IOException ex) {
         getReaderContext().error("Failed to resolve current resource location", ele, ex);
      }
      catch (BeanDefinitionStoreException ex) {
         getReaderContext().error("Failed to import bean definitions from relative location [" + location + "]",
               ele, ex);
      }
   }
   Resource[] actResArray = actualResources.toArray(new Resource[actualResources.size()]);
   //在解析完<Import>元素之后，发送容器导入其他资源处理完成事件
   getReaderContext().fireImportProcessed(location, actResArray, extractSource(ele));
}
//解析<Alias>别名元素，为Bean向Spring IoC容器注册别名
protected void processAliasRegistration(Element ele) {
   //获取<Alias>别名元素中name的属性值
   String name = ele.getAttribute(NAME_ATTRIBUTE);
   //获取<Alias>别名元素中alias的属性值
   String alias = ele.getAttribute(ALIAS_ATTRIBUTE);
   boolean valid = true;
   //<alias>别名元素的name属性值为空
   if (!StringUtils.hasText(name)) {
      getReaderContext().error("Name must not be empty", ele);
      valid = false;
   }
   //<alias>别名元素的alias属性值为空
   if (!StringUtils.hasText(alias)) {
      getReaderContext().error("Alias must not be empty", ele);
      valid = false;
   }
   if (valid) {
      try {
         //向容器的资源读入器注册别名
         getReaderContext().getRegistry().registerAlias(name, alias);
      }
      catch (Exception ex) {
         getReaderContext().error("Failed to register alias '" + alias +
               "' for bean with name '" + name + "'", ele, ex);
      }
      //在解析完<Alias>元素之后，发送容器别名处理完成事件
      getReaderContext().fireAliasRegistered(name, alias, extractSource(ele));
   }
}
//解析Bean定义资源Document对象的普通元素
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
   BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
   // BeanDefinitionHolder是对BeanDefinition的封装，即Bean定义的封装类  
   // 对Document对象中<Bean>元素的解析由BeanDefinitionParserDelegate实现
   // BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
   if (bdHolder != null) {
      bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
      try {
         // Register the final decorated instance.
         //向Spring IOC容器注册解析得到的Bean定义，这是Bean定义向IOC容器注册的入口
         BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
      }
      catch (BeanDefinitionStoreException ex) {
         getReaderContext().error("Failed to register bean definition with name '" +
               bdHolder.getBeanName() + "'", ele, ex);
      }
      // Send registration event.
      //在完成向Spring IOC容器注册解析得到的Bean定义之后，发送注册事件
      getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
   }
}
```

通过上述Spring IOC容齐对载入的Bean定义Docunment解析可以看出，使用Spring时，在Spring配置文件中可以使用``<Import>``元素来导入IOC容齐所需要的其他资源，SpringIOC容器在解析时会首先将制定导入的资源加载进容器中。使用``<Ailas>``别名时，Spring IOC容器首先将别名元素所定义的别名注册到容器中。

对于既不是``<Import>``元素，又不是``<Alias>``元素的元素，即Spring配置文件中普通的``<Bean>``元素的解析是BeanDefinitionParserDelegate类的parseBeanDefinitionElement方法来实现。

#### 载入 `<bean>` 元素

Bean 配置信息中的`<import>`和`<alias>`元素解析在 DefaultBeanDefinitionDocumentReader 中已经完成，对 Bean 配置信息中使用最多的`<bean>`元素交由 BeanDefinitionParserDelegate 来解析，其解析实现的源码如下： 

```java
//解析<Bean>元素的入口
public BeanDefinitionHolder parseBeanDefinitionElement(Element ele) {
   return parseBeanDefinitionElement(ele, null);
}

//解析Bean定义资源文件中的<Bean>元素，这个方法中主要处理<Bean>元素的id，name  
//和别名属性
public BeanDefinitionHolder parseBeanDefinitionElement(Element ele, BeanDefinition containingBean) {
   //获取<Bean>元素中的id属性值
   String id = ele.getAttribute(ID_ATTRIBUTE);
   //获取<Bean>元素中的name属性值
   String nameAttr = ele.getAttribute(NAME_ATTRIBUTE);

   //获取<Bean>元素中的alias属性值
   List<String> aliases = new ArrayList<String>();
   //将<Bean>元素中的所有name属性值存放到别名中
   if (StringUtils.hasLength(nameAttr)) {
      String[] nameArr = StringUtils.tokenizeToStringArray(nameAttr, MULTI_VALUE_ATTRIBUTE_DELIMITERS);
      aliases.addAll(Arrays.asList(nameArr));
   }

   String beanName = id;
   //如果<Bean>元素中没有配置id属性时，将别名中的第一个值赋值给beanName
   if (!StringUtils.hasText(beanName) && !aliases.isEmpty()) {
      beanName = aliases.remove(0);
      if (logger.isDebugEnabled()) {
         logger.debug("No XML 'id' specified - using '" + beanName +
               "' as bean name and " + aliases + " as aliases");
      }
   }

   //检查<Bean>元素所配置的id或者name的唯一性，containingBean标识<Bean>  
    //元素中是否包含子<Bean>元素
   if (containingBean == null) {
      //检查<Bean>元素所配置的id、name或者别名是否重复
      checkNameUniqueness(beanName, aliases, ele);
   }

   //详细对<Bean>元素中配置的Bean定义进行解析的地方
   AbstractBeanDefinition beanDefinition = parseBeanDefinitionElement(ele, beanName, containingBean);
   if (beanDefinition != null) {
      if (!StringUtils.hasText(beanName)) {
         try {
            if (containingBean != null) {
               //如果<Bean>元素中没有配置id、别名或者name，且没有包含子元素
               //<Bean>元素，为解析的Bean生成一个唯一beanName并注册
               beanName = BeanDefinitionReaderUtils.generateBeanName(
                     beanDefinition, this.readerContext.getRegistry(), true);
            }
            else {
               //如果<Bean>元素中没有配置id、别名或者name，且包含了子元素
               //<Bean>元素，为解析的Bean使用别名向IOC容器注册
               beanName = this.readerContext.generateBeanName(beanDefinition);
               // Register an alias for the plain bean class name, if still possible,
               // if the generator returned the class name plus a suffix.
               // This is expected for Spring 1.2/2.0 backwards compatibility.
                    //为解析的Bean使用别名注册时，为了向后兼容
               //Spring1.2/2.0，给别名添加类名后缀
               String beanClassName = beanDefinition.getBeanClassName();
               if (beanClassName != null &&
                     beanName.startsWith(beanClassName) && beanName.length() > beanClassName.length() &&
                     !this.readerContext.getRegistry().isBeanNameInUse(beanClassName)) {
                  aliases.add(beanClassName);
               }
            }
            if (logger.isDebugEnabled()) {
               logger.debug("Neither XML 'id' nor 'name' specified - " +
                     "using generated bean name [" + beanName + "]");
            }
         }
         catch (Exception ex) {
            error(ex.getMessage(), ele);
            return null;
         }
      }
      String[] aliasesArray = StringUtils.toStringArray(aliases);
      return new BeanDefinitionHolder(beanDefinition, beanName, aliasesArray);
   }
   //当解析出错时，返回null
   return null;
}

//详细对<Bean>元素中配置的Bean定义其他属性进行解析，由于上面的方法中已经对
//Bean的id、name和别名等属性进行了处理，该方法中主要处理除这三个以外的其他属性数据  
public AbstractBeanDefinition parseBeanDefinitionElement(
      Element ele, String beanName, BeanDefinition containingBean) {

   //记录解析的<Bean>
   this.parseState.push(new BeanEntry(beanName));

   //这里只读取<Bean>元素中配置的class名字，然后载入到BeanDefinition中去  
    //只是记录配置的class名字，不做实例化，对象的实例化在依赖注入时完成
   String className = null;
   if (ele.hasAttribute(CLASS_ATTRIBUTE)) {
      className = ele.getAttribute(CLASS_ATTRIBUTE).trim();
   }

   try {
      String parent = null;
      //如果<Bean>元素中配置了parent属性，则获取parent属性的值
      if (ele.hasAttribute(PARENT_ATTRIBUTE)) {
         parent = ele.getAttribute(PARENT_ATTRIBUTE);
      }
      
      //根据<Bean>元素配置的class名称和parent属性值创建BeanDefinition  
        //为载入Bean定义信息做准备
      AbstractBeanDefinition bd = createBeanDefinition(className, parent);

      //对当前的<Bean>元素中配置的一些属性进行解析和设置，如配置的单态(singleton)属性等
      parseBeanDefinitionAttributes(ele, beanName, containingBean, bd);
      //为<Bean>元素解析的Bean设置description信息
      bd.setDescription(DomUtils.getChildElementValueByTagName(ele, DESCRIPTION_ELEMENT));

      //对<Bean>元素的meta(元信息)属性解析
      parseMetaElements(ele, bd);
      //对<Bean>元素的lookup-method属性解析
      parseLookupOverrideSubElements(ele, bd.getMethodOverrides());
      //对<Bean>元素的replaced-method属性解析
      parseReplacedMethodSubElements(ele, bd.getMethodOverrides());

      //解析<Bean>元素的构造方法设置
      parseConstructorArgElements(ele, bd);
      //解析<Bean>元素的<property>设置
      parsePropertyElements(ele, bd);
      //解析<Bean>元素的qualifier属性
      parseQualifierElements(ele, bd);

      //为当前解析的Bean设置所需的资源和依赖对象
      bd.setResource(this.readerContext.getResource());
      bd.setSource(extractSource(ele));

      return bd;
   }
   catch (ClassNotFoundException ex) {
      error("Bean class [" + className + "] not found", ele, ex);
   }
   catch (NoClassDefFoundError err) {
      error("Class that bean class [" + className + "] depends on not found", ele, err);
   }
   catch (Throwable ex) {
      error("Unexpected failure during bean definition parsing", ele, ex);
   }
   finally {
      this.parseState.pop();
   }
   //解析<Bean>元素出错时，返回null
   return null;
}
```

在Spring配置文件中``<Bean>``元素中的配置的属性就是通过该方法解析和设置到Bean中去的。 

注意：在解析``<Bean>``元素过程中没有创建和实例化Bean对象，只是创建了Bean对象的定义类BeanDefinition，将``<Bean>``元素中的配置信息设置到BeanDefinition中作为记录，当依赖注入时才使用这些记录信息创建和实例化具体的Bean对象。

使用Spring的``<Bean>``元素时，配置最多的是``<property>``属性，因此下面了解Bean的属性在解析时是如何设置的。

#### 载入``<property>``元素

BeanDefinitionParserDelegate在解析``<Bean>``调用parsePropertyElements方法解析``<Bean>``元素中的``<property>``属性子元素：

```java
//解析<Bean>元素中的<property>子元素
public void parsePropertyElements(Element beanEle, BeanDefinition bd) {
   //获取<Bean>元素中所有的子元素
   NodeList nl = beanEle.getChildNodes();
   for (int i = 0; i < nl.getLength(); i++) {
      Node node = nl.item(i);
      //如果子元素是<property>子元素，则调用解析<property>子元素方法解析
      if (isCandidateElement(node) && nodeNameEquals(node, PROPERTY_ELEMENT)) {
         parsePropertyElement((Element) node, bd);
      }
   }
}
//解析<property>元素
public void parsePropertyElement(Element ele, BeanDefinition bd) {
   //获取<property>元素的名字
   String propertyName = ele.getAttribute(NAME_ATTRIBUTE);
   if (!StringUtils.hasLength(propertyName)) {
      error("Tag 'property' must have a 'name' attribute", ele);
      return;
   }
   this.parseState.push(new PropertyEntry(propertyName));
   try {
      //如果一个Bean中已经有同名的property存在，则不进行解析，直接返回。  
        //即如果在同一个Bean中配置同名的property，则只有第一个起作用
      if (bd.getPropertyValues().contains(propertyName)) {
         error("Multiple 'property' definitions for property '" + propertyName + "'", ele);
         return;
      }
      //解析获取property的值
      Object val = parsePropertyValue(ele, bd, propertyName);
      //根据property的名字和值创建property实例
      PropertyValue pv = new PropertyValue(propertyName, val);
      //解析<property>元素中的属性
      parseMetaElements(ele, pv);
      pv.setSource(extractSource(ele));
      bd.getPropertyValues().addPropertyValue(pv);
   }
   finally {
      this.parseState.pop();
   }
}
//解析获取property值
public Object parsePropertyValue(Element ele, BeanDefinition bd, String propertyName) {
   String elementName = (propertyName != null) ?
               "<property> element for property '" + propertyName + "'" :
               "<constructor-arg> element";

   // Should only have one child element: ref, value, list, etc.
   //获取<property>的所有子元素，只能是其中一种类型:ref,value,list等
   NodeList nl = ele.getChildNodes();
   Element subElement = null;
   for (int i = 0; i < nl.getLength(); i++) {
      Node node = nl.item(i);
      //子元素不是description和meta属性
      if (node instanceof Element && !nodeNameEquals(node, DESCRIPTION_ELEMENT) &&
            !nodeNameEquals(node, META_ELEMENT)) {
         // Child element is what we're looking for.
         if (subElement != null) {
            error(elementName + " must not contain more than one sub-element", ele);
         }
         else {//当前<property>元素包含有子元素
            subElement = (Element) node;
         }
      }
   }

   //判断property的属性值是ref还是value，不允许既是ref又是value 
   boolean hasRefAttribute = ele.hasAttribute(REF_ATTRIBUTE);
   boolean hasValueAttribute = ele.hasAttribute(VALUE_ATTRIBUTE);
   if ((hasRefAttribute && hasValueAttribute) ||
         ((hasRefAttribute || hasValueAttribute) && subElement != null)) {
      error(elementName +
            " is only allowed to contain either 'ref' attribute OR 'value' attribute OR sub-element", ele);
   }

   //如果属性是ref，创建一个ref的数据对象RuntimeBeanReference
    //这个对象封装了ref信息
   if (hasRefAttribute) {
      String refName = ele.getAttribute(REF_ATTRIBUTE);
      if (!StringUtils.hasText(refName)) {
         error(elementName + " contains empty 'ref' attribute", ele);
      }
      //一个指向运行时所依赖对象的引用
      RuntimeBeanReference ref = new RuntimeBeanReference(refName);
      //设置这个ref的数据对象是被当前的property对象所引用
      ref.setSource(extractSource(ele));
      return ref;
   }
   //如果属性是value，创建一个value的数据对象TypedStringValue
    //这个对象封装了value信息
   else if (hasValueAttribute) {
      //一个持有String类型值的对象
      TypedStringValue valueHolder = new TypedStringValue(ele.getAttribute(VALUE_ATTRIBUTE));
      //设置这个value数据对象是被当前的property对象所引用
      valueHolder.setSource(extractSource(ele));
      return valueHolder;
   }
   //如果当前<property>元素还有子元素
   else if (subElement != null) {
      //解析<property>的子元素
      return parsePropertySubElement(subElement, bd);
   }
   else {
      // Neither child element nor "ref" or "value" attribute found.
      //propery属性中既不是ref，也不是value属性，解析出错返回null
      error(elementName + " must specify a ref or value", ele);
      return null;
   }
}
```

在Spring配置文件中，``<Bean>``元素中``<property>``元素的相关匹配处理：

- ref被封装为指向依赖对象一个饮用 
- value配置都会被封装成一个字符串类型的对象 
- ref和value都通过  “解析的数据类型属性值.setSource(extractSource(ele));”  方法将属性值/引用与所引用的属性连接起来。

在方法的最后对于``<property>``元素的子元素通过parsePropertySubElement方法解析。

#### 载入 ``<property>`` 的子元素

在BeanDefinitionPaserDelegate类中的parsePropertySubElement方法对``<property>``中的子元素解析：

```java
//解析<property>元素中ref，value或者集合等子元素
public Object parsePropertySubElement(Element ele, BeanDefinition bd, String defaultValueType) {
   //如果<property>没有使用Spring默认的命名空间，则使用用户自定义的规则解析
   //内嵌元素
   if (!isDefaultNamespace(ele)) {
      return parseNestedCustomElement(ele, bd);
   }
   //如果子元素是bean，则使用解析<Bean>元素的方法解析
   else if (nodeNameEquals(ele, BEAN_ELEMENT)) {
      BeanDefinitionHolder nestedBd = parseBeanDefinitionElement(ele, bd);
      if (nestedBd != null) {
         nestedBd = decorateBeanDefinitionIfRequired(ele, nestedBd, bd);
      }
      return nestedBd;
   }
   //如果子元素是ref，ref中只能有以下3个属性：bean、local、parent
   else if (nodeNameEquals(ele, REF_ELEMENT)) {
      // A generic reference to any name of any bean.
      //获取<property>元素中的bean属性值，引用其他解析的Bean的名称  
        //可以不再同一个Spring配置文件中，具体请参考Spring对ref的配置规则
      String refName = ele.getAttribute(BEAN_REF_ATTRIBUTE);
      boolean toParent = false;
      if (!StringUtils.hasLength(refName)) {
         // A reference to the id of another bean in the same XML file.
         //获取<property>元素中的local属性值，引用同一个Xml文件中配置  
               //的Bean的id，local和ref不同，local只能引用同一个配置文件中的Bean
         refName = ele.getAttribute(LOCAL_REF_ATTRIBUTE);
         if (!StringUtils.hasLength(refName)) {
            // A reference to the id of another bean in a parent context.
            //获取<property>元素中parent属性值，引用父级容器中的Bean
            refName = ele.getAttribute(PARENT_REF_ATTRIBUTE);
            toParent = true;
            
            if (!StringUtils.hasLength(refName)) {
               error("'bean', 'local' or 'parent' is required for <ref> element", ele);
               return null;
            }
         }
      }
      
      //没有配置ref的目标属性值 
      if (!StringUtils.hasText(refName)) {
         error("<ref> element contains empty target attribute", ele);
         return null;
      }
      //创建ref类型数据，指向被引用的对象
      RuntimeBeanReference ref = new RuntimeBeanReference(refName, toParent);
      //设置引用类型值是被当前子元素所引用
      ref.setSource(extractSource(ele));
      return ref;
   }
   //如果子元素是<idref>，使用解析ref元素的方法解析
   else if (nodeNameEquals(ele, IDREF_ELEMENT)) {
      return parseIdRefElement(ele);
   }
   //如果子元素是<value>，使用解析value元素的方法解析
   else if (nodeNameEquals(ele, VALUE_ELEMENT)) {
      return parseValueElement(ele, defaultValueType);
   }
   //如果子元素是null，为<property>设置一个封装null值的字符串数据
   else if (nodeNameEquals(ele, NULL_ELEMENT)) {
      // It's a distinguished null value. Let's wrap it in a TypedStringValue
      // object in order to preserve the source location.
      TypedStringValue nullHolder = new TypedStringValue(null);
      nullHolder.setSource(extractSource(ele));
      return nullHolder;
   }
   //如果子元素是<array>，使用解析array集合子元素的方法解析
   else if (nodeNameEquals(ele, ARRAY_ELEMENT)) {
      return parseArrayElement(ele, bd);
   }
   //如果子元素是<list>，使用解析list集合子元素的方法解析
   else if (nodeNameEquals(ele, LIST_ELEMENT)) {
      return parseListElement(ele, bd);
   }
   //如果子元素是<set>，使用解析set集合子元素的方法解析
   else if (nodeNameEquals(ele, SET_ELEMENT)) {
      return parseSetElement(ele, bd);
   }
   //如果子元素是<map>，使用解析map集合子元素的方法解析
   else if (nodeNameEquals(ele, MAP_ELEMENT)) {
      return parseMapElement(ele, bd);
   }
   //如果子元素是<props>，使用解析props集合子元素的方法解析
   else if (nodeNameEquals(ele, PROPS_ELEMENT)) {
      return parsePropsElement(ele);
   }
   //既不是ref，又不是value，也不是集合，则子元素配置错误，返回null
   else {
      error("Unknown property sub-element: [" + ele.getNodeName() + "]", ele);
      return null;
   }
}
```

在Spring配置文件中，对``<property>``元素中配置的Array，List，Set，Map，Prop等各种集合子元素都通过上述方法解析，生产对应的数据对象，比如ManagedList，ManagedArray等，这些Managed类是Spring对象BeanDefinition的数据封装，对集合数据类型的具体解析有各自的解析方法实现，解析方法的命名非常规范，一目了然。

#### 载入 ``<list>`` 子元素

在BeanDefinitionParserDelegate类中的parseListElement方法具体实现解析``<property>``元素中的``<list>``集合子元素：

```java
//解析<list>集合子元素
public List parseListElement(Element collectionEle, BeanDefinition bd) {
   //获取<list>元素中的value-type属性，即获取集合元素的数据类型
   String defaultElementType = collectionEle.getAttribute(VALUE_TYPE_ATTRIBUTE);
   //获取<list>集合元素中的所有子节点
   NodeList nl = collectionEle.getChildNodes();
   //Spring中将List封装为ManagedList
   ManagedList<Object> target = new ManagedList<Object>(nl.getLength());
   target.setSource(extractSource(collectionEle));
   //设置集合目标数据类型
   target.setElementTypeName(defaultElementType);
   target.setMergeEnabled(parseMergeAttribute(collectionEle));
   //具体的<list>元素解析
   parseCollectionElements(nl, target, bd, defaultElementType);
   return target;
}
//具体解析<list>集合元素，<array>、<list>和<set>都使用该方法解析
protected void parseCollectionElements(
      NodeList elementNodes, Collection<Object> target, BeanDefinition bd, String defaultElementType) {
   //遍历集合所有节点
   for (int i = 0; i < elementNodes.getLength(); i++) {
      Node node = elementNodes.item(i);
      //节点不是description节点
      if (node instanceof Element && !nodeNameEquals(node, DESCRIPTION_ELEMENT)) {
         //将解析的元素加入集合中，递归调用下一个子元素 
         target.add(parsePropertySubElement((Element) node, bd, defaultElementType));
      }
   }
}
```

经过对Spring Bean定义资源文件转换的Document对象中的元素层层解析，Spring IOC现在已经<font color="red">将XML形式定义的Bean定义资源文件转换为SpringIOC所识别的数据结构——BeanDefinition（加载BeanDefinition）</font>，它是Bean定义资源文件中配置的POJO对象在SpringIOC容器中的映射，可以通过AbstractBeanDefinition为入口，看到了IOC容器进行索引，查询和操作。 

通过SpringIOC容器对Bean定义资源的解析后，IOC容器大致完成了管理Bean对象的准备工作，即初始化过程，但是最为重要的依赖注入还没有发生，现在在IOC容器中BeanFefinition存储的只是一些静态信息，接下来需要向容器注册Bean定义信息才能全部完成IOC容器的初始化过程。

### 3.2.12 分配注册策略

DefaultBeanDefinitionDocumentReader对Bean定义转换的Document对象解析流程中，在其parseDefaultElement方法中完成对Document对象的解析后得到封装BeanDefinition的BeanDefinitionHold对象，然后调用BeanDefinitionReaderUtils的resisterBeanDefinition方法向IOC容器注册解析的Bean：

```java
/**
* 方法传入的 BeanDefinitionRegistry 来源于  XmlReaderContext.reader.getRegistry() 方法
* 即 XmlBeanDefinitionReader.getRegistry()
* 在最早的 AbstractRefreshableApplicationContext 的 refreshBeanFactory() 方法中创建了 DefaultListableBeanFactory 并传入到了 XmlBeanDefinitionReader 中
* 所以此处的 registry 即是 DefaultListableBeanFactory
*/
//将解析的BeanDefinitionHold注册到容器中
public static void registerBeanDefinition(
      BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
      throws BeanDefinitionStoreException {

   // Register bean definition under primary name.
   //获取解析的BeanDefinition的名称
   String beanName = definitionHolder.getBeanName();
   //向IOC容器注册BeanDefinition
   registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());

   // Register aliases for bean name, if any.
   //如果解析的BeanDefinition有别名，向容器为其注册别名
   String[] aliases = definitionHolder.getAliases();
   if (aliases != null) {
      for (String aliase : aliases) {
         registry.registerAlias(beanName, aliase);
      }
   }
}
```

当调用 DeanDefinitionReaderUtils 向IOC容器注册解析的 BeanDefinition 时，真正完成注册功能的是`DefaultListableBeanFactory`。

### 3.2.13 向容器注册

DefaultListableBeanFactory 中使用 HashMap 的集合对象存放IOC容器中注册解析的 BeanDefinition，向 IOC 容器注册：

```java
//向IoC容器注册解析的BeanDefinito
public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
      throws BeanDefinitionStoreException {

   Assert.hasText(beanName, "Bean name must not be empty");
   Assert.notNull(beanDefinition, "BeanDefinition must not be null");

   //校验解析的BeanDefiniton
   if (beanDefinition instanceof AbstractBeanDefinition) {
      try {
         ((AbstractBeanDefinition) beanDefinition).validate();
      }
      catch (BeanDefinitionValidationException ex) {
         throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
               "Validation of bean definition failed", ex);
      }
   }

   //注册的过程中需要线程同步，以保证数据的一致性
   synchronized (this.beanDefinitionMap) {
      Object oldBeanDefinition = this.beanDefinitionMap.get(beanName);
      
      //检查是否有同名的BeanDefinition已经在IOC容器中注册，如果已经注册，  
        //并且不允许覆盖已注册的Bean，则抛出注册失败异常
      if (oldBeanDefinition != null) {
         if (!this.allowBeanDefinitionOverriding) {
            throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
                  "Cannot register bean definition [" + beanDefinition + "] for bean '" + beanName +
                  "': There is already [" + oldBeanDefinition + "] bound.");
         }
         else {//如果允许覆盖，则同名的Bean，后注册的覆盖先注册的
            if (this.logger.isInfoEnabled()) {
               this.logger.info("Overriding bean definition for bean '" + beanName +
                     "': replacing [" + oldBeanDefinition + "] with [" + beanDefinition + "]");
            }
         }
      }
      else {//IOC容器中没有已经注册同名的Bean，按正常注册流程注册
         this.beanDefinitionNames.add(beanName);
         this.frozenBeanDefinitionNames = null;
      }
      this.beanDefinitionMap.put(beanName, beanDefinition);
   }
   //重置所有已经注册过的BeanDefinition的缓存
   resetBeanDefinition(beanName);
}
```

至此，Bean定义资源文件中配置的Bean被解析过后，已经注册到IOC容器中，被容器管理起来，真正完成了IOC容器初始化所做的全部工作。现在IOC容器中已经建立了整个Bean的配置信息，这些BeanDefinition信息已经可以使用，并且可以被检索，IOC容器的作用就是对这些注册的Bean定义信息进行处理和维护。这些注册的Bean定义信息是IOC容器控制反转的基础，正是有了这些注册的数据，容器才可以进行依赖注入。

```java
//存储注册信息的BeanDefinition
private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<String, BeanDefinition>(64);
```

<font color="red">IOC容器实际上也是一个Map.</font>

## 3.3 UML 图

![20200331101843949](Spring IOC 源码分析.assets/7b3636e7556e390a1c0eb10ef5328cd.png)

# 4 基于 Annotation 的 IOC 初始化

## 4.1 Annotation 的前世今生

从 Spring2.0 以后的版本中，Spring 也引入了基于注解(Annotation)方式的配置，注解(Annotation) 是 JDK1.5 中引入的一个新特性，用于简化 Bean 的配置，可以取代 XML 配置文件。开发人员对注解 (Annotation)的态度也是萝卜青菜各有所爱，个人认为注解可以大大简化配置，提高开发速度，但也给后期维护增加了难度。目前来说 XML 方式发展的相对成熟，方便于统一管理。

随着 Spring Boot 的兴起，基于注解的开发甚至实现了零配置。但作为个人的习惯而言，还是倾向于 XML 配置文件和注解 (Annotation)相互配合使用。Spring IOC 容器对于类级别的注解和类内部的注解分以下两种处理策略： 

1. 类级别的注解：如@Component、@Repository、@Controller、@Service 以及 JavaEE6 的 @ManagedBean 和@Named 注解，都是添加在类上面的类级别注解，Spring 容器根据注解的过滤规则扫描读取注解 Bean 定义类，并将其注册到 Spring IOC 容器中。

2. 类内部的注解：如@Autowire、@Value、@Resource 以及 EJB 和 WebService 相关的注解等， 都是添加在类内部的字段或者方法上的类内部注解，SpringIOC 容器通过 Bean 后置注解处理器解析 Bean 内部的注解。下面将根据这两种处理策略，分别分析 Spring 处理注解相关的源码。

## 4.2 定位 Bean 扫描路径

> SpringBoot 应用在启动过程中，会调用 createApplicationContext() 方法，在该方法中通过 ‌WebApplicationType 枚举指定使用的 ApplicationContext 类型。
>
> WebApplicationType 有三种类型：
>
> NONE‌：应用程序不应作为Web应用程序运行，也不会启动嵌入式Web服务器
> ‌SERVLET‌：应用程序应作为基于Servlet的Web应用程序运行，并启动嵌入式Servlet Web服务器
> ‌REACTIVE‌：应用程序应作为响应式Web应用程序运行，并启动嵌入式响应式Web服务器。响应式应用程序是一种现代的、异步的、非阻塞的方式来处理高并发请求‌
>
> WebApplicationType的判断逻辑基于类路径上的类是否存在。具体来说：
>
> ‌REACTIVE‌：需要类`org.springframework.web.reactive.DispatcherHandler`，并且不含`org.glassfish.jersey.servlet.ServletContainer`和`org.springframework.web.servlet.DispatcherServlet`
> ‌NONE‌：不含类`javax.servlet.Servlet`或`org.springframework.web.context.ConfigurableWebApplicationContext`，使用 `AnnotationConfigApplicationContext` 作为容器
> ‌SERVLET‌：需要包含类`javax.servlet.Servlet`和`org.springframework.web.context.ConfigurableWebApplicationContext‌`，使用`AnnotationConfigWebApplicationContex` 作为容器
>
> 应用场景：
> ‌NONE‌：适用于那些不需要Web服务的应用程序，例如后台任务处理、批处理等。
> ‌SERVLET‌：适用于传统的基于Servlet的Web应用程序，适用于大多数Web应用场景。
> ‌REACTIVE‌：适用于需要高并发处理的应用程序，如实时数据分析、高性能Web服务等。

在 Spring 中管理注解 Bean 定义的容器有两个 ： AnnotationConfigApplicationContext 和 AnnotationConfigWebApplicationContex。这两个类是专门处理 Spring 注解方式配置的容器，直接依赖于注解作为容器配置信息来源的 IOC 容器。AnnotationConfigWebApplicationContext 是 AnnotationConfigApplicationContext 的 Web 版本，两者的用法以及对注解的处理方式几乎没有差别。现在我们以 AnnotationConfigApplicationContext 为例看看它的源码：

```java
public class AnnotationConfigApplicationContext extends GenericApplicationContext implements AnnotationConfigRegistry {
	//保存一个读取注解的 Bean 定义读取器，并将其设置到容器中
	private final AnnotatedBeanDefinitionReader reader;
	//保存一个扫描指定类路径中注解 Bean 定义的扫描器，并将其设置到容器中
	private final ClassPathBeanDefinitionScanner scanner;
	//默认构造函数，初始化一个空容器，容器不包含任何 Bean 信息，需要在稍后通过调用其 register()
	//方法注册配置类，并调用 refresh()方法刷新容器，触发容器对注解 Bean 的载入、解析和注册过程
	public AnnotationConfigApplicationContext() {
		this.reader = new AnnotatedBeanDefinitionReader(this);
		this.scanner = new ClassPathBeanDefinitionScanner(this);
	}
	public AnnotationConfigApplicationContext(DefaultListableBeanFactory beanFactory) {
		super(beanFactory);
		this.reader = new AnnotatedBeanDefinitionReader(this);
		this.scanner = new ClassPathBeanDefinitionScanner(this);
	}
	//最常用的构造函数，通过将涉及到的配置类传递给该构造函数，以实现将相应配置类中的 Bean 自动注册到容器中
	public AnnotationConfigApplicationContext(Class<?>... annotatedClasses) {
		this();
		register(annotatedClasses);
		refresh();
	}
	//该构造函数会自动扫描以给定的包及其子包下的所有类，并自动识别所有的 Spring Bean，将其注册到容器中
	public AnnotationConfigApplicationContext(String... basePackages) {
		this();
		scan(basePackages);
		refresh();
	}
	@Override
	public void setEnvironment(ConfigurableEnvironment environment) {
		super.setEnvironment(environment);
		this.reader.setEnvironment(environment);
		this.scanner.setEnvironment(environment);
	}
	//为容器的注解 Bean 读取器和注解 Bean 扫描器设置 Bean 名称产生器
	public void setBeanNameGenerator(BeanNameGenerator beanNameGenerator) {
		this.reader.setBeanNameGenerator(beanNameGenerator);
		this.scanner.setBeanNameGenerator(beanNameGenerator);
		getBeanFactory().registerSingleton(
		AnnotationConfigUtils.CONFIGURATION_BEAN_NAME_GENERATOR, beanNameGenerator);
	}
	//为容器的注解 Bean 读取器和注解 Bean 扫描器设置作用范围元信息解析器
	public void setScopeMetadataResolver(ScopeMetadataResolver scopeMetadataResolver) {
		this.reader.setScopeMetadataResolver(scopeMetadataResolver);
		this.scanner.setScopeMetadataResolver(scopeMetadataResolver);
	}
	//为容器注册一个要被处理的注解 Bean，新注册的 Bean，必须手动调用容器的
	//refresh()方法刷新容器，触发容器对新注册的 Bean 的处理
	public void register(Class<?>... annotatedClasses) {
		Assert.notEmpty(annotatedClasses, "At least one annotated class must be specified");
		this.reader.register(annotatedClasses);
	}
	//扫描指定包路径及其子包下的注解类，为了使新添加的类被处理，必须手动调用
	//refresh()方法刷新容器
	public void scan(String... basePackages) {
		Assert.notEmpty(basePackages, "At least one base package must be specified");
		this.scanner.scan(basePackages);
	}
	...
}
```

通过上面的源码分析，我们可以看到 Spring 对注解的处理分为两种方式： 

1. 直接将注解 Bean 注册到容器中可以在初始化容器时注册；也可以在容器创建之后手动调用注册方法向容器注册，然后通过手动刷新容器，使得容器对注册的注解 Bean 进行处理。 

2. 通过扫描指定的包及其子包下的所有类在初始化注解容器时指定要自动扫描的路径，如果容器创建以后向给定路径动态添加了注解 Bean，则需要手动调用容器扫描的方法，然后手动刷新容器，使得容器对所注册的 Bean 进行处理。 接下来，将会对两种处理方式详细分析其实现过程。

## 4.3 读取 Annotation 元数据

当创建注解处理容器时，如果传入的初始参数是具体的注解 Bean 定义类时，注解容器读取并注册。 

**1)、AnnotationConfigApplicationContext 通过调用注解 Bean 定义读取器**

AnnotatedBeanDefinitionReader 的 register()方法向容器注册指定的注解 Bean，注解 Bean 定义读取器向容器注册注解 Bean 的源码如下：

```java
//注册多个注解 Bean 定义类
public void register(Class<?>... annotatedClasses) {
	for (Class<?> annotatedClass : annotatedClasses) {
		registerBean(annotatedClass);
	}
}
//注册一个注解 Bean 定义类
public void registerBean(Class<?> annotatedClass) {
	doRegisterBean(annotatedClass, null, null, null);
}
public <T> void registerBean(Class<T> annotatedClass, @Nullable Supplier<T> instanceSupplier) {
	doRegisterBean(annotatedClass, instanceSupplier, null, null);
}
public <T> void registerBean(Class<T> annotatedClass, String name, @Nullable Supplier<T> instanceSupplier) {
	doRegisterBean(annotatedClass, instanceSupplier, name, null);
}
//Bean 定义读取器注册注解 Bean 定义的入口方法
@SuppressWarnings("unchecked")
public void registerBean(Class<?> annotatedClass, Class<? extends Annotation>... qualifiers) {
	doRegisterBean(annotatedClass, null, null, qualifiers);
}
//Bean 定义读取器向容器注册注解 Bean 定义类
@SuppressWarnings("unchecked")
public void registerBean(Class<?> annotatedClass, String name, Class<? extends Annotation>... qualifiers) {
	doRegisterBean(annotatedClass, null, name, qualifiers);
}
//Bean 定义读取器向容器注册注解 Bean 定义类
<T> void doRegisterBean(Class<T> annotatedClass, @Nullable Supplier<T> instanceSupplier, @Nullable String name,
@Nullable Class<? extends Annotation>[] qualifiers, BeanDefinitionCustomizer... definitionCustomizers) {
	//根据指定的注解 Bean 定义类，创建 Spring 容器中对注解 Bean 的封装的数据结构
	AnnotatedGenericBeanDefinition abd = new AnnotatedGenericBeanDefinition(annotatedClass);
	if (this.conditionEvaluator.shouldSkip(abd.getMetadata())) {
		return;
	}
	abd.setInstanceSupplier(instanceSupplier);
	//解析注解 Bean 定义的作用域，若@Scope("prototype")，则 Bean 为原型类型；若@Scope("singleton")，则 Bean 为单态类型
	ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(abd);
	//为注解 Bean 定义设置作用域
	abd.setScope(scopeMetadata.getScopeName());
	//为注解 Bean 定义生成 Bean 名称
	String beanName = (name != null ? name : this.beanNameGenerator.generateBeanName(abd, this.registry));
	//处理注解 Bean 定义中的通用注解
	AnnotationConfigUtils.processCommonDefinitionAnnotations(abd);
	//如果在向容器注册注解 Bean 定义时，使用了额外的限定符注解，则解析限定符注解。
	//主要是配置的关于 autowiring 自动依赖注入装配的限定条件，即@Qualifier 注解
	//Spring 自动依赖注入装配默认是按类型装配，如果使用@Qualifier 则按名称
	if (qualifiers != null) {
		for (Class<? extends Annotation> qualifier : qualifiers) {
			//如果配置了@Primary 注解，设置该 Bean 为 autowiring 自动依赖注入装//配时的首选
			if (Primary.class == qualifier) {
				abd.setPrimary(true);
			}
			//如果配置了@Lazy 注解，则设置该 Bean 为非延迟初始化，如果没有配置，
			//则该 Bean 为预实例化
			else if (Lazy.class == qualifier) {
				abd.setLazyInit(true);
			}
			//如果使用了除@Primary 和@Lazy 以外的其他注解，则为该 Bean 添加一个 autowiring 自动依赖注入装配限定符，该 Bean 在进 autowiring 自动依赖注入装配时，根据名称装配限定符指定的 Bean
			else {
				abd.addQualifier(new AutowireCandidateQualifier(qualifier));
			}
		}
	}
	for (BeanDefinitionCustomizer customizer : definitionCustomizers) {
		customizer.customize(abd);
	}
	//创建一个指定 Bean 名称的 Bean 定义对象，封装注解 Bean 定义类数据
	BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(abd, beanName);
	//根据注解 Bean 定义类中配置的作用域，创建相应的代理对象
	definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
	//向 IOC 容器注册注解 Bean 类定义对象
	BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, this.registry);
}
```

从上面的源码我们可以看出，注册注解 Bean 定义类的基本步骤： 

a、需要使用注解元数据解析器解析注解 Bean 中关于作用域的配置。

b、使用 AnnotationConfigUtils 的 processCommonDefinitionAnnotations()方法处理注解 Bean 定义类中通用的注解。 

c、使用 AnnotationConfigUtils 的 applyScopedProxyMode()方法创建对于作用域的代理对象。 

d、通过 BeanDefinitionReaderUtils 向容器注册 Bean。 

下面我们继续分析这 4 步的具体实现过程 

2)、AnnotationScopeMetadataResolver 解析作用域元数据

AnnotationScopeMetadataResolver 通过 resolveScopeMetadata() 方法解析注解 Bean 定义类的`作用域元信息`，即判断注册的 Bean 是原生类型(prototype)还是单态(singleton)类型，其源码如下：

```java
protected Class<? extends Annotation> scopeAnnotationType = Scope.class;

//解析注解 Bean 定义类中的作用域元信息
@Override
public ScopeMetadata resolveScopeMetadata(BeanDefinition definition) {
	ScopeMetadata metadata = new ScopeMetadata();
	if (definition instanceof AnnotatedBeanDefinition) {
		AnnotatedBeanDefinition annDef = (AnnotatedBeanDefinition) definition;
		//从注解 Bean 定义类的属性中查找属性为”Scope”的值，即@Scope 注解的值
		//annDef.getMetadata().getAnnotationAttributes 方法将 Bean
		//中所有的注解和注解的值存放在一个 map 集合中
		AnnotationAttributes attributes = AnnotationConfigUtils.attributesFor(
		annDef.getMetadata(), this.scopeAnnotationType);
		//将获取到的@Scope 注解的值设置到要返回的对象中
		if (attributes != null) {
			metadata.setScopeName(attributes.getString("value"));
			//获取@Scope 注解中的 proxyMode 属性值，在创建代理对象时会用到
			ScopedProxyMode proxyMode = attributes.getEnum("proxyMode");
			//如果@Scope 的 proxyMode 属性为 DEFAULT 或者 NO
			if (proxyMode == ScopedProxyMode.DEFAULT) {
				//设置 proxyMode 为 NO
				proxyMode = this.defaultProxyMode;
			}
			//为返回的元数据设置 proxyMode
			metadata.setScopedProxyMode(proxyMode);
		}
	}
	//返回解析的作用域元信息对象
	return metadata;
}
```

3)、AnnotationConfigUtils 处理注解 Bean 定义类中的通用注解

AnnotationConfigUtils 类的 processCommonDefinitionAnnotations()在向容器注册 Bean 之前，首先对注解 Bean 定义类中的通用 Spring 注解进行处理，源码如下：

```java
//处理 Bean 定义中通用注解
static void processCommonDefinitionAnnotations(AnnotatedBeanDefinition abd, AnnotatedTypeMetadata metadata) {
	AnnotationAttributes lazy = attributesFor(metadata, Lazy.class);
	//如果 Bean 定义中有@Lazy 注解，则将该 Bean 预实例化属性设置为@lazy 注解的值
	if (lazy != null) {
		abd.setLazyInit(lazy.getBoolean("value"));
	}
	else if (abd.getMetadata() != metadata) {
		lazy = attributesFor(abd.getMetadata(), Lazy.class);
		if (lazy != null) {
			abd.setLazyInit(lazy.getBoolean("value"));
		}
	}
	//如果 Bean 定义中有@Primary 注解，则为该 Bean 设置为 autowiring 自动依赖注入装配的首选对象
	if (metadata.isAnnotated(Primary.class.getName())) {
		abd.setPrimary(true);
	}
	//如果 Bean 定义中有 @DependsOn 注解，则为该 Bean 设置所依赖的 Bean 名称，
	//容器将确保在实例化该 Bean 之前首先实例化所依赖的 Bean
	AnnotationAttributes dependsOn = attributesFor(metadata, DependsOn.class);
	if (dependsOn != null) {
		abd.setDependsOn(dependsOn.getStringArray("value"));
	}
	if (abd instanceof AbstractBeanDefinition) {
		AbstractBeanDefinition absBd = (AbstractBeanDefinition) abd;
		AnnotationAttributes role = attributesFor(metadata, Role.class);
		if (role != null) {
			absBd.setRole(role.getNumber("value").intValue());
		}
		AnnotationAttributes description = attributesFor(metadata, Description.class);
		if (description != null) {
			absBd.setDescription(description.getString("value"));
		}
	}
}
```

4)、AnnotationConfigUtils 根据注解 Bean 定义类中配置的作用域为其应用相应的代理策略

AnnotationConfigUtils 类的 applyScopedProxyMode()方法根据注解 Bean 定义类中配置的作用域 @Scope 注解的值，为 Bean 定义应用相应的代理模式，主要是在 Spring 面向切面编程(AOP)中使用。 源码如下：

```java
//根据作用域为 Bean 应用引用的代码模式
static BeanDefinitionHolder applyScopedProxyMode(ScopeMetadata metadata, BeanDefinitionHolder definition, BeanDefinitionRegistry registry) {
	//获取注解 Bean 定义类中 @Scope 注解的 proxyMode 属性值
	ScopedProxyMode scopedProxyMode = metadata.getScopedProxyMode();
	//如果配置的@Scope 注解的 proxyMode 属性值为 NO，则不应用代理模式
	if (scopedProxyMode.equals(ScopedProxyMode.NO)) {
		return definition;
	}
	//获取配置的@Scope 注解的 proxyMode 属性值，如果为 TARGET_CLASS
	//则返回 true，如果为 INTERFACES，则返回 false
	boolean proxyTargetClass = scopedProxyMode.equals(ScopedProxyMode.TARGET_CLASS);
	//为注册的 Bean 创建相应模式的代理对象
	return ScopedProxyCreator.createScopedProxy(definition, registry, proxyTargetClass);
}
```

这段为 Bean 引用创建相应模式的代理，这里不做深入的分析。 

5)、BeanDefinitionReaderUtils 向容器注册 Bean

BeanDefinitionReaderUtils 主要是校验 BeanDefinition 信息，然后将 Bean 添加到容器中一个管理 BeanDefinition 的 HashMap 中。

## 4.4 扫描指定包并解析为 BeanDefinition

当创建注解处理容器时，如果传入的初始参数是注解 Bean 定义类所在的包时，注解容器将扫描给定的 包及其子包，将扫描到的注解 Bean 定义载入并注册。 

1)、ClassPathBeanDefinitionScanner 扫描给定的包及其子包

AnnotationConfigApplicationContext 通过调用类路径 Bean 定义扫描器 ClassPathBeanDefinitionScanner 扫描给定包及其子包下的所有类，主要源码如下：

```java
public class ClassPathBeanDefinitionScanner extends ClassPathScanningCandidateComponentProvider {
	//创建一个类路径 Bean 定义扫描器
	public ClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry) {
		this(registry, true);
	}
	//为容器创建一个类路径 Bean 定义扫描器，并指定是否使用默认的扫描过滤规则。
	//即 Spring 默认扫描配置：@Component、@Repository、@Service、@Controller
	//注解的 Bean，同时也支持 JavaEE6 的@ManagedBean 和 JSR-330 的@Named 注解
	public ClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry, boolean useDefaultFilters) {
		this(registry, useDefaultFilters, getOrCreateEnvironment(registry));
	}
	public ClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry, boolean useDefaultFilters, Environment environment) {
		this(registry, useDefaultFilters, environment, (registry instanceof ResourceLoader ? (ResourceLoader) registry : null));
	}
	public ClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry, boolean useDefaultFilters, Environment environment, @Nullable ResourceLoader resourceLoader) {
		Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
		//为容器设置加载 Bean 定义的注册器
		this.registry = registry;
		if (useDefaultFilters) {
			registerDefaultFilters();
		}
		setEnvironment(environment);
		//为容器设置资源加载器
		setResourceLoader(resourceLoader);
	}
	//调用类路径 Bean 定义扫描器入口方法
	public int scan(String... basePackages) {
		//获取容器中已经注册的 Bean 个数
		int beanCountAtScanStart = this.registry.getBeanDefinitionCount();
		//启动扫描器扫描给定包
		doScan(basePackages);
		// Register annotation config processors, if necessary.
		//注册注解配置(Annotation config)处理器
		if (this.includeAnnotationConfig) {
			AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
		}
		//返回注册的 Bean 个数
		return (this.registry.getBeanDefinitionCount() - beanCountAtScanStart);
	}
	//类路径 Bean 定义扫描器扫描给定包及其子包
	protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
		Assert.notEmpty(basePackages, "At least one base package must be specified");
		//创建一个集合，存放扫描到 Bean 定义的封装类
		Set<BeanDefinitionHolder> beanDefinitions = new LinkedHashSet<>();
		//遍历扫描所有给定的包
		for (String basePackage : basePackages) {
			//调用父类 ClassPathScanningCandidateComponentProvider 的方法扫描给定类路径，获取符合条件的 Bean 定义
			Set<BeanDefinition> candidates = findCandidateComponents(basePackage);
			//遍历扫描到的 Bean
			for (BeanDefinition candidate : candidates) {
				//获取 Bean 定义类中@Scope 注解的值，即获取 Bean 的作用域
				ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(candidate);
				//为 Bean 设置注解配置的作用域
				candidate.setScope(scopeMetadata.getScopeName());
				//为 Bean 生成名称
				String beanName = this.beanNameGenerator.generateBeanName(candidate, this.registry);
				//如果扫描到的 Bean 不是 Spring 的注解 Bean，则为 Bean 设置默认值，
				//设置 Bean 的自动依赖注入装配属性等
				if (candidate instanceof AbstractBeanDefinition) {
					postProcessBeanDefinition((AbstractBeanDefinition) candidate, beanName);
				}
				//如果扫描到的 Bean 是 Spring 的注解 Bean，则处理其通用的 Spring 注解
				if (candidate instanceof AnnotatedBeanDefinition) {
					//处理注解 Bean 中通用的注解，在分析注解 Bean 定义类读取器时已经分析过
					AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition) candidate);
				}
				//根据 Bean 名称检查指定的 Bean 是否需要在容器中注册，或者在容器中冲突
				if (checkCandidate(beanName, candidate)) {
					BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
					//根据注解中配置的作用域，为 Bean 应用相应的代理模式
					definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
					beanDefinitions.add(definitionHolder);
					//向容器注册扫描到的 Bean
					registerBeanDefinition(definitionHolder, this.registry);
				}
			}
		}
		return beanDefinitions;
	}
	...
}
```

类路径 Bean 定义扫描器 ClassPathBeanDefinitionScanner 主要通过 findCandidateComponents() 方法调用其父类 ClassPathScanningCandidateComponentProvider 类来扫描获取给定包及其子包下 的类。 

2)、ClassPathScanningCandidateComponentProvider 扫描给定包及其子包的类

ClassPathScanningCandidateComponentProvider 类的 findCandidateComponents() 方法具体实现`扫描给定类路径包`的功能，主要源码如下：

```java
public class ClassPathScanningCandidateComponentProvider implements EnvironmentCapable, ResourceLoaderAware {
	//保存过滤规则要包含的注解，即 Spring 默认的@Component、@Repository、@Service、
	//@Controller 注解的 Bean，以及 JavaEE6 的@ManagedBean 和 JSR-330 的@Named 注解
	private final List<TypeFilter> includeFilters = new LinkedList<>();
	//保存过滤规则要排除的注解
	private final List<TypeFilter> excludeFilters = new LinkedList<>();
	//构造方法，该方法在子类 ClassPathBeanDefinitionScanner 的构造方法中被调用
	public ClassPathScanningCandidateComponentProvider(boolean useDefaultFilters) {
		this(useDefaultFilters, new StandardEnvironment());
	}
	public ClassPathScanningCandidateComponentProvider(boolean useDefaultFilters, Environment environment) {
		//如果使用 Spring 默认的过滤规则，则向容器注册过滤规则
		if (useDefaultFilters) {
			registerDefaultFilters();
		}
		setEnvironment(environment);
		setResourceLoader(null);
	}
	//向容器注册过滤规则
	@SuppressWarnings("unchecked")
	protected void registerDefaultFilters() {
		//向要包含的过滤规则中添加@Component 注解类，注意 Spring 中@Repository
		//@Service 和@Controller 都是 Component，因为这些注解都添加了@Component 注解
		this.includeFilters.add(new AnnotationTypeFilter(Component.class));
		//获取当前类的类加载器
		ClassLoader cl = ClassPathScanningCandidateComponentProvider.class.getClassLoader();
		try {
			//向要包含的过滤规则添加 JavaEE6 的@ManagedBean 注解
			this.includeFilters.add(new AnnotationTypeFilter(((Class<? extends Annotation>) ClassUtils.forName("javax.annotation.ManagedBean", cl)), false));
			logger.debug("JSR-250 'javax.annotation.ManagedBean' found and supported for component scanning");
		}
		catch (ClassNotFoundException ex) {
			// JSR-250 1.1 API (as included in Java EE 6) not available - simply skip.
		}
		try {
			//向要包含的过滤规则添加@Named 注解
			this.includeFilters.add(new AnnotationTypeFilter(((Class<? extends Annotation>) ClassUtils.forName("javax.inject.Named", cl)), false));
			logger.debug("JSR-330 'javax.inject.Named' annotation found and supported for component scanning");
		}
		catch (ClassNotFoundException ex) {
		// JSR-330 API not available - simply skip.
		}
	}
	//扫描给定类路径的包
	public Set<BeanDefinition> findCandidateComponents(String basePackage) {
		if (this.componentsIndex != null && indexSupportsIncludeFilters()) {
			return addCandidateComponentsFromIndex(this.componentsIndex, basePackage);
		}
		else {
			return scanCandidateComponents(basePackage);
		}
	}
	private Set<BeanDefinition> scanCandidateComponents(String basePackage) {
		Set<BeanDefinition> candidates = new LinkedHashSet<>();
		try {
			String packageSearchPath = ResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX +
					resolveBasePackage(basePackage) + '/' + this.resourcePattern;
            // AnnotationConfigApplicationContext 实例化时，会出初始化 ClassPathBeanDefinitionScanner
            // 其构造器会调用 setResourceLoader 
            // this.resourcePatternResolver = ResourcePatternUtils.getResourcePatternResolver(resourceLoader);
            // getResourcePatternResolver 返回的是 AnnotationConfigApplicationContext 本身
            // AnnotationConfigApplicationContext 实现了 ResourcePatternResolver 接口
			Resource[] resources = getResourcePatternResolver().getResources(packageSearchPath);
			boolean traceEnabled = logger.isTraceEnabled();
			boolean debugEnabled = logger.isDebugEnabled();
			for (Resource resource : resources) {
				MetadataReader metadataReader = getMetadataReaderFactory().getMetadataReader(resource);
				ScannedGenericBeanDefinition sbd = new ScannedGenericBeanDefinition(metadataReader);
				sbd.setResource(resource);
				sbd.setSource(resource);
				candidates.add(sbd);
            }				
		return candidates;
	}	
}
```

getResources() 方法调用的是 AnnotationConfigApplicationContext 的父类 AbstractApplicationContext，这里和 FileSystemXml 分析时一样，调用了 PathMatchingResourcePatternResolver 的 getResources()，不同的是这里调用了 findPathMatchingResources() 方法：

```java
protected Resource[] findPathMatchingResources(String locationPattern) throws IOException {
   String rootDirPath = determineRootDir(locationPattern);
   String subPattern = locationPattern.substring(rootDirPath.length());
   Resource[] rootDirResources = getResources(rootDirPath);
   Set<Resource> result = new LinkedHashSet<>(16);
   for (Resource rootDirResource : rootDirResources) {
      // ...
      result.addAll(doFindPathMatchingFileResources(rootDirResource, subPattern));
   }
   if (logger.isTraceEnabled()) {
      logger.trace("Resolved location pattern [" + locationPattern + "] to resources " + result);
   }
   return result.toArray(new Resource[0]);
}
// rootDir 绝对路径
// rootDir = rootDirResource.getFile().getAbsoluteFile();
protected Set<Resource> doFindMatchingFileSystemResources(File rootDir, String subPattern) {
	// 根据绝对路径，获取所有的文件路径的 File
    Set<File> matchingFiles = retrieveMatchingFiles(rootDir, subPattern);
	Set<Resource> result = new LinkedHashSet<>(matchingFiles.size());
	for (File file : matchingFiles) {
		result.add(new FileSystemResource(file));
	}
	return result;
}
```

## 4.5 注册注解 BeanDefinition

AnnotationConfigWebApplicationContext 是 AnnotationConfigApplicationContext 的 Web 版， 它们对于注解 Bean 的注册和扫描是基本相同的，但是 AnnotationConfigWebApplicationContext 对注解 Bean 定义的载入稍有不同，AnnotationConfigWebApplicationContext 注入注解 Bean 定义源码如下：

```java
//载入注解 Bean 定义资源
@Override
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) {
	//为容器设置注解 Bean 定义读取器
	AnnotatedBeanDefinitionReader reader = getAnnotatedBeanDefinitionReader(beanFactory);
	//为容器设置类路径 Bean 定义扫描器
	ClassPathBeanDefinitionScanner scanner = getClassPathBeanDefinitionScanner(beanFactory);
	//获取容器的 Bean 名称生成器
	BeanNameGenerator beanNameGenerator = getBeanNameGenerator();
	//为注解 Bean 定义读取器和类路径扫描器设置 Bean 名称生成器
	if (beanNameGenerator != null) {
		reader.setBeanNameGenerator(beanNameGenerator);
		scanner.setBeanNameGenerator(beanNameGenerator);
		beanFactory.registerSingleton(AnnotationConfigUtils.CONFIGURATION_BEAN_NAME_GENERATOR, beanNameGenerator);
	}
	//获取容器的作用域元信息解析器
	ScopeMetadataResolver scopeMetadataResolver = getScopeMetadataResolver();
	//为注解 Bean 定义读取器和类路径扫描器设置作用域元信息解析器
	if (scopeMetadataResolver != null) {
		reader.setScopeMetadataResolver(scopeMetadataResolver);
		scanner.setScopeMetadataResolver(scopeMetadataResolver);
	}
	if (!this.annotatedClasses.isEmpty()) {
		if (logger.isInfoEnabled()) {
			logger.info("Registering annotated classes: [" +
			StringUtils.collectionToCommaDelimitedString(this.annotatedClasses) + "]");
		}
		reader.register(this.annotatedClasses.toArray(new Class<?>[this.annotatedClasses.size()]));
	}
	if (!this.basePackages.isEmpty()) {
		if (logger.isInfoEnabled()) {
			logger.info("Scanning base packages: [" + StringUtils.collectionToCommaDelimitedString(this.basePackages) + "]");
		}
		scanner.scan(this.basePackages.toArray(new String[this.basePackages.size()]));
	}
	//获取容器定义的 Bean 定义资源路径
	String[] configLocations = getConfigLocations();
	//如果定位的 Bean 定义资源路径不为空
	if (configLocations != null) {
		for (String configLocation : configLocations) {
			try {
				//使用当前容器的类加载器加载定位路径的字节码类文件
				Class<?> clazz = ClassUtils.forName(configLocation, getClassLoader());
				if (logger.isInfoEnabled()) {
					logger.info("Successfully resolved class for [" + configLocation + "]");
				}
				reader.register(clazz);
			}
			catch (ClassNotFoundException ex) {
				if (logger.isDebugEnabled()) {
					logger.debug("Could not load class for config location [" + configLocation + "] - trying package scan. " + ex);
				}
				//如果容器类加载器加载定义路径的 Bean 定义资源失败
				//则启用容器类路径扫描器扫描给定路径包及其子包中的类
				int count = scanner.scan(configLocation);
				if (logger.isInfoEnabled()) {
					if (count == 0) {
						logger.info("No annotated classes found for specified class/package [" + configLocation + "]");
					}
					else {
						logger.info("Found " + count + " annotated classes in package [" + configLocation + "]");
					}
				}
			}
		}
	}
}
```

# 5 总结

现在通过上面的代码，总结一下IOC容器初始化的基本步骤： 

1.初始化的入口在容器实现中的refresh()调用来完成 

2.对 bean 定义载入 IOC 容器使用的方法是 loadBeanDefinition，其中的大致过程如下：通过 `ResourceLoader 来完成资源文件位置的定位`，DefaultResourceLoader 是默认的实现，同时上下文本身就给出了 ResourceLoader 的实现，可以从类路径、文件系统、URL等方式来定为资源位置。如果是 XmlBeanFactory 作为IOC容器，那么需要为它指定bean定义的资源，也就是说bean定义文件时通过抽象成 Resource 来被IOC容器处理的，容器通过 `BeanDefinitionReader 来完成定义信息的解析和Bean 信息的注册`，往往使用的是 XmlBeanDefinitionReader 来解析 bean 的 xml 定义文件，实际的处理过程是`委托给BeanDefinitionParserDelegate 来完成`的，从而得到 bean 的定义信息，这些信息在Spring中使用 `BeanDefinition 对象来表示`，这个名字可以让我们想到 loadBeanDefinition, RegisterBeanDefinition 这些相关方法，他们都是为处理 BeanDefinitin 服务的，容器解析得到 BeanDefinition 以后，由 IOC 容器实现 `BeanDefinitionRegistry 接口来把它在 IOC 容器中注册`。注册过程就是在IOC容器内部维护的一个HashMap来保存得到的BeanDefinition的过程。这个HashMap是IOC容器持有bean信息的场所，以后对bean的操作都是围绕这个HashMap来实现的 

3.然后我们就可以通过BeanFactory和ApplicationContext来享受到SpringIoC的服务了,在使用IoC容器的时候，我们注意到除了少量粘合代码，绝大多数以正确IOC风格编写的应用程序代码完全不用关心如何到达工厂，因为容器将把这些对象与容器管理的其他对象钩在一起。基本的策略是把工厂放到己知的地方，最好是放在对预期使用的上下文有意义的地方，以及代码将实际需要访问工厂的地方。Spring本身提供了对声明式载入web应用程序用法的应用程序上下文,并将其存储在ServletContext中的框架实现。

在使用SpringIOC容器的时候我们还需要区别两个概念: 

``BeanFactory和FactoryBean``,其中BeanFactory指的是IoC容器的编程抽象，比如ApplicationContext，XmlBeanFactory 等，这些都是IOC容器的具体表现，需要使用什么样的容器由客户决定,但Spring为我们提供了丰富的选择。FactoryBean 只是一个可以在IOC而容器中被管理的一个bean,是对各种处理过程和资源使用的抽象,FactoryBean在需要时产生另一个对象，而不返回FactoryBean本身,我们可以把它看成是一个抽象工厂，对它的调用返回的是工厂生产的产品。所有的FactoryBean都实现特殊的org. spr ingframework. beans. factory. FactoryBean接口，当使用容器中FactoryBean的时候，该容器不会返回FactoryBean本身,而是返回其生成的对象。Spring 包括了大部分的通用资源和服务访问抽象的FactoryBean的实现，其中包括:对JNDI查询的处理，对代理对象的处 理，对事务性代理的处理，对RMI代理的处理等，这些我们都可以看成是具体的工厂,看成是Spring为我们建立好的工厂。也就是说Spring通过使用抽象工厂模式为我们准备了一系列工厂来生产一些特定的对象,免除我们手工重复的工作，我们要使用时只需要在IOC容器里配置好就能很方便的使用了

