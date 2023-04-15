##  **1.处理异常**

如果由于出现错误而使得某些操作没有完成，程序应该：

1.返回到一种安全状态，并能够让用户执行一些其他命令

2.允许用户保存所有操作的结果，并以妥善方式终止程序。



可能出现的错误和问题：

1.用户输入错误

2.设备错误

3.物理限制

4.代码错误

对于方法中的一个错误，传统的做法是返回一个特殊的错误码。

异常处理机制在代码无法执行后，会搜索能够处理这种异常状况的异常处理器（exception handler）。



### 异常分类

Java中，异常对象都是派生于Throwable类的一个实例。如果Java中内置的异常类不能够满足需求，可以创建自己的异常类。

![img](https://img-blog.csdn.net/20170210113218841?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

所有异常都是由Throwable继承而来，但在下层立即分为：Error和Exception。

Error类层次结构描述了Java运行时系统的内部错误和资源耗尽错误。应用程序不应该抛出这种类型的对象。如果出现了这种内部错误，除了通告给用户，并尽力使程序安全地终止之外，也无能为力。

Exception层次，分解为两个分支：一个派生于RuntimeException，另一个是其他异常。划分的规则是：由程序错误导致的异常属于RuntimeException：

1.错误类型转换

2.数组访问越界

3.访问null指针

而程序本身没问题，但由于想I/O错误这里问题导致的异常属于其他异常：

1.试图在文件尾部后面读取数据

2.试图打开一个不存在的文件

3.试图根据给定的字符串查找Class对象，而这个类不存在

**如果出现RuntimeException异常，那么就一定是你的问题。**



Java将派生于Error类和RuntimeException类的所有异常成为**非受检查（unchecked）异常**，所有其他异常被称为**受查（checked）异常**。



### 声明受查异常

遇到下面4中情况时应该抛出异常：

1.调用一个抛出受查异常的方法

2.程序运行过程中出现错误，并且利用throw语句抛出一个受查异常

3.程序出现错误，抛出一个非受查异常

4.java虚拟机和运行时库出现错误

如果可能抛出多个受查异常类型，就必须在方法首部列出所有异常类。每个类用逗号隔开：

```java
public Image loadImage(String s) throws FileNotFoundException, EOFException{
    ...
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

不需要声明Error继承的Java内部错误，任何程序都有抛出这些异常的潜能，而我们对其没有任何控制能力。同样也不应该声明RuntimeException继承的那些非受查异常。

如果子类中覆盖了超类的一个方法，子类方法声明的受查异常不能比超类中声明的异常更通用（子类可以抛出更特定的异常，或者根本不抛出异常）。



### 如何抛出异常

```java
String readDate(Scanner in) throws EOFException {
    ...
    while () {
        if (!in.hasNext()) {
            if (n < len) {
                throw new EOFException();
            }
        }
    }
    return s;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

EOFException类还有一个含有一个字符串型参数的构造器。 这个构造器可以更加细致的描述异常出现的情况。

```java
String gripe = "Content-length: " + len + ", Received: " + n;
throw new EOFException(gripe);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### 创建异常类

程序中可能会遇到任何标准异常都没有能够充分地描述清楚的问题，那需要做的只是定义一个派生于Exception的类，或者派生于Exception子类的类。

习惯上，定义的类应该包含两个构造器，一个是默认构造器，另一个是带有详细说明的构造器：
 

```java
public class FileFormatException extends IOException {
    public FileFormatException() {
    }
    public FileFormatException(String message) {
        super(message);
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 2.捕获异常

### 捕获异常

try(...) catch(ExceptionType e) {...}

如果在try语句块中的任何代码抛出一个catch子句中说明的异常类：

1.程序将跳过try语句块的其他代码

2.程序将执行catch子句中的处理器代码

如果try语句中没有任何异常抛出，那么程序将跳过cath子句。

如果try语句抛出了catch中没有声明的异常类型，那么方法就会立刻退出。

```java
public void read(String filename) {
    try {
        InputStream in = new FileInputStream(filename);
        int b;
        while ((b = in.read()) != -1) {
            // 输入
        }
    } catch (IOException exception) {
        exception.printStackTrace();
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

read方法有可能抛出一个IOException异常，这种情况下，将跳出整个while循环，进入catch子句，并生成一个栈轨迹。

通常，最好的选择是什么也不做，而是将异常传递给调用者：

```java
public void read(String filename) throws IOException {
    InputStream in = new FileInputStream(filename);
    int b;
    while ((b = in.read()) != -1) {
        // 输入
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

执行throw说明符，如果调用了一个抛出受查异常的方法，就必须对它进行处理，或者继续传递。

不允许在子类的throws说明符中出现超过超类方法所列出的异常类范围。



### 捕获多个异常

try{

}catch(FileNotFoundException e){

}carch(UnknownHostException e){

}catch(IOException e){}

异常对象可能包含与异常本身有关的信息，想获得对象的更多信息，可以使用e.getMessage()，或者使用e.getClass().getName()得到异常对象的事实类型。

在Java SE 7中，同一个catch子句中可以捕获多个异常类型：

```java
try {
  
} catch (FileNotFoundException | UnknownHostException e) {
    
} catch (IOException exception) {
    
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**只有当不活的异常类型彼此之间不存在子类关系时才需要这个特性。**



### 再次抛出异常与异常链

在catch子句中可以抛出一个异常，这样做的目的是改变异常的类型。

```java
try {

}catch (SQLException e) {
    throw new ServletException("database error:" + e.getMessage());
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

ServletException用带有异常信息文本的构造器来构造。

可以有一种更好的处理方法，并且将原始异常设置为新异常的原因：

```java
try {
    
} catch (SQLException e) {
    Throwable se = new ServletException("databse error");
    se.initCause(e);
    throw se;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

当捕获异常时，就可以使用下面语句重新得到原始异常：

Throwable e = se.getCasue();

这样可以让用户抛出子系统中的高级异常，而不会丢失原始异常的细节。

有时可能只想记录一个异常，再将它重新抛出，而不做任何改变：

```java
try {

} catch (Exception e) {
    logger.log(level, message, e);
    throw e;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

###   finally子句

回收资源问题：

一种解决方案是捕获并重新抛出所有的异常。这种方案比较乏味，这是因为需要在两个地方清除所分配的资源。一个在正常的代码中；另一个在异常代码中。

Java有一种更好的解决方案，就是finally子句。

不管是否有异常被捕获，finally子句中的代码都被执行。

```java
InputStream in = new FileInputStream(...);
try {
    // 1
    会出现异常的地方
    // 2
} catch (IOException e) {
    // 3
    展示错误信息
    // 4
} finally {
    // 5
    in.close();
}
// 6
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

3种情况会执行finally子句：

1.代码没有抛出异常。1,2,5,6

2.抛出一个在catch子句中捕获的异常。如果catch子句没有抛出异常，执行1，3，4，5，6。如果抛出异常执行1，3，5。

3.代码catch子句抛出了一个异常，但这个异常不是由catch子句捕获的。这种情况下，程序将执行try语句块中的所有语句，直到有异常被抛出为止。此时，将跳过try语句块中的剩余代码，然后执行finally子句中的语句，并将异常抛给这个方法的调用者。在这里执行1，5处的语句。



try语句可以只有finally子句，而没有catch子句：

```java
InputStream in = new FileInputStream(...);
try {

} finally {
    in.close();
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

无论在try语句块中是否遇到异常，finally子句中的in.close()语句都会被执行。

这里建议解耦合try/catch和try/finally语句块，可以提高代码的清晰度。

```java
InputStream in = ...;
try {
    try {
    
    } finally {
        in.close();
    }
} catch (IOException e) {

}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

内存的try语句块确保关闭输入流。外层的try语句块确保报告出现的错误。这样还有一个好处，就是将会报告finally子句中出现的错误。

当finally子句包含return语句时，这个返回值将会覆盖原始的返回值。

```java
InputStream in = new FileInputStream(...);
try {

} finally {
    in.close();
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

如果在try中抛出一个非IOException，执行finally，并调用close方法。而close本身可能抛出IOException，这种情况下原始的异常将会丢失，转而抛出close方法的异常。



### 带资源的try语句

假设资源属于一个实现了AutoCloseable接口的类，Java SE 7为上诉问题提供了有用的快捷方式。AutoCloseable接口有一个方法：

void close() throws Exception

带资源的try语句(try-with-resources)的最简形式：

```java
try (Resource res = ...) {
    使用资源
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

try块退出时，会自动调用res.close()。

```java
try (Scanner in = new Scanner(new FileInputStream("E://test.txt"), "UTF-8")) {
    while (in.hasNext()) {
        System.out.println(in.next());
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

这个块正常退出时或者存在一个异常时，都会调用in.close()方法。还可以指定多个资源：

```java
try (Scanner in = new Scanner(new FileInputStream("E://test.txt"), "UTF-8");
     PrintWriter out = new PrintWriter("out.text")) {
    while (in.hasNext()) {
        out.println(in.next().toUpperCase());
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

带资源的try语句会将原来的异常重新抛出，而close方法抛出的异常会被抑制。这些异常将自动捕获，并由addSuppressed方法增加到原来的异常。可以通过getSuppressed方法，得到从close方法抛出并被抑制的异常列表。

###   分析堆栈轨迹元素

**堆栈轨迹（stack trace）**是一个方法调用过程的列表，包含了程序执行过程中方法调用的特定位置。

可以调用Throwable类的printStackTrace方法访问堆栈轨迹的文本描述信息。

```java
Throwable t = new Throwable();
StringWriter out = new StringWriter();
t.printStackTrace(new PrintWriter(out));
String description = out.toString();
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

使用getStackTrace方法，会得到StackTraceElement对象的一个数组：

```java
Throwable t = new Throwable();
StackTraceElement[] frames = t.getStackTrace();
for (StackTraceElement frame : frames) {
    
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

StackTraceElement类含有能够获得文件名和当前执行代码行号的方法，同时能够获得类名和方法名，toString方法将产生一个格式化的字符串，其中包含所获得的信息。

静态的Thread.getAllStackTrace，可以产生所有线程的堆栈轨迹：

```java
Map<Thread, StackTraceElement[]> map = Thread.getAllStackTraces();
for (Thread t : map.keySet()) {
    StackTraceElement[] frames = map.get(t);
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```java
public class StackTraceTest {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        System.out.print("输入 n:" );
        int n = in.nextInt();
        factorial(n);
    }

    public static int factorial(int n) {
        System.out.println("阶层(" + n + "):");
        Throwable t = new Throwable();
        StackTraceElement[] frames = t.getStackTrace();
        for (StackTraceElement f : frames) {
            System.out.println(f);
        }
        int r;
        if (n <= 1) {
            r = 1;
        } else {
            r = n * factorial(n - 1);
        }
        System.out.println("返回 " + r);
        return r;
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



### 使用异常机制的技巧

1.异常处理不能代替简单的测试

2.不要过分地细化异常

3.利用异常层次结构

4.不要压制异常

5.在检测错误时，苛刻要比放任更好

6.不要羞于传递异常

7.在try后面catch不能先大后小，不能先IOException，后FileNotFoundException，因FileNotFoundException包括在IoException中



## 4.使用断言

### 断言的概念

如果一个数为负值，抛出一个异常：

```java
if (x < 0) {
    throw new IllegalAccessException("x < 0");
}
```

但这段代码会一直保留在程序中，即使测试完毕也不会自动删除。 断言机制允许在测试期间想代码插入一些检查语句，当代码发布时，这些插入的检测语句将会被自动地移走。 Java引入关键字assert： assert 条件; assert 条件 : 表达式; 这两种形式都会对条件进行检测，如果结果为false，则抛出一个AssertionError异常。第二种形式，表达式将被传入AssertionError构造器，并转换成一个消息字符串。

### 启动和禁用断言


在默认情况下断言被禁止。IDEA开启断言，在VM options中添加-ea： ![img](https://img-blog.csdnimg.cn/20191126112906342.png) 在启用或禁止断言时不必重新编译程序，启用或禁止断言是==类加载器（class loader）==的功能。当断言被禁用时，类加载器将会跳过断言代码，因此不会降低程序的运行速度。 -da或者删除-ea可以关闭断言。

### 使用断言完成参数检查

在Java中给出了3种处理系统错误的机制： 1.抛出异常 2.日志 3.断言 断言失败是致命的、不可恢复的错误，断言检查只用于开发和测试阶段。因此不应该使用断言向程序员的其他部分通告发生了可恢复性的错误，或者不应该做为程序向用户通告问题的手段。断言只应用于在测试阶段确定程序内部的错误位置。

假设方法的约定：

```java
@param a the array to be sorted(must not be null)
```

那这个方法的调用者就必须注意：不允许用null数组调用这个方法，并在这个方法开头使用断言： assert a != null； 计算机科学家将这种约定成为前置条件（Precondition）。调用者在调用方法时没有提供满足这个前置条件的参数，断言就会失败，将会出现难以预料的结果，有时会抛出一个断言错误，有时会产生一个null指针异常，这完全取决于类加载器的配置。

断言是一种测试和调试阶段所使用的战术性工具；而日志记录是一种在程序的整个生命周期都可以使用的策略性工具。

## 5.记录日志

### 基本日志

全局日志记录器（global logger）：

```java
Logger.getGlobal().info("File->Open menu item selected");
// 十一月 26, 2019 1:33:13 下午 LoggerTest main
// 信息: File->Open menu item selected
```

在适当的地方调用：

```java
Logger.getGlobal().setLevel(Level.OFF);
```

将会取消所有日志。

### 高级日志

企业级（industrial-strength）日志，在专业的应用程序中，不要讲所有的日志都记录到一个全局记录器中，而是可以自定义记录器。

```java
private static final Logger myLogger = Logger.getLogger("chapter7.section5");
```

日志记录器级别： SEVERE WARNING INFO CONFIG FINE FINER FINEST 默认情况下，只记录前三个级别。也可以设置其他级别： logger.setLevel(Level.FINE); 现在，FINE和更高级别的记录都可以记录下来。还可以使用Level.ALL开启所有级别的记录。或者用Level.OFF关闭所有级别的记录。 还有：

```java
logger.warning(message);
logger.fine(message);
logger.log(Level.FINE, message);
```

默认的日志记录显示包含日志调用类的类名和方法。但是如果虚拟机对执行过程进行优化，就得不到精确的调用信息。可以调用logp方法获得调用类和方法的确切位置： void logp(Level l, String className, String methodName, String message)

void entering，将生成FINER级别和以字符串ENTRY和RETURN开始的日志记录。 void throwing，以记录一条FINER级别的记录和一条以THROW开始的信息。

### 修改日志管理器配置

可以通过编辑配置文件来修改日志系统的各种配置。默认情况下，配置文件存在于：jre/lib/logging.properties 使用其他配置文件，就要将java.util.logging.config.file特性设置为配置文件的存储位置：

```java
java -Djava.util.logging.config.file=configFile MainClass
```


IDEA配置方式： ![img](https://img-blog.csdnimg.cn/2019112721174728.png)

日志管理器在VM启动过程中初始化，在main之前完成。如果在main中调用System.setProperty(“java.util.logging.config.file”, file)也会调用LogManager.readConfiguration()来重新初始化日志管理器。

```java
# 修改默认日志记录级别
.level = INFO
# 指定自己的日志记录
com.mycompany.myapp.level = FINE
# 指定控制台级别
java.util.logging.ConsoleHandler.level = FINE
```

### 本地化


本地化的应用程序包含资源包中的本地特定信息。资源包由各个地区的映射集组成。如某个资源包将字符串“readingFile”映射出英文的“Reading file”或中文的“读取文件”。 想要将映射添加到资源包中，需要为每个地区创建一个文件，logmessages_en.propertise,logmessages_cn.propertise（en和zh是语言编码）。 ![img](https://img-blog.csdnimg.cn/20191127213018767.png) logmessages.propertise文件内容：

```java
readingFile=读取文件
moreMessage=更多{0}信息{1}
```


注意设置本地语言编码： ![img](https://img-blog.csdnimg.cn/2019112721334193.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

```java
/**
 * 在请求日志记录器时，可以指定资源包
 */
private static final Logger myLogger = Logger.getLogger("loggerName", "chapter7.section5.logmessages");
public static void main(String[] args) {
    // 为日志指定资源包的关键字，而不是实际的日志消息字符串
    myLogger.info("readingFile");
    // 通常需要在本地化的消息中增加一些参数，可以包含占位符｛0｝，｛1｝等
    myLogger.log(Level.INFO, "moreMessage", new Object[] { "的", "." });
}
//十一月 27, 2019 9:36:31 下午 chapter7.section5.LoggerTest main
//信息: 读取文件
//十一月 27, 2019 9:36:31 下午 chapter7.section5.LoggerTest main
//信息: 更多的信息.
```

### 处理器

在默认情况下，日志记录器将记录发送到ConsoleHandler中，并由它输出到System.err流中。特别是，日志记录器还会将记录发送到父处理器中，而最终的处理器有一个ConsoleHandler。 与日志记录器一样，处理器也有日志记录级别。对于一个要被记录的日志记录，它的日志记录级别必须高于日志记录器和处理器的阈值。

```java
java.util.logging.ConsoleHandler.level = FINE
```

还可以绕过配置文件，安装自己的处理器：

```java
Logger logger = Logger.getLogger("com.mycompany.myapp");
logger.setLevel(Level.FINE);
logger.setUseParentHandlers(false);
Handler handler = new ConsoleHandler();
handler.setLevel(Level.FINE);
logger.addHandler(handler);
```

日志API为此提供了两个很有用的处理器，一个是FileHandler，可以收集文件中的记录；另一个是SocketHandler。SocketHandler将记录发送到特定的主机和端口。

```java
FileHandler handler = new FileHandler();
logger.addHandler(handler );
logger.info("读取文件");
```


这些记录被发送到用户主目录（Windows95/98/Me，C:\Window）的javan.log文件中，n是文件名的唯一编号。 在默认情况下，记录被格式化为XML。 ![img](https://img-blog.csdnimg.cn/20191127214417531.png)

```java
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!DOCTYPE log SYSTEM "logger.dtd">
<log>
<record>
  <date>2019-11-27T21:41:12</date>
  <millis>1574862072789</millis>
  <sequence>4</sequence>
  <logger>loggerName</logger>
  <level>INFO</level>
  <class>chapter7.section5.LoggerTest</class>
  <method>main</method>
  <thread>1</thread>
  <message>读取文件</message>
  <key>readingFile</key>
  <catalog>chapter7.section5.logmessages</catalog>
</record>
</log>
```

文件处理器配置参数：

日志记录文件模式变量：

开启文件循环，日志文件以myapp.log.0，myapp.log.1，这种循环序列的形式出现。只要文件超出了大小限制，最旧的文件会被删除，其他的文件将重新命名，同时创建一个文件，其编号为0。

还可以通过扩展Handler类或StreamHandler类自定义处理器，例如下述程序中的WindowHandler 。

### 过滤器

在默认情况下，过滤器根据日志记录的级别进行过滤。每个日志记录器和处理器都可以有一个可选的过滤器来完成附加的过滤。可以通过实现Filter接口并定义下列方法来自定义过滤器。

```java
boolean isLoggable(LogRecord record)
```

可以利用自己喜欢的标准，对日志记录进行分析，返回true表示这些记录应该包含在日志中。同一时刻最多只能有一个过滤器。

### 格式化器

ConsoleHandler类和FileHandler类可以生成文本和XML格式的日志记录。但是也可以自定义格式，这需要扩展Formatter类并覆盖下面方法： String format(LogRecord record) String formatMessage(LogRecord record)对记录中的部分信息进行格式化、参数替换和本地化应用操作。 String getHead(Handler h)和String getTail(Handler h)可以在已格式化的记录前后加一个头部和尾部。最后调用setFormatter方法将格式化器安装到处理器中。

### 日志记录说明

1.为一个简单的程序选择一个日志记录器。并把日志记录器命名为与主程序包一样的名字。为了方便起见，可以将静态域添加到类中：

```java
private static final Logger logger = Logger.getLogger("com.mycompany.myprog");
```

2.默认的日志配置将级别等于或高于INFO级别的所有消息记录到控制台。用户可以覆盖默认的配置文件。 3.可以记录自己想要的内容。所有级别为INFO、WARNING和SEVERE的消息都将显示在控制台上。最好只将对程序用户有意义的消息设置为这几个级别。程序员想要的日志记录，设置为FINE是一个很好的选择。

```java
public class LoggingImageViewer {
    public static void main(String[] args) {
        if (System.getProperty("java.util.logging.config.class") == null
                && System.getProperty("java.util.logging.config.file") == null) {
            try {
                Logger.getLogger("com.spring.corejava").setLevel(Level.ALL);
                final int LOG_ROTATION_COUNT = 10;
                Handler handler = new FileHandler("%h/LoggingImageViewer.log", 0, LOG_ROTATION_COUNT);
                Logger.getLogger("com.spring.corejava").addHandler(handler);
            } catch (IOException e) {
                Logger.getLogger("com.spring.corejava").log(Level.SEVERE, "不能创建日志文件处理器", e);
            }
        }
        EventQueue.invokeLater(() -> {
            Handler windowHandler = new WindowHandler();
            windowHandler.setLevel(Level.ALL);
            Logger.getLogger("com.spring.corejava").addHandler(windowHandler);
            JFrame frame = new ImageViewerFrame();
            frame.setTitle("LoggingImageViewer");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            Logger.getLogger("com.spring.corejava").fine("显示帧");
            frame.setVisible(true);
        });
    }
}
class ImageViewerFrame extends JFrame {
    private static final int DEFAULT_WIDTH = 300;
    private static final int DEFAULT_HEIGHT = 400;
    private JLabel label;
    private static Logger logger = Logger.getLogger("com.spring.corejava");
    public ImageViewerFrame() {
        logger.entering("ImageViewerFrame", "<init>");
        setSize(DEFAULT_WIDTH, DEFAULT_HEIGHT);
        JMenuBar menuBar = new JMenuBar();
        setJMenuBar(menuBar);
        JMenu menu = new JMenu("File");
        menu.add(menu);
        JMenuItem openItem = new JMenuItem("Open");
        menu.add(openItem);
        openItem.addActionListener(new FileOpenListener());
    }
    private class FileOpenListener implements ActionListener {
        @Override
        public void actionPerformed(ActionEvent e) {
            logger.entering("ImageViewerFrame.FileOpenListener", "actionPerformed", e);
            JFileChooser chooser = new JFileChooser();
            chooser.setCurrentDirectory(new File("."));
            chooser.setFileFilter(new javax.swing.filechooser.FileFilter(){
                @Override
                public boolean accept(File f) {
                    return f.getName().toLowerCase().endsWith(".gif") || f.isDirectory();
                }
                @Override
                public String getDescription() {
                    return "GIF图片";
                }
            });
            int r = chooser.showOpenDialog(ImageViewerFrame.this);
            if (r == JFileChooser.APPROVE_OPTION) {
                String name = chooser.getSelectedFile().getPath();
                logger.log(Level.FINE, "读取文件{0}", name);
                label.setIcon(new ImageIcon(name));
            } else {
                logger.fine("文件打开对话框已取消");
            }
            logger.exiting("ImageViewerFrame.FileOpenListener", "actionPerformed");
        }
    }
}
class WindowHandler extends StreamHandler {
    private JFrame frame;
    public WindowHandler() {
        frame = new JFrame();
        final JTextArea output = new JTextArea();
        output.setEditable(false);
        frame.setSize(200, 200);
        frame.add(new JScrollPane(output));
        frame.setFocusableWindowState(false);
        frame.setVisible(true);
        setOutputStream(new OutputStream() {
            @Override
            public void write(int b) throws IOException {
            }
            @Override
            public void write(byte[] b, int off, int len) throws IOException {
                output.append(new String(b, off, len));
            }
        });
    }
    @Override
    public void publish(LogRecord record) {
        if (!frame.isVisible()) {
            return;
        }
        super.publish(record);
        flush();
    }
}
```



![img](https://img-blog.csdnimg.cn/20191127215021827.png) ![img](https://img-blog.csdnimg.cn/20191127214947676.png)

```java
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!DOCTYPE log SYSTEM "logger.dtd">
<log>
<record>
  <date>2019-11-27T21:50:39</date>
  <millis>1574862639844</millis>
  <sequence>0</sequence>
  <logger>com.spring.corejava</logger>
  <level>FINER</level>
  <class>ImageViewerFrame</class>
  <method>&lt;init&gt;</method>
  <thread>17</thread>
  <message>ENTRY</message>
</record>
<record>
  <date>2019-11-27T21:50:39</date>
  <millis>1574862639890</millis>
  <sequence>1</sequence>
  <logger>com.spring.corejava</logger>
  <level>FINE</level>
  <class>chapter7.section5.LoggingImageViewer</class>
  <method>lambda$main$0</method>
  <thread>17</thread>
  <message>显示帧</message>
</record>
</log>
```

## 6.调试技巧

1.打印或记录任意变量的值 2.每个类但需放置main方法，这样可以对每个类进行单元测试 3.JUnit非常常见的单元测试框架 4.日志代理（logging proxy）是一个子类的对象，它可以截取方法调用，并进行日志记录，然后调用超类中的方法。

```java
Random generator = new Random() {
  @Override
  public double nextDouble() {
      double result = super.nextDouble();
      Logger.getGlobal().info("nextDouble: " + result);
      return result;
  }  
};
```

5.利用Throwable类提供的printStackTrace方法，可以从任何一个异常对象中获得堆栈情况。 6.堆栈轨迹一般显示在System.err上，也可以利用printStackTrace(PrintWriter s)方法将它发送到一个文件中。

