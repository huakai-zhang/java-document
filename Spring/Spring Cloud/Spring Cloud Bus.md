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