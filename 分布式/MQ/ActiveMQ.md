## JMS 基本概念


java 消息服务（Java Message Service）是 java 平台中关于面向消息中间件的 API，用于在两个应用程序之间，或者分布式系统中发布消息，进行异步通信。 JMS 是一个与具体平台无关的 API，绝大多数 MOM（Message Oriented Middleware，面向消息中间件）提供商都对 JMS 提供了支持。 **具体开源的 JMS 供应商** JbossMQ(jboss4)、jboss messagling(jboss5)、joram、ubermg、mantamg、openims

## MOM

面向消息的中间件，使用消息传送提供者来协调消息传输操作。 MOM 需要提供 API 和管理工具。 客户端调用 api， 把消息发送到消息传送提供者指定的目的地。在消息发送之后，客户端会技术执行其他的工作。并且在接收方收到这个消息确认之前。提供者一直保留该消息。

## JMS 模型


![img](https://img-blog.csdnimg.cn/20200608114943731.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

## 消息传递域

**点对点(p2p)**


1. 每个消息只能有一个消费者 
2. 消息的生产者和消费者之间没有时间上的相关性。无论消费者在生产者发送消息的时候是否处于运行状态，都可以提取消息 ![img](https://img-blog.csdnimg.cn/20200608133307617.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

```java
public class JmsReceiver {
    public static void main(String[] args) {
        ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("tcp://192.168.174.128:61616");
        Connection connection = null;
        try {
            // 创建连接
            connection = connectionFactory.createConnection();
            connection.start();

            Session session = connection.createSession(Boolean.TRUE, Session.AUTO_ACKNOWLEDGE);

            // 创建队列（如果队列已经存在则不会创建，first-queue是队列名称）
            // destination 表示目的地
            Destination destination = session.createQueue("first-queue");
            // 创建消息消费者
            MessageConsumer consumer = session.createConsumer(destination);
 			// 配置非阻塞模式
            /*consumer.setMessageListener(new MessageListener() {
                @Override
                public void onMessage(Message message) {

                }
            });*/
            TextMessage message = (TextMessage) consumer.receive();
            System.out.println(message.getText());
			System.out.println("key的属性：" + message.getStringProperty("key"));
            session.commit();
            session.close();
        } catch (JMSException e) {
            e.printStackTrace();
        } finally {
            if (connection != null) {
                try {
                    connection.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
public class JmsSender {
    public static void main(String[] args) {
        ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("tcp://192.168.174.128:61616");
        Connection connection = null;
        try {
            // 创建连接
            connection = connectionFactory.createConnection();
            connection.start();

            Session session = connection.createSession(Boolean.TRUE, Session.AUTO_ACKNOWLEDGE);

            // 创建队列（如果队列已经存在则不会创建，first-queue是队列名称）
            // destination 表示目的地
            Destination destination = session.createQueue("first-queue");
            // 创建消息发送者
            MessageProducer producer = session.createProducer(destination);
            TextMessage message = session.createTextMessage("hello，晓晓！我是Spring");
            message.setStringProperty("key", "value");
            producer.send(message);

            session.commit();
            session.close();
        } catch (JMSException e) {
            e.printStackTrace();
        } finally {
            if (connection != null) {
                try {
                    connection.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

**发布订阅（pub/sub）**


1. 每个消息可以有多个消费者 
2. 消息的生产者和消费者之间存在时间上的相关性，订阅一个主题的消费者只能消费自它订阅之后发布的消息。 ![img](https://img-blog.csdnimg.cn/20200608133752869.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

## API

ConnectionFactory 连接工厂 Connection 封装客户端与JMS provider之间的一个虚拟的连接 Session 生产和消费消息的一个单线程上下文； 用于创建producer、consumer、message、queue… Destination 消息发送或者消息接收的目的地 MessageProducer/consumer 消息生产者/消费者

### 消息组成


**消息头** 包含消息的识别信息和路由信息 **消息体** TextMessage MapMessage BytesMessage StreamMessage 输入输出流 ObjectMessage 可序列化对象 **属性** ![img](https://img-blog.csdnimg.cn/20200608134745542.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)


JMS消息之后被确认后，才会认为是被成功消费。消息的消费包含三个阶段： 客户端接收消息、客户端处理消息、消息被确认。

## 事务性会话

```java
Session session = connection.createSession(Boolean.TRUE, Session.AUTO_ACKNOWLEDGE);
```

如上述代码，设置为true的时候，消息会在session.commit以后自动签收

## 非事务性会话

```java
Session session = connection.createSession(Boolean.FALSE, Session.AUTO_ACKNOWLEDGE);
```

在该模式下，消息何时被确认取决于创建会话时的应答模式： AUTO_ACKNOWLEDGE 当客户端成功从recive方法返回以后，或者[MessageListener.onMessage] 方法成功返回以后，会话会自动确认该消息 CLIENT_ACKNOWLEDGE 客户端通过调用消息的textMessage.acknowledge();确认消息。在这种模式中，如果一个消息消费者消费一共是10个消息，那么消费了5个消息，然后在第5个消息通过textMessage.acknowledge()，那么之前的所有消息都会被消确认 DUPS_OK_ACKNOWLEDGE 延迟确认

### acknowledge主动执行ack确认消息

```java
public class JmsUnCommitSender {
    public static void main(String[] args) {
        ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("tcp://192.168.174.128:61616");
        Connection connection = null;
        try {
            connection = connectionFactory.createConnection();
            connection.start();
            Session session = connection.createSession(Boolean.FALSE, Session.CLIENT_ACKNOWLEDGE);
            Destination destination = session.createQueue("first-queue");

            MessageProducer producer = session.createProducer(destination);
            for (int i = 0; i < 5; i++) {
                TextMessage message = session.createTextMessage("hello，晓晓！我是Spring"+ i);
                producer.send(message);
            }

            session.close();
        } catch (JMSException e) {
            e.printStackTrace();
        } finally {
            if (connection != null) {
                try {
                    connection.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
public class JmsUnCommitReceiver {
    public static void main(String[] args) {
        ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("tcp://192.168.174.128:61616");
        Connection connection = null;
        try {
            connection = connectionFactory.createConnection();
            connection.start();
            Session session = connection.createSession(Boolean.FALSE, Session.AUTO_ACKNOWLEDGE);
            Destination destination = session.createQueue("first-queue");

            MessageConsumer consumer = session.createConsumer(destination);
            for (int i = 0; i < 5; i++) {
                TextMessage message = (TextMessage) consumer.receive();
                if (i == 3) {
                    message.acknowledge();
                }
                System.out.println(message.getText());
            }

            session.close();
        } catch (JMSException e) {
            e.printStackTrace();
        } finally {
            if (connection != null) {
                try {
                    connection.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

message.acknowledge();是一个会回调方法：

```java
public void acknowledge() throws JMSException {
    if (acknowledgeCallback != null) {
        try {
            acknowledgeCallback.execute();
        } catch (JMSException e) {
            throw e;
        } catch (Throwable e) {
            throw JMSExceptionSupport.create(e);
        }
    }
}
```

找到设置acknowledgeCallback的地方，org.apache.activemq.command.ActiveMQMessage$setAcknowledgeCallback(Callback acknowledgeCallback)方法。 org.apache.activemq.ActiveMQMessageConsumer$createActiveMQMessage调用了此方法：

```java
private ActiveMQMessage createActiveMQMessage(final MessageDispatch md) throws JMSException {
    ...
    if (session.isClientAcknowledge()) {
        m.setAcknowledgeCallback(new Callback() {
            @Override
            public void execute() throws Exception {
                checkClosed();
                session.checkClosed();
                session.acknowledge();
            }
        });
    } 
	...
}
```

org.apache.activemq.ActiveMQSession$acknowledge()：

```java
protected final CopyOnWriteArrayList<ActiveMQMessageConsumer> consumers = new CopyOnWriteArrayList<ActiveMQMessageConsumer>();
    
public void acknowledge() throws JMSException {
    for (Iterator<ActiveMQMessageConsumer> iter = consumers.iterator(); iter.hasNext();) {
        ActiveMQMessageConsumer c = iter.next();
        c.acknowledge();
    }
}
```


![img](https://img-blog.csdnimg.cn/20200608161142857.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) consumers.deliveredMessages的LinkedList表示已经被consumer接收但未确认的消息。

## 本地事务

在一个JMS客户端，可以使用本地事务来组合消息的发送和接收。JMS Session 接口提供了commit和rollback方法。JMS Provider会缓存每个生产者当前生产的所有消息，直到commit或者rollback，commit操作将会导致事务中所有的消息被持久存储；rollback意味着JMS Provider将会清除此事务下所有的消息记录。在事务未提交之前，消息是不会被持久化存储的，也不会被消费者消费事务提交意味着生产的所有消息都被发送。消费的所有消息都被确认； 事务回滚意味着生产的所有消息被销毁，消费的所有消息被恢复，也就是下次仍然能够接收到发送端的消息，除非消息已经过期了。

## JMS （pub/sub）模型

1. 订阅可以分为非持久订阅和持久订阅 
2. 当所有的消息必须接收的时候，则需要用到持久订阅。反之，则用非持久订阅

## JMS （P2P）模型

1. 如果session关闭时，有一些消息已经收到，但还没有被签收，那么当消费者下次连接到相同的队列时，消息还会被签收 
2. 如果用户在receive方法中设定了消息选择条件，那么不符合条件的消息会留在队列中不会被接收 
3. 队列可以长久保存消息直到消息被消费者签收。消费者不需要担心因为消息丢失而时刻与jms provider保持连接状态

## Broker

```java
public class DefineBrokerServer {
    public static void main(String[] args) {
        BrokerService brokerService = new BrokerService();
        try {
            brokerService.setUseJmx(true);
            brokerService.addConnector("tcp://localhost:61616");
            brokerService.start();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## 消息的发送策略

### 持久化消息

默认情况下，生产者发送的消息是持久化的。消息发送到broker以后，producer会等待broker对这条消息的处理情况的反馈可以设置消息发送端发送持久化消息的异步方式： connectionFactory.setUseAsyncSend(true); 回执窗口大小设置： connectionFactory.setProducerWindowSize();

### 非持久化消息

textMessage.setJMSDeliveryMode(DeliveryMode.NON_PERSISTENCE); 非持久化消息模式下，默认就是异步发送过程，如果需要对非持久化消息的每次发送的消息都获得broker的回执的话： connectionFactory.setAlwaysSyncSend();

如果是在非事务模型下，使用持久化发送策略，那么该发送方式为同步模型。

### consumer获取消息

默认情况下，mq服务器（broker）采用异步方式向客户端主动推送消息(push)。也就是说broker在向某个消费者会话推送消息后，不会等待消费者响应消息，直到消费者处理完消息以后，主动向broker返回处理结果。 prefetchsize 预取消息数量，broker端一旦有消息，就主动按照默认设置的规则推送给当前活动的消费者。 每次推送都有一定的数量限制，而这个数量就是prefetchSize：

假如prefetchSize=0，此时对于consumer来说，就是一个pull模式。

```java
Destination destination = session.createQueue("first-queue?customer.prefetchSize=0");
```

## 消息确认

ACK_TYPE 消费端和broker交换ack指令的时候，还需要告知 broker ACK_TYPE。 ACK_TYPE 表示确认指令的类型，broker可以根据不同的ACK_TYPE去针对当前消息做不同的应对策略。 REDELIVERED_ACK_TYPE (broker会重新发送该消息) 重发侧策略 DELIVERED_ACK_TYPE 消息已经接收，但是尚未处理结束 STANDARD_ACK_TYPE 表示消息处理成功


ActiveMQ 是 Apache 开源基金会研发的消息中间件，是完全支持 JMS1.1 和 J2EE1.4 规范的 JMS provider 实现。 ActiveMQ 主要应用在分布式系统架构中，帮助构建高可用、高性能、可伸缩的企业级面向消息服务的系统。

## 应用场景




![img](https://img-blog.csdnimg.cn/20200608112445534.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) ![img](https://img-blog.csdnimg.cn/20200608112502278.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) ![img](https://img-blog.csdnimg.cn/20200608112516235.png)

## 安装 ActiveMQ

1. 下载 activeMQ 安装包 
2. tar -zxvf **.tar.gz 
3. sh bin/activemq start 启动activeMQ服务 
4. 默认访问地址 http://localhost:8161/，默认账号密码 admin

## ActiveMQ 结合 Spring 开发

1. Spring提供了对JMS的支持，需要添加Spring 支持JMS的包：

```java
<dependency>
        <groupId>org.apache.activemq</groupId>
        <artifactId>activemq-all</artifactId>
        <version>5.15.0</version>
      </dependency>
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jms</artifactId>
        <version>4.3.10.RELEASE</version>
      </dependency>
      <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-pool2</artifactId>
        <version>2.4.2</version>
      </dependency>
```

1. 配置 spring 文件：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="connectionFactory" class="org.apache.activemq.pool.PooledConnectionFactory" destroy-method="stop">
        <property name="connectionFactory">
            <bean class="org.apache.activemq.ActiveMQConnectionFactory">
                <property name="brokerURL">
                    <value>tcp://192.168.174.128:61616</value>
                </property>
            </bean>
        </property>
        <property name="maxConnections" value="50"/>
    </bean>

    <bean id="destination" class="org.apache.activemq.command.ActiveMQQueue">
        <constructor-arg index="0" value="spring-queue"/>
    </bean>

    <!--<bean id="destinationTopic" class="org.apache.activemq.command.ActiveMQTopic">
        <constructor-arg index="0" value="spring-topic"/>
    </bean>-->

    <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
        <property name="connectionFactory" ref="connectionFactory"/>
        <property name="defaultDestination" ref="destination"/>
        <property name="messageConverter">
            <bean class="org.springframework.jms.support.converter.SimpleMessageConverter"/>
        </property>
    </bean>
</beans>
```

1. 编写发送端代码

```java
public class SpringJmsSender {

    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:META-INF/spring/service-jms.xml");
        JmsTemplate jmsTemplate = (JmsTemplate) context.getBean("jmsTemplate");

        jmsTemplate.send(session -> session.createTextMessage("Hello,晓晓"));
    }

}
```

1. 配置接收端 spring 文件，编写接收端代码

```java
public class SpringJmsReceiver {

    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:META-INF/spring/service-jms.xml");
        JmsTemplate jmsTemplate = (JmsTemplate) context.getBean("jmsTemplate");

        String msg = (String) jmsTemplate.receiveAndConvert();
        System.out.println(msg);
    }

}
```

### Spring的发布订阅模式配置

```java
<bean id="destinationTopic" class="org.apache.activemq.command.ActiveMQTopic">
    <constructor-arg index="0" value="spring-topic"/>
</bean>
<bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
    <property name="connectionFactory" ref="connectionFactory"/>
    <property name="defaultDestination" ref="destinationTopic"/>
    <property name="messageConverter">
        <bean class="org.springframework.jms.support.converter.SimpleMessageConverter"/>
    </property>
</bean>
```

### 以事件通知方式来配置消费者

1. 更改消费端的配置

```java
<bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
    <property name="connectionFactory" ref="connectionFactory"/>
    <property name="destination" ref="destination"/>
    <property name="messageListener" ref="messageListener"/>
</bean>
<bean id="messageListener" class="com.spring.dubbo.jms.SpringJmsListener"/>
```

1. 增加监听类，启动 spring 容器

```java
public class SpringJmsListener implements MessageListener {
    @Override
    public void onMessage(Message message) {
        try {
            System.out.println(((TextMessage) message).getText());
        } catch (JMSException e) {
            e.printStackTrace();
        }
    }
}
public class SpringJmsReceiver {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:META-INF/spring/service-jms.xml");
        try {
            System.in.read();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## ActiveMQ 支持的传输协议

client 端和 broker 端的通讯协议：TCP、UDP 、NIO、SSL、Http（s）、vm

### 配置传输协议

修改 activemq.xml：

```java
<!-- 添加 nio 配置 -->
<transportConnector name="nio" uri="nio://0.0.0.0:61618?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
```

重启 ActiveMQ

```java
sh activemq stop
sh activemq start
```

## ActiveMQ 持久化存储


![img](https://img-blog.csdnimg.cn/20200608171104811.png)

### 默认的存储方式 kahaDB

activemq.xml 中的配置：

```java
<persistenceAdapter>
    <kahaDB directory="${activemq.data}/kahadb"/>
    <!-- /usr/local/apache-activemq-5.15.13/data/kahadb -->
</persistenceAdapter>
```

### 基于文件的存储方式 AMQ

写入速度很快，容易恢复，文件默认大小是32M AmqPersistenceAdapter在v5.8.0中被弃用。

### 基于数据库的存储 JDBC

ACTIVEMQ_ACKS 存储持久订阅的信息 ACTIVEMQ_LOCK 锁表（用来做集群的时候，实现master选举的表） ACTIVEMQ_MSGS 消息表

修改配置：

```java
<persistenceAdapter>
    <!--<kahaDB directory="${activemq.data}/kahadb"/>-->
    <jdbcPersistenceAdapter dataSource="#mysqlDataSource" createTablesOnStartup="true"/>
</persistenceAdapter>
<bean id="mysqlDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/activemq"/>
    <property name="username" value="root"/>
    <property name="password" value="root"/>
</bean>
```

将jar包添加到 /usr/local/apache-activemq-5.15.13/lib 文件夹： commons-dbcp-1.4.jar commons-pool-1.6.jar mysql-connector-java-5.1.35.jar

#### JDBC Message store with activeMQ journal

1. 引入了快速缓存机制，缓存到Log文件中 
2. 性能会比jdbc store要好 
3. JDBC Message store with activeMQ journal 不能应用于master/slave模式 
4. Memory 基于内存的存储

### LevelDB

5.8以后引入的持久化策略。通常用于集群配置

## ActiveMQ 的网络连接

activeMQ如果要实现扩展性和高可用性的要求的话，就需要用用到网络连接模式。

### NetworkConnector


主要用来配置broker与broker之间的通信连接 ![img](https://img-blog.csdnimg.cn/20200608202752252.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) 如上图所示，服务器S1和S2通过NewworkConnector相连，则生产者P1发送消息，消费者C3和C4都可以接收到，而生产者P3发送的消息，消费者C1和C2同样也可以接收到。

#### 静态网络连接

```java
<networkConnectors>
    <networkConnector uri="static://(tcp://192.168.1.127:61616,tcp://192.168.1.128:61616)"/>
</networkConnectors>
```


![img](https://img-blog.csdnimg.cn/20200608204912461.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) 两个Brokers通过一个staic的协议来进行网络连接。一个Consumer连接到BrokerB的一个地址上，当Producer在BrokerA上以相同的地址发送消息是，此时消息会被转移到BrokerB上，也就是说BrokerA会转发消息到BrokerB上。

##### 丢失的消息

一些consumer连接到broker1、消费broker2上的消息。消息先被broker1从broker2消费掉，然后转发给这些consumers。假设，转发消息的时候broker1重启了，这些consumers发现brokers1连接失败，通过failover连接到broker2.但是因为有一部分没有消费的消息被broker2已经分发到broker1上去了，这些消息就好像消失了。除非有消费者重新连接到broker1上来消费。 从5.6版本开始，在destinationPolicy上新增了一个选项replayWhenNoConsumers属性，这个属性可以用来解决当broker1上有需要转发的消息但是没有消费者时，把消息回流到它原始的broker。同时把enableAudit设置为false，为了防止消息回流后被当作重复消息而不被分发通过如下配置，在activeMQ.xml中。 分别在两台服务器都配置。即可完成消息回流处理。

```java
<policyEntry queue=">" enableAudit="false">
    <networkBridgeFilterFactory>
        <conditionalNetworkBridgeFilterFactory replayWhenNoConsumers="true"/>
    </networkBridgeFilterFactory>
</policyEntry>
```

#### 动态网络连接

multicast networkConnector是一个高性能方案，并不是一个高可用方案。

##### 通过zookeeper+activemq实现高可用方案

1. 修改 activemq.xml 配置

```java
<persistenceAdapter>
    <!--<kahaDB directory="${activemq.data}/kahadb"/>-->
    <replicatedLevelDB directory="${activemq.data}/levelDB" replicas="2"
                       bind="tcp://0.0.0.0:61615" zkAddress="192.168.11.140:2181,192.168.11.137:2181,192.168.11.128:2181"
                       hostname="192.168.11.140" zkPath="/activemq/leveldb"/>
</persistenceAdapter>
```

1. 启动zookeeper服务器 
2. 启动activeMQ

**参数的意思** directory levelDB数据文件存储的位置 replicas 计算公式（replicas/2）+1 ， 当replicas的值为2的时候， 最终的结果是2. 表示集群中至少有2台是启动的 bind 用来负责slave和master的数据同步的端口和ip zkAddress 表示zk的服务端地址 hostname 本机ip

##### jdbc存储的主从方案

基于LOCK锁表的操作来实现master/slave

##### 基于共享文件系统的主从方案

挂载网络磁盘，将数据文件保存到指定磁盘上即可完成master/slave模式

##### 高可用+高性能方案


![img](https://img-blog.csdnimg.cn/20200609120401762.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

#### 容错的链接

```java
// randomize=false，关闭随机
ConnectionFactory connectionFactory = new ActiveMQConnectionFactory(
                "failover:(tcp://192.168.174.128:61616,tcp://192.168.174.129:61616)?randomize=false");
```

## ActiveMQ监控

ActiveMQ自带的管理界面的功能十分简单，只能查看ActiveMQ当前的Queue和Topics等简单信息，不能监控ActiveMQ自身运行的JMX信息等

### hawtio

HawtIO 是一个新的可插入式 HTML5 面板，设计用来监控 ActiveMQ, Camel等系统；ActiveMQ在5.9.0版本曾将hawtio嵌入自身的管理界面，但是由于对hawtio的引入产生了争议，在5.9.1版本中又将其移除，但是开发者可以通过配置，使用hawtio对ActiveMQ进行监控。本文介绍了通过两种配置方式，使用hawtio对ActiveMQ进行监控。

1. 从http://hawt.io/getstarted/index.html 下载hawtio的应用程序 
2. 下载好后拷贝到ActiveMQ安装目录的webapps目录下，改名为hawtio.war并解压到到hawtio目录下 
3. 编辑ActiveMQ安装目录下conf/jetty.xml文件,在第75行添加以下代码

```java
<bean class="org.eclipse.jetty.webapp.WebAppContext">        
        <property name="contextPath" value="/hawtio" />        
        <property name="resourceBase" value="${activemq.home}/webapps/hawtio" />        
        <property name="logUrlOnStart" value="true" />  
</bean>
```

1. 修改bin/env文件 需要注意的是-Dhawtio的三个设定必须放在ACTIVEMQ_OPTS设置的最前面(在内存参数设置之后),否则会出现验证无法通过的错误(另外,ACTIVEMQ_OPTS的设置语句不要回车换行)

```java
-Dhawtio.realm=activemq -Dhawtio.role=admins -Dhawtio.rolePrincipalClasses=org.apache.activemq.jaas.GroupPrincipal
```

1. 启动activeMQ服务。访问http://ip:8161/hawtio

