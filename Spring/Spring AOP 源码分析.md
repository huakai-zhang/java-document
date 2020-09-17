## Spring中主要的AOP组件

![img](https://img-blog.csdnimg.cn/20200331211706184.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70#pic_center)


Spring提供了两种方式生成代理对象：JDKProxy和CGLib，具体使用哪种方式生成由AopProxyFactory根据AdvisedSupport对象的配置来决定。默认的策略是如果目标类是接口，则使用JDK动态代理技术，否则使用CGLib来生成代理。下面研究Spring如何使用JDK来生成代理对象，具体生成代码在JdkDynamicAopProxy这个类中：

```java
/**
 * 获取代理类要实现的接口，除了Advised对象中配置的，还会加上SpringProxy，Advised（opaque=false）
 * 检查上面得到的接口中有没有定义equals或hashcode的接口
 * 调用Proxy.newProxyInstance创建代理对象
 */
public Object getProxy(ClassLoader classLoader) {
   if (logger.isDebugEnabled()) {
      logger.debug("Creating JDK dynamic proxy: target source is " + this.advised.getTargetSource());
   }
   Class[] proxiedInterfaces = AopProxyUtils.completeProxiedInterfaces(this.advised);
   findDefinedEqualsAndHashCodeMethods(proxiedInterfaces);
   return Proxy.newProxyInstance(classLoader, proxiedInterfaces, this);
}
```


InvocationHandler是JDK动态代理的核心，生成的代理对象的方法调用都会委托到InvocationHandler.invoke()方法。通过JdkDynamicAopProxy的签名可以看到这个类也实现了InvocationHandler，下面通过分析这个类中实现的invoke()方法具体来看下Spring AOP是如何织入切面的：

```java
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
   MethodInvocation invocation;
   Object oldProxy = null;
   boolean setProxyContext = false;

   TargetSource targetSource = this.advised.targetSource;
   Class targetClass = null;
   Object target = null;

   try {
      // equals()方法，具目标对象未实现此方法
      if (!this.equalsDefined && AopUtils.isEqualsMethod(method)) {
         // The target does not implement the equals(Object) method itself.
         return equals(args[0]);
      }
      // hashCode()方法，具目标对象未实现此方法
      if (!this.hashCodeDefined && AopUtils.isHashCodeMethod(method)) {
         // The target does not implement the hashCode() method itself.
         return hashCode();
      }
      // Advised接口或其父接口中定义的方法，直接反射调用，不应用通知
      if (!this.advised.opaque && method.getDeclaringClass().isInterface() &&
            method.getDeclaringClass().isAssignableFrom(Advised.class)) {
         // Service invocations on ProxyConfig with the proxy config...
         return AopUtils.invokeJoinpointUsingReflection(this.advised, method, args);
      }

      Object retVal;

      if (this.advised.exposeProxy) {
         // Make invocation available if necessary.
         oldProxy = AopContext.setCurrentProxy(proxy);
         setProxyContext = true;
      }

      // May be null. Get as late as possible to minimize the time we "own" the target,
      // in case it comes from a pool.
      // 获得目标对象的类
      target = targetSource.getTarget();
      if (target != null) {
         targetClass = target.getClass();
      }

      // Get the interception chain for this method.
      // 获取可以应用到此方法上的Interceptor列表
      List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);

      // Check whether we have any advice. If we don't, we can fallback on direct
      // reflective invocation of the target, and avoid creating a MethodInvocation.
      // 如果没有可以应用到此方法的通知(Interceptor)，此直接反射调用method.invoke(target, args)
      if (chain.isEmpty()) {
         // We can skip creating a MethodInvocation: just invoke the target directly
         // Note that the final invoker must be an InvokerInterceptor so we know it does
         // nothing but a reflective operation on the target, and no hot swapping or fancy proxying.
         retVal = AopUtils.invokeJoinpointUsingReflection(target, method, args);
      }
      else {
         // We need to create a method invocation...
         // 创建MethodInvocation
         invocation = new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
         // Proceed to the joinpoint through the interceptor chain.
         retVal = invocation.proceed();
      }

      // Massage return value if necessary.
      Class<?> returnType = method.getReturnType();
      if (retVal != null && retVal == target && returnType.isInstance(proxy) &&
            !RawTargetAccess.class.isAssignableFrom(method.getDeclaringClass())) {
         // Special case: it returned "this" and the return type of the method
         // is type-compatible. Note that we can't help if the target sets
         // a reference to itself in another returned object.
         retVal = proxy;
      } else if (retVal == null && returnType != Void.TYPE && returnType.isPrimitive()) {
         throw new AopInvocationException("Null return value from advice does not match primitive return type for: " + method);
      }
      return retVal;
   }
   finally {
      if (target != null && !targetSource.isStatic()) {
         // Must have come from TargetSource.
         targetSource.releaseTarget(target);
      }
      if (setProxyContext) {
         // Restore old proxy.
         AopContext.setCurrentProxy(oldProxy);
      }
   }
}
```

主流程可以简述为：获取可以应用到此方法上的通知链(Interceptor Chain)，如果有，则应用通知，并执行joinpoint；如果没有，则直接反射执行joinpoint。而这里的关键是通知链是如何获取的以及它又是如何执行的。 首先，从上面代码可以看到，通知链是通过Advised.getInterceptorAndDynamicInterceptionAdvice()这个方法来获取的：

```java
public List<Object> getInterceptorsAndDynamicInterceptionAdvice(Method method, Class targetClass) {
   MethodCacheKey cacheKey = new MethodCacheKey(method);
   List<Object> cached = this.methodCache.get(cacheKey);
   if (cached == null) {
      cached = this.advisorChainFactory.getInterceptorsAndDynamicInterceptionAdvice(
            this, method, targetClass);
      this.methodCache.put(cacheKey, cached);
   }
   return cached;
}
```

可以看到，实际工作由AdvisorChainFactory.getInterceptorsAndDynamicInterceptionAdvice()这个方法来完成的，获取得到的结果被缓存。

```java
/**
 * 从提供的配置实例config中获取advisor列表，遍历处理这些advisor。如果是IntroductionAdvisor
 * 则判断此Advisor能够应用到目标类targetClass上，如果是PointcutAdvisor，则判断
 * 此Advisor能否应用到目标方法method上，将满足条件的Advisor通过AdvisorAdaptor转化成Interceptor列表返回
 */
public List<Object> getInterceptorsAndDynamicInterceptionAdvice(
      Advised config, Method method, Class targetClass) {

   // This is somewhat tricky... we have to process introductions first,
   // but we need to preserve order in the ultimate list.
   List<Object> interceptorList = new ArrayList<Object>(config.getAdvisors().length);
   // 查看是否包含IntroductionAdvisor
   boolean hasIntroductions = hasMatchingIntroductions(config, targetClass);
   // 这里实际上注册一系列AdvisorAdapter，用于将Advisor转化成MethodInterceptor
   AdvisorAdapterRegistry registry = GlobalAdvisorAdapterRegistry.getInstance();
   for (Advisor advisor : config.getAdvisors()) {
      if (advisor instanceof PointcutAdvisor) {
         // Add it conditionally.
         PointcutAdvisor pointcutAdvisor = (PointcutAdvisor) advisor;
         if (config.isPreFiltered() || pointcutAdvisor.getPointcut().getClassFilter().matches(targetClass)) {
            // 这个地方这两个方法位置可互换下
            // 将Advisor转化成Interceptor
            // MethodInterceptor一定是一个链表结构，要有顺序的
            MethodInterceptor[] interceptors = registry.getInterceptors(advisor);

            // 检查当前advisor的pointcut是否可以匹配当前方法
            MethodMatcher mm = pointcutAdvisor.getPointcut().getMethodMatcher();
            if (MethodMatchers.matches(mm, method, targetClass, hasIntroductions)) {
               if (mm.isRuntime()) {
                  // Creating a new object instance in the getInterceptors() method
                  // isn't a problem as we normally cache created chains.
                  for (MethodInterceptor interceptor : interceptors) {
                     interceptorList.add(new InterceptorAndDynamicMethodMatcher(interceptor, mm));
                  }
               }
               else {
                  interceptorList.addAll(Arrays.asList(interceptors));
               }
            }
         }
      }
      else if (advisor instanceof IntroductionAdvisor) {
         IntroductionAdvisor ia = (IntroductionAdvisor) advisor;
         if (config.isPreFiltered() || ia.getClassFilter().matches(targetClass)) {
            Interceptor[] interceptors = registry.getInterceptors(advisor);
            interceptorList.addAll(Arrays.asList(interceptors));
         }
      }
      else {
         Interceptor[] interceptors = registry.getInterceptors(advisor);
         interceptorList.addAll(Arrays.asList(interceptors));
      }
   }
   return interceptorList;
}
```

这个方法执行完后，Advised中配置能够应用到连接点或目标类的Advisor全部被转换成了MethodInterceptor，接下来看一下连接器链是怎么起作用的：

```java
......
// Check whether we have any advice. If we don't, we can fallback on direct
// reflective invocation of the target, and avoid creating a MethodInvocation.
// 如果没有可以应用到此方法的通知(Interceptor)，此直接反射调用method.invoke(target, args)
if (chain.isEmpty()) {
   // We can skip creating a MethodInvocation: just invoke the target directly
   // Note that the final invoker must be an InvokerInterceptor so we know it does
   // nothing but a reflective operation on the target, and no hot swapping or fancy proxying.
   retVal = AopUtils.invokeJoinpointUsingReflection(target, method, args);
}
else {
   // We need to create a method invocation...
   // 创建MethodInvocation
   invocation = new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
   // Proceed to the joinpoint through the interceptor chain.
   retVal = invocation.proceed();
}
......
```

从这段代码可以看出，如果得到的拦截器链为空，则直接反射调用目标方法，否则创建MethodInvocation，调用其proceed方法，触发拦截器链的执行：

```java
public Object proceed() throws Throwable {
   // We start with an index of -1 and increment early.
   // 如果Interceptor执行完了，则执行joinPoint
   if (this.currentInterceptorIndex == this.interceptorsAndDynamicMethodMatchers.size() - 1) {
      return invokeJoinpoint();
   }

   Object interceptorOrInterceptionAdvice =
         this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);
   // 如果要动态匹配joinPoint
   if (interceptorOrInterceptionAdvice instanceof InterceptorAndDynamicMethodMatcher) {
      // Evaluate dynamic method matcher here: static part will already have
      // been evaluated and found to match.
      InterceptorAndDynamicMethodMatcher dm =
            (InterceptorAndDynamicMethodMatcher) interceptorOrInterceptionAdvice;
      // 动态匹配，运行时参数是否满足匹配条件
      if (dm.methodMatcher.matches(this.method, this.targetClass, this.arguments)) {
         // 执行当前Interceptor
         return dm.interceptor.invoke(this);
      }
      else {
         // Dynamic matching failed.
         // Skip this interceptor and invoke the next in the chain.
         // 动态匹配失败时，略过当前Interceptor，调用下一个Interceptor
         return proceed();
      }
   }
   else {
      // It's an interceptor, so we just invoke it: The pointcut will have
      // been evaluated statically before this object was constructed.
      // 执行当前Interceptor
      return ((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this);
   }
}
```

1. 加载配置信息，解析成AopConfig 
2. 交给AopProxyFactory，调用一个createAopProxy方法 
3. JdkDynamicAopPorxy调用AdvisedSuport的getInterceptorsAndDynamicInterceptionAdvice方法得到一个方法拦截器链，并保存到一个容器（List） 
4. 递归执行拦截器方法proceed()

