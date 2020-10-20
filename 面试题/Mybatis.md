#### Q1 Mybatis中#和$的区别？

${}是Properties文件中的变量占位符，它可以用于标签属性值和sql内部，属于静态文本替换，比如${driver}会被静态替换为com.mysql.jdbc.Driver。

\#{}是sql的参数占位符，Mybatis会将sql中的#{}替换为?号，在sql执行前会使用PreparedStatement的参数设置方法，按序给sql的?号占位符设置参数值，比如ps.setInt(0, parameterValue)，#{item.name}的取值方式为使用反射从参数对象中获取item对象的name属性值，相当于param.getItem().getName()。



#### Q2 Mybatis工作流程

1. 通过Reader对象读取src目录下的mybatis.xml配置文件(该文本的位置和名字可任意)
2. 通过SqlSessionFactoryBuilder对象创建SqlSessionFactory对象，从当前线程中获取SqlSession对象
3. 事务开始，通过SqlSession对象读取StudentMapper.xml映射文件中的操作编号，从而读取sql语句。调用 session.commit()提交事务
4. 调用 session.close()关闭会话，并且分开当前线程与SqlSession对象，让GC尽早回收



#### Q3 Mybatis 中一级缓存与二级缓存？

`一级缓存` 基于 PerpetualCache 的 HashMap 本地缓存，其存储作用域为 Session，当 Session flush 或 close 之后，该 Session 中的所有 Cache 就将清空。每次查询spring会重新创建SqlSession，所以一级缓存是不生效的。

`二级缓存` 即全局缓存，其作用域为 Mapper(Namespace)，默认关闭。

