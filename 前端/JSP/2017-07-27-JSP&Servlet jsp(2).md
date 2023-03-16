---
layout:  post
title:   JSP&Servlet jsp(2)
date:   2017-07-27 11:37:27
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-JAVA
-serlvet
-java
-jsp

---



















```java
<welcome-file-list>
        <welcome-file>index.html</welcome-file>
        <welcome-file>index.htm</welcome-file>
        <welcome-file>index.jsp</welcome-file>
        </welcome-file-list>
```














&nbsp; &nbsp; &nbsp; &nbsp; jspservice()方法被表用来处理客户端的请求。对每个请求，JSP引擎创建一个新的线程来处理该请求。如果有多个客户端同时请求该jsp文件，则jsp引擎会创建多个线程。每个客户端请求对应一个线程。以多线程方式执行可以大大降低对系统的资源需求，提高系统的并发量及响应时间。但也要注意多线程的编程带来的同步问题，由于该servlet始终驻于内存，所以响应是非常快的。


![img](https://img-blog.csdn.net/20170727113345781?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

















```java
<%
        out.println("<h2>静夜思</h2>");
        out.println("床前明月光<br>");
        out.println("疑是地上霜<br>");
        out.flush();
        //out.clear();//这里会抛出异常。
        out.clearBuffer();//这里不会抛出异常。
        out.println("举头望明月<br>");
        out.println("低头思故乡<br>");
        %>
```




















&nbsp; &nbsp; &nbsp; &nbsp; String request.getContextPath() &nbsp; 返回上下文路径


![img](https://img-blog.csdn.net/20170727113515607?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)











&nbsp; &nbsp; &nbsp; &nbsp;&nbsp;![img](https://img-blog.csdn.net/20170727113554624?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

**pageContext对象：**



&nbsp; &nbsp; &nbsp; &nbsp; pageContext.getSession().getAttribute();


![img](https://img-blog.csdn.net/20170727113619754?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)





&nbsp; &nbsp; &nbsp; &nbsp; Enumeration getInitParameterNames() &nbsp; 返回servlet初始化所需所有参数的枚举


![img](https://img-blog.csdn.net/20170727113648562?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)


