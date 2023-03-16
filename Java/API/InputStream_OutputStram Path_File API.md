# java.io.InputStream
abstract int read()
从数据中读入一个字节，并返回改字节。这个read方法碰到输入流结尾时返回-1
int read(byte[] b)
读入一个字节数组，并返回实际读入的字节数，或者碰到输入流结尾时返回-1，这个read方法最多读入b.length个字节
int read(byte[] b, int off, int len)
读入一个字节数组，并返回实际读入的字节数，或者碰到输入流结尾时返回-1
参数： b    数据读入的数组
			off  第一个读入字节应该被放置的位置在b中的偏移量
			len 读入字节的最大数量
long skip(long n)
在输入流中跳过n个字节，返回实际跳过字节数（结尾处可能小于n）
int available()
返回在不阻塞的情况下获取的字节数
void close()
关闭这个输入流
void mark(int readlimit)
在输入流的当前位置打一个标记，如果从输入流已经读入的字节多于readlimit个，则这个流允许忽略这个标记
void reset()
返回到最后一个标记，随后对read的调用将重新读入这些字节。如果当前没有任何标记，则这个流不被重置
boolean markSupported()
如果这个流支持打标记，则返回true

# java.io.OutputStream
abstract int write(int n)
写出一个字节的数据
int write(byte[] b)
int write(byte[] b, int off, int len)
写出所有字节或者某个范围的字节到数组b中
参数： b    数据写出的数组
			off  第一个写出字节在b中的偏移量
			len 写出字节的最大数量
void close()
关闭这个输出流
void flush()
冲刷输出流，也就是将所有缓冲的数据发送到目的地

# java.io.Closeable
void close()
关闭这个Closeable，这个方法可能抛出IOException

# java.io.Flushable
void flush()
冲刷这个Flushable

# java.lang.Readable
int read(CharBuffer cb)
尝试着向cb读入其可持有数量的char值，返回读入的char值的数量，或者当从这个Readable中无法再获得更多的值时返回-1

# java.lang.Appendable
Appendable append(char c)
Appendable append(CharSequence s)
向这个Appendable中追加给定的码元或者给定序列中的所有码元，返回this

# java.lang.CharSequence
char charAt(int index)
返回给定索引出的码元
int length()
返回在这个序列在的码元的数量
CharSequence subSequence(int startIndex, int endIndex)
返回由存储在startIndex到endIndex-1处的所有码元构成的CharSequenc
String toString()
返回这个序列中所有码元构成的字符串

# java.io.FileInputStream
FileInputStream(String name)
FileInputStream(File file)
使用有name字符串或file对象指定路径名的文件创建一个新的文件输入流。非绝对路径名将按照相对于VM启动时所设置的工作目录来解析
FileChannel getChannel()
返回用于访问这个输入流的通道

# java.io.FileOutputStream
FileOutputStream(String name)
FileOutputStream(String name, boolean append)
FileOutputStream(File file)
FileOutputStream(File file, boolean append)
使用由name字符串或file对象指定路径名的文件创建一个新的文件输出流。如果append为true，那么数据将被添加到文件尾，而具有相同名字的已有文件不会被删除；否则，这个方法会删除具有相同名字的已有文件。
FileChannel getChannel()
返回用于访问这个输出流的通道

# java.io.BufferedInputStream
BufferedInputStream(InputStream in)
创建一个带缓冲区的输入流。带缓冲区的输入流在从流中读入字符时，不会每次都对设备访问。当缓冲区为空时，会向缓冲区中读入一个新的数据块

# java.io.BufferedOutputStream
创建一个带缓冲区的输出流。带缓冲区的输处流在收集要写的字符时，不会每次都对设备访问。当缓冲区填满或当流被冲刷时，数据就会被写入。

# java.io.PushbackInputStream
PushbackInputStream(InputStream in)
PushbackInputStream(InputStream in, int size)
构建一个可以预览一个字节或具有指定尺寸的回推缓冲区的输入流
void unread(int b)
回推一个字节，它可以在下次调用read时被再次获取。

# java.io.PrintWriter
PrintWriter(Writer out)
PrintWriter(Writer writer)
创建一个向给定的写入器写出的新的PrintWriter
PrintWriter(String filename, String ecoding)
PrintWriter(File file, String encoding)
创建一个使用给定的编码方式向给定文件写入的新的PrintWriter
void print(Object obj)
通过打印从toString产生的字符串来打印一个对象
void print(String s)
打印一个包含Unicode码元的字符串
void println(String s)
打印一个字符串，后面紧跟一个行终止符。如果这个流处于自动冲刷模式，那么就会冲刷这个流
void print(char[] s)
打印在给定的字符串中的所有Unicode码元
void print(int i)
void print(long l)
void print(float f)
void print(double d)
void print(boolean b)
以文本格式打印给定的值
void printf(String format, Object... args)
按照格式字符串指定的方式打印给定的值（格式化字符串的规范）
boolean checkError()
如果产生格式化或输出错误，则返回true。一旦这个流碰到了错误，它就受到了污染，并且所有对checkError的调用都将返回true

# java.io.DataInput
char readChar()
byte readByte()
int readInt()
short readShort()
long readLong()
float readFloat()
double readDouble()
boolean readBoolean()
读入一个给定类型的值
void readFully(byte[] b)
将字节读入到数组b中，其间阻塞直至所有字节都读入
void readFully(byte[] b, int off, int len)
将字节读入到数组b中，其间阻塞直至所有字节都读入
b    数据读入的缓冲区
off  数据起始位置的偏移量
len 读入字节的最大数量
String readUTF()
读入由修订过的UTF-8格式的字符构成的字符串
int skipBytes(int n)
跳过n个字节，其间阻塞直至鄋字节被跳过

# java.io.DateOutput
void writeByte(int b)
void writeInt(int i)
void writeShort(int s)
void writeLong(long l)
void writeFloat(float f)
void writeDouble(double d)
void writeChar(int c)
void writeBoolean(boolean b)
写出一个给定类型的值
void writeUTF(String s)
写出字符串中的所有字符
void writeChars(String s)
写出由修订过的UTF-8格式的字符构成的字符串

# java.io.RandomAccessFile
RandomAccessFile(String file, String mode)
RandomAccessFile(File file, String mode)
mode，r表示只读模式，rw表示读写模式，rws表示每次更新时，都对数据和元数据的写磁盘操作进行同步的读写模式。rwd表示每次更新时，只对数据的写磁盘操作进行同步读写模式
long getFilePointer()
返回文件指针的当前位置
void seek(long pos)
将文件指针设置到距文件开头pos个字节处
long length()
返回文件按照字节来度量的长度
FileChannel getChannel()
返回用于访问这个文件的通道

# java.util.zip.ZipInputStream
ZipInputStream(InputStream in)
创建一个ZipInputStream，使得可以从给定的InputStream向其中填充数据
ZipEntry getNextEntry()
为下一项返回ZipEntry对象，或者在没有更多的项时返回null
void closeEntry()
关闭这个ZIP文件中当前打开的项。之后可以通过使用getNextEntry()读入下一项

# java.util.zip.ZipOutputStream
ZipOutputStream(OutputStream out)
创建一个将压缩数据写出到指定的OutputStream的ZipOutputStream
void putNextEntry(ZipEntry ze)
将给定的ZipEntry中的信息写出到输出流中，并定位用于写出数据的流，然后这些数据可以通过write()写出到这个输出流中
void closeEntry()
关闭这个ZIP文件中当前打开的项。之后可以通过使用putNextEntry()开始下一项
void setLevel(int level)
设置后续的各个DEFLATED项的默认压缩级别。这里默认值是Deflater.DEFAULT_COMPRESSION。如果级别无效则抛出IllegalArgumentException。
参数level，压缩机别，从0（NO_COMPRESSION）到9（BEST_COMPRESSION）
void setMethod(int method)
设置用于这个ZipOutputStream的默认压缩方法，这个压缩方法会作用域所有没有指定压缩方法的项上
参数method，压缩方法，DEFLATED或STORED

# java.util.zip.ZipEntry
ZipEntry(String name)
用给定的名字创建一个Zip项
long getCrc()
返回用于这个ZipEntry的CRC32校验和的值
String getName()
返回这一项的名字
long getSize()
返回这一项未压缩的尺寸，或者在未压缩的尺寸不可知的情况下返回-1
boolean isDirectory()
当这一项是目录时返回true
void setMethod(int method)
用于这一项的压缩方法，必须是DEFLATED或STORED
void setSize(long size)
设置这一项的尺寸，只有在压缩方法是STORED时才是必须的（size未压缩尺寸）
void setCrc(long crc)
给这一项设置CRC32校验和，这个校验和是使用CRC32类计算的。只有在压缩方法是STORED时才是必须的

# java.util.zip.ZipFile
ZipFile(String name)
ZipFile(File file)
创建一个ZipFile，用于从给定的字符串或File对象中读入数据
Enumeration entries()
返回一个Enumeration对象，它枚举了描述这个ZipFile中各个项的ZipEntry对象
ZipEntry getEntry(String name)
返回给定名字所对应的项，或者在没有对应项时候返回null
InputStream getInputStream(ZipEntry ze)
返回用于给定项的InputStream
String getName()
返回这个ZIP文件的路径

# java.io.ObjectOutputStream
ObjectOutputStream(OutputStream out)
创建一个ObjectOutputStream使得可以将对象写出到指定的OutputStream
void writeObject(Object obj)
写出指定的对象到ObjectOutputStream，这个方法将存储指定对象的类、类的签名以及这个类及其超类中所有非静态和非瞬时的域的值

# java.io.ObjectInputStream
ObjectInputStream(InputStream in)
创建一个ObjectInputStream用于从指定的InputStream中读回对象信息
Object readObject()
从ObjectInputStream中读入一个对象。特别是，这个方法会读回对象的类、类的签名以及这个类及其超类中所有非静态和非瞬时的域的值。它执行的反序列化允许恢复多个对象的引用

# java.nio.file.Paths
static Path get(String first, String... more)
通过连接给的的字符串创建一个路径

# java.nio.file.Path
Path resolve(Path other)
Path resolve(String other)
如果other是绝对路径，那么就返回other；否则，返回通过连接this和other获得的路径
Path resolveSibling(Path other)
Path resolveSibling(String other)
如果other是绝对路径，那么就返回other；否则，返回通过连接this的父路径和other获得的路径
Path relativize(Path other)
返回用this进行解析，相对于other的相对路径
Path normalize()
移除诸如.和..等冗余的路径元素
Path toAbsolutePath()
返回与该路径等价的绝对路径
Path getParent()
返回父路径，或者在该路径没有父路径时，返回null
Path getFileName()
返回该路径的最后一个部件，或该路径没有任何部件时，返回null
Path getRoot()
返回该路径的根部件，或在该路径没有根部件时，返回null
toFile()
从该路径创建一个File对象

# java.io.File
Path toPath()
从该文件中创建一个Path对象

# java.nio.file.Files
static byte[] readAllBytes(Path path)
static List< String > readAllLines(Path path, Charset charset)
读入文件的内容
static Path write(Path path, byte[] contents, OpenOption... options)
static Path write(Path path, Iterable< ? extends CharSequence > contents, OpenOption... options)
将给定内容写出到文件中，并返回path
static InputStream newInputStream(Path path, OpenOption... options)
static OutputStream newOutputStream(Path path, OpenOption... options)
static Reader newBufferedReader(Path path, Charset charset)
static Writer newBufferedWriter(Path path, Charset charset, OpenOption... options)
打开一个文件，用于读入或写出
static Path createFile(Path path, FileAttribute< ? >... attrs)
static Path createDirectory(Path path, FileAttribute< ? >... attrs)
static Path createDirectories(Path path, FileAttribute< ? >... attrs)
创建一个文件或目录，createDirectories方法还会创建路径中所有的中间目录
static Path createTempFile(String prefix, String suffix, FileAttribute< ? >... attrs)
static Path createTempFile(Path parentDir, String prefix, String suffix, FileAttribute< ? >... attrs)
static Path createTempDirectory(String prefix, FileAttribute< ? >... attrs)
static Path createTempDirectory(Path parentDir, String prefix, FileAttribute< ? >... attrs)
在合适临时文件的位置，或在给定的父目录中，创建一个临时文件或目录。返回所创建的文件或目录的路径

static Path copy(Path from, Path to, CopyOption... options)
static Path move(Path from, Path to, CopyOption... options)
将from复制或移动给定位置，并返回to
static long copy(InputStream from, Path to, CopyOption... options)
static long copy(Path from, OutputStream to, CopyOption... options)
从输入流复制到文件中，或者从文件复制到输出流中，返回复制的字节数
static void delete(Path path)
static boolean deleteIfExists(Path path)
删除给定文件或空目录。第一个方法在文件或目录不存在情况下抛出异常，而第二个方法在这种情况下会返回false

static boolean exists(Path path)
static boolean isHidden(Path path)
static boolean  isReadable(Path path)
static boolean  isWritable(Path path)
static boolean  isExecutable(Path path)
static boolean  isRegularFile(Path path)
static boolean  isDirectory(Path path)
static boolean  isSymbolicLink(Path path)
检查由路径指定的文件的给定属性
static long size(Path path)
获取文件按字节数度量的尺寸
A readAttributes(Path path, Class< A > type, LinkOption... option)
读取类型为A的文件熟悉

static DirectoryStream< Path > newDirectoryStream(Path path)
static DirectoryStream< Path > newDirectoryStream(Path path, String glob)
获取给定目录中可以遍历所有文件和目录的迭代器。第二个方法只接受那些与给定glob模式匹配的项
static Path walkFileTree(Path start, FileVisitor< ? super Path >)
遍历给定路径的所有子孙，并将访问器应用于这些子孙之上

# java.io.file.attribute.BasicFileAttributes
FileTime creationTime()
FileTime lastAccessTime()
FileTime lastModifiedTime()
boolean isRegularFile()
boolean isDirectory()
boolean isSymbolicLink()
long size()
Object fileKey()
获取所请求的属性

# java.nio.file.SimpleFileVisitor< T >
static FileVisitResult visitFile(T path, BasicFileAttributes attrs)
在访问文件或目录时被调用，返回CONTINUE、SKIP_SUBTREE、SKIP_SIBLINGS、TERMINATE之一，默认实现是不是做任何操作而继续访问
static FileVisitResult preVisitDirectory(T dir, BasicFileAttributes attrs)
static FileVisitResult postVisitDirectory(T dir, BasicFileAttributes attrs)
在访问目录之前或之后被调用，默认实现是不作任何操作继续访问
static FileVisitResult visitFileFailed(T path, IOException exc)
如果在试图获取给定文件的信息时抛出异常，则该方法被调用。默认实现是重新抛出异常，这会导致访问操作终止。如果想自己访问，可以覆盖这个方法

# java.nio.file.FileSystems
static FileSystem newFileSystem(Path path, ClassLoader loader)
对所安装的文件系统提供者进行迭代，并且如果loader不为null，那么就还迭代给定的类加载器能够加载的文件系统，返回由第一个可以接受给定路径的文件系统提供者创建的文件系统。默认情况下，对于ZIP文件系统是有一个提供者的，他接受的名字以.zip或.jar结尾的文件

# java.nio.file.FileSystem
static Path getPath(String first, String... more)
将给定的字符串连接起来创建一个路径

# java.nio.channels.FileChannel
static FileChannel open(Path path, OpenOption... options)
打开指定路径的文件通道，默认情况下，通道打开时用于读入。options，StandardOpenOption枚举中的WRITE、APPEND、TRUNCATE_EXIATING、CREATE值
MappedByteBuffer map(FileChannel.MapMode mode, long position, long size)
将文件的一个区域映射到内存中，mode：FileChannel.MapMode类中的常量READ_ONLY、READ_WRITE、PRIVATE，position映射区域的起始位置，size映射区域大小

FileLock lock()
在整个文件上获得一个独占的锁，这个方法将阻塞直至获得锁
FileLock teyLock()
在整个文件上获得一个独占的锁，或者在无法获得锁时返回null
FileLock lock(long position, long size, boolean shared)
FileLock tryLock(long position, long size, boolean shared)
在文件的一个区域上获得锁。第一个方法将阻塞直至获得锁，第二个在无法获得锁时返回null
position要锁定区域的起始位置，size要锁定区域的尺寸

# java.nio.Buffer
boolean hasRemaining()
如果当前的缓冲区位置没有达到这个缓冲区的界限位置，返回true
int limit()
返回这个缓冲区的界限位置，即没有任何值可用的第一个位置

Buffer clear()
通过将位置复位到0，并将界限设置到容量，使这个缓冲区为写出做好准备。返回this
Buffer flip()
通过将界限设置到位置，并将位置复位到0，使这个缓冲区为读入做好准备，返回this
Buffer rewind()
通过将读写位置复位到0，并保持界限不变，使这个缓冲区为重新读入相同的值做好准备，返回this
Buffer mark()
将这个缓冲区的标记设置到读写位置，返回this
Buffer reset()
将这个缓冲区的位置设置到标记，从而允许被标记的部分可以再次被读入或写出，返回this
int remaining()
返回剩余可读入或可写出的值的数量，即界限与位置之间的差异
int position()
void position(int newValue)
返回这个缓冲区的位置
int capacity()
返回这个缓冲区的容量

# java.nio.ByteBuffer
byte get()
从当前位置获得一个字节，并将当前位置移动到下一个字节
byte get(int index)
从指定索引处获得一个字节
ByteBuffer put(byte b)
向当前位置推入一个字节，并将当前位置移动到下一个字节。返回对这个缓冲区的应用
ByteBuffer put(int index, byte b)
向指定索引处推入一个字节。返回对这个缓冲区的引用
ByteBuffer get(byte[] destination)
ByteBuffer get(byte[] destination, int offset, int length)
用缓冲区中的字节来填充字节数组，或者字节数组的某个区域，并将当前位置向前移动读入的字节数个位置。如果缓冲区不够大，那么就不会读入任何字节，并抛出BufferUnderflowException。返回对这个缓冲区的引用。
ByteBuffer put(byte[] source)
ByteBuffer put(byte[] source, int offset, int length)
将字节数组中的所有字节或者给定区域的字节推入缓冲区中，并将当前位置向前移动写出的字节数个位置。如果缓冲区不够大，那么就不会读入任何字节，并抛出BufferUnderflowException。返回对这个缓冲区的引用。
Xxx getXxx()
Xxx getXxx(int index)
ByteBuffer putXxx(Xxx value)
ByteBuffer putXxx(int index, Xxx value)
获得或放置一个二进制数。Xxx是Int、Long、Short、Char、Float或Double中的一个
ByteBuffer order(ByteOrder order)
ByteOrder order()
设置或获得字节顺序，order的值是ByteOrder类的常量BIG_ENDIAN或LITTLE_ENDIAN中的一个
static ByteBuffer allocate(int capacity)
构建具有给定容量的缓冲区
static ByteBuffer wrap(byte[] values)
构建具有指定容量的缓冲区，该缓冲区是对给定数组的包装
CharBuffer asCharBuffer()
构建字符缓冲区，它是对这个缓冲区的包装。对该字符缓冲区的变更将在这个缓冲区中反应出来，但是该字符缓冲区有自己的位置、界限和标记。

# java.nio.CharBuffer
char get()
CharBuffer get(char[] destination)
CharBuffer get(Char[] destination, int offset, int length)
从这个缓冲区的当前位置开始，获得一个char值，或者一个范围内的所有char值，然后将位置向前移动越过所有读入的字符。最后两个方法将返回this
CharBuffer put(Char c)
CharBuffer put(char[] source)
CharBuffer put(char[] source, int offset, int length)
CharBuffer put(String source)
CharBuffer put(CharBuffer source)
从这个缓冲区的当前位置开始，放置一个char值，或者一个范围内的所有char值，然后将这个位置向前移动越过所有被出出的字符。当放置的值是从CharBuffer读入时，将读入所有剩余字符。所有方法将返回this。

# java.nio.channels.FileLock
void close()
释放这个锁
