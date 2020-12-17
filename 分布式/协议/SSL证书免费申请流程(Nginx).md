1.访问域名https://freessl.cn/

2.输入需要申请SSL证书的域名 

![img](SSL证书免费申请流程(Nginx).assets/20191115095144925.png)

选择免费品牌<font color=red>亚洲诚信</font>，然后点击<font color=red>创建免费的SSL证书</font>按钮

3.选择证书类型<font color=red>RSA</font>、验证类型<font color=red>文件验证</font>、CSP生成<font color=red>浏览器生成</font>，点击<font color=red>点击创建</font>按钮

![img](SSL证书免费申请流程(Nginx).assets/20191115095526850.png)

4.点击下图中的下载文件，获取<font color=red>fileauth.txt</font>

![img](SSL证书免费申请流程(Nginx).assets/20191115095752882.png)

5.将文件上传至域名对应服务器nginx文件夹的<font color=red>html/.well-known/pki-validation</font>文件夹内 

![img](https://img-blog.csdnimg.cn/2019111510002352.png)

6.点击<font color=red>点击验证</font>按钮 

![img](SSL证书免费申请流程(Nginx).assets/20191115100236892.png)

7.点击<font color=red>下载证书</font>按钮 

![img](SSL证书免费申请流程(Nginx).assets/20191115100433614.png)

8.下载得到一个压缩包，解压得到<font color=red>full_chain.pem、private.key</font>两个文件

![img](SSL证书免费申请流程(Nginx).assets/20191115100524372.png)

9.将这两个文件上传至服务器上nginx目录下的<font color=red>conf/ssl</font>目录，记住上传文件的目录

![img](SSL证书免费申请流程(Nginx).assets/20191115100715390.png)

10.配置<font color=red>nginx.conf</font>文件

```shell
server {
        listen       443 ssl;
        server_name  localhost;
        client_max_body_size 10m;

        ssl_certificate      ssl/full_chain.pem;
        ssl_certificate_key  ssl/private.key;

        ssl_session_cache    shared:SSL:10m;
        ssl_session_timeout  10m;

        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;
}
```

11.重启nginx

```shell
cd /nginx/sbin
./nginx -s reload
```

12.访问https的域名，点击证书（有效）可以查看证书

![img](SSL证书免费申请流程(Nginx).assets/20191115101503134.png)

![img](https://img-blog.csdnimg.cn/20191115101600500.png)

------

