# java.lang.Throwable
void pritStackTrace()
将Throwable对象和栈的轨迹输出到标准错误流
Throwable( )
构造一个新的 Throwabie 对象， 这个对象没有详细的描述信息。
Throwable(String message )
构造一个新的 throwabie 对象， 这个对象带有特定的详细描述信息。习惯上，所有派生的异常类都支持一个默认的构造器和一个带有详细描述信息的构造器。
String getMessage( )
获得 Throwabie 对象的详细描述信息
Throwable(Throwable cause)
Throwable(String message, Throwable cause)
用给定的原因构造一个Throwable对象
Throwable initCause(Throwable cause)
将这个对象设置为原因，如果这个对象已经被设置为原因，则抛出异常。返回this引用
Throwable getCause()
获得设置为这个对象的原因的异常对象。如果没有设置原因，则返回null
StackTraceElement[] getStackTrace()
获得构造这个对象时调用堆栈的跟踪
void addSuppressed(Throwable t)
为这个异常添加一个抑制异常，这出现在带资源的try语句中，其中t是close方法抛出的一个异常
Throwable[] getSuppressed()
得到这个异常的所有抑制异常，一般来说是带资源try语句中close方法抛出的异常

# java.lang.Exception
Exception(Throwable cause)
Exception(String message, Throwable cause)
用给定的原因构造一个异常对象

# java.lang.RuntimeException
RuntimeException(Throwable cause)
RuntimeException(String message, Throwable cause)
用给定的原因构造一个RuntimeException对象

# java.lang.StackTraceElement
String getFileName()
返回这个元素运行时对应的源文件名。如果这信息不存在， 则返回 null。
int getLineNumber()
返回这个元素运行时对应的源文件行数。如果这个信息不存在，则返回 -1。
String getClassName()
返回这个元素运行时对应的类的完全限定名。
String getMethodName()
返回这个元素运行时对应的方法名。构造器名< init >; 静态初始化器名是 < init > 。这里无法区分同名的重载方法
boolean isNativeMethod()
如果这个元素运行时在一个本地方法中， 则返回 true。
String toString()
如果存在的话， 返回一个包含类名、方法名、 文件名和行数的格式化字符串。

# javax.swing.JOptionPane
static void showMessageDialog(Component parent, Object message)
显示一个包含一条消息和ok按钮的对话框，这个对话框将位于parent组建的中央。如果parent为null，则位于屏幕中央。

# javax.swing.Timer
Timer(int interval, ActionListener listener)
构造一个定时器，每隔interval毫秒通告listener一次
void start()
启动定时器，一旦启动成功，定时器将调用监听器的actionPerformed
void stop()
停止定时器，一旦停止成功，定时器将不再调用监听器的actionPerformed

# java.awt.Toolkit
static Toolkit getDefaultToolkit()
获得默认的工具箱，工具箱包含有关GUI环境信息
void beep()
发出一声铃响 

# java.util.logging.Logger
• Logger getLogger(String loggerName)
• Logger getLogger(String loggerName, String bundleName)
获得给定名字的日志记录器。如果这个日志记录器不存在， 创建一个日志记录器。
参数：
loggerName 具有层次结构的日志记录器名。例如com.mycompany.myapp
bundleName 用来查看本地消息的资源包名
• void severe(String message)
• void warning(String message)
• void info(String message)
• void config(String message)
• void fine(String message)
• void finer(String message)
• void finest(String message)
记录一个由方法名和给定消息指示级别的日志记录。
• void entering(String className, String methodName)
• void entering(String className, String methodName, Object param)
• void entering(String className, String methodName, Object[] param)
• void exiting(String className, String methodName)
• void exiting(String className, String methodName, Object result)
记录一个描述进人 /退出方法的日志记录， 其中应该包括给定参数和返回值。
• void throwing(String className, String methodName, Throwable t)
记录一个描述拋出给定异常对象的日志记录。
• void log(Level level, String message)
• void log(Level level, String message, Object obj)
• void log(Level level, String message, Object[] objs)
• void log(Level level, String message, Throwable t)
记录一个给定级别和消息的日志记录， 其中可以包括对象或者可抛出对象。要想包括对象， 消息中必须包含格式化占位符 {0}、 {1} 等。
• void logp(Level level, String className, String methodName, String message)
• void logp(Level level, String className, String methodName, String message, Object obj)
• void logp(Level level, String className, String methodName, String message, Object[] objs)
• void logp(Level level, String className, String methodName, String message, Throwable t)
记录一个给定级别、 准确的调用者信息和消息的日志记录， 其中可以包括对象或可抛出对象。
•void logrb(Level level, String className, String methodName, String bundleName, String message)
•void logrb(Level level, String className, String methodName, String bundleName, String message, Object obj)
•void logrb(Level level, String className, String methodName, String bundleName, String message, Object[] objs)
•void logrb(Level level, String className, String methodName, String bundleName, String message, Throwable t)
记录一个给定级别、 准确的调用者信息、 资源包名和消息的日志记录， 其中可以包括对象或可拋出对象。
•Level getLevel()
• void setLevel(Level l)
获得和设置这个日志记录器的级别。
•Logger getParent()
•void setParent(Logger l)
获得和设置这个日志记录器的父日志记录器。
•Handler[] getHandlers()
获得这个日志记录器的所有处理器。
•void addHandler(Handler h)
•void removeHandler(Handler h)
增加或删除这个日志记录器中的一个处理器。
•boolean getUseParentHandlers()
•void setUseParentHandlers(boolean b)
获得和设置“ use parent handler ” 属性。如果这个属性是 true， 则日志记录器会将全部的日志记录转发给它的父处理器。
•Filter getFilter()
•void setFilter(FiIter f)
获得和设置这个日志记录器的过滤器。

# java.util.logging.Handler
•abstract void publish(LogRecord record)
将日志记录发送到希望的目的地。
•abstract  void flush()
刷新所有已缓冲的数据
•abstract void close( )
刷新所有已缓冲的数据， 并释放所有相关的资源。
•Filter getFi1ter( )
•void setFilter(Filter f)
获得和设置这个处理器的过滤器。
•Formatter getFormatter( )
•void setFormatter(Formatter f)
获得和设置这个处理器的格式化器。
•Level getLevel ( )
•void setLevel (Level l)
获得和设置这个处理器的级别。
# java.util.logging.ConsoleHandler
•ConsoleHandler( )
构造一个新的控制台处理器。
# java.util.logging.FileHandler
•FileHandler(String pattern)
•FileHandler(String pattern, boolean append )
•FileHandler(String pattern, int limit, int count )
•FileHandler(String pattern, int limit, int count, boolean append )
构造一个文件处理器。
参数： 
pattern 构造日志文件名的模式。参见表 7-2 列出的模式变量
limit 在打开一个新日志文件之前， 日志文件可以包含的近似最大字节数
count 循环序列的文件数量
append 新构造的文件处理器对象应该追加在一个已存在的日志文件尾部，则为 true
# java.util.logging.LogRecord
•Level getLevel( )
获得这个日志记录的记录级别。
•String getLoggerName( )
获得正在记录这个日志记录的日志记录器的名字。
•ResourceBundle getresourceBundle( )
•String getresourceBundleName( )
获得用于本地化消息的资源包或资源包的名字。如果没有获得，则返回null。
•String getMessage( )
获得本地化和格式化之前的原始消息。
• Object[] getParameters()
获得参数对象。如果没有获得， 则返回 null。
• Throwable getThrown()
获得被拋出的对象。 如果不存在， 则返回 null。
• String getSourceClassName()
• String getSourceMethodName()
获得记录这个日志记录的代码区域。这个信息有可能是由日志记录代码提供的， 也有可能是自动从运行时堆栈推测出来的。如果日志记录代码提供的值有误，或者运行时代码由于被优化而无法推测出确切的位置，这两个方法的返回值就有可能不准确。
• long getMillis()
获得创建时间。以毫秒为单位（从 1970 年开始 。)
• long getSequenceNumber()
获得这个日志记录的唯一序列序号。
• int getThreadID()
获得创建这个日志记录的线程的唯一 ID 这些 ID 是由 LogRecord 类分配的，并且与其他线程的 ID 无关。
# java.util.logging.Filter
• boolean isLoggable(LogRecord record)
如果给定日志记录需要记录， 则返回 true。
# java.util.logging.Formatter
• abstract String format(LogRecord record)
返回对日志记录格式化后得到的字符串。
• String getHead(Handler h)
• String getTail(Handler h)
返回应该出现在包含日志记录的文档的开头和结尾的字符串。超类 Formatter 定义了这些方法，它们只返回空字符串。如果必要的话，可以对它们进行覆盖。
• String formatMessage(LogRecord record)
返回经过本地化和格式化后的日志记录的消息内容。
