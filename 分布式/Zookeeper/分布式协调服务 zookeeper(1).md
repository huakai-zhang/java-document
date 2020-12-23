### leader选举

leaderElection，AuthFastLeaderElection，FastLeaderElection(默认方式) org.apache.zookeeper.server.quorum.QuorumPeer#startLeaderElection

源码地址：https://github.com/apache/zookeeper.git 需要的条件： jdk 1.7以上 、ant 、idea

#### FastLeaderElection

serverid：在配置server集群的时候，给定服务器的标识id（myid） zxid：服务器在运行时产生的数据ID， zxid的值越大，表示数据越新 Epoch：选举的轮数 server的状态：Looking、 Following、Observering、Leading

第一次初始化启动的时候： LOOKING


1. 所有在集群中的server都会推荐自己为leader，然后把（myid、zxid、epoch）作为广播信息，广播给集群中的其他server, 然后等待其他服务器返回 
2. 每个服务器都会接收来自集群中的其他服务器的投票。集群中的每个服务器在接受到投票后，开始判断投票的有效性 a)判断逻辑时钟(Epoch) ，如果Epoch大于自己当前的Epoch，说明自己保存的Epoch是过期。更新Epoch，同时clear其他服务器发送过来的选举数据。判断是否需要更新当前自己的选举情况 b)如果Epoch小于目前的Epoch，说明对方的epoch过期了，也就意味着对方服务器的选举轮数是过期的。这个时候，只需要讲自己的信息发送给对方 ![img](https://img-blog.csdnimg.cn/2020041410234291.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

内存数据和磁盘数据

zookeeper会定时把数据存储在磁盘上。 DataDir，存储的是数据的快照（快照： 存储某一个时刻全量的内存数据内容）

DataLogDir，存储事务日志，文件名为log.zxid 查看事务日志的命令：

```java
java -cp :/usr/local/zookeeper-3.4.10/lib/slf4j-api-1.6.1.jar:/usr/local/zookeeper-3.4.10/zookeeper-3.4.10.jar org.apache.zookeeper.server.LogFormatter log.200000001
```



## zookeeper 有三种日志

zookeeper.out，运行日志 快照，存储某一时刻的全量数据 事务日志，事务操作的日志记录

