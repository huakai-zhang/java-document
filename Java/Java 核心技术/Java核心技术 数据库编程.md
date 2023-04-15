 1996年，Sun发布了第一版的Java数据库连接(JDBC，根据Oracle的声明，JDBC是一个注册了商标的术语，而不是Java Database Connectivity的缩写)API，使编程人员可以通过这个API接口连接到数据库，并使用结构化查询语句（即SQL）完成对数据库的查找与更新（SQL是数据库访问的业界标准）。

# 1.JDBC的设计

JDBC提供一个驱动管理器，以允许第三方驱动程序可以连接到特定的数据库。数据库供应商就可以提供自己的驱动程序，将其插入到驱动管理器中。这将成为一种向驱动管理器注册第三方驱动程序的简单机制。

这种接口组织方式遵循了微软的ODBC模式，ODBC为C原因访问数据库提供了接口。JDBC和ODBC都基于同一思想：根据API编写的程序都可以与驱动管理器进行通信，而驱动管理器则通过驱动程序与实际的数据库进行通信。

### JDBC驱动程序类型

JDBC规范将驱动程序分为4类：

1.将JDBC翻译才ODBC，使用ODBC驱动程序进行数据库通信。

2.由Java程序和部分本地代码组成，用于与数据库的客户端API进行通信。

3.纯Java客户端类库，它使用一种与具体数据库无关的协议将数据库请求发送给服务器构建，然后该构建在将数据库请求翻译成数据库相关的协议。

4.纯Java类库，将JDBC请求直接翻译成数据库相关的协议。



### JDBC的经典用法

![img](https://img-blog.csdnimg.cn/20191230141130770.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

三层应用模型，客户端不直接调用数据库，而是调用服务器上的中间件层，由中间件层完成数据库查询操作。

优点：将可视化表示（客户端）从业务逻辑（中间层）和原始数据（数据库）中分离开来，这样从不同的客户端，来访问相同的数据和相同的业务规则。

客户端和中间层之间通信典型是通过HTTP实现。JDBC管理着中间层和后台数据之间的通信。

# 2.结构化查询语言

SQL是对所有关系型数据库都至关重要的命令行语言，JDBC使我们可以通过SQL与数据库进行通信。可以将JDBC包看做一个用于将SQL语句传递给数据库的应用编程接口（API）。

![img](https://img-blog.csdnimg.cn/20191230142215905.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑



### 3.JDBC配置

### 数据库URL

在连接数据库时，必须使用各种与数据库类型相关的参数，例如主机名、端口号和数据库名。

JDBC使用了一种与普通URL相类似的语法来描述数据源：

```
jdbc:derby://localhost:1527/COREJAVA;create=true
jdbc:mysql://localgost:3306/COREJAVA
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

上述JDBC URL指定了名为COREJAVA的一个Derby数据库（Apache Derby在某些JDK版本中包含了它，包含在JDK中的版本官方称为JavaDB）和一个Mysql数据库。JDBC URL一般语法为：

jdbc:subprotocol:other stuff，其中subprotocol用于连接到数据库的具体驱动程序。

### 驱动程序JAR文件

需要获得所使用的数据库的驱动程序的JAR文件，Derby需要derbyclient.jar，其他数据库需要找到恰当的驱动程序。
 在运行访问数据库程序时，需要将驱动程序JAR文件包括到类路径中（编译时并不需要这个JAR文件）。
 在从命令行启动程序时，需要会用下面命令：
 java -classpath dirverPath:. ProgramName
 在Windows上，可以使用分号将当前路径（即由 . 字符表示的路径）与驱动程序Jar文件分隔开。

### 启动数据库

数据库服务器在连接之前需要先启动，启动的细节取决于所使用的数据库。

### 注册驱动器类

许多JDBC的JAR文件会自动注册驱动器类。包含META-INF/services/java.sql.Driver文件的JAR文件可以自动注册启动器类，解压缩驱动程序JAR文件就可以检查其是否包含该文件。

![img](https://img-blog.csdnimg.cn/20191230154708633.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

这种注册机制使用的是JAR规范中几乎不为人知的特性，参见：[前往查看](https://docs.oracle.com/javase/8/docs/technotes/guides/jar/jar.html#Service Provider)。自动注册对于遵循JDBC4的驱动程序是必须具备的特性。

如果驱动程序JAR文件不支持自动注册，就需要找出数据库供应商使用的JDBC驱动器类的名字：

org.apache.derby.jabc.ClientDriver

通过使用DirverManager，在Java程序中加载驱动器类：

```java
Class.forName("org.apache.derby.jabc.ClientDriver");
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

这条语句将使得驱动器类被加载，由此将执行可以注册驱动器类的静态初始化器。

或者在应用中用下面方式调用来设置系统属性：

```java
System.setProperty("jdbc.drivers", "com.mysql.jdbc.Driver");
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### 连接到数据库

在Java程序中，可以在代码中打开一个数据库连接：

```java
Connection coon =  DriverManager.getConnection("jdbc:mysql://localhost:3306/COREJAVA", "root", "root");
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

驱动管理器遍历所有注册过的驱动程序，以便找到一个能够使用数据库URL中指定的子协议的驱动程序。

getConnection方法返回一个Connection对象，使用Connection对象来执行SQL语句。

database.properties:

```
jdbc.drivers=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/test
jdbc.username=root
jdbc.password=1234
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```java
public class TestDB {
    public static void main(String[] args) throws IOException {
        try {
            runTest();
        } catch (SQLException ex) {
            for (Throwable t : ex) {
                t.printStackTrace();
            }
        }
    }

    public static void runTest() throws SQLException, IOException {
        try (Connection conn = getConnection();
             Statement stat = conn.createStatement()) {
            stat.executeUpdate("CREATE TABLE Greetings (Message CHAR (20))");
            stat.executeUpdate("INSERT INTO Greetings VALUES('Hello, World!')");

            try (ResultSet result = stat.executeQuery("SELECT * FROM Greetings")) {
                if (result.next()) {
                    System.out.println(result.getString(1));
                }
                stat.executeUpdate("DROP TABLE Greetings");
            }
        }
    }

    public static Connection getConnection() throws SQLException, IOException {
        Properties props = new Properties();
        try (InputStream in = Files.newInputStream(Paths.get("database.properties"))) {
            props.load(in);
        }
        String drivers = props.getProperty("jdbc.drivers");
        if (drivers != null) {
            System.setProperty("jdbc.drivers", drivers);
        }
        String url = props.getProperty("jdbc.url");
        String username = props.getProperty("jdbc.username");
        String password = props.getProperty("jdbc.password");

        return DriverManager.getConnection(url, username, password);
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# 4.使用JDBC语句

### 执行SQL语句

在执行SQL语句之前，首先需要创建一个Statement对象，调用DirverManager.getConnection方法所获得的Connection对象。

```java
Statement stat = conn.createStatement();
// 将要执行的SQL语句放入字符串
String command = "UPDATE Books"
    + "SET Price = Price - 5.00"
    + "WHRER Title NOT LIKE '%Introduction%'";
// 调用Statement的executeUpdate方法
stat.executeUpdate(command);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

executeUpdate可以执行INSERT、UPDATE、DELETE、CREATE TABLE和DROP TABLE语句。但执行SELECT查询时必须使用executeQuery方法。

另外还有一个execute语句可以执行任意的SQL语句，此方法通常只用于由用户提供的交互式查询。

executeQuery会返回一个ResultSet类型的对象，可以通过它来每次一行地迭代遍历所有查询结果。ResultSet的迭代协议和Iterator接口稍有不同。ResultSet接口迭代器初始化被定义在第一行之前的位置，必须调用next方法将它移动到第一行。另外它没有hasNext方法，需要不断地调用next方法，直至该方法返回false。

结果集中行是任意排列的，除非使用ORDER BY子句指定行的顺序。查看每一行时，有许多访问器（accessor）方法可以用于获取这些信息：

```java
String isbn = rs.getString(1);
double price = rs.getDouble("Price");
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

不同的数据类型有不同的访问器。每个访问器都有两个形式，一种接受数字型参数，另一种接受字符串参数。当使用数字型参数时，我们指的是该数字对应的列。与数组的索引不同，数据库的列序号是从1开始计算的。当使用字符串参数时，指的是结果集中以该字符串为列名的列。

使用数字型参数效率更高一下，但是使用字符串参数可以使代码更易阅读和维护。

当get方法的类型和列的数据类型不一致时，每个get方法都会进行合理的类型转换。

### 管理连接、语句和结果集

每个Connection对象都可以创建一个或多个Statement对象。同一个Statement对象可以用于多个不相关的命令和查询。

一个Statement对象最多只能有一个打开的结果集。如果需要多个查询操作，并同时分析查询结果，必须创建多个Statement对象。

至少有一种常用数据库的JDBC驱动程序只允许同时存在一个活动的Statement对象。使用DatabaseMetaData接口中的getMaxStatements方法可以获取JDBC程序支持的同时活动的语句对象的总数。

实际上不需要同时处理多个结果集，结果集如果互相关联，可以使用组合查询，分析一个结果。对数据库进行组合查询比使用Java程序遍历多个结果集要高效得多。



使用完ResultSet、Statement或Connection对象后，应立即调用close方法。这些对象都使用了大量的数据结构，会占用数据库存服务器上的有限资源。Statement对象如果有一个结果集，调用close方法将自动关闭该结果集。Connection的close方法将关闭该连接上的所有语句。

JavaSE 7的Statement调用closeOnCompletion范发给，在其所有结果集都被关闭后，该语句会立即被自动关闭。

如果所用连接是短时的，无需考虑关闭语句和结果集。只需将close语句放在带有资源的try语句中，以便确保最终连接对象不可能继续保持打开状态。

应该使用带资源的try语句块来关闭连接，并使用一个单独的try/catch块处理异常。分离try程序块可以提高代码可读性和可维护性。

### 分析SQL异常

每个SQLException都有一个由多个SQLException对象构成的链，这些对象可以通过getNextException方法获取。这个异常链是每个异常都具有由Throwable对象构成的成因链之外的异常链。

Java SE 6改进了SQLException对象，让其实现了Iterable< Throwable >接口，其iterator()方法可以产生一个Iterator< Throwable >：

```java
// 产生符合X/Open或SQL:2013标准的字符串（调用DatabaseMetaData接口的getSQLStateType方法可以查看驱动程序所使用的标准）
e.getSQLState();
// 错误代码是与具体的供应商相关
e.getErrorCode();
for (Throwable t : e) {
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

SQL异常按照层次结构树的方式组织在一起，可以按照提供方无关的方式来捕获具体的错误类型：

![img](https://img-blog.csdnimg.cn/20191231095830998.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

数据库驱动程序，可以将非致命问题作为警告报告，可以从连接、语句和结果集中获取。SQLWarning类是SQLException的子类（尽管SQLWarning不会被当做异常抛出）。可以调用getSQLState和getErrorCode来获取更多警告信息。

与SQL异常类似，警告也是串成链：

```java
SQLWarning w = stat.getWarnings();
while (w != null) {
    w = w.getNextWarning();
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

当数据从数据库中读出并意外被截断时，如果截断发生在更新语句中，SQLWarning的DataTruncation将会被当作异常抛出。

### 组装数据库

用一系列的SQL指令来创建数据表并向其中插入数据。

```java
public class ExecSQL {
    public static void main(String[] args) throws IOException {
        try (Scanner in = args.length == 0 ? new Scanner(System.in) : new Scanner(Paths.get(args[0]), "UTF-8")) {
            try (Connection conn = getConnection();
                    Statement stat= conn.createStatement()) {
                while (true) {
                    if (args.length == 0) {
                        System.out.println("Enter command or EXIT to exit:");
                    }
                    if (!in.hasNextLine()) {
                        return;
                    }

                    String line = in.nextLine().trim();
                    if (line.equalsIgnoreCase("EXIT")) {
                        return;
                    }
                    if (line.endsWith(";")) {
                        line = line.substring(0, line.length() - 1);
                    }

                    try {
                        boolean isResult = stat.execute(line);
                        if (isResult) {
                            try (ResultSet rs = stat.getResultSet()) {
                                showResultSet(rs);
                            }
                        } else {
                            int updateCount = stat.getUpdateCount();
                            System.out.println(updateCount + " rows updated");
                        }
                    } catch (SQLException ex) {
                        for (Throwable t : ex) {
                            t.printStackTrace();
                        }
                    }
                }
            } catch (SQLException ex) {
                for (Throwable t : ex) {
                    t.printStackTrace();
                }
            }
        }
    }

    public static Connection getConnection() throws SQLException, IOException {
        Properties props = new Properties();
        try (InputStream in = Files.newInputStream(Paths.get("database.properties"))) {
            props.load(in);
        }

        String drivers = props.getProperty("jdbc.drivers");
        if (drivers != null) {
            System.setProperty("jdbc.drivers", drivers);
        }
        String url = props.getProperty("jdbc.url");
        String username = props.getProperty("jdbc.username");
        String password = props.getProperty("jdbc.password");

        return DriverManager.getConnection(url, username, password);
    }

    public static void showResultSet(ResultSet result) throws SQLException {
        ResultSetMetaData metaData = result.getMetaData();
        int columnCount = metaData.getColumnCount();

        for (int i = 1; i <= columnCount; i++) {
            if (i > 1) {
                System.out.print(", ");
            }
            System.out.print(metaData.getColumnLabel(i));
        }
        System.out.println();

        while (result.next()) {
            for (int i = 1; i <= columnCount; i++) {
                if (i > 1) {
                    System.out.print(", ");
                }
                System.out.print(result.getString(i));
            }
            System.out.println();
        }
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# 5.执行查询操作

### 预备语句

预备语句（prepared statement）,没必要在每次开始一个查询时都建立新的查询语句，而是准备一个带有宿主变量的查询语句，每次查询时只需为变量填入不同的字符串就可以反复多次使用该语句。

宿主变量都用？表示：

```java
String query = "SELECT * FROM publishers WHERE name = ?";
PreparedStatement statement = conn.prepareStatement(query);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```java
stat.setString(1, name)
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

在执行预备语句之前，使用set方法将变量绑定到实际值上：
 

第一个参数指的是需要设置的宿主变量的位置，位置1表示第一个“？”。第二个参数表示赋予宿主变量的值。

在从一个查询到另一个查询过程中，只需要使用setXxx方法重新绑定改变的变量即可。

一旦绑定了具体的值，就可以执行了：

```java
ResultSet rs = stat.executeQuery();
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

UPDATE时调用executeUpdate方法，因为UPDATE不返回结果集，而是被修改的行数。



### 读写LOB

除了数字、字符串和日期之外，许多数据库还可以存储大对象，例如图片或其他数据。在SQL中，二进制大对象成为BLOB，字符型大对象称为CLOB。

读取LOB，在ResultSet上调用getBlob或getClob方法，就可以获得Blob或Clob对象。调用Blob的getBytes或getBinaryStream方法，可以从中读取二进制数据。

```java
Image coverImage = ImageIO.read(blob.getBinartStream());
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

类似的，Clob对象，可以通过getSubString或getCharacterStream方法来获取其中的字符数据。

要将LOB置于数据库中，需要在Connection对象上调用createBlob或createClob，然后获取一个用于该LOB的输出流或写出流，写出数据并将该对象存储到数据库中：

```java
Blob coverBlob = conn.createBlob();
int offset = 0;
OutputStream out = coverBlob.setBinaryStream(offset);
ImageIO.write(coverImage, "PNG", out);
PreparedStatement statement = conn.prepareStatement("INSERT INTO Cover VALUE (?, ?)");
statement.set(1, isbn);
statement.set(2, coverBlob);
statement.executeUpdate();
while (true) {
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### SQL转义

1.日期和时间字面常量 随数据库的不同变化很大。要嵌入日期或时间字面常量，需要按照ISO 8601格式指定它的值，之后驱动程序会将其转义为本地格式。应该使用d、t、ts来表示DATE、TIME和TIMESTAMP值：

{d '2019-12-31'}

{t '23:59:59'}

{ts '2019-12-31 23:59:59'}

2.标量函数，指仅返回单个值的函数。在数据库包含大量函数，但是不同数据库中函数名存在差异。JDBC规范提供了标准的名字，并将其转义为数据库相关名字：

{fn left(?, 20)}

{fn user()}

3.存储过程，是在数据库中执行的用数据库相关的语言编写的过程。要调用存储过程，需要使用call转义命令，在存储过程没有任何参数时，可以不用加上括号，用=来捕获存储过程的返回值：

{call PROC1(?, ?)}

{call PROC2}

{call ? = PROC3(?)}

4.外连接，并不要求每个表的所有行都要根据连接条件进行匹配：

SELECT * FROM {oj books LEFT UOTER JOIN publishers ON books.publisher_id = publishers.publisher_id}

此查询将包含publisher_id在publishers表中没有任何匹配的数据。

如果使用RIGHT OUTER JOIN，就包括没任何匹配的图书。而是用FULL OUTER JOIN可以同时返回两类没有匹配的信息。

5._和%字符在LIKE子句中具有特殊含义，用来匹配一个字符或一个字符序列。

... WHERE ? LIKE %!_% {escape '!'}

这里将！定义为转义字符，而！_组合表示字面常量下划线。

### 多结果集

在执行存储过程中，或者在使用允许在单个查询中提交多个SELECT语句的数据库时，一个查询有可能会返回多个结果集：

1.使用execute方法来执行SQL语句

2.获取第一个结果集或更新计数

3.重复调用getMoreResults方法以移动到下一个结果集

4.当不存在更多的结果集或更新计数时，完成操作

如果由多结果集构成的链中的下一项是结果集，execute和getMoreResults方法将返回true，而如果下一项不是更新计数，getUpdateCount将返回-1

```java
boolean isResult = stat.execute(command);
boolean done = false;
while(!done) {
    if(isResult) {
        ResultSet result = stat.getResultSet();
        ...
    } else {
        int updateCount = stat.getUpdateCount();
        if(updateCount >= 0) {
            ...
        } else {
            done = true;
        }
    }
    if (!done) {
        isResult = stat.getMoreResults();
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### 获取自动生成的键

大多数数据库都支持某种在数据库中对行自动编号的机制（经常做主键）。当向数据库表插入新行，且其键自动生成时，可以获取这个键：

```java
stat.executeUpdate(insertStatement, Statement.RETURN_CENERATED_KEYS);
ResultSet rs = stat.getGeneratedKeys();
if (rs.next()) {
    int key = rs.getInt(1);
    ...
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# 6.可滚动和可更新的结果集

### 可滚动的结果集

默认情况下，结果集是不可滚动和不可更新的。为了从查询中获取可滚动的结果集，必须使用一个不同的Statement对象：

```java
Statement stat = conn.createStatment(type, concurrency);
// 预备语句
PreparedStatment stat = conn.prepareStatement(command, type, concurrency);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

![img](https://img-blog.csdnimg.cn/20200101145222784.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

如果只想滚动遍历结果集，而不想编辑数据：

```java
Statement stat = conn.createStatment(ResultSet.TYPE_SCROLL_INSENSITIVE, ResultSet.CONCUR_READ_ONLY);
// 调用以下方法获得所有结果集将都是可滚动的
ResultSet rs = stat.executeQuery(query);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

可滚动的结果集有一个游标，用以指示当前位置。并非所有数据库驱动程序都支持可滚动和可更新的结果集。

在结果集上滚动非常简单，使用rs.previous()向后滚动。如果游标位于一个实际的行上，那么返回true；如果游标位于第一行之前，那么返回false。

可以使用rs.relative(n)将游标向后或向前移动多行。正负号。如果游标将移动到范围之外，该方法将返回false，且不移动游标。如果游标位于一个实际的行上，那么方法将返回true。

rs.absolute(n)将游标设置到指定行号上。

rs.getRow()返回当前行的行号。游标不在任何行上返回0，要么位于第一行之前，要么位于最后一行之后。

first、last、beforeFirst和afterLast用于将游标移动到第一行、最后一行、第一行之前或最后一行之后。

isFirst、isLast、isBeforeFirst和isAfterLast用于测试游标是否位于这些位置上。

可滚动的结果集，将查询数据放入缓存中的复杂工作是由数据库驱动程序在后台完成的。

### 可更新结果集

可更新结果集的数据变更会自动反映到数据库中。可更新的结果集并非必须是可滚动的（只是通常希望是可滚动的）。

```java
// 获得可更新结果集
Statment stat = conn.createStatment(ResultSet.TYPE_SCROLL_INSENSITIVE, ResultSet.CONCUR_UPDATABLE);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

并非所有的查询都会返回可更新的结果集，如果查询涉及多个表的链接操作，那么它所产生的结果集将是不可更新的。可以调用getConcurrency方法来确定结果集是否可更新。

```java
String query = "SELECT * FROM books";
ResultSet rs = stat.executeQuery(query);
while (rs.next()) {
    double increase = ...;
    double price = rs.getDouble("price");
    rs.updateDouble("price", price + increase);
    rs.updateRow();
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

所有对应于SQL类型的数据类型都配有updateXxx。updateXxx必须制定列的名称或序号。这里的序号指的是在结果集中的序号。

updateXxx方法改变的是结果集中的行值，而不是数据库的值，当更新完行中的字段值后，必须调用updateRow方法，这个方法将当前行中的所有更新信息发送给数据库（否则更新将被丢弃）。调用cancelRowUpdate方法来取消对前行的更新。

如果需要插入一条新的记录，首先使用moveToInsertRow方法将游标移动到特定位置，称之为插入行（insert row）。调用updateXxx方法在插入行的位置建立一个新的行。然后调用insertRow将新建的行发送给数据库。之后调用moveToCurrentRow方法将游标移回到调用moveToInsertRow方法之前的位置：

```java
rs.moveToInsertRow();
rs.updateString("title", title);
rs.updateString("publisher_id", publid);
rs.updateString("price", price);
rs.insertRow();
rs.moveToCurrentRow();
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

无法控制在结果集或数据库中添加新数据的位置。对于插入行中没有指定值的列，将被设置为SQL的NULL。如果这个列有NOT NULL约束，那就会抛出异常，而这一行也无法插入。

deleteRow()将立即将游标所在的行从结果集和数据库中删除。



# 7.行集

RowSet接口扩展自ResultSet接口，却无需始终保持与数据库的连接（数据库连接属于稀有资源）。行集还适用于将查询结果移动到复杂应用的其它层，或者是诸如手机之类的其他设备。

### 构建行集

以下为javax.sql.rowset包提供的接口，它们都扩展了RowSet接口：

1.CachedRowSet对象代表了一个被缓存的行集，该行集可以保存为XML文件。该文件可以移动到Web应用的其它层中，只要在该层使用另一个WebRowSet对象重新打开该文件即可

2.FilteredRowSet和JoinRowSet接口支持对行集的轻量级操作，它们等同于SQL的SELECT和JOIN操作。这两个接口的操作对象是存储在行集中的数据，因此运行时无需建立数据库连接。

3.JdbcRowSet是ResultSet接口的一个瘦包装器，它在RowSet接口中添加了有用的方法。

在Java 7中，有一种获取行集的标准方式：

```java
RowSetFactory factory = RowSetProvider.newFacotry();
CachedRowSet crs = factory.createCachedRowSet();
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### 被缓存的行集

一个被缓存的行集中包含一个结果集中所有的数据。CachedRowSet是ResultSet接口的子接口（可以像使用结果集一样使用被缓存的行集）。

被缓存的行集有一个优点：断开数据库连接后仍然可以使用行集。

甚至可以修改被缓存的行集数据，但这些修改不会立即反馈到数据库，必须发起一个显式的请求，以便让数据库真正修改。此时CachedRowSet类会重新连接数据库，并通过执行SQL语句向数据库写入所有修改。

```java
// 使用结果集填充CachedRowSet
ResultSet result = ...;
RowSetFactory factory = RowSetProvider.newFactory();
CachedRowSet crs = factory.createCachedRowSet();
crs.populate(result);
conn.close();

// 让CachedRowSet对象自动建立一个数据库连接
crs.setURL("jdbc:mysql://localhost:3306/test");
crs.setUsername("root");
crs.setPassword("root");
// 设置查询语句和所有参数
crs.setCommand("SELECT * FROM books WHERE publisher_id = ?");
crs.setString(1, publisherId);
// 查询结果填充行集
crs.execute();
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

execute方法会建立数据库连接，执行查询，填充行集，最后断开连接。

如果查询结果非常大，可以指定每一页尺寸：

```java
CachedRowSet crs = ...;
crs.setCommand(command);
// 只获得20行数据
crs.setPageSize(20);
...
crs.execute();

// 获取下一批数据
crs.nextPage();
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

行集和结果集一样查看和修改数据，如果修改了行集数据，必须调用方法将修改写回到数据库中：

```java
crs.acceptChanges(conn);
// 或
// 只有在行集中设置了连接数据库所需信息，下面方法调用才会有效
crs.acceptChanges();
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

同样的，行集包含的是复杂的查询结果，将无法对行集数据的修改写回到数据库中。如果数据都来自同一种数据库表，就可以安全地写回数据。

如果使用结果集填充行集，那么行集无法获知需要更新数据的数据库表名，此时必须调用setTable来设置表名。

在填充行集之后，数据库发生了变化，容易造成数据不一致性。参考实现会首先检查行集中的原始值是否与数据库的当前值一直，如果一致将会覆盖数据库中的当前值。否则抛出SyncProviderException异常，且不向数据库写回任何值。

# 8.元数据

JDBC还可以提供关于数据库及其表结构的详细信息。

在SQL中，描述数据库或其组成部分的数据称为元数据。可以获得三类元数据：关于数据库的元数据，关于结果集的元数据，预备语句参数的元数据。

```java
DatabaseMetaData meta = conn.getMetaData();
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

返回一个包含所有数据库表信息的结果集。该结果集的每一行都包含了数据库一张表的详细信息，其中第三列是表的名称。

数据库元数据还有第二个重要应用。DatabaseMetaData接口中有上百个方法可以用于查询数据库的相关信息，包括一些使用奇特名字进行调用的方法：

```java
meta.supportsCatalogsInPrivilegeDefinitions();
// 和
meta.nullPlusNonNullIsNull();
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

第二个元数据接口ResultSetMetaData用于提供结果集的相关信息。每当通过查询得到一个结果时，都可以获取该结果集的列数以及每一列的名称、类型和字段宽度：

```java
ResultSet rs = stat.executeQuery("SELECT * FROM " + tableName);
ResultSetMetaData meta = rs.getMetaData();
for (int i = 1; i <= meta.getColumnCount(); i++) {
    String columnName = meta.getColumnLabel(i);
    int columnWidth = meta.getColumnDisplaySize(i);
    ...
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```java
package chapter13;

import javax.sql.RowSet;
import javax.sql.rowset.CachedRowSet;
import javax.sql.rowset.RowSetFactory;
import javax.sql.rowset.RowSetProvider;
import javax.swing.*;
import java.awt.*;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;
import java.io.IOException;
import java.io.InputStream;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.sql.*;
import java.util.ArrayList;
import java.util.Properties;

public class ViewDB {
    public static void main(String[] args) {
        EventQueue.invokeLater(() -> {
            JFrame frame = new ViewDBFrame();
            frame.setTitle("ViewDB");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setVisible(true);
        });
    }
}

class ViewDBFrame extends JFrame {
    private JButton previousButton;
    private JButton nextButton;
    private JButton deleteButton;
    private JButton saveButton;
    private DataPanel dataPanel;
    private Component scrollPane;
    private JComboBox<String> tableNames;
    private Properties props;
    private CachedRowSet crs;
    private Connection conn;

    public ViewDBFrame() {
        tableNames = new JComboBox<String>();

        try {
            readDatabaseProperties();
            conn = getConnection();
            DatabaseMetaData meta = conn.getMetaData();
            try (ResultSet mrs = meta.getTables(null, null, null, new String[] {"TABLE"})) {
                while (mrs.next()) {
                    tableNames.addItem(mrs.getString(3));
                }
            }

        } catch (SQLException ex) {
            for (Throwable r : ex) {
                r.printStackTrace();
            }
        } catch (IOException io) {
            io.printStackTrace();
        }

        tableNames.addActionListener(
                e -> showTable((String) tableNames.getSelectedItem(), conn));
        add(tableNames, BorderLayout.NORTH);
        addWindowListener(new WindowAdapter() {
            @Override
            public void windowClosing(WindowEvent event) {
                try {
                    if (conn != null) {
                        conn.close();
                    }
                } catch (SQLException ex) {
                    for (Throwable r : ex) {
                        r.printStackTrace();
                    }
                }
            }
        });

        JPanel buttonPanel = new JPanel();
        add(buttonPanel, BorderLayout.SOUTH);

        previousButton = new JButton("Previous");
        previousButton.addActionListener(event -> showPreviousRow());
        buttonPanel.add(previousButton);

        nextButton = new JButton("Next");
        nextButton.addActionListener(event -> showNextRow());
        buttonPanel.add(nextButton);

        deleteButton = new JButton("Delete");
        deleteButton.addActionListener(event -> deleteRow());
        buttonPanel.add(deleteButton);

        saveButton = new JButton("Save");
        saveButton.addActionListener(event -> saveChanges());
        buttonPanel.add(saveButton);
        if (tableNames.getItemCount() > 0) {
            showTable(tableNames.getItemAt(0), conn);
        }
    }

    /**
     * 准备显示新表的文本字段，并显示第一行
     * @param tableName
     * @param conn
     */
    public void showTable(String tableName, Connection conn) {
        try(Statement stat = conn.createStatement();
            ResultSet result = stat.executeQuery("SELECT * FROM " + tableName)) {
            RowSetFactory factory = RowSetProvider.newFactory();
            crs = factory.createCachedRowSet();
            crs.setTableName(tableName);
            crs.populate(result);

            if (scrollPane != null) {
                remove(scrollPane);
            }
            dataPanel = new DataPanel(crs);
            scrollPane = new JScrollPane(dataPanel);
            add(scrollPane, BorderLayout.CENTER);
            pack();
            showNextRow();
        } catch (SQLException ex) {
            for (Throwable r : ex) {
                r.printStackTrace();
            }
        }
    }

    public void showPreviousRow() {
        try {
            if (crs == null || crs.isFirst()) {
                return;
            }
            crs.previous();
            dataPanel.showRow(crs);
        } catch (SQLException ex) {
            for (Throwable r : ex) {
                r.printStackTrace();
            }
        }
    }

    public void showNextRow() {
        try {
            if (crs == null || crs.isLast()) {
                return;
            }
            crs.next();
            dataPanel.showRow(crs);
        } catch (SQLException ex) {
            for (Throwable r : ex) {
                r.printStackTrace();
            }
        }
    }

    public void deleteRow() {
        if (crs == null) {
            return;
        }
        new SwingWorker<Void, Void>() {
            @Override
            public Void doInBackground() throws Exception {
                crs.deleteRow();
                crs.acceptChanges(conn);
                if (crs.isAfterLast()) {
                    if (!crs.last()) {
                        crs = null;
                    }
                }
                return null;
            }

            @Override
            public void done() {
                dataPanel.showRow(crs);
            }
        }.execute();
    }

    public void saveChanges() {
        if (crs == null) {
            return;
        }

        new SwingWorker<Void, Void>() {
            @Override
            public Void doInBackground() throws Exception {
                dataPanel.setRow(crs);
                crs.acceptChanges(conn);
                return null;
            }
        }.execute();
    }

    private void readDatabaseProperties() throws IOException {
        props = new Properties();
        try (InputStream in = Files.newInputStream(Paths.get("database.properties"))) {
            props.load(in);
        }
        String drivers = props.getProperty("jdbc.drivers");
        if (drivers != null) {
            System.getProperty("jdbc.drivers", drivers);
        }
    }

    private Connection getConnection() throws SQLException {
        String url = props.getProperty("jdbc.url");
        String username = props.getProperty("jdbc.username");
        String password = props.getProperty("jdbc.password");

        return DriverManager.getConnection(url, username, password);
    }
}

/**
 * 此面板显示结果集的内容
 */
class DataPanel extends JPanel {
    private java.util.List<JTextField> fields;

    /**
     * 构造数据面板
     * @param rs 此面板显示其内容的结果集
     * @throws SQLException
     */
    public DataPanel(RowSet rs) throws SQLException {
        fields = new ArrayList<>();
        setLayout(new GridBagLayout());
        GridBagConstraints gbc = new GridBagConstraints();
        gbc.gridwidth = 1;
        gbc.gridheight = 1;

        ResultSetMetaData rsmd = rs.getMetaData();
        for (int i = 1; i <= rsmd.getColumnCount(); i++) {
            gbc.gridy = i - 1;

            String columnName = rsmd.getColumnLabel(i);
            gbc.gridx = 0;
            gbc.anchor = GridBagConstraints.EAST;
            add(new JLabel(columnName), gbc);

            int columnWidth = rsmd.getColumnDisplaySize(i);
            JTextField tb = new JTextField(columnWidth);
            if (!rsmd.getColumnClassName(i).equals("java.lang.String")) {
                tb.setEditable(false);
            }

            fields.add(tb);

            gbc.gridx = 1;
            gbc.anchor = GridBagConstraints.WEST;
            add(tb, gbc);
        }
    }

    /**
     * 通过用列值填充所有文本字段来显示数据库行
     */
    public void showRow(ResultSet rs) {
        try {
            if (rs == null) {
                return;
            }
            for (int i = 1; i <= fields.size(); i++) {
                String field = rs == null ? "" : rs.getString(i);
                JTextField tb = fields.get(i - 1);
                tb.setText(field);
            }
        } catch (SQLException ex) {
            for (Throwable r : ex) {
                r.printStackTrace();
            }
        }
    }

    /**
     * 将更改的数据更新为行集的当前行
     */
    public void setRow(RowSet rs) throws SQLException {
        for (int i = 1; i <= fields.size(); i++) {
            String field = rs.getString(i);
            JTextField tb = fields.get(i - 1);
            if (!field.equals(tb.getText())) {
                rs.updateString(i, tb.getText());
            }
        }
        rs.updateRow();
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

![img](https://img-blog.csdnimg.cn/20200101181556485.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

# 9.事务

可以将一组语句构成一个事务（transaction），当所有语句都顺利执行之后，事务可以被提交。否则如果其中某个语句遇到错误，那么事务将被回滚。

主要是为了确保数据库完整性（database integrity）。

### 用JDBC对事务编程

默认情况下，数据库连接处于自动提交模式（autocommit mode）。每个SQL语句一旦被执行便被提交给数据库。一旦命令被提交，就无法对它进行回滚。在使用事务时，需要关闭这个默认值：

```java
conn.setAutoCommit(false);
// 使用通常方法创建一个语句对象
Statement stat = conn.createStatement();
// 然后多次调用executeUpdate方法
stat.executeUpdate(command1);
stat.executeUpdate(command2);
stat.executeUpdate(command3);
...
// 如果没有错误
conn.commit();
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

如果出现错误调用conn.rollback()，程序将自动撤销自上次提交以来的所有语句。当事务被SQLException异常中断时，典型的办法就是发起回滚操作。

### 保存点

在使用某些驱动程序时，使用保存点（save point）可以更细粒度地控制回滚操作。创建一个保存点意味着稍后只需回到这个点，而非事物的开头。

```java
Statement stat = conn.createStatement();
stat.executeUpdate(command1);
Savepoint svpt = conn.setSavepoint();
stat.executeUpdate(command2);
if (...) {
    conn.rollback(svpt);
}
...
conn.commit();
// 当不再需要保存点时，必须释放它
conn.releaseSavepoint(svpt);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### 批量更新

在使用批量更新（batch update）时，一个语句序列作为一批操作将同时被收集和提交。

处于同一批的语句可以是INSERT、UPDATE、DELETE，也可以是数据库定义语句CREATE TABLE、DROP TABLE。在批量处理中添加SELECT语句会抛出异常。

```java
Statement stat = conn.createStatement();
// 应该调用addBatch方法，而非executeUpdate方法
String command = "CREATE TABLE ...";
stat.addBatch(command);
while (...) {
    command = "INSERT INTO ... VALUES ("+ ... +")";
    stat.addBatch(command);
}
// 最后提交整个批量更新语句
int[] counts = stat.executeBatch();
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

调用executeBatch方法将为所有已提交的语句返回一个记录数的数组。

为了在批量模式下正确的处理错误，必须将批量操作视为单个事务。如果批量更新在过程中失败，那么必须将它回滚到批量操作开始之前的状态。

```java
boolean autoCommit = conn.getAutoCommit();
conn.setAutoCommit(false);
Statement stat = conn.createStatement();
...
stat.executeBatch();
conn.commit();
conn.setAutoCommit(autoCommit);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# 10.高级SQL类型

![img](https://img-blog.csdnimg.cn/20200101212839627.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑
