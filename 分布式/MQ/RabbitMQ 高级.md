## 1 RabbitMQ 进阶知识

### 1.1 TTL(Time to Live)

#### 1.1.1 消息的过期时间

**通过队列属性设置消息的过期时间**

所有队列中的消息超过时间未被消费时，都会过期。

```java
// 原生API设置方式
Map<String, Object> arguments = new HashMap<String, Object>();
// 1.绑定死信交换机
arguments.put("x-dead-letter-exchange", "DLX_EXCHANGE");
// 2.队列中的消息未被消费 11 秒后过期
arguments.put("x-message-ttl", 11000);
channel.exchangeDeclare("DLX_TEST_EXCHANGE", "topic");
channel.queueDeclare("DLX_TEST", true, false, false, arguments);
// 3.绑定交换机和队列，过期属性只有队列中可以设置
channel.queueBind("DLX_TEST", "DLX_TEST_EXCHANGE", "spring.test");
// 发送消息
channel.basicPublish("DLX_TEST_EXCHANGE", "spring.test", null, ("发送死信消息").getBytes());

// DLX_TEST_EXCHANGE 对应路由的消费者不存在，11 秒后 DLX_EXCHANGE 交换机接收到死信消息

// 4.声明死信交换机消费者
// 通道绑定交换机
channel.exchangeDeclare("DLX_EXCHANGE", "topic");
// 临时队列
String queueName = channel.queueDeclare().getQueue();
// 绑定队列和交换机，动态通配符形式route key
channel.queueBind(queueName, "DLX_EXCHANGE",  "#");
```

**设置单条消息的过期时间**

发送消息时指定消息属性。

```java
// 原生API
BasicProperties properties = new BasicProperties().builder().expiration("4000").build();
channel.basicPublish("DLX_TEST_EXCHANGE", "spring.test", properties, ("发送死信消息").getBytes());

// RabbitTemplate
MessageProperties messageProperties = new MessageProperties(); 
messageProperties.setExpiration("4000"); // 消息的过期属性，单位 ms 
Message message = new Message("这条消息 4 秒后过期".getBytes(), messageProperties); rabbitTemplate.send("", "spring.ttl", message);
```

> 如果同时指定了 Message TTL 和 Queue TTL，则小的那个时间生效。

### 1.2 死信(Dead Letter)队列

什么情况下消息会变成死信？

1. 消息被消费者拒绝并且设置不重回队列（NACK || Reject）&& requeue == false

2. 消息过期
3. 队列达到最大长度，超过了Max length(消息数)或者Max length bytes(字节数)，最先入队的消息会被发送到DLX

队列在创建的时候可以指定一个死信交换机DLX（Dead Letter Exchange）。死信交换机绑定的队列成为死信队列（Dead Letter Queue），DLX实际上也是普通的交换机，DLQ也是普通的队列。

#### 1.2.1 死信队列的使用

声明原交换机（GP_ORI_USE_EXCHANGE）、原队列（GP_ORI_USE_QUEUE），相互绑定。队列中的消息 10 秒钟过期，因为没有消费者，会变成死信。指定原队列的死信交换机（GP_DEAD_LETTER_EXCHANGE）。 

```java
@Bean("oriUseExchange") 
public TopicExchange exchange() { 
  return new TopicExchange("GP_ORI_USE_EXCHANGE", true, false, new HashMap<>()); 
}
@Bean("oriUseQueue") 
public Queue queue() { 
  Map<String, Object> map = new HashMap<String, Object>(); 
  map.put("x-message-ttl", 10000); // 10 秒钟后成为死信 
  map.put("x-dead-letter-exchange", "GP_DEAD_LETTER_EXCHANGE"); // 队列中的消息变成死信后，进入死信 交换机
  return new Queue("GP_ORI_USE_QUEUE", true, false, false, map); 
}
@Bean public Binding binding(@Qualifier("oriUseQueue") Queue queue,@Qualifier("oriUseExchange") TopicExchange exchange) { 
  return BindingBuilder.bind(queue).to(exchange).with("gupao.ori.use"); 
}
```

声明死信交换机 （ GP_DEAD_LETTER_EXCHANGE ） 、 死信队列（GP_DEAD_LETTER_QUEUE），相互绑定 

``` java
@Bean("deatLetterExchange") 
public TopicExchange deadLetterExchange() { 
  return new TopicExchange("GP_DEAD_LETTER_EXCHANGE", true, false, new HashMap<>()); 
}
@Bean("deatLetterQueue") 
public Queue deadLetterQueue() { 
  return new Queue("GP_DEAD_LETTER_QUEUE", true, false, false, new HashMap<>()); 
} 
@Bean public Binding bindingDead(@Qualifier("deatLetterQueue") Queue queue,@Qualifier("deatLetterExchange") TopicExchange exchange) { 
  return BindingBuilder.bind(queue).to(exchange).with("#"); // 无条件路由 
}
```

![image-20201025143049303](RabbitMQ 高级.assets/image-20201025143049303.png)

### 1.3 延迟队列

我们在实际业务中有一些需要延时发送消息的场景，例如： 

1. 家里有一台智能热水器，需要在 30 分钟后启动 

2. 未付款的订单，15 分钟后关闭 

RabbitMQ 本身不支持延迟队列，总的来说有三种实现方案： 

1. 先存储到数据库，用定时任务扫描

2. 利用 RabbitMQ 的死信队列（Dead Letter Queue）实现 

3. 利用 rabbitmq-delayed-message-exchange 插件 

#### 1.3.1 TTL+DLX实现

消息的流转流程： 

生产者——原交换机——原队列（超过 TTL 之后）——死信交换机——死信队列——最终消费者 

使用死信队列实现延时消息的缺点： 

1. 如果统一用队列来设置消息的 TTL，当梯度非常多的情况下，比如 1 分钟，2分钟，5 分钟，10 分钟，20 分钟，30 分钟……需要创建很多交换机和队列来路由消息

2. 如果单独设置消息的 TTL，则可能会造成队列中的消息阻塞——前一条消息没有出队（没有被消费），后面的消息无法投递（比如第一条消息过期 TTL 是 30min，第二条消息 TTL 是 10min。10 分钟后，即使第二条消息应该投递了，但是由于第一条消息还未出队，所以无法投递）

3. 可能存在一定的时间误差

#### 1.3.2.基于延迟队列插件的实现（Linux）

在 RabbitMQ 3.5.7 及以后的版本提供了一个插件 `rabbitmq-delayed-message-exchange` 来实现延时队列功能。同时插件依赖Erlang/OPT 18.0 及以上。 

[下载插件](https://www.rabbitmq.com/community-plugins.html) --》 rabbitmq_delayed_message_exchange --》 Download for 3.7.x and 3.8.x

```markdown
# 上传 rabbitmq_delayed_message_exchange-3.8.0.ez 至
# Linux /usr/lib/rabbitmq/lib/rabbitmq_server-3.8.5/plugins
# Windows C:\Program Files\RabbitMQ Server\rabbitmq_server-3.8.5\plugins
	rabbitmq-plugins enable rabbitmq_delayed_message_exchange
```

```java
@Configuration
public class RabbitMqConfig {

   public static final String DELAY_EXCHANGE = "delay_exchange";

   public static  final String DELAY_QUEUE = "delay_queue";

   //延时队列
   @Bean
   public Queue delayQueue(){
      return new Queue(DELAY_QUEUE,true);
   }

   //延时交换机
   @Bean
   public CustomExchange delayExchange(){
      Map<String,Object> args = new HashMap<>();
      args.put("x-delayed-type", "direct");
      return new CustomExchange(DELAY_EXCHANGE, "x-delayed-message",true, false, args);
   }

   //绑定延时队列与交换机
   @Bean
   public Binding delayBind(){
      return BindingBuilder.bind(delayQueue()).to(delayExchange()).with("order.*").noargs();
   }
}
@Slf4j
@Component
public class DelayConsumer {
    @RabbitListener(queues = RabbitMqConfig.DELAY_QUEUE)
    public void process(String context, Message message, Channel channel)throws IOException {
        try {
            log.info("开始执行订单[{}]的支付超时订单关闭......", context);
            System.out.println(new Date() + " Delay get: " + context);
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
            log.info("超时订单{}处理完毕",context);
        }catch (Exception e){
            log.error("超时订单处理失败:{}",context);
            e.printStackTrace();
            //这里会不断消费吗？
            channel.basicReject(message.getMessageProperties().getDeliveryTag(),false);
        }
    }
}

// messge ==> MessagePostProcessor
rabbitTemplate.convertAndSend(RabbitMqConfig.DELAY_EXCHANGE, "order.delay", "延迟队列测试" , message -> {
    message.getMessageProperties().setDelay(20 * 60 * 1000);
    System.out.println(new Date() + " Delay sent.");
    return message;
});
```

### 1.4 服务端流控(Flow Control)

当 RabbitMQ 生产 MQ 消息的速度远大于消费消息的速度时，会产生大量的消息堆积，占用系统资源，导致机器的性能下降。我们想要控制服务端接收的消息的数量，应该怎么做呢？ 

队列有两个控制长度的属性： 

`x-max-length` 队列中最大存储最大消息数，超过这个数量，队头的消息会被丢弃。

`x-max-length-bytes` 队列中存储的最大消息容量（单位 bytes），超过这个容量，队头的消息会被丢弃。 

需要注意的是，设置队列长度只在消息堆积的情况下有意义，而且会删除先入队的消息，不能真正地实现服务端限流。 

#### 1.4.1 内存控制

RabbitMQ 会在启动时检测机器的物理内存数值。默认当MQ占用40%内存以上时，MQ 会主动抛出一个内存警告并阻塞所有连接（Connections）。可通过修改rabbitmq.config文件来调整内存阈值，默认是0.4：

```
[{rabbit, [{vm_memory_high_watermark, 0.4}]}].
```

也可以用命令动态设置，如果设置成 0，则所有的消息都不能发布。 

```
rabbitmqctl set_vm_memory_high_watermark 0.3 
```

#### 1.4.2 磁盘控制

另一种方式是通过磁盘来控制消息的发布。当磁盘空间低于指定的值时（默认50MB），触发流控措施。 

例如：指定为磁盘的 30%或者 2GB： 

```
disk_free_limit.relative = 3.0 
disk_free_limit.absolute = 2GB
```

### 1.5 消费端限流

默认情况下，如果不进行配置，RabbitMQ会尽可能快速地把队列中的消息发送给消费者。因为消费者会在本地缓存消息，如果消息数过多，可能会导致OOM或影响其他进程运行。

在消费者处理能力有限，例如消费者太少或单条消息处理时间过长，我们希望在一定数量的消息消费完之前，不再推送消息过来，就要用到消费端的流量限制措施。

可以基于 Consumer 或者 channel 设置 preferch count 的值，含义为 Consumer 端的最大的 unacked message 数目。当超过这个数值的消息未被确认，RabbitMQ 会停止投递新的消息给该消费者。

```java
channel.basicQos(2); // 如果超过 2 条消息没有发送 ACK，当前消费者不再接受队列消息 
channel.basicConsume(QUEUE_NAME, false, consumer); 
```

SimpleMessageListenerContainer 

```java
container.setPrefetchCount(2); 
```

Spring Boot 配置

```yaml
spring:
	rabbitmq:
		listener:
			simple:
				prefetch: 2 
```

## 2 Spring AMQP

### 2.1 SPRING AMQP 介绍

Spring 封装 RabbitMQ 的时候，它做了什么事情？ 

1. 管理对象（队列、交换机、绑定） 

2. 封装方法（发送消息、接收消息） 

Spring AMQP 是对 Spring 基于 AMQP 的消息收发解决方案，它是一个抽象层，不依赖于特定的 AMQP Broker 实现和客户端的抽象，所以可以很方便地替换。比如我们可以使用 spring-rabbit 来实现。 

```xml
<dependency> 
  <groupId>org.springframework.amqp</groupId> 
  <artifactId>spring-rabbit</artifactId> 
  <version>1.3.5.RELEASE</version>
</dependency>
```

包括 3 个 jar 包： 

Amqp-client-3.3.4.jar 

Spring-amqp.jar 

Spring.rabbit.jar 

### 2.2 Spring AMQP 核心组件

#### 2.2.1 ConnectionFactory

Spring AMQP 的连接工程接口，用于创建连接。CachingConnectionFactory 是 ConnectionFactory 的一个实现类。

```java
@Configuration
public class AmqpConfig {

    @Bean
    public ConnectionFactory connectionFactory() throws Exception {
        CachingConnectionFactory cachingConnectionFactory = new CachingConnectionFactory();
        cachingConnectionFactory.setUri(ResourceUtil.getKey("rabbitmq.uri"));
        return cachingConnectionFactory;
    }

    @Bean
    public RabbitAdmin amqpAdmin(ConnectionFactory connectionFactory) {
        RabbitAdmin admin = new RabbitAdmin(connectionFactory);
        // admin.setAutoStartup(true);
        return admin;
    }

    @Bean
    public SimpleMessageListenerContainer container(ConnectionFactory connectionFactory) {
        SimpleMessageListenerContainer container = new SimpleMessageListenerContainer(connectionFactory);
        container.setConsumerTagStrategy(new ConsumerTagStrategy() {
            public String createConsumerTag(String queue) {
                return null;
            }
        });
        return container;
    }

}
```

#### 2.2.2 RabbitAdmin

RabbitAdmin 是 AmqpAdmin 的实现，封装了对 RabbitMQ 的基础管理操作，比如对交换机、队列、绑定的声明和删除等。 

```java
@ComponentScan(basePackages = "com.gupaoedu.amqp")
public class AdminTest {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AdminTest.class);
        RabbitAdmin rabbitAdmin = context.getBean(RabbitAdmin.class);
        // 声明一个交换机
        rabbitAdmin.declareExchange(new DirectExchange("GP_ADMIN_EXCHANGE", false, false));
        // 声明一个队列
        rabbitAdmin.declareQueue(new Queue("GP_ADMIN_QUEUE", false, false, false));
        // 声明一个绑定
        rabbitAdmin.declareBinding( new Binding("GP_ADMIN_QUEUE", Binding.DestinationType.QUEUE,
                "GP_ADMIN_EXCHANGE", "admin", null));
    }
}
```

为什么我们在配置文件（Spring）或者配置类（SpringBoot）里面定义了交换机、队列、绑定关系，并没有直接调用 Channel 的 declare 的方法，Spring 在启动的时候就可以帮我们创建这些元数据？这些事情就是由 RabbitAdmin 完成的。 

RabbitAdmin 实 现 了 InitializingBean 接 口 ， 里面有唯一的一个方法 afterPropertiesSet()，这个方法会在 RabbitAdmin 的属性值设置完的时候被调用。 

在 afterPropertiesSet ()方法中，调用了一个 initialize()方法。这里面创建了三个 Collection，用来盛放交换机、队列、绑定关系。 最后依次声明返回类型为 Exchange、Queue 和 Binding 这些 Bean，底层还是调用了 Channel 的 declare 的方法。 

```java
    public void initialize() {
        if (this.applicationContext == null) {
            this.logger.debug("no ApplicationContext has been set, cannot auto-declare Exchanges, Queues, and Bindings");
        } else {
            this.logger.debug("Initializing declarations");
            Collection<Exchange> contextExchanges = new LinkedList(this.applicationContext.getBeansOfType(Exchange.class).values());
            Collection<Queue> contextQueues = new LinkedList(this.applicationContext.getBeansOfType(Queue.class).values());
            Collection<Binding> contextBindings = new LinkedList(this.applicationContext.getBeansOfType(Binding.class).values());
            this.processLegacyCollections(contextExchanges, contextQueues, contextBindings);
            this.processDeclarables(contextExchanges, contextQueues, contextBindings);
            Collection<Exchange> exchanges = this.filterDeclarables(contextExchanges);
            Collection<Queue> queues = this.filterDeclarables(contextQueues);
            Collection<Binding> bindings = this.filterDeclarables(contextBindings);
            Iterator var7 = exchanges.iterator();

            while(true) {
                Exchange exchange;
                do {
                    if (!var7.hasNext()) {
                        var7 = queues.iterator();

                        while(true) {
                            Queue queue;
                            do {
                                if (!var7.hasNext()) {
                                  ......
                                    this.rabbitTemplate.execute((channel) -> {
                                        this.declareExchanges(channel, (Exchange[])exchanges.toArray(new Exchange[exchanges.size()]));
                                        this.declareQueues(channel, (Queue[])queues.toArray(new Queue[queues.size()]));
                                        this.declareBindings(channel, (Binding[])bindings.toArray(new Binding[bindings.size()]));
                                        return null;
                                    });
                                   ......
    }
```

#### 2.2.3 Message

Message 是 Spring AMQP 对消息的封装。 

两个重要的属性： 

​	body：消息内容。 

​	messageProperties：消息属性。 

#### 2.2.4 RabbitTemplate 消息模版

RabbitTemplate 是 AmqpTemplate 的一个实现（目前为止也是唯一的实现），用来简化消息的收发，支持消息的确认（Confirm）与返回（Return）。跟 JDBCTemplate一 样 ， 它封装了创建连接 、创建消息信道 、收发消息 、消息格式转换（ConvertAndSend→Message）、关闭信道、关闭连接等等操作。 

针对于多个服务器连接，可以定义多个 Template。可以注入到任何需要收发消息的地方使用。 

```java
@Bean
public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
    RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
    rabbitTemplate.setMandatory(true);
    rabbitTemplate.setReturnCallback(new RabbitTemplate.ReturnCallback(){
        public void returnedMessage(Message message,
                                    int replyCode,
                                    String replyText,
                                    String exchange,
                                    String routingKey){
            System.out.println("回发的消息：");
            System.out.println("replyCode: "+replyCode);
            System.out.println("replyText: "+replyText);
            System.out.println("exchange: "+exchange);
            System.out.println("routingKey: "+routingKey);
        }
    });
    rabbitTemplate.setChannelTransacted(true);
    rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback() {
        public void confirm(CorrelationData correlationData, boolean ack, String cause) {
            if (!ack) {
                System.out.println("发送消息失败：" + cause);
                throw new RuntimeException("发送异常：" + cause);
            }
        }
    });
    return rabbitTemplate;
}

@ComponentScan(basePackages = "com.gupaoedu.amqp.template")
public class TemplateSender {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(TemplateSender.class);
        RabbitTemplate rabbitTemplate = context.getBean(RabbitTemplate.class);

        rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback(){
            public void confirm(CorrelationData correlationData, boolean ack, String cause) {
                if (ack) {
                    System.out.println("消息确认成功");
                } else {
                    // nack
                    System.out.println("消息确认失败");
                }
            }
        });

        rabbitTemplate.convertAndSend("GP_BASIC_FANOUT_EXCHANGE", "", "this is a msg");
    }

}
```

#### 2.2.5 MessageListener 消息侦听

**MessageListener** 

MessageListener 是 Spring AMQP 异步消息投递的监听器接口，它只有一个方法 onMessage，用于处理消息队列推送来的消息，作用类似于 Java API 中的 Consumer。 

**MessageListenerContainer** 

MessageListenerContainer可以理解为MessageListener的容器，一个Container 只有一个 Listener，但是可以生成多个线程使用相同的 MessageListener 同时消费消息。

Container 可以管理 Listener 的生命周期，可以用于对于消费者进行配置。 

例如：动态添加移除队列、对消费者进行设置，例如 ConsumerTag、Arguments、并发、消费者数量、消息确认模式等等。 

```java
@Bean
public SimpleMessageListenerContainer container(ConnectionFactory connectionFactory) {
    SimpleMessageListenerContainer container = new SimpleMessageListenerContainer(connectionFactory);
    container.setQueues(getSecondQueue(), getThirdQueue()); //监听的队列
  	container.setConcurrentConsumers(1); // 最小消费者数 
  	container.setMaxConcurrentConsumers(5); // 最大的消费者数量
 		container.setDefaultRequeueRejected(false); //是否重回队列 
 		container.setAcknowledgeMode(AcknowledgeMode.AUTO); //签收模式 
	  container.setExposeListenerChannel(true); 
	  container.setConsumerTagStrategy(new ConsumerTagStrategy() { //消费端的标签策略 
	    @Override public String createConsumerTag(String queue) { 
 	     return queue + "_" + UUID.randomUUID().toString(); 
	    } 
	  });
    return container;
}
```

在 SpringBoot2.0 中新增了一个 DirectMessageListenerContainer。

**MessageListenerContainerFactory**

```java
/**
 * 在消费端转换JSON消息
 * 监听类都要加上containerFactory属性
 * @param connectionFactory
 * @return
 */
@Bean
public SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory(ConnectionFactory connectionFactory) {
    SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
    factory.setConnectionFactory(connectionFactory);
    factory.setMessageConverter(new Jackson2JsonMessageConverter());
    factory.setAcknowledgeMode(AcknowledgeMode.MANUAL);
    factory.setAutoStartup(true);
    return factory;
}
```

可以在消费者上指定，当我们需要监听多个 RabbitMQ 的服务器的时候，指定不同的 MessageListenerContainerFactory。 

```java
@Component
@PropertySource("classpath:gupaomq.properties")
@RabbitListener(queues = "${com.gupaoedu.firstqueue}", containerFactory="rabbitListenerContainerFactory")
public class FirstConsumer {
    @RabbitHandler
    public void process(@Payload Merchant merchant){
        System.out.println("First Queue received msg : " + merchant.getName());
    }
}
```

```java
/**
 * 配置类的代码用不到，只用来演示
 */
public class ContainerSender {
    public static void main(String[] args) throws Exception {
        ConnectionFactory connectionFactory = new CachingConnectionFactory(new URI("amqp://guest:guest@localhost:5672"));
        SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory);
        SimpleMessageListenerContainer container = factory.createListenerContainer();
        // 不用工厂模式也可以创建
        // SimpleMessageListenerContainer container = new SimpleMessageListenerContainer(connectionFactory);
        container.setConcurrentConsumers(1);
        container.setQueueNames("GP_BASIC_SECOND_QUEUE");
        container.setMessageListener(new MessageListener() {
            @Override
            public void onMessage(Message message) {
                System.out.println("收到消息："+message);
            }
        });
        container.start();

        AmqpTemplate template = new RabbitTemplate(connectionFactory);
        template.convertAndSend("GP_BASIC_SECOND_QUEUE", "msg 1");
        template.convertAndSend("GP_BASIC_SECOND_QUEUE", "msg 2");
        template.convertAndSend("GP_BASIC_SECOND_QUEUE", "msg 3");
    }

}
```

#### 2.2.6 转换器 MessageConvertor

**MessageConvertor 的作用？** 

RabbitMQ 的消息在网络传输中需要转换成 byte[]（字节数组）进行发送，消费者需要对字节数组进行解析。 

在 Spring AMQP 中，消息会被封装为 org.springframework.amqp.core.Message对象。消息的序列化和反序列化，就是处理 Message 的消息体 body 对象。 

如果消息已经是 byte[]格式，就不需要转换。 

如果是 String，会转换成 byte[]。 

如果是 Java 对象，会使用 JDK 序列化将对象转换为 byte[]（体积大，效率差）。 

在调用 RabbitTemplate 的 convertAndSend() 方法发送消息时 ， 会使用 MessageConvertor 进行消息的序列化，默认使用 SimpleMessageConverter。 

在某些情况下，我们需要选择其他的高效的序列化工具。如果我们不想在每次发送消息时自己处理消息，就可以直接定义一个 MessageConvertor。 

```java
@Bean 
public RabbitTemplate rabbitTemplate(final ConnectionFactory connectionFactory) { 
  final RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory); 
  rabbitTemplate.setMessageConverter(new Jackson2JsonMessageConverter()); 
  return rabbitTemplate; 
}
```

**MessageConvertor 如何工作？** 

调用了 RabbitTemplate 的 convertAndSend() 方法时会使用对应的 MessageConvertor 进行消息的序列化和反序列化。 

序列化：Object —— Json —— Message(body) —— byte[] 

反序列化：byte[] ——Message —— Json —— Object 

**有哪些 MessageConvertor？** 

在 Spring 中提供了一个默认的转换器：SimpleMessageConverter。 

Jackson2JsonMessageConverter（RbbitMQ 自带）：将对象转换为 json，然后再转换成字节数组进行传递。 

**如何自定义 MessageConverter？** 

例如：我们要使用 Gson 格式化消息： 

创建一个类，实现 MessageConverter 接口，重写 toMessage()和 fromMessage() 方法。

toMessage(): Java 对象转换为 Message 

fromMessage(): Message 对象转换为 Java 对象

![image-20201025153900158](RabbitMQ 高级.assets/image-20201025153900158.png)

## 3 SpringBoot 中使用 RabbitMQ

> MQ 的消息什么时候发，在哪一层发，是先更新数据库后发送 MQ 消息还是后先发送 MQ 消息发送数据库。 
>
> 这里的顺序千万要注意。一定是先更新数据库后发送消息。否则，数据库回滚的话 就会导致数据一致性问题。 
>
> 但是，如果先更新数据库成功，后发送消息失败了呢？比如，服务器没有成功接收， 或者路由出现问题，或者在 Queue 存储出现问题，或者消费者消费时出现问题，怎么解 决？

### 3.1 搭建环境

**1.引入依赖**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

**2.配置文件**

```yml
server:
  port: 83
spring:
  application:
    name: chinook-order
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
    virtual-host: /ems
```

`RabbitTemplate`  用来简化操作     使用时候直接在项目中注入即可使用。

### 3.2 配置文件

**定义交换机**

```java
@Bean("vipDirectExchange") 
public DirectExchange getDirectExchange(){ 
  return new DirectExchange(directExchange); 
}
@Bean("vipTopicExchange") 
public TopicExchange getTopicExchange(){ 
  return new TopicExchange(topicExchange); 
}
@Bean("vipFanoutExchange") 
public FanoutExchange getFanoutExchange(){ 
  return new FanoutExchange(fanoutExchange); 
}
```

**定义队列**

```java
@Bean("vipFirstQueue") 
public Queue getFirstQueue(){ 
  return new Queue(firstQueue); 
}
@Bean("vipSecondQueue") 
public Queue getSecondQueue(){ 
  return new Queue(secondQueue); 
}
@Bean("vipThirdQueue") 
public Queue getThirdQueue(){ 
  return new Queue(thirdQueue); 
}
@Bean("vipFourthQueue") 
public Queue getFourthQueue(){ 
  return new Queue(fourthQueue); 
}
```

**定义绑定**

```java
@Bean 
public Binding bindFirst(@Qualifier("vipFirstQueue") Queue queue, @Qualifier("vipDirectExchange") DirectExchange exchange){ 
  return BindingBuilder.bind(queue).to(exchange).with("gupao.best"); 
}
@Bean 
public Binding bindSecond(@Qualifier("vipSecondQueue") Queue queue, @Qualifier("vipTopicExchange") TopicExchange exchange){ 
  return BindingBuilder.bind(queue).to(exchange).with("*.gupao.*"); 
} 
@Bean 
public Binding bindThird(@Qualifier("vipThirdQueue") Queue queue, @Qualifier("vipFanoutExchange") FanoutExchange exchange){ 
  return BindingBuilder.bind(queue).to(exchange); 
}
@Bean 
public Binding bindFourth(@Qualifier("vipFourthQueue") Queue queue, @Qualifier("vipFanoutExchange") FanoutExchange exchange){ 
  return BindingBuilder.bind(queue).to(exchange); 
}
```

### 3.3 消费者

在消费者类中可以有多个处理（不同类型的消息）的方法。

```java
@Component 
@PropertySource("classpath:gupaomq.properties") 
@RabbitListener(queues = "${com.gupaoedu.firstqueue}") 
public class FirstConsumer { 
  @RabbitHandler 
  public void process(@Payload Merchant merchant){ 
    System.out.println("First Queue received msg : " + merchant.getName()); 
  } 
}
```

![image-20201025160719948](RabbitMQ 高级.assets/image-20201025160719948.png)

### 3.4 生产者

```java
// 自定义的模板，所有的消息都会转换成JSON发送
@Autowired
AmqpTemplate gupaoTemplate;
public void send() throws JsonProcessingException {
    Merchant merchant =  new Merchant(1001,"a direct msg : 中原镖局","汉中省解放路266号");
    gupaoTemplate.convertAndSend(directExchange,directRoutingKey, merchant);
    gupaoTemplate.convertAndSend(topicExchange,topicRoutingKey1, "a topic msg : shanghai.gupao.teacher");
    gupaoTemplate.convertAndSend(topicExchange,topicRoutingKey2, "a topic msg : changsha.gupao.student");
    // 发送JSON字符串
    ObjectMapper mapper = new ObjectMapper();
    String json = mapper.writeValueAsString(merchant);
    System.out.println(json);
    gupaoTemplate.convertAndSend(fanoutExchange,"", json);
}
```

### 3.5 Spring Boot 参数解析

注：前缀 spring.rabbitmq.全部省略 

全部配置总体上分成三大类：连接类、消息消费类、消息发送类 

基于 Spring Boot 2.1.5 

| 属性值                                   | 说明                                                         | 默认值    |
| ---------------------------------------- | ------------------------------------------------------------ | --------- |
| address                                  | 客户端连接的地址，有多个的时候使用逗号分隔，该地址可以是 IP 与 Port 的结合 |           |
| host                                     | RabbitMQ 的主机地址                                          | localhost |
| port                                     | RabbitMQ 的端口号                                            |           |
| virtual-host                             | 连接到 RabbitMQ 的虚拟主机                                   |           |
| username                                 | 登录到 RabbitMQ 的用户名                                     |           |
| password                                 | 登录到 RabbitMQ 的密码                                       |           |
| ssl.enabled                              | 启用 SSL 支持                                                | false     |
| ssl.key-store                            | 保存 SSL 证书的地址                                          |           |
| ssl.key-store-password                   | 访问 SSL 证书的地址使用的密码                                |           |
| ssl.trust-store                          | SSL 的可信地址                                               |           |
| ssl.trust-store-password                 | 访问 SSL 的可信地址的密码                                    |           |
| ssl.algorithm                            | SSL 算法，默认使用 Rabbit 的客户端算法库                     |           |
| cache.channel.checkout-timeout           | 当缓存已满时，获取 Channel 的等待时间，单位为毫秒            |           |
| cache.channel.size                       | 缓存中保持的 Channel 数量                                    |           |
| cache.connection.mode                    | 连接缓存的模式                                               | CHANNEL   |
| cache.connection.size                    | 缓存的连接数                                                 |           |
| connnection-timeout                      | 连接超时参数单位为毫秒：设置为“0”代表无穷大                  |           |
| dynamic                                  | 默认创建一个 AmqpAdmin 的 Bean                               | true      |
| listener.simple.acknowledge-mode         | 容器的 acknowledge 模式                                      |           |
| listener.simple.auto-startup             | 启动时自动启动容器                                           | true      |
| listener.simple.concurrency              | 消费者的最小数量                                             |           |
| listener.simple.default-requeue-rejected | 投递失败时是否重新排队                                       | true      |
| listener.simple.max-concurrency          | 消费者的最大数量                                             |           |
| listener.simple.missing-queues-fatal     | 容器上声明的队列不可用时是否失败                             |           |
| listener.simple.prefetch                 | 在单个请求中处理的消息个数，他应该大于等于事务数量           |           |
| listener.simple.retry.enabled            | 不论是不是重试的发布                                         | false     |
| listener.simple.retry.initial-interval   | 第一次与第二次投递尝试的时间间隔                             | 1000ms    |
| listener.simple.retry.max-attempts       | 尝试投递消息的最大数量                                       | 3         |
| listener.simple.retry.max-interval       | 两次尝试的最大时间间隔                                       | 10000ms   |
| listener.simple.retry.multiplier         | 上一次尝试时间间隔的乘数                                     | 1.0       |
| listener.simple.retry.stateless          | 重试是有状态的还是无状态的                                   | true      |
| listener.simple.transaction-size         | 在一个事务中处理的消息数量。为了获得最佳效果，该值应设置 为小于等 于每个请 求中处理的 消息个数 ，即listener.prefetch 的值 |           |
| publisher-confirms                       | 开启 Publisher Confirm 机制                                  |           |
| publisher-returns                        | 开启 Publisher Return 机制                                   |           |
| template.mandatory                       | 启用强制信息                                                 | false     |
| template.receive-timeout                 | receive()方法的超时时间                                      | 0         |
| template.reply-timeout                   | sendAndReceive()方法的超时时间                               | 5000      |
| template.retry.enabled                   | 设置为 true 的时候 RabbitTemplate 能够实现重试               | false     |
| template.retry.initial-interval          | 第一次与第二次发布消息的时间间隔                             | 1000      |
| template.retry.max-attempts              | 尝试发布消息的最大数量                                       | 3         |
| template.retry.max-interval              | 尝试发布消息的最大时间间隔                                   | 10000     |
| template.retry.multiplier                | 上一次尝试时间间隔的乘数                                     | 1.0       |

## 4 RabbitMQ 可靠性投递

在 RabbitMQ 里面提供了很多保证消息可靠投递的机制，这个也是 RabbitMQ 的一个特性。

在保证可靠性的同时，必须要明确一个问题，因为效率与可靠性是无法兼得的，如果要保证每一个环节都成功，势必会对消息的收发效率造成影响。

回顾一下 RabbitMQ 的工作模型：

![image-20201101181219562](RabbitMQ 高级.assets/image-20201101181219562.png)

在使用 RabbitMQ 收发消息的时候，有几个主要环节：

1. 消息从生产者发送到 Broker（怎么知道自己的消息有没有被Broker成功接收）
2. 消息从Exchange路由到Queue（Exchange是一个绑定列表，如果消息没有办法路由到正确的队列，会发什么什么，怎么处理）
3. 消息在Queue存储，队列是一个独立运行的服务，有自己的数据库 Mnesia，它是真正用来存储消息的。如果还没有消费者来消费，那么消息要一直存储在队列里（如果队列出现问题，消息肯定会丢失，怎么保证消息在队列稳定地存储）
4. 消费者订阅Queue并消费消息，队列FIFO，队列里的消息是一条一条投递的，只要上一条消息被消费者接收以后，才能把一条消息从数据库删掉，继续投递下一条消息（Broker怎么知道消费者已经接收到消息）

### 4.1 消息发送到 RabbitMQ 服务器

在 RabbitMQ 中提供了两种`服务端确认机制`，也就是在生产者发送消息给 RabbitMQ 的服务端时候，服务端会通过某种方式返回一个应答，只要生产者收到了这个应答，就知道消息发送成功。

#### Transaction 事务模式

通过 `channel.txSelect()` 方法把信道设置成事务模式，然后就可以发布消息给RabbitMQ，如果 `channel.txCommit()` 方法调用成功，就说明事务提交成功，则消息一定到达了 RabbitMQ 中。

如果在事务提交执行之前由于RabbitMQ异常崩溃或其他原因抛出异常，这个时候可以将异常捕获，通过 `channel.txRollback() `来回滚事务。

![image-20201101182355244](RabbitMQ 高级.assets/image-20201101182355244.png)

在事务模式中只有接收到服务端的 Commit-OK 指令，才能提交成功，所以可以解决生产者和服务端消息确认的问题。但是事务模式有一个缺点，它是阻塞的，一条消息没有发送完毕，不能发送下一条消息，它会榨干RabbitMQ服务器的性能，所以不建议生产环境使用。

```java
/**
* 原生API
*/
try {
    // 开启事务模式
    channel.txSelect();
    // 发送消息
    channel.basicPublish("logs", "", null, "fanout type message".getBytes());
    //int i = 1 / 0;
    channel.txCommit();
    System.out.println("消息发送成功");
} catch (Exception e) {
    channel.txRollback();
    System.out.println("消息已经会滚");
  
/**
* spring boot
*/
rabbitTemplate.setChannelTransacted(true);
```

#### Confirm 确认模式

**普通确认模式**

在生产者通过调用 ``channel.confirmSelect()`` 方法将信道设置成为 Confirm 模式，然后发送消息。所有在该信道上面发布的消息都会被指派一个唯一的 ID(从 1 开始)，一旦消息`被投递到所有匹配的队列`之后，RabbitMQ 就会发送确认(Basic.ACK)给生产者(包含消息的唯一 ID)，也就是调用`channel.waitForConfirms()`返回true，这样生产者就知道消息被服务端接收了。

```java
// 指定我们的消息投递模式: 消息的确认模式
channel.confirmSelect();

channel.exchangeDeclare("topics", "topic");
channel.basicPublish("topics", "user.save", null, ("topic type message, key: " + routingKey).getBytes());

try {
    // 普通confirm，发送一条确认一条
    if (channel.waitForConfirms()) {
        System.out.println("消息发送完毕");
    }
} catch (InterruptedException e) {
    e.printStackTrace();
}
```

这种发送 1 条确认 1 条的方式消息还不是太高，所以我们还有一种批量确认的方式。`批量确认`就是在开启 Confirm 模式后， 先发送一批消息 。 只要 `channel.waitForConfirmsOrDie()` 方法没有抛出异常，就代表消息都被服务端接收了。

```java
try {
    // 批量确认
    channel.waitForConfirmsOrDie();
    System.out.println("消息全部发送完毕");
} catch (InterruptedException e) {
    e.printStackTrace();
}
```

批量确认的方式比单条确认的方式效率要高，但是也有两个问题，第一个就是批量的数量的确定。对于不同的业务，到底发送多少条消息确认一次？数量太少，效率提升不上去。数量多的话，又会带来另一个问题，比如我们发 1000 条消息才确认一次，如果前面 999 条消息都被服务端接收了，如果第 1000 条消息被拒绝了，那么前面所有的消息都要重发。 

有没有一种方式，可以一边发送一边确认的呢？这个就是`异步确认模式`。 

异步确认模式需要添加一个 ConfirmListener，并且用一个 SortedSet 来维护没有被确认的消息。 

Confirm 模式是在 Channel 上开启的，因为 RabbitTemplate 对 Channel 进行了封装，叫做 ConfimrCallback。 

```java
/**
* 原生API
*/
// 指定我们的消息投递模式: 消息的确认模式
channel.confirmSelect();
// 添加一个异步确认监听
channel.addConfirmListener(new ConfirmListener() {
    @Override
    public void handleAck(long deliveryTag, boolean multiple) throws IOException {
        System.err.println("-------ack!-----------");
    }
    @Override
    public void handleNack(long deliveryTag, boolean multiple) throws IOException {
        System.err.println("-------no ack!-----------");
    }
});
```

Spring Boot:

```yml
spring:
  rabbitmq:
    publisher-confirms: true
```

```java
@Bean
public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
    RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
    rabbitTemplate.setConfirmCallback((correlationData, ack, cause) -> {
        if (ack) {
            System.out.println("发送消息成功");
        }
        if (!ack) {
            System.out.println("发送消息失败：" + cause);
            throw new RuntimeException("发送异常：" + cause);
        }
    });
    return rabbitTemplate;
}
```

> ```java
> channel.basicPublish("topics", "user123.save456", null, ("topic type message, key: " + routingKey).getBytes());
> ```
>
> 这种情况下，交换机无法根据自身的类型和路由键找到一个符合条件的队列，默认情况下消息会被丢弃，并返回生产者 ACK。

### 4.2 消息从交换机路由到队列

`路由键错误或队列不存在` 会导致消息无法路由到正确的队列。

这里有两种方式处理无法路由的消息，一种就是让服务端重发给生产者，一种是让交换机路由到另一个备份的交换机。

消息回发的方式：使用 `mandatory` 参数和 `ReturnListener`：

```java
// 监听器收到了无法路由的消息
channel.addReturnListener(new ReturnListener() {
    @Override
    public void handleReturn(int replyCode, String replyText,
                             String exchange, String routingKey,
                             AMQP.BasicProperties properties, byte[] body)
            throws IOException {
        String msg = new String(body);
        System.out.println("replyText:"+replyText);
        System.out.println("exchange:"+exchange);
        System.out.println("routingKey:"+routingKey);
        System.out.println("msg:"+msg);
    }
});

// 无法路由的情况
String routingKey = "user1212.save";
// 第三个参数设置mandatory，如果mandatory为false，消息会被直接丢弃
channel.basicPublish("topics", routingKey, true, null, ("topic type message, key: " + routingKey).getBytes());
```

在 spring AMQP 中是`ReturnCallback`：

```java
@Bean
public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
    RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
    
    rabbitTemplate.setMandatory(true);
    rabbitTemplate.setReturnCallback((message, replyCode, replyText, exchange, routingKey) -> {
        System.out.println("replyCode:"+replyCode);
        System.out.println("replyText:"+replyText);
        System.out.println("exchange:"+exchange);
        System.out.println("routingKey:"+routingKey);
    });
    return rabbitTemplate;
}
```

消息路由到备份交换机的方式：在创建交换机的时候，从属性中指定备份交换机。

```java
Map<String,Object> arguments = new HashMap<String,Object>(); 
arguments.put("alternate-exchange","ALTERNATE_EXCHANGE"); 
// 指定交换机的备份交换机 
channel.exchangeDeclare("TEST_EXCHANGE","topic", false, false, false, arguments);
```

注意区别，队列可以指定死信交换机；交换机可以指定备份交换机。

### 4.3 消息在队列存储

第三个环节是消息在队列存储，如果没有消费者的话，队列一直存在在数据库中。 

如果 RabbitMQ 的服务或者硬件发生故障，比如系统宕机、重启、关闭等等，可能会导致内存中的消息丢失，所以我们要把消息本身和元数据（队列、交换机、绑定）都保存到磁盘。

**队列持久化** 

```java
@Bean("GpQueue") 
public Queue GpQueue() { 
  // queueName, durable, exclusive, autoDelete, Properties 
  return new Queue("GP_TEST_QUEUE", true, false, false, new HashMap<>()); 
} 

// org.springframework.amqp.core.Queue 的构造方法默认是持久化的
public Queue(String name) {
	this(name, true, false, false);
}
```

**交换机持久化**

```java
@Bean("GpExchange") 
public DirectExchange exchange() { 
  // exchangeName, durable, exclusive, autoDelete, Properties 
  return new DirectExchange("GP_TEST_EXCHANGE", true, false, new HashMap<>()); 
}

// org.springframework.amqp.core.AbstractExchange 的构造方法默认是持久化的
public DirectExchange(String name) {
		super(name);
}
public AbstractExchange(String name) {
	this(name, true, false);
}
```

**消息持久化**

```java
// 原生API
//channel.basicPublish("", queue_name, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes());

rabbitTemplate.convertAndSend("exchange", "routingKey", msg);
public void convertAndSend(String exchange, String routingKey, Object object) throws AmqpException {
    this.convertAndSend(exchange, routingKey, object, (CorrelationData)null);
}
public void convertAndSend(String exchange, String routingKey, Object object, @Nullable CorrelationData correlationData) throws AmqpException {
    this.send(exchange, routingKey, this.convertMessageIfNecessary(object), correlationData);
}
// 最关键的方法出现了，它是 convertMessageIfNecessary(final Object object)
protected Message convertMessageIfNecessary(Object object) {
    return object instanceof Message ? (Message)object : this.getRequiredMessageConverter().toMessage(object, new MessageProperties());
}
// org.springframework.amqp.core.MessageProperties 的deliveryMode属性默认为持久化的
private volatile MessageDeliveryMode deliveryMode = DEFAULT_DELIVERY_MODE;
public static final MessageDeliveryMode DEFAULT_DELIVERY_MODE = MessageDeliveryMode.PERSISTENT;

// org.springframework.amqp.core.Message 实力化时，需要指定一个MessageProperties
public Message(byte[] body, MessageProperties messageProperties) { //NOSONAR
	this.body = body; //NOSONAR
	this.messageProperties = messageProperties;
}

MessageProperties messageProperties = new MessageProperties(); messageProperties.setDeliveryMode(MessageDeliveryMode.PERSISTENT); 
Message message = new Message("持久化消息".getBytes(), messageProperties); 
rabbitTemplate.send("GP_TEST_EXCHANGE", "gupao.test", message);
```

**集群**

如果只有一个 RabbitMQ 的节点，即使交换机、队列、消息做了持久化，如果服务崩溃或者硬件发生故障，RabbitMQ 的服务一样是不可用的，所以为了提高 MQ 服务的可用性，保障消息的传输，我们需要有多个 RabbitMQ 节点。

### 4.4 消息投递到消费者

RabbitMQ 提供了消费者的`消息确认机制（message acknowledgement）`，消费者可以自动或者手动地发送 ACK 给服务端。 

如果不手动确认，也不抛出异常，<font color=red>消息不会自动重新推送（包括其他消费者，但如果消费端应用重启，消息会自动重新推送）</font>，因为对于rabbitmq来说始终没有接收到消息消费是否成功的确认，并且Channel是在消费端有缓存的，没有断开连接。

如果没有收到 ACK 的消息，消费者断开连接后（不管是网络问题还是宕机），RabbitMQ 会把这条消息发送给其他消费者。如果没有其他消费者，消费者重启后会重新消费这条消息，重复执行业务逻辑。 

消费者在订阅队列时，可以指定 autoAck 参数，当 autoAck 等于 false 时，RabbitMQ 会等待消费者显式地回复确认信号后才从队列中移去消息。 

如果消息无法处理或者消费失败，也有两种`拒绝的方式`，`Basic.Reject()`拒绝单条， `Basic.Nack()`批量拒绝。如果 requeue 参数设置为 true，可以把这条消息重新存入队列， 以便发给下一个消费者（当然，只有一个消费者的时候，这种方式可能会出现无限循环重复消费的情况。可以投递到新的队列中，或者只打印异常日志）。 

```java
/**
* 原生API
*/
// 消费消息
channel.basicConsume(queueName, false, new DefaultConsumer(channel){
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
        System.out.println("消费者1：" + new String(body) + "====" + envelope.getDeliveryTag());
        // basic 方法的 multiple 参数，表示到这个序号之前的所有消息都得到处理
        channel.basicAck(envelope.getDeliveryTag(), false);
        // 拒绝消息
        // requeue 是否重新入队列，true 是，false 直接丢弃，相当于告诉队列可以直接删除掉
        channel.basicReject(envelope.getDeliveryTag(), false);
        // 批量拒绝
        // multiple true可拒绝包含DeliveryTag的所有消息；如果为false，则仅拒绝当前DeliveryTag
        // requeue
        channel.basicNack(envelope.getDeliveryTag(), true, false);
    }
});
```

 Spring AMQP：

SimpleRabbitListenerContainer 或者 SimpleRabbitListenerContainerFactory 

```java
factory.setAcknowledgeMode(AcknowledgeMode.MANUAL); 
```

```yml
spring:
  rabbitmq:
    listener:
      simple:
        acknowledge-mode: manual # 手动应答
        # NONE 自动应答
        # AUTO
          # 1.没有异常，自动ACK
          # 2.抛出ImmediateAcknowledgeAmqpException异常时，消费者会发送 ACK
          # 3.抛出AmqpRejectAndDontRequeueException异常时，消息被拒绝,不重新入队
          # 4.其他的异常，消息被拒绝，重新入队
```

```java
@Component
public class TopicConsumer {

    @RabbitListener(bindings = {
            @QueueBinding(
                    value = @Queue,
                    exchange = @Exchange(value = "topics", type = "topic"),
                    key = {"user.*"}
            )
    })
    public void receive(String msg, Channel channel, Message message) throws IOException {
        try {
            System.out.println("message = " + msg);
            //int i = 1 / 0;
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
        } catch (Exception e) {
            // multiple true可拒绝包含DeliveryTag的所有消息；如果为false，则仅拒绝当前DeliveryTag
            // requeue 是否重新入队列，true 是，false 直接丢弃
            channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, true);
        }
    }
}
```

思考：服务端收到了 ACK 或者 NACK，生产者会知道吗？即使消费者没有接收到消息，或者消费时出现异常，`生产者也是完全不知情的`（requeue 为 true 情况下，只是将消息重返给队列，但生产者得不到任何消息，而且重返队列然后再分配给消费者可能会导致永远无法消费，循环往复）。 

这个是生产者最终确定消费者有没有消费成功的两种方式： 

1） 消费者收到消息，处理完毕后，调用生产者的 API（思考：是否破坏解耦？） 

2） 消费者收到消息，处理完毕后，发送一条响应消息给生产者 

### 4.5 消费者回调

1） 调用生产者 API 

例如：提单系统给其他系统发送了碎屏保消息后，其他系统必须在处理完消息后调用提单系统提供的 API，来修改提单系统中数据的状态。只要 API 没有被调用，数据状态没有被修改，提单系统就认为下游系统没有收到这条消息。 

2） 发送响应消息给生产者 

例如：商业银行与人民银行二代支付通信，无论是人行收到了商业银行的消息，还是商业银行收到了人行的消息，都必须发送一条响应消息（叫做回执报文）。

![image-20201101193607869](RabbitMQ 高级.assets/image-20201101193607869.png)

### 4.6 补偿机制

如果生产者的 API 就是没有被调用，也没有收到消费者的响应消息，怎么办？ 

不要着急，可能是消费者处理时间太长或者网络超时。 生产者与消费者之间应该约定一个超时时间，比如 5 分钟，对于超出这个时间没有得到响应的消息，可以设置一个定时重发的机制，但要发送间隔和控制次数，比如每隔 2 分钟发送一次，最多重发 3 次，否则会造成消息堆积。 

重发可以通过`消息落库+定时任务`来实现。 

重发，是否发送一模一样的消息？ 

参考： 

ATM 机上运行的系统叫 C 端（ATMC），前置系统叫 P 端（ATMC），它接收 ATMC 的消息，再转发给卡系统或者核心系统。 

1）如果客户存款，没有收到核心系统的应答，不知道有没有记账成功，最多发送 5 次存款确认报文，因为已经吞钞了，所以要保证成功； 

2）如果客户取款，ATMC 未得到应答时，最多发送 5 次存款冲正报文。因为没有吐钞，所以要保证失败。 

### 4.7 消息幂等性

如果消费者每一次接收生产者的消息都成功了，只是在响应或者调用 API 的时候出了问题，会不会出现消息的重复处理？例如：存款 100 元，ATM 重发了 5 次，核心系统一共处理了 6 次，余额增加了 600 元。 

所以，为了避免相同消息的重复处理，必须要采取一定的措施。RabbitMQ 服务端是没有这种控制的（同一批的消息有个递增的 DeliveryTag），它不知道你是不是就要把一条消息发送两次，只能在消费端控制。 

如何避免消息的重复消费？ 

消息出现重复可能会有两个原因： 

1、生产者的问题，环节①重复发送消息，比如在开启了 Confirm 模式但未收到确认，消费者重复投递。 

2、环节④出了问题，由于消费者未发送 ACK 或者其他原因，消息重复投递。 

3、生产者代码或者网络问题。 

业界主流的幂等性操作：

- 唯一ID + 指纹码机制，利用数据库主键去重
- 利用Redis的原子性去实现

#### 唯一ID+指纹码机制

```mysql
# 唯一ID + 指纹码机制，利用数据库主键去重
SELECT COUNT(1) FROM T_ORDER WHERE ID = 唯一ID +指纹码
```

- 好处：实现简单
- 坏处：高并发下有数据库写入的性能瓶颈
- 解决方案：跟进ID进行分库分表进行算法路由

整个思路就是首先我们需要根据消息生成一个全局唯一的ID，然后还需要加上一个指纹码。这个指纹码它并不一定是系统去生成的，而是一些外部的规则或者内部的业务规则去拼接，它的目的就是为了保障这次操作是绝对唯一的。

将ID + 指纹码拼接好的值作为数据库主键，就可以进行去重了。即在`消费消息前，先去数据库查询`这条消息的指纹码标识是否存在，没有就执行insert操作，如果有就代表已经被消费了，就不需要管了。

对于高并发下的数据库性能瓶颈，可以跟进ID进行分库分表策略，采用一些路由算法去进行分压分流。应该保证ID通过这种算法，消息即使投递多次都落到同一个数据库分片上，这样就由单台数据库幂等变成多库的幂等。

#### 利用Redis的原子性去实现

我们都知道redis是单线程的，并且性能也非常好，提供了很多原子性的命令。比如可以使用 `setnx` 命令。

在接收到消息后将消息ID作为key执行 `setnx` 命令，如果执行成功就表示没有处理过这条消息，可以进行消费了，执行失败表示消息已经被消费了。

使用 redis 的原子性去实现主要需要考虑两个点

- 第一：我们是否要进行数据落库，如果落库的话，关键解决的问题是数据库和缓存如何做到原子性？
- 第二：如果不进行落库，那么都存储到缓存中，如何设置定时同步的策略(同步到关系型数据库)？缓存又如何做到数据可靠性保障呢

关于不落库，定时同步的策略，目前主流方案有两种：

第一种为双缓存模式，异步写入到缓存中，也可以异步写到数据库，但是最终会有一个回调函数检查，这样能保障最终一致性，不能保证100%的实时性。

第二种是定时同步，比如databus同步。

### 4.8 最终一致

如果确实是消费者宕机了，或者代码出现了 BUG 导致无法正常消费，在我们尝试多次重发以后，消息最终也没有得到处理，怎么办？ 

例如存款的场景，客户的钱已经被吞了，但是余额没有增加，这个时候银行出现了长款，应该怎么处理？如果客户没有主动通知银行，这个问题是怎么发现的？银行最终怎么把这个账务做平？ 

在我们的金融系统中，都会有双方对账或者多方对账的操作，通常是在一天的业务结束之后，第二天营业之前。我们会约定一个标准，比如 ATM 跟核心系统对账，肯定是以核心系统为准。ATMC 获取到核心的对账文件，然后解析，登记成数据，然后跟自己记录的流水比较，找出核心有 ATM 没有，或者 ATM 有核心没有，或者两边都有但是金额不一致的数据。 

对账之后，我们再手工平账。比如取款记了账但是没吐钞的，做一笔冲正。存款吞了钞没记账的，要么把钱退给客户，要么补一笔账。 

### 4.9 消息的顺序性

消息的顺序性指的是消费者消费消息的顺序跟生产者生产消息的顺序是一致的。 

例如：商户信息同步到其他系统，有三个业务操作：1、新增门店 2、绑定产品 3、激活门店，这种情况下消息消费顺序不能颠倒（门店不存在时无法绑定产品和激活）。 

又比如：1、发表微博；2、发表评论；3、删除微博。顺序不能颠倒。 

在 RabbitMQ 中，一个队列有多个消费者时，由于不同的消费者消费消息的速度是不一样的，顺序无法保证。`只有一个队列仅有一个消费者的情况才能保证顺序消费`（不同的业务消息发送到不同的专用的队列）。

![image-20201101194140777](RabbitMQ 高级.assets/image-20201101194140777.png)

------

