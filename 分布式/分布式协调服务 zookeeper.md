## 分布式环境特点

分布性 并发性：程序运行过程中，并发性操作是很常见的。比如同一分布式系统中的多个节点，同时访问一个共享资源。数据库、分布式存储 无序性：进程之间的消息通信，会出现顺序不一致问题

## 分布式环境下面临的问题

网络通信：网络本身的不可靠性，因此会涉及到一些网络通信问题 网络分区：当网络发生异常导致分布式系统中部分节点之间的网络延时不断增大，最终导致组成分布式架构的所有节点，只有部分节点能够正常通信 三态：在分布式架构里面，除了成功、失败、超时 分布式事务：ACID(原子性、一致性、隔离性、持久性)

## 中心化和去中心化

冷备或者热备 分布式架构里面，很多的架构思想采用的是：当集群发生故障的时候，集群中的人群会自动“选举”出一个新的领导。 最典型的是： zookeeper / etcd

## 经典的CAP/BASE理论

### CAP

C（一致性 Consistency）：所有节点上的数据时刻保持一致 A（可用性Availability）：每个请求都能够收到一个响应，无论响应成功或失败 P（分区容错 Partition-tolerance）：表示系统出现脑裂以后，可能导致某些server与集群中的其他机器失去联系 CAP理论仅适用于原子读写的Nosql场景，不适用于数据库系统。

### BASE

基于CAP理论，CAP理论并不适用于数据库事务（因为更新一些错误的数据而导致数据出现紊乱，无论什么样的数据库高可用方案都是徒劳） ，虽然 XA 事务可以保证数据库在分布式系统下的ACID特性，但是会带来性能方面的影响； eBay尝试了一种完全不同的套路，放宽了对事务ACID的要求。提出了BASE理论。 Basically available：数据库采用分片模式， 把100W的用户数据分布在5个实例上。如果破坏了其中一个实例，仍然可以保证80%的用户可用。 soft-state：在基于client-server模式的系统中，server端是否有状态，决定了系统是否具备良好的水平扩展、负载均衡、故障恢复等特性。Server端承诺会维护client端状态数据，这个状态仅仅维持一小段时间, 这段时间以后，server端就会丢弃这个状态，恢复正常状态。 Eventually consistent：数据的最终一致性。


zookeeper是一个开源的分布式协调服务，是由雅虎创建的，基于google chubby。

## zookeeper是什么

分布式数据一致性的解决方案

## zookeeper能做什么

数据的发布/订阅（配置中心:disconf） 负载均衡（dubbo利用了zookeeper机制实现负载均衡） 命名服务 master选举(kafka、hadoop、hbase) 分布式队列 分布式锁

## zookeeper的特性

顺序一致性：从同一个客户端发起的事务请求，最终会严格按照顺序被应用到zookeeper中 原子性：所有的事务请求的处理结果在整个集群中的所有机器上的应用情况是一致的，也就是说，要么整个集群中的所有机器都成功应用了某一事务、要么全都不应用 可靠性：一旦服务器成功应用了某一个事务数据，并且对客户端做了响应，那么这个数据在整个集群中一定是同步并且保留下来的 实时性：一旦一个事务被成功应用，客户端就能够立即从服务器端读取到事务变更后的最新数据状态；（zookeeper仅仅保证在一定时间内，近实时）


## 单机环境安装

1. 下载zookeeper的安装包 http://apache.fayea.com/zookeeper/stable/zookeeper-3.4.10.tar.gz 
<li>解压zookeeper tar -zxvf zookeeper-3.4.10.tar.gz</li> 
<li>cd 到 ZK_HOME/conf , copy一份zoo.cfg cp zoo_sample.cfg zoo.cfg</li> 
<li>sh zkServer.sh xxx {start|start-foreground|stop|restart|status|upgrade|print-cmd}</li> 
5. sh zkCli.sh -server ip:port

## 集群环境

zookeeper集群，包含三种角色:

- Leader：接受所有Follower的提案请求并统一协调发起提案的投票，负责与所有的Follower进行内部数据交换（同步） 
- Follower：直接为客户端服务并参与提案的投票，同时与Leader进行数据交换（同步） 
- Observer：直接为客户端服务但不参与提案的投票，同时与Leader进行数据交换（同步），observer 是一种特殊的zookeeper节点。可以帮助解决zookeeper的扩展性（如果大量客户端访问我们zookeeper集群，需要增加zookeeper集群机器数量。从而增加zookeeper集群的性能。 导致zookeeper写性能下降， zookeeper的数据变更需要半数以上服务器投票通过。造成网络消耗增加投票成本） 1.observer不参与投票。 只接收投票结果。 2.不属于zookeeper的关键部位。

#### 创建Observer服务器

在zoo.cfg里面增加： peerType=observer



server.1=192.168.11.129:2181:3181:observer server.2=192.168.11.131:2181:3181 server.3=192.168.11.135:2181:3181 ![img](https://img-blog.csdnimg.cn/20200411150228977.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) ![img](https://img-blog.csdnimg.cn/2020041116070855.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

### 修改配置文件

server.id=host:port:port id的取值范围：1~255，用id来标识该机器在集群中的机器序号 2888表示follower节点与leader节点交换信息的端口号 3181表示leader选举的端口，如果leader节点挂掉了, 需要一个端口来重新选举

server.1=192.168.11.129:2888:3181 server.2=192.168.11.131:2888:3181 server.3=192.168.11.135:2888:3181

### 创建myid

在每一个服务器的zookeeper配置文件zoo.cfg中配置的dataDir目录下创建一个myid的文件，文件就一行数据，数据内容是每台机器对应的server ID的数字

### 启动zookeeper

## zoo.cfg 配置文件分析

tickTime=2000 zookeeper中最小的时间单位长度 （ms） initLimit=10 follower节点启动后与leader节点完成数据同步的时间(10个tickTime时间) syncLimit=5 leader节点和follower节点进行心跳检测的最大延时时间(5个tickTime时间) dataDir=/tmp/zookeeper 表示zookeeper服务器存储快照文件的目录 dataLogDir 表示配置 zookeeper事务日志的存储路径，默认指定在dataDir目录下 clientPort 表示客户端和服务端建立连接的端口号： 2181

## zookeeper 中的一些概念

### 数据模型

zookeeper 的数据模型和文件系统类似，每一个节点称为：znode，是 zookeeper 中的最小数据单元。每一个 znode 上都可以保存数据和挂载子节点。从而构成一个层次化的属性结构。

#### 节点特性


持久化节点：节点创建后会一直存在zookeeper服务器上，直到主动删除 持久化有序节点：每个节点都会为它的一级子节点维护一个顺序 临时节点：临时节点的生命周期和客户端的会话保持一致。当客户端会话失效，该节点自动清理。临时节点不能挂子节点。 临时有序节点：在临时节点上多了一个顺序性特性 ![img](https://img-blog.csdnimg.cn/20200411164735445.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

### 会话


![img](https://img-blog.csdnimg.cn/2020041117595849.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

### Watcher


zookeeper提供了分布式数据发布/订阅,zookeeper允许客户端向服务器注册一个watcher监听。当服务器端的节点触发指定事件的时候会触发watcher。服务端会向客户端发送一个事件通知。 watcher的通知是一次性，一旦触发一次通知后，该watcher就失效。如果需要永久监听，则需要反复注册。 zkClient 和 curator 都做了永久监听的封装。 java api的话， zk.exists , zk.getData 创建一个watcher监听。 zookeeper序列化使用的是Jute。 ![img](https://img-blog.csdnimg.cn/20200414133904290.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

### ACL

zookeeper提供控制节点访问权限的功能，用于有效的保证zookeeper中数据的安全性。避免误操作而导致系统出现重大事故。 CREATE /READ/WRITE/DELETE/ADMIN

## zookeeper 命令操作

create [-s] [-e] path data acl -s 表示节点是否有序 -e 表示是否为临时节点 默认情况下，是持久化节点 get path [watch] 获得指定 path的信息 set path data [version] 修改节点 path对应的data version 表示乐观锁的概念，数据库里面有一个 version 字段去控制数据行的版本号 delete path [version] 删除节点

### stat 信息

```java
# 节点被创建时的事务ID
cZxid = 0x4
ctime = Sat Apr 11 16:57:57 CST 2020
# 节点最后一次被更新的事务ID
mZxid = 0x5
mtime = Sat Apr 11 17:04:47 CST 2020
# 当前节点下的子节点最后一次被修改时的事务ID
pZxid = 0x4
# 子节点的版本号
cversion = 0
# 表示的是当前节点数据的版本号
dataVersion = 1
# 表示acl的版本号，修改节点权限
aclVersion = 0
# 创建临时节点的时候，会有一个sessionId 。 该值存储的就是这个sessionid
ephemeralOwner = 0x0
# 数据值长度
dataLength = 6
# 子节点数
numChildren = 0
```


依赖：

```java
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.8</version>
</dependency>
```

实例：

```java
public class ZooKeeperApiDemo implements Watcher {
    private final static String CONNECT_STRING = "localhost:2181";

    private static CountDownLatch countDownLatch = new CountDownLatch(1);

    private static ZooKeeper zooKeeper;
    private static Stat stat;
    public static void main(String[] args) throws IOException, InterruptedException, KeeperException {
        getZooKeeper();

        // 创建节点
        Stat stat = zooKeeper.exists("/spring", true);
        if (stat == null) {
            String result = zooKeeper.create("/spring", "spring".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
            zooKeeper.getData("/spring", new ZooKeeperApiDemo(), stat);
            System.out.println("创建成功：" + result);
        }

        // 修改数据
        zooKeeper.setData("/spring", "xiaoxiao".getBytes(), -1);
        Thread.sleep(2000);

        // 修改数据
        zooKeeper.setData("/spring", "xiaoxiao123".getBytes(), -1);
        Thread.sleep(2000);

        // 删除数据
        // zooKeeper.delete("/hello", -1);
        // Thread.sleep(2000);

        // 创建节点和子节点
        String path = "/node";
        zooKeeper.create(path, "123".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
        Thread.sleep(1000);
        stat = zooKeeper.exists(path + "/node1", true);
        if (stat == null) {
            zooKeeper.create(path + "/node1", "123".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
            Thread.sleep(1000);
        }

        // 修改子节点
        zooKeeper.setData(path + "/node1", "spring123".getBytes(), -1);
        Thread.sleep(1000);

        // 获取指定节点下的子路径
        List<String> children = zooKeeper.getChildren("/node",true);
        System.out.println(children);

		// 权限的使用
        ACL acl1 = new ACL(ZooDefs.Perms.CREATE, new Id("digest", "root:root"));
        ACL acl2 = new ACL(ZooDefs.Perms.CREATE, new Id("ip", "192.168.1.1"));
        List<ACL> acls = new ArrayList<>();
        acls.add(acl1);
        acls.add(acl2);
        zooKeeper.create("/auth", "123".getBytes(), acls, CreateMode.PERSISTENT);
        // 或者
        zooKeeper.addAuthInfo("digest", "root:root".getBytes());
        zooKeeper.create("/auth","123".getBytes(), ZooDefs.Ids.CREATOR_ALL_ACL, CreateMode.PERSISTENT);

    }

    public static void getZooKeeper() throws IOException, InterruptedException {
        zooKeeper = new ZooKeeper(CONNECT_STRING, 5000, new ZooKeeperApiDemo());
        countDownLatch.await();
        System.out.println(zooKeeper.getState());
    }

    @Override
    public void process(WatchedEvent watchedEvent) {
        // 如果当前的连接状态就是连接成功的，那么通过计数器去控制
        if (watchedEvent.getState() == Event.KeeperState.SyncConnected) {
            if (Event.EventType.None == watchedEvent.getType() && null == watchedEvent.getPath()) {
                countDownLatch.countDown();
                System.out.println(watchedEvent.getState());
            } else if (watchedEvent.getType() == Event.EventType.NodeDataChanged) {
                try {
                    // watch监听只会出发一次，要在此触发需要重新注册，true表示重新注册
                    System.out.println("数据变更路径：" + watchedEvent.getPath() + " -> 改变后的值：" +
                            zooKeeper.getData(watchedEvent.getPath(), true, stat));
                } catch (KeeperException e) {
                    e.printStackTrace();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            // 子节点的数据变化触发
            else if (watchedEvent.getType() == Event.EventType.NodeChildrenChanged) {
                try {
                    System.out.println("子节点数据变更路径：" + watchedEvent.getPath() + " -> 改变后的值：" +
                            zooKeeper.getData(watchedEvent.getPath(), true, stat));
                } catch (KeeperException e) {
                    e.printStackTrace();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            // 创建节点的时候触发
            else if (watchedEvent.getType() == Event.EventType.NodeCreated) {
                try {
                    System.out.println("节点创建路径：" + watchedEvent.getPath() + " -> 节点的值：" +
                            zooKeeper.getData(watchedEvent.getPath(), true, stat));
                } catch (KeeperException e) {
                    e.printStackTrace();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            // 节点删除会触发
            else if (watchedEvent.getType() == Event.EventType.NodeDeleted) {
                System.out.println("节点删除路径：" + watchedEvent.getPath());
            }
            System.out.println(watchedEvent.getType());
        }
    }
}
```

## 权限控制模式

## 连接状态

## 事件类型

## zkclient

依赖：

```java
<dependency>
    <groupId>com.101tec</groupId>
    <artifactId>zkclient</artifactId>
    <version>0.10</version>
</dependency>
```

```java
public class SessionDemo {

    private final static String CONNECT_STRING = "localhost:2181";

    public static void main(String[] args) throws InterruptedException {
        ZkClient zkClient = new ZkClient(CONNECT_STRING, 4000);
        System.out.println(zkClient + " - > success!");

        //zkClient.createEphemeral("/zkclient");
        // zkclient 提供递归创建父节点的功能
        //zkClient.createPersistent("/node/node1/node1-1", true);
        //System.out.println("create success!");

        // 删除节点
        //zkClient.deleteRecursive("/node");

        // 获取子节点
        List<String> list = zkClient.getChildren("/node");
        System.out.println(list);

        // watcher
        zkClient.subscribeDataChanges("/node", new IZkDataListener() {
            @Override
            public void handleDataChange(String s, Object o) throws Exception {
                System.out.println("节点名称：" + s + "-> 节点修改后的值：" + o);
            }

            @Override
            public void handleDataDeleted(String s) throws Exception {

            }
        });

        zkClient.writeData("/node", "node");
        TimeUnit.SECONDS.sleep(2);
        
        zkClient.subscribeChildChanges("/node", new IZkChildListener() {
            @Override
            public void handleChildChange(String s, List<String> list) throws Exception {
                
            }
        });
    }
}
```

## curator

Curator 本身是Netflix公司开源的zookeeper客户端：

- curator 提供了各种应用场景的实现封装 
- curator-framework 提供了fluent风格api 
- curator-replice 提供了实现封装

### curator 连接的重试策略

ExponentialBackoffRetry 衰减重试 RetryNTimes 指定最大重试次数 RetryOneTime 仅重试一次 RetryUnitilElapsed 一直重试直到规定的时间

```java
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>2.11.0</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>2.11.0</version>
</dependency>
```

```java
public class CuratorClientUtils {

    private static CuratorFramework curatorFramework;

    private final static String CONNECT_STRING = "localhost:2181";

    public static CuratorFramework getInstance() {
        // 创建会话的两种方式
        // 1.normal 方式
        curatorFramework = CuratorFrameworkFactory
                .newClient(CONNECT_STRING, 5000, 5000,
                        new ExponentialBackoffRetry(1000, 3));
        // start方法启动连接
        curatorFramework.start();
        System.out.println("success!");
        return curatorFramework;

        // 2.fluent 风格
        //CuratorFramework curatorFramework1 = CuratorFrameworkFactory.builder().connectString(CONNECT_STRING).sessionTimeoutMs(5000)
        //            .retryPolicy(new ExponentialBackoffRetry(1000, 3))
        //            .namespace("/curator").build();
        //curatorFramework1.start();
    }

}
public class CuratorCreateSessionDemo {

    public static void main(String[] args) {
        CuratorFramework curatorFramework = CuratorClientUtils.getInstance();

        try {
            // 创建节点
            //String result = curatorFramework.create().creatingParentsIfNeeded().withMode(CreateMode.PERSISTENT)
                    //.forPath("/curator/curator1/curator1-1", "123".getBytes());

            // 删除节点，默认情况下，version为-1
            //curatorFramework.delete().deletingChildrenIfNeeded().forPath("/node");

            // 更新
            Stat stat1 = curatorFramework.setData().forPath("/curator", "123".getBytes());
            System.out.println(stat1);

            // 查询
            Stat stat = new Stat();
            byte[] bytes = curatorFramework.getData().storingStatIn(stat).forPath("/curator");
            System.out.println(new String(bytes) + "-->stat:" + stat);

			// 异步操作
            ExecutorService service = Executors.newFixedThreadPool(1);
            CountDownLatch countDownLatch = new CountDownLatch(1);
            curatorFramework.create().creatingParentsIfNeeded().withMode(CreateMode.PERSISTENT)
                    .inBackground(new BackgroundCallback() {
                        @Override
                        public void processResult(CuratorFramework curatorFramework, CuratorEvent curatorEvent) throws Exception {
                            System.out.println(Thread.currentThread().getName() + "->resultCode:" + curatorEvent.getResultCode()
                                    + "->" + curatorEvent.getType());
                            countDownLatch.countDown();
                        }
                    },service).forPath("/spring", "123".getBytes());
            countDownLatch.await();
            service.shutdownNow();

            // 事务操作（curator独有）
            Collection<CuratorTransactionResult> results = curatorFramework.inTransaction().create().forPath("/trans", "111".getBytes())
                    .and().setData().forPath("/xxxx", "111".getBytes()).and().commit();
            for (CuratorTransactionResult result : results) {
                System.out.println(result.getForPath() + "->" + result.getType());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### 三种 watcher 节点监听

```java
public class CuratorEventDemo {
    /**
     * 三种watcher来做节点的监听
     * patcache 监听一个路径下子节点的创建，删除，节点数据更新
     * NodeCache 监听一个节点的创建，更新，删除
     * TreeCache patcache+nodecache 的合体（监视路径下的创建，更新，删除时间），缓存路径下的所有子节点数据
     */
    public static void main(String[] args) throws Exception {
        CuratorFramework curatorFramework = CuratorClientUtils.getInstance();

        // NodeCache，节点变化
        NodeCache cache = new NodeCache(curatorFramework, "/curator", false);
        cache.start(true);

        cache.getListenable().addListener(() -> System.out.println("节点数据发生变化，变化后的结果："
            + new String(cache.getCurrentData().getData())));
        curatorFramework.setData().forPath("/curator", "xiaoxiao".getBytes());

        // PathChildrenCache
        PathChildrenCache pathChildrenCache = new PathChildrenCache(curatorFramework, "/event", true);
        pathChildrenCache.start(PathChildrenCache.StartMode.POST_INITIALIZED_EVENT);
        // NORMAL,BUILD_INITIAL_CACHE,POST_INITIALIZED_EVENT

        pathChildrenCache.getListenable().addListener((curatorFramework1, pathChildrenCacheEvent) -> {
            switch (pathChildrenCacheEvent.getType()) {
                case CHILD_ADDED:
                    System.out.println("增加子节点");
                    break;
                case CHILD_REMOVED:
                    System.out.println("删除子节点");
                    break;
                case CHILD_UPDATED:
                    System.out.println("更新子节点");
                    break;
                default:break;
            }
        });
        curatorFramework.create().withMode(CreateMode.PERSISTENT).forPath("/event", "event".getBytes());
        TimeUnit.SECONDS.sleep(1);


        curatorFramework.create().withMode(CreateMode.EPHEMERAL).forPath("/event/event1", "1".getBytes());
        TimeUnit.SECONDS.sleep(1);

        curatorFramework.setData().forPath("/event/event1", "222".getBytes());
        TimeUnit.SECONDS.sleep(1);

        curatorFramework.delete().forPath("/event/event1");
        System.in.read();
    }
}
```


- 订阅发布 
 <ul> 
  - watcher机制 
  - 统一配置管理（disconf） 
 </ul>  
- 分布式锁 
 <ul> 
  - redis 
  - zookeeper 
  - 数据库 
 </ul>  
- 负载均衡 
- ID生成器 
- 分布式队列 
- 统一命名服务 
- master选举

## 数据发布订阅/配置中心

实现配置信息的集中式管理和数据的动态更新 实现配置中心有两种模式：push、pull zookeeoer 采用的是推拉相结合的方式。客户端想服务器注册自己需要关注的节点。一旦节点数据发生变化，那么服务器端会向客户端发送 watcher 事件通知。客户端发送通知后，主动到服务器端获取更新后的数据。

1. 数据量比较小 
2. 数据内容在运行时会发生动态变更 
3. 集群中的各个机器共享配置


![img](https://img-blog.csdnimg.cn/20200413133304132.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

## 负载均衡

请求/数据分摊多个计算机单元上

## 分布式锁

通常实现分布式锁有几种方式：

1. redis，setNX 存在则会返回0 
2. 数据方式去实现 创建一个表， 通过索引唯一的方式

```java
create table (id , methodname …)   # methodname增加唯一索引
insert # 需要获取锁时候，先往表中添加一条数据，方法名XXX
delete # 使用完方法后，删除这条记录
```

另外一种方式，innodb 可以设置行数，mysql for update



1. zookeeper 实现 排他锁 ![img](https://img-blog.csdnimg.cn/20200413134955604.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) 共享锁（读锁） ![img](https://img-blog.csdnimg.cn/20200413135311227.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) java api实现：

```java
public class ZookeeperClient {
    private final static String CONNECT_STRING = "192.168.174.128:2181";
    private static int sessionTimeout = 5000;

    //获取连接
    public static ZooKeeper getInstance() throws IOException, InterruptedException {
        final CountDownLatch conectStatus=new CountDownLatch(1);
        ZooKeeper zooKeeper=new ZooKeeper(CONNECT_STRING, sessionTimeout, new Watcher() {
            public void process(WatchedEvent event) {
                if(event.getState()== Event.KeeperState.SyncConnected){
                    conectStatus.countDown();
                }
            }
        });
        conectStatus.await();
        return zooKeeper;
    }

    public static int getSessionTimeout() {
        return sessionTimeout;
    }
}
public class LockWatcher implements Watcher{

    private CountDownLatch latch;

    public LockWatcher(CountDownLatch latch) {
        this.latch = latch;
    }

    public void process(WatchedEvent event) {
        if(event.getType()== Event.EventType.NodeDeleted){
            latch.countDown();
        }
    }
}
public class DistributeLock {

    // 根节点
    private static final String ROOT_LOCKS = "/LOCKS";

    private ZooKeeper zooKeeper;

    // 会话超时时间
    private int sessionTimeout;

    // 记录所节点id
    private String lockID;

    // 节点数据
    private final static byte[] data = {1,2};

    private CountDownLatch countDownLatch = new CountDownLatch(1);

    public DistributeLock() throws IOException, InterruptedException {
        this.zooKeeper = ZookeeperClient.getInstance();
        this.sessionTimeout = ZookeeperClient.getSessionTimeout();
    }

    // 获取锁的方法
    public boolean lock() {
        try {
            lockID = zooKeeper.create(ROOT_LOCKS + "/", data, ZooDefs.Ids.OPEN_ACL_UNSAFE,
                    CreateMode.EPHEMERAL_SEQUENTIAL);
            System.out.println(Thread.currentThread().getName() + "->成功创建了lock节点【" + lockID + "】，开始去竞争锁");
            // 获取根节点下的所有子节点
            List<String> childrenNodes = zooKeeper.getChildren(ROOT_LOCKS, true);
            // 排序，从小到大排序
            SortedSet<String> sortedSet = new TreeSet<>();
            for (String children : childrenNodes) {
                sortedSet.add(ROOT_LOCKS + "/" + children);
            }
            // 拿到最小的节点
            String first = sortedSet.first();
            if (lockID.equals(first)) {
                // 表示当前就是最小的节点
                System.out.println(Thread.currentThread().getName() + "->成功获得锁，lock节点为：【" + lockID + "】");
                return true;
            }
            SortedSet<String> lessThanLockId = sortedSet.headSet(lockID);
            if (!lessThanLockId.isEmpty()) {
                // 拿到比当前LockID这个节点更小的上一个节点
                String prevLockID = lessThanLockId.last();
                zooKeeper.exists(prevLockID, new LockWatcher(countDownLatch));
                // 如果会话超时或者节点被释放
                countDownLatch.await(sessionTimeout, TimeUnit.MILLISECONDS);
                System.out.println(Thread.currentThread().getName() + "->成功获得锁：【" + lockID + "】");
            }
            return true;
        } catch (KeeperException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return false;
    }

    public boolean unlock() {
        System.out.println(Thread.currentThread().getName() + "->开始释放锁：【" + lockID + "】");
        try {
            zooKeeper.delete(lockID, -1);
            System.out.println("节点【" + lockID + "】成功被删除");
            return true;
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (KeeperException e) {
            e.printStackTrace();
        }
        return false;
    }

    public static void main(String[] args) {
        final CountDownLatch latch = new CountDownLatch(10);
        Random random = new Random();
        for(int i = 0; i < 10; i++){
            new Thread(() -> {
                DistributeLock lock = null;
                try {
                    lock = new DistributeLock();
                    latch.countDown();
                    latch.await();
                    lock.lock();
                    Thread.sleep(random.nextInt(500));
                } catch (IOException e) {
                    e.printStackTrace();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    if(lock != null){
                        lock.unlock();
                    }
                }
            }).start();
        }
    }
}
```

## master 选举

7*24小时可用， 99.999%可用 master-slave 模式 使用zookeeper解决

### ZkClient 实现

```java
public class UserCenter implements Serializable {
    private static final long serialVersionUID = 8704729764515013279L;
    // 机器信息
    private int mc_id;

    // 机器名
    private String mc_name;

    public int getMc_id() {
        return mc_id;
    }

    public void setMc_id(int mc_id) {
        this.mc_id = mc_id;
    }

    public String getMc_name() {
        return mc_name;
    }

    public void setMc_name(String mc_name) {
        this.mc_name = mc_name;
    }

    @Override
    public String toString() {
        return "UserCenter{" +
                "mc_id=" + mc_id +
                ", mc_name='" + mc_name + '\'' +
                '}';
    }
}
public class MasterSelector {
    private ZkClient zkClient;

    // 需要争抢的节点
    private final static String MASTER_PATH = "/master";

    // 注册节点内容变化
    private IZkDataListener dataListener;

    // 其他服务器
    private UserCenter server;

    // master服务器
    private UserCenter master;

    private static boolean isRunning = false;

    ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(1);

    public MasterSelector(UserCenter server, ZkClient zkClient) {
        System.out.println("["+server+"] 去争抢master权限");
        this.server = server;
        this.zkClient = zkClient;

        this.dataListener = new IZkDataListener() {
            @Override
            public void handleDataChange(String s, Object o) throws Exception {

            }

            @Override
            public void handleDataDeleted(String s) throws Exception {
                // 节点如果被删除，发起选主操作
                chooseMaster();
            }
        };
    }

    public void start() {
        // 开始选举
        if (!isRunning) {
            isRunning = true;
            // 注册节点事件
            zkClient.subscribeDataChanges(MASTER_PATH, dataListener);
            chooseMaster();
        }
    }

    public void stop() {
        // 停止选举
        if (isRunning) {
            isRunning = false;
            scheduledExecutorService.shutdown();
            zkClient.unsubscribeDataChanges(MASTER_PATH, dataListener);
            releaseMaster();
        }
    }

    // 具体选master的实现逻辑
    private void chooseMaster() {
        if (!isRunning) {
            System.out.println("当前服务没有启动");
            return;
        }
        try {
            zkClient.createEphemeral(MASTER_PATH, server);
            // 把server节点赋值给master
            master = server;
            System.out.println(master.getMc_name() + "->我现在已经是master，你们要听我的");

            // 定时器
            // master释放（master 出现故障）
            scheduledExecutorService.schedule(this::releaseMaster, 5, TimeUnit.SECONDS);
        } catch (ZkNodeExistsException e) {
            // master已经存在
            UserCenter userCenter = zkClient.readData(MASTER_PATH, true);
            if (userCenter == null) {
                chooseMaster();
            } else {
                master = userCenter;
            }

        }

    }

    private void releaseMaster() {
        // 释放锁（故障模拟）
        // 判断当前是不是master，只有master才需要释放
        if (checkIsMaster()) {
            zkClient.deleteRecursive(MASTER_PATH);
        }
    }

    private boolean checkIsMaster() {
        // 判断当前是不是master
        UserCenter userCenter = zkClient.readData(MASTER_PATH);
        if (userCenter.getMc_name().equals(server.getMc_name())) {
            master = userCenter;
            return true;
        }
        return false;
    }
}
public class MasterChoseTest {

    private final static String CONNECT_STRING = "localhost:2181";

    public static void main(String[] args) {
        List<MasterSelector> selectorList = new ArrayList<>();
        try {
            for (int i = 0; i < 10; i++) {
                ZkClient zkClient = new ZkClient(CONNECT_STRING, 5000, 5000, new SerializableSerializer());
                UserCenter userCenter = new UserCenter();
                userCenter.setMc_id(i);
                userCenter.setMc_name("客户端-" + i);

                MasterSelector selector = new MasterSelector(userCenter, zkClient);
                selectorList.add(selector);
                // 触发选举操作
                selector.start();
                TimeUnit.SECONDS.sleep(4);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            for (MasterSelector masterSelector : selectorList) {
                masterSelector.stop();
            }
        }
    }
}
```

### curator 实现

LeaderLatch 写一个master LeaderSelector 每一个应用都写一个临时有序节点，根据最小的节点来获得优先权

```java
public class MasterSelector {
    private final static String CONNECT_STRING = "localhost:2181";

    private final static String MASTER_PATH = "/curator_master_path";

    public static void main(String[] args) {
        CuratorFramework curatorFramework = CuratorFrameworkFactory.builder().connectString(CONNECT_STRING)
                .retryPolicy(new ExponentialBackoffRetry(1000, 3)).build();

        LeaderSelector leaderSelector = new LeaderSelector(curatorFramework, MASTER_PATH, new LeaderSelectorListenerAdapter() {
            @Override
            public void takeLeadership(CuratorFramework curatorFramework) throws Exception {
                System.out.println("获得leader成功");
                TimeUnit.SECONDS.sleep(2);
            }
        });
        leaderSelector.autoRequeue();
        // 开始选举
        leaderSelector.start();
    }
}
```


## zookeeper集群角色

### leader

leader是zookeeper集群的核心。

1. 事务请求的唯一调度者和处理者，保证集群事务处理的顺序性 
2. 集群内部各个服务器的调度者

### follower

1. 处理客户端非事务请求，以及转发事务请求给leader服务器 
2. 参与事务请求提议（proposal）的投票（客户端的一个事务请求，需要半数服务器投票通过以后才能通知leader commit；leader会发起一个提案，要求follower投票） 
3. 参与leader选举的投票

### observer

观察zookeeper集群中最新状态的变化并将这些状态同步到observer服务器上 增加observer不影响集群中事务处理能力，同时还能提升集群的非事务处理能力

## zookeeper 的集群组成

zookeeper 一般是由 2n+1 台服务器组成

### leader选举

leaderElection，AuthFastLeaderElection，FastLeaderElection(默认方式) org.apache.zookeeper.server.quorum.QuorumPeer#startLeaderElection

源码地址：https://github.com/apache/zookeeper.git 需要的条件： jdk 1.7以上 、ant 、idea

#### FastLeaderElection

serverid：在配置server集群的时候，给定服务器的标识id（myid） zxid：服务器在运行时产生的数据ID， zxid的值越大，表示数据越新 Epoch：选举的轮数 server的状态：Looking、 Following、Observering、Leading

第一次初始化启动的时候： LOOKING


1. 所有在集群中的server都会推荐自己为leader，然后把（myid、zxid、epoch）作为广播信息，广播给集群中的其他server, 然后等待其他服务器返回 
2. 每个服务器都会接收来自集群中的其他服务器的投票。集群中的每个服务器在接受到投票后，开始判断投票的有效性 a)判断逻辑时钟(Epoch) ，如果Epoch大于自己当前的Epoch，说明自己保存的Epoch是过期。更新Epoch，同时clear其他服务器发送过来的选举数据。判断是否需要更新当前自己的选举情况 b)如果Epoch小于目前的Epoch，说明对方的epoch过期了，也就意味着对方服务器的选举轮数是过期的。这个时候，只需要讲自己的信息发送给对方 ![img](https://img-blog.csdnimg.cn/2020041410234291.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

## ZAB 协议

拜占庭将军问题 paxos 协议主要就是如何保证在分布式环网络环境下，各个服务器如何达成一致最终保证数据的一致性问题。 ZAB 协议基于paxos协议的一个改进。

ZAB 协议为分布式协调服务 zookeeper 专门设计的一种支持崩溃恢复的原子广播协议。 zookeeper 并没有完全采用 paxos 算法， 而是采用 zab（Zookeeper atomic broadcast）。

### ZAB 协议原理

1. 在zookeeper 的主备模式下，通过zab协议来保证集群中各个副本数据的一致性 
2. zookeeper使用的是单一的主进程来接收并处理所有的事务请求，并采用zab协议，把数据的状态变更以事务请求的形式广播到其他的节点 
3. zab协议在主备模型架构中，保证了同一时刻只能有一个主进程来广播服务器的状态变更 
4. 所有的事务请求必须由全局唯一的服务器来协调处理，这个的服务器叫leader，其他的叫follower leader节点主要负责把客户端的事务请求转化成一个事务提议（proposal），并分发给集群中的所有follower节点 再等待所有follower节点的反馈。一旦超过半数服务器进行了正确的反馈，那么leader就会commit这条消息

### ZAB 协议工作原理

1. 什么情况下zab协议会进入崩溃恢复模式？ 当服务器启动时； 当leader服务器出现网络中断、崩溃或者重启的情况； 集群中已经不存在过半的服务器与该leader保持正常通信； 
<li>zab协议进入崩溃恢复模式会做什么？ 当leader出现问题，zab协议进入崩溃恢复模式，并且选举出新的leader。当新的leader选举出来以后，如果集群中已经有过半机器完成了leader服务器的状态同（数据同步），退出崩溃恢复，进入消息广播模式； 当新的机器加入到集群中的时候，如果已经存在leader服务器，那么新加入的服务器就会自觉进入数据恢复模式，找到leader进行数据同步；</li> 
<li>假设一个事务在leader服务器被提交了，并且已经有过半的follower返回了ack。 在leader节点把commit消息发送给folower机器之前 leader服务器挂了怎么办？ zab协议，一定需要保证已经被leader提交的事务也能够被所有follower提交 zab协议需要保证，在崩溃恢复过程中跳过哪些已经被丢弃的事务</li>


![img](https://img-blog.csdnimg.cn/20200414105311225.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)


内存数据和磁盘数据

zookeeper会定时把数据存储在磁盘上。 DataDir，存储的是数据的快照（快照： 存储某一个时刻全量的内存数据内容）

DataLogDir，存储事务日志，文件名为log.zxid 查看事务日志的命令：

```java
java -cp :/usr/local/zookeeper-3.4.10/lib/slf4j-api-1.6.1.jar:/usr/local/zookeeper-3.4.10/zookeeper-3.4.10.jar org.apache.zookeeper.server.LogFormatter log.200000001
```

## zookeeper 有三种日志

zookeeper.out，运行日志 快照，存储某一时刻的全量数据 事务日志，事务操作的日志记录

