## 认识Mysql

1985年，瑞典的几位志同道合小伙子(以 David Axmark 为首)成立了一 家公司，这就是 MySQL AB 的前身。这个公司最初并不是为了开发数据库产品，而是在实现他们想法的过程中，需要一个数据库。他们希望能够使用开源的产品。但在当时并没-一个合适的选择，没办法，那就自己开发吧。

## 架构图


![img](https://img-blog.csdnimg.cn/20200327142128826.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

## 详情





![img](https://img-blog.csdnimg.cn/20200327142905661.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) ![img](https://img-blog.csdnimg.cn/20200327143425826.png) ![img](https://img-blog.csdnimg.cn/20200327143314693.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) ![img](https://img-blog.csdnimg.cn/20200327143549542.png)

## 引擎介绍


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

