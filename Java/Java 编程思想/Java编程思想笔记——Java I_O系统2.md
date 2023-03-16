##I/O流的典型使用方式
####缓冲输入文件
使用以String或File对象作为文件名的FileInputReader。为了提高速度，对文件进行缓冲，将所产生的引用传给一个BufferedReader构造器。
```
public class BufferedInputFile {
    public static String read(String filename) throws IOException {
        BufferedReader in = new BufferedReader(new FileReader(filename));
        String s;
        StringBuilder sb = new StringBuilder();
        while ((s = in.readLine()) != null) {
            sb.append(s + "\n");
        }
        // 调用close()关闭文件
        in.close();
        return sb.toString();
    }

    public static void main(String[] args) throws IOException {
        System.out.print(read("BufferedInputFile.java"));
    }
}/*输出该程序*/
```

####从内存输入
从BufferedInputFile.read()读入的String结果被用来创建一个StringReader。然后调用read()每次读取一个字符，并发送到控制台。
```
public class MemoryInput {
    public static void main(String[] args) throws IOException {
        StringReader in = new StringReader(BufferedInputFile.read("MemoryInput.java"));
        int c;
        while ((c = in.read()) != -1) {
            System.out.print((char) c);
        }
    }
}
```
read()是以int形式返回下一个字节，因此必须类型转换为char才能正确打印。

####格式化的内存输入
要读取格式化数据，可以使用DataInputStream，它是面向字节的IO类，因此必须用InputStream而不是Reader。
```
public class FormattedMemoryInput {
    public static void main(String[] args) throws IOException {
        try {
            DataInputStream in = new DataInputStream(new ByteArrayInputStream(BufferedInputFile.read("E:\\JAVA\\IdeaProjects\\untitled\\src\\io\\FormattedMemoryInput.java").getBytes()));
            while (true) {
                System.out.print((char) in.readByte());
            }
        } catch (EOFException e) {
            System.err.println("End of stream");
        }
    }
}
```
必须为ByteArrayInputStream提供字节数组，为了产生该数组String包含了一个可以实现此项工作的getBytes()方法。所产生的ByteArrayInputStream是一个适合传递给DataInputStream的InputStream。
从DataInputStream用readByte()一次一个字节地读取字符，那么任何字节的值都是合法结果，因此返回值不能用来检测输入是否结束。
```
public class TestEOF {
    public static void main(String[] args) throws IOException {
        DataInputStream in = new DataInputStream(new BufferedInputStream(new FileInputStream("TestEOF.java")));
        // 使用available()方法查询还有多少个可供存取的字符
        while (in.available() != 0) {
            System.out.print((char) in.readByte());
        }
    }
}
```
available()字面意思：在没有阻塞情况下所能读取的字节数。对于文件，这意味着整个文件，但是对于不同类型的流，可能就不是这样，因此要谨慎使用。

####基本的文件输出
FileWriter对象可以向文件写入数据。通常会用BufferedWriter将其包装起来用以缓冲输出。
```
public class BasicFileOutput {
    static String file = "BasicFileOutput.out";

    public static void main(String[] args) throws IOException {
        BufferedReader in = new BufferedReader(new StringReader(
                BufferedInputFile.read("BasicFileOutput.java")));
        PrintWriter out = new PrintWriter(new BufferedWriter(new FileWriter(file)));
        int lineCount = 1;
        String s;
        while ((s = in.readLine()) != null) {
            out.println(lineCount++ + ": " + s);
        }
        out.close();

        System.out.println(BufferedInputFile.read(file));
    }
}/*加入行号的输出*/
```
出现行号，并未用到LineNumberInputStream，因为这个类没有多大帮助，所以没必要用它。
一旦读完输入数据流，readLine()会返回null。要为out显式调用close()。如果不为所有的输出文件调用close()，就会发现缓冲区内容不会被刷新清空，那么他们也就不会完整。

####文本文件输出的快捷方式
Java SE5在PrintWriter中添加了一个辅助构造器，是的不必每次创建文本并向其中写入时，都去执行所有的装饰工作，BasicFileOutput.java：
```
public class FileOutputShortcut {
    static String file = "FileOutputShortcut.out";

    public static void main(String[] args) throws IOException {
        BufferedReader in = new BufferedReader(new StringReader(
                BufferedInputFile.read("FileOutputShortcut.java")));
        PrintWriter out = new PrintWriter(file);
        int lineCount = 1;
        String s;
        while ((s = in.readLine()) != null) {
            out.println(lineCount++ + ": " + s);
        }
        out.close();
        System.out.println(BufferedInputFile.read(file));
    }
}
```
仍旧是在进行缓存，只是不必自己去实现。遗憾的是，其他常见的写入任务都没有快捷方式，因此典型的IO仍旧包含大量的冗余文本。

####存储和恢复数据
PrintWriter可以对数据进行格式化，以便阅读。但是为了输出可供另一个流恢复的数据，需要用DataOutputStream写入数据，并用DataInputStream恢复数据。当然，这些流可以使用任何形式，下面示例使用的是一个文件，并且对于读和写进行了缓冲处理。注意DataOutputStream和DataInputStream是面向字节的，因此要使用InputStream和OutputStream。
```
public class StoringAndRecoveringData {
  public static void main(String[] args) throws IOException {
    DataOutputStream out = new DataOutputStream(
      new BufferedOutputStream(
        new FileOutputStream("Data.txt")));
    out.writeDouble(3.14159);
    out.writeUTF("That was pi");
    out.writeDouble(1.41413);
    out.writeUTF("Square root of 2");
    out.close();
    DataInputStream in = new DataInputStream(
      new BufferedInputStream(
        new FileInputStream("Data.txt")));
    System.out.println(in.readDouble());
    System.out.println(in.readUTF());
    System.out.println(in.readDouble());
    System.out.println(in.readUTF());
  }
} /*
3.14159
That was pi
1.41413
Square root of 2
*/
```
如果使用DataOutputStream写入数据，Java保证可以使用DataInputStream准确地读取数据——无论读和写数据的平台多么不同，只要两个平台都有Java。
当使用DataOutputStream时，写字符串并且让DataInputStream能够恢复它的唯一可靠做法是使用UTF-8编码，示例中是使用wrtteUTF()和readUTF()来实现的。UTF-8是一种多字节格式，其编码长度根据实际使用的字符集会有所变化。如果只是ASCII或者几乎都是ASCII字符(只占7位)，那么就显得及其浪费空间和带宽，所以UTF-8将ASCII字符编码成单一字节的形式，而非ASCII字符则编码成两到三字节的形式。另外，字符串的长度存储在UTF-8字符串的前两个字节中。但是，writeUTF()和readUTF()使用的适合于java的UTF-8变体。因此如果用一个非Java程序读取用writeUTF()所写的字符串时，必须编写一些特殊代码才能正确读取字符串。
writeDouble()将double类型的数字存储到流中，并用相应的readDouble()恢复它（对于其他数据类型，也有类似方法）。但是为了保证所有的读方法都能正常工作，必须知道流中数据项所在的确切位置。因此，必须：要么为文件中的数据采用固定的格式；要么将额外的信息保存到文件中，以便能够对其进行解析以确定数据的存放位置。注意，对象序列化和XML可能是更容易的存储和读取复杂数据结构的方式。

####读写随机访问文件
在使用RandomAccessFile时，必须知道文件排版，这样才能正确地操作它。RandomAccessFile拥有读取基本数据和UTF-8字符串的各种具体方法：
```
public class UsingRandomAccessFile {
    static String file = "rtest.dat";

    static void display() throws IOException {
        RandomAccessFile rf = new RandomAccessFile(file, "r");
        for (int i = 0; i < 7; i++) {
            System.out.println("Value " + i + ": " + rf.readDouble());
        }
        System.out.println(rf.readUTF());
        rf.close();
    }

    public static void main(String[] args) throws IOException {
        RandomAccessFile rf = new RandomAccessFile(file, "rw");
        for (int i = 0; i < 7; i++) {
            rf.writeDouble(i * 1.414);
        }
        rf.writeUTF("The end of the file");
        rf.close();
        display();
        rf = new RandomAccessFile(file, "rw");
        rf.seek(5 * 8);
        rf.writeDouble(47.0001);
        rf.close();
        display();
    }
}/*
Value 0: 0.0
Value 1: 1.414
Value 2: 2.828
Value 3: 4.242
Value 4: 5.656
Value 5: 7.069999999999999
Value 6: 8.484
The end of the file
Value 0: 0.0
Value 1: 1.414
Value 2: 2.828
Value 3: 4.242
Value 4: 5.656
Value 5: 47.0001
Value 6: 8.484
The end of the file
*/
```
因为double总是8字节长，所以为了用seek()查找第5个双精度值，只需用5*8来产生查找位置。RandomAccessFile除了实现DataInput和DataOutput接口之外，有效地与IO继承层次结构的其他部分实现了分离。因为它不支持装饰，所以不能将其与InputStream及OutputStream子类的任何部分组合起来。必须假定RandomAccessFile已经被正确缓冲，因为不能为他添加这样的功能。
可以自行选择的是第二个构造器参数：只读  r   或读写  rw。

##文件读写的使用工具
读取文件到内存，修改，然后在写入，下面TextFile类简化对文件的读写操作，用一个ArrayList来保存文件的若干行：
```
public class TextFile extends ArrayList<String> {
    public static String read(String fileName) {
        StringBuilder sb = new StringBuilder();

        try {
            BufferedReader in = new BufferedReader(new FileReader((new File(fileName)).getAbsoluteFile()));

            String s;
            try {
                while((s = in.readLine()) != null) {
                    sb.append(s);
                    sb.append("\n");
                }
            } finally {
                in.close();
            }
        } catch (IOException var8) {
            throw new RuntimeException(var8);
        }

        return sb.toString();
    }

    public static void write(String fileName, String text) {
        try {
            PrintWriter out = new PrintWriter((new File(fileName)).getAbsoluteFile());

            try {
                out.print(text);
            } finally {
                out.close();
            }

        } catch (IOException var7) {
            throw new RuntimeException(var7);
        }
    }

    public TextFile(String fileName, String splitter) {
        super(Arrays.asList(read(fileName).split(splitter)));
        if (((String)this.get(0)).equals("")) {
            this.remove(0);
        }

    }

    public TextFile(String fileName) {
        this(fileName, "\n");
    }

    public void write(String fileName) {
        try {
            PrintWriter out = new PrintWriter((new File(fileName)).getAbsoluteFile());

            try {
                Iterator var4 = this.iterator();

                while(var4.hasNext()) {
                    String item = (String)var4.next();
                    out.println(item);
                }
            } finally {
                out.close();
            }

        } catch (IOException var9) {
            throw new RuntimeException(var9);
        }
    }

    public static void main(String[] args) {
        String file = read("TextFile.java");
        write("test.txt", file);
        TextFile text = new TextFile("test.txt");
        text.write("test2.txt");
        TreeSet<String> words = new TreeSet(new TextFile("TextFile.java", "\\W+"));
        System.out.println(words.headSet("a"));
    }
}
```
在任何打开文件的代码在finally子句中，作为防卫措施都添加了对文件的close()调用，以保证文件将会被正确关闭。
这个构造器利用read()方法将文件转换成字符串，接着使用String.split()以换行符为界把结果划分成行。非静态write()方法必须一行一行地输出这些行。因为这个类希望将读取和写入文件的过程简化，因此所有IOException都被转型为RuntimeException，因此用户不必使用try-catch语句块。
另外一个读取文件的方法是使用Java SE5引入的java.util.Scanner类。

####读取二进制文件
```
public class BinaryFile {
    public BinaryFile() {
    }

    public static byte[] read(File bFile) throws IOException {
        BufferedInputStream bf = new BufferedInputStream(new FileInputStream(bFile));

        byte[] var4;
        try {
            byte[] data = new byte[bf.available()];
            bf.read(data);
            var4 = data;
        } finally {
            bf.close();
        }

        return var4;
    }

    public static byte[] read(String bFile) throws IOException {
        return read((new File(bFile)).getAbsoluteFile());
    }
}
```
一个重载方法接受File参数，第二个重载方法接受表示文件名的String 参数。都返回产生byte数组。
available()方法被用来产生恰当的数组尺寸，并且read()方法的特定重载版本填充了这个数组。

##标准IO
标准IO源自于Unix的“程序所使用的单一信息流”这一概念。程序的所有输入都可以来自于标准输入，所有输出都可以发送到标准输出。

####从标准输入中读取
标准输入：System.in未加工的InputStream
标准输出：System.out  PrintStream对象
标准错误：System.err  PrintStream对象

```
public class Echo {
    public static void main(String[] args) throws IOException {
        BufferedReader stdin = new BufferedReader(new InputStreamReader(System.in));
        String s;
        while ((s = stdin.readLine()) != null && s.length() != 0) {
            System.out.println(s);
        }
    }
}
```
通常会用readLine()一次一行读取输入，将System.in包装城BufferedReader来使用，这要求必须用InputStreamReader把Sytem.in转换成Reader。System,in通常应该对它进行缓冲。

####将System.out转换成PrintWriter
System.out是一个PrintStream，而PrintStream是一个OutputStream。PrintWriter可以接受OutputStream做为参数的构造器。
```
public class ChangeSystemOut {
  public static void main(String[] args) {
    PrintWriter out = new PrintWriter(System.out, true);
    out.println("Hello, world");
  }
} /*
Hello, world
*/
```
第二个参数true，以便开启自动清空功能；否则看不到输出。

####标准IO重定向
Java的System类提供了静态方法嗲用，以允许对标准输入输出和错误IO流进行重定向：
setIn(InputStream)
setOut(PrintStream)
setErr(PrintStream)

```
public class Redirecting {
    public static void main(String[] args) throws IOException {
        PrintStream console = System.out;
        BufferedInputStream in = new BufferedInputStream(new FileInputStream("Redirecting.java"));
        PrintStream out = new PrintStream(
                new BufferedOutputStream(new FileOutputStream("test.out")));
        System.setIn(in);
        System.setOut(out);
        System.setErr(out);
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        String s;
        while ((s = br.readLine()) != null) {
            System.out.println(s);
        }
        out.close();
        System.setOut(console);
    }
}
```
程序将标准输入附接到文件上，并将标准输出和标准错误重定向到另一个文件。
IO重定向操作的是字节流，而不是字符流。

##进程控制
Java内部执行其他操作系统的程序，并且控制这些程序输入输出，Java类库提供了执行这些操作的类。
运行程序输出到控制台，本节包含了可以简化这项任务的工具，工具会产生两种类型的错误：普通的导致异常的错误以及进程自身执行过程中产生的错误，用单独的异常来报告这些错误：
```
public class OSExecuteException extends RuntimeException {
    public OSExecuteException(String why) {
        super(why);
    }
}
```
想运行一个程序，向OSException.command()传递一个command字符串，它与在控制台上运行该程序所键入的命令相同。这个命令被传递给java,lang.ProcessBuilder构造器，然后产生的ProcessBuilder对象被启动：
```
public class OSExecute {
    public OSExecute() {
    }

    public static void command(String command) {
        boolean err = false;

        try {
            Process e = (new ProcessBuilder(command.split(" "))).start();
            BufferedReader results = new BufferedReader(new InputStreamReader(e.getInputStream()));

            String s;
            while((s = results.readLine()) != null) {
                System.out.println(s);
            }

            for(BufferedReader errors = new BufferedReader(new InputStreamReader(e.getErrorStream())); (s = errors.readLine()) != null; err = true) {
                System.err.println(s);
            }
        } catch (Exception var6) {
            if(command.startsWith("CMD /C")) {
                throw new RuntimeException(var6);
            }

            command("CMD /C " + command);
        }

        if(err) {
            throw new OSExecuteException("Errors executing " + command);
        }
    }
}
```
为了捕获程序执行产生的标准输出流，需要调用getInputStream()，这是因为InputStream可以从中读取信息的流。该程序的错误被发送到了标准错误流，并通过调用getErrorStream()得以捕获。如果存在任何错误，它们都会被打印并且会抛出OSException，因此调用程序需要处理这个问题。
```
public class OSExecuteDemo {
  public static void main(String[] args) {
    OSExecute.command("javap OSExecuteDemo");
  }
} /*
Compiled from "OSExecuteDemo.java"
public class OSExecuteDemo extends java.lang.Object{
    public OSExecuteDemo();
    public static void main(java.lang.String[]);
}
*/
```
这里使用了javap反编译器(随JDK发布)来反编译该程序。
