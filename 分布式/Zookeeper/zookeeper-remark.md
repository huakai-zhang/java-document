## 内存数据和磁盘数据

zookeeper会定时把数据存储在磁盘上。 DataDir，存储的是数据的快照（快照： 存储某一个时刻全量的内存数据内容）

DataLogDir，存储事务日志，文件名为log.zxid 查看事务日志的命令：

```java
java -cp :/usr/local/zookeeper-3.4.10/lib/slf4j-api-1.6.1.jar:/usr/local/zookeeper-3.4.10/zookeeper-3.4.10.jar org.apache.zookeeper.server.LogFormatter log.200000001
```



## zookeeper 有三种日志

zookeeper.out，运行日志 快照，存储某一时刻的全量数据 事务日志，事务操作的日志记录

