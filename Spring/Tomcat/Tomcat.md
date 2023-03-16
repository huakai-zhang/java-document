## 架构图


![img](https://img-blog.csdnimg.cn/20200325103226139.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

## 目录结构

### conf

catalina.policy：Tomcat 安全策略文件，控制 JVM 相关权限，具体可以参考java.security.Permission catalina.properties：Tomcat Catalina 行为控制配置文件，比如Common ClassLoader logging.properties：Tomcat 日志配置文件，JDK Logging server.xml：Tomcat Server 配置文件

- GlobalNamingResources：全局JNDI资源

context.xml：全局 Context 配置文件 tomcat-users.xml：Tomcat角色配置文件（Realm文件实现方式） web.xml：Servlet 标准的web.xml部署文件，Tomcat 默认实现部分配置入内：

- org.apache.catalina.servlets.DefaultServlet 
- org.apache.jasper.servlet.JspServlet

### lib

Tomcat 存放公用类库 ecj-*.jar：Eclipse Java 编译器 jasper.jar：JSP 编译器

### logs

localhost.${date}.log：当 Tomcat 应用起不来的时候，多看该文件，比如：类冲突

- NoClassDefFoundError 
- ClassNotFoundExecption

catalina.${date}.log：控制台输出，System.out外置

### webapps

简化 web 应用部署的方式

## 部署 Web 应用

### 方法一：放置在webapps录

### 方法二：修改conf/server.xml

添加Context元素：

```java
<!-- webapps下具有多个目录，只有ROOT被覆盖掉 -->
<!-- 访问路径：http://localhost:8080/ -->
<Context docBase="${webAppAbsolutePath}" path="/" reloadable="true"/>
<!-- 访问路径：http://localhost:8080/tomcat/ -->
<Context docBase="${webAppAbsolutePath}" path="/tomcat" reloadable="true"/>
```

熟悉配置元素可以参考org.apache.catalina.core.StandardContextsetter方法

该方式不支持动态部署，建议考虑在生产环境使用。

### 方法三：独立contextxml 配置文件

目录conf\Catalina\localhost 添加独立 Context XML 配置文件：${TOMCAT_HOME}\conf\Catalina\localhost + ${ContextPath}.xml abc.xml：

```java
<?xml version='1.0' encoding='utf-8'?>
<!-- 访问路径：http://localhost:8080/abc/ -->
<Context docBase="${webAppAbsolutePath}"  reloadable="true">
</Context>
```

### 方法四：/META-INF/context.xml

### I/O 连接器

参考文件：https://tomcat.apache.org/tomcat-7.0-doc/config/http.html 实现类：org.apache.catalina.connector.Connector

```java
public void setProtocol(String protocol) {

        if (AprLifecycleListener.isAprAvailable()) {
            if ("HTTP/1.1".equals(protocol)) {
                setProtocolHandlerClassName
                ("org.apache.coyote.http11.Http11AprProtocol");
            } else if ("AJP/1.3".equals(protocol)) {
                setProtocolHandlerClassName
                ("org.apache.coyote.ajp.AjpAprProtocol");
            } else if (protocol != null) {
                setProtocolHandlerClassName(protocol);
            } else {
                setProtocolHandlerClassName
                ("org.apache.coyote.http11.Http11AprProtocol");
            }
        } else {
            if ("HTTP/1.1".equals(protocol)) {
                setProtocolHandlerClassName
                ("org.apache.coyote.http11.Http11Protocol");
            } else if ("AJP/1.3".equals(protocol)) {
                setProtocolHandlerClassName
                ("org.apache.coyote.ajp.AjpProtocol");
            } else if (protocol != null) {
                setProtocolHandlerClassName(protocol);
            }
        }

    }
```


![img](https://img-blog.csdnimg.cn/20200325155527105.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) 默认实现是bio，tomcat8之后才是nio

### 小结

1.该方式可以实现热部署，热加载，因此建议在开发环境使用。 2.独立 Contex XML 配置文件时，设置path是无效的。 3.根独立 Context XML 配置文件路径： ${TOMCAT_HOME}/conf/${Engine.name}/${HOST.name}/ROOT.xml 4.实现热部署 调整<Context>元素中的属性reloadable="true" 5.连接器里面的连接池 注意conf/server.xml文件中得的一段注释：

```java
<Connector executor="tomcatThreadPool"
               port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
```

```java
public interface Executor extends java.util.concurrent.Executor, Lifecycle {

    public String getName();
    
    void execute(Runnable command, long timeout, TimeUnit unit);
}
```

标准实现：org.apache.catalina.core.StandardThreadExecutor，将连接处理交付给 Java 标准线程池：org.apache.tomcat.util.threads.ThreadPoolExecutor

6.JNDI

```java
<Context ...>
  ...
  <Resource name="mail/Session" auth="Container"
            type="javax.mail.Session"
            mail.smtp.host="localhost"/>
  ...
</Context>
```

```java
Context initCtx = new InitialContext();
Context envCtx = (Context) initCtx.lookup("java:comp/env");
Session session = (Session) envCtx.lookup("mail/Session");

Message message = new MimeMessage(session);
message.setFrom(new InternetAddress(request.getParameter("from")));
InternetAddress to[] = new InternetAddress[1];
to[0] = new InternetAddress(request.getParameter("to"));
message.setRecipients(Message.RecipientType.TO, to);
message.setSubject(request.getParameter("subject"));
message.setContent(request.getParameter("content"), "text/plain");
Transport.send(message);
```


## Web 技术栈

### Servlet 技术栈

### Web Flux（Netty）

## Web 自动装配

### API角度分析

Servlet 3.0 + API 实现 ServletContainerInitializer

### 容器角度分析

传统的Web应用，将webapp部署到Servlet容器中 嵌入式Web应用，灵活部署，任意指定位置（或通过复杂的条件判断） Tomcat 7 是 Servlet 3.0 的实现，ServletContainerInitializer Tomcat 8 是 Servlet 3.1 的实现，NIO HttpServletRequest、HttpServletResponse

>NIO 并非一定能够提高性能，比如请求数据量较大，NIO 性能比 BIO还要差<br> NIO 多工，读、写，同步的

### jar 启动

maven配置：

```java
<build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>3.1.0</version>
                <configuration>
                    <failOnMissingWebXml>false</failOnMissingWebXml>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <version>2.1</version>
                <executions>
                    <execution>
                        <id>tomcat-run</id>
                        <goals>
                            <goal>
                                exec-war-only
                            </goal>
                        </goals>
                        <phase>package</phase>
                        <configuration>
                            <path>/</path>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

打包： mvn -Dmaven.test.skip -U clean package

java -jar或者 jar 读取.jar META-INF/MANIFEST.MF，其中属性Main-Class就是引导类所在。

>参考 JDK API ： <code>java.util.jar.Manifest</code>

## Tomcat Maven 插件

### Tomcat 7 Maven 插件

```java
<dependency>
    <groupId>org.apache.tomcat.maven</groupId>
    <artifactId>tomcat7-maven-plugin</artifactId>
    <version>2.1</version>
</dependency>
```

```java
Manifest-Version: 1.0
Main-Class: org.apache.tomcat.maven.runner.Tomcat7RunnerCli
```

得出Tomcat 7 可执行 jar 引导类是：org.apache.tomcat.maven.runner.Tomcat7RunnerCli

## Tomcat 7 API 编程

### 确定 Classpath 目录

class 目录绝对路径：E:\JAVA\tomcat\target

```java
// System.getProperty("user.dir")，用户目录，即当前执行 Java 命令的目录
        String classPath = System.getProperty("user.dir")
                + File.separator + "target" + File.separator + "classes";
```

### 创建 Tomcat 实例

org.apache.catalina.startup.Tomcat Maven 坐标：org.apache.tomcat.embed:tomcat-embed-core:7.0.37

### 设置 Host 对象

```java
// 设置 Host
Host host = tomcat.getHost();
host.setName("localhost");
host.setAppBase("webapps");
```

### 设置 Classpath

Classpath 读取资源：配置、类文件 conf/web.xml作为配置文件，并且放置 Classpath 目录下（绝对路径）

设置 DemoServlet

```java
// 设置 Classpath 到 Context
// 添加 DemoServlet 到 Tomcat 容器
Wrapper wrapper = tomcat.addServlet(contextPath, "DemoServlet", new DemoServlet());
wrapper.addMapping("/demo");
```

### 避免 main 方法结束

```java
// 启动 Tomcat 服务器
        tomcat.start();

        // 强制 Tomcat Server 等待，避免 main 线程执行结束关闭
        /*synchronized (tomcat) {
            tomcat.wait();
        }*/
        tomcat.getServer().await();
```

tomcat.getServer().await() 和 tomcat.wait()的区别？ tomcat.getServer().await() 利用 Thread.sleep(long)实现：

```java
if (this.port == -1) {
    try {
        this.awaitThread = Thread.currentThread();
        while(!this.stopAwait) {
            try {
                Thread.sleep(10000L);
            } catch (InterruptedException var57) {
                ;
            }
        }
    } finally {
        this.awaitThread = null;
    }
}
```

tomcat.wait() 实际上是 Object#wait()方法，是 Native 方法

## Spring Boot 2.0 嵌入式 Tomcat 编程

### 实现 WebServerFactoryCustomizer

```java
@Configuration
public class TomcatConfiguration implements WebServerFactoryCustomizer {
    @Override
    public void customize(WebServerFactory factory) {
        if (factory instanceof TomcatServletWebServerFactory) {
            TomcatServletWebServerFactory tomcatFactory = (TomcatServletWebServerFactory) factory;

            // 设置 Connector
            Connector connector = new Connector();
            connector.setPort(9090);
            connector.setURIEncoding("UTF-8");
            tomcatFactory.addAdditionalTomcatConnectors(connector);
        }
    }
}
```

### 自定义 Context

实现 TomcatContextCustomizer

```java
tomcatFactory.addContextCustomizers((context) -> {
    // 相当于 new TomcatContextCustomizer
    if (context instanceof StandardContext) {
        StandardContext standardContext = (StandardContext) context;
        // standardContext.setDefaultWebXml();
    }
});
```

### 自定义 Connector

实现 TomcatConnectorCustomizer

```java
tomcatFactory.addConnectorCustomizers((connector) -> {
    // 相当于 new TomcatConnectorCustomizer
    connector.setPort(12345);
});
```


## 配置优化

### 减少配置优化

- 场景一：假设当前为 REST 应用 （微服务） 分析：它不需要静态资源，Tomcat 容器静态和动态

1.静态：DefaultServlet 通过移除conf/web.xml中 org.apache.jasper.servlet.DefaultServlet 2.动态：JspServlet 优化方案：通过移除conf/web.xml中 org.apache.jasper.servlet.JspServlet

><code>DispatcherServlet</code>：Spring Web MVC 应用 Servlet<br> <code>JspServlet</code>：编译并执行Jsp页面<br> <code>DefaultServlet</code>：Tomcat 处理静态资源的Servlet

3.移除 welcome-file-list

```java
<welcome-file-list>
    <welcome-file>index.html</welcome-file>
    <welcome-file>index.htm</welcome-file>
    <welcome-file>index.jsp</welcome-file>
</welcome-file-list>
```

4.如果程序是REST JSON Content-Type 或者 MIME Type：applcation/json，可考虑移除 mime-mapping 5.移除 session-config 设置 对于微服务/REST应用，不需要Session，因为不需要状态 Spring Security OAuth 2.0、JWT Session 通过 jsessionId 进行用户跟踪，HTTP无状态，需要一个 ID 与当前用户会话联系。Spring Session HttpSession jsessionId 作为 Redis，实现多个机器登录，用户会话不丢失 6.移除 conf/server.xml Valve Valve 类似于 Filter 移除 AccessLogValve，可以通过Nginx的Access Log 替代，Valve 实现都需要消耗Java应用的计算时间

- 场景二：需要JSP的情况 分析：JspServlet 无法去除，了解 JspServlet 处理原理

>Servlet周期：<br> 实例化：Servlet 和 Filter 实现必须包含默认构造器。反射的方式进行实例化。<br> 初始化：Servlet 容器调用 Servlet 或 Filter init() 方法<br> 销毁：Servlet 容器关闭时，Servlet 或 Filter distory() 方法被调用

Servlet 或 Filter 在一个容器中，是一般情况在一个 Web App 中是一个单例，不排除定义多个。 JspServlet 相关的优化 ServletConfig 参数：

- 需要编译 
 <ul> 
  - complier 
  - modificationTestInterval 
 </ul>  
- 不需要编译 
 <ul> 
  - development 设置 false development = false，那么这JSP要如何编译？优化的方法： Ant Task 执行 JSP 编译 Maven 插件：org.codehaus.mojo:jspc-maven-plugin 
 </ul> 

```java
<dependency>
    <groupId>org.apache.sling</groupId>
    <artifactId>jspc-maven-plugin</artifactId>
    <version>2.1.0</version>
</dependency>
```

JSP -&gt; 翻译 .jsp 或者 .jspx 文件成 .java -&gt; .class

总结，conf/web.xml 作为 Servlet 应用的默认 web.xml 实际上，应用程序存在两份 web.xml，其中包括应用的 web.xml ，最用将两者合并。

JspServlet 如果development参数为true，他会自行检查文件是否修改，如果修改重新翻译，再编译（加载和执行）。言外之意，JspServlet 开发模式可能会导致内存溢出 OOM。卸载 Class 不及时所导致 Perm 区域不够。

>ClassLoader -&gt; 1.class 2.class 3.class<br> ChildClassLoader -&gt; 4.class 5.class<br> ChildClassLoader load 1 - 5.class<br> 1.class 需要卸载，需要将 ParentClassLoader 设置为 null，当 ClassLoader 被 GC 后，1-3.class 都会被卸载<br> 1.class 它是文件，文件被 JVM 加载，二进制 -&gt; Verify -&gt; 解析

## 配置调整

### 关闭自动重载

```java
<Context docBase="E:/JAVA/tomcat/target/tomcat-0.0.1-SNAPSHOT" path="/" reloadable="false"/>
```

### 修改连接线程池数量

#### 通过 server.xml

```java
<Executor name="tomcatThreadPool" namePrefix="catalina-exec-"
        maxThreads="150" minSpareThreads="4"/>
<Connector executor="tomcatThreadPool"
               port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
```

通过程序理解，Executor 实际的 Tomcat 接口：

- org.apache.catalina.Executor 
 <ul> 
  - 扩展：J.U.C 的标准接口 java.util.concurrent.Executor 
  - 实现：org.apache.catalina.core.StandardThreadExecutor 
  - 线程池：org.apache.tomcat.util.threads.ThreadPoolExecutor (java.util.concurrent.ThreadPoolExecutor) 
  - 总结：Tomcat IO 连接器使用的线程池实际标准的 Java 线程池的扩展，最大线程数量和最小线程数量实际上分别是 MaximumPoolSize 和 CorePoolSize。 
 </ul> 

线程数量范畴：

```java
protected int maxThreads = 200;
protected int minSpareThreads = 25;

public void setMaxThreads(int maxThreads) {
    this.maxThreads = maxThreads;
    if (this.executor != null) {
        this.executor.setMaximumPoolSize(maxThreads);
    }
}

public void setMinSpareThreads(int minSpareThreads) {
    this.minSpareThreads = minSpareThreads;
    if (this.executor != null) {
        this.executor.setCorePoolSize(minSpareThreads);
    }
}
```

#### 通过 JMX 调整

观察 StandardThreadExecutor 是否存在调整线程池数量的 API

问题：到底设置多少的线程数量才是最优？ 首先，评估整体的请求量，假设 100W QPS，有机器数量 100 台，每台支撑 1W QPS。 第二，进行压力测试，需要一些测试样本，JMeter来实现，1000 线程发送请求，假设一次情况需要消耗 10ms，1秒可以同时完成100个请求。10000 / 100 = 100 线程。 确保，Load 太高。减少 Full GC，GC 取决于 JVM 堆的大小。执行一次操作需要5 MB 内存，50GB。假如只有16 GB 内存，必然执行 GC。要不调优程序，最好对象存储外化，比如 Redis，同时又需要评估 Redis 网络开销，又要评估网卡的接受能力。 第三，常规性压测，由于业务变更，会导致底层性能变化。

## JVM 优化

### 调整 GC 算法

如果 Java 版本小于9，默认 PS MarkSweep，可选设置 CMS、G1 如果 Java 9 的话，默认 G1

#### 默认算法

```java
java -jar -server -XX:-PrintGCDetails -X1oggc:./1g/gc.log -XX:+HeapDumpOnOutOfMemoryError -
Xms1g -Xmx1g -XX:MaxGCPauseMillis=250 -Djava.awt.headless-true stress-test-demo-0.0.-SNAPSHOT.jar
```

#### G1 算法

```java
java -jar -server -XX:-PrintGCDetails -Xloggc:./1g/g1-gc.1og - 
XX:+HeapDumpOnOutOfMemoryError -Xms1g -Xmx1g -XX: +UseG1GC -XX:+UseNUMA -
XX:MaxGCPauseMillis=250 -Djava.awt.headless-true stress-test-demo-0.0.1-SNAPSHOT.jar
```

64位系统，默认为-server -server主要提高吞吐量，在有限的资源，实现最大化利用。-client主要提高响应时间，主要是提高用户体验。

## Spring Boot 配置调整

```java
# <Executor name="tomcatThreadPool" namePrefix="catalina-exec-" maxThreads="99" minSpareThreads="9"/>
# 线程池大小
server.tomcat.max-threads=99
server.tomcat.min-spare-threads=9

# 取消 Tomcat AccessLogValve
server.tomcat.accesslog.enabled=false

# 取消 JspServlet
# Spring Boot 需要引入jar包才会加载jspServlet
#<dependency>
#   <groupId>org.apache.tomcat.embed</groupId>
#   <artifactId>tomcat-embed-jasper</artifactId>
#</dependency>
server.servlet.jsp.registered=false
```

