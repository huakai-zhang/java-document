---
layout:  post
title:   CentOS7 安装 Java 8、Tomcat8（tar.gz安装 ）
date:   2018-03-23 20:08:10
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-日常积累

---



## 在服务器网站配置规则

#### 进入阿里云主机控制台，安全组，如下图所示，点击配置规则


![img](https://img-blog.csdn.net/20180323200627984?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

#### 配置规则，添加规则允许任何IP访问80，如下图所示


![img](https://img-blog.csdn.net/20180323200634597?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## tar.gz下载地址

链接：https://pan.baidu.com/s/1ydg3yVE72_4SG5b-Zu6A8Q 密码：h95a

## 安装JAVA8

#### 解压安装 tar.gz

```java
tar -zxvf jdk-8u151-linux-x64.tar.gz -C /opt/soft
```

#### 配置环境变量

```java
# 修改配置文件,路径为刚才解压的路径/jdk1.8.0_151
vi /etc/profile
# 键盘输入 i
# 移动光标，在export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL下添加

export JAVA_HOME=/opt/soft/jdk1.8.0_151
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

添加完成后，按Esc键。然后按Shift + :，输入wq，进行保存并退出操作。

```java
# 刷新配置文件
source /etc/profile
```

#### 检查Java8是否安装成功

```java
javac -version
```


下图表示安装成功： ![img](https://img-blog.csdn.net/20180323194915832?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 安装Tomcat8

#### 解压

```java
tar -zxvf apache-tomcat-8.5.29.tar.gz -C /usr/tomcat
```

#### 修改端口

```java
# 目录调节到conf
cd /usr/tomcat/apache-tomcat-8.5.29/conf/
# 编辑server.xml文件
vi server.xml
# 键盘输入 i
# 移动光标，修改<Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />中的8080为80
```

修改完成后，按Esc键。然后按Shift + :，输入wq，进行保存并退出操作。

#### 启动Tomcat

```java
cd /usr/tomcat/apache-tomcat-8.5.29/bin/
./startup.sh
```

#### 配置防火墙

```java
# 启动防火墙
systemctl start firewalld
# 将80端口添加到防火墙例外并重启
firewall-cmd --zone=public --add-port=8080/tcp --permanent
firewall-cmd --reload
```

## 访问IP地址测试：


![img](https://img-blog.csdn.net/20180323200138975?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

