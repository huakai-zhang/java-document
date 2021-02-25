什么是 Nacos

下载 releases 

nacos-server 服务

Source code 服务对应源码，不要下载master分支

官网，快速开始

### Nacos 的基本使用

mvn -Prelease-nacos clean install -U //-D maven.test.skip=true

pom springboot-nacos-starter

nacos.config.server-addr=localhost:8848

```
// dataId groupId？？？
Controller  @NacosPropertySource.  dataId groupId autoRefreshed

// hello Nacos 表示本地属性，降级
@NacosValue value("${info:hello Nacos}")  autoRefreshed
```

Nacos 提供两种方式来访问和改变配置信息：open api 和 sdk

 pom nacos-client  NacosSdkDemo监听





如果要实现一个配置中心，需要满足什么？



源码

NacosFactory createConfigServer http





服务端配置存储

derby

改变数据库

startup -m cluster







 







