---
layout:  post
title:   SpringCloud 异步和消息
date:   2018-11-08 14:00:31
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-# 分布式

---



## 异步

客户端请求不会阻塞进程，服务端的响应可以是非即时的

## RabbitMQ的简单使用

1.添加依赖

```java
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
```

2.创建接收器

```java
@Slf4j
@Component
public class MqReceiver {

    //1.无法自动创建，需要在Rabbit控制面板自己创建队列@RabbitListener(queues = "myQueue")
    //2.自动创建队列@RabbitListener(queuesToDeclare = @Queue("myQueue"))
    //3.自动创建，Exchange和Queue绑定
    @RabbitListener(bindings = @QueueBinding(
            exchange = @Exchange("myExchange"),
            value = @Queue("myQueue")

    ))
    public void process(String message) {
        log.info("MqReceiver: {}", message);
    }

    @RabbitListener(bindings = @QueueBinding(
            exchange = @Exchange("myOrder"),
            key = "computer",
            value = @Queue("computerOrder")

    ))
    public void processComputer(String message) {
        log.info("computer MqReceiver: {}", message);
    }

    @RabbitListener(bindings = @QueueBinding(
            exchange = @Exchange("myOrder"),
            key = "fruit",
            value = @Queue("fruitOrder")
    ))
    public void processFruit(String message) {
        log.info("fruit MqReceiver: {}", message);
    }
}
```

3.测试

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class OrderApplicationTests {

	@Test
	public void contextLoads() {
	}

}
@Component
public class MqSenderTest extends OrderApplicationTests {

    @Autowired
    private AmqpTemplate amqpTemplate;

    @Test
    public void send() {
        amqpTemplate.convertAndSend("myQueue", "now " + new Date());
    }

    @Test
    public void sendOrder() {
        amqpTemplate.convertAndSend("myOrder", "computer", "now " + new Date());
    }
}
```

## Spring Cloud Stream


![img](https://img-blog.csdnimg.cn/2018110713473878.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

1.引入依赖

```java
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
        </dependency>
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

