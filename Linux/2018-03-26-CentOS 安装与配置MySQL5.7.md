---
layout:  post
title:   CentOS 安装与配置MySQL5.7
date:   2018-03-26 20:48:14
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-日常积累
-CentOS
-mysql

---



## 下载MySQL

```java
wget http://mirrors.sohu.com/mysql/MySQL-5.7/mysql-5.7.17-linux-glibc2.5-x86_64.tar.gz
```

## 解压文件

```java
tar -zxvf mysql-5.7.17-linux-glibc2.5-x86_64.tar.gz -C /usr/local/
 # 修改文件名为mysql
 mv mysql-5.7.17-linux-glibc2.5-x86_64/ mysql
```

## 配置启动文件

```java
cd mysql/support-files/
cp my-default.cnf /etc/my.cnf
# 如果你在安装时Linux虚拟机时同时安装了默认的mysql，此时操作以上步骤，终端将会提示你文件已存在是否覆盖，输入yes覆盖即可。
cp: overwrite ‘/etc/my.cnf’? yes
```

## 配置数据库编码

```java
vi /etc/my.cnf
[mysql]
default-character-set=utf8

[mysqld]
default-storage-engine=INNODB
character_set_server=utf8
lower_case_table_names=1
```

lower_case_table_names=1 说明：Linux下mysql安装完后是默认区分表名的大小写，不区分列名的大小写，在/etc/my.cnf中的[mysqld]后添加添加lower_case_table_names=1，重启MYSQL服务，设置不区分表名的大小写；

```java
# 重启mysql的两种方式
service mysqld restart  
/etc/init.d/mysql restart
```

## 复制mysql.server到/etc/init.d/目录下(目的想实现开机自动执行效果)

```java
cp mysql.server /etc/init.d/mysql
```

## 修改/etc/init.d/mysql参数

```java
vi /etc/init.d/mysql
dir=/usr/local/mysql
datadir=/usr/local/mysql/data
```

## 创建用户组和用户

```java
groupadd mysql
#建立mysql用户，并且把用户放到mysql组:
useradd -r -g mysql mysql
#给mysql用户设置一个密码:
passwd mysql
#给目录/usr/local/mysql 更改拥有者:
chown -R mysql:mysql /usr/local/mysql/
#初始化数据库
cd /usr/local/mysql/bin/
./mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
```

初始化后会生成一个临时密码，在打印的数据的最后部分有一个 root@localhost:：*临时密码，要记住这个临时密码  初始化数据库时候可能会出现错误：  /usr/local/mysql# ./bin/mysqld – defaults-file=/etc/my.cnf –initialize –user=mysql  ./bin/mysqld: error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory  解决方法：

```java
yum install -y libaio  //安装后在初始化就OK了
```

```java
#给数据库加密
./mysql_ssl_rsa_setup --datadir=/usr/local/mysql/data
#启动mysql
./mysqld_safe --user=mysql &
#检查mysql是否启动,发现有进程便代表启动成功
ps -ef|grep mysql
```

## 设置密码

```java
#进入客户端
./mysql -u root -p
Enter password:临时密码
mysql> set password=password('新密码');
```

## 设置远程访问

```java
#mysql的默认端口3306
firewall-cmd --zone=public --add-port=3306/tcp --permanent
success
firewall-cmd --reload
success
```

## 设置mysql的远程访问

```java
#设置远程访问账号:grant all privileges on . to 远程访问用户名@’%’ identified by ‘用户密码’
mysql> grant all privileges on *.* to root@'%' identified by 'root';
#刷新
mysql> flush privileges;
```

## 设置开机自启动

```java
#添加服务mysql
chkconfig --add mysql
#设置mysql服务为自启动
chkconfig mysql on
#：配置环境变量
vi /etc/profile
#最后一行添加
export PATH=$JAVA_HOME/bin:/usr/local/mysql/bin:$PATH
#source /etc/profile
```

