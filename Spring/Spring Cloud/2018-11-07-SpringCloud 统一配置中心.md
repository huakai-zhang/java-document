---
layout:  post
title:   SpringCloud 统一配置中心
date:   2018-11-07 09:45:57
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-# 分布式

---



## 统一配置中心



1.创建项目 ![img](https://img-blog.csdnimg.cn/20181102094335284.png) ![img](https://img-blog.csdnimg.cn/20181102094348603.png)

2.启动类添加注解@EnableDiscoveryClient和@EnableConfigServer


3.在码云或者GitHub创建一个git项目，并将需要使用统一配置的项目的配置文件放入git项目中 ![img](https://img-blog.csdnimg.cn/20181102095642540.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

4.复制git项目路径并修改配置文件

```java
spring:
  application:
    name: config
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/1z9zfcawsf/config-repo
          username: git帐号
          password: git密码
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

5.启动项目，访问http://localhost:8080/order-a.yml，配置文件内容就会被访问到 /{label}/{name}-{profiles}.yml label：分支 profiles:环境 order.yml，order-test.yml，order-dev.yml，访问order-a.yml找不到会默认访问order.yml

6.可添加配置spring.cloud.config.server.git.basedir: E:\data，表明将git项目下载到该文件，而不使用默认地址，增加安全性

## 使用统一配置中心的配置

1.添加依赖

```java
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-client</artifactId>
        </dependency>
```

2.修改application.yml为bootstrap.yml,并修改内容为：

```java
spring:
  application:
    name: order
  cloud:
    config:
      discovery:
        enabled: true
        service-id: CONFIG
      profile: dev
```

调用config的服务时同样使用的是负载均衡。

## eureka如果不使用默认8761端口

1.eureka使用8762端口，启动项目 2.修改config的配置注册到eureka服务中心，启动项目 3.但是此时order项目并未注册到eureka上去，是因为git服务器上没有修改配置文件。修改git上的order配置文件为8762。 4.访问http://localhost:8080/order-dev.yml，配置生效 5.启动order项目无法启动，而且访问的config也变成了http://localhost:8888 原因在于，首先order回去访问eureka，现在配置放在config中，访问不到，就会去访问默认的8761端口。 所以需要把git上关于eureka的配置挪到bootstrap.yml中

```java
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8762/eureka/
```


6.但配置仍未生效，原因在于 ![img](https://img-blog.csdnimg.cn/20181102173720403.png) 在获取order-test.yml的配置时，同时会拿到order.yml（默认）并将两个配置文件合并。 所以，无需将所有配置都写入到order-环境.yml配置文件中，可以将各个环境下配置文件的公有配置放到order.yml。

## Bus自动更新配置


![img](https://img-blog.csdnimg.cn/20181106152640210.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

1.修改spring boot和Spring cloud版本为2.0.0.BUILD-SNAPSHOT和Finchley.BUILD-SNAPSHOT 2.添加依赖

```java
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
```

3.修改配置

```java
# 将bus所有的接口暴漏出去（包括bus-refresh）
management:
  endpoints:
    web:
      exposure:
        include: "*"
```


4.启动项目RabbitMQ会多出一个队列 ![img](https://img-blog.csdnimg.cn/20181106154946257.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

5.order项目做同样操作，也会多出一个RabbitMQ队列 6.测试方法添加注解

```java
@RestController
@RequestMapping("/env")
@RefreshScope
public class EnvController {

    @Value("${env}")
    private String env;

    @GetMapping("/print")
    public String print() {
        return env;
    }
}
```

7.修改git上的配置文件，同时使用post方式请求http://localhost:8080/actuator/bus-refresh 访问http://localhost:8081/env/print，发现配置已改

#### RefreshScope

前缀方式：

```java
@Data
@Component
@ConfigurationProperties("girl")
@RefreshScope
public class GirlConfig {

    private String name;

    private Integer age;
}
```

```java
@RestController
public class GirlController {

    @Autowired
    private GirlConfig girlConfig;

    @GetMapping("/girl/print")
    public String print() {
        return "name:" + girlConfig.getName() + " age:" + girlConfig.getAge();
    }
}
```

git配置文件中添加：

```java
girl:
  name: xiaoxiao
  age: 19
```

#### 通过Git类型的网站设置push时自动调用bus-refresh接口


![img](https://img-blog.csdnimg.cn/20181106163935955.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

