### 解决 Mapper 文件出现黄色提示

1. 配置 Database

<img src="Mybatis&SQL.assets/image-20210810181346424.png" alt="image-20210810181346424" style="zoom: 80%;" />

2. 配置 SQL 方言

<img src="Mybatis&SQL.assets/image-20210810181758449.png" alt="image-20210810181758449" style="zoom: 80%;" />



### Mapper 文件 if 数值型不用写 != ''

通过源码了解到，mybatis在预编译sql时，使用OGNL表达式来解析if标签，对于Integer类型属性，在判断`不等于''`时，例如age != ''，OGNL会返回''的长度，源码：(s.length() == 0) ? 0.0 : Double.parseDouble( s )，因此表达式age != ''被当做`age != 0`来判断，所以当age为0时，if条件判断不通过。



### Mybatis配置日志打印

(SSM) mybatis-config.xml:


```xml
<settings>
	...
    <setting name="logImpl" value="STDOUT_LOGGING" />
</settings>
```

| 设置名  | 描述                                                | 有效值                                                       | 默认值  |
| ------- | --------------------------------------------------- | ------------------------------------------------------------ | ------- |
| logImpl | 指定 MyBatis 所用日志的具体实现，未指定时将自动查找 | SLF4J、LOG4J、LOG4J2、JDK_LOGGING、COMMONS_LOGGING、STDOUT_LOGGING、NO_LOGGING | Not set |

(Spring Boot)application.yml：

```yml
mybatis-plus/mybatis:
  # mybatis日志打印格式
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

如果使用 logback 日志打印，需要在 logback 配置文件指定 mybatis mapper 所在包的 DEGUB 级别打印：

logback-spring.xml：

```xml
<!--指定只有dev环境生效-->
<springProfile name="dev">
    <!--单独指定日志级别 优先级高于全局-->
    <logger name="com.spring.mapper">
        <level value="DEBUG"/>
    </logger>
</springProfile>
```



### Mybatis if 标签中使用字符判断时

```xml
<if test="create_name == '无'.toString()">
</if>
```









