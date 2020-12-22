## Dubbo能解决什么问题


1. 怎么去维护url 通过注册中心去维护url（zookeeper、redis、memcache…） 
<li>F5硬件负载均衡器的单点压力比较大 软负载均衡</li> 
<li>怎么去整理出服务之间的依赖关系 自动去整理各个服务之间的依赖</li> 
<li>如果服务器的调用量越来越大，服务器的容量问题怎么去评估，扩容的指标 需要一个监控平台，可以监控调用量、响应时间</li>


Dubbo 是一个分布式的服务框架，提供高性能的以及透明化的 RPC 远程服务调用解决方法，以及 SOA 服务治理方案。 Dubbo的核心部分： 远程通信 集群容错 服务的自动发现 负载均衡



核心角色： Provider Consumer Registry Monitor Container ![img](https://img-blog.csdnimg.cn/20200414143729221.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)


## 简单实践

1.创建 dubbo-order 项目，同时创建 order-api 和 order-provider 模块

```java
<!-- pom.xml(dubbo-order) -->
<dependency>
  <groupId>com.spring.dubbo.order</groupId>
  <artifactId>order-api</artifactId>
  <version>1.0-SNAPSHOT</version>
</dependency>
<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>dubbo</artifactId>
  <version>2.6.2</version>
</dependency>
<dependency>
  <groupId>com.github.sgroschupf</groupId>
  <artifactId>zkclient</artifactId>
  <version>0.1</version>
</dependency>
<dependency>
  <groupId>org.apache.curator</groupId>
  <artifactId>curator-framework</artifactId>
  <version>4.0.1</version>
</dependency>
<dependency>
  <groupId>org.apache.zookeeper</groupId>
  <artifactId>zookeeper</artifactId>
  <version>3.4.13</version>
</dependency>
<!-- pom.xml(order-provider) -->
<dependency>
    <groupId>com.spring.dubbo.order</groupId>
    <artifactId>order-api</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>dubbo</artifactId>
</dependency>
<dependency>
    <groupId>com.github.sgroschupf</groupId>
    <artifactId>zkclient</artifactId>
</dependency>
<dependency>
  <groupId>org.apache.curator</groupId>
  <artifactId>curator-framework</artifactId>
</dependency>
<dependency>
  <groupId>org.apache.zookeeper</groupId>
  <artifactId>zookeeper</artifactId>
</dependency>
```

2.order-api 模块

```java
public class DoOrderRequest implements Serializable {
    private static final long serialVersionUID = 4296088277401004361L;
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "DoOrderRequest{" +
                "name='" + name + '\'' +
                '}';
    }
}
public class DoOrderResponse implements Serializable {
    private static final long serialVersionUID = 2243774451440109149L;
    private Object data;

    private String code;

    private String memo;

    public Object getData() {
        return data;
    }

    public void setData(Object data) {
        this.data = data;
    }

    public String getCode() {
        return code;
    }

    public void setCode(String code) {
        this.code = code;
    }

    public String getMemo() {
        return memo;
    }

    public void setMemo(String memo) {
        this.memo = memo;
    }

    @Override
    public String toString() {
        return "DoOrderResponse{" +
                "data=" + data +
                ", code='" + code + '\'' +
                ", memo='" + memo + '\'' +
                '}';
    }
}
public interface IOrderService {

    /**
     * 下单操作
     * @param request
     * @return
     */
    DoOrderResponse doOrder(DoOrderRequest request);

}
```

3.order-provider 模块下，resources 下添加日志配置文件

```java
log4j.rootLogger=info, stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n
```

4.如果需要搭配 zookeeper，resources 下添加 /META-INF/spring/order-provider.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.0.xsd
       http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd" default-autowire="byName">

    <!-- 当前项目在整个分布式架构的唯一名称，计算依赖关系的标签 -->
    <dubbo:application name="order-provider" owner="spring"/>

    <!-- dubbo这个服务所要暴露的服务地址所对应的注册中心 -->
    <!-- 不使用注册中心<dubbo:registry address="N/A"/>-->
    <!-- 使用 zookeeper 来做为注册中心 -->
    <!--<dubbo:registry address="zookeeper://192.168.174.128:2181?backup=192.168.174.129:2181,192.168.174.130:2181"/>-->
    <!-- 写在前面的不一定是leader，等同于下面 -->
    <!--<dubbo:registry protocol="zookeeper" address="192.168.174.128:2181,192.168.174.129:2181,192.168.174.130:2181"/>-->
    <dubbo:registry address="zookeeper://192.168.174.128:2181"/>

    <!-- 当前服务发布所依赖的协议：webservice,Thrift,Hessian,http -->
    <dubbo:protocol name="dubbo" port="20880"/>

    <!-- 服务发布的配置，需要暴露的服务接口 -->
    <dubbo:service interface="com.spring.dubbo.order.IOrderService" ref="orderService"/>

    <!-- Bean 的定义 -->
    <bean id="orderService" class="com.spring.dubbo.order.OrderServiceImpl"/>
</beans>
```

5.order-provider 模块的实现

```java
public class OrderServiceImpl implements IOrderService {
    @Override
    public DoOrderResponse doOrder(DoOrderRequest request) {
        System.out.println("曾经来过：" + request);
        DoOrderResponse response = new DoOrderResponse();
        response.setCode("1000");
        response.setMemo("处理成功");
        return response;
    }
}
public class App {
    public static void main(String[] args) {
        Main.main(args);
    }
}
```

6.将 order-api 模块打包，已被 dubbo-user 使用 7.创建 dubbo-user 项目，同时引入 order-api-1.0-SNAPSHOT.jar

```java
<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>dubbo</artifactId>
  <version>2.5.3</version>
</dependency>
<dependency>
  <groupId>com.github.sgroschupf</groupId>
  <artifactId>zkclient</artifactId>
  <version>0.1</version>
</dependency>
```

```java
public class App {
    public static void main( String[] args ) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("order-consumer.xml");

        // 用户下单过程
        IOrderService service = (IOrderService) context.getBean("orderService");

        DoOrderRequest request = new DoOrderRequest();
        request.setName("spring");
        DoOrderResponse response = service.doOrder(request);
        System.out.println(response);
    }
}
```

8.在 dubbo-user 项目 resources 中添加文件 order-consumer.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <!-- 当前项目在整个分布式架构的唯一名称，计算依赖关系的标签 -->
    <dubbo:application name="order-provider" owner="spring"/>

    <!-- 使用 zookeeper，dubbo这个服务所要暴露的服务地址所对应的注册中心 -->
    <dubbo:registry address="zookeeper://192.168.174.128:2181"/>

    <!-- 生成一个远程服务的调用代理 -->
    <dubbo:reference id="orderService" interface="com.spring.dubbo.order.IOrderService"/>
    <!-- 不使用 zookeeper 时，dubbo:reference 标签需要添加 url="dubbo://192.168.174.1:20880/com.spring.dubbo.order.IOrderService" -->
</beans>
```

### 服务地址分析

启动 order-provider 时，控制台会打印服务所要暴露的服务地址： dubbo://192.168.174.1:20880/com.spring.dubbo.order.IOrderService?anyhost=true&amp;application=order-provider&amp;dubbo=2.5.3&amp;interface=com.spring.dubbo.order.IOrderService&amp;methods=doOrder&amp;owner=spring&amp;pid=15320&amp;side=provider&amp;timestamp=1586852986986, dubbo version: 2.5.3, current host: 127.0.0.1


如果使用 zookeeper 时，在 zookeeper 中可以获取如下的信息： ![img](https://img-blog.csdnimg.cn/20200414212643485.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) dubbo://192.168.174.1/com.spring.dubbo.order.IOrderService%3Fanyhost%3Dtrue%26application%3Dorder-provider%26dubbo%3D2.5.3%26interface%3Dcom.spring.dubbo.order.IOrderService%26methods%3DdoOrder%26owner%3Dspring%26pid%3D4200%26side%3Dprovider%26timestamp%3D1586856057540

## 新版 dubbo-admin 控制台的安装

控制中心是用来做服务治理的，比如控制服务的权重、服务的路由。 1.下载dubbo的源码( [源码 github 地址](https://github.com/apache/dubbo-samples.git)， [新版 dubbo-admin github 地址](https://github.com/apache/dubbo-admin.git)) 2.新版 dubbo-admin 分为 dubbo-admin-server 和 dubbo-admin-ui 两个模块 3.dubbo-admin-server 为一个 spring boot 项目，可以在 resources/application.properties中指定注册中心地址修改(同时可以修改登录的账号密码)，可通过正常的 spring boot 项目方式启动 4.dubbo admin ui 由 npm 管理和构建，在开发环境中，可以运行:

```java
npm install
npm run dev
```


5.页面访问 访问 http://localhost:8081 ![img](https://img-blog.csdnimg.cn/20200414222519606.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

## simple 监控中心

监控服务的调用次数、调用关系、响应事件， [github 地址](https://github.com/apache/dubbo-admin/tree/master) Monitor 也是一个dubbo服务，所以也会有端口和url。 1.修改/conf/dubbo.properties：

```java
dubbo.registry.address=zookeeper://127.0.0.1:2181
```

2.修改服务端配置，即 order-provider 的 order-provider.xml：

```java
<dubbo:monitor protocol="registry"/>
```


![img](https://img-blog.csdnimg.cn/20200414230411215.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

## telnet

telnet ip port ls、cd、pwd、clear、invoker

## 启动服务检查

dubbo:reference 属性：check ，默认值是true 如果提供方没有启动的时候，默认回去检查所有以来的服务是否正常提供服务。如果 check 为 false，表示启动的时候不去检查。当服务出现循环依赖的时候，check 设置成 false。 dubbo:consumer check=false，关闭所有服务的启动时检查：（没有提供者时报错） dubbo:registry check=false，关闭注册中心启动时检查：（注册订阅失败时报错）

## 多协议支持

dubbo 支持的协议：dubbo，RMI，hessian，webservice，http，Thrift

### hessian 协议演示

1.引入 jar 包

```java
<dependency>
  <groupId>com.caucho</groupId>
  <artifactId>hessian</artifactId>
  <version>4.0.38</version>
</dependency>
 <dependency>
     <groupId>javax.servlet</groupId>
     <artifactId>servlet-api</artifactId>
     <version>2.5</version>
 </dependency>
<dependency>
  <groupId>org.mortbay.jetty</groupId>
  <artifactId>jetty</artifactId>
  <version>6.1.26</version>
</dependency>
```

2.修改 order-provider.xml 文件

```java
<!-- 增加 hessian 协议 -->
<dubbo:protocol name="hessian" port="8090" server="jetty"/>
<!-- 指定 service 服务的协议版本号 -->
<dubbo:service interface="com.spring.dubbo.order.IOrderService" ref="orderService" protocol="hessian"/>
```

3.消费端改造

```java
<!-- 指定引用服务所需要的协议版本 -->
<dubbo:reference id="orderService" interface="com.spring.dubbo.order.IOrderService" protocol="hessian"/>
```

服务端在 zookeeper 中注册的服务地址： hessian://192.168.174.1:8090/com.spring.dubbo.order.IOrderService

## 多注册中心支持

```java
<!-- 多注册中心 -->
<dubbo:registry id="zkOne" address="zookeeper://192.168.174.128:2181"/>
<dubbo:registry id="zkTwo" address="zookeeper://192.168.174.129:2181"/>
<!-- 针对服务指定注册中心 -->
<dubbo:service interface="com.spring.dubbo.order.IOrderService" ref="orderService" registry="zkOne" />
```

## 多版本支持

服务端：

```java
<dubbo:service interface="com.spring.dubbo.order.IOrderService" ref="orderService" version="1.0.0"/>
    <dubbo:service interface="com.spring.dubbo.order.IOrderService" ref="orderService2" version="2.0.0"/>
```

消费端：

```java
<dubbo:reference id="orderService" interface="com.spring.dubbo.order.IOrderService" version="2.0.0"/>
```

dubbo://192.168.174.1:20880/com.spring.dubbo.order.IOrderService?..&amp;version=1.0.0 dubbo://192.168.174.1:20880/com.spring.dubbo.order.IOrderService?..&amp;version=2.0.0

## 异步调用

消费端，async="true" 表示接口异步返回：

```java
<dubbo:reference id="orderService" interface="com.spring.dubbo.order.IOrderService" async="true"/>
<!-- 可单指定某个方法 -->
<!-- <dubbo:method name="doOrder" async="true"/> -->
```

```java
public class App {
    public static void main( String[] args ) throws ExecutionException, InterruptedException {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("order-consumer.xml");

        // 用户下单过程
        IOrderService service = (IOrderService) context.getBean("orderService");

        DoOrderRequest request = new DoOrderRequest();
        request.setName("spring");
        service.doOrder(request);

        Future<DoOrderResponse> response = RpcContext.getContext().getFuture();
        DoOrderResponse r = response.get();
        System.out.println(r);
    }
}
```


![img](https://img-blog.csdnimg.cn/20200415111514366.png)

## 主机绑定

provider://192.168.174.1:20880 org.apache.dubbo.config.ServiceConfig#findConfigedHosts

```java
String hostToBind = getValueFromConfig(protocolConfig, DUBBO_IP_TO_BIND);
if (hostToBind != null && hostToBind.length() > 0 && isInvalidLocalHost(hostToBind)) {
    throw new IllegalArgumentException("Specified invalid bind ip from property:" + DUBBO_IP_TO_BIND + ", value:" + hostToBind);
}
if (StringUtils.isEmpty(hostToBind)) {
    hostToBind = protocolConfig.getHost();
    if (provider != null && StringUtils.isEmpty(hostToBind)) {
        hostToBind = provider.getHost();
    }
    if (isInvalidLocalHost(hostToBind)) {
        anyhost = true;
        try {
            logger.info("No valid ip found from environment, try to find valid host from DNS.");
            hostToBind = InetAddress.getLocalHost().getHostAddress();
        } catch (UnknownHostException e) {
            logger.warn(e.getMessage(), e);
        }
        if (isInvalidLocalHost(hostToBind)) {
            if (CollectionUtils.isNotEmpty(registryURLs)) {
                for (URL registryURL : registryURLs) {
                    if (MULTICAST.equalsIgnoreCase(registryURL.getParameter("registry"))) {
                        // skip multicast registry since we cannot connect to it via Socket
                        continue;
                    }
                    try (Socket socket = new Socket()) {
                        SocketAddress addr = new InetSocketAddress(registryURL.getHost(), registryURL.getPort());
                        socket.connect(addr, 1000);
                        hostToBind = socket.getLocalAddress().getHostAddress();
                        break;
                    } catch (Exception e) {
                        logger.warn(e.getMessage(), e);
                    }
                }
            }
            if (isInvalidLocalHost(hostToBind)) {
                hostToBind = getLocalHost();
            }
        }
    }
}
```

1. 通过dubbo:protocol host配置的地址去找 
2. hostToBind = InetAddress.getLocalHost().getHostAddress(); 
3. 通过socket发起连接连接到注册中心的地址。再获取连接过去以后本地的ip地址 
4. hostToBind = getLocalHost()

## dubbo 服务只订阅

```java
<dubbo:registry address="zookeeper://192.168.174.128:2181" register="false"/>
```

## dubbo 服务只注册

只提供服务

```java
<dubbo:registry address="zookeeper://192.168.174.128:2181" subscribe="false"/>
```


1.复制一份 dubbo-order 项目，修改端口号

```java
<dubbo:protocol name="dubbo" port="20881"/>
```

2.同时启动 两个 dubbo-order 3.消费端调用

```java
public class App {
    public static void main( String[] args ) throws InterruptedException {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("order-consumer.xml");

        // 用户下单过程
        IOrderService service = (IOrderService) context.getBean("orderService");

        DoOrderRequest request = new DoOrderRequest();
        request.setName("spring");
        for (int i = 0; i < 10; i++){
            DoOrderResponse r = service.doOrder(request);
            System.out.println(r);
            TimeUnit.SECONDS.sleep(1);
        }
    }
}
```

## 负载均衡算法

在集群负载均衡时，Dubbo 提供了多种均衡策略，缺省为 random 随机调用。可以自行扩展负载均衡策略。 Random LoadBalance（默认）随机，按权重设置随机概率。 在一个截面上碰撞的概率高，但调用量越大分布越均匀，而且按概率使用权重后也比较均匀，有利于动态调整提供者权重。

```java
<dubbo:service interface="com.spring.dubbo.order.IOrderService" ref="orderService" loadbalance="random"/>
```

roundRobin LoadBalance 轮循，按公约后的权重设置轮循比率。存在慢的提供者累积请求的问题，比如：第二台机器很慢，但没挂，当请求调到第二台时就卡在那，久而久之，所有请求都卡在调到第二台上。 LeastActive LoadBalance 最少活跃调用数，相同活跃数的随机，活跃数指调用前后计数差。使慢的提供者收到更少请求，因为越慢的提供者的调用前后计数差会越大。 Consistent LoadBaleance 一致性Hash，相同参数的请求总是发到同一提供者。当某一台提供者挂时，原本发往该提供者的请求，基于虚拟节点，平摊到其它提供者，不会引起剧烈变动。

## 连接超时 timeout

必须要设置服务的处理的超时时间

```java
<dubbo:service interface="com.spring.dubbo.order.IOrderService" ref="orderService" timeout="200"/>
```

## 集群容错

Failover cluster（默认），失败的时候自动切换并重试其他服务器。 通过retries=2，来设置重试次数 failfast cluster 快速失败，只发起一次调用 ; 写操作。比如新增记录的时候， 非幂等请求 failsafe cluster 失败安全。 出现异常时，直接忽略异常 failback cluster 失败自动恢复。 后台记录失败请求，定时重发 forking cluster 并行调用多个服务器，只要一个成功就返回。 只能应用在读请求 broadcast cluster 广播调用所有提供者，逐个调用。其中一台报错就会返回异常

```java
<dubbo:service interface="com.spring.dubbo.order.IOrderService" cluster="failfast" ref="orderService"/>
```

## 配置的优先级


消费端有限最高，其次服务端 ![img](https://img-blog.csdnimg.cn/20200415145902901.png)


1.dubbo-order 项目和其 order-provider 模块引入 spring 相关 jar 包 dubbo 中已经省缺配置了org.springframework:spring-context:jar:4.3.10.RELEASE

```java
<dependency>
  <groupId>com.spring.dubbo.order</groupId>
  <artifactId>order-api</artifactId>
  <version>1.0-SNAPSHOT</version>
</dependency>
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-core</artifactId>
  <version>${org.springframework-version}</version>
</dependency>
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-beans</artifactId>
  <version>${org.springframework-version}</version>
</dependency>
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-context-support</artifactId>
  <version>${org.springframework-version}</version>
</dependency>
```

2.order-provider.xml

```java
<!-- 不需要在xml中引入bean，可以直接通过注解方式引入bean，@Service(value = "orderService") -->
<context:component-scan base-package="com.spring.dubbo.order"/>
```

3.order-api 添加 resources\META-INF\client\order-customer.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.0.xsd
       http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd" default-autowire="byName">
    <!-- 生成一个远程服务的调用代理 -->
    <dubbo:reference id="orderServiceSpring" interface="com.spring.dubbo.order.IOrderService" protocol="dubbo" version="2.0.0"/>

</beans>
```

4.创建一个简单的 springmvc 项目 pom.xml

```java
<properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <org.springframework-version>4.3.10.RELEASE</org.springframework-version>
    <commons-logging-version>1.1.1</commons-logging-version>
</properties>
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>${org.springframework-version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>${org.springframework-version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
        <version>${org.springframework-version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>${org.springframework-version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>${org.springframework-version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-tx</artifactId>
        <version>${org.springframework-version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-support</artifactId>
        <version>${org.springframework-version}</version>
    </dependency>
    <dependency>
        <groupId>commons-logging</groupId>
        <artifactId>commons-logging</artifactId>
        <version>${commons-logging-version}</version>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>dubbo</artifactId>
        <version>2.6.2</version>
    </dependency>
    <dependency>
        <groupId>com.spring.dubbo.order</groupId>
        <artifactId>order-api</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
    <dependency>
        <groupId>com.github.sgroschupf</groupId>
        <artifactId>zkclient</artifactId>
        <version>0.1</version>
    </dependency>
    <dependency>
        <groupId>org.apache.curator</groupId>
        <artifactId>curator-framework</artifactId>
        <version>4.0.1</version>
    </dependency>
    <dependency>
        <groupId>org.apache.zookeeper</groupId>
        <artifactId>zookeeper</artifactId>
        <version>3.4.13</version>
    </dependency>
</dependencies>
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>2.0.2</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
    </plugins>
</build>
```

```java
<!-- web.xml -->
<!DOCTYPE web-app PUBLIC
        "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
        "http://java.sun.com/dtd/web-app_2_3.dtd" >
<web-app>
    <display-name>dubbo-spring</display-name>
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring/application*.xml</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring/springmvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <!--配置全局方法的拦截-->
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
<!-- applictionContext.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context-3.0.xsd
       http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd" default-autowire="byName">

    <context:component-scan base-package="com.spring">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    <dubbo:application name="dubbo-spring"/>
    <dubbo:registry protocol="zookeeper" address="192.168.174.128:2181"/>

    <import resource="classpath*:META-INF/client/order-customer.xml"/>
</beans>
<!-- springmvc.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
       http://www.springframework.org/schema/context
	   http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 扫描 controller -->
    <context:component-scan base-package="com.spring.controller" use-default-filters="false">
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
        <context:include-filter type="annotation" expression="org.springframework.web.bind.annotation.ControllerAdvice"/>
    </context:component-scan>

    <mvc:annotation-driven/>
    <!-- 对静态资源文件的访问 -->
    <mvc:resources location="/system/" mapping="/system/**"/>

    <!-- 配置jsp视图 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```

```java
@Controller
@RequestMapping("/home")
public class HomeController {

    @Autowired
    private UserService userService;

    // 按名称注入，变量名需要与api.jar提供的id相同
    @Autowired
    IOrderService orderServiceSpring;

    @RequestMapping(value = "/index")
    public ModelAndView index(){
        DoOrderRequest request = new DoOrderRequest();
        request.setName("dubbo-spring");
        orderServiceSpring.doOrder(request);
        userService.insert();
        return new ModelAndView("index");
    }
}
```


## 分包

1. 服务接口，请求服务模型，异常信息都放在 api 里面，符合重用发布等价原则，共同重用原则 
2. api 里面放入 spring 的引用配置。也可以放在模块的包目录下。com.spring.dubbo.order/***-reference.xml

## 粒度

1. 尽可能把接口设置成粗粒度，每个服务方法代表一个独立的功能，而不是某个功能的步骤。否则就会涉及到分布式事务 
2. 服务接口建议以业务场景为单位划分。并对相近业务做抽象，防止接口暴增 
3. 不建议使用过于抽象的通用接口，接口没有明确的语义，带来后期的维护

## 版本

1. 每个接口都应该定义版本，为后续的兼容性提供前瞻性的考虑 
2. 建议使用两位版本号，因为第三位版本号表示的兼容性升级，只有不兼容时才需要变更服务版本 
3. 当接口做到不兼容升级的时候，先升级一半或者一台提供者为新版本，再将消费全部升级新版本，然后再将剩下的一半提供者升级新版本（预发布环境）

## 推荐配置

1. 在 provider 端尽可能配置 consumer 端的属性，比如timeout、retires、线程池大小、LoadBalance 
2. 配置管理员信息，application 上面配置的 owner ，owner建议配置2个人以上。因为owner都能够在监控中心看到 
3. 配置 dubbo 缓存文件

## 配置 dubbo 缓存文件

```java
<!-- dubbo-spring，消费端配置 -->
<dubbo:registry protocol="zookeeper" file="/Users/spring_zhang/Code/dubbo.cache" address="127.0.0.1:2181"/>
```

缓存：注册中心的列表，服务提供者列表

### dubbo.cache

```java
#Dubbo Registry Cache
#Thu Apr 16 21:01:06 CST 2020
com.spring.dubbo.order.IOrderService\:2.0.0=empty\://10.1.1.231/com.spring.dubbo.order.IOrderService?application\=dubbo-spring&category\=configurators&dubbo\=2.6.2&interface\=com.spring.dubbo.order.IOrderService&methods\=doOrder&pid\=5340&protocol\=dubbo&revision\=1.0-SNAPSHOT&side\=consumer&timestamp\=1587042064681&version\=2.0.0 empty\://10.1.1.231/com.spring.dubbo.order.IOrderService?application\=dubbo-spring&category\=routers&dubbo\=2.6.2&interface\=com.spring.dubbo.order.IOrderService&methods\=doOrder&pid\=5340&protocol\=dubbo&revision\=1.0-SNAPSHOT&side\=consumer&timestamp\=1587042064681&version\=2.0.0 dubbo\://10.1.1.231\:20880/com.spring.dubbo.order.IOrderService2?anyhost\=true&application\=order-provider&dubbo\=2.6.2&generic\=false&interface\=com.spring.dubbo.order.IOrderService&methods\=doOrder&owner\=spring&pid\=4201&revision\=2.0.0&side\=provider&timestamp\=1587040010193&version\=2.0.0
com.spring.dubbo.user.IUserService=empty\://10.1.1.231/com.spring.dubbo.user.IUserService?application\=dubbo-spring&category\=configurators&dubbo\=2.6.2&interface\=com.spring.dubbo.user.IUserService&methods\=toLogin,login&pid\=5340&revision\=1.0-SNAPSHOT&side\=consumer&timestamp\=1587042066209 empty\://10.1.1.231/com.spring.dubbo.user.IUserService?application\=dubbo-spring&category\=routers&dubbo\=2.6.2&interface\=com.spring.dubbo.user.IUserService&methods\=toLogin,login&pid\=5340&revision\=1.0-SNAPSHOT&side\=consumer&timestamp\=1587042066209 dubbo\://10.1.1.231\:20881/com.spring.dubbo.user.IUserService?anyhost\=true&application\=user-provider&dubbo\=2.6.2&generic\=false&interface\=com.spring.dubbo.user.IUserService&methods\=toLogin,login&owner\=spring&pid\=4142&side\=provider&timestamp\=1587039921924
```

