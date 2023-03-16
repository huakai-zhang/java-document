# java.sql.DriverManager
static Connection getConnection(String url, String user, String password)
建立一个到指定数据库的连接，并返回一个Connection对象

# java.sql.Connection
Statement createStatement()
创建一个Statement对象，用以执行不带参数的SQL查询和更新
void close()
立即关闭当前的连接，并释放由它创建的JDBC资源
PreparedStatement preparedStatement(String sql)
返回一个含预编译语句的PreparedStatement对象。字符串sql代表一个SQL语句，该语句可以包含一个或多个由？字符指明的参数占位符
Blob createBlob()
Clob createClob()
创建一个空的BLOB或CLOB
Statement createStatement(int type, int concurrency)
PreparedStatement preparedStatement(String command, int type, int concurrency)
创建一个语句或预备语句，且该语句可以产生指定类型和并发模式的结果集
DatabaseMetaData getMetaData()
返回一个DatabaseMetaData对象，该对象封装了有关数据库连接的元数据
boolean getAutoCommit()
void setAutoCommit(boolean b)
获取该连接中的自动提交模式，或将其设置为b。如果自动更新为true，那么所有语句将在执行结束后立刻被提交。
void commit()
提交自上次提交以来所有执行过的语句
void rollback()
撤销自上次提交以来所有执行过的语句所产生的影响
Savepoint setSavepoint()
Savepoint setSavepoint(String name)
设置一个匿名或具名的保存点
void rollback(ISavepoint svpt)
回滚到给定保存点
void releaseSavepoint(Savepoint svpt)
释放给定的保存点

# java.sql.Statement
ResultSet executeQuery(String sqlQuery)
执行给定字符串中的SQL语句，并返回一个用于查看查询结果的ResultSet对象
int executeUpdate(String sqlStatement)
long executeLargeUpdate(String sqlStatement)
执行字符串中指定的INSERT、UPDATE和DELETE等SQL语句。还可以执行数据定义语言（Data Definition Language，DDL）语句，如CREATE TABLE。返回受影响的行数，如果是没有更新计数的语句，则返回0
boolean execute(String sqlStatement)
执行字符串中指定的SQL语句，可能会产生多个结果集和更新计数，如果第一个执行结果是结果集，则返回true；反之，返回false。调用getResultSet或getUpdateCount方法可以得到第一个执行结果。
ResultSet getResultSet()
返回前一条查询语句的结果集。如果前一条语句未产生结果集，则返回null值。对于每一条执行过的语句，该方法只能被调用一次。
int getUpdateCount()
long getLargeUpdateCount()
返回受前一条更新语句影响的行数。如果前一条语句未更新数据库，则返回-1。对于每一条执行过的语句，该方法只能被调用一次。
void close()
关闭Statement对象以及它所对应的结果集
boolean isClosed
如果语句被关闭，则返回true
void closeOnCompletion()
使得一旦该语句的所有结果集被关闭，则关闭该语句
boolean getMoreResults()
boolean getMoreResults(int current)
获取该语句的下一个结果集，Current参数是CLOSE_CURRENT_RESULT（默认值），KEEP_CURRENT_RESULT或CLOSE_ALL_RESULTS之一。如果存在下一个结果集，并且它确实是一个结果集，则返回true
boolean execute(String statement, int autogenerared)
int executeUpdate(String statement, int autogenerared)
像前面描述的那样执行给定的SQL语句，如果autogenerared被设置为Statement，RETURN_CENERATED_KEYS，并且该语句是一条INSERT语句，那么第一列中就是自动生成的键。
void addBatch(String command)
添加命令到该语句当前的批量命令中
int[] executeBatch()
long[] executeLargeBatch()
执行当前批量更新中的所有命令。返回一个记录数的数组，其中每一个元素都对应一条语句，如果其值非负，则表示受影响的记录总数；如果其值为SUCCESS_NO_INFO，则表示成功，但是没有记录数可用；如果其值为EXECUTE_FAILED，则表示执行失败

# java.sql.ResultSet
boolean next()
将结果集中的当前行向前移动一行。如果已经到达最后一行的后面，则返回false。注意，初始情况下必须调用该方法才能转到第一行。
Xxx getXxx(int columnNumber)
Xxx getXxx(String columnLabel)
(Xxx指数据类型，例如int、double、String和Date等)
< T > T getObject(int columnIndex, Class< T > type)
< T > T getObject(String columnIndex, Class< T > type)
void updateObject(int columnIndex, Object x, SQLType targetSqlType)
void updateObject(String columnIndex, Object x, SQLType targetSqlType)
用给定的序列号或列标签返回或更新该列的值，并将值转换成指定的类型。列标签是SQL的AS子句中指定的标签，在没有使用AS时，它就是列名。
int findColumn(String columnName)
根据给定的列名，返回该列的序号
void close()
立即关闭当前的结果集
boolean isClosed()
如果该语句被关闭，则返回true
Blob getBlob(int columnIndex)
Blob getBlob(String columnIndex)
Clob getClob(int columnIndex)
Clob getClob(String columnIndex)
获取给定列的BLOB或CLOB

int getType()
返回结果集的类型。返回值为以下常量之一：TYPE_FORWARD_ONLY、TYPE_SCROLL_INSENSITIVE或TYPE_SCROLL_SENSITIVE
int getConcurrency()
返回结果集的并发设置。返回值为以下常量之一：CONCUR_READ_ONLY或CONCUR_UPDATABLE
boolean previous()
将游标移动到前一行。如果游标位于某一行上，则返回true；如果游标位于第一行之前的位置，则返回false。
int getRow()
得到当前行的序号，所有行从1开始编号
boolean absolute(int r)
移动游标到第r行。如果游标位于某一行上，则返回true
boolean relative(int d)
将游标移动d行。如果d为负数，则游标向后移动。如果游标位于某一行上，则返回true
boolean first()
boolean last()
移动游标到第一行或最后一行。如果游标位于某一行上，则返回true
void beforeFirst()
void afterLast()
移动游标到第一行之前或最后一行之后。
boolean isFirst()
boolean isLast()
测试游标是否在第一行或最后一行
boolean isBeforeFirst()
boolean isAfterLast()
测试游标是否在第一行之前或最后一行之后
void moveToInsertRow()
移动游标到插入行。插入行是一个特殊的行，可以在该行上使用updateXxx和insertRow方法来插入新数据
void moveToCUrrentRow()
将游标从插入行移回到调用moveToInsertRow方法之前它所在的那一行
void insertRow()
将插入行上的内容插入到数据库和结果集中
void deleteRow()
从数据库和结果集中删除当前行
void updateXxx(int column, Xxx data)
void updateXxx(String columnName, Xxx data)
更新结果中当前行上的某个字段值
void updateRow()
将当前行的更新信息发送到数据库
void cancelRowUpdates()
撤销对当前行的更新
ResultSetMetaData getMetaData()
返回与当前ResultSet对象中的列相关的元数据

# java.sql.SQLException
SQLException getNextException()
返回链接到该SQL异常的下一个SQL异常，或者在到达链尾时返回null
Iterator< Throwable > iterator()
获取迭代器，可以迭代链接的SQL异常和它们的成因
String getSQLState()
获取SQL动态，即标准化的错误代码
int getErrorCode()
获取提供商相关的错误代码

# java.sql.SQLWarning
SQLWarning getNextSQLWarning()
返回链接到该警告的下一个警告，或者在到达链尾时返回null

### java.sql.Connection
### java.sql.ResultSet
### java.sql.Statement
SQLWarning getNextWarning()
返回链接到该警告的下一个警告，或者在到达链尾时返回null

# java.sql.DataTruncation
boolean getParameter()
如果在参数上进行了数据截断，则返回true；如果在列上进行了数据截断，则返回false
int getIndex()
返回被截断的参数或列的索引
int getDataSize()
返回应该被传输的字节数量，或者该值在位置情况下返回-1
int getTransferSize()
返回实际被传输的字节数量，或者该值在位置情况下返回-1

# java.sql.PreparedStatement
void setXxx(int n, Xxx x)
(Xxx 指 int、double、String、Date之类的数据类型)设置第n个参数值为x
void clearParameters()
清楚预备语句中的所有当前参数
ResultSet executeQuery()
执行预备SQL查询，并返回一个ResultSet对象
int executeUpdate()
执行预备SQL语句INSERT、UPDATE或DELETE，这些语句由PreparedStatement对象表示。该方法返回在执行上述语句过程中所有受影响的记录总数。如果执行的是数据定义语言DDL中的语句，如CREATE TABLE，则该方法返回0

# java.sql.Blob
long length()
获取该BLOB的长度
byte[] getBytes(long startPosition, long length)
获取该BLOB中给定范围的数据
InputStream getBinaryStream()
InputStream getBinaryStream(long startPosition, long length)
返回一个输入流，用于读取该BLOB中全部或给定范围的数据
OutputStream setBinaryStream(long startPosition)
返回一个输出流，用于从给定位置开始写入改BLOB

# java.sql.Clob
long length()
获取该CLOB的长度
String getSubString(long startPosition, long length)
获取该CLOB中给定范围的字符
Reader getCharacterStream()
Reader getCharacterStream(long startPosition, long length)
返回一个读入器，用于读取该CLOB中全部或给定范围的数据
Writer setCharacterStream(long startPosition)
返回一个写出器，用于从给定位置开始写入改CLOB

# java.sql.DatabaseMetaData
boolean supportResultSetType(int type)
如果数据库支持给定类型的结果集，则返回true。type是ResultSet接口的常量之一：TYPE_FORWARD_ONLY、TYPE_SCROLL_INSENSITIVE和TYPE_SCROLL_SENSITIVE
boolean supportResultSetConcurrency(int type, int concurrency)
如果数据库支持给定类型和并发模式的结果集，则返回true，type和concurrency都是ResultSet接口常量之一
ResultSet getTables(String catalog, String schemaPattern, String tableNamePattern, String types[])
返回某个目录（catalog）中所有表的描述，该目录必须匹配给定的模式（schema）、表名字模式以及类型标准。
catalog和schema参数可以为""，用于检索那些没有目录或模式的表。如果不想考虑目录和模式，设为null。
types数组包含了所需的表类型的名称，通常表类型有TABLE、VIEW、SYSTEM TABLE、GLOBAL TEMPORARY、LOCAL TEMPORARY、ALIAS和SYNONYM。如果types为null，则返回所有类型的表。
返回的结果集共五列，均为String类型：
| 行 | 名称 | 解释 |
|--|--|--|
| 1 | TABLE_CAT | 表目录，可以为null |
| 2 | TABLE_SCHEM | 表模式，可以为null |
| 3 | TABLE_NAME | 表名称 |
| 4 | TABLE_TYPE | 表类型 |
| 5 | REMARKS | 注释 |
int getJDBCMajorVersion()
int getJDBCMinorVersion()
返回建立数据库链接的JDBC驱动器的主版本号和次版本号。
int getMaxConnections()
返回可同时连接到数据库的最大并发连接数
int getMaxStatements()
返回单个数据库连接允许同时打开的最大并发语句数。如果对允许打开的语句数目没有限制或者不可知，则返回0
boolean supportsBatchUpdates()
如果驱动程序支持批量更新，则返回true


# javax.sql.RowSet
String getURL()
void setURL(String url)
获取或设置数据库URL
String getUsername()
void setUsername(String Username)
获取或设置数据库用户名
String getPassword()
void setPassword(String Password)
获取或设置数据库密码
String getCommand()
void setCommand(String Command)
获取或设置向行集中填充数据时所需的命令
void execute()
通过执行使用setCommand方法设置的语句集来填充行集。为了使驱动管理器可以获得连接，必须实现设定URL、用户名和密码

# javax.sql.rowset.CachedRowSet
void execute(Connection conn)
通过执行使用setCommand方法设置的语句集来填充行集。该方法使用给定的连接，并复杂关闭它。
void populate(ResultSet result)
将指定的结果集中的数据填充到被缓存的行集中
String getTableName()
void setTableName(String tableName)
获取或设置数据库名称，填充被缓存的行集时所需的数据来自该表
int getPageSize()
void setPageSize(int size)
获取和设置页的尺寸
boolean nextPage()
boolean previousPage()
加载下一页或上一页，如果加载的页不存在，则返回true
void acceptChanges()
void acceptChanges(Connection conn)
重新连接数据库，并写回行集中修改过的数据。如果因为数据库中的数据已经被修改而导致无法写回行集中的数据，该方法可能会抛出SyncProviderException异常

# javax.sql.rowset.RowSetProvider
static RowSetFactory newFactory()
创建一个行集工厂
CachedRowSet createCachedRowSet()
FilteredRowSet createFilteredRowSet()
JdbcRowSet createJdbcRowSet()
JoinRowSet createJoinRowSet()
WebRowSet createWebRowSet()
创建一个指定类型的行集

# java.sql.ResultSetMetaData 
int getColumnCount()
返回当前ResultSet对象中的列数
int getColumnDisplaySize(int column)
返回给定列序号的列的最大宽度
String getColumnLabel(int column)
返回该列所建议的名称
String getColumnName(int column)
返回指定的列序号所对应的列名

# java.sql.Savepoint
int getSavepointId()
获取该匿名保存点的ID号。如果该保存点具有名字，则抛出一个SQLException异常
String getSavepointName()
获取该保存点的名称。如果该对象为匿名保存点，则抛出一个SQLException异常
