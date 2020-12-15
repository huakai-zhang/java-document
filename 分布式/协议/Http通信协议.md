# 分布式通信协议HTTP

## HTTP协议的概述

### 客户端和服务器端


![img](https://img-blog.csdnimg.cn/20200408154503629.png)

### 资源

html/文本、word、视频、其他资源

### 媒体类型

MIME类型，text/html，image/jpeg

### URI 和 URL

URI:web服务器资源的名字。（index.html） URL:http://www.baidu.com:80/java/index.html[?query-string]#location

- schema: http/https/ftp 
- host: web服务器的ip地址或者域名 
- port: 服务端端口， http默认访问的端口是80 
- path: 资源访问路径 
- query-string: 查询参数

### 方法

GET、PUT、DELETE、POST、HEAD

## 报文

### request 参数


request 消息结构包含是三部分： 起始行：METHOD / path / http/version-number 首部字段：Header-Name:value 空行 主体：optional request body ![img](https://img-blog.csdnimg.cn/20200408155703278.png)

### response 参数

http/version-number status code message header-name:value


body ![img](https://img-blog.csdnimg.cn/20200408160506238.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

## 状态码

http/1.1版本的协议里面定义了五种类型的状态码：

- 1XX 提示信息 
- 2XX 成功 
- 3XX 重定向 
- 4XX 客户端错误 
- 5XX 服务器端的错误

## HTTP协议的特点

1. 无状态 cookie+session 
2. 多次请求 
3. 基于TCP协议

## HTTPS

在HTTP协议上，多了一个SSL/TLS的加密。ISOC 在SSL的基础上发布了升级版本 TLS1.2。

### HTTPS 工作原理


![img](https://img-blog.csdnimg.cn/20200408163445155.png)

#### 1.使用对称加解密


![img](https://img-blog.csdnimg.cn/2020040816340994.png)

#### 2.密钥是公开的，所有的客户端都可以拿到


![img](https://img-blog.csdnimg.cn/20200408163715565.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

#### 3.针对不同的客户端使用不同的密钥


![img](https://img-blog.csdnimg.cn/20200408164529713.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) 协商过程是没有加密的，所以还会出现被截断的问题

#### 4.使用非对称加密


非对称：公钥和私钥的概念 ![img](https://img-blog.csdnimg.cn/20200408165301860.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) 客户端如何拿到公钥？

1. 服务器端把公钥发送给每一个客户端 
2. 服务器端把公钥放到远程服务器，客户端可以请求到 
3. 让浏览器保存所有的公钥（不现实）

#### 5.公钥被调包的问题按照上面的方案，永远存在


选择方案1： ![img](https://img-blog.csdnimg.cn/20200408165942708.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

#### 6.使用第三方机构来解决

通过第三方机构，使用第三方机构的私钥对我们【需要传输的公钥】进行加密

#### 7.数字证里面包含的内容


公司信息、网站信息、数字证书的算法、公钥 ![img](https://img-blog.csdnimg.cn/20200408174208551.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) https申请证书并部署到网站流程：

1. 生成一对秘钥，设公钥为pubk1，私钥为prik1 
2. 假设发布的网站地址为https://www.example.com 
3. 生成一个CSR文件（Cerificate Signing Request），该文件内容包括: pubk1，网站地址，以及营业执照等信息，然后将该文件发给CA机构 
4. CA机构收到CSR文件后，进行审核，主要检查网站地址的拥有者是否是证书的申请者 
5. 审核通过后，CA机构生成一对秘钥，假设采用ECDSA签名算法，公钥为pubk2，私钥为prik2。用prik2对CSR文件进行签名得到签名值sigVal，将sigVal附在CSR文件后面形成证书文件caFile，caFile中还要添加CA机构的信息，如: 签名算法，CA机构名称等 
6. 将证书文件caFile放到网站服务器对应目录下

浏览器验证证书流程：

1. 客户端发起一个https请求 a)客户端支持的加密方式 b)客户端生成的随机数（第一个随机数） 
   <li>服务端收到请求后，拿到随机数，返回 a)证书（颁发机构（CA）、证书内容本身的数字签名（使用第三方机构的私钥加密）、证书持有者的公钥、证书签名用到的hash算法） b)生成一个随机数，返回给客户端（第二个随机数）</li> 
   <li>客户端拿到证书以后做验证 a)根据颁发机构找到本地的跟证书 b)根据CA得到根证书的公钥，通过公钥对数字签名解密，得到证书的内容摘要 A c)用证书提供的算法对证书内容进行摘要，得到摘要 B d)通过A和B的对比，也就是验证数字签名</li> 
2. 验证通过以后，生成一个随机数（第三个随机数），通过证书内的公钥对这个随机数加密，发送给服务器端 
3. （随机数1+2+3）通过对称加密得到一个密钥。（会话密钥） 
4. 通过会话密钥对内容进行对称加密传输