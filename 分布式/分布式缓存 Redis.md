# redis 基础

## 存储结构

1. String 字符类型（简单动态字符串（Simple Dynamic String），简称SDS）

   ```c
   struct sdshdr{
       //  记录已使用长度
       int len;
       // 记录空闲未使用的长度
       int free;
       // 字符数组
       char[] buf;
   };
   ```

   作为常规的key-value缓存应用。例如微博数、粉丝数等，一个键最大能存储512MB

2. 散列类型 ， 主要用来存储对象信息

3. 列表类型 ，比如twitter的关注列表，粉丝列表等都可以用Redis的list结构来实现 

4. 集合类型 ， 交集，并集，差集 

5. 有序集合，排行榜

## 功能

1. 可以为每个key设置超时时间 
2. 可以通过列表类型来实现分布式队列的操作 
3. 支持发布订阅的消息模式

## 简单

提供了很多命令与redis进行交互

## redis的应用场景

1. 数据缓存（商品数据、新闻、热点数据） 
2. 单点登录 
3. 秒杀、抢购 
4. 网站访问排名 
5. 应用的模块开发

## redis的安装

1. 下载redis安装包 
2. tar -zxvf 安装包 
3. 在redis目录下 执行 make 
4. 可以通过make test测试编译状态 
5. make install [prefix=/path]完成安装

### 启动停止 redis

```java
./redis-server ../redis.conf
./redis-cli shutdown
```

以后台进程的方式启动，修改redis.conf daemonize =yes

### 连接到redis的命令

```java
./redis-cli
 ./redis-cli -h 127.0.0.1 -p 6379
```

### 其他命令说明

Redis-server 启动服务 

Redis-cli 访问到redis的控制台 

redis-benchmark 性能测试的工具 

redis-check-aof aof文件进行检测的工具 

redis-check-dump rdb文件检查工具 

redis-sentinel sentinel 服务器配置

## 多数据支持

默认支持16个数据库，可以理解为一个命名空间。跟关系型数据库不一样的点：

1. redis不支持自定义数据库名词 
2. 每个数据库不能单独设置授权 
3. 每个数据库之间并不是完全隔离的。 可以通过flushall命令清空redis实例面的所有数据库中的数据

通过select dbid去选择不同的数据库命名空间，dbid的取值范围默认是0 -15。

# redis 使用入门


1. 获得一个符合匹配规则的键名列表

```java
keys pattern ? * []
```

1. 判断一个键是否存在 ， EXISTS key 
2. type key 去获得这个key的数据结构类型

## 各种数据结构的使用

### key 的设计

对象类型:对象id:对象属性:对象子属性，建议对key进行分类，同步在wiki统一管理

### 字符类型

一个字符类型的key默认存储的最大容量是512M

set key value 赋值 

get key 取值 

incr key 递增数字 

incryby key increment 递增指定的整数 

decr key 原子递减 

append key value 向指定的key追加字符串 

strlen key 获得key对应的value的长度 

mget key [key...] 同时获得多个key的value 

mset key value [key value …] 同时设置多个key的value 

setnx 只在键 key 不存在的情况下， 将键 key 的值设置为 value 。若键 key 已经存在， 则 SETNX 命令不做任何动作。SETNX 是『SET if Not eXists』(如果不存在，则 SET)的简写。命令在设置成功时返回 1 ， 设置失败时返回 0 。

应用场景：短信重发机制：sms:limit:mobile 138… expire （incr）

### 列表类型

list, 可以存储一个有序的字符串列表 

lpush/rpush key value [value…] 从左边或者右边push数据 

llen num 获得列表的长度 

lrange key start stop 索引可以是负数， -1表示最右边的第一个元素 

lrem key count value 根据参数 COUNT 的值，移除列表中与参数 VALUE 相等的元素。count 的值可以是以下几种： 

count &gt; 0 : 从表头开始向表尾搜索，移除与 VALUE 相等的元素，数量为 COUNT 。 

count &lt; 0 : 从表尾开始向表头搜索，移除与 VALUE 相等的元素，数量为 COUNT 的绝对值。 

count = 0 : 移除表中所有与 VALUE 相等的值。

lset key index value 设置索引为 index 的值 

lpop/rpop 取数据(并移除)

应用场景：可以用来做分布式消息队列

### 散列类型

不支持数据类型的嵌套，比较适合存储对象 

hset key field value 设置值 

hget key filed 取值 

hmset key filed value [filed value …] 一次性设置多个值 

hmget key field [field …] 一次性获得多个值 

hgetall key 获得hash的所有信息，包括key和value 

hexists key field 判断字段是否存在。 存在返回1. 不存在返回0 

hincryby/hsetnx hdel key field [field…] 删除一个或者多个字段

### 集合类型

set 跟list 不一样的点。 集合类型不能存在重复的数据，而且是无序的。 

sadd key member [member...] 增加数据； 如果value已经存在，则会忽略存在的值，并且返回成功加入的元素的数量 

srem key member 删除元素 

smembers key 获得所有数据 

sdiff key [key…]对多个集合执行差集运算 

sunion key [key…] 对多个集合执行并集操作, 同时存在在两个集合里的所有值

### 有序集合

zadd key score member 

zrange key start stop [withscores] 去获得元素。 withscores是可以获得元素的分数，如果两个元素的score是相同的话，那么根据(0&lt;9&lt;A&lt;Z&lt;a&lt;z) 方式从小到大。 

应用场景：网站访问的前10名

# redis 事务处理

MULTI 去开启事务 

EXEC 去执行事务

运行的错误，只有在执行的过程中才能知道的问题，是不会回滚的。

```java
127.0.0.1:6379> hset person age 18
(integer) 1
127.0.0.1:6379> multi
OK
127.0.0.1:6379> sadd member spring java redis
QUEUED
127.0.0.1:6379> rpush person 22 # 错误的对集合类型的数据使用列表操作
QUEUED
127.0.0.1:6379> sadd password 23 32 322
QUEUED
127.0.0.1:6379> exec
1) (integer) 3
2) (error) WRONGTYPE Operation against a key holding the wrong kind of value
3) (integer) 3
127.0.0.1:6379> smembers member # 设置集合操作未回滚
1) "spring"
2) "java"
3) "redis"
```

## 过期时间

``expire key seconds ``

``ttl key`` 获得key的过期时间

## 发布订阅

publish channel message 

subscribe [channel…]

# redis 分布式锁

## 分布式锁的实现

锁是用来解决什么问题的：

1. 一个进程中的多个线程，多个线程并发访问同一个资源的时候，如何解决线程安全问题。 
2. 一个分布式架构系统中的两个模块同时去访问一个文件对文件进行读写操作 
3. 多个应用对同一条数据做修改的时候，如何保证数据的安全性

在进程中，我们可以用到synchronized、lock之类的同步操作去解决，但是对于分布式架构下多进程的情况下，如何做到跨进程的锁。就需要借助一些第三方手段来完成。

## 分布式锁的解决方案

### 数据库

通过唯一约束

```java
lock(
  id  int(11)
  methodName  varchar(100)
  memo varchar(1000) 
  modifyTime timestamp
  unique key mn (methodName)  --唯一约束
)
```

获取锁的伪代码

```java
try{
	exec insert into lock(methodName,memo) values(‘method’,’desc’);
	return true;
}Catch(DuplicateException e){
	return false;
}
```

释放锁

```java
delete from lock where methodName=''
```

#### 存在的需要思考的问题

1. 锁没有失效时间，一旦解锁操作失败，就会导致锁记录一直在数据库中，其他线程无法再获得到锁 
2. 锁是非阻塞的，数据的insert操作，一旦插入失败就会直接报错。没有获得锁的线程并不会进入排队队列，要想再次获得锁就要再次触发获得锁操作 
3. 锁是非重入的，同一个线程在没有释放锁之前无法再次获得该锁

### zookeeper实现分布式锁

利用 zookeeper 的唯一节点特性或者有序临时节点特性获得最小节点作为锁. zookeeper 的实现相对简单，通过curator客户端，已经对锁的操作进行了封装，原理如下： 

![img](分布式缓存 Redis.assets/20200611112108888.png)

#### zookeeper 的优势

1. 可靠性高、实现简单 
2. zookeeper因为临时节点的特性，如果因为其他客户端因为异常和zookeeper连接中断了，那么节点会被删除，意味着锁会被自动释放 
3. zookeeper本身提供了一套很好的集群方案，比较稳定 
4. 释放锁操作，会有watch通知机制，也就是服务器端会主动发送消息给客户端这个锁已经被释放了

### 基于缓存的分布式锁实现

redis中有一个setNx命令，这个命令只有在key不存在的情况下为key设置值。所以可以利用这个特性来实现分布式锁的操作。

1. 添加依赖

```XML
<dependency>
  <groupId>redis.clients</groupId>
  <artifactId>jedis</artifactId>
  <version>2.9.0</version>
</dependency>
```

1. 编写 redis 连接的代码

```java
public class RedisManager {

    private static JedisPool jedisPool;

    static {
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        jedisPoolConfig.setMaxTotal(20);
        jedisPoolConfig.setMaxIdle(10);
        jedisPool = new JedisPool(jedisPoolConfig, "127.0.0.1", 6379, 10000, "1234");

    }

    public static Jedis getJedis() throws Exception {
        if (null != jedisPool) {
            return jedisPool.getResource();
        }
        throw new Exception("Jedispool was not init");
    }
}
```

1. 分布式锁的具体实现

```java
public class RedisLock {

    public String getLock(String key, int timeout) {
        try {
            Jedis jedis = RedisManager.getJedis();
            String value = UUID.randomUUID().toString();

            long end = System.currentTimeMillis() + timeout;
            // 阻塞
            while (System.currentTimeMillis() < end) {
                if (jedis.setnx(key, value) == 1) {
                    jedis.expire(key, timeout);
                    // 锁设置成功，redis 操作成功
                    return value;
                }
                if (jedis.ttl(key) == -1) {
                    jedis.expire(key, timeout);
                }
                Thread.sleep(1000);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    public boolean releaseLock(String key, String value) {
        try {
            Jedis jedis = RedisManager.getJedis();
            while (true) {
                jedis.watch(key);
                // 判断获得锁的线程和当前 redis 中存的锁是同一个
                if (value.equals(jedis.get(key))) {
                    Transaction transaction = jedis.multi();
                    transaction.del(key);

                    List<Object> list = transaction.exec();
                    if (list == null) {
                        continue;
                    }
                    return true;
                }
                jedis.unwatch();
                break;
            }

        } catch (Exception e) {
            e.printStackTrace();
        }
        return false;
    }

    public static void main(String[] args) {
        RedisLock lock = new RedisLock();
        String lockId = lock.getLock("lock:first", 10000);
        if (null != lockId) {
            System.out.println("获得锁成功");
        }

        String lockId1 = lock.getLock("lock:first", 10000);
        System.out.println(lockId1);
    }
}
```

# redis 多路复用机制

linux的内核会把所有外部设备都看作一个文件来操作，对一个文件的读写操作会调用内核提供的系统命令，返回一个 ``file descriptor``（文件描述符）。对于一个socket的读写也会有响应的描述符，称为 socketfd (socket 描述符)。而IO多路复用是指内核一旦发现进程指定的一个或者多个文件描述符IO条件准备好以后就通知该进程。 IO多路复用又称为事件驱动，操作系统提供了一个功能，当某个socket可读或者可写的时候，它会给一个通知。当配合非阻塞socket使用时，只有当系统通知我哪个描述符可读了，我才去执行read操作，可以保证每次read都能读到有效数据。操作系统的功能通过select/pool/epoll/kqueue之类的系统调用函数来使用，这些函数可以同时监视多个描述符的读写就绪情况，这样多个描述符的I/O操作都能在一个线程内并发交替完成，这就叫I/O多路复用，这里的复用指的是同一个线程。 多路复用的优势在于用户可以在一个线程内同时处理多个socket的 io请求。达到同一个线程同时处理多个IO请求的目的。而在同步阻塞模型中，必须通过多线程的方式才能达到目的。

# lua脚本

Lua是一个高效的轻量级脚本语言，用标准C语言编写并以源代码形式开放， 其设计目的是为了嵌入应用程序中，从而为应用程序提供灵活的扩展和定制功能。 使用脚本的好处：

1. 减少网络开销，在Lua脚本中可以把多个命令放在同一个脚本中运行 
2. 原子操作，redis会将整个脚本作为一个整体执行，中间不会被其他命令插入。换句话说，编写脚本的过程中无需担心会出现竞态条件 
3. 复用性，客户端发送的脚本会永远存储在redis中，这意味着其他客户端可以复用这一脚本来完成同样的逻辑

### Lua在linux中的安装

到官网下载lua的tar.gz的源码包

```java
tar -zxvf lua-5.3.0.tar.gz
# 进入解压的目录
cd lua-5.2.0
# linux环境下编译
make linux  
make install
```

如果报错：lua.c:80:10: fatal error: readline/readline.h: 没有那个文件或目录

```java
yum -y install readline-devel ncurses-devel
# 安装完以后
make linux
make install
```

最后，直接输入 lua 命令即可进入lua的控制台

## Redis与Lua

在Lua脚本中调用Redis命令，可以使用redis.call函数调用。比如我们调用string类型的命令：

```java
redis.call(‘set’,’hello’,’world’)
```

redis.call 函数的返回值就是redis命令的执行结果。前面我们介绍过redis的五种类型的数据返回的值的类型也都不一样。redis.call函数会将这五种类型的返回值转化对应的Lua的数据类型。

### 从Lua脚本中获得返回值

在很多情况下我们都需要脚本可以有返回值，在脚本中可以使用return 语句将值返回给redis客户端，通过return语句来执行，如果没有执行return，默认返回为nil。

### 如何在redis中执行lua脚本

Redis提供了EVAL命令可以使开发者像调用其他Redis内置命令一样调用脚本。

```java
[EVAL]  [脚本内容] [key参数的数量]  [key …] [arg …]
```

可以通过key和arg这两个参数向脚本中传递数据，他们的值可以在脚本中分别使用KEYS和ARGV 这两个类型的全局变量访问。比如我们通过脚本实现一个set命令，通过在redis客户端中调用，那么执行的语句是：

```java
# KEYS和ARGV必须大写  
eval "return redis.call('set',KEYS[1],ARGV[1])" 1 hello world
```

lua脚本的内容为： EVAL命令是根据 key参数的数量，也就是上面例子中的 1 来将后面所有参数分别存入脚本中KEYS和ARGV两个表类型的全局变量。当脚本不需要任何参数时也不能省略这个参数。如果没有参数则为0。

```java
eval "return redis.call('get','hello')" 0
```

### EVALSHA命令

考虑到我们通过eval执行lua脚本，脚本比较长的情况下，每次调用脚本都需要把整个脚本传给redis，比较占用带宽。为了解决这个问题，redis提供了EVALSHA命令允许开发者通过脚本内容的SHA1摘要来执行脚本。该命令的用法和EVAL一样，只不过是将脚本内容替换成脚本内容的SHA1摘要。

1. Redis在执行EVAL命令时会计算脚本的SHA1摘要并记录在脚本缓存中 
2. 执行EVALSHA命令时Redis会根据提供的摘要从脚本缓存中查找对应的脚本内容，如果找到了就执行脚本，否则返回NOSCRIPT No matching script,Please use EVAL

通过以下案例来演示EVALSHA命令的效果

```java
# 将脚本加入缓存并生成sha1命令
127.0.0.1:6379> script load "return redis.call('get','hello')"
"fc1c184a3f0f566037349a820acef20fc7ece599"
127.0.0.1:6379> evalsha "fc1c184a3f0f566037349a820acef20fc7ece599" 0
"world"
```

我们在调用eval命令之前，先执行evalsha命令，如果提示脚本不存在，则再调用eval命令。

### lua脚本实战

实现一个针对某个手机号的访问频次， 以下是lua脚本，保存为phone_limit.lua：

```java
local num=redis.call('incr',KEYS[1])
if tonumber(num)==1 then
   redis.call('expire',KEYS[1],ARGV[1])
   return 1
elseif tonumber(num)>tonumber(ARGV[2]) then
   return 0
else
   return 1
end
```

通过如下命令调用

```java
./redis-cli --eval phone_limit.lua rate.limiting:13700000000 , 10 3
```

语法为 ./redis-cli –eval [lua脚本] [key…]空格,空格[args…]

```java
public class LuaDemo {
    public static void main(String[] args) throws Exception {
        Jedis jedis = RedisManager.getJedis();

        String lua = "local num=redis.call('incr',KEYS[1])\n" +
                "if tonumber(num)==1 then\n" +
                " redis.call('expire',KEYS[1],ARGV[1])\n" +
                " return 1\n" +
                "elseif tonumber(num)>tonumber(ARGV[2]) then\n" +
                " return 0\n" +
                "else\n" +
                " return 1\n" +
                "end";
        String luaSha = jedis.scriptLoad(lua);
        System.out.println(luaSha);// e3ecace37b338749e6cfb3b6901013836f396bdc
        List<String> keys = new ArrayList<>();
        keys.add("ip:limit:127.0.0.1");
        List<String> argvs = new ArrayList<>();
        argvs.add("6000");
        argvs.add("10");
        // Object obj = jedis.eval(lua, keys, argvs);
        // 先让 redis 缓存，redis重启后，缓存会消失
        // Object obj = jedis.evalsha("e3ecace37b338749e6cfb3b6901013836f396bdc", keys, argvs);
        Object obj = jedis.evalsha(luaSha, keys, argvs);
        System.out.println(obj);
    }
}
```

### 脚本的原子性

redis的脚本执行是原子的，即脚本执行期间Redis不会执行其他命令。所有的命令必须等待脚本执行完以后才能执行。为了防止某个脚本执行时间过程导致Redis无法提供服务。Redis提供了lua-time-limit参数限制脚本的最长运行时间。默认是5秒钟。 当脚本运行时间超过这个限制后，Redis将开始接受其他命令但不会执行（以确保脚本的原子性），而是返回BUSY的错误实践操作。

打开两个客户端窗口，在第一个窗口中执行lua脚本的死循环eval “while true do end” 0 在第二个窗口中运行get hello

最后第二个窗口的运行结果是Busy, 可以通过script kill命令终止正在执行的脚本。如果当前执行的lua脚本对redis的数据进行了修改，比如（set）操作，那么script kill命令没办法终止脚本的运行，因为要保证lua脚本的原子性。如果执行一部分终止了，就违背了这一个原则。在这种情况下，只能通过 shutdown nosave命令强行终止。

# redis 持久化机制

redis 提供了两种持久化策略

## RDB

``RDB（Redis DataBase）``的持久化策略：  在指定的时间间隔内生成数据集的时间点快照 

``snapshot 快照`` bin/dump.rdb 

redis在指定的情况下会触发快照：

1.自己配置的快照规则``save <seconds> <changes>``

```java
# 默认配置
# 当在900秒内被更改的key的数量大于1的时候，就执行快照
save 900 1  
save 300 10
save 60 10000
```

2.save 或者 bgsave 

save 执行内存的数据同步到磁盘的操作，这个操作会阻塞客户端的请求 

bgsave 在后台异步执行快照操作，这个操作不会阻塞客户端的请求 

3.执行 flushall 的时候 清除内存的所有数据，只要快照的规则不为空，也就是第一个规则存在，那么redis会执行快照

4.执行复制的时候

### 快照的实现原理

1. redis使用fork函数复制一份当前进程的副本(子进程) 
2. 父进程继续接收并处理客户端发来的命令，而子进程开始将内存中的数据写入硬盘中的临时文件 
3. 当子进程写入完所有数据后会用该临时文件替换旧的RDB文件，至此，一次快照操作完成

注意：redis在进行快照的过程中不会修改RDB文件，只有快照结束后才会将旧的文件替换成新的，也就是说任何时候RDB文件都是完整的。 这就使得我们可以通过定时备份RDB文件来实现redis数据库的备份， RDB文件是经过压缩的二进制文件，占用的空间会小于内存中的数据，更加利于传输。

### RDB的优缺点

#### RDB的优点

1. RDB 是一个非常紧凑（compact）的文件，它保存了 Redis 在某个时间点上的数据集。这种文件非常适合用于进行备份：比如说，你可以在最近的 24 小时内，每小时备份一次 RDB 文件，并且在每个月的每一天，也备份一个 RDB 文件。这样的话，即使遇上问题，也可以随时将数据集还原到不同的版本。
2. RDB 非常适用于灾难恢复（disaster recovery）：它只有一个文件，并且内容都非常紧凑，可以（在加密后）将它传送到别的数据中心，或者亚马逊 S3 中。
3. RDB 可以最大化 Redis 的性能：父进程在保存 RDB 文件时唯一要做的就是 fork 出一个子进程，然后这个子进程就会处理接下来的所有保存工作，父进程无须执行任何磁盘 I/O 操作。
4. RDB 在恢复大数据集时的速度比 AOF 的恢复速度要快

#### RDB的缺点

1. 如果你需要尽量避免在服务器故障时丢失数据，那么 RDB 不适合你。虽然 Redis 允许你设置不同的保存点（save point）来控制保存 RDB 文件的频率， 但是， 因为RDB 文件需要保存整个数据集的状态， 所以它并不是一个轻松的操作。因此你可能会至少 5 分钟才保存一次 RDB 文件。在这种情况下， 一旦发生故障停机， 你就可能会丢失好几分钟的数据。
2. 每次保存 RDB 的时候，Redis 都要 fork() 出一个子进程，并由子进程来进行实际的持久化工作。在数据集比较庞大时， fork()可能会非常耗时，造成服务器在某某毫秒内停止处理客户端；如果数据集非常巨大，并且 CPU 时间非常紧张的话，那么这种停止时间甚至可能会长达整整一秒。虽然 AOF 重写也需要进行 fork() ，但无论 AOF 重写的执行间隔有多长，数据的耐久性都不会有任何损失。

## AOF

AOF  记录服务器执行的所有写操作命令 ，可以将Redis执行的每一条写命令追加到硬盘文件中，这一过程显然会降低Redis的性能，但大部分情况下这个影响是能够接受的，另外使用较快的硬盘可以提高AOF的性能。

### 实践

默认情况下Redis没有开启AOF（append only file）方式的持久化，可通过修改redis.conf中的``appendonly yes`` ， 重启后执行对数据的变更命令， 会在bin目录下生成对应的.aof文件， aof文件中会记录所有的操作命令 如下两个参数可以去对aof文件做优化。

 ``auto-aof-rewrite-percentage 100`` 表示当前aof文件大小超过上一次aof文件大小的百分之多少的时候会进行重写。如果之前没有重写过，以启动时aof文件大小为准

``auto-aof-rewrite-min-size 64mb`` 限制允许重写最小aof文件大小，也就是文件大小小于64mb的时候，不需要进行优化

AOF文件的保存位置和RDB文件的位置相同，都是通过dir参数设置的，默认的文件名是apendonly.aof. 可以在redis.conf中的属性 appendfilename appendonlyh.aof修改。

### AOF 重写的原理

Redis 可以在 AOF 文件体积变得过大时，自动地在后台对 AOF 进行重写： 重写后的新 AOF 文件包含了恢复当前数据集所需的最小命令集合。 整个重写操作是绝对安全的，因为 Redis 在创建新 AOF 文件的过程中，会继续将命令追加到现有的 AOF 文件里面，即使重写过程中发生停机，现有的 AOF 文件也不会丢失。 而一旦新 AOF 文件创建完毕，Redis 就会从旧 AOF 文件切换到新 AOF 文件，并开始对新 AOF 文件进行追加操作。

AOF 文件有序地保存了对数据库执行的所有写入操作， 这些写入操作以 Redis 协议的格式保存， 因此 AOF 文件的内容非常容易被人读懂， 对文件进行分析（parse）也很轻松。导出（export） AOF 文件也非常简单：举个例子， 如果你不小心执行了 FLUSHALL 命令， 但只要 AOF 文件未被重写， 那么只要停止服务器， 移除 AOF 文件末尾的 FLUSHALL 命令， 并重启 Redis ， 就可以将数据集恢复到 FLUSHALL 执行之前的状态。

### 同步磁盘数据

redis每次更改数据的时候， aof机制都会将命令记录到aof文件，但是实际上由于操作系统的缓存机制，数据并没有实时写入到硬盘，而是进入硬盘缓存。再通过硬盘缓存机制去刷新到保存到文件：

``appendfsync always`` 每次执行写入都会进行同步 ， 这个是最安全但是是效率比较低的方式

``appendfsync everysec`` 每一秒执行

``appendfsync no`` 不主动进行同步操作，由操作系统去执行，这个是最快但是最不安全的方式

### AOF 文件损坏以后如何修复

服务器可能在程序正在对 AOF 文件进行写入时停机， 如果停机造成了 AOF 文件出错（corrupt）， 那么 Redis 在重启时会拒绝载入这个 AOF 文件， 从而确保数据的一致性不会被破坏。 当发生这种情况时， 可以用以下方法来修复出错的 AOF 文件：

1. 为现有的 AOF 文件创建一个备份。 
2. 使用 Redis 附带的 redis-check-aof 程序，对原来的 AOF 文件进行修复。

``redis-check-aof --fix`` 重启 Redis 服务器，等待服务器载入修复后的 AOF 文件，并进行数据恢复。

### AOF 优缺点

#### AOF 的优点

1. 使用 AOF 持久化会让 Redis 变得非常耐久（much more durable）：你可以设置不同的 fsync 策略，比如无 fsync ，每秒钟一次 fsync ，或者每次执行写入命令时 fsync 。AOF 的默认策略为每秒钟 fsync 一次，在这种配置下，Redis 仍然可以保持良好的性能，并且就算发生故障停机，也最多只会丢失一秒钟的数据（ fsync 会在后台线程执行，所以主线程可以继续努力地处理命令请求）。
2. AOF 文件是一个只进行追加操作的日志文件（append only log）， 因此对 AOF 文件的写入不需要进行 seek ， 即使日志因为某些原因而包含了未写入完整的命令（比如写入时磁盘已满，写入中途停机，等等）， redis-check-aof 工具也可以轻易地修复这种问题。
3. 同  ``AOF 重写的原理``

#### AOF 的缺点

1. 对于相同的数据集来说，AOF 文件的体积通常要大于 RDB 文件的体积。
2. 根据所使用的 fsync 策略，AOF 的速度可能会慢于 RDB 。在一般情况下， 每秒 fsync 的性能依然非常高， 而关闭 fsync 可以让 AOF 的速度和 RDB 一样快， 即使在高负荷之下也是如此。不过在处理巨大的写入载入时，RDB 可以提供更有保证的最大延迟时间（latency）。
3. AOF 在过去曾经发生过这样的 bug ：因为个别命令的原因，导致 AOF 文件在重新载入时，无法将数据集恢复成保存时的原样。（举个例子，阻塞命令 BRPOPLPUSH 就曾经引起过这样的 bug 。） 测试套件里为这种情况添加了测试：它们会自动生成随机的、复杂的数据集， 并通过重新载入这些数据来确保一切正常。虽然这种 bug 在 AOF 文件中并不常见， 但是对比来说， RDB 几乎是不可能出现这种 bug 的

## RDB 和 AOF 如何选择

一般来说,如果对数据的安全性要求非常高的话，应该同时使用两种持久化功能。如果可以承受数分钟以内的数据丢失，那么可以只使用 RDB 持久化。有很多用户都只使用 AOF 持久化， 但并不推荐这种方式： 因为定时生成 RDB 快照（snapshot）非常便于进行数据库备份， 并且 RDB 恢复数据集的速度也要比 AOF 恢复的速度要快 。 两种持久化策略可以同时使用，也可以使用其中一种。如果同时使用的话， 那么Redis重启时，会优先使用AOF文件来还原数据。

# 集群

## 复制（master、slave）

<img src="分布式缓存 Redis.assets/20200612125328405.png" alt="img" style="zoom:50%;" />

### 配置过程

修改192.168.11.140和192.168.11.141的redis.conf文件：slaveof masterip masterport redis5.0只有弃用了 slave 语法，改用replica-announce-ip 和 replica-announce-port

```java
slaveof 192.168.11.138 6379
```

可以通过info replication查看：

```java
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:2
master_replid:e1e1d9ba4ffe4075dbfc5bf92d87a4aa0290b220
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

### 实现原理

slave第一次或者重连到master上以后，会向master发送一个SYNC的命令，master收到SYNC的时候，会做两件事：

1. 执行bgsave（rdb的快照文件） 
2. master 会把新收到的修改命令存入到缓冲区

master通过命令的方式把快照发送给slave，slave收到数据之后，会把自己的数据清空，然后加载这个文件到内存 

![img](分布式缓存 Redis.assets/2020061213453976.png)

当 replica 失去与 master 的连接时，或者当复制仍在进行时，replica 可以以两种不同的方式操作：

1. 如果将replica serve stale data设置为“yes”（默认值），则replica仍将答复客户端请求，可能包含过期数据，或者如果这是第一次同步，则数据集可能只是空的。 
2. 如果replica serve stale data设置为’no’，则replica将对所有类型的命令回复错误“SYNC with master in progress”。

### 缺点

没有办法对master进行动态选举

### 复制的方式

1. 基于rdb文件的复制（第一次连接或者重连的时候） 
2. 无硬盘复制 repl-diskless-sync 
3. 增量复制 PSYNC master run id. offset

## 哨兵机制

sentinel：

1. 监控master和salve是否正常运行 
2. 如果master出现故障，那么会把其中一台salve数据升级为master

redis 目录下有一个sentinel.conf文件： sentinel monitor <master-name> <ip> <redis-port> <quorum> quorum 表示最小投票数

```java
sentinel monitor mymaster 192.168.11.11.138 6379 2
```

通过 ./redis-sentinel ../sentinel.conf 启动哨兵。

master 在挂掉动态恢复之后，会以slave的身份加入到集群（哨兵会重写redis.conf配置文件），但它也有一个问题，就是不能动态扩充。

## 集群（redis3.0以后的功能）

根据 key 的hash值取模服务器的数量 。

### 集群的原理

Redis Cluster中，Sharding采用slot(槽)的概念，一共分成16384个槽，这有点儿类似pre sharding思路。对于每个进入Redis的键值对，根据key进行散列，分配到这16384个slot中的某一个中。使用的hash算法也比较简单，就是CRC16后16384取模。Redis集群中的每个node(节点)负责分摊这16384个slot中的一部分，也就是说，每个slot都对应一个node负责处理。当动态添加或减少node节点时，需要将16384个槽做个再分配，槽中的键值也要迁移。当然，这一过程，在目前实现中，还处于半自动状态，需要人工介入。Redis集群，要保证16384个槽对应的node都正常工作，如果某个node发生故障，那它负责的slots也就失效，整个集群将不能工作。为了增加集群的可访问性，官方推荐的方案是将node配置成主从结构，即一个master主节点，挂n个slave从节点。这时，如果主节点失效，Redis Cluster会根据选举算法从slave节点中选择一个上升为主节点，整个集群继续对外提供服务。这非常类似服务器节点通过Sentinel监控架构成主从结构，只是Redis Cluster本身提供了故障转移容错的能力。

slot（槽）的概念，在redis集群中一共会有16384个槽，根据key 的CRC16算法，得到的结果再对16384进行取模。 假如有3个节点： 

node1 0 5460 

node2 5461 10922 

node3 10923 16383

**节点新增** 

node4 0-1364,5461-6826,10923-12287 

**删除节点** 

先将节点的数据移动到其他节点上，然后才能执行删除

## 市面上提供了集群方案

1.redis shardding 而且jedis客户端就支持shardding操作 SharddingJedis：增加和减少节点的问题； pre shardding，3台虚拟机 redis 。但是我部署了9个节点 。每一台部署3个redis增加cpu的利用率。9台虚拟机单独拆分到9台服务器。 

2.codis基于redis2.8.13分支开发了一个codis-server 

3.twemproxy twitter提供的开源解决方案

## redis 缓存的更新


1. 先删除缓存， 再更新数据库 
2. 先更新数据库，更新成功后，让缓存失效 
3. 更新数据的时候， 只更新缓存，不更新数据库，然后同步异步调度去批量更新数据库

# 缓存击穿

## 缓存穿透

缓存穿透是指用户查询数据，在数据库没有，自然在缓存中也不会有。这样就导致用户查询的时候，在缓存中找不到，每次都要去数据库中查询。 

解决思路：

1，如果查询数据库也为空，直接设置一个默认值存放到缓存，这样第二次到缓冲中获取就有值了，而不会继续访问数据库，这种办法最简单粗暴。

2，根据缓存数据Key的规则。例如我们公司是做机顶盒的，缓存数据以Mac为Key，Mac是有规则，如果不符合规则就过滤掉，这样可以过滤一部分查询。在做缓存规划的时候，Key有一定规则的话，可以采取这种办法。这种办法只能缓解一部分的压力，过滤和系统无关的查询，但是无法根治。

3，采用布隆过滤器，将所有可能存在的数据哈希到一个足够大的BitSet中，不存在的数据将会被拦截掉，从而避免了对底层存储系统的查询压力。

大并发的缓存穿透会导致缓存雪崩。

## 缓存击穿

缓存击穿是指缓存中没有但数据库中有的数据（一般是缓存时间到期），这时由于并发用户特别多，同时读缓存没读到数据，又同时去数据库去取数据，引起数据库压力瞬间增大，造成过大压力。

解决思路：

1，设置热点数据永远不过期

2，加互斥锁

## 缓存雪崩

缓存雪崩可能是因为数据未加载到缓存中，或缓存宕机，或者缓存同一时间大面积的失效，从而导致所有请求都去查数据库，导致数据库CPU和内存负载过高，甚至宕机。

解决思路：

1，采用加锁计数，或者使用合理的队列数量来避免缓存失效时对数据库造成太大的压力。这种办法虽然能缓解数据库的压力，但是同时又降低了系统的吞吐量。

2，分析用户行为，尽量让失效时间点均匀分布。避免缓存雪崩的出现。

3，如果是因为某台缓存服务器宕机，可以考虑做主备，比如：redis主备，但是双缓存涉及到更新事务的问题，update可能读到脏数据，需要好好解决。

## 缓存失效

如果缓存集中在一段时间内失效，DB的压力凸显。这个没有完美解决办法，但可以分析用户行为，尽量让失效时间点均匀分布。 

# Redis 与 MongDB

![image-20201011113841532](分布式缓存 Redis.assets/image-20201011113841532.png)