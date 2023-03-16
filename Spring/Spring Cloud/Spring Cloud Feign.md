# 声明式服务调用 Spring Cloud Feign

在实践过程中，我们会发现对 Ribbon 和 Hystrix 这两个框架的使用几乎是同时出现的。既然如此，那么是否有更高层次的封装来整合这两个基础工具以简化开发呢？

Spring Cloud Feign 就是这样一个工具。它基于Netflix Feign 实现，整合了 Spring Cloud Ribbon 与 Spring Cloud Hystrix，除了提供这两者的强大功能之外，它还提供了一种`声明式的Web服务客户端定义方式`。

在Spring Cloud Feign的实现下，我们只需创建一个接口并用注解的方式来配置它，即可完成对服务提供方的接口绑定，简化了在使用Spring CloudRibbon 时自行封装服务调用客户端的开发量。Spring Cloud Feign具备可插拔的注解支持，包括Feign注解和JAX-RS注解。同时，为了适应 Spring 的广大用户，它在 Netflix Feign 的基础上扩展了对Spring MVC的注解支持。

## 1 快速入门

创建 Spring Boot 基础项目，取名为 feign-consumer。

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableFeignClients
@EnableDiscoveryClient
public class FeignConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(FeignConsumerApplication.class, args);
    }
}
```

```java
// 定义好需要调用的接口
// 注意:这里服务名不区分大小写，所以使用hello-service和 HELLO-SERVICE都是可以的。
// @FeignClient 的类必须在启动类同一级目录下，如果不在，需要启动类添加 @EnableFeignClients(basePackages = "com.spring.helloservice.service")
@FeignClient("hello-service")
public interface HelloService {

    @RequestMapping("/hello")
    String hello();

}
```

```java
@RestController
public class ConsumerController {
    @Autowired
    private HelloService helloService;

    @GetMapping("/feign-consumer")
    public String helloConsumer() {
        return helloService.hello();
    }
}
```

```yml
server:
  port: 9001
spring:
  application:
    name: feign-consumer
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

发送几次GET请求到 http://localhost:9001/feign-consumer，可以得到如之前 Ribbon 实现时一样的效果，正确返回了“Hello World”。并且根据控制台的输出，我们可以看到Feign实现的消费者，依然是利用Ribbon维护了针对 HELLO-SERVICE 的服务列表信息，并且通过轮询实现了客户端负载均衡。而与Ribbon不同的是，通过Feign我们只需定义服务绑定接口，以声明式的方法，优雅而简单地实现了服务调用。

## 2 参数绑定

在开始介绍 Spring Cloud Feign 的参数绑定之前，我们先扩展一下服务提供方hello-service。增加下面这些接口定义，其中包含带有Request参数的请求、带有Header信息的请求、带有RequestBody 的请求以及请求响应体中是一个对象的请求。

```java
@GetMapping("/hello1")
public String hello(@RequestParam String name)   {
    return "Hello " + name;
}
@GetMapping("/hello2")
public User hello(@RequestParam String name, @RequestParam Integer age)   {
    return new User(name, age);
}
@PostMapping("/hello3")
public String hello(@RequestBody User user)   {
    return "Hello " + user.getName() + ", " + user.getAge();
}
```

`User 对象的定义必须要有User 的默认构造函数`。不然，Spring Cloud Feign 根据 JSON 字符串转换 User 对象时会抛出异常。

```java
@FeignClient("hello-service")
public interface HelloService {

    @RequestMapping("/hello")
    String hello();

    @GetMapping("/hello1")
    String hello(@RequestParam("name") String name);

    @GetMapping("/hello2")
    User hello(@RequestParam("name") String name, @RequestParam("age") Integer age);

    @PostMapping("/hello3")
    String hello(@RequestBody User user);
}
```

这里一定要注意，`在定义各参数绑定时，@RequestParam、@RequestHeader等可以指定参数名称的注解，它们的 value 千万不能少`。在 Spring MVC 程序中，这些注解会根据参数名来作为默认值，但是在Feign中绑定参数必须通过 value 属性来指明具体的参数名，不然`会抛出 IllegalstateException 异常，value属性不能为空`。

```java
@GetMapping("/feign-consumer2")
public String helloConsumer2() {
    StringBuilder sb = new StringBuilder();
    sb.append(helloService.hello()).append("\n");
    sb.append(helloService.hello("XiaoXiao")).append("\n");
    sb.append(helloService.hello("XiaoXiao", 18)).append("\n");
    sb.append(helloService.hello(new User("XiaoXiao", 18))).append("\n");
    return sb.toString();
}
// Hello World
// Hello XiaoXiao 
// User{name='XiaoXiao', age=18} 
// Hello XiaoXiao, 18
```

## 3 继承特性

使用继承特性，参考 spring-cloud-demo --> hello-service --> hello-service-api(refactor部分)

实际开发，参考 spring-cloud-demo --> hello-service --> hello-service-api(pro部分)

## 4 Ribbon 配置

### 4.1 全局配置

全局配置的方法非常简单，我们可以直接使用 `ribbon.<key>=<value>` 的方式来设置ribbon的各项默认参数。比如，修改默认的客户端调用超时时间:

```yml
ribbon:
  ribbonConnectTimeout: 500
  ReadTimeout: 2000
```

### 4.2 指定服务配置

在使用Spring Cloud Feign的时候，针对各个服务客户端进行个性化配置的方式与使用Spring Cloud Ribbon时的配置方式是一样的，都采用 `<client>.ribbon. key=value` 的格式进行设置。但是，这里就有一个疑问了，client 所指代的Ribbon客户端在哪里呢?

回想一下，在定义Feign客户端的时候，我们使用了@Feignclient注解。在初始化过程中，Spring Cloud Feign 会根据该注解的name属性或value属性指定的服务名，自动创建一个同名的Ribbon 客户端。也就是说，在之前的示例中,使用 @Feignclient (value="HELLO-SERVICE") 来创建Feign客户端的时候，同时也创建了一个名为 HELLO-SERVICE 的Ribbon客户端。既然如此，我们就可以使用@Feignclient注解中的name或value属性值来设置对应的Ribbon参数，比如:

```yml
HELLO-SERVICE:
  ribbon:
    ribbonConnectTimeout: 500
    ReadTimeout: 2000
    OkToRetryonAllOperations: true
    MaxAutoRetriesNextServer: 2
    MaxAutoRetries: 1
```

### 4.3 重试机制

在 Spring Cloud Feign 中默认实现了请求的重试机制，我们可以通过修改之前的实例做一些验证。

* 在 hello-service 应用的 /hello 接口实现中，增加一些随机延迟。

* 在 feign-consumer 应用中增加上文中提到的重试配置参数。其中，由于`HELLO-SERVICE.ribbon.MaxAutoRetries`设置为1，所以重试策略先尝试访问首选实例一次，失败后才更换实例访问，而更换实例访问的次数通过 `HELLO-SERVICE.ribbon.MaxAutoRetriesNextserver` 参数设置为2，所以会尝试更换两次实例进行重试。
* 最后，启动这些应用，并尝试访问几次 http://localhost:9001/feign-consumer 接口。当请求发生超时的时候，我们在hello-service 的控制台中可能会获得如下输出内容（由于sleepTime的随机性，并不一定每次相同)：

```markdown
sleepTime:2690
sleepTime:53
/hello, host:localhost, service_id:HELLO-SERVICE
/hello, host:localhost, service_id:HELLO-SERVICE
```

从控制台输出中，我们可以看到这次访问的第一次请求延迟时间为 2690 毫秒，由于超时时间设置为2000 毫秒，Feign客户端发起了重试，第二次请求的延迟为 53 秒，没有超时。Feign 客户端在进行服务调用时，虽然经历了一次失败，但是通过重试机制，最终还是获得了请求结果。所以，对于重试机制的实现，对于构建高可用的服务集群来说非常重要，而 Spring Cloud Feign也为其提供了足够的支持。
这里需要注意一点，Ribbon 的超时与 Hystrix 的超时是两个概念。为了让上述实现有效，我们需要让Hystrix 的超时时间大于Ribbon 的超时时间，否则 Hystrix命令超时后，该命令直接熔断，重试机制就没有任何意义了。

## 5 Hystrix 配置

### 5.1 全局配置

对于Hystrix的全局配置同Spring Cloud Ribbon 的全局配置一样,直接使用它的默认配置前缀hystrix.command.default就可以进行设置，比如设置全局的超时时间;

```properties
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds=5000
```

另外，在对Hystrix进行配置之前，我们需要确认`feign. hystrix.enabled` 参数没有被设置为false，否则该参数设置会关闭 Feign 客户端的Hystrix支持。而对于我们之前测试重试机制时，对于Hystrix 的超时时间控制除了可以使用上面的配置来增加熔断超时时间，也可以通过`feign.hystrix.enabled=false` 来`关闭 Hystrix` 功能，或者使用`hystrix.command.default.execution.timeout.enabled=false` 来`关闭熔断`功能。

### 5.2 部分客户端禁用 Hystrix

针对某个服务客户端关闭 Hystrix 支持时，需要通过使用 `@Scope("prototype")` 注解为指定的客户端配置 Feign.Builder 实例，详细实现步骤如下所示。

```java
@Configuration
public class DisableHystrixConfiguration {
    @Bean
    @Scope("prototype")
    public Feign.Builder feignBuilder() {
        return Feign.builder();
    }
}
@FeignClient(name="hello-service", configuration = DisableHystrixConfiguration.class)
public interface HelloService {...}
```

### 5.3 服务降级配置

Hystrix 提供的服务降级是服务容错的重要功能，由于Spring Cloud Feign在定义服务户端的时候与Spring Cloud Ribbon有很大差别，HystrixCommand定义被封装了起来，我们无法像之前介绍Spring Cloud Hystrix时，通过@HystrixCommand注解的 fallback参数样来指定具体的服务降级处理方法。

服务降级逻辑的实现只需要为 Feign 客户端的定义接口编写一个具体的接口实现类。比如为HelloService 接口实现一个服务降级类 HelloServiceFallback，其中每个重写方法的实现逻辑都可以用来定义相应的服务降级逻辑，具体如下:

```yml
# 开启 Feign 客户端的Hystrix支持
feign:
  hystrix:
    enabled: true
```

```java
@Component
public class HelloServiceFallback implements HelloService {
    @Override
    public String hello() {
        return "error";
    }
    @Override
    public String hello(String name) {
        return "error";
    }
    @Override
    public User hello(String name, Integer age) {
        return new User("未知", 0);
    }
    @Override
    public String hello(User user) {
        return "error";
    }
}
// 在服务绑定接口HelloService 中，通过@Feignclient注解的 fallback属性来指定对应的服务降级实现类。
@FeignClient(name="hello-service", fallback = HelloServiceFallback.class) 
public interface HelloService {...}
```

启动服务注册中心和feign- consumer,但是不启动hello-service 服务。发送 GET请求到http://localhost:9001/feign-consumer2，因为hello-service服务没有启动，会直接触发服务降级，并获得下面的输出内容:

```
error
error
name=未知,age=0
error
```

正如我们在HelloServiceFallback类中实现的内容，每一个服务接口的断路器实际就是实现类中的重写函数的实现。

------

