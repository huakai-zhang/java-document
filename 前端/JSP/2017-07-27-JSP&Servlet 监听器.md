---
layout:  post
title:   JSP&Servlet 监听器
date:   2017-07-27 11:57:22
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-JAVA

---

Servlet规范中定义的一种特殊类，用于监听servletcontext，HttpSession和servletRequest等域对象的创建于销毁事件，用于监听域对象的属性发生修改的事件，可以在事件发生前，发生后做一些必要的处理。


![img](https://img-blog.csdn.net/20170727115100675?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)








```java
//销毁
public void contextDestroyed(ServletContextEvent arg0) {
System.out.println("Destroyed");
}
//初始化
public void contextInitialized(ServletContextEvent arg0) {
System.out.println("Initialized");
}
```

```java
<listener>
        <listener-class>cn.hpu.edu.FirstListener</listener-class>
    </listener>
```

监听器的启动顺序：


![img](https://img-blog.csdn.net/20170727115156110?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)










```java
<context-param>
   <param-name>initParam</param-name>
   <param-value>Java</param-value>
   </context-param>
```





```java
public void sessionCreated(HttpSessionEvent arg0) {
System.out.println("sessionCreated");
}
public void sessionDestroyed(HttpSessionEvent arg0) {
System.out.println("sessionDestroyed");
}
```






```java
<session-config>
    <session-timeout>1</session-timeout>
    </session-config>     //设置1分钟后session过期
```



```java
public void requestDestroyed(ServletRequestEvent arg0) {
System.out.println("requestDestroyed");
}
@Override
public void requestInitialized(ServletRequestEvent arg0) {
System.out.println("requestInitialized");
}
```












![img](https://img-blog.csdn.net/20170727115359104?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

增加 删除 替换的方法




































session钝化机制：


![img](https://img-blog.csdn.net/20170727115511257?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)







默认情况下，tomcat提供2个钝化驱动类：org.apache.catalina.FileStore和org.apache.Catalina.JDBCStore。


![img](https://img-blog.csdn.net/20170727115541052?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



```java
public void valueBound(HttpSessionBindingEvent httpsessionbindingevent) {
System.out.println("valueBound Name:"+httpsessionbindingevent.getName());
}
public void valueUnbound(HttpSessionBindingEvent httpsessionbindingevent) {
System.out.println("valueUnbound Name:"+httpsessionbindingevent.getName());
}
//钝化
public void sessionWillPassivate(HttpSessionEvent httpsessionevent) {
System.out.println("sessionWillPassivate "+httpsessionevent.getSource());
}
//活化
public void sessionDidActivate(HttpSessionEvent httpsessionevent) {
System.out.println("sessionDidActivate "+httpsessionevent.getSource());
}
```












&nbsp;![img](https://img-blog.csdn.net/20170727115638431?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)


![img](https://img-blog.csdn.net/20170727115651209?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

