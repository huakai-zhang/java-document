#### Q1 Mybatis中#和$的区别？

${}是Properties文件中的变量占位符(拼接)，它可以用于标签属性值和sql内部，属于静态文本替换，比如${driver}会被静态替换为com.mysql.jdbc.Driver。

\#{}是sql的参数占位符(预编译)，Mybatis会将sql中的#{}替换为?号，在sql执行前会使用PreparedStatement的参数设置方法，按序给sql的?号占位符设置参数值，比如ps.setInt(0, parameterValue)，#{item.name}的取值方式为使用反射从参数对象中获取item对象的name属性值，相当于param.getItem().getName()。

##### PrepareStatement 和 Statement 的区别？

两个都是接口，PrepareStatement 是继承自 Statement 的；

Statement 处理静态 SQL，PreparedStatement 主要用于执行带参数的语句； 

PreparedStatement 的 addBatch()方法一次性发送多个查询给数据库； 

PS 相似 SQL 只编译一次（对语句进行了缓存，相当于一个函数），减少编译次数；

PS 可以防止 SQL 注入；

MyBatis 默认值：PREPARED



#### Q2 Mybatis工作流程

1. 通过Reader对象读取src目录下的mybatis.xml配置文件(该文本的位置和名字可任意)
2. 通过SqlSessionFactoryBuilder对象创建SqlSessionFactory对象，从当前线程中获取SqlSession对象
3. 事务开始，通过SqlSession对象读取StudentMapper.xml映射文件中的操作编号，从而读取sql语句。调用 session.commit()提交事务
4. 调用 session.close()关闭会话，并且分开当前线程与SqlSession对象，让GC尽早回收



#### Q3 Mybatis 中一级缓存与二级缓存？

`一级缓存` 在同一个会话（SqlSession）中共享，默认开启，维护在 BaseExecutor 中。当 Session flush 或 close 之后，该 Session 中的所有 Cache 就将清空。每次查询spring会重新创建SqlSession，所以一级缓存是不生效的。

`二级缓存` 即全局缓存，默认关闭。在同一个 namespace 共享，需要在 Mapper.xml 中开启，维护在 CachingExecutor 中。



#### Q4 resultType 和 resultMap 的区别？

resultType 是标签的一个属性，适合简单对象（POJO、JDK 自带类型： Integer、String、Map 等），只能自动映射，适合单表简单查询。

resultMap 是一个可以被引用的标签，适合复杂对象，可指定映射关系，适合关联复合查询。



#### Q5 collection 和 association 的区别？

association：一对一

collection：一对多、多对多



#### Q6 Java 类型和数据库类型怎么实现相互映射？ 

通过 TypeHandler，例如 Java 类型中的 String 要保存成 varchar，就会自动调用相应的 Handler。如果没有系统自带的 TypeHandler，也可以自定义。



#### Q7 SIMPLE/REUSE/BATCH 三种执行器的区别？ 

SimpleExecutor 使用后直接关闭 Statement：closeStatement(stmt)

ReuseExecutor 放在缓存中，可复用：PrepareStatement——getStatement()

BatchExecutor 支持复用且可以批量执行 update()，通过 ps.addBatch()实现 handler.batch(stmt)



#### Q8 关联查询的延迟加载是怎么实现的？

动态代理（JAVASSIST、CGLIB），在创建实体类对象时进行代理，在调用代理 对象的相关方法时触发二次查询。



#### Q9 怎么解决表字段变化引起的 MBG 文件变化的问题？ 

Mapper 继承：自动生成的部分不变，创建接口继承原接口，创建 MapperExt.xml。在继承接口和 MapperExt.xml 中修改。 

通用 Mapper：提供支持泛型的通用 Mapper 接口，传入对象类型。



#### Q10 解析全局配置文件的时候，做了什么？ 

创建 Configuration，设置 Configuration 

解析 Mapper.xml，设置 MappedStatement，同时设置 MapperRegistry Map<Class<?>, MapperProxyFactory<? > > knownMappers



#### Q11 没有实现类，MyBatis 的方法是怎么执行的？

MapperProxy 代理，代理类的 invoke()方法中调用了 mapperMethod.execute() -> SqlSession.selectOne()



#### Q12 接口方法和映射器的 statement id 是怎么绑定起来的？

（怎么根据接口方法拿到 SQL 语句的？） 

Configuration Map<String, MappedStatement> mappedStatements

MappedStatement 对象中存储了 statement 和 SQL 的映射关系



#### Q13 四大对象是什么时候创建的？ 

Executor：openSession() 

StatementHandler、ResultsetHandler、ParameterHandler： 执行 SQL 时，在 SimpleExecutor 的 doQuery()中创建



#### Q14 MyBatis 哪些地方用到了代理模式？

接口查找 SQL：MapperProxy 

日志输出：ConnectionLogger、StatementLogger 

连接池：PooledDataSource 管理的 PooledConnection 

延迟加载：ProxyFactory（JAVASSIST、CGLIB） 

插件：Plugin 

Spring 集成：SqlSessionTemplate 的内部类 SqlSessionInterceptor



#### Q15 MyBatis 插件怎么编写和使用？原理是什么？

使用：继承 Interceptor 接口，加上注解，在 mybatis-config.xml 中配置 

原理：动态代理，责任链模式，使用 Plugin

创建代理对象 在被拦截对象的方法调用的时候，先走到 Plugin 的 invoke()方法，再走到 Interceptor 实现类的 intercept()方法，最后通过 Invocation.proceed()方法调用被拦截对象的原方法



#### Q16 MyBatis 集成到 Spring 的原理是什么？

SqlSessionTemplate 中有内部类SqlSessionInterceptor对DefaultSqlSession 进行代理； 

MapperFactoryBean 继 承 了 SqlSessionDaoSupport 获 取 SqlSessionTemplate；

接口注册到 IOC 容器中的 beanClass 是 MapperFactoryBean。 



#### Q17 DefaulSqlSession 和 SqlSessionTemplate 的区别是什么？

1）为什么 SqlSessionTemplate 是线程安全的？ 

其内部类 SqlSessionInterceptor 的 invoke()方法中的 getSqlSession()方法： 

如果当前线程已经有存在的 SqlSession 对象，会在 ThreadLocal 的容器中拿到 SqlSessionHolder，获取 DefaultSqlSession。 

如果没有，则会 new 一个 SqlSession，并且绑定到 SqlSessionHolder，放到 ThreadLocal 中。 

SqlSessionTemplate 中在同一个事务中使用同一个 SqlSession。 

调用 closeSqlSession()关闭会话时，如果存在事务，减少 holder 的引用计数。否 则直接关闭 SqlSession。 

2）在编程式的开发中，有什么方法保证 SqlSession 的线程安全？ 

SqlSessionManager 同时实现了 SqlSessionFactory、SqlSession 接口，通过 ThreadLocal 容器维护 SqlSession。



#### Q18 MyBatis 解决了什么问题？ 

或：为什么要用 MyBatis？ 或：MyBatis 的核心特性？ 

1）资源管理（底层对象封装和支持数据源） 

2）结果集自动映射 

3）SQL 与代码分离，集中管理 

4）参数映射和动态 SQL 

5）其他：缓存、插件等



#### Q19 MyBatis 编程式开发中的核心对象及其作用？

SqlSessionFactoryBuilder 创建工厂类 

SqlSessionFactory 创建会话 

SqlSession 提供操作接口 

MapperProxy 代理 Mapper 接口后，用于找到 SQL 执行



#### Q20 常见问题

1、用注解还是用 xml 配置？

注解的缺点是 SQL 无法集中管理，复杂的 SQL 很难配置。所以建议在业务复杂的项目中只使用 XML 配置的形式，业务简单的项目中可以使用注解和 XML 混用的形式。

2、如何实现模糊查询 LIKE？

字符串拼接 在 Java 代码中拼接%%（比如 name = "%" + name + "%"; ），存在 SQL 注入的风险

CONCAT（推荐）

bind 标签

```xml
<select id="getEmpList_bind" resultType="empResultMap" parameterType="Employee">
<bind name="pattern1" value="'%' + empName + '%'" />
<bind name="pattern2" value="'%' + email + '%'" />
	SELECT * FROM tbl_emp
	<where>
		<if test="empId != null">
			emp_id = #{empId,jdbcType=INTEGER},
		</if>
		<if test="empName != null and empName != ''">
			AND emp_name LIKE #{pattern1}
		</if>
		<if test="email != null and email != ''">
			AND email LIKE #{pattern2}
		</if>
	</where>
	ORDER BY emp_id
</select>
```

3、什么时候用#{}，什么时候用${}？

能用#的地方都用#

常量的替换，比如排序条件中的字段名称，不用加单引号，可以使用$

4、对象属性是基本类型 int double，数据库返回 null 是报错

使用包装类型。如 Integer，不要使用基本类型如 int

4、If test !=null 失效了？ 

在实体类中使用包装类型

5、XML 中怎么使用特殊符号，比如小于号

```
转义 &lt; （大于可以直接写）
使用   <![CDATA[ ]]>   当 XML 遇到这种格式就会把[]里面的内容原样输出，不进行解析
```

------

