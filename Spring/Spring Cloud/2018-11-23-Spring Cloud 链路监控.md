---
layout:  post
title:   Spring Cloud 链路监控
date:   2018-11-23 11:42:50
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-# 分布式

---



## Spring Cloud Sleuth

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
```

```java
DEBUG [order,1fd82ebf55698e67,99904c1fe1a0165e,false]
```

order：application name — 应用的名称，也就是application.properties中的spring.application.name参数配置的属性。 1fd82ebf55698e67：traceId — 为一个请求分配的ID号，用来标识一条请求链路（一条链路里面只会包含一个traceId）。 99904c1fe1a0165e：spanId — 表示一个基本的工作单元，一个请求可以包含多个步骤，每个步骤都拥有自己的spanId。一个请求包含一个TraceId，多个SpanId。 false : export — 布尔类型。表示是否要将该信息输出到其他服务进行收集和展示。

## Zipkin

1.安装Zipkin

```java
docker run -d -p 9411:9411 openzipkin/zipkin
```

2.引入依赖

```java
<!--<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-sleuth</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-sleuth-zipkin</artifactId>
        </dependency>-->

        <!-- 包含sleuth和zipkin -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
        </dependency>
```

3.配置

```java
spring:
  #采样率
  sleuth:
    sampler:
      probability: 1.0
  zipkin:
    base-url: http://localhost:9411/
    #配置zipkin发送类型为web
    sender:
      type: web
```



zipkin.sender.type: web，此配置在不同的版本中配置不同，此处使用的是Spring Boot 2.0.0.BUILD-SNAPSHOT和Spring Cloud Finchley.BUILD-SNAPSHOT。如果不配置此项，后台Sleuth数据export状态为true，但是zipkin接受不到采集数据。 ![img](https://img-blog.csdnimg.cn/20181122162617311.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) ![img](https://img-blog.csdnimg.cn/20181122162648277.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

## 分布式追踪系统

核心步骤： 数据采集 数据存储 查询展示

#### OpenTracing规范

在数据采集过程中，由于不同系统的api并不兼容，这就导致如果希望切换追踪系统，往往会带来较大改动。因此就诞生了OpenTracing规范。

通过提供平台无关，商务无关的API，使得开发人员能够方便的添加或者更换追踪系统的实现。OpenTracing正在为全球的分布式追踪提供统一的概念和数据标准。

#### Annotation

事件类型 cs(Client Send) 客户端发起请求的时间 cr(Client Received)客户端收到处理完请求的时间 ss(Server Send)服务端处理完逻辑的时间 sr(Server Received)服务端收到调用端请求的时间

客户端调用时间=cr-cs 服务端处理时间=sr-ss

## ZipKin


![img](https://img-blog.csdnimg.cn/20181122164119548.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

几个关键概念： traceId 全局的跟踪id，是追踪的入口点 spanId 下一层的请求跟踪id，一个traceId可以包含一个以上的spanId parentId 上一层的请求跟踪id，用来将前后的请求串联起来

