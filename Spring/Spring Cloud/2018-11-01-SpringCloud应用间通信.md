---
layout:  post
title:   SpringCloud应用间通信
date:   2018-11-01 17:20:55
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-# 分布式
-SpringCloud

---



## 应用间通信

HTTP：Spring Cloud RPC：Dubbo

Spring Cloud中服务间两种restful调用方式： 1，RestTemplate 2，Fegin

## RestTemplate的三种使用方式

1.第一种方式(直接使用restTemplate，url写死)

```java
RestTemplate restTemplate = new RestTemplate();
        String response = restTemplate.getForObject("http://localhost:8080/msg", String.class);
```

2.第二种方式(利用loadBalancerClient通过应用名获取url，然后使用restTemplate)

```java
@Autowired
    private LoadBalancerClient loadBalancerClient;

        ServiceInstance serviceInstance = loadBalancerClient.choose("PRODUCT");
        String url = String.format("http://%s:%s", serviceInstance.getHost(), serviceInstance.getPort() + "/msg");
        RestTemplate restTemplate = new RestTemplate();
        String response = restTemplate.getForObject(url, String.class);
```

3.第三种方式(需要将restTemplate作为一个bean，利用LoadBalanced，可在restTemplate里使用应用名字)

```java
@Component
public class RestTemplateConfig {
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

```java
@Autowired
    private RestTemplate restTemplate;
    
    String response = restTemplate.getForObject("http://PRODUCT/msg", String.class);
```

RestTemplate默认负载均衡策略为轮询， 自定义负载均衡策略配置（修改为随机）：

```java
PRODUCT:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```

## 客户端负载均衡器：Ribbon

RestTemplate，Feign，Zuul 服务发现 服务选择规则 服务监听 ServerList（通过ServerList获取所有的可用服务列表），ServerListFilter（根据ServerListFilter过滤掉一部分服务），IRule（通过IRule选择一个实例作为最终目标结果）

## Feign的使用

声明式REST客户端（伪RPC） 采用了基于接口的注解 1.引入依赖

```java
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```

2.在启动类上加注解@EnableFeignClients 3.定义好需要调用的接口

```java
@FeignClient(name = "product")
public interface ProductClient {

    @PostMapping("/product/listForOrder")
    List<ProductInfoOutput> listForOrder(@RequestBody List<String> productIdList);

    @PostMapping("/product/decreaseStock")
    void decreaseStock(@RequestBody List<DecreaseStockInput> decreaseStockInputList);
}
```

4.调用

```java
@Autowired
    private ProductClient productClient;

       //查询商品信息(调用商品服务)
        List<ProductInfoOutput> productInfoList = productClient.listForOrder(productIdList);
```

## 消息队列

#### 消息中间件的选择

RabbitMQ，Kafka，ActiveMQ

## RabbitMQ

#### 安装


1.进入 [RabbitMQ](http://www.rabbitmq.com/download.html)下载页面 2.使用Docker镜像 ![img](https://img-blog.csdnimg.cn/20181101170334361.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

```java
docker run -d --hostname my-rabbit -p 5672:5672 -p 15672:15672 rabbitmq:3.7.3-management
```

默认帐号：guest，密码：guest

## 微服务和容器

从系统环境开始，自底至上打包应用 轻量级，对资源的有效隔离和管理 可复用，版本化

