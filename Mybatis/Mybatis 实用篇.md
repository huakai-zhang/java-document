## 认识 Mybatis

What is MyBatis? MyBatis is a first class persistence framework with support for custom SQL, stored procedures and advanced mappings. MyBatis eliminates almost all of the JDBC code and manual setting of parameters and retrieval of results. MyBatis can use simple XML or Annotations for configuration and map primitives, Map interfaces and Java POJOs (Plain Old Java Objects) to database records.- MyBatis 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。 ![img](https://img-blog.csdnimg.cn/20200402203209844.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70#pic_center)


## Generator 生成代码

### 引入 plugin

```java
<build>
    <plugins>
        <plugin>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-maven-plugin</artifactId>
            <version>1.3.3</version>
            <configuration>
                <configurationFile>${project.basedir}/src/main/resources/generator/generatorConfig.xml</configurationFile>
                <overwrite>true</overwrite>
            </configuration>
            <dependencies>
                <dependency>
                    <groupId>mysql</groupId>
                    <artifactId>mysql-connector-java</artifactId>
                    <version>5.1.31</version>
                </dependency>
            </dependencies>
        </plugin>
    </plugins>
</build>
```

### generatorConfig.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
   <context id="testTables" targetRuntime="MyBatis3">
      <commentGenerator>
         <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
         <property name="suppressAllComments" value="true" />
      </commentGenerator>
      <!--数据库连接的信息：驱动类、连接地址、用户名、密码 -->
      <jdbcConnection driverClass="com.mysql.jdbc.Driver"
         connectionURL="jdbc:mysql://127.0.0.1:3306/spring" userId="root"
         password="root">
      </jdbcConnection>

      <!-- 默认false，把JDBC DECIMAL 和 NUMERIC 类型解析为 Integer，为 true时把JDBC DECIMAL 和 
         NUMERIC 类型解析为java.math.BigDecimal -->
      <javaTypeResolver>
         <property name="forceBigDecimals" value="false" />
      </javaTypeResolver>

      <!-- targetProject:生成PO类的位置 -->
      <javaModelGenerator targetPackage="com.mybatis.po"
         targetProject=".\src\main\java\">
         <!-- enableSubPackages:是否让schema作为包的后缀 -->
         <property name="enableSubPackages" value="false" />
         <!-- 从数据库返回的值被清理前后的空格 -->
         <property name="trimStrings" value="true" />
      </javaModelGenerator>
        <!-- targetProject:mapper映射文件生成的位置 -->
      <sqlMapGenerator targetPackage="com.mybatis.mapper" 
         targetProject=".\src\main\java\">
         <!-- enableSubPackages:是否让schema作为包的后缀 -->
         <property name="enableSubPackages" value="false" />
      </sqlMapGenerator>
      <!-- targetPackage：mapper接口生成的位置 -->
      <javaClientGenerator type="XMLMAPPER"
         targetPackage="com.mybatis.mapper" 
         targetProject=".\src\main\java\">
         <!-- enableSubPackages:是否让schema作为包的后缀 -->
         <property name="enableSubPackages" value="false" />
      </javaClientGenerator>
      <!-- 指定数据库表 -->

      <table tableName="customer"></table>
      
      <!-- 有些表的字段需要指定java类型
       <table schema="" tableName="">
         <columnOverride column="" javaType="" />
      </table> -->
   </context>
</generatorConfiguration>
```

### 执行 mybatis-generator:generate


![img](https://img-blog.csdnimg.cn/20200402203914122.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70#pic_center)


## 通用配置

### 数据库连接池

db.properties:

```java
sysbase.mysql.jdbc.driverClassName=com.mysql.jdbc.Driver
sysbase.mysql.jdbc.url=jdbc:mysql://127.0.0.1:3306/spring?characterEncoding=UTF-8&rewriteBatchedStatements=true
sysbase.mysql.jdbc.username=root
sysbase.mysql.jdbc.password=root

dbPool.initialSize=1
dbPool.minIdle=1
dbPool.maxActive=200
dbPool.maxWait=60000
dbPool.timeBetweenEvictionRunsMillis=60000
dbPool.minEvictableIdleTimeMillis=300000
dbPool.validationQuery=SELECT 'x' 
dbPool.testWhileIdle=true
dbPool.testOnBorrow=false
dbPool.testOnReturn=false
dbPool.poolPreparedStatements=false
dbPool.maxPoolPreparedStatementPerConnectionSize=20
dbPool.filters=stat,log4j,wall
```

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd"
       default-init-method="start" default-destroy-method="stop">
    <bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="locations">
            <list>
                <!--配置jdbc-->
                <value>classpath:db.properties</value>
            </list>
        </property>
    </bean>

    <bean id="datasourcePool" abstract="true" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
        <property name="initialSize" value="${dbPool.initialSize}" />
        <property name="minIdle" value="${dbPool.minIdle}" />
        <property name="maxActive" value="${dbPool.maxActive}" />
        <property name="maxWait" value="${dbPool.maxWait}" />
        <property name="timeBetweenEvictionRunsMillis" value="${dbPool.timeBetweenEvictionRunsMillis}" />
        <property name="minEvictableIdleTimeMillis" value="${dbPool.minEvictableIdleTimeMillis}" />
        <property name="validationQuery" value="${dbPool.validationQuery}" />
        <property name="testWhileIdle" value="${dbPool.testWhileIdle}" />
        <property name="testOnBorrow" value="${dbPool.testOnBorrow}" />
        <property name="testOnReturn" value="${dbPool.testOnReturn}" />
        <property name="poolPreparedStatements" value="${dbPool.poolPreparedStatements}" />
        <property name="maxPoolPreparedStatementPerConnectionSize" value="${dbPool.maxPoolPreparedStatementPerConnectionSize}" />
        <property name="filters" value="${dbPool.filters}" />
    </bean>

    <bean id="dataSource" parent="datasourcePool">
        <property name="driverClassName" value="${sysbase.mysql.jdbc.driverClassName}" />
        <property name="url" value="${sysbase.mysql.jdbc.url}" />
        <property name="username" value="${sysbase.mysql.jdbc.username}" />
        <property name="password" value="${sysbase.mysql.jdbc.password}" />
    </bean>
</beans>
```

### mybatis.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
						http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--mybatis 创建session工厂-->
    <bean id="sessionFactoryBean" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <!--加载mybatis映射文件-->
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
        <!-- 加载mapper映射文件 -->
        <property name="mapperLocations" value="classpath:mapper/*.xml"/>
        <!-- 加载别名 -->
        <property name="typeAliasesPackage" value="com.mybatis.model"/>
    </bean>

    <bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg ref="sessionFactoryBean" />
    </bean>
    <!-- 扫包 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.mybatis.dao"/>
    </bean>

</beans>
```

### mybatis-config.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD SQL Map Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <!-- 全局映射器启用缓存 -->
        <setting name="cacheEnabled" value="true" />
        <setting name="useGeneratedKeys" value="true" />
        <setting name="defaultExecutorType" value="REUSE" />
        <setting name="callSettersOnNulls" value="true"/>
        <setting name="logImpl" value="STDOUT_LOGGING" />
    </settings>
</configuration>
```

实体类com.mybatis.model.User和接口com.mybaits.dao.UserMapper(包含List<User> selectAll())。

## XML

UserMapper.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.mybatis.dao.UserMapper">
    <resultMap id="BaseResultMap" type="com.mybatis.model.User">
        <id column="id" jdbcType="INTEGER" property="id" />
        <result column="name" jdbcType="VARCHAR" property="name"/>
    </resultMap>

    <select id="selectAll" resultMap="BaseResultMap">
        SELECT * FROM user
    </select>

</mapper>
```

## Annotation

```java
public interface UserMapper {
    @Select("SELECT * FROM user")
    List<User> selectAllByAnnotation();
}
```

## XML映射配置文件

 [配置详解](https://mybatis.org/mybatis-3/zh/configuration.html)

### 类型处理器(TypeHandlers)

MyBatis 在设置预处理语句（PreparedStatement）中的参数或从结果集中取出一个值时， 都会用类型处理器将获取到的值以合适的方式转换成 Java 类型。

#### 自定义TypeHandler

```java
public class TestTypeHandler extends BaseTypeHandler<String> {
    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, String parameter, JdbcType jdbcType) throws SQLException {
        ps.setString(i, parameter + "-Spring");
    }

    @Override
    public String getNullableResult(ResultSet rs, String columnName) throws SQLException {
        return rs.getString(columnName);
    }

    @Override
    public String getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        return rs.getString(columnIndex);
    }

    @Override
    public String getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
        return cs.getString(columnIndex);
    }
}
```

UserMapper.xml：

```java
<!--插入和更新也需要声明typeHandler-->
<insert id="insertUser" parameterType="com.mybatis.model.User">
    INSERT INTO user (name) VALUES (#{name, jdbcType=VARCHAR, typeHandler=com.mybatis.handler.TestTypeHandler})
</insert>
```

## plugins

MyBatis 允许在映射语句执行过程中的某一点进行拦截调用。默认情况下，MyBatis 允许使用插件来拦截的方法调用包括：

- Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed) 
- ParameterHandler (getParameterObject, setParameters) 
- ResultSetHandler (handleResultSets, handleOutputParameters) 
- StatementHandler (prepare, parameterize, batch, update, query)

### 实现org.apache.ibatis.plugin.Interceptor

```java
@Intercepts({@Signature(type = Executor.class, method = "query",
        args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class})})
public class SpringPlugins implements Interceptor {

    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        MappedStatement mappedStatement = (MappedStatement) invocation.getArgs()[0];
        Object parameter = null;
        if (invocation.getArgs().length > 1) {
            parameter = invocation.getArgs()[1];
        }

        BoundSql boundSql = mappedStatement.getBoundSql(parameter);
        System.out.println("SpringPlugins: " + boundSql.getSql());
        return invocation.proceed();
    }

    @Override
    public Object plugin(Object target) {
        return Plugin.wrap(target, this);
    }

    @Override
    public void setProperties(Properties properties) {

    }
}
```

### 配置mybatis-config.xml

```java
<plugins>
    <plugin interceptor="com.mybatis.plugins.SpringPlugins"/>
</plugins>
```

## 动态SQL

查询条件不确定，需要根据情况产生SQL语法，这种情况叫动态SQL。

## Batch批量操作

三种方式：

## 分页

### 逻辑分页

数据库返回的不是分页结果，而是全部数据，然后通过代码获取分页数据

```java
private void skipRows(ResultSet rs, RowBounds rowBounds) throws SQLException {
  if (rs.getType() != ResultSet.TYPE_FORWARD_ONLY) {
    if (rowBounds.getOffset() != RowBounds.NO_ROW_OFFSET) {
      rs.absolute(rowBounds.getOffset());
    }
  } else {
    for (int i = 0; i < rowBounds.getOffset(); i++) {
      rs.next();
    }
  }
}
```

### 物理分页

#### 拼接Sql(limit 0,10)

#### 分页插件

依赖：

```java
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>5.1.8</version>
</dependency>
```

配置插件：

```java
<plugins>
     <!-- com.github.pagehelper为PageHelper类所在包名 -->
     <plugin interceptor="com.github.pagehelper.PageInterceptor">
         <!-- 使用下面的方式配置参数，后面会有所有的参数介绍 -->
         <property name="helperDialect" value="mysql"/>
     </plugin>
 </plugins>
```

```java
@Test
public void test() {
    PageHelper.startPage(2, 10);
    List<User> userList = mapper.selectAll();
    System.out.println(userList);
}
```

## 联合查询

### 两种 result resultType vs resultMap

MyBatis中在查询进行select映射的时候，返回类型可以用resultType，也可以用resultMap，resultType是直接表示返回类型的(对应着我们的model对象中的实体)，而resultMap则是对外部ResultMap的引用(提前定义了db和model之间的隐射key–&gt;value关系)，但是resultType跟resultMap不能同时存在。 在MyBatis进行查询映射时，其实查询出来的每一个属性都是放在一个对应的Map里面的，其中键是属性名，值则是其对应的值。

- 当提供的返回类型属性是resultType时，MyBatis会将Map里面的键值对取出赋给resultType所指定的对象对应的属性。所以其实MyBatis的每一个查询映射的返回类型都是ResultMap，只是当提供的返回类型属性是resultType的时候，MyBatis对自动的给把对应的值赋给resultType所指定对象的属性。 
- 当提供的返回类型是resultMap时，因为Map不能很好表示领域模型，就需要自己再进一步的把它转化为对应的对象，这常常在复杂查询中很有作用。

### 嵌套查询和嵌套结果

```java
<!-- 嵌套查询 start -->
<resultMap id="BaseResultMap" type="com.spring.model.Blog">
    <id column="bid" jdbcType="INTEGER" property="bid" />
    <result column="name" jdbcType="VARCHAR" property="name"/>
    <association property="author" column="aid" select="com.spring.mybatis.dao.AuthorMapper.selectByPrimaryKey">

    </association>
</resultMap>

<select id="selectBlogAuthor" resultMap="BaseResultMap" parameterType="int">
    select * from blog
    where bid = #{bid}
</select>
<!-- 嵌套查询 end -->

<!-- 嵌套结果 start -->
<resultMap id="BlogResultMap" type="com.spring.model.Blog">
    <id column="bid" jdbcType="INTEGER" property="bid" />
    <result column="name" jdbcType="VARCHAR" property="name"/>
    <association property="author" javaType="com.spring.model.Author">
        <id column="aid" jdbcType="INTEGER" property="aid" />
        <result column="author_name" jdbcType="VARCHAR" property="authorName"/>
    </association>
</resultMap>

<select id="selectBlogAuthor2" resultMap="BlogResultMap" parameterType="int">
    select b.*,a.* from blog b, author a
    where b.bid = #{bid}
    and b.aid = a.aid
</select>
<!-- 嵌套结果 end -->

<!-- 嵌套查询 1：N start -->
<resultMap id="BlogPostsResultMap" type="com.spring.model.Blog">
    <id column="bid" jdbcType="INTEGER" property="bid" />
    <result column="name" jdbcType="VARCHAR" property="name"/>
    <collection column="bid" property="posts" select="selectByBlogId" ofType="com.spring.model.Posts">
    </collection>
</resultMap>

<select id="selectByBlogId" resultType="com.spring.model.Posts" parameterType="java.lang.Integer">
    select pid, post_name AS postName, bid
    from posts
    where bid = #{id}
</select>

<select id="selectBlogPosts" resultMap="BlogPostsResultMap" parameterType="int">
    select * from blog
    where bid = #{id}
</select>
<!-- 嵌套查询 1：N end -->
```

### 懒加载

不主动加载级联Mapper resultMap配置 mybatis-config.xml：

```java
<settings>
   <!-- 全局性设置懒加载。如果设为‘false'，则所有相关联的都会被初始化加载。 -->
   <setting name="lazyLoadingEnabled" value="true"/>
   <!-- 当设置为‘true'的时候，懒加载的对象可能被任何懒属性全部加载。否则，每个属性都按需加载。 -->
   <setting name="aggressiveLazyLoading" value="false"/>
</settings>
```

```java
@Test
public void test() {
	Blog blog = mapper.selectBlogPosts(1);
	System.out.println(blog.getName());
	Thread.sleep(5000);
	System.out.println(blog.getPosts().get(0).getBid());
}
```

日志:

```java
JDBC Connection [com.alibaba.druid.proxy.jdbc.ConnectionProxyImpl@3e7634b9] will not be managed by Spring
==>  Preparing: select * from blog where bid = ? 
==> Parameters: 1(Integer)
<==    Columns: bid, name, aid
<==        Row: 1, 花开, 1
<==      Total: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@4212a0c8]
花开

5s之后

JDBC Connection [com.alibaba.druid.proxy.jdbc.ConnectionProxyImpl@3e7634b9] will not be managed by Spring
==>  Preparing: select pid, post_name AS postName, bid from posts where bid = ? 
==> Parameters: 1(Integer)
<==    Columns: pid, postName, bid
<==        Row: 1, spring, 1
<==        Row: 2, mybatis, 1
<==      Total: 2
```

