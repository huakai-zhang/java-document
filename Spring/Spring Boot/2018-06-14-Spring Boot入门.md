---
layout:  post
title:   Spring Boot入门
date:   2018-06-14 10:27:41
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-# Spring Boot

---



## 第一个Spring Boot应用


IDEA—&gt;File—&gt;New—&gt;Project—&gt;Spring Initializr  ![img](https://img-blog.csdn.net/20180613164614425?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

```java
@SpringBootApplication
public class GirlApplication {

    public static void main(String[] args) {
        SpringApplication.run(GirlApplication.class, args);
    }
}
```


Run ‘GirlApplication ‘，访问8080端口  ![img](https://img-blog.csdn.net/20180613165138936?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

```java
@RestController
public class HelloController {

    @RequestMapping(value = "/hello", method = RequestMethod.GET)
    public String say() {
        return "Hello Spring Boot!";
    }
}
```

访问localhost:8080/hello，页面输出Hello Spring Boot!

#### 其他启动方式

1.命令行进入项目目录下，运行mvn spring-boot:run  2.mvn install把程序编译，进入target目录下，多出一个.jar文件，java -jar girl-0.0.1-SNAPSHOT.jar

## 项目属性配置

application.properties：

```java
server.port=8080
server.context-path=/girl
```

application.yml：

```java
server:
  port: 8080
  context-path: /girl
```

.yml文件格式：后面必须有一个空格，这两种配置保留一个即可。

#### 配置变量

```java
cupSize: B
age: 18
```

HelloController:

```java
@RestController
public class HelloController {

    @Value("${cupSize}")
    private String cupSize;

    @RequestMapping(value = "/hello", method = RequestMethod.GET)
    public String say() {
        return cupSize + " " + age;
    }
}
```

打印 B 18

#### 配置中使用配置

```java
content: "cupSize: ${cupSize}, age: ${age}"

 @Value("${content}")
    private String content;

    @RequestMapping(value = "/hello", method = RequestMethod.GET)
    public String say() {
        return content;
    }
```

打印 cupSize: B, age: 18

#### 多配置变量

```java
girl:
  cupSize: B
  age: 18
```

```java
/**
 * 定义Spring管理Bean
 * 获取前缀是girl的配置
 */
@Component
@ConfigurationProperties(prefix = "girl")
public class GirlProperties {
    private String cupSize;

    private Integer age;

    public String getCupSize() {
        return cupSize;
    }

    public void setCupSize(String cupSize) {
        this.cupSize = cupSize;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}

@RestController
public class HelloController {

    @Autowired
    private GirlProperties girlProperties;

    @RequestMapping(value = "/hello", method = RequestMethod.GET)
    public String say() {
        return girlProperties.getCupSize();
    }
}
```

## 开发环境和生产环境

application.yml只保留:

```java
spring:
  profiles:
    active: dev
```

application-dev.yml:

```java
server:
  port: 8080
  context-path: /girl
girl:
  cupSize: B
  age: 18
```

application-prod.yml:

```java
server:
  port: 8081
  context-path: /girl
girl:
  cupSize: F
  age: 18
```

可以使用其他方法启动：  java -jar girl-0.0.1-SNAPSHOT.jar –spring.profiles.active=prod

## Controller的使用

```java
@Controller
public class HelloController {

    @Autowired
    private GirlProperties girlProperties;

    @RequestMapping(value = "/hello", method = RequestMethod.GET)
    public String say() {
        return girlProperties.getCupSize();
    }
}
```

访问路径会出现异常，没有模版B。

```java
<!-- spring官方模版 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
```

新建resources/templates/index.html

```java
@Controller
public class HelloController {

    @Autowired
    private GirlProperties girlProperties;

    @RequestMapping(value = "/hello", method = RequestMethod.GET)
    public String say() {
        return "index";
    }
}
```

模版生效。类似于java web的jsp，不过使用的引擎是thymeleaf。

```java
@Controller
@ResponseBody
等同于
@RestController
```

#### 多路径访问

```java
@RequestMapping(value = {"/hello", "/hi"}, method = RequestMethod.GET)
    public String say() {
        return girlProperties.getCupSize();
    }
```

修改方法为RequestMethod.POST，使用postman访问

如果不写method参数，任何请求方式都可以访问。

#### 处理URL的参数

@PathVariable 获取url中的数据  @RequestParam 获取请求参数的值  @GetMapping 组合注解

```java
@RestController
@RequestMapping("/hello")
public class HelloController {

    @Autowired
    private GirlProperties girlProperties;

    @RequestMapping(value = "/say/{id}", method = RequestMethod.GET)
    public String say(@PathVariable("id") Integer id) {
        return "id: " + id;
    }
}
```

```java
@RestController
@RequestMapping("/hello")
public class HelloController {

    @Autowired
    private GirlProperties girlProperties;

    @RequestMapping(value = "/say", method = RequestMethod.GET)
    public String say(@RequestParam("id") Integer myId) {
        return "id: " + myId;
    }
}
```

```java
@RestController
@RequestMapping("/hello")
public class HelloController {

    @RequestMapping(value = "/say", method = RequestMethod.GET)
    public String say(@RequestParam(value = "id", required = false, defaultValue = "0") Integer myId) {
        return "id: " + myId;
    }
}
```

```java
@RestController
@RequestMapping("/hello")
public class HelloController {

    @GetMapping(value = "/say")
    public String say(@RequestParam(value = "id", required = false, defaultValue = "0") Integer myId) {
        return "id: " + myId;
    }
}
```

## 数据库操作

JPA(Java Persistence API)定义了一系列对象持久化的标准，目前实现这一规范的产品有Hibenate、TopLikn等。  Spring-Data-Jpa，就是Spring对Hibenate的整合。

```java
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
```

application.yml:

```java
spring:
  profiles:
    active: dev
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/dbgirl
    username: root
    password: root
  jpa:
    hibernate:
      # create在运行时自动创建表，每次都会建一个空的表，之前有这个表会先删掉
      # ddl-auto: create
      # 之前有这个表不会删掉，会保留着
      ddl-auto: update
    # 控制台打印sql
    show-sql: true
```

```java
@Entity
public class Girl {

    @Id
    @GeneratedValue
    private Integer id;  //自增

    private String cupSize;

    private Integer age;

    /**
     * 必须有一个无参构造器，不然数据库生成会报错
     */
    public Girl() {
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getCupSize() {
        return cupSize;
    }

    public void setCupSize(String cupSize) {
        this.cupSize = cupSize;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
```


![img](https://img-blog.csdn.net/20180614094712362?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

#### ddl-auto

create-drop在应用停下来时就会把表全部删掉  none什么都不做  validate会验证类里面的数据是否和表中一致，如果不一致会报错

#### RestFul API

```java
public interface GirlRepository extends JpaRepository<Girl, Integer> {

    /**
     * 通过年龄来查询，方法名一定按照这个格式来写
     * @param age
     * @return
     */
    List<Girl> findByAge(Integer age);

}

@RestController
public class GirlController {

    @Autowired
    private GirlRepository girlRepository;

    /**
     * 查询所有女生列表
     * @return
     */
    @GetMapping(value = "/girls")
    public List<Girl> girlList() {
        return girlRepository.findAll();
    }

    /**
     * 添加一个女生
     * @param cupSize
     * @param age
     * @return
     */
    @PostMapping(value = "/girls")
    public Girl girlAdd(@RequestParam("cupSize") String cupSize,
                          @RequestParam("age") Integer age) {
        Girl girl = new Girl();
        girl.setCupSize(cupSize);
        girl.setAge(age);

        return girlRepository.save(girl);
    }

    @GetMapping(value = "/girls/{id}")
    public Girl girlFindOne(@PathVariable("id") Integer id) {
        return girlRepository.findOne(id);
    }

    @PutMapping(value = "/girls/{id}")
    public Girl girlUpdate(@PathVariable("id") Integer id,
                           @RequestParam("cupSize") String cupSize,
                           @RequestParam("age") Integer age) {
        Girl girl = new Girl();
        girl.setId(id);
        girl.setCupSize(cupSize);
        girl.setAge(age);

        return girlRepository.save(girl);
    }

    @DeleteMapping(value = "/girls/{id}")
    public void girlDelete(@PathVariable("id") Integer id) {
        girlRepository.delete(id);
    }

   //通过年龄查询
    @GetMapping(value = "/girls/age/{age}")
    public List<Girl> girlListByAge(@PathVariable("age") Integer age) {
        return girlRepository.findByAge(age);
    }
}
```


PUT要使用x-www-form-urlencoded  ![img](https://img-blog.csdn.net/20180614101023696?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 事务管理

```java
@Service
public class GirlService {

    @Autowired
    private GirlRepository girlRepository;

    public void insertTwo() {
        Girl girlA = new Girl();
        girlA.setCupSize("A");
        girlA.setAge(18);
        girlRepository.save(girlA);

        Girl girlB = new Girl();
        girlB.setCupSize("BBBB");
        girlB.setAge(19);
        girlRepository.save(girlB);
    }
}

@RestController
public class GirlController {
    @Autowired
    private GirlService girlService;

    @PostMapping(value = "/girls/two")
    public void girlTwo() {
        girlService.insertTwo();
    }
}
```

修改数据库的cupSize字段的长度为1  此时会报错：com.mysql.jdbc.MysqlDataTruncation: Data truncation: Data too long for column ‘cup_size’ at row 1  但是A可以插入成功，B插入失败。

为方法添加import org.springframework.transaction.annotation.Transactional;注解，就会产生事务。

