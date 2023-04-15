#  1.输入/输出流

 

从其中读入一个字节序列的对象成为输入流，而可以向其中写入一个字节序列的对象成为输出流。这些字节序列的来源地和目的地可以是文件，而且通常是文件，但也可以是网络连接，甚至是内存块。抽象类InputStream和OutPutStream构成了输入/输出（I/O）类层次结构的基础。

面向字节的流不便于处理以Unicode形式存储的信息（字符使用多个字节表示），所以从抽象类Reader和Writer中继承出来一个专门用于处理Unicode字符的单独的类层次结构，这些类基于两字节的Char值，而不是byte值。    

|        | 字节流       | 字符流 |
| ------ | ------------ | ------ |
| 输入流 | InputStream  | Reader |
| 输出流 | OutputStream | Writer |



### 读写字节

InputStream类的抽象方法：abstract int read()，这个方法将读入一个字节，并返回读入的字节，或者在遇到输入结尾时返回-1。

FileInputStream从某文件中读入一个字节，而System.in（InputStream的一个子类的预定义对象）却是从标准输入中读入信息，即控制台或重定向文件。

InputStream类还有若干个非抽象的方法，这些方法都要调用抽象的read方法，各个子类都只需覆盖这个方法。



OutputStream类定义了抽象方法：abstract void write(int b)，它可以向某个输出位置写出一个字节。

read和write方法在执行时都被阻塞，直至字节确实被读入或者写出（如果流不能被立即访问，当前线程将被阻塞）。

available方法可以去检查当前读入的字节数量，下面的代码片段就不会被阻塞：

```java
int bytesAvailable = in.available();
if (bytesAvailable  > 0) {
    byte[] data = new byte[bytesAvailable];
    in.read(data);
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

当完成输入/输出流的读写时，应该通过调用close方法来关闭它，这个调用会释放掉十分有限的操作系统资源。如果打开过多的输入/输出流而没有关闭，那么系统资源将被耗尽。关闭一个输出流的同时还会冲刷用于该输出流的缓冲区：所有被临时置于缓冲区中，以便用更大的包的形式传递的字节在关闭输出流时都被送出。

特别是，如果不关闭文件，那么写出字节的最后一个包可能将永远得不到传递，可以用flush方法来认为冲刷这些输出。

尽管输入输出流都提供了原生的read和write的某些具体方法，但是很少使用它们，大家感兴趣的数据包括数字，字符串和对象，而不是字节。

```java
FileInputStream in = new FileInputStream("1.txt");
int b;
while ((b = in.read()) != -1) {
    System.out.println(b + " " + (char) b);
}
in.close();
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```java
byte[] bytes = {122,104,97,110,103};
FileOutputStream out = new FileOutputStream("2.txt");
out.write(bytes);
out.close();
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### 完整的流家族

![img](https://img-blog.csdnimg.cn/20191219141200497.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

按照使用方法来进行划分，处理字节和字符的两个单独的层次结构。InputStream和OutputStream可以读写单个字节或字节数组。想要读写字符串和数字，需要功能更强大的子类。

对于Unicode文本，可以使用抽象类Reader和Writer，与输入输出流类似，read方法将返回一个Unicode码元（0~65535的整数）。write需要传递一个Unicode码元。

![img](https://img-blog.csdnimg.cn/2019121914185091.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

还有四个附加的接口：Closeable、Flushable、Readable和Appendable。前面两个接口分别拥有void close() throw IOException和void flush()。

InputStream、OutputStream、Reader和Writer都实现了Closeable接口，而OutputStream和Writer还实现了Flushable接口。

Readable接口只有一个方法int read(CharBuffer cb)，CharBuffer类拥有按顺序和随机地进行读写访问的方法，它表示一个内存中的缓冲区或者一个内存映像的文件。

Appendable接口有两个用于添加单个字符和字符序列的方法：

Appendable append(char c)

Appendable append(CharSequence s)

CharSequence接口描述了一个char值序列的基本属性，String、CharBuffer、StringBuilder和StringBuffer都实现了它。

在流类的家族中，只有Wirter实现了Appendable。

![img](https://img-blog.csdnimg.cn/20191219144341737.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

### 组合输入/输出流过滤器

FileInputStream和FileOutputStream可以提供附着在一个磁盘文件上的输入输出流，只需向构造器提供文件名：

```java
FileInputStream fin = new FileInputStream("employee.dat");
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

由于反斜杠字符在java中是转义字符，Windows风格的路径名可以使用\\或者/。

与抽象类InputStream和OutputStream一样，这些类只支持字节级别的读写。

然而DataInputStream就只能读入数值类型：

```java
DataInputStream din = ...;
double x = din.readDouble();
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

FileInputStream没有任何读入数值类型的方法，DataInputStream也没有任何从文件中获取数据的方法，必须对二者进行组合：

```java
FileInputStream fin = new FileInputStream("employee.dat");
DateInputStream din = new DateInputStream(fin);
double x = din.readDouble();
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

可以通过嵌套过滤器来添加多重功能。输入流默认情况下不被缓冲区缓存，每个对read的调用都会请求操作系统在分发一个字节。相比之下数据块置于缓冲区中会更高效：

```java
DateInputStream din = new DateInputStream(new BufferedInputStream(new FileInputStream("employee.dat")));
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



有时当多个输入流链接在一起时，需要跟踪各个中介输入流（intermediate inputstream）：

```java
PushbackInputStream pbin = new PushbackInputStream(new BufferedInputStream(
   new FileInputStream("employee.dat")));
int b = pbin.read();
if (b != '<') {
    pbin.unread(b);
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

预读下一个字节，并非所期望的值时，将其推回流中。

但是读入和推回是可应用于可回推（pushback）输入流的仅有的方法。如果希望预先浏览并且还可以读入数字：

```java
DataInputStream din = new DataInputStream(pbin = new PushbackInputStream(new BufferedInputStream(
        new FileInputStream("employee.dat"))));
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

这种混合并匹配过滤器类以构建真正有用的输入输出流序列的能力，将带来极大的灵活性。

```java
ZipInputStream zin = new ZipInputStream(new FileInputStream("employee.zip"));
DataInputStream in = new DataInputStream(zin);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

![img](https://img-blog.csdnimg.cn/20191219152448971.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑



# 2.文本输入与输出

在保存数据时，可以选择二进制格式或文本格式。例如整数1234，二进制由字节00 00 04 D2构成的序列（十六进制表示法），而文本存成字符串“1234”。尽管二进制IO高效但是不宜人来阅读。

在存储文本字符串时，需要考虑字符编码（character encoding）方式。在Java内部使用UTF-16编码方式，字符串1234编码为00 31 00 32 00 33 00 34 十六进制。许多程序希望文本按照其他的编码方式编码，UTF-8是最常用的的编码方式，字符串讲写成4A 6F 73 C3 A9，其中并没有用于前3个字母的任何0字节，而字符é占用了两个字节。

OutputStreamWriter类将使用选定的字符编码方式，把Unicode码元的输出流转换为字节流。而InputStreamReader类将包含字节（用某种字符编码方式表示的字符）的输入流转换为可以产生Unicode码元的读入器。

```java
Reader in = new InputStreamReader(System.in);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

这个输入流读入器会假定使用主机系统所使用的默认字符编码方式。应该总是在InputStreamReader的构造器中选择一种具体的编码方式：

```java
Reader in = new InputStreamReader(new FileInputStream("data.txt"), StandardCharsets.UTF_8);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### 如何写出文本输出

对于文本输出，可以使用PrintWriter。这个类拥有以文本格式打印字符串和数字的方法，它还有一个将PrintWriter链接到FileWriter的便捷方法：

```java
PrintWriter out = new PrintWriter("employee.txt", "UTF-8");
// 等于
PrintWriter out_p = new PrintWriter(new FileOutputStream("employee.txt"), "UTF-8");
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

为了输出到打印写出器，需要使用与System.out相同的方法：

```java
String name = "Harry Hacker";
double salary = 75000;
out.print(name);
out.print(' ');
out.println(salary);
out.close();
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

输出到写入器out，之后这些字符将会被转换成字节并最终写入employee.txt中。println方法在行中添加了对目标系统来说恰当的行结束符（Windows系统是“\r\n”，Unix系统是“\n”）。

如果写入器设置为自动冲刷模式，那么只要println被调用，缓冲区中的所有字符都会被发送到它们的目的地（打印写出器总是带缓冲区的）。默认情况下，自动冲刷机制是禁用的的，可以使用PrintWriter(Writer out, Boolean autoFlush)来启用或禁用自动冲刷机制：

```java
PrintWriter out = new PrintWriter(new OutputStreamWriter(new FileOutputStream("employee.txt"), "UTF-8"), true);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

print方法不抛出异常，可以调用checkError方法来查看输出流是否出现了某些错误。

### 如何读入文本输出

最简单的方式是使用Scanner类，可以从任意输入流中构建Scanner对象。

也可以将短小文本文件读入到一个字符串：

```java
String content = new String(Files.readAllBytes(path), charset);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

一行行读入：

```java
List<String> lines = Files.readAllLines(path, charset);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

文件过大，可以将行惰性处理为一个Stream< String >对象：

```java
Stream<String> lines = Files.lines(path, charset);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

早期的Java版本中，处理文本输入的唯一方式是通过BufferedReader类，它的readLine方法会产生一行文本：

```java
InputStream inputStream = new FileInputStream("f.txt");
BufferedReader in = new BufferedReader(new InputStreamReader(inputStream, StandardCharsets.UTF_8));
String line;
while ((line = in.readLine()) != null) {
    System.out.println(line);
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

如今，BufferedReader类又有了一个lines方法，可以产生一个Stream< String >对象。但与Scanner不同，BufferedReader没有任何用于读入数字的方法。



### 以文本格式存储对象

Employee记录数组存储成一个文本文件：

Harry Hacker|35500|2019-12-20

需要使用PrintWriter类：

```java
public static void writeEmployee(PrintWriter out, Employee e) {
    out.println(e.getName() + "|" + e.getSalary() + "|" + e.getHireDay());
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

为了读入记录，每次读入一行然后分离所有字段，用String.split方法将这一行断开成一组标记：

```java
public static Employee readEmployee(Scanner in) {
    String line = in.nextLine();
    // 正则表达式 \字符表示转义，需要用另一个\来转义
    String[] tokens = line.split("\\|");
    String name = tokens[0];
    double salary = Double.parseDouble(tokens[1]);
    LocalDate hireDate = LocalDate.parse(tokens[2]);
    int year = hireDate.getYear();
    int month = hireDate.getMonthValue();
    int day = hireDate.getDayOfMonth();
    return new Employee(name, salary, year, month, day);
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```java
public class TextFileTest {

    public static void main(String[] args) throws IOException {
        Employee[] staff = new Employee[3];
        staff[0] = new Employee("Carl Cracker", 75000, 2019, 12, 20);
        staff[1] = new Employee("Harry Hacker", 50000, 2019, 12, 19);
        staff[2] = new Employee("Tony Tester", 40000, 2019, 12, 18);

        try (PrintWriter out = new PrintWriter("employee.dat", "UTF-8")) {
            writeData(staff, out);
        }

        try (Scanner in = new Scanner(new FileInputStream("employee.dat"), "UTF-8")) {
            Employee[] newStaff = readData(in);

            for (Employee e : newStaff) {
                System.out.println(e);
            }
        }
    }

    private static void writeData(Employee[] employees, PrintWriter out) {
        out.println(employees.length);

        for (Employee e: employees) {
            writeEmployee(out, e);
        }
    }

    private static Employee[] readData(Scanner in) {
        int n = in.nextInt();
        // nextInt读入的是数组长度，但不包括行尾的换行字符，必须调用nextLine方法获得下一行输入
        in.nextLine();

        Employee[] employees = new Employee[n];
        for (int i = 0; i < n; i++) {
            employees[i] = readEmployee(in);
        }
        return employees;
    }

    public static void writeEmployee(PrintWriter out, Employee e) {
        out.println(e.getName() + "|" + e.getSalary() + "|" + e.getHireDay());
    }

    public static Employee readEmployee(Scanner in) {
        String line = in.nextLine();
        String[] tokens = line.split("\\|");
        String name = tokens[0];
        double salary = Double.parseDouble(tokens[1]);
        LocalDate hireDate = LocalDate.parse(tokens[2]);
        int year = hireDate.getYear();
        int month = hireDate.getMonthValue();
        int day = hireDate.getDayOfMonth();
        return new Employee(name, salary, year, month, day);
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### 字符编码方式

输入流和输出流都用于字节序列，但许多情况下操作的是文本即字符序列。字符如何编码成字节？

Java字符使用Unicode标准。每个字符或编码都具有一个21位的整数。有多种不同的字符编码方式，也就是说，将这些21位数字包装成字节的方式有很多种。

最常见的编码方式是UTF-8，它将每个Unicode编码点编码为1到4个字节的序列。UTF-8的好处是传统的包含了英语中用到的所有字符的ASCⅡ字符集中的每个字符都只会占用一个字节。

![img](https://img-blog.csdnimg.cn/20191220135702405.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

UTF-16会将每个Unicode编码点编码为1个或2个16位值。这是一种在Java字符串中使用的编码方式。

![img](https://img-blog.csdnimg.cn/20191220140140450.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

ISO 8859-1是一种单字节编码，包含了西欧各种语言中用到的带有重音符号的字符。Shift-JIS是一种用于日文字符的可变长编码。

平台使用的编码方式可以由静态方法Charset.defaultCharset返回。静态方法Charset.availableCharsets会返回所有可用的Charset实例，返回结果从字符集的规范名称到Charset对象的映射表。

StandardCharsets类具有类型为Charset的静态变量，用于表示虚拟机必须支持的字符编码方式：

StandardCharsets.US_ASCII
 StandardCharsets.ISO_8859_1
 StandardCharsets.UTF_8
 StandardCharsets.UTF_16BE
 StandardCharsets.UTF_16LE
 StandardCharsets.UTF_16

为了获得另一种编码方式的Charset，可以使用静态forName方法：

```java
Charset shiftJIS = Charset.forName("Shift-JIS");
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

在读入或写出文本时，应该使用Charset对象：

```java
String str = new String(bytes, StandardCharsets.UTF_8);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

在不指定任何编码方式时，有些方法（例如String(byte[])）会使用默认平台编码方式，而其他方法（例如Files.readAllLines）会使用UTF-8。



# 3.读写二进制数据

### DataInput和DataOutput接口

DataOutput接口定义了下面用以二进制格式写数组、字符、boolean值和字符串的方法：

writeChars, writeByte, writeInt, writeShort, writeLong, writeFloat, writeDouble, writeChar, writeBoolean, writeUTF

writeInt总是将一个整数写出为4字节的二进制数量值。对于给定类型的每个值，所需空间相同，而且将其读回也比解析文本要更快。

writeUFT方法用修订版的8位Unicode转换格式写出字符串（与标准UFT-8不同，Unicode码元序列首先用UTF-16表示，其结果之后用UTF-8进行编码）。为了向后兼容在Unicode还没有超过16位时构建的虚拟机。因为没有其他方法会使用UTF-8的这种修订，所以只在写出用于java虚拟机的字符串时才使用writeUTF，例如当编写一个生产字节码的程序时。其他场合都应该使用wirteChars方法。

为了读回数据，DateInput接口中定义了方法：

readByte, readInt, readShort, readLong, readFloat, readDouble, readChar, readBoolean, readUTF

DataInputStream类实现了DataInput接口，为了从文件中读入二进制数据，可以将DataInputStream与某个字节源结合：

```java
DateInputStream in = new DateInputStream(new FileInputStream("employee.dat"));
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

写入二进制数据，与此类似。



### 随机访问文件

RandomAccessFile类可以在文件中的任何位置查找或写入数据。磁盘文件都是随机访问的。可以打开一个随机文件，用于读入访问（r）同时用于读写（rw）。

```java
RandomAccessFile in = RandomAccessFile("employee.dat", "r");
RandomAccessFile inOut = RandomAccessFile("employee.dat", "rw");
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

当将已有文件做为RandomAccessFile打开时，这个文件并不会删除。

随机访问文件有一个表示下一个将被读入或写出的字节所处位置的文件指针，seek可以把这个指针设置到任意位置。seek参数是一个long类型的整数，它的值位于0到文件按照字节来度量的长度之间。

getFilePointer返回文件指针的当前位置。

RandomAccessFile同时实现了DataInput和DataOutput。

将雇员记录存储到随机访问的文件中：每条记录都有一样的大小，很容易读入任何记录。

将文件指针置于第三条纪录：

```java
long n = 3;
in.seek((n - 1) * RECORD_SIZE);
Employee e = new Employee()
e.readDate(in);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

总记录数：

```java
long nbytes = in.length();
int nrecords = (int) (nbytes  / RECOED_SIZE);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

整数和浮点数都有固定尺寸。但是字符串就需要助手方法来读写具有固定尺寸的字符串：

writeFixedString写出从字符串开头开始的指定数量的码元（如果码元过少，用0值补齐字符串）。

readFixedString从输入流中读入字符，直至读入size个码元，或遇到具有0值的字符值，然后跳过输入字段中剩余的0值。为了提高效率这个方法使用StringBuilder类来读入字符串。

使用40个字符来表示姓名，因此每条记录包含100个字节：

40字符=80字节，用于姓名

1 double=8字节，用于薪水

3 int=12字节，用于日期

```java
public static final int NAME_SIZE = 40;
public static final int RECODE_SIZE = 2 * NAME_SIZE + 8 + 4 + 4 + 4;

public class DataIO {
    public static void writeFixedString(String s, int size, DataOutput out) throws IOException {
        for (int i = 0; i < size; i++) {
            char ch = 0;
            if (i < s.length()) {
                ch = s.charAt(i);
            }
            out.writeChar(ch);
        }
    }
    public static String readFixedString(int size, DataInput in) throws IOException {
        StringBuilder b = new StringBuilder(size);
        int i = 0;
        boolean more = true;
        while (more && i < size) {
            char ch = in.readChar();
            i++;
            if (ch == 0) {
                more = false;
            } else {
                b.append(ch);
            }
        }
        in.skipBytes(2 * (size - i));
        return b.toString();
    }
}

public class RandomAccessTest {

    public static void main(String[] args) throws IOException {
        Employee[] staff = new Employee[3];
        staff[0] = new Employee("Carl Cracker", 75000, 2019, 12, 20);
        staff[1] = new Employee("Harry Hacker", 50000, 2019, 12, 19);
        staff[2] = new Employee("Tony Tester", 40000, 2019, 12, 18);

        try (DataOutputStream out = new DataOutputStream(new FileOutputStream("employee.dat"))) {
            for (Employee e : staff) {
                writeData(out, e);
            }
        }

        try (RandomAccessFile in = new RandomAccessFile("employee.dat", "r")) {
            int n = (int)(in.length() / Employee.RECODE_SIZE);
            Employee[] newStaff = new Employee[n];

            for (int i = n - 1; i >= 0; i--) {
                newStaff[i] = new Employee();
                in.seek(i * Employee.RECODE_SIZE);
                newStaff[i] = readData(in);
            }

            for (Employee e : newStaff) {
                System.out.println(e);
            }
        }
    }

    // 为了写出一条固定尺寸的记录，直接以二进制方式写出所有字段
    public static void writeData(DataOutput out, Employee e) throws IOException {
        DataIO.writeFixedString(e.getName(), Employee.NAME_SIZE, out);
        out.writeDouble(e.getSalary());

        LocalDate hireDay = e.getHireDay();
        out.writeInt(hireDay.getYear());
        out.writeInt(hireDay.getMonthValue());
        out.writeInt(hireDay.getDayOfMonth());
    }

    public static Employee readData(DataInput in) throws IOException {
        String name = DataIO.readFixedString(Employee.NAME_SIZE, in);
        double salary = in.readDouble();
        int y = in.readInt();
        int m = in.readInt();
        int d = in.readInt();
        return new Employee(name, salary, y, m - 1, d);
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### ZIP文档

ZIP文档以压缩格式存储了一个或多个文件，每个ZIP文档都有一个头，包含诸如每个文件名字和所使用的压缩方法等信息。

Java中，使用ZipInputStream来读入ZIP文档。getNextEntry方法可以返回一个描述文档中每个项的ZipEntry类型的对象。向ZipInputStream的geiInputStream方法传递该项可以获取用于读取该项的输入流。然后调用closeEntry来读入下一项：

```java
ZipFile zf = new ZipFile("first.zip");
ZipInputStream zin = new ZipInputStream(new FileInputStream("first.zip"));
ZipEntry entry;
while ((entry = zin.getNextEntry()) != null) {
    InputStream inputStream = zf.getInputStream(entry);
    BufferedReader in = new BufferedReader(new InputStreamReader(inputStream, StandardCharsets.UTF_8));
    String line;
    while ((line = in.readLine()) != null) {
        System.out.println(line);
    }
    zin.closeEntry();
}
zin.close();
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

写入Zip文件使用ZipOutputStream，希望放入Zip文件都要创建一个ZipEntry对象，并将文件名传递给ZipEntry构造器，它将设置文件日期和解压方法等参数（可以覆盖这些设置）。然后调用putNextEntry来开始写出新文件，并将文件发送给ZIP输出流中。当完成时，需要调用closeEntry：

```java
FileOutputStream fout = new FileOutputStream("haha.zip");
ZipOutputStream zout = new ZipOutputStream(fout);
for (int i = 0; i < 3; i++) {
    ZipEntry ze = new ZipEntry("hello" + i + ".txt");
    zout.putNextEntry(ze);
    byte[] bytes = {122,104,97,110,103};
    zout.write(bytes);
    zout.closeEntry();
}
zout.close();
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

JAR文件只是带有一个特殊项的ZIP文件，这个项称作清单（JarInput/OutputStream）。

ZIP输入流是一个能够展示流的抽象化的强大之处的实例。当读入以压缩格式存储的数据时，不必担心边请求边解压数据的问题。而且ZIP格式的字节源并非必须是文件，也可以是来自网络连接的ZIP数据。事实上，当Applet类加载器读入JAR文件时，就是在读入和解压来自网络的数据。



一个有趣的程序：

```java
public class TestFileWriter {
    public static void main(String args[]) {
        FileWriter fw = null;
        try {
            fw = new FileWriter("E:/2.txt");
            for (int c = 0; c <= 50000; c++) {
                fw.write(c);
            }
            fw.close();
        } catch (IOException e1) {
            e1.printStackTrace();
            System.out.println("文件写入错误");
            System.exit(-1);
        }
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 

结果是：生成文件B.txt,包含50000个不同地区的字符，如下：



![img](https://img-blog.csdn.net/20170208165356870?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑![img](https://img-blog.csdn.net/20170208165405706?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

![img](https://img-blog.csdn.net/20170208165417972?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑![img](https://img-blog.csdn.net/20170208165427996?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑





存储同类型数据时很少具有相同的类型（继承会出现存储多态集合），Java语言支持一种称为对象序列化（object serialization），它可以将任何对象写出到输出流中，并在之后将其读回。

### 保存和加载序列化对象

为了保存对象数据，首先需要打开一个ObjectOutputStream对象：

```java
ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("employee.dat"));
```

为了保存对象，可以直接使用ObjectOutputStream的writeObject方法。

为了将这些对象读回，首先需要获得一个ObjectInputStream对象：

```java
ObjectInputStream in = new ObjectInputStream(new FileInputStream("employee.dat"));
```

然后用readObject方法以这些对象写入时的顺序获得它们：

```java
Employee e1 = (Employee) in.readObject();
```

但是对希望在对象输出流中存储或从对象输入流中恢复的所有类都应该实现Serializable接口：

```java
class Employee implements Serializable {...}
```

Serializable接口没有任何方法，与Cloneable接口相似。但为了是类可克隆，仍需覆盖Object类的clone方法，而序列化不需要做任何事。

只有在写出对象时才能用writeObject/readObject方法，对于基本类型需要使用writeInt等（对象流都实现了DataInput/DataOutput接口）。

在幕后，是ObjecyOutputStream在浏览对象的所有域，并存储它们的内容。

当一个对象被多个对象共享，做为它们各自状态的一部分时：

```java
class Manager extends Employee {
    private Employee secretary;
    ...
}
//两个经理共用一个秘书
harry = new Employee("Harry Hacker", ...);
Manager carl = new Manager("Carl Carcker", ...);
carl.setSecretary(harry);
Manager tony = new Manager("Tony Tester", ...);
tony.setSecretary(harry);
```

这里不能去保存和恢复秘书对象的内存地址，因为当对象被重新加载时，它可能占据的是与原来完全不同的内存地址。与此不同的是，每个对象都用一个序列号（serial number）保存的，这就是这种机制之所以成为对象序列化的原因。其算法：

1.对你遇到的每一个对象引用都关联一个序列号

2.对于每个对象，当第一次遇到时，保存其对象数据到输出流中

3.如果某个对象之前已经被保存过，那么只写出“与之前保存过的序列号为x的对象相同”


![img](https://img-blog.csdnimg.cn/20191223192347854.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

在读回对象时，过程是反过来的：

1.对于对象输入流中的对象，在第一次遇到其序列号时，构建它，并使用流中数据来初始化它，然后记录这个序列号和新对象之间的关联

2.当遇到“与之前保存过的序列号为x的对象相同”标记时，标记时，获取与这个顺序号相关联的对象引用

```java
public class ObjectStreamTest {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        Employee harry = new Employee("Harry Hacker", 50000, 2019, 12, 23);
        Manager carl = new Manager("Carl Cracker", 80000, 2019, 12, 20);
        carl.setSecretary(harry);
        Manager tony = new Manager("Tony Tester", 40000, 2019, 12, 19);
        tony.setSecretary(harry);
        Employee[] staff = new Employee[3];
        staff[0] = carl;
        staff[1] = harry;
        staff[2] = tony;
        try (ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("employee.dat"))) {
            out.writeObject(staff);
        }
        try (ObjectInputStream in = new ObjectInputStream(new FileInputStream("employee.dat"))) {
            Employee[] newStaff = (Employee[]) in.readObject();
            // 秘书对象重新加载之后是唯一的，当秘书被恢复时，会反映到经理们的secretary域中
            newStaff[1].raiseSalary(10);
            for (Employee e : newStaff) {
                System.out.println(e);
            }
        }
    }
}
```

### 理解对象序列化的文件格式

### 修改默认的序列化机制

某些数据域是不可以序列化的。将它们标记为transient(瞬时)的。瞬时的域在对象被序列化时总被跳过。

序列化机制为单个类提供了一种方式，去向默认的读写行为添加验证或任何想要的行为。可序列化的类可以定义下列方法：

```java
private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException;
private void writeObject(ObjectOutputStream out) throws IOException;
```

之后数据域就不再会自动序列化，取而代之的是调用这些方法。

在java.aut.geom包中有大量的类都是不可序列化的。例如Point2D.Double，想要序列化一个LabeledPoint类，需要存储一个String和一个Point2D.Double。首先将Point2D.Double标记为transient，以避免NotSerializableException。

```java
public class LabeledPoint implements Serializable {
    private String label;
    private transient Point2D.Double point;
    
    // 调用defaultWriteObject方法写出描述符和String域label，这是ObjectOutputStream的一个特殊方法，只能在可序列化的writeObject方法中被调用
    private void writeObject(ObjectOutputStream out) throws IOException {
        out.defaultWriteObject();
        out.writeDouble(point.getX());
        out.writeDouble(point.getY());
    }

    private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
        in.defaultReadObject();
        double x = in.readDouble();
        double y = in.readDouble();
        point = new Point2D.Double(x, y);
    }
}
```

java.util.Date类，它提供了自己的readObject和writeObject方法，这些方法将日期写出为纪元（UTC时间1970年1月1日0点）开始的毫秒。Date类有一个复杂的内部表示，为了优化查询它存储一个Calendar对象和一个毫秒计数值。Calendar的状态是冗余的，因此不需要保存。

readObject和writeObject方法只需要保存和加载它们的数据域，而不需要关心超类数据和任何其他类的信息。

除了让序列化机制来保存和恢复对象数据，类还可以定义自己的机制。但必须实现Externalizable接口的writeExternal和readExternal。

与read/writeObject不同，这些方法对包含超类数据在内的整个对象的存储和恢复负全责。在写出对象时，序列化机制在输出流中仅仅只是记录该对象所属的类。在读入可外部化类时，对象输入流将用无参构造器创建一个对象，然后调用readExternal方法：

```java
@Override
public void writeExternal(ObjectOutput out) throws IOException {
    out.writeUTF(name);
    out.writeDouble(salary);
    out.writeLong(hireDay.toEpochDay());
}
@Override
public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
    name = in.readUTF();
    salary = in.readDouble();
    hireDay = LocalDate.ofEpochDay(in.readLong());
}
```

read/writeObject是私有的，并且只能被序列化机制调用。read/writeExternal是公有的，readExternal还潜在地允许修改现有对象的状态。

### 序列化单例和类型安全的枚举

在序列化和反序列化时，如果目标对象是唯一的，这通常会在实现单例和类型安全的枚举时发生。

如果使用Java语言的enum结构，不必担心序列化。可以使用==操作符来测试对象的等同性。但是：

```java
public class Orientation{
    public static final Orientation HORIZONTAL = new Orientation(1);
    public static final Orientation VERTICAL = new Orientation(2);
    private int value;

    private Orientation(int x) {
        value = x;
    }
}
```

这种风格在枚举被添加到Java之前很普遍。其构造器是私有的，因此不可能创建出HORIZONTAL和VERTICAL 之外的对象。

当类型安全的枚举实现Serializable接口时，默认序列化是不适用的。

```java
Orientation original = Orientation.HORIZONTAL;
System.out.println(original == Orientation.HORIZONTAL);
ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("orientation.dat"));
out.writeObject(original);
out.close();
ObjectInputStream in = new ObjectInputStream(new FileInputStream("orientation.dat"));
Orientation saved = (Orientation) in.readObject();
System.out.println(saved == Orientation.HORIZONTAL);
```

事实上saved是一个全新的对象，即使构造器是私有的，序列化机制也可以创建新对象。

此时，需要定义readResolve的特殊序列化方法，在对象被序列化之后会调用它，它必须返回一个对象，而该对象会成为readObject的返回值：

```java
protected  Object readResolve() throws Exception {
    if (value == 1) {
        return Orientation.HORIZONTAL;
    }
    if (value == 2) {
        return Orientation.VERTICAL;
    }
    throw new Exception();
}
```

请想遗留代码中所有类型安全的枚举以及所有支持单例设计模式的类中添加readResolve方法。

### 版本管理

### 为克隆使用序列化

序列化机制提供了一种克隆对象的简便途径，只要对应的类是可序列化的即可。直接将对象序列化到输出流，然后读回，产生的新对象是对现有对象的一个深拷贝（deep copy）。不必将对象写入文件，可以用ByteArrayOutputStream将数据保存在字节数组中：

```java
public class SerialCloneTest {
    public static void main(String[] args) throws CloneNotSupportedException {
        Employee harry = new Employee("Harry Hacker", 35000, 2019, 12, 24);
        Employee harry2 = (Employee) harry.clone();
        harry.raiseSalary(10);
        System.out.println(harry);
        System.out.println(harry2);
    }
}
class SerialCloneable implements Cloneable, Serializable {
    @Override
    public Object clone() throws CloneNotSupportedException {
        ByteArrayOutputStream bout = new ByteArrayOutputStream();
        try {
            try (ObjectOutputStream out = new ObjectOutputStream(bout)) {
                out.writeObject(this);
            }
            try (InputStream bin = new ByteArrayInputStream(bout.toByteArray())){
                ObjectInputStream in = new ObjectInputStream(bin);
                return in.readObject();
            }
        } catch (IOException | ClassNotFoundException e) {
            CloneNotSupportedException e2 = new CloneNotSupportedException();
            e2.initCause(e);
            throw e2;
        }
    }
}
class Employee extends SerialCloneable {
    private String name;
    private double salary;
    private LocalDate hireDay;
    public Employee(String n, double s, int year, int month, int day) {
        name = n;
        salary = s;
        hireDay = LocalDate.of(year, month, day);
    }
    public String getName() {
        return name;
    }
    public double getSalary() {
        return salary;
    }
    public LocalDate getHireDay() {
        return hireDay;
    }
    public void raiseSalary(double byPercent) {
        double raise = salary * byPercent / 100;
        salary += raise;
    }
    @Override
    public String toString() {
        return "Employee{" +
                "name='" + name + '\'' +
                ", salary=" + salary +
                ", hireDay=" + hireDay +
                '}';
    }
}
```


Path接口和Files类是在java SE 7中添加进来的，它们用起来比自JDK 1.0以来就一直使用的File类要方便的多。

### Path

Path表示的是一个目录名序列，其后还可以跟着一个文件名。路径中的第一个部件可以是根部件，例如/或C:\，而允许访问的根部件取决于文件系统。以根部件开始的路径是绝对路径，否则就是相对路径（相对于项目的根目录，即src的上级）。

```java
Path absolute = Paths.get("D:\\", "test", "test2.txt");
Path relative = Paths.get("test", "test.txt");
```


![img](https://img-blog.csdnimg.cn/20191224195011562.png)

静态的Paths.get方法接受一个或多个字符串，并用默认的文件系统的路径分隔符（Unix是/，Windows是\）连接起来。如果解析出来的不是给定文件系统的合法路径，就抛出InvalidPathException异常。这个连接起来的结果是一个Path对象。

get方法可以获取包含多个部件构成的单个字符串：

```java
// base.dir是一个存储路径的属性
String baseDir = props.getProperty("base.dir");
Path basePath = Paths.get(baseDir);
```

调用p.resolve(q)将按照下列规则返回一个路径：

如果q是绝对路径，则结果就是q，否则根据文件系统规则，将p后面跟着q作为结果。

resolveSibling，它通过解析指定路径的父路径产生其兄弟路径：

```java
Path basePath = Paths.get("E:\\corejava\\src");
Path workRelative = Paths.get("chapter3");
Path chapter3 = basePath.resolve(workRelative);
Path chapter4 = chapter3.resolveSibling("chapter4");
System.out.println(chapter3.toString());
System.out.println(chapter4.toString());
//E:\corejava\src\chapter3
//E:\corejava\src\chapter4
```

resolve的对立面是relativize，即调用p.relativize(r)将产生路径q，而对q进行解析的结果正是r。以/home/cay为目标对/home/fred/myprog进行相对化操作，会产生../fred/myprog，其中..表示文件系统中的父目录。

normalize方法将移除所有冗余的.和..部件（或者文件系统认为冗余的所有部件）。

toAbsolutePath方法将产生给定路径的绝对路径，该绝对路径从根部件开始。

Path类有许多有用方法用来将路径断开：

```java
Path p = Paths.get("/home", "fred", "myprog.properties");
Path parent = p.getParent(); // /home/fred
Path file = p.getFileName(); // myprog.properties
Path root = p.getRoot();     // /
```

还可以从Path对象中构建Scanner对象：

```java
Scanner in = new Scanner(Paths.get("/home/fred/input.txt"));
```

### 读写文件

Files类可以很容易地读取文件的所有内容：

```java
byte[] bytes = Files.readAllBytes(path);
// 将文件当作字符串读入
String content = new String(bytes, StandardCharsets.UTF_8);
// 将文件当做行序列读入
List<String> lines = Files.readAllLines(path, StandardCharsets.UTF_8);
```

相反的，写出字符串到文件中：

```java
Files.write(path, content.getBytes(StandardCharsets.UTF_8));
// 向指定文件追加内容
Files.write(path, content.getBytes(StandardCharsets.UTF_8), StandardOpenOption.APPEND);
// 将一个行的集合写出到文件
Files.write(path, lines);
```

这些简单的方法适用于中等长度的文本文件，如果要处理的文件长度比较长，或者是二进制文件，还是应该使用输入输出流和读入器写出器。

```java
InputStream in = Files.newInputStream(path);
OutputStream out = Files.newOutputStream(path);
Reader reader = Files.newBufferedReader(path);
Writer writer = Files.newBufferedWriter(path);
```

### 创建文件和目录

创建新目录可以调用：

Files.createDirectory(path)除了路径中最后一个部件外，其他部分必须都是已经存在的。

要创建路径中的中间目录要使用Files.createDirectories(path)。

可以使用Files.createFile(path)创建一个空文件，如果文件已存在，就会抛出异常。检查文件是否存在和创建文件是原子性的，如果文件不存在会被创建，并且其他程序在此过程是无法执行文件创建操作的。

可以在给定位置或系统指定位置创建临时文件或临时目录：

```java
Path newPath = Files.createTempFile(dir, prefix, suffix);
Path newPath = Files.createTempFile(prefix, suffix);
Path newPath = Files.createTempDirectory(dir, prefix);
Path newPath = Files.createTempDirectory(prefix);
```

dir是一个Path独享，prefix和suffix可以为null。调用Files.createTempFile(null, "".txt")会返回一个像/tmp/214565234.txt的路径。

### 复制、移动和删除文件

将文件从一个位置复制到另一个位置可以直接用：

Files.copy(fromPath, toPath);

移动文件（复制并删除原文件）可以调用：

File.move(fromPath, toPath)，如果目标路径已经存在，那么复制或移动将失败。如果想覆盖已有目标路径，可以使用REPLACE_EXISTING选项。如果想复制所有文件属性，可以使用COPY_ATTRIBUTES选项：

```java
Files.copy(fromPath, toPath, StandardCopyOption.REPLACE_EXISTING, StandardCopyOption.COPY_ATTRIBUTES);
```

使用ATOMIC_MOVE选项来实现原子性，保证要么移动操作完成，要么源文件保持在原来位置。

还可以将一个输入流复制到Path中，表示想要将输入流存储到硬盘上。同样的，可以将一个Path复制到输出流：

Files.copy(inputStream, toPath);

Files.copy(fromPath, outputStream);

删除文件：

Files.delete(path)，如果要删除的文件不存在，就会抛出异常。因此需要调用：

boolean deleted = Files.deleteIfExists(path)，该方法还可以用来移除空目录。

### 获取文件信息

下面的静态方法都将返回一个boolean值，表示检查路径的某个属性的结果：

exists, isHidden, isReadable, isWritable, isExecutable, isRegularFile, isDirectory, isSymbolicLink

size方法将返回文件的字节数：

```java
long fileSize = Files.size(path);
```

getOwner方法将文件的拥有者作为java.nio.file.attribute.UserPrincipal的一个实例返回。

所有文件系统都会报告一个基本属性集，被封装在BasicFileAttributes接口中，基本文件属性包括：

1.创建文件、最后一次访问以及最近一次修改文件的时间，这些时间都表示成java.nio.file.attribute.FileTime

2.文件是常规文件、目录还有符号链接，抑或三种都不是

3.文件尺寸

4.文件主键，这是某种类的对象

```java
BasicFileAttributes attributes = Files.readAttributes(path, BasicFileAttributes.class);
```

如果文件系统兼容POSIX：

```java
PosixFileAttributes attributes = Files.readAttributes(path, PosixFileAttributes.class);
```

从中找到组拥有者，以及文件的拥有者、组合访问权限。

### 访问目录中的项

静态的Files.list方法会返回一个可以读取目录中各个项的Stream&lt; Path &gt;对象。目录是被惰性读取的，因为读取目录涉及需要关闭的系统资源，所以应该使用try块：

```java
Path path = Paths.get("corejava\\src");
try(Stream<Path> entries = Files.list(path)) {
    entries.forEach(p -> System.out.println(p.getFileName()));
}
```

list方法不会进入子目录，为了处理目录中的所有子目录，需要使用Files.walk方法，只要遍历的项是目录，那么在进入它之前，会继续访问它的兄弟项。 可以通过File.walk(pathToRoot, depth)来限制想要访问的树的深度。两种walk方法都具有FileVisitOption…的可变长参数，但只能提供一种选项：FOLLOW_LINKS，即跟踪符号链接。 无法通过Files.walk方法来删除目录树，因为需要在删除父目录之前先删除子目录。

### 使用目录流

如果需要对遍历过程进行更加细粒度的控制，应该使用Files.newDirectoryStream，它会产生一个DirectoryStream，它不是java.util.stream.Stream的子接口，而是专门用于目录遍历的接口。它是Iterable的子接口，因此可以在增强for循环中使用目录流，访问目录中的项并没有具体顺序。 可以用glob模式来过滤文件：

```java
// try块确保目录流被正确关闭
try (DirectoryStream<Path> directories = Files.newDirectoryStream(path, "*.java")) {
    for (Path directory : directories) {
        System.out.println(directory.getFileName());
    }
}
```

Windows的glob语法则必须对反斜杠转义两次，一次为glob语法转义，一次为java字符串转义：

```java
Files.newDirectoryStream(dir, "C:\\\\");
```

如果想要访问某个子目录的所有子孙成员，可以调用walkFileTree方法，并向其传递一个FileVisitor类型的对象，这个对象会得到下列通知： 1.在遇到一个文件或目录时，FileVisitResult visitFile(T path, BasicFileAttributes attrs) 2.在一个目录被处理前，FileVisitResult preVisitDirectory(T dir, IOException ex) 3.在一个目录被处理后，FileVisitResult postVisitDirectory(T dir, IOException ex) 4.在视图访问文件或目录时发送错误，例如没有权限打开，FileVisitResult visitDirectory(path, IOException) 对于上述每种情况，都可以指定是否希望执行下面的操作： 1.继续访问下一个文件：FileVisitResult.CONTINUE 2.继续访问，但是不在访问这个目录的任何项，FileVisitResult.SKIP_SUBTREE 3.继续访问，但是不在访问这个文件的兄弟文件，FileVisitResult.SKIP_SIBLINGS 4.终止访问，FileVisitResult.TERMINATE 当有任何方法抛出异常时，就会终止访问，这个异常从walkFileTree方法抛出。 FileVisitor&lt; ? super Path &gt;接口是泛型类型，但也不太可能使用出Path之外的东西，因为Path并没有多少超类型。 便捷类SimpleFileVisitor实现了FileVisitor接口，但是其除visitFileFailed方法之外的所有方法并不做任何处理而是直接访问，而visitFileFailed方法会抛出由失败导致的异常，并进而终止访问：

```java
Files.walkFileTree(path, new SimpleFileVisitor<Path>(){
    @Override
    public FileVisitResult preVisitDirectory(Path p, BasicFileAttributes attrs) {
        System.out.println(p);
        return FileVisitResult.CONTINUE;
    }
    @Override
    public FileVisitResult postVisitDirectory(Path dir, IOException exc) {
       return FileVisitResult.CONTINUE;
    }
    
    @Override
    public FileVisitResult visitFileFailed(Path p, IOException exc) {
        return FileVisitResult.SKIP_SUBTREE;
    }
});
```

需要覆盖postVisitDirectory和visitFileFailed方法，否则，访问会遇到不允许打开或不允许任何访问的文件时立即失败。 路径的众多属性是作为preVisitDirectory和visitFile方法的参数传递的。访问者不得不通过操作系统调用来获得这些属性，因为它需要区分文件和目录。因此不需要再次执行系统调用了。 如果在进入或离开一个目录是要执行某些操作（删除目录树）：

```java
Files.walkFileTree(path, new SimpleFileVisitor<Path>(){
    @Override
    public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
        Files.delete(file);
        return FileVisitResult.CONTINUE;
    }
    
    @Override
    public FileVisitResult postVisitDirectory(Path dir, IOException e) throws IOException {
        if (e != null) {
            throw e;
        }
        Files.delete(dir);
        return FileVisitResult.CONTINUE;
    }
});
```

### ZIP文件系统

Paths类会在默认文件系统查找路径，即在用户本地磁盘中的文件。也可以使用别的文件系统，最有用之一是ZIP文件系统：

```java
FileSystem fs = FileSystems.newFileSystem(Paths.get(zipname), null);
```

将建立一个文件系统，包含ZIP文档中的所有文件。如果知道文件名，那么从ZIP文档中复制出这个文件就变得容易：

```java
Files.copy(fs.getPath(sourceName), targetPath);
```

列出ZIP文档中的所有文件，可以遍历文件树：

```java
Files.walkFileTree(fs.getPath("/"), new SimpleFileVisitor<Path>(){
   @Override
   public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) {
       System.out.println(file);
       return FileVisitResult.CONTINUE;
   }
});
```


大多数操作系统都可以利用虚拟内存实现将一个文件或文件的一部分映射到内存中。然后这个文件就可以当做是内存数组一样地访问，这比传统的文件操作要快得多。

### 内存映射文件的性能


![img](https://img-blog.csdnimg.cn/20191226153125195.png) 内存映射比使用带缓冲的顺序输入要稍微快一点，但比使用RandomAccessFile快很多。对于中等尺寸的顺序读入则没有必要使用内存映射。

首先从文件中获得一个通道（channel），通道是用于磁盘文件的一种抽象，它可以访问注入内存映射、文件加锁机制以及文件间快速数据传递等操作系统特性。

```java
FileChannel channel = FileChannel.open(path, options);
```

然后通过FileChannel类的map方法从通道中获得ByteBuffer。可以指定想要的文件区域与映射模式，支持的模式有三种： 1.FileChannel.MapMode.READ_ONLY：所产生的缓冲区是只读的，任何对该缓冲区写入的尝试都会导致ReadOnlyBufferException异常 2.FileChannel.MapMode.READ_WRITE：所产生的缓冲区是可写的，任何修改都会在某个时刻写回到文件中。注意，其他映射同一个文件的程序可能不能立即看到这些修改，多个程序同时进行文件映射的确切行为是依赖于操作系统的 3.FileChannel.MapMode.PRIVATE：所产生的缓冲区是可写的，但是任何修改对这个缓冲区来说都是私有的，不会传播到文件中 一旦有了缓存区，就可以使用ByteBuffer类和Buffer超类的方法读写数据了。缓冲区支持顺序和随机数据访问，它有一个可以通过get和put操作来移动的位置。

```java
// 顺序遍历缓冲区的所有字节
while(buffer.hasRemaining()){
	byte b = buffer.get();
}
// 随机访问
for (int i = 0; i < buffer.limit(); i++) {
	byte b = buffer.get(i);
}
```

使用下面方法读写字节数组： get(byte[] bytes) get(byte[] bytes, int offset, int length) 还有getInt，getLong，getShort，getChar，getFloat，getDouble用来读写在文件中存储为二进制的基本类型。Java对二进制数据使用高位在前的排序机制。但是如果需要以低位在前方式处理二进制数字的文件：

```java
// 查询缓冲区当前字节顺序
ByteOrder b = buffer.order();
buffer.order(ByteOrder.LITTLE_ENDIAN);
```

向缓冲区写数字，putInt，putLong，putShort，putChar，putFloat，putDouble，在恰当时机，以及当通道关闭时，会将这些修改写回到文件中。

下面程序是用于计算文件的32位的循环冗余校验和（CRC32），这个数值就是经常用来判断一个文件是否已损坏的，因为文件损坏极可能导致校验和改变。java.util.zip包中包含一个CRC32类，可以计算一个字节序列的校验和：

```java
public class MemoryMaoTest {
    public static long checksumInputStream(Path filename) throws IOException {
        try (InputStream in = Files.newInputStream(filename)) {
            CRC32 crc = new CRC32();
            int c;
            while ((c = in.read()) != -1) {
                crc.update(c);
            }
            return crc.getValue();
        }
    }

    public static long checksumBufferedInputStream(Path filename) throws IOException {
        try (InputStream in = new BufferedInputStream(Files.newInputStream(filename))) {
            CRC32 crc = new CRC32();
            int c;
            while ((c = in.read()) != -1) {
                crc.update(c);
            }
            return crc.getValue();
        }
    }

    public static long checksumRandomAccessFile(Path filename) throws IOException {
        try (RandomAccessFile file = new RandomAccessFile(filename.toFile(), "r")) {
            long length = file.length();
            CRC32 crc = new CRC32();
            for (long p = 0; p < length; p++) {
                file.seek(p);
                int c = file.readByte();
                crc.update(c);
            }
            return crc.getValue();
        }
    }

    public static long checksumMappedFile(Path filename) throws IOException {
        try (FileChannel channel = FileChannel.open(filename)) {
            CRC32 crc = new CRC32();
            int length = (int) channel.size();
            MappedByteBuffer buffer = channel.map(FileChannel.MapMode.READ_ONLY, 0, length);
            for (int p = 0; p < length; p++) {
                int c = buffer.get(p);
                crc.update(c);
            }
            return crc.getValue();
        }
    }

    public static void main(String[] args) throws IOException {
        System.out.println("Input Stream:");
        long start = System.currentTimeMillis();
        Path filename = Paths.get("C:\\Program Files\\Java\\jre1.8.0_77\\lib\\rt.jar");
        long crcValue = checksumInputStream(filename);
        long end = System.currentTimeMillis();
        System.out.println(Long.toHexString(crcValue));
        System.out.println((end - start) + " ms");

        System.out.println("Buffered Input Stream:");
        start = System.currentTimeMillis();
        crcValue = checksumBufferedInputStream(filename);
        end = System.currentTimeMillis();
        System.out.println(Long.toHexString(crcValue));
        System.out.println((end - start) + " ms");

        System.out.println("Random Access File:");
        start = System.currentTimeMillis();
        crcValue = checksumRandomAccessFile(filename);
        end = System.currentTimeMillis();
        System.out.println(Long.toHexString(crcValue));
        System.out.println((end - start) + " ms");

        System.out.println("Mapped File:");
        start = System.currentTimeMillis();
        crcValue = checksumMappedFile(filename);
        end = System.currentTimeMillis();
        System.out.println(Long.toHexString(crcValue));
        System.out.println((end - start) + " ms");
    }
    //Input Stream:
    //414112ba
    //98183 ms
    //Buffered Input Stream:
    //414112ba
    //288 ms
    //Random Access File:
    //414112ba
    //109077 ms
    //Mapped File:
    //414112ba
    //203 ms
}
```

### 缓冲区数据结构


缓冲区是由相同类型的数值构成的数组，Buffer类是一个抽象类，他有众多的具体子类，包括ByteBuffer、CharBuffer、DoubleBuffer、IntBuffer、LongBuffer和ShortBuffer（StringBuffer和缓冲区没关系）。 最常用的将是ByteBuffer和CharBuffer。每个缓冲区都具有： 1.一个容量，它永远不能改变 2.一个读写位置，下一个值将在此进行读写 3.一个界限，超过它进行读写是没有意义的 4.一个可选的标记，用于重复一个读入或写出操作 ![img](https://img-blog.csdnimg.cn/20191226175257873.png) 0&lt;=标记&lt;=位置&lt;=界限&lt;=容量 使用缓冲区的主要目的是执行“写，然后读入”循环。假设一个缓冲区在一开始，它的位置为0，界限等于容量。不断地调用put将值添加到这个缓冲区中，当耗尽所有的数据或写出的数据量达到容量大小时，就该切换到读入操作了。 这时调用flip方法将界限设置到当前位置，并把位置复位到0。现在在remaining方法返回正数时（它返回的值是“界限-位置”），不断地调用get。将缓冲区中所有的值都读入之后，调用clear使缓冲区为下一次写循环做准备。clear方法将位置复位到0，并将界限复位到容量。 如果想重读缓冲区，可以使用rewind或mark/reset方法。 要获取缓冲区，可以调用诸如ByteBuffer.allocate或ByteBuffer.wrap这样的静态方法。 然后可以用来自某个通道的数据填充缓冲区，或者将缓冲区内容写出通道中：

```java
ByteBuffer buffer = ByteBuffer.allocate(RECORD_SIZE);
channel.read(buffer);
channel.position(newpos);
buffer.flip();
channel.write(buffer);
```

### 文件加锁机制

多个同时执行的程序修改同一个文件的情形，很明显这些程序需要以某种方式通信，不然文件很容易被损坏。文件锁可以解决这个问题，它可以控制对文件或文件中某个范围的字节的访问。 两个实例都会希望写一个配置文件，第一个实例应该锁定这个文件，当第二个实例发现这个文件被锁定时，必须决策是等待直至这个文件解锁，还是直接跳过这个写操作。 要锁定一个文件，可以调用FileChannel类的lock域或tryLock方法：

```java
FileChannel channel = FileChannel.open(path);
FileLock lock = channel.lock();
// 或
// FileLock lock = channel.tryLock();
```

第一个调用会阻塞直至可获得锁，而第二个调用将立即返回，要么返回锁，要么在锁不可获得的情况下返回null。这个文件将保持锁定状态，直至这个通道关闭，或者在锁上调用了release方法。 可以通过调用锁文件的一部分：

```java
FileLock lock(long start, long size, boolean shared)
FileLock tryLock(long start, long size, boolean shared)
```

如果锁定了文件的结尾，但这个文件的长度随增长超过了锁定部分，那么额外部分是未锁定的，如果想要锁定所有字节，可以使用Long.MAX_VALUE来表示尺寸。 如果shared标志为false，则锁定文件的目的是读写，而如果为true，则是一个共享锁，它允许多个进程从文件中读入，并阻止任何进程获得独占的锁。 并非所有操作系统都支持共享锁，因此可能在请求共享锁时候得到独占的锁。调用FileLock类的isShared方法可以查询锁的类型。

要确保在操作完释放锁时，与往常一样最好在一个try语句中释放锁：

```java
try (FileLock lock = channel.lock()) {
	//访问锁定的文件或文件段
}
```

文件加锁机制依赖于操作系统，需要注意几点： 1.在某些系统中，文件加锁仅仅是建议性的，如果一个应用未能得到锁，它仍旧可以被另一个应用并发锁定的文件执行写操作 2.在某些系统中，不能在锁定一个问加你的同时将其映射到内存中 3.文件锁是由整个Java虚拟机持有的（两个程序由同一虚拟机启动，那不可能在每一个获得同一文件的锁），调用lock时，如果虚拟机已经在同一个文件锁持有了另一个重叠的锁，那么将抛出OverlappingFileLockException 4.在一些文件中，关闭一个通道会释放由Java虚拟机持有的底层文件上的所有锁。因此，在同一个锁定文件上应避免使用多个通道 5.在网络文件系统上锁定是高度依赖于系统的，因此应尽量避免

