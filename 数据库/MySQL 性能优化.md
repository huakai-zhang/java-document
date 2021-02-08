

## 5 性能优化

### 5.1 MySQL Query Optimizer

MySQL 中有专门负责优化SELECT语句的优化器模块，主要功能：通过计算分析系统中收集到的统计信息，为客户端请求的Query提供它认为最优的执行计划（它认为最优的数据检索方式，但不见得是DBA认为是最优的，这部分最耗费时间）。

当客户端向MySQL请求一条Query，命令解析器模块完成请求分类，区别出是SELECT并转发给MySQL Query Optimizer时，MySQL Query Optimizer 首先会对整条Query进行优化，处理掉一些常量表达式的预算，直接换算成常量值。并对 Query 中的查询条件进行简化和转换，如去掉一些无用或显而易见的条件、结构调整等。然后分析 Query 中的 Hint 信息（如果有），看显示Hint信息是否可以完全确定该Query的执行计划。如果没有Hint或Hint信息还不足以完全确定执行计划，则会读取所涉及对象的统计信息，根据Query进行写相应的计算分析，然后再得出最后的执行计划。

### 5.2 MySQL 常见性能瓶颈

``CPU`` CPU在饱和的时候一般发生在数据装入在内存或从磁盘上读取数据时候

``IO`` 磁盘I/O瓶颈发生在装入数据远大于内存容量时

``服务器硬件的性能瓶颈`` top,free,iostat和vmstat来查看系统的性能状态

### 5.3 EXPLAIN

``EXPLAIN(执行计划)`` 用EXPLAIN关键字可以模拟优化器执行SQL语句，从而知道MySQL是如何处理你的SQL语句的。

分析你的查询语句或是结构的性能瓶颈，主要功能：

* 表的读取顺序
* 数据读取操作的操作类型
* 哪些索引可以使用
* 哪些索引被实际使用
* 表之间的引用
* 每张表有多少行被优化器查询

``QEP(Query Execution Plan)`` 打印执行计划，加上 explain：

```mysql
EXPLAIN SELECT * FROM user
```

![image-20200923195934590](MySQL 性能优化.assets/image-20200923195934590.png)

#### id

select 查询的序列号，包含一组数字，表示查询中执行select子句或操作表的顺序。

id相同，执行顺序由上至下：

![image-20200923200411220](MySQL 性能优化.assets/image-20200923200411220.png)

id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行：

![image-20200923200713777](MySQL 性能优化.assets/image-20200923200713777.png)

id相同不同同时存在：

![image-20200923201206002](MySQL 性能优化.assets/image-20200923201206002.png)

id如果相同，可以认为是一组，从上往下执行；在所有组中，id值越大，优先级越高，越先执行。

``derived 衍生``

#### select_type

查询的类型，主要用于区别普通查询、联合查询、子查询等的复杂查询，主要有以下这几种查询类型：

* ``SIMPLE`` 简单的 select 查询，不包含子查询或 UNION
* ``PRIMARY`` 查询中若包含任何复杂的子查询，最外层查询则被标记为PRIMARY 

 - ``SUBQUERY``  在select 或 where 列表中包含了子查询
 - ``DERIVED`` 在from列表中包含的子查询被标记为DETIVED，MySQL 会递归执行这些子查询，把结果放在临时表里 
  - ``UNION`` 若第二个 select 出现在 UNION 之后，则被标记为 UNION；若UNION包含在FROM子句的子查询中，外层 select 将被标记为：DERIVED
  - ``UNION RESULT`` 从 UNION 表获取结果的 select 

#### table

显示这一行的数据的表的名称 

#### type

访问类型，显示查询使用了何种类型。

从最好到最差依次是：system>const>eq_ref>ref>range>index>ALL 

一般来说，得保证查询至少达到range级别，最好能达到ref。


  - ``system`` 表只有一行记录（等于系统表）这是const类型的特例，平时很少出现 

  - ``const`` 表示通过索引一次就能找到（单表），const用于比较primary key 或者unique索引。因为只匹配一行数据，所以很快。如将逐渐置于where列表中，MySQL 就能将该查询转换为一个常量

    ```mysql
    EXPLAIN SELECT * FROM tb_emp
    WHERE tb_emp.id = 1
    ```

  - ``eq_ ref`` 唯一性索引扫描，对于每个索引键，表中只会有一条匹配结果(对于前表的每一行，后表只有一行被扫描)，常见于主键或者唯一键索引扫描

    ```mysql
    EXPLAIN SELECT * FROM tb_emp,tb_dept
    WHERE tb_emp.deptId = tb_dept.id
    ```

  - ``ref`` 非唯一索引扫描，返回匹配某个单独值的所有行。本质上也是一种索引访问，它返回所有匹配单个单独值的行，然而，它可能会找到多个符合条件的行，所以应该属于查找和扫描的混合体

    ```mysql
    # 为 name 列创建普通索引
    EXPLAIN SELECT * FROM tb_emp
    WHERE tb_emp.name = 'z3'
    ```

  - ``range`` 只检索给定范围的行，使用一个索引来选择行。key列显示使用了哪个索引。一般就是在你的where语句中出现了between、<、>、in等的查询，这种范围扫描索引扫描比全表扫描要好，因为他只需要开始索引的某一点，而结束语另一点，不用扫描全部索引（可能和最终得到数的结果有关，数量多为ALL）

    ```mysql
    # 为 deptId 列创建普通索引
    EXPLAIN SELECT * FROM tb_emp
    WHERE deptId > 3
    ```

  - ``index`` Full Index Scan,index与ALL区别为index类型只遍历索引树。这通常比ALL快，因为索引文件通常比数据文件小。

    （也就是说虽然all和index都是读全表，但index是从索引中读取的，而all是从硬盘中读的）

    ```mysql
    EXPLAIN SELECT id FROM tb_emp
    EXPLAIN SELECT deptId FROM tb_emp
    ```

  - ``all`` Full Table Scan，全表扫描 

  - 依次从好到差:system，const，eq_ref，ref，fulltext，ref_or_null， unique_subquery，index_subquery，range，index_merge，index，ALL 

#### possible_ _keys

显示可能应用在这张表中的索引，一个或多个。

查询涉及到的字段上若存在索引，则该索引被列出，但不一定被查询实际使用。如果没有任何索引可以使用，就会显示成null。

#### key

实际使用的索引。如果为null则没有使用索引，查询中若使用了``覆盖索引``，则索引和查询的select字段重叠。

#### key_ len

表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。在不损失精确性的情况下，长度越短越好。

key_len显示的值为索引最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的。

#### ref

显示索引``那一列``被使用了，如果可能的话，是一个常数。那些列或常量被用于查找索引列上的值。

查询中与其他表关联的字段，外键关系建立索引。

#### rows

根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数。

#### extra

包含不适合在其他列中显示但十分重要的额外信息：


  - ``using filesort`` 说明 MySQL 会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。MySQL 中无法利用索引完成的排序操作称为``文件排序``。

  - ``using temporary`` 使用了临时表保存中间结果，MySQL在对查询结果排序时使用临时表。常见于排序order by 和分组查询 group by。

  - ``using index`` 表示相应的select操作中使用了``覆盖索引（Coveing Index）``，避免访问了表的数据行，效率不错！
    如果同时出现 using where，表明索引被用来执行索引键值的查找；
    如果没有同时出现 using where，表面索引用来读取数据而非执行查找动作。

    覆盖索引（Coveing Index），一说索引覆盖。就是 select 的数据列只用从索引中就能够得到，不必读取数据行，MySQL 可以利用索引返回 select 列表中的字段，而不必根据索引再次读取数据文件，换句话说查询列要被所建的索引覆盖。

    注意：

    如果要使用覆盖索引，一定要注意 select列表中只取出需要的列，不可 select *，因为如果将所有字段一起做索引会导致索引文件过大，查询性能下降。

  - ``using where`` 表面使用了where过滤

  - ``using join buffer`` 使用了连接缓存

  - ``impossible where`` where子句的值总是false，不能用来获取任何元组

  - ``select tables optimized away`` 在没有GROUPBY子句的情况下，基于索引优化MIN/MAX操作或者对于MyISAM存储引擎优化COUNT(*)操作，不必等到执行阶段再进行计算，查询执行计划生成的阶段即完成优化。

  - ``distinct`` 优化distinct，在找到第一匹配的元组后即停止找同样值的工作

### 5.4 示例分析

![1600939704485](MySQL 性能优化.assets/1600939704485.png)

第一行（执行顺序4）：id列为1，表示是union里的第一个select，select_tyep列的primary

第二行（执行顺序2）：id列为3，是整个查询中第三个select的一部分。因查询包含在from中，所以为derived。【select id,name from t1 where other_column=''】

第三行（执行顺序3）：select列表中的子查询select_type为subquery，为整个查询中的第二个select。【select id from t3】

第四行（执行顺序1）：select_type为union，说明第四个select是union里的第二个select，最先执行【select name,id from t2】

第五行（执行顺序5）：代表从union的临时表中读取行的阶段，table列的<union1,4>表示用第一个和第四个select的结果进行union操作。【两个结果union操作】

### 5.5 性能优化一般流程

1. 开启慢查询日志（设置阙值，如超过5秒就是慢SQL，抓取出来）并捕获
2. explain + 慢SQL分析
3. show profile 查询SQL在MySQL服务器里面的执行细节和生命周期情况
4. SQL数据库服务器的参数调优



## 2 查询截取优化

### 2.1 查询优化

#### 2.1.1 永远小表驱动大表

即小的数据集驱动大的数据集，类似嵌套循环Nested Loop

```mysql
select * from A where id in (select id from B)
# 等价于
for select id from B
for select * from A where A.id = B.id
```

当B表的数据集必须小于A表的数据集时，用in优于exists

```mysql
select * from A where exists (select 1 from B where B.id = A.id)
# 等价于
for select * from A
for select * from B where B.id = A.id
```

当A表的数据集小于B表的数据集时，用exists优于in

##### EXISTS

SELECT ... FROM table WHERE EXISTS(subquery)

该语法可以理解为：将主查询的数据，放到子查询中做条件验证，根据验证结果来决定主查询的数据结果是否得以保留。

1. EXISTS(subquery)只返回TRUE和FALSE，因此子查询中的SELECT * 也可以是SELECT 1 或者其他，官方说法是实际执行时会忽略SELECT清单，因此没有区别
2. EXISTS子查询的实际执行过程可能经过了优化而不是我们理解上的逐条对比，如果担心效率问题，可以进行实际检验已确定是否有效率问题
3. EXISTS子查询往往也可以用条件表达式、其他子查询或者JOIN替代，何种最优需要具体问题具体分析

#### 2.1.2 order by 关键字优化

ORDER BY子句，尽量使用Index方式排序，避免使用FileSort方式排序

```mysql
CREATE TABLE `tba` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `age` int(11) DEFAULT NULL,
  `birth` timestamp NULL DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;

INSERT INTO tbA(age, birth) VALUES(22, NOW());
INSERT INTO tbA(age, birth) VALUES(23, NOW());
INSERT INTO tbA(age, birth) VALUES(24, NOW());

CREATE INDEX idx_A_ageBirth ON tbA(age, birth);

# Using where; Using index
explain select * from tbA where age > 20 order by age;
explain select * from tbA where age > 20 order by age, birth;
explain select * from tbA WHERE birth > '2020-09-25 00:00:00' order by age;
# Using where; Using index; Using filesort
explain select * from tbA where age > 20 order by birth;
explain select * from tbA where age > 20 order by birth, age;
explain select * from tbA WHERE birth > '2020-09-25 00:00:00' order by birth;
#Using index; Using filesort
explain select * from tbA order by birth;
explain select * from tbA order by age ASC, birth DESC;
```

MySQL支持二种方式的排序，FileSort和Index，Index效率高。它指MySQL扫描索引本身完成排序。FileSort方式效率较低。

ORDER BY满足两情况，会使用Index方式排序：

1. ORDER BY语句使用索引最左前列

2. 使用where子句与OrderBy子句条件列组合满足索引最左前列，尽可能在索引列上完成排序操作，遵照索引建的最佳左前缀

##### filesort

![img](MySQL 性能优化.assets/20200328163111593.png)

如果sort_buffer能够承载所有的字段的时候，mysql就会自动选择第二种，如果不够就会使用第一种，第一种速度略逊于第一种，因为要两次读取数据，两次IO。

如果不在索引列上，filesort有两种算法，双路排序和单路排序：

``双路排序`` MySQL4.1之前是使用双路排序，字面意思是两次扫描磁盘，最终得到数据。读取行指针和orderby列，对他们进行排序，然后扫描已经排序好的列表，按照列表中的值重新从列表中读取对应的数据传输。

从磁盘取排序字段，在buffer进行排序，再从磁盘取其他字段。

取一批数据，要对磁盘进行两次扫描，众所周知，I\O是很耗时的，所以在mysql4.1之后，出现了第二张改进的算法，就是单路排序。

``单路排序`` 从磁盘读取查询需要的所有列，按照orderby列在buffer对它们进行排序，然后扫描排序后的列表进行输出，它的效率更快一些，避免了第二次读取数据，并且把随机IO变成顺序IO，但是它会使用更多的空间，因为它把每一行都保存在内存中了。

**单路排序引申出的问题**

由于单路是后出来的，总体而言好过双路，但是用单路有问题：

在sort_buffer中，单路排序比双路排序占用空间多，因为单路排序是把所有字段都取出，所有有可能去除的数据的总大小超出了sort_buffer的容量，导致每次只能取sort_buffer容量大小的数据，进行排序（创建tmp文件，多路合并），排再取sort_buffer容量大小，再排......从而多次I/O。

**优化策略**

增大sort_buffer_size参数的设置

增大max_length_for_sort_data参数的设置

##### 提高Order By的速度

1. Order By时select * 是一个大忌，只query需要的字段，这点非常重要：

   当query的字段大小总和小于max_length_for_sort_data而排序字段不是TEXT|BLOB类型时，会用单路排序，否则用多路排序

   两种算法的数据都有可能超出sort_buffer的容量，超出之后，会创建tmp文件进行合并排序，导致多次I/O，但是用单路排序算法风险会更大，所以要提高sort_buffer_size

2. 尝试提高 sort_buffer_size

   不管用哪种算法，提高这个参数都会提高效率，当然，要根据系统的能力去提高，因为这个参数是针对每个进程的

3. 尝试提高 max_length_for_sort_data

   提高这个参数，会增加用改进算法的概率。但是如果设的太高，数据总容量超出sort_buffer_size的概率就增大，明显症状是高的磁盘I/O活动和低的处理器使用率

##### 为排序使用索引

MySQL两种排序方式：文件排序或扫描有序索引排序

MySQL能为排序与查询使用相同的索引

```markdown
KEY a_b_c(a,b,c)
# order by 能使用索引最左前缀
	ORDER BY a
	ORDER BY a, b
	ORDER BY a, b, c
	ORDER BY a DESC, b DESC, c DESC

# 如果where使用索引的最左前缀定义为常量，则order by能使用索引
	WHERE a = const ORDER BY b, c
	WHERE a = const AND b = const ORDER BY c
	WHERE a = const AND b > const ORDER BY b, c

# 不能使用索引进行排序
	ORDER BY a ASC, b DESC, c DESC /*排序不一致*/
	WHERE g = const ORDER BY b, c  /*丢失a索引*/
	WHERE a = const ORDER BY c     /*丢失b索引*/
	WHERE a = const ORDER BY a, d  /*d不是索引的一部分*/
	WHERE a in (...) ORDER BY b, c /*对于排序来说，多个相等条件也是范围查询*/
```

#### 2.1.3 group by 关键字优化

groupby实质是先排序后进行分组，遵照索引建的最佳左前缀。

当无法使用索引列，增大max_length_for_sort_data参数的设置 + 增大sort_buffer_size参数的设置。

where高于having,能写在where限定的条件就不要去having限定了。

### 2.2 慢查询日志

MySQL的慢查询日志是MySQL提供的一种日志记录，它用来记录在MySQL中响应时间超过阙值的语句，具体指运行时间超过``long_query_time``值的SQL，则会被记录到慢查询日志中。

long_query_time默认值为10，意思是运行10秒以上的语句。

由慢查询日志来查看哪些SQL超出了我们最大忍耐时间值，比如一条SQL执行查过5秒，就算慢SQL，希望能收集超过5秒的SQL，结合之前explain进行全面分析。

默认情况下，MySQL数据库没有开启慢查询日志，需要手动来设置这个参数。当然如果不是调优需要的话，一般不建议开启该参数，因为开启慢查询日志或多或少带来一定的性能影响，慢查询日志支持将日志记录写入文件。

#### 2.2.1 开启慢查询日志

```mysql
# 查看是否开启
SHOW VARIABLES LIKE '%slow_query_log%';
```

![image-20200926152636280](MySQL 性能优化.assets/image-20200926152636280.png)

默认情况下``slow_query_log``的值为OFF，表示慢查询日志是禁用的。

```mysql
# 使用下面语句开启慢查询日志只对当前数据库生效，如果MySQL重启后则会失效
set global slow_query_log = 1;
```

如果要永久生效，就必须修改配置文件m y.cnf，在[mysqld]下增加或修改参数，然后重启MySQL服务器：

```markdown
slow_query_log=1
slow_query_log_file=/usr/local/mysql/data/spring-slow.log
long_query_time=3
log_output=FILE
```

关于慢查询的参数slow_query_log_file，它指定慢查询日志的存放路径，系统默认会给一个缺省的文件host_name-slow.log(如果没有指定参数slow_query_log_file的话)。

```mysql
# 开启慢查询日志后，什么样的SQL参会记录到慢查询里面？
SHOW VARIABLES LIKE 'long_query_time%';
show global variables like 'long_query_time';

# 设置慢查询SQL的阙值时间，也可以在my.cnf参数里面修改
# 需要重新连接或者新开一个回话才能看到修改值
set global long_query_time=3;
```

假如运行时间正好等于long_query_time的情况，并不会被记录下来。也就是说，在MySQL源码里是``判断大雨long_query_time，并非大不等于``。

#### 2.2.2 记录慢SQL

```mysql
SELECT SLEEP(4);
```

查看对应的slow_query_log_file：

```
D:\Mysql5.5\bin\mysqld, Version: 5.5.28 (MySQL Community Server (GPL)). started with:
TCP Port: 3306, Named Pipe: MySQL
Time                 Id Command    Argument
# Time: 200926 15:50:58
# User@Host: root[root] @ localhost [127.0.0.1]
# Query_time: 4.000666  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 0
use test;
SET timestamp=1601106658;
#SHOW VARIABLES LIKE 'long_query_time%';
#SHOW VARIABLES LIKE '%slow_query_log%';
SELECT SLEEP(4);
```

#### 2.2.3 查看当前系统有多少条慢查询记录

```mysql
show global status like '%Slow_queries%';

# Variable_name		Value
# Slow_queries		1
```

#### 2.2.4 日志分析工具mysqldumpshow

在生产环境中，如果要手工分析日志，查找、分析SQL，显然是一个体力活，MySQL提供了日志分析工具mysqldumpshow。

```shell
mysqldumpslow --help
```

![image-20200926161015073](MySQL 性能优化.assets/image-20200926161015073.png)

s:是表示按何种方式排序

c:访问次数

l:锁定时间

r:返回记录

t:查询时间

al:平均锁定时间

ar:平均返回记录数

at:平均查询时间

t:即为返回前面多少条的数据

g:后边搭配一个正则匹配模式，大小写不敏感的

```shell
# 得到返回记录集最多的10个SQL
mysqldumpslow -s r -t 10 /var/lib/mysql/spring-slow.log
# 得到访问次数最多的10个SQL
mysqldumpslow -s c -t 10 /var/lib/mysql/spring-slow.log
# 得到按照时间排序的前10条里面含有左连接的查询语句
mysqldumpslow -s t -t 10 -g "left join" /var/lib/mysql/spring-slow.log
# 另外建议在使用这些命令时结合 ｜ 和 more 使用，否则可能出现爆屏情况
mysqldumpslow -s r -t 10 /var/lib/mysql/spring-slow.log ｜ more
```

### 2.3 批量数据脚本

往表里插入1000W数据

```mysql
# 建表
CREATE TABLE `dept` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `deptno` mediumint(9) DEFAULT NULL,
  `dname` varchar(20) DEFAULT NULL,
  `loc` varchar(13) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `emp` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `empno` mediumint(9) DEFAULT NULL COMMENT '编号',
  `ename` varchar(20) DEFAULT NULL COMMENT '名字',
  `job` varchar(9) DEFAULT NULL COMMENT '工作',
  `mgr` mediumint(9) DEFAULT '0' COMMENT '上级编号',
  `hiredate` date DEFAULT NULL COMMENT '入职时间',
  `sal` decimal(7,2) DEFAULT NULL COMMENT '薪水',
  `comn` decimal(7,2) DEFAULT NULL COMMENT '红利',
  `deptno` mediumint(9) DEFAULT NULL COMMENT '部门编号',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

#### 3.3.1 设置参数log_trust_function_createors

创建函数，假如报错：This function has none of DETERMINISTIC......

由于开启过慢查询日志，因为我们开启了bin-log，就必须为function指定一个参数。

```mysql
SHOW VARIABLES LIKE 'log_bin_trust_function_creators';
set global log_bin_trust_function_creators=1;
```

#### 2.3.2 创建函数

```mysql
# 创建函数保证每条数据都不同
# 随机参数指定长度的字符串
DELIMITER $$
CREATE FUNCTION rand_string(n INT) RETURNS VARCHAR(255)
BEGIN
	DECLARE chars_str VARCHAR(100) DEFAULT 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
	DECLARE return_str VARCHAR(255) DEFAULT '';
	DECLARE i INT DEFAULT 0;
	WHILE i < n DO
	SET return_str = CONCAT(return_str,SUBSTRING(chars_str, FLOOR(1+RAND()*52), 1));
	SET i = i + 1;
	END WHILE;
	RETURN return_str;
END $$

# 随机产生100~110之间的整数
DELIMITER $$
CREATE FUNCTION rand_num() RETURNS INT(5)
BEGIN
	DECLARE i INT DEFAULT 0;
	SET i = FLOOR(100+RAND()*10);
	RETURN i;
END $$

SELECT rand_string(10);
SELECT rand_num();

# 删除函数
DROP FUNCTION rand_num;
```

#### 2.3.3 创建存储过程

```mysql
# 创建往emp表中插入数据的存储过程
DELIMITER $$
CREATE PROCEDURE insert_emp(IN START INT(10), IN max_num INT(10))
BEGIN
	DECLARE i INT DEFAULT 0;
	# set autocommit = 0 把autocommit设置成0
	SET autocommit = 0;
	REPEAT
	SET i = i + 1;
	INSERT INTO emp(empno, ename, job, mgr, hiredate, sal, comn, deptno) VALUES ((START+i), rand_string(6), 'SALESMAN', 0001, CURDATE(), 2000, 400, rand_num());
	UNTIL i = max_num
	END REPEAT;
	COMMIT;
END $$

# 创建往dept表中插入数据的存储过程
DELIMITER $$
CREATE PROCEDURE insert_dept(IN START INT(10), IN max_num INT(10))
BEGIN
	DECLARE i INT DEFAULT 0;
	SET autocommit = 0;
	REPEAT
	SET i = i + 1;
	INSERT INTO dept(deptno, dname, loc) VALUES ((START+i), rand_string(10), rand_string(8));
	UNTIL i = max_num
	END REPEAT;
	COMMIT;
END $$
```

#### 2.3.4 调用存储过程

```mysql
CALL insert_dept(100, 10);
CALL insert_emp(100001, 500000);
```

### 2.4 Show profiles

``Show profiles`` 是mysql提供可以用来分析当前会话中语句执行的资源消耗情况。可以用于SQL的调优测量。

官网：https://dev.mysql.com/doc/refman/8.0/en/show-profile.html

默认情况下，参数处于``关闭状态，并保存最近15次``的运行结果。

```mysql
# 是否支持，看看当前的SQL版本是否支持
show variables like 'profiling';
# 开启功能，默认是关闭，使用前需要开启
set profiling=on;

# 运行SQL
SELECT * from tb_emp e INNER JOIN tb_dept d on e.deptId = d.id;
select * from emp group by id%10 limit 150000;
select * from emp group by id%20 order by 5;

# 查看结果
show profiles;
```

![1601199549766](MySQL 性能优化.assets/1601199549766.png)

```mysql
# 诊断SQL，show profile cpu,block io for query 上一步前面的问题SQL数字号码;
show profile cpu,block io for query 3;
```

![1601199661301](MySQL 性能优化.assets/1601199661301.png)

参数备注：

``ALL`` 显示所有的开销信息

``BLOCK IO`` 显示块IO相关开销

``CONTEXT SWITCHES`` 上下文切换相关开销

``CPU`` 显示CPU相关开销信息

``IPC`` 显示发送和接收相关开销信息

``MEMORY`` 显示内存相关开销信息

``PAGE FAULTS`` 显示页面错误相关开销信息

``SOURCE`` 显示和Source_function，Source_file，Source_line相关的开销信息

``SWAPS`` 显示交换次数相关开销信息

**日常开发注意点**

``converting HEAP to MyISAM`` 查询结果太大，内存都不够用了往磁盘上搬了

``Creating tmp table`` 创建临时表，拷贝数据到临时表，用完再删除

``Copying to tmp table on disk`` 把内存中临时表复制到磁盘，危险！！！

``locked``

### 2.5 全局查询日志

**配置启用**

在 MySQL 的 my.cnf中，设置如下：

```
# 开启
general_log=1
# 记录日志文件的路径
general_log_file=/path/logfile
# 输出格式
log_output=FILE
```

**编码启用**

```mysql
set global general_log=1;
set global log_output='TABLE';

# 此后，所编写的SQL语句，就会记录到mysql库里的general_log表，可以用下面的命令查看
select * from mysql.general_log;
```

``永远不要在生产环境开启这个功能。``