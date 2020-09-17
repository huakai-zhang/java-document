---
layout:  post
title:   SpringBoot开发实战—日志框架
date:   2018-06-21 10:46:13
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-# Spring Boot

---



## 日志框架

一套能实现日志输出的工具包，能够描述系统运行状态的所有时间都可以算作日志（用户下线、接口超时、数据库崩溃、hello world）。

#### 日志框架的能力

定制输出目标  定制输出格式  携带上下文信息  运行时选择性输出  灵活的配置  优异的性能

#### 常见的日志框架

日志门面：  JCL：apache自带的common logging  SLF4j  jboss-logging  日志实现：  Log4j  Log4j2  Logback  JUL：JDK自带的log

选择SLF4j和Logback同一个作者。

#### 测试日志

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class LoggerTest {

    private final Logger logger = LoggerFactory.getLogger(LoggerTest.class);

    @Test
    public void test1() {
        logger.debug("debug...");
        // 系统默认日志级别为info，不会打印debug
        logger.info("info...");
        logger.error("error...");
    }

}
```

打印：  2018-06-20 11:43:20.782 INFO 11620 — [ main] com.imooc.LoggerTest : info…  2018-06-20 11:43:20.782 ERROR 11620 — [ main] com.imooc.LoggerTest : error…

```java
package org.slf4j.event;

public enum Level {
    ERROR(40, "ERROR"),
    WARN(30, "WARN"),
    INFO(20, "INFO"),
    DEBUG(10, "DEBUG"),
    TRACE(0, "TRACE");
 }
```

#### @Slf4j的使用

添加依赖：

```java
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@Slf4j
public class LoggerTest {

    @Test
    public void test1() {
        log.debug("debug...");
        log.info("info...");
        log.error("error...");
    }

}
```


如果还不能解析log：  1.需要安装插件：  ![img](https://img-blog.csdn.net/20180620122138271?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

2.可能是idea的版本过低

#### 打印变量

```java
String name = "zcy";
    String password = "123456";
    log.info("name: {}, password: {}", name, password);
```

#### Logback的配置

第一种方式：application.yml：

```java
# 配置日志格式
logging:
  pattern:
    console: "%d - %msg%n"
  # 日志生成的路径，不受console影响
  path: /var/log/tomcat/
  # 自定义名字
  file: /var/log/tomcat/sell.log
  level: debug
  # 指定到某个类的级别限制
  # level:
      # com.imooc.LoggerTest: debug
```

第二种方式：logback-spring.xml

```java
<?xml version="1.0" encoding="UTF-8"?>

<configuration>

    <appender name="consoleLog" class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>
                %d - %msg%n
            </pattern>
        </layout>
    </appender>

    <appender name="fileInfoLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>DENY</onMatch>
            <onMismatch>ACCEPT</onMismatch>
        </filter>
        <encoder>
            <pattern>
                %msg%n
            </pattern>
        </encoder>
        <!--滚动策略-->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--路径-->
            <fileNamePattern>/var/log/tomcat/sell/info.%d.log</fileNamePattern>
        </rollingPolicy>
    </appender>

    <appender name="fileErrorLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
        <encoder>
            <pattern>
                %msg%n
            </pattern>
        </encoder>
        <!--滚动策略-->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--路径-->
            <fileNamePattern>/var/log/tomcat/sell/error.%d.log</fileNamePattern>
        </rollingPolicy>
    </appender>

    <root level="info">
        <appender-ref ref="consoleLog"/>
        <appender-ref ref="fileInfoLog"/>
        <appender-ref ref="fileErrorLog"/>
    </root>

</configuration>
```

