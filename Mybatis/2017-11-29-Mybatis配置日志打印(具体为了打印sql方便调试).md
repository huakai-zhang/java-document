---
layout:  post
title:   Mybatis配置日志打印(具体为了打印sql方便调试)
date:   2017-11-29 10:47:04
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-# Mybatis

---

mybatis-config.xml:


```java
<settings>
		<setting name="cacheEnabled" value="true" />
		<setting name="useGeneratedKeys" value="true" />
		<setting name="defaultExecutorType" value="REUSE" />
		<setting name="logImpl" value="STDOUT_LOGGING" />
	</settings>
```

logImpl

描述：

指定 MyBatis 所用日志的具体实现，未指定时将自动查找。

有效值：

SLF4J | LOG4J | LOG4J2 | JDK_LOGGING | COMMONS_LOGGING | STDOUT_LOGGING | NO_LOGGING

默认值：

Not set

