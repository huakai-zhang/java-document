---
layout:  post
title:   JSP&Servlet 过滤器
date:   2017-07-27 11:47:27
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-JAVA
-web开发
-实例
-jsp
-java

---






```java
<filter>
        <filter-name>FristFilter</filter-name>
        <filter-class>com.filter.FristFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>FristFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```
```java
HttpServletRequest req =(HttpServletRequest) request;
HttpServletResponse response2 =(HttpServletResponse) response;
//重定向
response2.sendRedirect(req.getContextPath()+"/main.jsp");
//转发
//req.getRequestDispatcher("main.jsp").forward(request, response);
//req.getRequestDispatcher("main.jsp").include(request, response);
```










**过滤器链**


![img](https://img-blog.csdn.net/20170727114223998?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)









```java
<filter-mapping>
        <filter-name>FirstFilter</filter-name>
        <url-pattern>/main.jsp</url-pattern>
    </filter-mapping>
```









```java
<error-page>
    <error-code>404</error-code>
    <location>/error.jsp</location>
  </error-page>
  <filter>
        <filter-name>ErrorFilter</filter-name>
        <filter-class>com.imooc.filter.ErrorFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>ErrorFilter</filter-name>
        <url-pattern>/error.jsp</url-pattern>
        <dispatcher>ERROR</dispatcher>
    </filter-mapping>
```





在3.0中加入了一个@WebFilter的方式：用于将一个类声明为过滤器，该注解将会在部署时被容器处理，容器将根据具体的属性配置将相应的类部署为过滤器。


![img](https://img-blog.csdn.net/20170727114356182?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)


![img](https://img-blog.csdn.net/20170727114414677?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

```java
@WebFilter(filterName="AsynFilter",asyncSupported=true,value={"/servlet/AsynServlet"},dispatcherTypes={DispatcherType.REQUEST,DispatcherType.ASYNC})
```


```java
public void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("Servlet执行开始时间:" + new Date());
        AsyncContext context = request.startAsync();
        new Thread(new Executor(context)).start();
        System.out.println("Servlet执行结束时间:" + new Date());
    }

    public class Executor implements Runnable {
        private AsyncContext context;

        public Executor(AsyncContext context) {
            this.context = context;
        }

        @Override
        public void run() {
            //执行相关复杂业务
            try {
                Thread.sleep(1000 * 10);
                //context.getRequest();
                //context.getResponse();
                System.out.println("业务执行完成时间:" + new Date());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
```

```java
<servlet-name>AsynServlet</servlet-name>
    <servlet-class>com.imooc.servlet.AsynServlet</servlet-class>
        <async-supported>true</async-supported>
  </servlet>
```
运行servlet：


![img](https://img-blog.csdn.net/20170727114620424?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



```java
private FilterConfig config;

    public void doFilter(ServletRequest arg0, ServletResponse arg1, FilterChain arg2) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) arg0;
        HttpServletResponse response = (HttpServletResponse) arg1;
        HttpSession session = request.getSession();
        String noLoginPaths = config.getInitParameter("noLoginPaths");//获取不予与过滤的页面
        String charset = config.getInitParameter("charset");//获取编码方式
        if (charset == null) {
            charset = "UTF-8";//设置默认为utf-8
        }                          //解决编码乱码问题
        request.setCharacterEncoding(charset);
        if (noLoginPaths != null) {
            String[] strArray = noLoginPaths.split(";");
            for (int i = 0; i < strArray.length; i++) {
                if (strArray[i] == null || "".equals(strArray[i])) continue;
                if (request.getRequestURI().indexOf(strArray[i]) != -1) {
                    arg2.doFilter(arg0, arg1);
                    return;
                }
            }
        }
        if (session.getAttribute("username") != null) {
            arg2.doFilter(arg0, arg1);
        } else {
            response.sendRedirect("login.jsp");
        }
    }

    @Override
    public void init(FilterConfig arg0) throws ServletException {
        config = arg0;
    }
```

```java
<filter>
        <filter-name>LoginFilter</filter-name>
        <filter-class>com.imooc.filter.LoginFilter</filter-class>
        <init-param>
            <param-name>noLoginPaths</param-name>
            <param-value>login.jsp;fail.jsp;LoginServlet</param-value> //不予与过滤的页面                                        
</init-param>
        <init-param>
            <param-name>charset</param-name>
            <param-value>UTF-8</param-value>//配置编码方式                       </init-param>
    </filter>
    <filter-mapping>
        <filter-name>LoginFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

