# 1 介绍


通过对 Spring IOC 容器的源码分析，已经基本上了解了 Spring IOC 容器对 Bean定义资源的定位、读入和解析过程，同时也清楚了当用户通过 getBean 方法向 IOC 容器获取被管理的Bean 时，IOC 容器对 Bean 进行的初始化和依赖注入过程，这些是 Spring IOC 容器的基本功能特性。Spring IOC 容器还有一些高级特性，如使用 lazy-init 属性对 Bean 预初始化、FactoryBean 产生或者修饰 Bean 对象的生成、IOC 容器初始化 Bean 过程中使用 BeanPostProcessor 后置处理器对 Bean声明周期事件管理和 IOC 容器的 autowiring 自动装配功能等。


通过前面对 IOC 容器的实现和工作原理分析知道 IOC 容器的初始化过程就是对 Bean 定义资源的定位、载入和注册，此时容器对 Bean 的依赖注入并没有发生，依赖注入主要是在应用程序第一次向容器索取 Bean 时，通过 getBean 方法的调用完成。当Bean定义资源的<Bean>元素中配置了lazy-init属性时，容器将会在初始化的时候对所配置的Bean进行预实例化，Bean 的依赖注入在容器初始化的时候就已经完成。这样，当应用程序第一次向容器索取被管理的 Bean 时，就不用再初始化和对 Bean 进行依赖注入了，直接从容器中获取已经完成依赖注入的现成 Bean，可以提高应用第一次向容器获取 Bean 的性能。下面通过代码分析容器预实例化的实现过程：

## refresh()

先从 IOC 容器的初始会过程开始，通过前面文章分析，我们知道 IOC 容器读入已经定位的 Bean 定义资源是从 refresh 方法开始的，我们首先从 AbstractApplicationContext 类的 refresh 方法入手分析，源码如下：

```java
//容器初始化的过程，读入Bean定义资源，并且解析注册
public void refresh() throws BeansException, IllegalStateException {
   synchronized (this.startupShutdownMonitor) {
      // Prepare this context for refreshing.
      //调用容器准备刷新的方法，获取容器的当时时间，同时给容器设置同步标识
      prepareRefresh();

      // Tell the subclass to refresh the internal bean factory.
      //告诉子类启动refreshBeanFactory()方法，Bean定义资源方法的载入从子类的refreshBeanFactory()方法启动
      ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

      // Prepare the bean factory for use in this context.
      //为BeanFactory配置容器特性，例如类加载器，事件处理器等
      prepareBeanFactory(beanFactory);

      try {
         // Allows post-processing of the bean factory in context subclasses.
         //为容器的某些子类指定特殊的BeanPost事件处理器
         postProcessBeanFactory(beanFactory);

         // Invoke factory processors registered as beans in the context.
         // 调用所有注册的BeanFactoryPostProcessor的Bean
         invokeBeanFactoryPostProcessors(beanFactory);

         // Register bean processors that intercept bean creation.
         //为BeanFactory注册BeanPost事件处理器 
         //BeanPostProcessor是Bean后置处理器，用于监听容器触发的事件
         registerBeanPostProcessors(beanFactory);

         // Initialize message source for this context.
         //初始化信息源，国际化相关
         initMessageSource();

         // Initialize event multicaster for this context.
         //初始化容器事件传播器
         initApplicationEventMulticaster();

         // Initialize other special beans in specific context subclasses.
         //调用子类的某些Bean初始化方法
         onRefresh();

         // Check for listener beans and register them.
         //为事件传播器注册事件监听器
         registerListeners();

         // Instantiate all remaining (non-lazy-init) singletons.
         //初始化所有剩余的单例Bean
         finishBeanFactoryInitialization(beanFactory);

         // Last step: publish corresponding event.
         // 初始化容器的生命周期事件处理器，并发布容器的生命周期事件
         finishRefresh();
      }

      catch (BeansException ex) {
         // Destroy already created singletons to avoid dangling resources.
         //销毁以创建的单例Bean
         destroyBeans();

         // Reset 'active' flag.
         //取消refresh操作，重置容器的同步标识
         cancelRefresh(ex);

         // Propagate exception to caller.
         throw ex;
      }
   }
}
```

在 refresh()方法中,ConfigurableListableBeanFactorybeanFactory = obtainFreshBeanFactory();启动了 Bean定义资源的载入、注册过程，而 finishBeanFactoryInitialization 方法是对注册后的 Bean 定义中 的预实例化(lazy-init=false,Spring 默认就是预实例化,即为 true)的 Bean 进行处理的地方。

# 2 lazy-init

## 2.1 预实例化 Bean

当 Bean 定义资源被载入 IOC 容器之后，容器将 Bean 定义资源解析为容器内部的数据结构BeanDefinition 注册到容器中 ， AbstractApplicationContext 类中的 `finishBeanFactoryInitialization 方法`对配置了`预实例化属性的 Bean` 进行预初始化过程，源码如下：

```java
//对配置了 lazy-init 属性的 Bean 进行预实例化处理
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
	//这是 Spring3 以后新加的代码，为容器指定一个转换服务(ConversionService)
	//在对某些 Bean 属性进行转换时使用
	// Initialize conversion service for this context.
	if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
			beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
		beanFactory.setConversionService(
				beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
	}
	// Initialize LoadTimeWeaverAware beans early to allow for registering their transformers early.
	String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
	for (String weaverAwareName : weaverAwareNames) {
		getBean(weaverAwareName);
	}
	// Stop using the temporary ClassLoader for type matching.
	//为了类型匹配，停止使用临时的类加载器
	beanFactory.setTempClassLoader(null);
	// Allow for caching all bean definition metadata, not expecting further changes.
	//缓存容器中所有注册的 BeanDefinition 元数据，以防被修改
	beanFactory.freezeConfiguration();
	// Instantiate all remaining (non-lazy-init) singletons.
	//对配置了 lazy-init 属性的单态模式 Bean 进行预实例化处理
	beanFactory.preInstantiateSingletons();
}
```

ConfigurableListableBeanFactory 是一个接口，其 preInstantiateSingletons 方法由其子类 DefaultListableBeanFactory 提供。

## 2.2 DefaultListableBeanFactory 对配置 lazy-init 属性单态 Bean 的预实例化

```java
public void preInstantiateSingletons() throws BeansException {
	if (this.logger.isDebugEnabled()) {
		this.logger.debug("Pre-instantiating singletons in " + this);
	}
	// Iterate over a copy to allow for init methods which in turn register new bean definitions.
	// While this may not be part of the regular factory bootstrap, it does otherwise work fine.
	List<String> beanNames = new ArrayList<String>(this.beanDefinitionNames);
	// Trigger initialization of all non-lazy singleton beans...
	for (String beanName : beanNames) {
		//获取指定名称的 Bean 定义
		RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
		//Bean 不是抽象的，是单态模式的，且 lazy-init 属性配置为 false
		if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
			//如果指定名称的 bean 是创建容器的 Bean
			if (isFactoryBean(beanName)) {
				//FACTORY_BEAN_PREFIX=”&”，当 Bean 名称前面加”&”符号
				//时，获取的是产生容器对象本身，而不是容器产生的 Bean.
				//调用 getBean 方法，触发容器对 Bean 实例化和依赖注入过程
				final FactoryBean<?> factory = (FactoryBean<?>) getBean(FACTORY_BEAN_PREFIX + beanName);
				//标识是否需要预实例化
				boolean isEagerInit;
				if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
					//一个匿名内部类
					isEagerInit = AccessController.doPrivileged(new PrivilegedAction<Boolean>() {
						@Override
						public Boolean run() {
							return ((SmartFactoryBean<?>) factory).isEagerInit();
						}
					}, getAccessControlContext());
				}
				else {
					isEagerInit = (factory instanceof SmartFactoryBean &&
							((SmartFactoryBean<?>) factory).isEagerInit());
				}
				if (isEagerInit) {
					//调用 getBean 方法，触发容器对 Bean 实例化和依赖注入过程
					getBean(beanName);
				}
			}
			else {
				getBean(beanName);
			}
		}
	}
	...
}
```

通过对 lazy-init 处理源码的分析，我们可以看出，如果设置了 lazy-init 属性，则容器`在完成 Bean 定义的注册之后，会通过 getBean 方法触发对指定 Bean 的初始化和依赖注入过程`，这样当应用第一次向容器索取所需的 Bean 时，容器不再需要对 Bean 进行初始化和依赖注入，直接从已经完成实例化和依赖注入的 Bean 中取一个现成的 Bean，这样就提高了第一次获取 Bean 的性能。

# 3 BeanFactory 和 FactoryBean

在 Spring 中，有两个很容易混淆的类：BeanFactory 和 FactoryBean：

BeanFactory：Bean 工厂，是一个工厂(Factory)，我们 Spring IOC 容器的最顶层接口就是这个BeanFactory，它的作用是管理 Bean，即实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。 

FactoryBean：工厂 Bean，是一个 Bean，作用是产生其他 bean 实例。通常情况下，这种 bean 没有什么特别的要求，仅需要提供一个工厂方法，该方法用来返回其他 bean 实例。通常情况下，bean 无须自己实现工厂模式，Spring 容器担任工厂角色；但少数情况下，容器中的 bean 本身就是工厂，其作用是产生其它 bean 实例。 当用户使用容器本身时，可以使用转义字符”&amp;”来得到 FactoryBean 本身，以区别通过 FactoryBean产生的实例对象和 FactoryBean 对象本身。在 BeanFactory 中通过如下代码定义了该转义字符：

```java
String FACTORY_BEAN_PREFIX = "&";
```

如果 myJndiObject 是一个 FactoryBean，则使用&amp;myJndiObject 得到的是 myJndiObject 对象，而 不是 myJndiObject 产生出来的对象。

## 3.1 FactoryBean 的源码

```java
//工厂 Bean，用于产生其他对象
public interface FactoryBean<T> {
	//获取容器管理的对象实例
	T getObject() throws Exception;

	//获取 Bean 工厂创建的对象的类型
	Class<?> getObjectType();

	//Bean 工厂创建的对象是否是单态模式，如果是单态模式，则整个容器中只有一个实例
	//对象，每次请求都返回同一个实例对象
	boolean isSingleton();
}
```

## 3.2 AbstractBeanFactory 的 getBean 方法调用 FactoryBean

在前面分析 Spring IOC 容器实例化 Bean 并进行依赖注入过程的源码时，提到在 getBean 方法触发容器实例化 Bean 的时候会调用 AbstractBeanFactory 的 doGetBean 方法来进行实例化的过程中，会调用getObjectForBeanInstance()获取给定Bean的实例对象：

```java
//Bean 工厂生产 Bean 实例对象
protected Object getObjectFromFactoryBean(FactoryBean<?> factory, String beanName, boolean shouldPostProcess) {
	//Bean 工厂是单态模式，并且 Bean 工厂缓存中存在指定名称的 Bean 实例对象
	if (factory.isSingleton() && containsSingleton(beanName)) {
		//多线程同步，以防止数据不一致
		synchronized (getSingletonMutex()) {
			//直接从 Bean 工厂缓存中获取指定名称的 Bean 实例对象
			Object object = this.factoryBeanObjectCache.get(beanName);
			//Bean 工厂缓存中没有指定名称的实例对象，则生产该实例对象
			if (object == null) {
				//调用 Bean 工厂的 getObject 方法生产指定 Bean 的实例对象
				object = doGetObjectFromFactoryBean(factory, beanName);
				// Only post-process and store if not put there already during getObject() call above
				// (e.g. because of circular reference processing triggered by custom getBean calls)
				Object alreadyThere = this.factoryBeanObjectCache.get(beanName);
				if (alreadyThere != null) {
					object = alreadyThere;
				}
				else {
					if (object != null && shouldPostProcess) {
						try {
							object = postProcessObjectFromFactoryBean(object, beanName);
						}
						catch (Throwable ex) {
							throw new BeanCreationException(beanName,
									"Post-processing of FactoryBean's singleton object failed", ex);
						}
					}
					//将生产的实例对象添加到 Bean 工厂缓存中
					this.factoryBeanObjectCache.put(beanName, (object != null ? object : NULL_OBJECT));
				}
			}
			return (object != NULL_OBJECT ? object : null);
		}
	}
	//调用 Bean 工厂的 getObject 方法生产指定 Bean 的实例对象
	else {
		Object object = doGetObjectFromFactoryBean(factory, beanName);
		if (object != null && shouldPostProcess) {
			try {
				object = postProcessObjectFromFactoryBean(object, beanName);
			}
			catch (Throwable ex) {
				throw new BeanCreationException(beanName, "Post-processing of FactoryBean's object failed", ex);
			}
		}
		return object;
	}
}
//调用 Bean 工厂的 getObject 方法生产指定 Bean 的实例对象
private Object doGetObjectFromFactoryBean(final FactoryBean<?> factory, final String beanName)
		throws BeanCreationException {
	Object object;
	try {
		if (System.getSecurityManager() != null) {
			AccessControlContext acc = getAccessControlContext();
			try {
				//实现 PrivilegedExceptionAction 接口的匿名内置类
				//根据 JVM 检查权限，然后决定 BeanFactory 创建实例对象
				object = AccessController.doPrivileged(new PrivilegedExceptionAction<Object>() {
					@Override
					public Object run() throws Exception {
							return factory.getObject();
						}
					}, acc);
			}
			catch (PrivilegedActionException pae) {
				throw pae.getException();
			}
		}
		else {
			//调用 BeanFactory 接口实现类的创建对象方法
			object = factory.getObject();
		}
	}
	catch (FactoryBeanNotInitializedException ex) {
		throw new BeanCurrentlyInCreationException(beanName, ex.toString());
	}
	catch (Throwable ex) {
		throw new BeanCreationException(beanName, "FactoryBean threw exception on object creation", ex);
	}
	// Do not accept a null value for a FactoryBean that's not fully
	// initialized yet: Many FactoryBeans just return null then.
	//创建出来的实例对象为 null，或者因为单态对象正在创建而返回 null
	if (object == null && isSingletonCurrentlyInCreation(beanName)) {
		throw new BeanCurrentlyInCreationException(
				beanName, "FactoryBean which is currently in creation returned null from getObject");
	}
	return object;
}
```

从上面的源码分析中，可以看出，BeanFactory 接口调用其实现类的 getObject 方法来实现创建Bean 实例对象的功能。

## 3.3 工厂 Bean 的实现类 getObject 方法创建 Bean 实例对象

FactoryBean 的实现类有非常多，比如：Proxy、RMI、JNDI、ServletContextFactoryBean 等等，FactoryBean 接口为 Spring 容器提供了一个很好的封装机制，具体的 getObject 有不同的实现类根据不同的实现策略来具体提供，分析一个最简单的 AnnotationTestFactoryBean 的实现源码：

```java
public class AnnotationTestBeanFactory implements FactoryBean<FactoryCreatedAnnotationTestBean> {

   private final FactoryCreatedAnnotationTestBean instance = new FactoryCreatedAnnotationTestBean();

   public AnnotationTestBeanFactory() {
      this.instance.setName("FACTORY");
   }

   @Override
   // AnnotationTestBeanFactory产生Bean实例对象的实现
   public FactoryCreatedAnnotationTestBean getObject() throws Exception {
      return this.instance;
   }

   @Override
   public Class<? extends IJmxTestBean> getObjectType() {
      return FactoryCreatedAnnotationTestBean.class;
   }

   @Override
   public boolean isSingleton() {
      return true;
   }

}
```

其他的Proxy，RMI，JNDI 等等，都是根据相应的策略提供 getObject 的实现。这里不做一一分析，这已经不是 Spring 的核心功能，有需要的时候再去深入研究。

BeanPostProcessor 后置处理器是 Spring IOC 容器经常使用到的一个特性，这个 Bean 后置处理器是一个监听器，可以监听容器触发的 Bean 声明周期事件。后置处理器向容器注册以后，容器中管理的 Bean就具备了接收 IOC 容器事件回调的能力。 BeanPostProcessor 的使用非常简单，只需要提供一个实现接口 BeanPostProcessor 的实现类，然后在 Bean 的配置文件中设置即可。

# 4 BeanPostProcessor

## 4.1 BeanPostProcessor 的源码

```java
public interface BeanPostProcessor {

	//为在 Bean 的初始化前提供回调入口
	Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;

	//为在 Bean 的初始化之后提供回调入口
	Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;
}
```

这两个回调的入口都是和容器管理的 Bean 的生命周期事件紧密相关，可以为用户提供在 Spring IOC容器初始化 Bean 过程中自定义的处理操作。

## 4.2 AbstractAutowireCapableBeanFactory 类对容器生成的 Bean 添加后置处理器

`BeanPostProcessor 后置处理器的调用`发生在 Spring IOC 容器完成对 Bean 实例对象的创建和属性的`依赖注入完成之后`，在对 Spring 依赖注入的源码分析过程中知道，当应用程序第一次调用 getBean方法(lazy-init 预实例化除外)向 Spring IOC 容器索取指定 Bean 时触发 Spring IOC 容器创建 Bean 实例对象并进行依赖注的过程，其中真正实现创建 Bean 对象并进行依赖注入的方法是 AbstractAutowireCapableBeanFactory 类的 doCreateBean 方法，主要源码如下：

```java
//真正创建 Bean 的方法
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final 						@Nullable Object[] args)
throws BeanCreationException {
	//创建 Bean 实例对象
	...
	Object exposedObject = bean;
	try {
		//对 Bean 属性进行依赖注入
		populateBean(beanName, mbd, instanceWrapper);
		//在对 Bean 实例对象生成和依赖注入完成以后，开始对 Bean 实例对象
		//进行初始化 ，为 Bean 实例对象应用 BeanPostProcessor 后置处理器
		exposedObject = initializeBean(beanName, exposedObject, mbd);
	}
	catch (Throwable ex) {
		if (ex instanceof BeanCreationException && beanName.equals(((BeanCreationException) ex).getBeanName())) {
			throw (BeanCreationException) ex;
		}
		else {
			throw new BeanCreationException(
				mbd.getResourceDescription(), beanName, "Initialization of bean failed", ex);
			}
		}
	...
	//为应用返回所需要的实例对象
	return exposedObject;
}
```

从上面的代码中知道，为 Bean 实例对象添加 BeanPostProcessor 后置处理器的入口的是 initializeBean 方法。

## 4.3 initializeBean 方法为容器产生的 Bean 实例对象添加 BeanPostProcessor 后置处理器

同样在 AbstractAutowireCapableBeanFactory 类中，initializeBean 方法实现为容器创建的 Bean 实例对象添加 BeanPostProcessor 后置处理器，源码如下：

```java
//初始容器创建的 Bean 实例对象，为其添加 BeanPostProcessor 后置处理器
protected Object initializeBean(final String beanName, final Object bean, RootBeanDefinition mbd) {
	//JDK 的安全机制验证权限
	if (System.getSecurityManager() != null) {
		//实现 PrivilegedAction 接口的匿名内部类
		AccessController.doPrivileged(new PrivilegedAction<Object>() {
			@Override
			public Object run() {
				invokeAwareMethods(beanName, bean);
				return null;
			}
		}, getAccessControlContext());
	}
	else {
		//为 Bean 实例对象包装相关属性，如名称，类加载器，所属容器等信息
		invokeAwareMethods(beanName, bean);
	}
	Object wrappedBean = bean;
	//对 BeanPostProcessor 后置处理器的 postProcessBeforeInitialization
	//回调方法的调用，为 Bean 实例初始化前做一些处理
	if (mbd == null || !mbd.isSynthetic()) {
		wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
	}
	//调用 Bean 实例对象初始化的方法，这个初始化方法是在 Spring Bean 定义配置
	//文件中通过 init-Method 属性指定的
	try {
		invokeInitMethods(beanName, wrappedBean, mbd);
	}
	catch (Throwable ex) {
		throw new BeanCreationException(
				(mbd != null ? mbd.getResourceDescription() : null),
				beanName, "Invocation of init method failed", ex);
	}
	//对 BeanPostProcessor 后置处理器的 postProcessAfterInitialization
	//回调方法的调用，为 Bean 实例初始化之后做一些处理
	if (mbd == null || !mbd.isSynthetic()) {
		wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
	}
	return wrappedBean;
}
//调用 BeanPostProcessor 后置处理器实例对象初始化之前的处理方法
public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
		throws BeansException {
	Object result = existingBean;
	//遍历容器为所创建的 Bean 添加的所有 BeanPostProcessor 后置处理器
	for (BeanPostProcessor beanProcessor : getBeanPostProcessors()) {
		//调用 Bean 实例所有的后置处理中的初始化前处理方法，为 Bean 实例对象在
		//初始化之前做一些自定义的处理操作
		result = beanProcessor.postProcessBeforeInitialization(result, beanName);
		if (result == null) {
			return result;
		}
	}
	return result;
}
//调用 BeanPostProcessor 后置处理器实例对象初始化之后的处理方法
public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
		throws BeansException {
	Object result = existingBean;
	//遍历容器为所创建的 Bean 添加的所有 BeanPostProcessor 后置处理器
	for (BeanPostProcessor beanProcessor : getBeanPostProcessors()) {
		//调用 Bean 实例所有的后置处理中的初始化后处理方法，为 Bean 实例对象在
		//初始化之后做一些自定义的处理操作
		result = beanProcessor.postProcessAfterInitialization(result, beanName);
		if (result == null) {
			return result;
		}
	}
	return result;
}
```

BeanPostProcessor 是一个接口，其初始化前的操作方法和初始化后的操作方法均委托其实现子类来实现，在 Spring 中，BeanPostProcessor 的实现子类非常的多，分别完成不同的操作，如：AOP 面向切面编程的注册通知适配器、Bean 对象的数据校验、Bean 继承属性/方法的合并等等，我们以最简单的 AOP 切面织入来简单了解其主要的功能。

## 4.4 AdvisorAdapterRegistrationManager 在 Bean 对象初始化后注册通知适配器

`AdvisorAdapterRegistrationManager` 是 BeanPostProcessor 的一个实现类，其主要的作用`为容器中管理的 Bean 注册一个面向切面编程的通知适配器`，以便在 Spring 容器为所管理的 Bean 进行面向切面编程时提供方便，其源码如下：

```java
//为容器中管理的 Bean 注册一个面向切面编程的通知适配器
public class AdvisorAdapterRegistrationManager implements BeanPostProcessor {
    //容器中负责管理切面通知适配器注册的对象
    private AdvisorAdapterRegistry advisorAdapterRegistry = GlobalAdvisorAdapterRegistry.getInstance();
    public AdvisorAdapterRegistrationManager() {
    }
    public void setAdvisorAdapterRegistry(AdvisorAdapterRegistry advisorAdapterRegistry) {
        //如果容器创建的 Bean 实例对象是一个切面通知适配器，则向容器的注册
        this.advisorAdapterRegistry = advisorAdapterRegistry;
    }
    //BeanPostProcessor 在 Bean 对象初始化后的操作
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
    //BeanPostProcessor 在 Bean 对象初始化前的操作
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if (bean instanceof AdvisorAdapter) {
            this.advisorAdapterRegistry.registerAdvisorAdapter((AdvisorAdapter)bean);
        }
        //没有做任何操作，直接返回容器创建的 Bean 对象
        return bean;
    }
}
```

其他的 BeanPostProcessor 接口实现类的也类似，都是对 Bean 对象使用到的一些特性进行处理，或者向 IOC 容器中注册，为创建的 Bean 实例对象做一些自定义的功能增加，这些操作是容器初始化 Bean时自动触发的，不需要认为的干预。

# 5 Bean 依赖关系管理


Spring IOC 容器提供了两种管理 Bean 依赖关系的方式：

- 显式管理：通过 BeanDefinition 的属性值和构造方法实现 Bean 依赖关系管理 
- autowiring：Spring IOC 容器的依赖自动装配功能，不需要对 Bean 属性的依赖关系做显式的声明，只需要在配置好 autowiring 属性，IOC 容器会自动使用反射查找属性的类型和名称，然后基于属性的类型或者名称来自动匹配容器中管理的 Bean，从而自动地完成依赖注入。

通过对 autowiring 自动装配特性的理解，我们知道容器对 Bean 的自动装配发生在容器对 Bean 依赖注入的过程中。在前面对 Spring IOC 容器的依赖注入过程源码分析中，我们已经知道了容器对 Bean实例对象的属性注入的处理发生在 AbstractAutoWireCapableBeanFactory 类中的 populateBean方法中，我们通过程序流程分析 autowiring 的实现原理：

## 5.1 AbstractAutoWireCapableBeanFactory 对 Bean 实例进行属性依赖注入

应用第一次通过 getBean 方法(配置了 lazy-init 预实例化属性的除外)向 IOC 容器索取 Bean 时，容器创建 Bean 实例对象 ， 并且对 Bean 实例对象进行属性依赖注入 ，AbstractAutoWireCapableBeanFactory 的 populateBean 方法就是实现 Bean 属性依赖注入的功能，其主要源码如下：

```java
//将 Bean 属性设置到生成的实例对象上
protected void populateBean(String beanName, RootBeanDefinition mbd, BeanWrapper bw) {
	PropertyValues pvs = mbd.getPropertyValues();
	if (bw == null) {
		if (!pvs.isEmpty()) {
			throw new BeanCreationException(
					mbd.getResourceDescription(), beanName, "Cannot apply property values to null instance");
		}
		else {
			// Skip property population phase for null instance.
			return;
		}
	}
	// Give any InstantiationAwareBeanPostProcessors the opportunity to modify the
	// state of the bean before properties are set. This can be used, for example,
	// to support styles of field injection.
	boolean continueWithPropertyPopulation = true;
	if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
		for (BeanPostProcessor bp : getBeanPostProcessors()) {
			if (bp instanceof InstantiationAwareBeanPostProcessor) {
				InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
				if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
					continueWithPropertyPopulation = false;
					break;
				}
			}
		}
	}
	if (!continueWithPropertyPopulation) {
		return;
	}
	//获取容器在解析 Bean 定义资源时为 BeanDefiniton 中设置的属性值
	PropertyValues pvs = (mbd.hasPropertyValues() ? mbd.getPropertyValues() : null);
	//对依赖注入处理，首先处理 autowiring 自动装配的依赖注入
	if (mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_BY_NAME ||
			mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_BY_TYPE) {
		MutablePropertyValues newPvs = new MutablePropertyValues(pvs);
		// Add property values based on autowire by name if applicable.
		//根据 Bean 名称进行 autowiring 自动装配处理
		if (mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_BY_NAME) {
			autowireByName(beanName, mbd, bw, newPvs);
		}
		// Add property values based on autowire by type if applicable.
		//根据 Bean 类型进行 autowiring 自动装配处理
		if (mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_BY_TYPE) {
			autowireByType(beanName, mbd, bw, newPvs);
		}
		pvs = newPvs;
	}
	...
}
```

## 5.2 Spring IOC 容器根据 Bean 名称或者类型进行 autowiring 自动依赖注入

```java
//根据类型对属性进行自动依赖注入
protected void autowireByType(
		String beanName, AbstractBeanDefinition mbd, BeanWrapper bw, MutablePropertyValues pvs) {
	//获取用户定义的类型转换器
	TypeConverter converter = getCustomTypeConverter();
	if (converter == null) {
		converter = bw;
	}
	//存放解析的要注入的属性
	Set<String> autowiredBeanNames = new LinkedHashSet<String>(4);
	//对 Bean 对象中非简单属性(不是简单继承的对象，如 8 中原始类型，字符
	//URL 等都是简单属性)进行处理
	String[] propertyNames = unsatisfiedNonSimpleProperties(mbd, bw);
	for (String propertyName : propertyNames) {
		try {
			//获取指定属性名称的属性描述器
			PropertyDescriptor pd = bw.getPropertyDescriptor(propertyName);
			// Don't try autowiring by type for type Object: never makes sense,
			// even if it technically is a unsatisfied, non-simple property.
			//不对 Object 类型的属性进行 autowiring 自动依赖注入
			if (Object.class != pd.getPropertyType()) {
				//获取属性的 setter 方法
				MethodParameter methodParam = BeanUtils.getWriteMethodParameter(pd);
				// Do not allow eager init for type matching in case of a prioritized post-processor.
				//检查指定类型是否可以被转换为目标对象的类型
				boolean eager = !PriorityOrdered.class.isAssignableFrom(bw.getWrappedClass());
				//创建一个要被注入的依赖描述
				DependencyDescriptor desc = new AutowireByTypeDependencyDescriptor(methodParam, eager);
				//根据容器的 Bean 定义解析依赖关系，返回所有要被注入的 Bean 对象
				Object autowiredArgument = resolveDependency(desc, beanName, autowiredBeanNames, converter);
				if (autowiredArgument != null) {
					//为属性赋值所引用的对象
					pvs.add(propertyName, autowiredArgument);
				}
				for (String autowiredBeanName : autowiredBeanNames) {
					//指定名称属性注册依赖 Bean 名称，进行属性依赖注入
					registerDependentBean(autowiredBeanName, beanName);
					if (logger.isDebugEnabled()) {
						logger.debug("Autowiring by type from bean name '" + beanName + "' via property '" +
								propertyName + "' to bean named '" + autowiredBeanName + "'");
					}
				}
				//释放已自动注入的属性
				autowiredBeanNames.clear();
			}
		}
		catch (BeansException ex) {
			throw new UnsatisfiedDependencyException(mbd.getResourceDescription(), beanName, propertyName, ex);
		}
	}
}
```

通过上面的源码分析，我们可以看出来通过属性名进行自动依赖注入的相对比通过属性类型进行自动依赖注入要稍微简单一些，但是真正实现属性注入的是 DefaultSingletonBeanRegistry 类的registerDependentBean 方法。

## 5.3 DefaultSingletonBeanRegistry 的 registerDependentBean 方法对属性注入

```java
//为指定的 Bean 注入依赖的 Bean
public void registerDependentBean(String beanName, String dependentBeanName) {
	// A quick check for an existing entry upfront, avoiding synchronization...
	//处理 Bean 名称，将别名转换为规范的 Bean 名称
	String canonicalName = canonicalName(beanName);
	Set<String> dependentBeans = this.dependentBeanMap.get(canonicalName);
	if (dependentBeans != null && dependentBeans.contains(dependentBeanName)) {
		return;
	}
	// No entry yet -> fully synchronized manipulation of the dependentBeans Set
	//多线程同步，保证容器内数据的一致性
	//先从容器中：bean 名称-->全部依赖 Bean 名称集合找查找给定名称 Bean 的依赖 Bean
	synchronized (this.dependentBeanMap) {
		//获取给定名称 Bean 的所有依赖 Bean 名称
		dependentBeans = this.dependentBeanMap.get(canonicalName);
		if (dependentBeans == null) {
			//为 Bean 设置依赖 Bean 信息
			dependentBeans = new LinkedHashSet<String>(8);
			this.dependentBeanMap.put(canonicalName, dependentBeans);
		}
		//向容器中：bean 名称-->全部依赖 Bean 名称集合添加 Bean 的依赖信息
		//即，将 Bean 所依赖的 Bean 添加到容器的集合中
		dependentBeans.add(dependentBeanName);
	}
	//从容器中：bean 名称-->指定名称 Bean 的依赖 Bean 集合找查找给定名称 Bean 的依赖 Bean
	synchronized (this.dependenciesForBeanMap) {
		Set<String> dependenciesForBean = this.dependenciesForBeanMap.get(dependentBeanName);
		if (dependenciesForBean == null) {
			dependenciesForBean = new LinkedHashSet<String>(8);
			this.dependenciesForBeanMap.put(dependentBeanName, dependenciesForBean);
		}
		//向容器中：bean 名称-->指定 Bean 的依赖 Bean 名称集合添加 Bean 的依赖信息
		//即，将 Bean 所依赖的 Bean 添加到容器的集合中
		dependenciesForBean.add(canonicalName);
	}
}
```

通过对 autowiring 的源码分析，我们可以看出 autowiring 的实现过程： 

1. 对 Bean 的属性代调用 getBean 方法，完成依赖 Bean 的初始化和依赖注入
2. 将依赖 Bean 的属性引用设置到被依赖的 Bean 属性上
3. 将依赖 Bean 的名称和被依赖 Bean 的名称存储在 IOC 容器的集合中

Spring IOC 容器的 autowiring 属性自动依赖注入是一个很方便的特性，可以简化开发时的配置，但是凡是都有两面性，自动属性依赖注入也有不足，首先，Bean 的依赖关系在配置文件中无法很清楚地看出来，对于维护造成一定困难。其次，由于自动依赖注入是 Spring 容器自动执行的，容器是不会智能判断的，如果配置不当，将会带来无法预料的后果，所以自动依赖注入特性在使用时还是综合考虑。

------

