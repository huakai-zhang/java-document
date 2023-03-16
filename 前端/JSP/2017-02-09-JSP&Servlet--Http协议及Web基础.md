---
layout:  post
title:   JSP&Servlet--Http协议及Web基础
date:   2017-02-09 15:33:37
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-JAVA
-java
-jsp
-网络应用
-服务器
-web

---





















```java
public class Test {
    public static void main(String[] args) throws Exception {
        Socket s = null;
        PrintWriter pw = null;
        BufferedReader br = null;
        // 建立连接端口，s指向本地机器tomcat服务器端口上
        s = new Socket("localhost", 8080);// 主机，可以是域名，也可以是ip地址 ,8080是端口
        // 对于本程序而言是输出，则相当于是准备向tomcat服务器端口写请求
        pw = new PrintWriter(new OutputStreamWriter(s.getOutputStream()));
        // 请求访问页面（此处等同于访问http://localhost:8888/）
        pw.println("GET http://localhost:8080/untitled/index.jsp HTTP/1.1\r\n");
        pw.println("Host: localhost:8080");
        pw.println("Content-Type:text/html");
        pw.println("");
        // 上一句表示请求内容结束
        pw.flush();
        // 上面这一段用于本程序向Tomcat服务器发出访问请求（get）
        // 服务器端作出响应，对于本程序而言是输入
        br = new BufferedReader(new InputStreamReader(s.getInputStream()));
        String str = "";
        // 将服务器端的响应信息打印输出（即将http://localhost:8888/页面代码源文件中的内容输出）
        // 用此方法，我们可以将我们访问到的页面的内容都拿下来
        while ((str = br.readLine()) != null) {
            System.out.println(str);
        }
    }
}
```












