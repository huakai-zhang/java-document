# 1 数据库事务

## 1.1 事务（Transation）及其ACID属性

《参考 Spring 事务原理详解》

> 在 InnoDB 里面是通过 undo log 来实现`原子性`，它记录了数据修改之前的值(逻辑日志)，一旦发生异常就可以用 undo log 来实现回滚操作。
>
> `持久性`是通过 redo log 和 double write 双写缓冲来实现的，我们操作数据的时候，会先写到内存的 buffer pool 里面，同时记录 redo log，如果在刷盘之前出现异常，在重启后就可以读取 redo log 的内容，写入到磁盘，保证数据的持久性。 当然，恢复成功的前提是数据页本身没有被破坏，是完整的，这个通过双写缓冲 （double write）保证。
>
> InnoDB 引擎通过`锁机制、MVCC` 等手段来保证事务的`隔离性` 。
>
> 原子性，隔离性，持久性，最后都是为了实现一致性。

## 1.2 数据库什么时候会出现事务

执行一条更新语句，实际上，它自动开启了一个事务，并且提交了，所以最终写入了磁盘。这个是开启事务的第一种方式，自动开启和自动提交。

```mysql
# InnoDB 里面有一个 autocommit 的参数（分成两个级别， session 级别和 global级别）
# 它的默认值是 ON,是否自动提交
show variables like 'autocommit';
```

手动开启事务也有几种方式，一种是用 `begin`，一种是用 `start transaction`。 

结束事务也有两种方式，第一种就是提交一个事务 `commit`，还有一种就是 `rollback`，回滚的时候，事务也会结束。还有一种情况，客户端的连接断开的时候，事务也会结束。

当我们结束一个事务的时候，事务持有的锁就会被释放，无论是提交还是回滚。

## 1.3 并发事务处理带来的问题

``更新丢失(Lost Update)`` 当两个或多个事务选择同一行，然后基于最初选定的值更新该行时，由于每个事务都不知道其他事务的存在，就会发生丢失更新问题，最后的更新覆盖了由其他事务所做的更新。

``脏读(Dirty Reads)`` 一个事务对数据进行了增删改，但未提交，这条记录就处于不一致状态，另一个事务可以读取到未提交的数据。如果第一个事务这时候回滚，那么第二个事务读到了脏数据，不符合一致性要求。

事务A读取到了事务B已修改但尚未提交的数据。 

``不可重复读(Non-Reoeatable Reads)`` 一个事务中发生了两次读操作，第一次读操作和第二次读操作之间，另一个事务对数据进行了修改，这时候两次读取的数据是不一致的。 

事务A读取到了事务B已经提交的修改数据，不符合隔离性。

``幻读(Phantom Reads)`` 一个事物按相同的查询条件重新读取检索过的数据，却发现其他事务插入了满足其查询条件的新数据。

事务A读取到了事务B已提交的新增数据，不符合隔离性。

幻读和不可重复读有点类似，不可重复读是事务B修改(或删除)了数据，幻读是事务B新增了数据。

## 1.4 数据库事务隔离级别

| 隔离级别                  | 隔离级别的值                     | 导致的问题                                                   |
| ------------------------- | -------------------------------- | ------------------------------------------------------------ |
| 未提交读 Read-Uncommitted | 0 只能保证不读取物理上损坏的数据 | 允许读取还未提交的改变了的数据，导致脏读，幻，不可重复读     |
| 已提交读 Read-Committed   | 1 语句级                         | 允许在并发事务已经提交后读取，避免脏读，允许不可重复读和幻读（默认） |
| 可重复读Repeatable-Read   | 2 事务级                         | 对相同字段的多次读取是一致的，除非数据被事务本身改变，避免脏读，不可重复读，允许幻读（有增删改操作，不允许读取） |
| 可串行化 Serializable     | 3 最高级别，事务级               | 串行化读，事务只能一个一个执行，避免了脏读，不可重复读，幻读。执行效率慢，使用时谨慎 |

InnoDB 支持的四个隔离级别和 [SQL92](http://www.contrib.andrew.cmu.edu/~shadow/sql/sql1992.txt) 定义的基本一致，隔离级别越高，越能保证数据的完整性和一致性，但是对并发性能的影响也越大。 

大多数的数据库默认隔离级别为Read Commited，比如SqlServer、 Oracle 

少数数据库默认隔离级别为: Repeatable Read，比如: MySQL InnoDB

> InnoDB 在 RR 的级别就解决了幻读的问题，这个也是 InnoDB 默认使用 RR 作为事务隔离级别的原因，既保证了数据的一致性，又支持较高的并发度。

```mysql
# 查看当前数据库的事务隔离级别
show variables like 'tx_isolation';
```

## 1.5 MVCC 实现

> `快照读` 单纯的 select 操作
>
> 快照读的实现方式：undolog 和多版本并发控制 MVCC

如果要让一个事务前后两次读取的数据保持一致， 那么我们可以`在修改数据的时候`给它建立一个`备份或者叫快照`，后面再来读取这个快照就行了。这种方案我们叫做多版本的并发控制 `Multi Version Concurrency Control (MVCC)`。

> 既然要保证前后两次读取数据一致，那么读取数据的时候锁定要操作的数据，不允许其他的事务修改就行了。这种方案我们叫做基于锁的并发控制 `Lock Based Concurrency Control（LBCC）`。

问题：这个快照什么时候创建？读取数据的时候，怎么保证能读取到这个快照而不是最新的数据？

在 InnoDB 中，MVCC 是`通过 Undo log 实现`的。

InnoDB 为每行记录都实现了两个隐藏字段： 

* `DB_TRX_ID` 6 字节：插入或更新行的最后一个事务的事务 ID，事务编号是自动递增的(我们把它理解为创建版本号，在数据新增或者修改为新数据的时候，记录当前事务 ID)

* `DB_ROLL_PTR` 7 字节：回滚指针(我们把它理解为删除版本号，数据被删除或记录为旧数据的时候，记录当前事务 ID)

我们把这两个事务 ID 理解为版本号。

![img](MySQL 高级.assets/WechatIMG226.png)

# 2 MySQL 锁机制

## 2.1 概述

锁是计算机协调多个进程或线程并发访问某一资源的机制。

在数据库中，除传统的计算资源(如CPU、RAM、I/O等)的争用以外，数据也是一种供许多用户共享的资源，如何保证数据并发访问的一致性、有效性是所有数据库必须解决的一个问题，锁冲突也是影响数据库并发访问性能的一个重要因素。从这个角度来说，锁对数据库而言显得尤其重要，也更加复杂。

**锁的分类**

从锁的类型（或者叫基本的锁的模式）分：

`读锁(共享锁，包括意向锁)` 针对同一份数据，多个读操作可以同时进行而不互相影响。

```mysql
begin;
select …… lock in share mode;
rollback / commit;
```

`写锁(排它锁，包括意向锁)` 当前写操作没有完成前，它会阻断其他写锁和读锁。

```mysql
# delete / insert / update 默认自动加上写锁
# 手动
select …… for update;
rollback / commit;
```

从对数据操作的颗粒度分：`行锁`、`表锁`

```mysql
# 表级锁争用状态变量
show status like 'table%'
# 行级锁争用状态变量
show status like 'innodb_row_lock%'
```

> 锁定粒度，表锁大于行锁
>
> 加锁效率，表锁大于行锁，表锁只需要 直接锁住这张表就行了，而行锁还需要在表里面去检索这一行数据
>
> 冲突概率，表锁大于行锁，锁表其他任何一个事务都不能操作这张表
>
> 并发性能，表锁的冲突概率更大，所以并发性能更低

## 2.2 意向锁

`意向锁`是由数据库自己维护的，当我们给一行数据加上共享锁之前，数据库会自动在这张表上面加一个意向共享锁。当我们给一行数据加上排他锁之前，数据库会自动在这张表上面加一个意向排他锁。

反过来说：如果一张表上面至少有一个意向共享锁，说明有其他的事务给其中的某些数据行加上了共享锁。如果一张表上面至少有一个意向排他锁，说明有其他的事务给其中的某些数据行加上了排他锁。

**那么这两个表级别的锁存在的意义是什么呢？**

* 我们有了表级别的锁，在 InnoDB 里面就可以支持更多粒度的锁
* 如果说没有意向锁的话，当给一张表加上表锁的时候，必须先要去判断有没其他的事务锁定了其中了某些行(需要进行全表扫描)，引入了意向锁后，只要判断这张表上面有没有意向锁，如果有就直接返回失败，如果没有就可以加锁成功。所以 InnoDB 里面的表锁，我们可以把它理解成一个标志

## 2.3 表锁（偏读）

偏向 MyISAM 存储引擎，开销小，获取释放锁快，避免死锁，锁定粒度大，发生锁冲突的概率最高，并发最低。

```mysql
CREATE TABLE `mylock` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(20) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;

insert into mylock(name) values('a');
insert into mylock(name) values('b');
insert into mylock(name) values('c');
insert into mylock(name) values('d');
insert into mylock(name) values('e');

# 手动增加表锁
# 为mylock表添加读锁，为book添加写锁
lock table mylock read, book write;

# 查看表上加过的锁
show open tables;

# 释放表锁
unlock tables;
```

![image-20200927214048725](MySQL 高级.assets/image-20200927214048725.png)

### 2.3.1 加读锁

为mylock表加read锁（读阻塞写例子）

| session_1                                                    | session_2                                                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 获得表mylock的READ锁定<br>lock table mylock read;            | 连接终端                                                     |
| 当前session可以查询该表记录<br>select * from mylock;         | 其他session也可以查询该表记录<br>select * from mylock;       |
| 当前session不能查询其他没有锁定的表<br>select * from book;<br/>ERROR 1100 (HY000): Table 'book' was not locked with LOCK TABLES | 其他session可以查询或更新未锁定的表<br>select * from book;<br>update book set card=20 where bookid = 25; |
| 当前session中插入或更新锁定的表都会提示错误<br>update mylock set name='a2' where id = 1;<br/>ERROR 1099 (HY000): Table 'mylock' was locked with a READ lock and can't be updated | 其他session插入或更新锁定表会一直等待获得锁<br>update mylock set name='a2' where id = 1; |
| 释放锁<br>unlock tables;                                     | session2 获得锁，插入操作完成<br>Query OK, 0 rows affected (40.17 sec) |

### 2.3.2 加写锁

为mylock表加write锁（MyISAM存储引擎的写阻塞读例子）

| session_1                                                    | session_2                                                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 获得锁mylock的write锁定                                      | session2再连接终端                                           |
| 当前session对锁定表的查询更新插入操作都可以执行<br>select * from mylock;<br>update mylock set name='a2' where id = 1; | 其他session对锁定表的查询被阻塞，需要等待锁被释放<br>select * from mylock where id=1; |
| 释放锁<br/>unlock tables;                                    | session2获得锁，查询返回<br>5 rows in set (29.73 sec)        |

MySQL的表级锁有两种模式：

表共享读锁（Table Read Lock）

表独占写锁（Table Write Lock）

1. 对MyISAM表的读操作（加读锁），不会阻塞其他进程对同一表的读请求，但会阻塞对同一表的写请求。只有当前读锁释放后，才会执行其他进程的写操作。
2. 对MyISAM表的写操作（加写锁），会阻塞其他线程对同一表的读和写操作，只有当写锁释放后，才会执行其他进程的读写操作。

简而言之，就是读锁会阻塞写，但不会阻塞读。而写锁则会把读和写都阻塞。

### 2.3.3 分析表锁定

```mysql
show status like 'table%';
```

![image-20200927222724030](MySQL 高级.assets/image-20200927222724030.png)

可以通过检查table_locks_waited和table_locks_immediate状态变量来分析系统上的表锁定：

``table_locks_immediate`` 产生表级锁的次数，表示可以立即获取锁的查询次数，每立即获取锁值加1；

``table_locks_waited`` 出现表级锁争用而产生等待的次数（不能立即获取锁的次数，每等待一次锁值加1），此值高则说明存在着较严重的表级锁争用情况；

此外，MyISAM的读写锁调度是写优先，这也是MyISAM不适合做写为主表的引擎。因为写锁后，其他线程不能做任何操作，大量的更新会是查询很难得到锁，从而造成阻塞。

## 2.4 行锁（偏写）

偏向InnoDB存储引擎，开销大，获取释放锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。

InnoDB与MyISAM的最大不同有两点：一是支持事务（TRANSACTION）;二是采用了行级锁

### 2.4.1 行锁演示

```mysql
create table innodb_lock(a int(11), b varchar(16)) engine=innodb;

insert into innodb_lock values(1, 'b2');
insert into innodb_lock values(3, '3');
insert into innodb_lock values(4, '4000');
insert into innodb_lock values(5, '5000');
insert into innodb_lock values(6, '6000');
insert into innodb_lock values(7, '7000');

create index innodb_a_ind on innodb_lock(a);
create index innodb_b_ind on innodb_lock(b);
```

| session1                                                     | session2                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| set autocommit=0;                                            | set autocommit=0;                                            |
| 更新但不提交，没有手动commit<br>update innodb_lock set b='4001' where a=4; | session更新同一行数据，只能阻塞等待<br>update innodb_lock set b='4002' where a=4; |
| 提交更新<br>commit;                                          | 解除阻塞，更新正常进行<br>Query OK, 1 row affected (15.42 sec) |
|                                                              | commit执行提交                                               |
| 更新但不提交，没有手动commit<br/>update innodb_lock set b='4002' where a=4;<br>session1自己可以查询到更新 | session2查询不到session1的更改<br>select  * from innodb_lock; |
| 提交更新<br/>commit;                                         | 依然查询不到，需要session2也执行commit;<br>才能查询到session1的更新，保证可重复读 |
| session1更新a=4;                                             | session更新a=5;<br>不会阻塞                                  |

#### 不使用索引行锁升级为表锁

一个没有定义索引的表(主键索引也没有)，查询时会导致锁表，原因在于 InnoDB 会选择内置 6 字节长的 ROWID 作为隐藏的聚集索引，它会随着行记录的写入而主键递增。查询没有使用索引，会进行全表扫描，然后把每一个隐藏的聚集索引都锁住了。

`varchar`  不用 ' '  导致系统自动转换类型，行锁变表锁(`转换导致没有使用索引`)

```mysql
# session1做如下更新（b使用int类型变表锁），session2会阻塞(有索引未加‘’)
update innodb_lock set a=8 where b=4001;

# 删除 b 字段的索引后，session1使用 b 字段做更新（无索引变表锁），session2会阻塞
update innodb_lock set a=8 where b='4001';
```

> 通过唯一索引给数据行加锁，主键索引也会被锁住：
>
> 在辅助索引里面，索引存储的是二级索引和主键的值。而主键索引里面除了索引之外，还存储了完整的数据。所以我们通过辅助索引锁定一行数据的时候，它跟我们检索数据的步骤是一样的，会通过主键值找到主键索引，然后也锁定。 

#### 如何锁定一行

```mysql
# select ... for update;锁定某一行后，其他操作会被阻塞，直到锁定行的会话commit
begin;
select * from innodb_lock where a=8 for update;
...
commit;
```

#### 行锁总结

Innodb存储引擎由于实现了行级锁定，虽然在锁定机制的实现方面所带来的性能损耗可能比表级锁定会更高一些，但是在整体并发处理能力方面要远远优于MyISAM的表级锁定的。当系统并发量较高的时候，Innodb的整体性能和MyISAM相比就会有比较明显的优势了。

但是Innodb的行级锁定同样也有脆弱的一面，当我们使用不当的时候，可能会让InnoDB的整体性能表现不仅不能比MyISAM高甚至会更差。

#### 行锁分析

```mysql
show status like 'innodb_row_lock%';
```

![1601261501682](MySQL 高级.assets/1601261501682.png)

各个状态量的说明如下：

``Innodb_row_lock_current_waits`` 当前正在等待锁定的数量

``Innodb_row_lock_time`` 从系统启动到现在锁定总时间长度

``Innodb_row_lock_time_avg `` 每次等待所花平均时间

``Innodb_row_lock_time_max`` 从系统启动到现在等待最长的一次所花的时间

``Innodb_row_lock_waits`` 系统启动后到现在总共能带的次数

比较重要的主要是：

``Innodb_row_lock_time_avg `` 等待平均时长

``Innodb_row_lock_waits`` 等待总次数

``Innodb_row_lock_time`` 等待总时长

尤其等等待次数很高，而且每次等待时长也不小的时候，我们就需要分析系统中为什么如此多的等待，然后根据分析结果着手指定优化计划（show profile）。

### 2.4.2 优化建议

尽可能让所有数据检索都通过索引来完成，避免无索引行锁升级为表锁

合理设计索引，尽量缩小锁的范围

尽可能较少检索条件，避免间隙锁

尽量控制事务大小，减少锁定资源量和时间长度

尽可能低级别事务隔离

## 2.5 锁的算法

<img src="MySQL 高级.assets/image-20210216124920346.png" alt="image-20210216124920346" style="zoom:40%;" />

数据库里面存在的主键值，我们把它叫做 `Record 记录`，那么这里我们就有 4 个 Record。 

根据主键，这些存在的 Record 隔开的数据不存在的区间，我们把它叫做 `Gap 间隙`，它是一个左开右开的区间。 

间隙（Gap）连同它左边的记录（Record），我们把它叫做`Next-key 临键` 的区间，它是一个左开右闭的区间。

### 记录锁

当我们对于唯一性的索引（包括唯一索引和主键索引）使用等值查询，精准匹配到一条记录的时候，这个时候使用的就是`记录锁`。

> 当查询的索引含有唯一属性时，InnoDB 会对 Next-Key Lock 进行优化，将其降级为 Record Lock，即仅锁住索引本身，而不是范围。

使用不同的 key 去加锁，不会冲突，它只锁住这个 record。 

### 间隙锁

当我们查询的记录不存在，没有命中任何一个 record，无论是用等值查询还是范围查询(间隙+记录)的时候，它使用的都是`间隙锁`。

间隙锁主要是阻塞插入 insert，相同的间隙锁之间不冲突。

```mysql
CREATE TABLE `t1` (
  `id` int(11) NOT NULL,
  `name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

INSERT INTO `t1` VALUES (1, '1');
INSERT INTO `t1` VALUES (4, '4');
INSERT INTO `t1` VALUES (7, '7');
INSERT INTO `t1` VALUES (10, '10');
```

| session1                                             | session2                                                     |
| ---------------------------------------------------- | ------------------------------------------------------------ |
| select * from t1 where id = 5 for update;            |                                                              |
| (session2 insert 6 之后session1 insert 5 会出现死锁) | 未产生阻塞<br/>select * from t1 where id = 6 for update;<br>select * from t1 where id > 4 and id < 7 for update;<br>阻塞产生，暂时不能插入<br/>insert into t1 values(6, '2021'); |
| commit;                                              | 阻塞解除，完成插入<br>Query OK, 1 row affected (4.47 sec)    |
| update t1 set name='0928' where id > 100;            |                                                              |
|                                                      | 未产生阻塞<br>update t1 set name='0928' where id > 30;<br>阻塞产生<br>insert into t1 values(20, '2000'); |

Gap Lock 只在 RR 中存在。如果要关闭间隙锁，可以通过以下两种方式：

* 将事务隔离级别设置成 RC
* 将 `innodb_locks_unsafe_for_binlog` 设置为 ON(已弃用)

这种情况下除了外键约束和唯一性检查会加间隙锁，其他情况都不会用间隙锁。

### 临键锁

> `当前读` select...lock in share mode (共享读锁)、select...for update、update、delete、insert
>
> 当前读的实现方式：next-key 锁(行记录锁 + Gap 间隙锁)

当我们使用了范围查询时使用的就是`临键锁`，它是 MySQL 里面默认的行锁算法，结合了间隙锁和记录锁的一种锁定算法。临键锁，锁住最后一个 key 的`下一个左开右闭的区间`。 

> 如果范围查询未命中 Record 记录，那么下一个左开右闭区间就是当前区间(间隙锁) + 下一个记录(记录锁)。

```mysql
select * from t1 where id >4 and id < 7 for update;  -- 锁住 (4,7]
select * from t1 where id >5 and id <=7 for update;  -- 锁住 (4,7] 和(7,10] 
select * from t1 where id >8 and id <=10 for update; -- 锁住 (7,10]和(10,+∞)
```

| session1                                            | session2                                                     |
| --------------------------------------------------- | ------------------------------------------------------------ |
| select * from t1 where id >5 and id < 9 for update; |                                                              |
|                                                     | 阻塞产生，暂时不能插入<br/>INSERT INTO t1 (id,name) VALUES (6, '6');<br>INSERT INTO t1 (id,name) VALUES (8, '8');<br>阻塞，无法查询区间存在的值，不存在的可以查询例如 9 <br>select * from t1 where id = 10 for update; |
| commit;                                             | 阻塞解除，完成插入<br>Query OK, 1 row affected (4.47 sec)    |

**id >5 and id < 9 锁住 (4,7] 和 (7,10] 为什么 9 可以查询而 10 不能？**

实际上临键锁是间隙锁和记录锁的组合，即是 (4,7)、7、(7,10)、10

> Gap locks in InnoDB are “purely inhibitive”, which means that their only purpose is to prevent other transactions from inserting to the gap. Gap locks can co-exist. A gap lock taken by one transaction does not prevent another transaction from taking a gap lock on the same gap.
>
> InnoDB 中的 Gap 锁是“纯抑制性的”，这意味着它的唯一目的是防止其他事务插入到 Gap 中。间隙锁可以共存。一个事务获取的间隙锁并不阻止另一个事务在同一间隙上获取间隙锁。

查询 9 是获得间隙锁(可以共存)，而查询 10 是获得记录锁。

**为什么要锁住下一个左开右闭的区间？**

就是为了解决幻读的问题。

**危害**

因为 Query 执行过程中通过范围查找的话，它会锁定整个范围内所有的索引键值，即使这个键值并不存在。

间隙锁有一个比较致命的弱点，就是当锁定一个范围键值之后，即使某些不存在的键值也会被无辜的锁定，而造成在锁定的时候无法插入锁定范围的任何数据。在某些场景下这可能会对性能造成很大的危害。

| session1                                            | session2                                                    |
| --------------------------------------------------- | ----------------------------------------------------------- |
| select * from t1 where id >5 and id < 9 for update; | 阻塞产生，暂时不能插入<br>insert into t1 values(6, '2000'); |
| commit;                                             | 阻塞解除，完成插入<br>Query OK, 1 row affected (4.47 sec)   |

## 2.6 死锁

### 2.6.1 锁的释放与阻塞

事务结束(commit，rollback)或客户端连接断开时，锁会释放。

如果一个事务一直未释放锁，MySQL 有一个参数来控制获取锁的等待时间，默认是 50 秒。

```mysql
#[Err] 1205 - Lock wait timeout exceeded; try restarting transaction
show VARIABLES like 'innodb_lock_wait_timeout';
```

### 2.6.2 死锁的发生和检测

死锁演示：

| session1                                            | session2                                |
| --------------------------------------------------- | --------------------------------------- |
| begin; <br>select * from t2 where id =1 for update; |                                         |
|                                                     | begin;<br/>delete from t2 where id =4 ; |
| update t2 set name= '4d' where id =4 ;              |                                         |
|                                                     | delete from t2 where id =1 ;            |

在第一个事务中，检测到了死锁马上退出了，第二个事务获得了锁，不需要等待 50 秒：

```
ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction
```

**为什么可以直接检测到呢？**

是因为死锁的发生需要满足一定的条件，所以在发生死锁时，InnoDB 一般都能通过`算法 wait-for graph` 自动检测到。 

死锁的产生条件： 

* 同一时刻只能有一个事务持有这把锁

* 其他的事 务需要在这个事务释放锁之后才能获取锁，而不可以强行剥夺

* 当多个事务形成等 待环路的时候，即发生死锁

### 2.6.3 查看锁信息（日志） 

SHOW STATUS 命令中，包括了一些行锁的信息：

```mysql
show status like 'innodb_row_lock_%';
# Innodb_row_lock_current_waits：当前正在等待锁定的数量
# Innodb_row_lock_time ：从系统启动到现在锁定的总时间长度，单位 ms
# Innodb_row_lock_time_avg ：每次等待所花平均时间
# Innodb_row_lock_time_max：从系统启动到现在等待最长的一次所花的时间
# Innodb_row_lock_waits ：从系统启动到现在总共等待的次数

# SHOW 命令是一个概要信息，InnoDB 还提供了三张表来分析事务与锁的情况
# 当前运行的所有事务
select * from information_schema.INNODB_TRX;
# 当前出现的锁
select * from information_schema.INNODB_LOCKS;
# 锁等待的对应关系
select * from information_schema.INNODB_LOCK_WAITS;
```

如果一个事务长时间持有锁不释放，可以 kill 事务对应的线程 ID，也就是 INNODB_TRX 表中的 trx_mysql_thread_id。 

死锁的问题不能每次都靠 kill 线程来解决，应尽量在编码的过程中避免。

### 2.6.4 死锁的避免

1. 在程序中，操作多张表时，尽量以相同的顺序来访问（避免形成等待环路）

2. 批量操作单张表数据的时候，先对数据进行排序（避免形成等待环路）

3. 申请足够级别的锁，如果要操作数据，就申请排它锁

4. 尽量使用索引访问数据，避免没有 where 条件的操作，避免锁表

5. 如果可以，大事务化成小事务

6. 使用等值查询而不是范围查询查询数据，命中记录，避免间隙锁对并发的影响

## 2.7 隔离级别的实现

### Read Uncommited

RU 隔离级别：不加锁

### Read Commited

RC 隔离级别下，普通的 select 都是快照读，使用 MVCC 实现。 

加锁的 select 都使用记录锁，因为没有 Gap Lock，除了`外键约束检查(foreign-key constraint checking)`以及`重复键检查(duplicate-key checking)`时会使用间隙锁封锁区间，所以 RC 会出现幻读的问题。

### Repeatable Read

RR 隔离级别下，普通的 select 使用`快照读(snapshot read)`，底层使用 MVCC 来实现。

加锁的 select(select ... in share mode / select ... for update)以及更新操作 update, delete 等语句使用`当前读(current read)`，底层使用记录锁、间隙锁、临键锁。 

### Serializable

Serializable 所有的 select 语句都会`被隐式的转化为 select ... in share mode`，会和 update、delete 互斥。 

**RU 和 Serializable 肯定不能用。为什么有些公司要用 RC，或者说网上有些文章推荐有 RC？ **

RC 和 RR 主要有几个区别： 

1. RR 的间隙锁会导致锁定范围的扩大

2. 条件列未使用到索引，RR 锁表，RC 锁行

3. RC 的“半一致性”（semi-consistent）读可以增加 update 操作的并发性

在 RC 中，一个 update 语句，如果读到一行已经加锁的记录，此时 InnoDB 返回记录最近提交的版本，由 MySQL 上层判断此版本是否满足 update 的 where 条件。若满足(需要更新)，则 MySQL 会重新发起一次读操作，此时会读取行的最新版本(并加锁)。 

实际上，如果能够正确地使用锁（避免不使用索引去加锁），只锁定需要的数据，用默认的 RR 级别就可以了。 

# 3 主从复制

## 3.1 复制的基本原理

复制的节点 slave 会从被复制的节点 master 读取 binlog 来进行数据同步。slave 本身也可以作为其他节点的数据来源，这个叫做`级联复制`。

<img src="MySQL 高级.assets/image-20210217111452786.png" alt="image-20210217111452786" style="zoom:50%;" />

MySQL 复制过程分为三步：

1. master 将更新语句记录到`二进制日志(binary log)`，它是一种逻辑日志，这些记录过程叫做`二进制日志事件(binary log events)`，master 节点上有一个 `log dump 线程`，是用来发送 binlog 给 slave 的
2. slave 的 `I/O 线程` 连接到 master 获取 binlog，并且解析 binlog 写入`中继日志(relay log)`
3. 从库的 `SQL 线程`，是用来读取 relay log，把数据写入到数据库的，MySQL复制是异步的且串行化的

## 3.2 复制的基本原则

每个 slave 只有一个 master

每个 slave 只能有一个唯一的服务器ID

每个 master 可以有多个 salve

## 3.3 一主一从常见配置

MySQL版本一致且后台以服务运行，主从都配置在【mysqld】结点下，都是小写。

主机从机都关闭防火墙。

### 3.3.1 主机修改my.cnf配置文件

```markdown
[mysqld]
# 1.【必须】主服务器唯一ID
server-id =1
# 2.【必须】启用二进制日志
log-bin=自己本地的路径/mysqlbin
log-bin=D:/Mysql5.5/data/mysqlbin
# 3.【可选】启动错误日志
log-err=自己本地的路径/mysqlerr
# 4.【可选】根目录
basedir="自己本地路径"
# 5.【可选】临时目录
tmpdir="自己的本地路劲"
# 6.【可选】数据目录
datadir="自己本地路径/Data/"
# 7.主机，读写都可以
read-only=0
# 8.【可选】设置不要复制的数据库
binlog-ignore-db=mysql
# 9.【可选】设置需要复制的数据
binlog-do-db=需要复制的主数据库名字
```

### 3.3.2 从机修改my.cnf配置文件

```markdown
# 【必须】从服务器唯一ID
server-id       = 2
# 【可选】启用二进制文件
log-bin=mysql-bin
```

### 3.3.3 在主机上建立账户并授权slave

```mysql
# 给ip为192.168.174.128的从机授权帐号为zhangsan密码为123456的帐号访问
GRANT REPLICATION SLAVE  ON*.* TO '帐号'@'从机器数据库IP' IDENTIFIED BY '密码';
GRANT REPLICATION SLAVE  ON*.* TO 'zhangsan'@'192.168.174.128' IDENTIFIED BY '123456';
# 刷新MySQL的系统权限相关表
flush privileges;
# 查询主机状态，记录下File和Position的值（从File文件的Position行开始复制）
show master status;
```

![1601276350119](MySQL 高级.assets/1601276350119.png)

### 3.3.4 在从机上配置需要复制的主机

```mysql
# 如果之前做过主从复制，要先停止
stop slave;

CHANGE MASTER TO 
MASTER_HOST='主机IP',
MASTER_USER='帐号',
MASTER_PASSWORD='密码',
MASTER_LOG_FILE='File名字',MASTER_LOG_POS=Position数字;
CHANGE MASTER TO 
MASTER_HOST='192.168.174.1',
MASTER_USER='zhangsan',
MASTER_PASSWORD='123456',
MASTER_LOG_FILE='mysqlbin.000001',MASTER_LOG_POS=107;

# 启动从服务器复制功能
start slave;

# 查看slave状态
show slave status;
```

下面两个参数都是YES，则说明主从配置成功！

Slave_IO_Running:Yes

Slave_SQL_Running:Yes

![1601276806420](MySQL 高级.assets/1601276806420.png)

------


