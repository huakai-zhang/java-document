## 什么是Docker

Docker：an open source project to pack,ship and run any application as a lightweight container. 将任何应用以轻量级的形式来打包，发布和运行


Docker可以被粗糙地理解为轻量级的虚拟机。 ![img](https://img-blog.csdnimg.cn/20181031114648948.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

Docker利用Host OS里面的namespace，controlgroup来做到将应用程序分离，因为Docker没有hypervisor虚拟层，它会比虚拟机轻量很多，包括程序的启动速度以及存储的需求等等。

## 安装Docker

#### windows


1.前往 [Docker官网](https://www.docker.com/)下载Docker 2.安装 ![img](https://img-blog.csdnimg.cn/20181031145933647.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)


3.命令行运行Docker ![img](https://img-blog.csdnimg.cn/20181031150141825.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

#### Linux centos7.3

```java
# O-是大写的o和减号，-q标识输出要简单，O-标识标准输出，而不是输出到文件
sudo wget -qO- https://get.docker.com | sh
# 把xxx用户添加到docker用户组中
sudo usermod -aG docker xxx 
#启动docker服务
service docker start
#设置成开机自启
sudo chkconfig docker on
```

## 术语

## Docker命令

## Dockerfile

通过编写简单的文件自创docker镜像。 1.创建文件Dockerfile，内容为

```java
FROM alpine:latest
MAINTAINER spring
CMD echo "Hello Docker!"
```

alpine为专门为Docker设计的一款体积很小的linux，MAINTAINER 作者

2.运行Dockerfile -t 表示给一个标签叫做hello_docker，. 表示路径名，将该路径下所有文件都送给Docker engine

```java
docker build -t hello_docker .
```


![img](https://img-blog.csdnimg.cn/20181031160129474.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

```java
docker images hello_docker
```

REPOSITORY TAG IMAGE ID CREATED SIZE hello_docker latest 77ca1e37168e 13 seconds ago 4.413 MB

```java
docker run hello_docker
```

Hello Docker!

#### 复杂的Dockerfile

```java
FROM ubuntu
MAINTAINER spring
# 采用国内镜像下载地址
RUN sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
RUN apt-get update
RUN apt-get install -y nginx
COPY index.html /var/www/html
# 将nginx在前台执行，而不是作为守护进程
ENTRYPOINT ["/usr/sbin/nginx", "-g", "daemon off;"]
EXPOSE 80
```

同目录下创建一个index.html

```java
docker build -t hello_nginx .
docker run -d -p 80:80 hello_nginx
curl http://localhost
```

#### Dockerfile语法

#### 镜像分层


Dickerfile中的每一行都产生一个新层 ![img](https://img-blog.csdnimg.cn/20181031163719657.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

已经存在的image里面的层是只读的，一旦一个image被运行成为一个容器的话，会产生一个新层，这一次是可读可写的。 分层的好处：多个image可以共享分层，这样存储压力就小很多。

## Volume

提供独立于容器之外的持久化存储

```java
docker run -d --name nginx -v /usr/share/nginx/html nginx
docker inspect nginx
```


![img](https://img-blog.csdnimg.cn/20181031172037754.png)

这个路径，Alpal的主机里面，不是在容器里面

```java
cd    /var/lib/docker/volumes/8e02ff6ffec3441a4812212bdbbb6b69aaf69fcdba72855a55c65686b80ff51d/_data
ls
cat index.html
echo "it's 2018" > index.html
```

在容器里面查看：

```java
docker exec -it nginx /bin/bash
cd /usr/share/nginx/html/
cat index.html
```

#### 将本地目录挂载到容器里面的数据卷

将目录html挂载到容器

```java
# /html为绝对路径，windows下如果文件在E盘下，需要写E:/html
docker run -p 80:80 -d -v /html:/usr/share/nginx/html nginx
```


问题： 1.windows下运行如果出错：docker: Error response from daemon: E: drive is not shared. Please share it in Docker for Windows Settings. 需要右键点击任务栏Docker图标选择Setting： ![img](https://img-blog.csdnimg.cn/20181101104716514.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

2.共享磁盘可能会出现 Firewall detected A firewall is blocking file sharing between Windows and the containers. See documentation for more info. Windows防火墙阻止Docker共享目录解决方案： （1）进入网络和共享中心； （2）选择“VEthernet (DockerNAT) ”适配器； （3）选择“属性” （4）卸载“Microsoft 网络的文件和打印机共享” （5）重新安装“Microsoft 网络的文件和打印机共享”，点击“安装”，选择“服务”，点击“添加”，选择“Microsoft”-&gt;“Microsoft 网络的文件和打印机共享”，点击“确定” （6）安装完成后，重新配置驱动盘即可。

#### 创建一个仅有数据的容器并将该容器挂载到其他容器中

```java
docker create -v E:/data:/var/mydata --name data_container ubuntu
docker run -it --volumes-from data_container ubuntu /bin/bash
mount
```

会发现一个mydata的文件夹

```java
ca /var/mydata
touch whatever.txt
```

在本地会发现一个whatever.txt文件

## Registry镜像仓库

```java
# 搜索镜像
docker search whalesay
# 拉取下来
docker pull whalesay
# 将自己镜像push到仓库
docker push myname/whalesay
```

#### 国内仓库

daocloud 时速云 阿里云

```java
docker search whalesay
docker pull docker/whalesay
docker run docker/whalesay cowsay Docker很好玩！
```


![img](https://img-blog.csdnimg.cn/20181101113231638.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

```java
# 创建自己的tag
docker tag docker/whalesay spring/whalesay
# 上传到仓库
docker push spring/whalesay
```

如果提示unauthorized: authentication required，需要登录自己的Docker帐号。

```java
docker login
Username:
Password:
```


上传成功后，在Docker官网可以看到自己上传的镜像： ![img](https://img-blog.csdnimg.cn/2018110111420991.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

## compose

Mac和Windows的Docker中已带有compose，Linux需要单独安装。

```java
curl -L https://github.com/docker/compose/releases/download/1.9.0/docker-compose-$(uname -s )-$(uname -m) > /usr/local/bin/docker-compose
# 给与可执行权限
chmod a+x /usr/local/bin/docker-compose
```

#### ghost配置

Docker:

```java
FROM ghost
COPY ./config.production.json /var/lib/ghost/content/config.production.json
EXPOSE 2368
#CMD ["npm","start","--production"]
```

config.production.json

```java
{
    "url": "http://localhost:2368/",
    "server": {
        "port": 2368,
        "host": "0.0.0.0"
    },
    "database": {
        "client": "mysql",
        "connection": {
            "host": "db",
            "user": "ghost",
            "password": "ghost",
            "database": "ghost",
            "port": 3307,
            "charset": "utf8"
        }
    },
    "mail": {
        "transport": "Direct"
    },
    "logging": {
        "transports": [
            "file",
            "stdout"
        ]
    },
    "process": "systemd",
    "paths": {
        "contentPath": "/var/lib/ghost/content"
    }
}
```

#### nginx配置

Dockerfile：

```java
FROM nginx
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
```

nginx.conf:

```java
worker_processes 4;
events {worker_connections 1024;}
http {
    server {
        listen 80;
        location / {
            proxy_pass http://ghost-app:2368;
        }
    }
}
```

#### docker-compose.yml

```java
version: '2'
networks:
  ghost:
services:
  ghost-app:
    build: ghost
    networks:
      - ghost
    depends_on:
      - db
    ports:
      - "2368:2368"
  nginx:
    build: nginx
    networks:
      - ghost
    depends_on:
      - ghost-app
    ports:
      - "80:80"
  db:
    image: "mysql:5.7.15"
    networks:
      - ghost
    environment:
      MYSQL_ROOT_PASSWORD: mysqlroot
      MYSQL_USER: ghost
      MYSQL_PASSWORD: ghost
    volumes:
      - ./data:/var/lib/mysql
    ports:
      - "3307:3307"
```


![img](https://img-blog.csdnimg.cn/20181101150227556.png)


![img](https://img-blog.csdnimg.cn/20181101163730966.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

#### docker-compose.yml常用命令

```java
docker-compost stop     #停止容器   
docker-compose rm            #删除容器
docker-compose build         #重新建立
docker-compose up -d        #启动运行
```

