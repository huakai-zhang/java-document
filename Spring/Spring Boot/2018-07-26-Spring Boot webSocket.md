---
layout:  post
title:   Spring Boot webSocket
date:   2018-07-26 13:40:11
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-# Spring Boot

---



## 客户端代码

```java
<script>
        var websocket = null;
        if('WebSocket' in window) {
            websocket = new WebSocket('ws://127.0.0.1：8080/sell/webSocket');
        } else {
            alert("该浏览器不支持websocket!")
        }

        websocket.onopen = function (event) {
            console.log("建立连接");
        }

        websocket.onclose = function (event) {
            console.log("连接关闭")
        }

        websocket.onmessage = function (event) {
            console.log("收到消息：" + event.data)
            //弹窗提醒, 播放音乐
            $('#myModal').modal('show');

            document.getElementById('notice').play();
        }

        websocket.error = function () {
            alert("websocket通信发生错误！");
        }

        window.onbeforeunload = function () {
            websocket.close();
        }
    </script>
```

## 后端代码

#### pom依赖

```java
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-websocket</artifactId>
		</dependency>
```

#### WebSocketConfig

此处需要注意，仅使用Application文件启动项目才使用此配置。如果使用本地tomcat启动项目，无需进行此步配置，否则会产生java.lang.IllegalStateException: Failed to register @ServerEndpoint class的错误。

```java
// 使用tomcat启动无需配置
@Component
public class WebSocketConfig {
    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }
}
```

#### WebSocket

```java
@Component
@ServerEndpoint("/webSocket")
@Slf4j
public class WebSocket {

    private Session session;

    private static CopyOnWriteArraySet<WebSocket> webSocketSet = new CopyOnWriteArraySet<>();

    @OnOpen
    public void onOpen(Session session) {
        this.session = session;
        webSocketSet.add(this);
        log.info("【websocket消息】有新的连接，总数：{}", webSocketSet.size());
    }

    @OnClose
    public void onClose() {
        webSocketSet.remove(this);
        log.info("【websocket消息】连接断开，总数：{}", webSocketSet.size());
    }

    @OnMessage
    public void onMessage(String message) {
        log.info("【websocket消息】收到客户端发来的消息：{}", message);
    }

    public void sendMessage(String message) {
        for (WebSocket webSocket : webSocketSet) {
            log.info("【websocket消息】广播消息，message={}", message);
            try {
                webSocket.session.getBasicRemote().sendText(message);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

}
```

#### 引用WebSocket

```java
@Service
@Slf4j
public class OrderServiceImpl implements OrderService {
    @Autowired
    private WebSocket webSocket;

    @Override
    public void create() {
        // 创建订单时发送websocket消息
        webSocket.sendMessage(orderDTO.getOrderId());
    }
}
```

