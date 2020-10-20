#### Q1 出Springboot项目启动的几种方式吗？

1. main

2. java -jar jar包路径 --参数(--server.port=8081)

3. spring-boot-maven-plugin

   mvn spring-boot:run 

   mvn spring-boot:help -Ddetail 

   mvn spring-boot:run -Drun.arguments="--server.port=8888" 



#### Q2 SpringMVC 工作原来？

1. 用户向服务器发送请求，请求被SpringMVC的前端控制前DispatchServlet捕获
2. DispatchServlet收到请求调用处理器映射器HandlerMapping，根据请求URL找到具体的处理器，生成处理器执行链HandlerExecutionChain(包括处理对象和处理器拦截器)，一并返回给DispatchServlet
3. DispatchServlet根据处理器Handler获得处理器适配器HandlerAdapter，执行HandlerAdapter处理一系列的操作，如：参数封装，数据格式转换，数据验证等
4. 执行处理器Handler(Controller，也叫页面控制器)，Handler执行完成返回ModelAndView
5. HandlerAdapter将Handler执行结果ModelAndView返回到DispatcherServlet
6. DispatcherServlet将ModelAndView传给ViewReslover视图解析器，ViewReslover解析后返回具体View
7. DispatcherServlet对View进行渲染视图（即将模型数据model填充至视图中），DispatcherServlet响应用户

![1603186872884](Spring.assets/1603186872884.png)

Controller		RequestMapping

RequestBody		PathVariable		RequestParam

ResponseBody



#### Q3 简单介绍一下 Spring bean 的生命周期



