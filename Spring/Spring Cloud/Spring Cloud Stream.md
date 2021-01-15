# 消息驱动的微服务 Spring Cloud Stream

Spring Cloud Stream 是一个用来为微服务应用构建消息驱动能力的框架。它可以基于 Spring Boot 来创建独立的、可用于生产的Spring应用程序。它通过使用 Spring Integration 来连接消息代理中间件以实现消息事件驱动。Spring Cloud Stream为一些供应商的消息中间件产品提供了个性化的自动化配置实现，并且引入了发布-订阅、消费组以及分区这三个核心概念。

简单地说，Spring Cloud Stream 本质上就是整合了Spring Boot和Spring Integration，实现了一套轻量级的消息驱动的微服务框架。通过使用Spring Cloud Stream，可以有效简化开发人员对消息中间件的使用复杂度，让系统开发人员可以有更多的精力关注于核心业务逻辑的处理。

由于Spring Cloud Stream基于Spring Boot实现，所以它秉承了Spring Boot的优点，自动化配置的功能可帮助我们快速上手使用。

## 1 快速入门

创建一个基础的Spring Boot工程，命名为stream-hello。

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
</dependency>
```

```java
// 创建用于接收来自RabbitMQ消息的消费者
@EnableBinding(Sink.class)
public class SinkReceiver {

    private Logger log = LoggerFactory.getLogger(SinkReceiver.class);

    @StreamListener(Sink.INPUT)
    public void receive(Object payload) {
        log.info("Received: " + payload);
    }

}
```

创建应用主类，并启动：

```markdown
o.s.a.r.c.CachingConnectionFactory       : Created new connection: rabbitConnectionFactory#2715644a:0/SimpleConnection@64524dd [delegate=amqp://guest@127.0.0.1:5672/, localPort= 50300]
o.s.c.stream.binder.BinderErrorChannel   : Channel 'application.input.anonymous.ImRJKn6-TV-NRrL5_sHNZQ.errors' has 1 subscriber(s).
o.s.i.a.i.AmqpInboundChannelAdapter      : started inbound.input.anonymous.ImRJKn6-TV-NRrL5_sHNZQ
```

使用 guest 用户创建了一个127.0.0.1:5672 RabbitMQ 连接。

![image-20210115185554918](Spring Cloud Stream.assets/image-20210115185554918.png)

声明了一个名为 input.anonymous.ImRJKn6-TV-NRrL5_sHNZQ 的队列，并通过 RabbitMessageChannelBinder 将自己绑定为消费者。

![image-20210115185759475](Spring Cloud Stream.assets/image-20210115185759475.png)

可以在 RabbitMQ 控制台发送一条消息。

![image-20210115185833371](Spring Cloud Stream.assets/image-20210115185833371.png)

```
INFO 13908 --- [emjYlBwftYKAw-1] com.spring.rabbit.SinkReceiver           : Received: [B@6e73a4d0
```

可以发现在应用控制台中输出的内容就是 sinkReceiver 中的 receive 方法定义的，而输出的具体内容则来自消息队列中获取的对象。这里由于我们没有对消息进行序列化，所以输出的只是该对象的引用。

## 2 原理解析

首先，我们对 Spring Boot 应用做的就是引入 spring-cloud-starter-stream-rabbit 依赖，该依赖包是Spring Cloud Stream 对 RabbitMQ 支持的封装，其中包含了对 RabbitMQ 的自动化配置等内容。从下面它定义的依赖关系中，我们还可以知道它等价于 spring-cloud-stream-binder-rabbit 依赖。

```xml
<dependency>
	<groupId>org.springframework.clouds/groupId>
	<artifactId>spring-cloud-stream-binder-rabbit</artifactId>
</dependency>
```

在sinkReceiver 中，@EnableBinding 注解用来指定一个或多个定义了 @Input 或 @Output 注解的接口，以此实现对消息通道（Channel）的绑定。在上面的例子中，我们通过 @EnableBinding(Sink.class) 绑定了 Sink 接口，该接口是 Spring Cloud Stream中默认实现的对输入消息通道绑定的定义，它的源码如下:

```java
public interface Sink {
    String INPUT = "input";

    @Input("input")
    SubscribableChannel input();
}
```

它通过 @Input 注解绑定了一个名为 input 的通道。除了 Sink 之外，Spring Cloud Stream还默认实现了绑定 output通道的 source 接口，还有结合了 Sink 和 Source 的 Processor 接口，实际使用时我们也可以自己通过 @Input 和 @Output 注解来定义绑定消息通道的接口。当需要为 @EnableBinding 指定多个接口来绑定消息通道的时候，可以这样定义:

```java
@EnableBinding(value = {Sink.class, source.class})
```

@StreamListener，它主要定义在方法上，作用是将被修饰的方法注册为消息中间件上数据流的事件监听器，注解中的属性值对应了监听的消息通道名。在上面的例子中，我们通过@StreamListener(sink.INPUT)注解将 receive 方法注册为 input 消息通道的监听处理器，所以当我们在 RabbitMQ 的控制页面中发布消息的时候，receive方法会做出对应的响应动作。

3 核心概念

下图是官方文档中 Spring Cloud Stream 应用模型的结构图。从中我们可以看到，Spring Cloud Stream 构建的应用程序与消息中间件之间是通过绑定器 Binder 相关联的，绑定器对于应用程序而言起到了隔离作用，它使得不同消息中间件的实现细节对应用程序来说是透明的。所以对于每一个 Spring Cloud Stream的应用程序来说，它不需要知晓消息中间件的通信细节，它只需知道 Binder 对应程序提供的抽象概念来使用消息中间件来实现业务逻辑即可，而这个抽象概念就是在快速入门中我们提到的消息通道:channel。如下图所示，在应用程序和 Binder之间定义了两条输入通道和三条输出通道来传递消息，而绑定器则是作为这些通道和消息中间件之间的桥梁进行通信。

![image-20210115185833371](Spring Cloud Stream.assets/2018110713473878.png)

3.1 绑定器

`Binder 绑定器`是Spring Cloud Stream中一个非常重要的概念。在没有绑定器这个概念的情况下，实现的消息交互逻辑就会非常笨重。

通过定义绑定器作为中间层，完美地实现了应用程序与消息中间件细节之间的隔离。通过向应用程序暴露统一的 Channel 通道，使得应用程序不需要再考虑各种不同的消息中间件的实现。当需要升级消息中间件，或是更换其他消息中间件产品时，我们要做的就是更换它们对应的 Binder 绑定器而不需要修改任何 Spring Boot 的应用逻辑。

示例中，并没有使用 application.properties 或是 application.ym l来做任何属性设置。那是因为它也秉承了SpringBoot的设计理念，提供了对RabbitMQ默认的自动化配置。当然,我们也可以通过Spring Boot应用支持的任何方式来修改这些配置。比如，下面就是通过配置文件来对 RabbitMQ 的连接信息以及 input 通道的主题进行配置的示例:

```properties
spring.cloud.stream.bindings.input.destination=raw-sensor-data
spring.rabbitmg.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=springcloud
spring.rabbitmq.password=123456
```











2.定义input和output接口

```java
public interface StreamClient {

    String INPUT = "input";

    String OUTPUT = "output";

    String INPUT2 = "input2";

    String OUTPUT2 = "output2";

    @Input(StreamClient.INPUT)
    SubscribableChannel input();

    @Output(StreamClient.OUTPUT)
    MessageChannel output();

    @Input(StreamClient.INPUT2)
    SubscribableChannel input2();

    @Output(StreamClient.OUTPUT2)
    MessageChannel output2();
}
```

3.消息接收类

```java
@Component
@EnableBinding(StreamClient.class)
@Slf4j
public class StreamReceiver {

    @StreamListener(StreamClient.INPUT)
    // 通知回调
    @SendTo(StreamClient.INPUT2)
    public String process(OrderDTO message) {
        log.info("StreamReceiver: {}", message);
        // 发送MQ消息
        return "received";
    }

    @StreamListener(StreamClient.INPUT2)
    public void process2(String message) {
        log.info("StreamReceiver2: {}", message);
    }
}
```

4.配置

```java
spring:
    stream:
      bindings:
        input:
          destination: myMessage
          # 通过json格式传递数据
          content-type: application/json
          # 消息分组，把一个服务划到一个组里，无论你起了多少个实例，只会有一个实例消费
          group: order
        output:
          destination: myMessage
          content-type: application/json
          group: order
        input2:
          destination: myMessage2
          content-type: application/json
          group: order
        output2:
          destination: myMessage2
          content-type: application/json
          group: order
```

5.调用

```java
@RestController
public class SendMessageController {

    @Autowired
    private StreamClient streamClient;

    @GetMapping("/sendMessage")
    public void process() {
        OrderDTO orderDTO = new OrderDTO();
        orderDTO.setOrderId("123456");
        streamClient.output().send(MessageBuilder.withPayload(orderDTO).build());
    }

}
```


![img](https://img-blog.csdnimg.cn/20181108135954859.png) StreamReceiver: OrderDTO(orderId=123456, buyerName=null, buyerPhone=null, buyerAddress=null, buyerOpenid=null, orderAmount=null, orderStatus=null, payStatus=null, orderDetailList=null) StreamReceiver2: received

