---
layout:  post
title:   Spring Cloud Rancher与部署
date:   2018-11-28 09:29:03
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-# 分布式

---



## 将Eureka运行到Docker中

1.创建一个简单的Eureka项目 2.yml配置

```java
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
    register-with-eureka: false
    fetch-registry: false
  server:
    enable-self-preservation: false
spring:
  application:
    name: eureka
server:
  port: 8761
```

3.打包

```java
mvn clean package -Dmaven.test.skip=true
```


![img](https://img-blog.csdnimg.cn/20181123162716166.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) 4.docker build

```java
docker build -t springcloud/eureka .
```


![img](https://img-blog.csdnimg.cn/20181123162759925.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) 5.docker run

```java
docker run -p 8761:8761 -d springcloud/eureka
```

启动后稍等一会，访问http://localhost:8761

## rancher


在rancher安装之前，先说一个VirtualBox的坑。 Docker安装之后需要Windows启用Hyper-V，而VirtualBox需要Windows关闭Hyper-V。 所以在安装Docker之后，打开VirtualBox启动.ova文件时，会报错: 不能为虚拟电脑 XXX 打开一个新任务 明细： Raw-mode is unavailable courtesy of Hyper-V.(VERR_SUPDRV_NO_RAW_MODE_HYPER_V_ROOT) ![img](https://img-blog.csdnimg.cn/20181123174642913.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)


此时需要前往控制面板，重新关闭Hyper-V。 ![img](https://img-blog.csdnimg.cn/20181123174708864.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

#### Linux下安装rancher

1.安装docker  [点击前往Linux下安装docker](https://blog.csdn.net/wszcy199503/article/details/83579172)

2.安装rancher

```java
sudo docker run -d --restart=unless-stopped -p 8080:8080 rancher/server:stable
```


3.启动成功，添加主机 ![img](https://img-blog.csdnimg.cn/20181126095910794.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)


![img](https://img-blog.csdnimg.cn/20181126100412934.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)



4.填写代理服务器地址，并且在安装好docker的代理服务器上执行复制好的命令 ![img](https://img-blog.csdnimg.cn/20181126101546558.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) 这段地址一定要复制，千万不要手动去输入，被坑的很惨。 ![img](https://img-blog.csdnimg.cn/20181126175025251.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)


![img](https://img-blog.csdnimg.cn/20181126175202775.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

## 部署镜像

1.修改config项目springboot版本为2.0.0.M3，spring cloud版本为Finchley.M2 2.生成镜像，并启动校验

```java
mvn clean package -Dmaven.test.skip=true
docker build -t springcloud/config .
docker run -p 8080:8080 -d springcloud/config
```

3.登录镜像仓库

```java
docker login -u 登录账号 -p 密码 对应网址
```

返回「Login Succeded」即为登录成功。 4.标记本地镜像(这里使用网易云)

```java
# 查看镜像id
docker images | grep config
# 标记本地镜像 docker tag {镜像名或ID} hub.c.163.com/{你的用户名}/{标签名}
docker tag af6086bf7995 hub.c.163.com/springzhang/config
```


![img](https://img-blog.csdnimg.cn/20181127102916350.png)

5.推送至镜像仓库

```java
docker push hub.c.163.com/springzhang/config
```


此时的镜像是私有的需要在设置中设置为公有。 ![img](https://img-blog.csdnimg.cn/20181127103413802.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

6.修改config配置

```java
eureka:
  client:
    service-url:
      defaultZone: http://eureka:8761/eureka/
```


7.进入rancher页面 应用 -》 用户 -》 添加应用 -》填写应用名，添加成功 点击应用名进入应用-》 添加服务 ![img](https://img-blog.csdnimg.cn/2018112714571966.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)



8.启动成功，点击访问 ![img](https://img-blog.csdnimg.cn/2018112715075247.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) ![img](https://img-blog.csdnimg.cn/20181127150847223.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

## 构建Eureka高可用服务

1.修改配置 application.yml:

```java
spring:
  profiles:
    active: eureka1
```

application-eureka1.yml:

```java
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8762/eureka/
    fetch-registry: false
  server:
    enable-self-preservation: false
spring:
  application:
    name: eureka
server:
  port: 8761
```

application-eureka2.yml:

```java
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
    fetch-registry: false
  server:
    enable-self-preservation: false
spring:
  application:
    name: eureka
server:
  port: 8762
```

启动项目：

```java
java -jar -Dspring.profiles.active=eureka1 target/eureka-0.0.1-SNAPSHOT.jar
java -jar -Dspring.profiles.active=eureka2 target/eureka-0.0.1-SNAPSHOT.jar
```


2.重新push到docker仓库 3.rancher添加服务，并添加环境变量 ![img](https://img-blog.csdnimg.cn/20181127170104693.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

4.重新部署config

```java
eureka:
  client:
    service-url:
      defaultZone: http://eureka1:8761/eureka/,http://eureka2:8762/eureka/
```

5.部署product 修改config的配置

```java
# 注册时使用ip而不是主机名
eureka:
  instance:
    prefer-ip-address: true
```

其他同样的服务，也需要通信，同样需要找个配置。



6.部署api-geteway 不需要添加端口映射 ![img](https://img-blog.csdnimg.cn/20181128092306466.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) ![img](https://img-blog.csdnimg.cn/20181128091851152.png)


![img](https://img-blog.csdnimg.cn/20181128092447223.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)


![img](https://img-blog.csdnimg.cn/20181128092843240.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

