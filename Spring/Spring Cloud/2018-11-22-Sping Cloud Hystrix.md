---
layout:  post
title:   Sping Cloud Hystrix
date:   2018-11-22 14:04:37
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-# 分布式

---



## Spring Cloud Hystrix

防雪崩利器 基于Netflix对应的Hystrix 具备的功能： 1.服务降级 2.依赖隔离 3.服务熔断 4.监控（Hystrix Dashboard）

## 服务降级

优先核心服务，非核心服务不可用或弱可用 通过HystrixCommand注解指定 fallbackMethod（回退函数）中具体实现降级逻辑

#### 使用RestTemplate的简单实例：

1.引入依赖

```java
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix</artifactId>
        </dependency>
        <dependency>
            <groupId>com.netflix.hystrix</groupId>
            <artifactId>hystrix-javanica</artifactId>
            <version>RELEASE</version>
        </dependency>
```

2.修改启动类配置

```java
@SpringCloudApplication
/*@SpringBootApplication
@EnableDiscoveryClient
// hystrix需添加
@EnableCircuitBreaker*/
```

3.Controller

```java
@RestController
// 设置默认返回方法
@DefaultProperties(defaultFallback = "defaultFallback")
public class HystrixController {

    @HystrixCommand(commandProperties = {
            // 设置超时时长
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "3000")
    })//(fallbackMethod = "fallback")
    @GetMapping("/getProductInfoList")
    public String getProductInfoList() {
        RestTemplate restTemplate = new RestTemplate();
        return restTemplate.postForObject("http://127.0.0.1:8005/product/listForOrder",
                Arrays.asList("1530607189075321022"),
                String.class);
    }

    private String fallback() {
        return "太拥挤了，请稍候再试~~";
    }
    private String defaultFallback() {
        return "默认：太拥挤了，请稍候再试~~";
    }
}
```

#### 配置文件中配置超时时间

```java
hystrix:
  command:
  	 # 默认超时时间
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 1000
    # 单个方法自定义超时时间，名称为controller方法名
    getProductInfoList:
      execution:
              isolation:
                thread:
                  timeoutInMilliseconds: 3000
```

#### 依赖隔离

线程池隔离：会为每个HystrixCommand设置一个独立的线程池，这样在一个HystrixCommand包装下的依赖服务出现延迟过高的情况，也只是对该依赖服务的调用产生影响，并不会拖慢其他服务。 Hystric自动实现了依赖隔离。

#### Feign-Hystric

1.配置：

```java
feign:
  hystrix:
    enabled: true
```

2.服务端

```java
@FeignClient(name = "product", fallback = ProductClient.ProductClientFallback.class)
public interface ProductClient {

    @PostMapping("/product/listForOrder")
    List<ProductInfoOutput> listForOrder(@RequestBody List<String> productIdList);

    @Component
    static class ProductClientFallback implements ProductClient {

        @Override
        public List<ProductInfoOutput> listForOrder(List<String> productIdList) {
            return null;
        }
    }
}
```

3.调用方启动类配置

```java
@ComponentScan(basePackages =  "com.imooc")
```

## 服务熔断


![img](https://img-blog.csdnimg.cn/20181121172550455.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

Circuit Breaker:断路器

断路器模式设计状态机：3种模式closed，open，half open 调用失败累计达到一定的阈值或者一定的比例就会启动熔断机制，open是容器打开状态，此时对服务都直接返回错误，但是会设置一个时钟选项，默认的到达这个时间之后，就会进入半熔断状态，允许定量的服务请求。如果调用都成功，或者达到一定的比例，则会关闭熔断器，否则再次进入open。

```java
@HystrixCommand(commandProperties = {
    @HystrixProperty(name = "circuitBreaker.enabled", value = "true"), // 设置熔断
    @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),
    @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "10000"),
    @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "60")
})
```

circuitBreaker.sleepWindowInMilliseconds：休眠时间窗（10000毫秒），休眠时间结束之后，会将断路器设置为half open，尝试熔断请求命令，如果失败，会重新进入熔断状态，休眠时间窗重新计时。如果成功，则关闭熔断器。 circuitBreaker.requestVolumeThreshold：滚动时间窗口中，断路器的最小请求数（10次） circuitBreaker.errorThresholdPercentage：滚动时间窗口中请求超过这个比例，会进入熔断状态（60%，也就是10次中的7次）

## Hystrix Dashboard

1.引入依赖

```java
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-netflix-hystrix-dashboard</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

2.修改启动类

```java
@EnableHystrixDashboard
```

如果出现Failed opening connection to  [http://localhost:8091/hystrix.stream?delay=100](http://localhost:8091/hystrix.stream?delay=100) : 404 : HTTP/1.1 404 错误，需要在主类添加方法：

```java
@Bean
	public ServletRegistrationBean getServlet() {
		HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
		ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
		registrationBean.setLoadOnStartup(1);
		registrationBean.addUrlMappings("/actuator/hystrix.stream");
		registrationBean.setName("HystrixMetricsStreamServlet");
		return registrationBean;
	}
```

3.其他版本SpringBoot可能需要配置yml文件，如果不配置会出现上诉404错误

```java
management:
	context-path: /
```




![img](https://img-blog.csdnimg.cn/20181122135031669.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) ![img](https://img-blog.csdnimg.cn/20181122135023266.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) ![img](https://img-blog.csdnimg.cn/20181122135536371.png)

