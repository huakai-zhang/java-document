## 1 认识 MySQL

1985年，瑞典的几位志同道合小伙子(以 David Axmark 为首)成立了一 家公司，这就是 MySQL AB 的前身。这个公司最初并不是为了开发数据库产品，而是在实现他们想法的过程中，需要一个数据库。他们希望能够使用开源的产品。但在当时并没-一个合适的选择，没办法，那就自己开发吧。

## 2 MySQL 下载安装

下载地址：https://dev.mysql.com/ »  [MySQL Downloads](https://dev.mysql.com/downloads/) » [MySQL Community Server](https://dev.mysql.com/downloads/mysql/)

### 2.1 MySQL 安装完成相关命令

```markdown
# 查看Mysql安装时创建的mysql用户和mysql组
	cat /etc/passwd | grep mysql
		_mysql:*:74:74:MySQL Server:/var/empty:/usr/bin/false
	cat /etc/group | grep mysql
		_mysql:*:74:

# 查看 MySQL 版本
	mysqladmin --version
		mysqladmin  Ver 8.42 Distrib 5.7.22, for macos10.13 on x86_64

# MySQL 服务启停
	service mysql start
	service mysql stop

# 连接数据库
	mysql -u root -p

# Mysql的安装位置
	ps -ef | grep mysql
		74   141     1   0  7:00下午 ??         0:01.75 /usr/local/mysql/bin/mysqld --user=_mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --plugin-dir=/usr/local/mysql/lib/plugin --log-error=/usr/local/mysql/data/mysqld.local.err --pid-file=/usr/local/mysql/data/mysqld.local.pid --keyring-file-data=/usr/local/mysql/keyring/keyring --early-plugin-load=keyring_file=keyring_file.so
```

### 2.2 修改字符集和数据存储路径

```mysql
# 查看当前字符集
show variables like '%char%'
```

![image-20200921212803689](mysql基础.assets/image-20200921212803689.png)

数据库和服务端的字符集默认都是latin1，中文会乱码。

```markdown
# Linux 自定义配置文件/etc/my.cnf
[client]
default-character-set = utf8

[mysqld]
character_set_server=utf8
character_set_client=utf8
collation-server=utf8_general_ci
# (注意linux下mysql安装完默认：表名区分大小写，列名不区分大小写；0:区分大小写，1:不区分大小写)
lower_case_table_names=1
# (设置最大连接数，默认为151，mysql服务器允许最大连接数16384)
max_connections=1000
# 二进制日志配置 log-bin=''
# 错误日志配置 log-err=''

[mysql]
default-character-set = utf8
```

### 2.3 主要配置文件

``二进制日志log-bin`` 主重复制

``错误日志log-error`` 默认是关闭的,记录严重的警告和错误信息,每次启动和关闭的详细信息等

``查询日志log`` 默认关闭,记录查询的sql语句，如果开启会减低mysql的整体性能，因为记录日志也是需要消耗系统资源的

#### 数据文件

``frm文件`` 存放表结构

``myd文件`` 存放表数据

``myi文件`` 存放表索引

## 3 MySQL 逻辑架构

和其他数据库相比，MySQL架构可以在多种不同场景中应用并发挥良好作用。主要体现在存储引擎的架构上，``插件式的存储引擎架构将查询处理和其他的系统任务以及数据的存储提取相分离``。这种架构可以根据业务的需求和实际需要选择合适的存储引擎。

![img](mysql基础.assets/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70.png)  

 											![20200327143425826](mysql基础.assets/20200327143425826.png)

![watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70-20200921215629234](mysql基础.assets/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70-20200921215629234.png)

![20200327143549542](mysql基础.assets/20200327143549542.png)

### 3.1 连接层

最上层是一些客户端和连接服务，包含本地sock通信和大多数基于客户端/服务端实现的类似于tcp/ip的通信。主要完成一些类似于连接处理、授权认证及相关的安全方案。在该层上引入了线程池的概念，为通过认证安全接入的客户端提供线程。同样在该层上可以实现基于SSL的安全链接。服务器也会为安全接入的每个客户端验证它所有的操作权限。

``Connectors`` 指的是不同语言中与SQL的交互

``Management Serveices & Utilities`` 系统管理和控制工具

``Connection Pool 连接池`` 管理缓冲用户连接，线程处理等需要缓存的需求。

负责监听对 MySQL Server 的各种请求，接收连接请求，转发所有连接请求到线程管理模块。每一个连接上 MySQL Server 的客户端请求都会被分配（或创建）一个连接线程为其单独服务。而连接线程的主要工作就是负责 MySQL Server 与客户端的通信，接受客户端的命令请求，传递 Server 端的结果信息等。线程管理模块则负责管理维护这些连接线程。包括线程的创建，线程的 cache 等。

### 3.2 服务层

第二层架构主要完成大多数的核心服务功能，如SQL接口，并完成缓存的查询，SQL的分析和优化及部分内置函数的执行，所有跨存储引擎的功能也在这一层实现，如过程、函数等。在该层，服务器会解析查询并创建相应的内部解析树，并对其完成相应的优化如确定查询表的顺序，是否利用索引等，最后生成相应的执行操作。如果是select语句，服务器还会查询内部的缓存，如果缓存空间足够大，这样在解决大量读操作的环境中能够很好的提升系统性能。

``SQL Interface: SQL 接口`` 接受用户的SQL命令，并且返回用户需要查询的结果。比如select from就是调用SQL Interface

``Parser: 解析器`` SQL命令传递到解析器的时候会被解析器验证和解析。解析器是由Lex和YACC实现的，是一个很长的脚本。

在 MySQL中我们习惯将所有 Client 端发送给 Server 端的命令都称为 query ，在 MySQL Server 里面，连接线程接收到客户端的一个 Query 后，会直接将该 query 传递给专门负责将各种 Query 进行分类然后转发给各个对应的处理模块。

主要功能：

1. 将SQL语句进行语义和语法的分析，分解成数据结构，然后按照不同的操作类型进行分类，然后做出针对性的转发到后续步骤，以后SQL语句的传递和处理就是基于这个结构的。

2. 如果在分解构成中遇到错误，那么就说明这个sql语句是不合理的

``Optimizer: 查询优化器`` SQL语句在查询之前会使用查询优化器对查询进行优化。就是优化客户端请求的 query（sql语句） ，根据客户端请求的 query 语句，和数据库中的一些统计信息，在一系列算法的基础上进行分析，得出一个最优的策略，告诉后面的程序如何取得这个 query 语句的结果

他使用的是“选取-投影-联接”策略进行查询。

​    用一个例子就可以理解： select uid,name from user where gender = 1;

​    这个select 查询先根据where 语句进行选取，而不是先将表全部查询出来以后再进行gender过滤

​    这个select查询先根据uid和name进行属性投影，而不是将属性全部取出以后再进行过滤

​    将这两个查询条件联接起来生成最终查询结果

``Cache和Buffer：查询缓存`` 他的主要功能是将客户端提交 给MySQL 的 Select 类 query 请求的返回结果集 cache 到内存中，与该 query 的一个 hash 值 做一个对应。该 Query 所取数据的基表发生任何数据的变化之后， MySQL 会自动使该 query 的Cache 失效。在读写比例非常高的应用系统中， Query Cache 对性能的提高是非常显著的。当然它对内存的消耗也是非常大的。

如果查询缓存有命中的查询结果，查询语句就可以直接去查询缓存中取数据。这个缓存机制是由一系列小缓存组成的。比如表缓存，记录缓存，key缓存，权限缓存等

### 3.3 引擎层

存储引擎层，存储引擎真正的负责了MySQL中数据的存储和提取，服务器通过API与存储进行通信，不同的存储引擎具有的功能不同，这样可以根据自己的实际需要进行选取。

``存储引擎接口`` 存储引擎接口模块可以说是 MySQL 数据库中最有特色的一点了。目前各种数据库产品中，基本上只有 MySQL 可以实现其底层数据存储引擎的插件式管理。这个模块实际上只是 一个抽象类，但正是因为它成功地将各种数据处理高度抽象化，才成就了今天 MySQL 可插拔存储引擎的特色。

MySQL区别于其他数据库的最重要的特点就是其插件式的表存储引擎。MySQL插件式的存储引擎架构提供了一系列标准的管理和服务支持，这些标准与存储引擎本身无关，可能是每个数据库系统本身都必需的，如SQL分析器和优化器等，而存储引擎是底层物理结构的实现，每个存储引擎开发者都可以按照自己的意愿来进行开发。

注意：存储引擎是基于表的，而不是数据库。

### 3.4 存储层

数据存储层，主要是将数据存储在运行于裸设备的文件系统之上，并完成于存储引擎的交互。

## 4 存储引擎

```mysql
# 查看MySQL现在提供什么存储引擎
show engines
# 查看当前存储引擎
show variables like '%storage_engine%'
```

#### 4.1 MyISAM 和 InnoDB

| 对比项 | MyISAM                                                     | InnoDB                                                       |
| ------ | ---------------------------------------------------------- | ------------------------------------------------------------ |
| 主外键 | 不支持                                                     | 支持                                                         |
| 事务   | 不支持                                                     | 支持                                                         |
| 行表锁 | 表锁，即使操作一条记录也会锁住整个表<br>不适合高并发的操作 | 行锁，操作时只锁某一行，不对其他行有影响<br>适合高并发操作   |
| 表空间 | 小                                                         | 大                                                           |
| 关注点 | 性能                                                       | 事务                                                         |
| 缓存   | 只缓存索引，不缓存缓存真实数据                             | 不仅缓存索引还要缓存真实数据<br>对内存要求较高，而且内存对性能有决定性影响 |
| 总行数 | 存储                                                       | 不存储                                                       |
| 索引   | 非聚集索引<br>支持全文索引，查询效率上MyISAM要高           | 聚集索引(索引的数据域存储数据文件本身)<br>不支持全文索引     |
| 持久化 | 一个表三个文件（索引文件，表结构文件，数据文件）           | 表空间                                                       |

















## 影响性能的因素



1. 人为因素 - 需求 count(*)，实时、准实时、有误差 
2. 程序员因素 - 面向对象 
3. cache 
4. 对可扩展过度追求 
5. 表范式 
<li>应用场景 OLTP，On-Line Transaction Processioning ![img](https://img-blog.csdnimg.cn/20200327162451591.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) OLAP，On-Line Analysis Processioning ![img](https://img-blog.csdnimg.cn/20200327163232325.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)</li>

## 提高性能

### 索引

#### 索引结构

MySql 支持的数据结构：

1. Hash，针对某个字段做索引，会对这个字段做Hash MySQL大面积不用Hash，首先有冲突，第二个无法做范围查询：

```java
select * from table where id > 1
```

1.  FullText 全文搜索  
2.  R-Tree 

### 锁

#### 行锁


![img](https://img-blog.csdnimg.cn/20200328133745662.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

#### 表锁


![img](https://img-blog.csdnimg.cn/20200328134121767.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

```java
# 表级锁争用状态变量
show status like 'table%'
# 行级锁争用状态变量
show status like 'innodb_row_lock%'
```



![img](https://img-blog.csdnimg.cn/20200328142825873.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) ![img](https://img-blog.csdnimg.cn/2020032814150452.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

### 优化

#### 原则


![img](https://img-blog.csdnimg.cn/20200328143235326.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

#### QEP

Query Execution Plan 打印执行计划，加上 explain：

```java
EXPLAIN SELECT * FROM user
```

先看一下在MySQL Explain功能中展示的各种信息的解释：

- ID：Query Optimizer 所选定的执行计划中查询的序列号 
- Select_type：所使用的查询类型，主要有以下这几种查询类型： 
 <ul> 
  - DEPENDENT SUBQUERY：子查询中内层的第一个 SELECT，依赖于外部查询的结果集 
  - DEPENDENT UNION：子查询中的UNION， 且为UNION中从第二个SELECT开始的后面所有SELECT，同样依赖于外部查询的结果集 
  - PRIMARY：子查询中的最外层查询，注意并不是主键查询 
  - SIMPLE：除子查询或者UNION之外的其他查询 
  - SUBQUERY：子查询内层查询的第一个SELECT, 结果不依赖于外部查询结果集 
  - UNCACHEABLE SUBQUERY：结果集无法缓存的子查询 
  - UNION：UNION语句中第二个SELECT开始的后面所有SELECT, 第一个SELECT为PRIMARY 
  - UNION RESULT：UNION中的合并结果 
 </ul>  
- Table：显示这一步所访问的数据库中的表的名称 
- Type：告诉我们对表所使用的访问方式，主要包含如下集中类型 
 <ul> 
  - all：全表扫描 
  - const：读常量，且最多只会有一条记录匹配,由于是常量，所以实际上只需要读一次 
  - eq_ ref：最多只会有一条匹配结果，一般是通过主键或者唯一键索引来访问 
  - fulltext 
  - index：全索引扫描 
  - index_ merge：查询中同时使用两个(或更多)索引，然后对索引结果进行merge之后再读取表数据 
  - index_subquery：子查询中的返回结果字段组合是一-个索引(或索引组合)，但不是一个主键或者唯一索引 
  - rang：索引范围扫描 
  - ref: Join：语中被驱动表索引引用查询 
  - ref_or_null：与ref的唯一区别就是在使用索引引用查询之外再增加一个空值的查询 
  - system：系统表，表中只有一行数据 
  - unique_ subquery：子查询中的返回结果字段组合是主键或者唯-约束 
  - 依次从好到差:system，const，eq_ref，ref，fulltext，ref_or_null， unique_subquery，index_subquery，range，index_merge，index，ALL 
 </ul>  
- Possible_ _keys: 该查询可以利用的索引，如果没有任何索引可以使用，就会显示成null,这一项内容对于优化时候索引的调整非常重要 
- Key: MySQL Query Optimizer 从possible_ keys 中所选择使用的索引 
- Key_ len: 被选中使用索引的索引键长度 
- Ref:列出是通过常量(const) ，还是某个表的某个字段(如果是join)来过滤(通过key)的 
- Rows: MySQL Query Opt imizer通过系统收集到的统计信息估算出来的结果集记录条数 
- Extra: 查询中每一步实现的额外细节信息，主要可能会是以下内容： 
 <ul> 
  - Distinct: 查找distinct值，所以当mysql找到了第一条匹配的结果后，将停止该值的查询而转为后面其他值的查询 
  - Full scan on NULL key:子查询中的一种优化方式，主要在遇到无法通过索引访问null值的使用使用 
  - Impossible WHERE noticed after reading const tables: MySQL Query Optimizer 通过收集到的统计信息判断出不可能存在结果 
  - No tables: Query语句中使用FROM DUAL或者不包含任何FROM子句 
  - Not exists: 在某些左连接中MySQL Query Opt imizer 所通过改变原有Query 的组成而使用的优化方法，可以部分减少数据访问次数 
  - Range checked for each record (index map: N): 通过MySQL官方手册的描述，当MySQL Query Optimizer 没有发现好的可以使用的索引的时候，如果发现如果来自前面的表的列值已知，可能部分索引可以使用。对前面的表的每个行组合，MySQL 检查是否可以使用range或index_merge 访问方法来索取行。 
  - Select tables optimized away: 当我们使用某些聚合函数来访问存在索引的某个字段的时候，MySQL Query Optimizer会通过索引而直接一次定 位到所需的数据行完成整个查询。当然，前提是在Query 中不能有GROUP BY操作。如使用MIN()或者MAX ()的时候; 
  - Using filesort: 当我们的Query 中包含ORDER BY操作，而且无法利用索引完成排序操作的时候，MySQL Query Opt imizer不得不选择相应的排序算法来实现。 
  - Usingindex:所需要的数据只需要在Index即可全部获得而不需要再到表中取数据 
  - Using index for group-by: 数据访问和Using index一样， 所需数据只需要读取索引即可，而当Query 中使用了GROUP BY或者DISTINCT 子句的时候，如果分组字段也在索引中，Extra 中的信息就会是Using index for group-by; 
  - Using temporary: 当MySQL 在某些操作中必须使用临时表的时候，在Extra信息中就会出现Using temporary 。主要常见于GROUP BY和ORDER BY等操作中。 
  - Using where: 如果我们不是读取表的所有数据，或者不是仅仅通过索引就可以获取所有需要的数据，则会出现Using where 信息; 
  - Using where with pushed condition: 这是一个仅仅在NDBCluster存储引擎中才会出现的信息，而且还需要通过打开Condition Pushdown 优化功能才可能会被使用。控制参数为 engine condition pushdown。 
 </ul> 

#### profiling(有个概念)

```java
set profiling=1;
select nick_name ,count(*) from user group by nick_name;
show profiles;
show profile cpu,block io for query 75;
```

### join \ order by \ group by 解释

#### join



![img](https://img-blog.csdnimg.cn/20200328155236811.png) ![img](https://img-blog.csdnimg.cn/20200328155954998.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

```java
# join
show variables like "join_%"
```

#### order by



![img](https://img-blog.csdnimg.cn/20200328163712558.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) ![img](https://img-blog.csdnimg.cn/20200328163111593.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) 如果sort_buffer能够承载所有的字段的时候，mysql就会自动选择第二种，如果不够就会使用第一种，第一种速度略逊于第一种，因为要两次读取数据，两次IO。

```java
# sort_buffer_size
show variables like "sort_buffer_%"
```

#### group by

前提是先排序

DISTINCT 基于group by LIMIT

```java
SELECT * FROM user limit 10000,10;
# limit慢的原因是，它要取 10010 条数据
SELECT * FROM user where id> 10000 limit 10;
```

#### Slow Sql 配置


![img](https://img-blog.csdnimg.cn/20200328170140712.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

#### 建索引的几大原则

1.最左前缀匹配原则，非常重要的原则，mysql会一直向右匹配直到遇到范围查询(&gt;、&lt;、between、like)就停止匹配，比如a = 1 and b = 2 and c &gt; 3 and d = 4 如果建立(a,b,c,d)顺序的索引，d是用不到索引的，如果建立(a,b,d,c)的索引则都可以用到，a,b,d的顺序可以任意调整。

2.=和in可以乱序，比如a = 1 and b = 2 and c = 3 建立(a,b,c)索引可以任意顺序，mysql的查询优化器会帮你优化成索引可以识别的形式。

3.尽量选择区分度高的列作为索引，区分度的公式是count(distinct col)/count(*)，表示字段不重复的比例，比例越大我们扫描的记录数越少，唯一键的区分度是1，而一些状态、性别字段可能在大数据面前区分度就是0，那可能有人会问，这个比例有什么经验值吗？使用场景不同，这个值也很难确定，一般需要join的字段我们都要求是0.1以上，即平均1条扫描10条记录。

4.索引列不能参与计算，保持列“干净”，比如from_unixtime(create_time) = ’2014-05-29’就不能使用到索引，原因很简单，b+树中存的都是数据表中的字段值，但进行检索时，需要把所有元素都应用函数才能比较，显然成本太大。所以语句应该写成create_time = unix_timestamp(’2014-05-29’)。

5.尽量的扩展索引，不要新建索引。比如表中已经有a的索引，现在要加(a,b)的索引，那么只需要修改原来的索引即可。

