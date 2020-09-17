---
layout:  post
title:   spring cloud微服务
date:   2018-08-22 16:13:35
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-# 分布式

---



## SpringCloud

Eureka：Eureka Server，Eureka Client，Eureka高可用，服务发现机制 Config：Config Server，Config Client，Spring Cloud Bus（结合RabibtMQ）自动刷新 Ribbon：RestTemplate，Feign Zuul：动态路由，校验 Hystrix：Hystrix Dashboar，熔断机制

## 微服务的提出

James Lewis &amp; Martin Fowler，2014年3月25日的《Microservices》。 微服务是一种架构风格。

#### 原文中的重要部分

一系列微小的服务共同组成 跑在自己的进程里 每个服务为独立的业务开发 独立部署 分布式的管理


单一应用架构-》垂直应用架构-》分布式服务架构-》流动计算架构 ![img](https://img-blog.csdn.net/20180821143935988?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 分布式定义

一个项目既可以是单体架构也可以是分布式（容易产生误区，单体架构就不是分布式的）。 旨在支持应用程序和服务的开发，可以利用物理架构由多个自治的处理元素，不共享主内存，但通过网络发送消息合作。 微服务必然是分布式的。

## 微服务架构的基础框架/组件


服务注册发现 服务网关（ServiceGateway） 后端通用服务（中间层服务Middle Tier Service） 前端服务（边缘服务Edge Service） ![img](https://img-blog.csdn.net/20180821150147599?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 微服务两大门派

阿里系：Duboo,Zookeeper,SpringMvc or SpringBoot Spring Cloud栈：Spring Cloud,Netflix Eureka,SpringBoot

## Spring Cloud

Spring Cloud是一个开发工具集，含了多个子项目 1.利用Spring Bootd的开发便利 2.主要是基于对Netflix开源组件的进一步封装

Spring Cloud简化了分布式开发。

掌握如何使用，更要理解分布式、架构的特点

## Spring Cloud Eureka

基于Netflix Eureka做了二次封装 两个组件组成：Eureka Server注册中心，Eureka Client服务注册

## 注册中心Eureka Server



![img](https://img-blog.csdn.net/20180821164104446?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70) ![img](https://img-blog.csdn.net/20180821164233985?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

启动类添加@EnableEurekaServer注解，表示项目具有注册中心的功能。

application.yml:

```java
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
    # 自身不注册为实例
    register-with-eureka: false
spring:
  application:
    name: eureka
server:
  port: 8761
```


启动项目： ![img](https://img-blog.csdn.net/20180821165002896?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 服务注册


![img](https://img-blog.csdn.net/2018082215393054?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

#### 检查pom文件

确保下面版本与注册中心一致：

```java
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Finchley.SR1</spring-cloud.version>
    </properties>
```

同时检查是否引入spring-boot-starter-web依赖：

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

如果未引入该依赖会出现：Unregistering application unknown with eureka with status DOWN，项目会在启动后自动停止，在控制台没看到启动的端口。

#### 修改启动类

添加注解：@EnableDiscoveryClient

#### 修改application.yml

```java
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    hostname: clientName
spring:
  application:
    name: client
```


![img](https://img-blog.csdn.net/20180822154556389?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 取消注册中心自保模式

开发环境中，会出现： EMERGENCYI EUREKA MAY BE INCORRECTLY CL AIMING INSTANCES ARE UP WHEN THEY"RE NOT. RENEWALS ARE LESSER THANTHRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE.

原因在于：client上线率太低，注册中心不知道client是否还在线，但是会默认将client显示出来。这样会出现一个问题，在调用client时，不确定client是否还在线。

此时可在server的开发环境配置文件中添加：

```java
eureka:
  server:
    enable-self-preservation: false
```

已达到关闭自保模式。 此配置仅在开发环境中使用，生成环境不用。

关闭后会出现： THE SELF PRESERVATION MODE IS TURNED OFF.THIS MAY NOT PROTECT INSTANCE EXPIRY IN CASE OF NETWORK/OTHER PROBLEMS. 意思是：自保模式已经被你关闭，这可能无法保护网络/其他问题的情况下失效。

## Eureka Server高可用


![img](https://img-blog.csdn.net/20180824133704571?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

#### 配置3个Eureka服务端


使用复制功能复制3个服务器配置，分别配置端口为8761，8762，8763 ![img](https://img-blog.csdn.net/20180824134027155?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

#### 取消application.yml中的端口配置

```java
#server:
#  port: 8761
```

#### 依次启动Eureka Server

启动EurekaApplication1时，配置：defaultZone:  [http://localhost:8762/eureka/,http://localhost:8763/eureka/](http://localhost:8762/eureka/,http://localhost:8763/eureka/) 启动EurekaApplication2时，配置：defaultZone:  [http://localhost:8761/eureka/,http://localhost:8763/eureka/](http://localhost:8761/eureka/,http://localhost:8763/eureka/) 启动EurekaApplication3时，配置：defaultZone:  [http://localhost:8761/eureka/,http://localhost:8762/eureka/](http://localhost:8761/eureka/,http://localhost:8762/eureka/)

#### 启动client

配置defaultZone:  [http://localhost:8761/eureka/,http://localhost:8762/eureka/,http://localhost:8763/eureka/](http://localhost:8761/eureka/,http://localhost:8762/eureka/,http://localhost:8763/eureka/)

会发现三个服务端都注册上了client

## 总结

@EnableEurekaServer @EnableEurekaClient 心跳检测、健康检查、负载均衡等功能 Eureka的高可用，生产上建议至少两台以上 分布式系统中，服务注册中心是最重要的基础部分

## 分布式下服务注册的地位和原理


![img](https://img-blog.csdn.net/20180824155025752?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

客户端发现，注册中心：Eureka 服务端发现，代理：Nginx,Zookeeper

微服务的特点：异构 1.不同语言 2.不同类型的数据库

SpringCloud的服务调用方式： REST，eureka的服务端提供了较为完善的api，支持将非java语言实现的服务纳入到自己的服务治理体系中来，只需要其他语言自己实现eureka的客户端程序，比如Node.js实现了一套eureka-js-client。

