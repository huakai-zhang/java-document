## MVC本质


MVC的核心思想是业务数据抽取业务数据呈现相分离 MVC，Model-View-Controller

- View，视图层：为用户提供UI重点关注数据的呈现 
- Model，模型层：业务数据的信息表示，关注支持业务的信息构成，通常是多个业务实体的组合 
- Controller，控制层：调用业务逻辑产生合适的数据(Model)传递数据给视图层用于呈现

>区别于，三层机构，Dao数据访问层，Service业务处理层，Web层（J2EE的内容，Request和Response）


## Spring MVC 请求处理流程



引用 Spring in Action 上的一张图来说明了 SpringMVC 的核心组件和请求处理流程： ![img](https://img-blog.csdnimg.cn/20200401100000571.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) ![img](https://img-blog.csdnimg.cn/20200401103111107.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) (1) DispatcherServlet（实现Awaer接口，能够得到ApplicationContext）是springmvc中的前端控制器(front controller)，负责接收request并将request转发给对应的处理组件 (2) HanlerMapping是springmvc中完成url到controller映射的组件，DispatcherServlet接收request，然后从HandlerMapping查找处理request的controller (3) Controller处理request，并返回ModelAndView对象，Controller是springmvc中负责处理request的组件，ModelAndView是封装结果视图的组件 (4) (5) (6)视图解析器解析ModelAndView对象返回对应的视图给客户端

### 流程简述

#### 初始化

1. web.xml配置DispatcherServlet(启动入口) 
2. 默认加载IOC容器（ApplicationContext） 
3. 开始扫描SpringMVC的配置，一般来说扫描注解（Controller/RequestMapping/ResponseBody），View的配置，插件（拦截器，转换器，视图解析器） 
4. 解析成一个HandlerMapping的List，主要是保存了URL和具体的执行方法的对应关系

#### 等待用户请求的过程

1. 从浏览器中输入URL 
2. 统一拦截（扫描HandlerMapping），如果是404/500 
3. DispatcherServlet接收到请求从上面初始化已经保存的数据中找到请求URL对应方法然后调用 
4. 把响应结果输出

#### Serlvet的执行过程

1. 从web.xml开始，在这个文件里配置了n个Servlet，一般而言一个Servlet对应一个url，以后要增加功能，增加URL，每次都要去修改配置文件，增加Servlet配置，导致配置膨胀，代码膨胀 
2. 返回结果，在Servlet里面直接输入HTML

## Spring MVC 工作机制

在容器初始化时会建立所有url和controller的对应关系，保存到Map<url, controller>中。Tomcat启动时会通知Spring初始化容器（加载bean的定义信息和初始化所有单例bean，然后springmvc会遍历容器中的bean），获取每一个controller中的方法访问的url，然后将url和controller保存到一个Map中； 这样就可以根据request快速定位到controller，因为最终处理request的是controller中的方法，Map中只保留了url和controller中的对应关系，所以要根据request的url进一步确认controller中的method。这一步工作的原理就是拼接controller的url（controller上@RequestMapping的值）和方法的url（metnod上的@RequestMapping的值），与request的url进行匹配，找到匹配的那个方法； 确定处理请求的method后，接下来的任务就是要参数绑定，把request中参数绑定到方法的形式参数上，这一步是整个请求处理过程中最复杂的一个步骤。springmvc提供了两种request参数与方法形参的绑定方法：

1. 通过注解进行绑定@RequestParam 使用注解进行绑定,我们只要在方法参数前面声明@RequestParam(“a”),就可以将request中参数a的值绑定到方法的该参数上。RequestParam，自动调用request的getParament方法，而且能够自动转型，getParament获得是字符转（自动转为其他类型） 
<li>通过参数名称进行绑定 使用参数名称进行绑定的前提是必须要获取方法中参数的名称,Java反射只提供了获取方法的参数的类型,并没有提供获取参数名称的方法.springmvc解决这个问题的方法是用asm框架读取字节码文件,来获取方法的参数名称.asm框架是一个字节码操作框架,关于asm更多介绍可以参考它的官网.个人建议,使用注解来完成参数绑定,这样就可以省去asm框架的读取字节码的操作</li>


根据工作机制中三部分来分析springmvc的源代码：

- ApplicationContext初始化时建立所有url和controller类的对应关系(用Map保存) 
- 根据请求url找到对应的controller,并从controller中找到处理请求的方法 
- request参数绑定到方法的形参,执行方法处理请求,并返回结果视图

## 建立Map<urls,controller>的关系

我们首先看第一个步骤,也就是建立Map<url,controller>关系的部分.第一部分的入口类为ApplicationObjectSupport的setApplicationContext方法.setApplicationContext方法中核心部分就是初始化容器initApplicationContext(context),子类AbstractDetectingUrlHandlerMapping实现了该方法,所以我们直接看子类中的初始化容器方法.

```java
@Override
public void initApplicationContext() throws ApplicationContextException {
   super.initApplicationContext();
   detectHandlers();
}
// 建立当前ApplicationContext中的所有controller和url的对应关系
protected void detectHandlers() throws BeansException {
   if (logger.isDebugEnabled()) {
      logger.debug("Looking for URL mappings in application context: " + getApplicationContext());
   }
   // 获取ApplicationContext容器中所有bean的Name
   String[] beanNames = (this.detectHandlersInAncestorContexts ?
         BeanFactoryUtils.beanNamesForTypeIncludingAncestors(getApplicationContext(), Object.class) :
         getApplicationContext().getBeanNamesForType(Object.class));

   // Take any bean name that we can determine URLs for.
   // 遍历beanNames，并找到这些bean对应的url
   for (String beanName : beanNames) {
      // 找bean上的所有url(controller上url+方法上的url)，该方法由对应的子类实现
      String[] urls = determineUrlsForHandler(beanName);
      if (!ObjectUtils.isEmpty(urls)) {
         // URL paths found: Let's consider it a handler.
         // 保存urls和beanName的对应关系，put it to Map<urls, beanName>，该方法
         // 在父类AbstractUrlHandlerMapping中实现
         registerHandler(urls, beanName);
      }
      else {
         if (logger.isDebugEnabled()) {
            logger.debug("Rejected bean name '" + beanName + "': no URL paths identified");
         }
      }
   }
}
// 获取controller中所有方法的url，由子类实现，典型的模版模式
protected abstract String[] determineUrlsForHandler(String beanName);
```

determineUrlsForHandler(String beanName)方法的作用是获取每个controller中的url，不同的子类有不同的实现，这是一个典型的模版设计模式。因为开发中我们用的最多的就是注解来配置controller中的url，DefaultAnnotationHandlerMapping是AbstractDetectingUrlHandlerMapping的子类，处理注解形式的url映射，所以这里以DefaultAnnotationHandlerMapping来进行分析：

```java
// 获取controller中的鄋url
protected String[] determineUrlsForHandler(String beanName) {
   // 获取ApplicationContext容器
   ApplicationContext context = getApplicationContext();
   // 从容器中获取controller
   Class<?> handlerType = context.getType(beanName);
   // 获取controller上的@RequestMapping注解
   RequestMapping mapping = context.findAnnotationOnBean(beanName, RequestMapping.class);
   // controller上有注解
   if (mapping != null) {
      // @RequestMapping found at type level
      this.cachedMappings.put(handlerType, mapping);
      // 返回结果集
      Set<String> urls = new LinkedHashSet<String>();
      // controller的映射url
      String[] typeLevelPatterns = mapping.value();
      if (typeLevelPatterns.length > 0) {
         // @RequestMapping specifies paths at type level
         // 获取controller中所有方法及方法的映射url
         String[] methodLevelPatterns = determineUrlsForHandlerMethods(handlerType, true);
         for (String typeLevelPattern : typeLevelPatterns) {
            if (!typeLevelPattern.startsWith("/")) {
               typeLevelPattern = "/" + typeLevelPattern;
            }
            boolean hasEmptyMethodLevelMappings = false;
            for (String methodLevelPattern : methodLevelPatterns) {
               if (methodLevelPattern == null) {
                  hasEmptyMethodLevelMappings = true;
               }
               else {
                  // controller的映射url+方法映射的url
                  String combinedPattern = getPathMatcher().combine(typeLevelPattern, methodLevelPattern);
                  // 保存到set集合中
                  addUrlsForPath(urls, combinedPattern);
               }
            }
            if (hasEmptyMethodLevelMappings ||
                  org.springframework.web.servlet.mvc.Controller.class.isAssignableFrom(handlerType)) {
               addUrlsForPath(urls, typeLevelPattern);
            }
         }
         // 以数组星矢返回controller上的所有url
         return StringUtils.toStringArray(urls);
      }
      else {
         // actual paths specified by @RequestMapping at method level
         // controller上的@RequestMapping映射url为空串，直接找方法的映射url
         return determineUrlsForHandlerMethods(handlerType, false);
      }
   }//controller上没有@RequestMapping注解
   else if (AnnotationUtils.findAnnotation(handlerType, Controller.class) != null) {
      // @RequestMapping to be introspected at method level
      // 获取controller中方法上的映射url
      return determineUrlsForHandlerMethods(handlerType, false);
   }
   else {
      return null;
   }
}
```

到这里HandlerMapping组件就已经建立所有url和controller的对应关系。

## 根据访问url找到对应controller中处理请求的方法

初始化过程：

```java
@Override
// 只要IOC容器启动以后，就会调用onRefresh方法
protected void onRefresh(ApplicationContext context) {
   initStrategies(context);
}
protected void initStrategies(ApplicationContext context) {
   //请求解析
   initMultipartResolver(context);
   //多语言、国际化
   initLocaleResolver(context);
   //主题View层的
   initThemeResolver( context);
   //解析url和Method的关联关系
   initHandlerMappings( context);
   //适配器( 匹配的过程)
   initHandlerAdapters ( context);
   //异常解析
   initHandlerExceptionResolvers (context);
   //视图转发(根据视图名字匹配到一个具体模板)
   initRequestToViewNameTranslator(context);
   //解析模板中的内容(拿到服务器传过来的数据，生成HTML代码)
   initViewResolvers ( context);
   initFlashMapManager(context);
}
private void initHandlerMappings(ApplicationContext context) {
   this.handlerMappings = null;

   if (this.detectAllHandlerMappings) {
      // Find all HandlerMappings in the ApplicationContext, including ancestor contexts.
      // 从ApplicationContext获取所有的HandlerMappings
      Map<String, HandlerMapping> matchingBeans =
            BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false);
      if (!matchingBeans.isEmpty()) {
         // Map转换为List
         this.handlerMappings = new ArrayList<HandlerMapping>(matchingBeans.values());
         // We keep HandlerMappings in sorted order.
         OrderComparator.sort(this.handlerMappings);
      }
   }
   else {
      try {
         HandlerMapping hm = context.getBean(HANDLER_MAPPING_BEAN_NAME, HandlerMapping.class);
         this.handlerMappings = Collections.singletonList(hm);
      }
      catch (NoSuchBeanDefinitionException ex) {
         // Ignore, we'll add a default HandlerMapping later.
      }
   }

   // Ensure we have at least one HandlerMapping, by registering
   // a default HandlerMapping if no other mappings are found.
   if (this.handlerMappings == null) {
      this.handlerMappings = getDefaultStrategies(context, HandlerMapping.class);
      if (logger.isDebugEnabled()) {
         logger.debug("No HandlerMappings found in servlet '" + getServletName() + "': using default");
      }
   }
}
```

下面开始分析第二个步骤,第二个步骤是由请求触发的,所以入口为DispatcherServlet的核心方法为doService(),doService()中的核心逻辑由doDispatch()实现,我们查看doDispatch()的源代码。

```java
/** 中央控制器,控制请求的转发 **/
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
   HttpServletRequest processedRequest = request;
   HandlerExecutionChain mappedHandler = null;
   boolean multipartRequestParsed = false;

   WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

   try {
      ModelAndView mv = null;
      Exception dispatchException = null;

      try {
         //1.检查是否是复合请求（文件流（多媒体））
         processedRequest = checkMultipart(request);
         multipartRequestParsed = processedRequest != request;

         // Determine handler for the current request.
         // 取得处理当前请求的controller,这里也称为hanlder处理器
         // 第一个步骤的意义就在这里体现了.这里并不是直接返回controller
         // 而是返回的HandlerExecutionChain请求处理器链对象,该对象封装了handler和interceptors.
         mappedHandler = getHandler(processedRequest, false);
         // 如果handler为空,则返回404
         if (mappedHandler == null || mappedHandler.getHandler() == null) {
            noHandlerFound(processedRequest, response);
            return;
         }

         // Determine handler adapter for the current request.
         //3. 获取处理request的处理器适配器handler adapter
         HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

         // Process last-modified header, if supported by the handler.
         // 处理 last-modified 请求头，判断request的请求类型，request带过来的所有信息
         String method = request.getMethod();
         boolean isGet = "GET".equals(method);
         if (isGet || "HEAD".equals(method)) {
            
            long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
            if (logger.isDebugEnabled()) {
               String requestUri = urlPathHelper.getRequestUri(request);
               logger.debug("Last-Modified value for [" + requestUri + "] is: " + lastModified);
            }
            if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
               return;
            }
         }

         // 4.拦截器的预处理方法
         if (!mappedHandler.applyPreHandle(processedRequest, response)) {
            return;
         }

         try {
            // Actually invoke the handler.
            // 5.实际的处理器处理请求,返回结果视图对象
            mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
         }
         finally {
            if (asyncManager.isConcurrentHandlingStarted()) {
               return;
            }
         }

         // 结果视图对象的处理
         applyDefaultViewName(request, mv);
         // 6.拦截器的后处理方法
         mappedHandler.applyPostHandle(processedRequest, response, mv);
      }
      catch (Exception ex) {
         dispatchException = ex;
      }
      processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
   }
   catch (Exception ex) {
      // 请求成功响应之后的方法
      triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
   }
   catch (Error err) {
      triggerAfterCompletionWithError(processedRequest, response, mappedHandler, err);
   }
   finally {
      if (asyncManager.isConcurrentHandlingStarted()) {
         // Instead of postHandle and afterCompletion
         mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
         return;
      }
      // Clean up any resources used by a multipart request.
      if (multipartRequestParsed) {
         cleanupMultipart(processedRequest);
      }
   }
}
```

getHandler(processedRequest)方法实际上就是从HandlerMapping中找到url和controller的对应关系。这也就是建立Map<url, Controller>的意义，我们知道最终处理request的是controller中的方法，我们现在只是知道了controller，还要进一步确认controller中处理request的方法。由于下面的步骤和第三个步骤关系更加紧密，直接转到第三个步骤。

## 反射调用处理请求的方法返回结果视图

上面的方法中,第2步其实就是从第一个步骤中的Map&lt;urls,beanName&gt;中取得controller,然后经过拦截器的预处理方法,到最核心的部分–图中第5步调用controller的方法处理请求.在第2步中我们可以知道处理request的controller,第5步就是要根据url确定controller中处理请求的方法,然后通过反射获取该方法上的注解和参数,解析方法和参数上的注解,最后反射调用方法获取ModelAndView结果视图。因为上面采用注解url形式说明的,所以我们这里继续以注解处理器适配器来说明.第5步调用的就是AnnotationMethodHandlerAdapter的handle().handle()中的核心逻辑由invokeHandlerMethod(request, response, handler)实现。

```java
/** 获取处理请求的方法,执行并返回结果视图 **/
protected ModelAndView invokeHandlerMethod(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
   // 1.获取方法解析器
   ServletHandlerMethodResolver methodResolver = getMethodResolver(handler);
   // 2.解析request中的url,获取处理request的方法 
   Method handlerMethod = methodResolver.resolveHandlerMethod(request);
   // 3.方法调用器
   ServletHandlerMethodInvoker methodInvoker = new ServletHandlerMethodInvoker(methodResolver);
   ServletWebRequest webRequest = new ServletWebRequest(request, response);
   ExtendedModelMap implicitModel = new BindingAwareModelMap();
   // 4.执行方法
   Object result = methodInvoker.invokeHandlerMethod(handlerMethod, handler, webRequest, implicitModel);
   // 5.封装结果视图
   ModelAndView mav = methodInvoker.getModelAndView(handlerMethod, handler.getClass(), result, implicitModel, webRequest);
   methodInvoker.updateModelAttributes(handler, (mav != null ? mav.getModel() : null), implicitModel, webRequest);
   return mav;
}
```

这一部分的核心就在2和4了.先看第2步,通过request找controller的处理方法.实际上就是拼接controller的url和方法的url,与request的url进行匹配,找到匹配的方法.

```java
// 根据url获取处理请求的方法
public Method resolveHandlerMethod(HttpServletRequest request) throws ServletException {
   // 如果请求url为,localhost:8080/springmvc/helloWorldController/say.action, 则lookupPath=helloWorldController/say.action
   String lookupPath = urlPathHelper.getLookupPathForRequest(request);
   Comparator<String> pathComparator = pathMatcher.getPatternComparator(lookupPath);
   Map<RequestSpecificMappingInfo, Method> targetHandlerMethods = new LinkedHashMap<RequestSpecificMappingInfo, Method>();
   Set<String> allowedMethods = new LinkedHashSet<String>(7);
   String resolvedMethodName = null;
   // 遍历controller上的所有方法,获取url匹配的方法
   for (Method handlerMethod : getHandlerMethods()) {
      RequestSpecificMappingInfo mappingInfo = new RequestSpecificMappingInfo(this.mappings.get(handlerMethod));
      boolean match = false;
      // 获取方法上的url
      if (mappingInfo.hasPatterns()) {
         // 方法上可能有多个url,springmvc支持方法映射多个url
         for (String pattern : mappingInfo.getPatterns()) {
            if (!hasTypeLevelMapping() && !pattern.startsWith("/")) {
               pattern = "/" + pattern;
            }
            // 获取controller上的映射和url和方法上的url,拼凑起来与lookupPath是否匹配
            String combinedPattern = getCombinedPattern(pattern, lookupPath, request);
            if (combinedPattern != null) {
               if (mappingInfo.matches(request)) {
                  match = true;
                  mappingInfo.addMatchedPattern(combinedPattern);
               }
               else {
                  if (!mappingInfo.matchesRequestMethod(request)) {
                     allowedMethods.addAll(mappingInfo.methodNames());
                  }
                  break;
               }
            }
         }
         mappingInfo.sortMatchedPatterns(pathComparator);
      }
      else if (useTypeLevelMapping(request)) {
      // other     
}
```

通过上面的代码，已经可以找到处理request的controller中的方法了，现在看如何解析该方法上的参数,并调用该方法。也就是执行方法这一步。执行方法这一步最重要的就是获取方法的参数,然后我们就可以反射调用方法了。

```java
public final Object invokeHandlerMethod(Method handlerMethod, Object handler,
            NativeWebRequest webRequest, ExtendedModelMap implicitModel) throws Exception {
       
　　　　 Method handlerMethodToInvoke = BridgeMethodResolver.findBridgedMethod(handlerMethod);
        try {
            boolean debug = logger.isDebugEnabled();
　　　　　　　// 处理方法上的其他注解
            for (String attrName : this.methodResolver.getActualSessionAttributeNames()) {
                Object attrValue = this.sessionAttributeStore.retrieveAttribute(webRequest, attrName);
                if (attrValue != null) {
                    implicitModel.addAttribute(attrName, attrValue);
                }
            }
            for (Method attributeMethod : this.methodResolver.getModelAttributeMethods()) {
                Method attributeMethodToInvoke = BridgeMethodResolver.findBridgedMethod(attributeMethod);
                Object[] args = resolveHandlerArguments(attributeMethodToInvoke, handler, webRequest, implicitModel);
                if (debug) {
                    logger.debug("Invoking model attribute method: " + attributeMethodToInvoke);
                }
                String attrName = AnnotationUtils.findAnnotation(attributeMethod, ModelAttribute.class).value();
                if (!"".equals(attrName) && implicitModel.containsAttribute(attrName)) {
                    continue;
                }
                ReflectionUtils.makeAccessible(attributeMethodToInvoke);
                Object attrValue = attributeMethodToInvoke.invoke(handler, args);
                if ("".equals(attrName)) {
                    Class resolvedType = GenericTypeResolver.resolveReturnType(attributeMethodToInvoke, handler.getClass());
                    attrName = Conventions.getVariableNameForReturnType(attributeMethodToInvoke, resolvedType, attrValue);
                }
                if (!implicitModel.containsAttribute(attrName)) {
                    implicitModel.addAttribute(attrName, attrValue);
                }
            }
　　　　　　　// 核心代码,获取方法上的参数值
            Object[] args = resolveHandlerArguments(handlerMethodToInvoke, handler, webRequest, implicitModel);
            if (debug) {
                logger.debug("Invoking request handler method: " + handlerMethodToInvoke);
            }
            ReflectionUtils.makeAccessible(handlerMethodToInvoke);
            return handlerMethodToInvoke.invoke(handler, args);
        }
    catch (IllegalStateException ex) {
      // Internal assertion failed (e.g. invalid signature):
      // throw exception with full handler method context...
      throw new HandlerMethodInvocationException(handlerMethodToInvoke, ex);
   }
   catch (InvocationTargetException ex) {
      // User-defined @ModelAttribute/@InitBinder/@RequestMapping method threw an exception...
      ReflectionUtils.rethrowException(ex.getTargetException());
      return null;
   }
}
```

resolveHandlerArguments方法实现代码比较长,它最终要实现的目的就是:完成request中的参数和方法参数上数据的绑定. springmvc中提供两种request参数到方法中参数的绑定方式:

- 通过注解进行绑定,@RequestParam 
- 通过参数名称进行绑定.

使用注解进行绑定,我们只要在方法参数前面声明@RequestParam(“a”),就可以将request中参数a的值绑定到方法的该参数上.使用参数名称进行绑定的前提是必须要获取方法中参数的名称,Java反射只提供了获取方法的参数的类型,并没有提供获取参数名称的方法.springmvc解决这个问题的方法是用asm框架读取字节码文件,来获取方法的参数名称.asm框架是一个字节码操作框架,关于asm更多介绍可以参考它的官网.个人建议,使用注解来完成参数绑定,这样就可以省去asm框架的读取字节码的操作.

```java
public class ModelAndView {
   /** View instance or view name String */
   // 页面模版
   private Object view;

   /** Model Map */
   // 往页面上带过去的值
   private ModelMap model;
   ......
｝
public class ModelMap extends LinkedHashMap<String, Object> {
    ......
}
```


上面我们已经对 SpringMVC 的工作原理和源码进行了分析，在这个过程发现了几个优化点: 1、Controller 如果能保持单例，尽量使用单例 这样可以减少创建对象和回收对象的开销。也就是说，如果 Controller 的类变量和实例变量可以以方法形参声明的尽量以方法的形参声明，不要以类变量和实例变量声明，这样可以避免线程安全问题。 2、处理 Request 的方法中的形参务必加上@RequestParam 注解 这样可以避免 Spring MVC 使用 asm 框架读取 class 文件获取方法参数名的过程。 即便 Spring MVC 对读取出的方法参数名进行了缓存，如果不要读取 class 文件当然是更好。 3、缓存 URL 阅读源码的过程中，我们发现 Spring MVC 并没有对处理 url 的方法进行缓存，也就是说每次都要根据请求 url 去匹配 Controller 中的方法 url，如果把 url 和 Method 的关系缓存起来，会不会带来性能上的提升呢？有点恶心的是，负责解析 url 和 Method 对应关系的 ServletHandlerMethodResolver 是一个 private 的内部类，不能直接继承该类增强代码，必须要该代码后重新编译。当然，如果缓存起来，必须要考虑缓存的线程安全问题。

