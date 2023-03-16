##压缩
Java IO类库中的类支持读写压缩格式的数据流。
这些类属于InputStream和OutputStream继承层次结构的一部分。因为压缩类库是按字节方式而不是字符方式处理的。
![这里写图片描述](https://img-blog.csdn.net/20180521144727159?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
尽管存在多种压缩算法，但是Zip和GZIP是最常用的。

####用GZIP进行简单压缩
如果相对单一数据流进行压缩，GZIP接口是比较合适的选择：
```
// {Args: GZIPcompress.java}
public class GZIPcompress {
    public static void main(String[] args) throws IOException {
        if (args.length == 0) {
            System.out.println(
                    "Usage: \nGZIPcompress file\n" +
                            "\tUses GZIP compression to compress " +
                            "the file to test.gz");
            System.exit(1);
        }
        BufferedReader in = new BufferedReader(
                new FileReader(args[0]));
        BufferedOutputStream out = new BufferedOutputStream(
                new GZIPOutputStream(
                        new FileOutputStream("test.gz")));
        System.out.println("Writing file");
        int c;
        while ((c = in.read()) != -1) {
            out.write(c);
        }
        in.close();
        out.close();
        System.out.println("Reading file");
        BufferedReader in2 = new BufferedReader(
                new InputStreamReader(new GZIPInputStream(
                        new FileInputStream("test.gz"))));
        String s;
        while ((s = in2.readLine()) != null) {
            System.out.println(s);
        }
    }
}
```
压缩类直接将输出流封装成GZIPOutputStream或ZipOutputStream，并将输入流封装成GZIPInputStream或ZipInputStream即可。

####用Zip进行多文件保存
支持Zip格式的Java库更加全面，它显示了用Checksum类来计算和校验文件的校验和的方法。一共有两种Checksum类型：Adler32（快）和CRC32（慢，准确）。
```
// {Args: ZipCompress.java}
public class ZipCompress {
    public static void main(String[] args)
            throws IOException {
        FileOutputStream f = new FileOutputStream("test.zip");
        CheckedOutputStream csum =
                new CheckedOutputStream(f, new Adler32());
        ZipOutputStream zos = new ZipOutputStream(csum);
        BufferedOutputStream out =
                new BufferedOutputStream(zos);
        zos.setComment("A test of Java Zipping");
        // No corresponding getComment(), though.
        for (String arg : args) {
            print("Writing file " + arg);
            BufferedReader in =
                    new BufferedReader(new FileReader(arg));
            zos.putNextEntry(new ZipEntry(arg));
            int c;
            while ((c = in.read()) != -1) {
                out.write(c);
            }
            in.close();
            out.flush();
        }
        out.close();
        // Checksum valid only after the file has been closed!
        print("Checksum: " + csum.getChecksum().getValue());
        // Now extract the files:
        print("Reading file");
        FileInputStream fi = new FileInputStream("test.zip");
        CheckedInputStream csumi =
                new CheckedInputStream(fi, new Adler32());
        ZipInputStream in2 = new ZipInputStream(csumi);
        BufferedInputStream bis = new BufferedInputStream(in2);
        ZipEntry ze;
        while ((ze = in2.getNextEntry()) != null) {
            print("Reading file " + ze);
            int x;
            while ((x = bis.read()) != -1) {
                System.out.write(x);
            }
        }
        if (args.length == 1) {
            print("Checksum: " + csumi.getChecksum().getValue());
        }
        bis.close();
        // Alternative way to open and read Zip files:
        ZipFile zf = new ZipFile("test.zip");
        Enumeration e = zf.entries();
        while (e.hasMoreElements()) {
            ZipEntry ze2 = (ZipEntry) e.nextElement();
            print("File: " + ze2);
            // ... and extract the data as before
        }
    /* if(args.length == 1) */
    }
}
```
对于每一个要加入压缩档案的文件，都必须调用putNextEntry()，并将其传递给一个ZipEntry对象。ZipEntry包含一个功能很广泛的接口，允许获取和设置Zip文件内该特定项上所有可利用的数据：名字、压缩的和未压缩的文件大小、日期、CRC检验和、额外字段数据、注释、压缩方法以及他是否是一个目录入口等等。
尽管Zip格式提供设置密码的方法，但Java的Zip类库并不提供支持。虽然CheckedInputStream和CheckedOutputStream都支持Adler32和CRC32两种类型的校验和，但是ZipEntry类只有一个支持CRC的接口。这是底层Zip格式的限制，但却限制了人们不能使用数度更快的Adler32。
为了能够解压缩文件，ZipInputStream提供了一个getNextEntry()方法返回下一个ZipEntry。利用ZipFile对象读取文件。该对象有一个entries()方法来想ZipEntries返回一个Enumeration(枚举)。
为了读取校验和，必须拥有对与之相关联的Checksum对象的访问权限。在这里保留了指向CheckedOutputStream和CheckedInputStream对象的引用。但是，也可以只保留一个指向Checksum对象的引用。
Zip流中setComment()，可以在文件时写注释，但却没有任何方法恢复ZipInputStream内的注释。似乎只能通过ZipEntry，带能以逐条方式完全支持注释的获取。
GZIP或Zip库的使用不仅仅局限于文件——可以压缩任何东西，包括网络发送的数据。

####Java档案文件
Zip格式也被应用于JAR(Java ARchive，Java档案文件)文件格式中。将一组文件压缩到单个压缩文件中。同Java中任何其他东西一样，JAR也是跨平台的。由于采用压缩技术，可以使传输时间更短，只需向服务器发送一次请求即可。
Sun的JDK自带jar程序，可根据选择自动压缩文件：
jar [options] destination [manifest] inputfile(s)
其中options只是一个字母几个：
![这里写图片描述](https://img-blog.csdn.net/20180521153007630?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

创建一个名为myJarFile.jar的JAR文件，包含当前目录下所有类文件，以及自动产生的清单文件：
jar cf myJarFile.jar *.class
下面的命令与前例类似，但添加了一个名为myManifestFile.mf的用户自建清单文件:
jar cmf myJarFile.jar myHanifestFile.nf  *.class
下面的命令会产生myJarFile.jar内所有文件的一个目录表:
jar tf myJarFile.jar
下面的命令添加“v”(详尽)标志，可以提供有关myJarFle.jar中的文件的更详细的信息:
jar. tvf myJarfile.jar
假定audio、classes和image是子目录， 下面的命令将所有子目录合并到文件myApp.jar中,其中也包括了“v” 标志。当jar程序运行时，该标志可以提供更详细的信息:
jar cvf myApp.jar audio classes image
如果用0 (零)选项创建一个JAR文件，那么该文件就可放入类路径变量(CLASSPATH) 中:
CLASSPATH="lib1.jar;lib2.jar:"
然后Java就可以在1lib1.jar和lib2.jar中搜索目标类文件了。
jar工具的功能没有zip工具那么强大。例如，不能够对已有的JAR文件进行添加或更新文件的操作，只能从头创建一个JAR文件。同时，也不能将文件移动至-一个JAR文件，并在移动后将它们删除。然而，在一种平台.上创建的JAR文件可以被在其他任何平台上的jar工具透明地阅读(这个问题有时会困扰zip工具)。

##对象序列化
如果对象能够在程序不运行的情况下仍能存在并保存其信息，那将非常有用。这样，在下次运行程序时，该对象将被重建并且拥有的信息与在程序上次运行时它所拥有的信息相同。如果能够将一个对象声明为是“持久性”的，并为处理掉所有细节，那将会显得十分方便。
Java的<font color=red>对象序列化</font>将那些实现了Serializable接口的对象转换成一个字节序列，并能够在以后将这个字节序列完全恢复为原来的对象。这一过程甚至可通过网络进行;这意味着序列化机制能自动弥补不同操作系统之间的差异。也就是说，可以在运行Windows系统的计算机上创建一个对象，将其序列化，通过网络将它发送给一台运行Unix系统的计算机，然后在那里谁确地重新组装，而却不必担心数据在不同机器上的表示会不同，也不必关心字节的顺序或者其他任何细节。
利用序列化可以实现轻量级持久性(lightweight persistence)。“持久性”意味着一个对象的生存周期并不取决于程序是否正在执行;它可以生存于程序的调用之间。
 对象序列化的概念加入到语言中是为了支持两种主要特性。
 一是Java的远程方法调用(Remote Method Invocation, RMI)，它使存活于其他计算机上的对象使用起来就像是存活于本机上一样。
再者，对Java Beans来说，对象的序列化也是必需的。使用一个Bean时，一般情况下是在设计阶段对它的状态信息进行配置。这种状态信息必须保存下来，并在程序启动时进行后期恢复，这种具体工作就是由对象序列化完成的。
只要对象实现了Serializable接口(该接口仅是一个标记接口，不包括任何方法)，对象的序列化处理就会非常简单。当序列化的概念被加入到语言中时，许多标准库类都发生了改变，以便具备序列化特性一其中包括所有基 本数据类型的封装器、所有容器类以及许多其他的东西。甚至Class对象也可以被序列化。
要序列化一个对象，首先要创建某些OutputStream对象，然后将其封装在一个ObjectOutputStream对象内。这时，只需调用writeObject0即可将对象序列化，并将其发送给OutputStream(对象化序列是基于字节的，因要使用InputStream和OutputStream继承层次结构)。 要反向进行该过程(即将-一个序列还原为一个对象)，需要将个InputStream封装在ObjectmputStream内，然后调用readObject()。和往常一样，我们最后获得的是一个引用，它指向一个向上转型的Object,所以必须向下转型才能直接设置它们。

通过对链接的对象生成worm对序列化机制进行了测试。每个对象都与worm中的下一段链接，同时又与属于不同类的对象引用数组链接：
```
class Data implements Serializable {
    private int n;

    public Data(int n) {
        this.n = n;
    }

    @Override
    public String toString() {
        return Integer.toString(n);
    }
}

public class Worm implements Serializable {
    private static Random rand = new Random(47);
    private Data[] d = {
            new Data(rand.nextInt(10)),
            new Data(rand.nextInt(10)),
            new Data(rand.nextInt(10))
    };
    private Worm next;
    private char c;

    // Value of i == number of segments
    public Worm(int i, char x) {
        print("Worm constructor: " + i);
        c = x;
        if (--i > 0) {
            next = new Worm(i, (char) (x + 1));
        }
    }

    public Worm() {
        print("Default constructor");
    }

    @Override
    public String toString() {
        StringBuilder result = new StringBuilder(":");
        result.append(c);
        result.append("(");
        for (Data dat : d) {
            result.append(dat);
        }
        result.append(")");
        if (next != null) {
            result.append(next);
        }
        return result.toString();
    }

    public static void main(String[] args)
            throws ClassNotFoundException, IOException {
        Worm w = new Worm(6, 'a');
        print("w = " + w);
        ObjectOutputStream out = new ObjectOutputStream(
                new FileOutputStream("worm.out"));
        out.writeObject("Worm storage\n");
        out.writeObject(w);
        out.close(); // Also flushes output
        ObjectInputStream in = new ObjectInputStream(
                new FileInputStream("worm.out"));
        String s = (String) in.readObject();
        Worm w2 = (Worm) in.readObject();
        print(s + "w2 = " + w2);
        ByteArrayOutputStream bout =
                new ByteArrayOutputStream();
        ObjectOutputStream out2 = new ObjectOutputStream(bout);
        out2.writeObject("Worm storage\n");
        out2.writeObject(w);
        out2.flush();
        ObjectInputStream in2 = new ObjectInputStream(
                new ByteArrayInputStream(bout.toByteArray()));
        s = (String) in2.readObject();
        Worm w3 = (Worm) in2.readObject();
        print(s + "w3 = " + w3);
    }
} /*
Worm constructor: 6
Worm constructor: 5
Worm constructor: 4
Worm constructor: 3
Worm constructor: 2
Worm constructor: 1
w = :a(853):b(119):c(802):d(788):e(199):f(881)
Worm storage
w2 = :a(853):b(119):c(802):d(788):e(199):f(881)
Worm storage
w3 = :a(853):b(119):c(802):d(788):e(199):f(881)
*/
```
Worm内的Data对象数组是用随机数初始化的。每个Worm段都用一个char加以标记。该char是在递归生成链接的Worm列表时自动产生的。要创建一个Worm，必须告诉构造器所希望的它的长度。在产生下一个引用时，要调用Worm构造器，并将长度减1，以此类推。最后一个next引用则为null，表示已到达Worm尾部。
以上操作使事情变得更加复杂，从而加大了对象序列化的难度。真正的序列化过程非常简单。一旦从另外某个流创建了ObjectOutputStream，writeObject()就会将对象序列化。
从输出上看，被还原后的对象确实包含了原对象中的所有链接。
在对一个Serializable对象进行还原的过程中，没有调用任何构造器，包含默认的构造器。整个对象都是通过InputStream中取得数据恢复过来的。

####寻找类
将一个对象从它的序列化状态中恢复出来，哪些工作是必须的？
```
public class Alien implements Serializable {}
```
用于创建和序列化一个Alien对象的文件位于相同目录下：
```
public class FreezeAlien {
    public static void main(String[] args) throws Exception {
        ObjectOutput out = new ObjectOutputStream(new FileOutputStream("X.file"));
        Alien quellek = new Alien();
        out.writeObject(quellek);
    }
}
public class ThawAlien {
  public static void main(String[] args) throws Exception {
    ObjectInputStream in = new ObjectInputStream(new FileInputStream(new File("X.file")));
    Object mystery = in.readObject();
    System.out.println(mystery.getClass());
  }
} /*
class io.Alien
*/
```
打开和读取mystery对象中的内容都需要Alien的Class对象；Java虚拟机找不到Alien.class（除非它正好在类路径Classpath内，而本例不在类路径之内）。这样就会得到一个名叫ClassNotFoundException的异常（同样，除非能够验证Alien存在，否则它等于消失）。必须保证Java虚拟机能够找到相关的.class文件。

####序列化的控制
默认序列化机制并不难操纵。如果希望部分序列化或子对象不必序列化。可通过Exterbalizable接口代替Serializable。Exterbalizable接口继承了Serializable，同时增添了两个方法：writeExternal()和readExternal()。这两个方法会在序列化和反序列化还原的过程中被自动调用。
```
class Blip1 implements Externalizable {
  public Blip1() {
    print("Blip1 Constructor");
  }
  @Override
  public void writeExternal(ObjectOutput out) throws IOException {
    print("Blip1.writeExternal");
  }
  @Override
  public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
    print("Blip1.readExternal");
  }
}

class Blip2 implements Externalizable {
  Blip2() {
    print("Blip2 Constructor");
  }
  @Override
  public void writeExternal(ObjectOutput out) throws IOException {
    print("Blip2.writeExternal");
  }
  @Override
  public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
    print("Blip2.readExternal");
  }
}

public class Blips {
  public static void main(String[] args)
  throws IOException, ClassNotFoundException {
    print("Constructing objects:");
    Blip1 b1 = new Blip1();
    Blip2 b2 = new Blip2();
    ObjectOutputStream o = new ObjectOutputStream(
      new FileOutputStream("Blips.out"));
    print("Saving objects:");
    o.writeObject(b1);
    o.writeObject(b2);
    o.close();
    // Now get them back:
    ObjectInputStream in = new ObjectInputStream(
      new FileInputStream("Blips.out"));
    print("Recovering b1:");
    b1 = (Blip1)in.readObject();
    // OOPS! Throws an exception:
//! print("Recovering b2:");
//! b2 = (Blip2)in.readObject();
  }
} /*
Constructing objects:
Blip1 Constructor
Blip2 Constructor
Saving objects:
Blip1.writeExternal
Blip2.writeExternal
Recovering b1:
Blip1 Constructor
Blip1.readExternal
*/
```
Blip1和Blip2区别在Blip1的构造器是公共的。这样在恢复Blip2时会造成异常。将Blip2变为public，然后删除//!注释标记，可得到正确结果。
恢复b1后，会调用Blip1默认构造器。这与恢复一个Serializable对象不同。对于Serializable对象，对象完全以它存储的二进制位为基础来构造，而不是调用构造器。而对于一个Externalizable对象，所有普通的默认构造器都会被调用（包括在字段定义时的初始化），然后调用readExternal()。
```
public class Blip3 implements Externalizable {
    private int i;
    private String s; // No initialization

    public Blip3() {
        print("Blip3 Constructor");
        // s, i not initialized
    }

    public Blip3(String x, int a) {
        print("Blip3(String x, int a)");
        s = x;
        i = a;
        // s & i initialized only in non-default constructor.
    }

    @Override
    public String toString() {
        return s + i;
    }

    @Override
    public void writeExternal(ObjectOutput out)
            throws IOException {
        print("Blip3.writeExternal");
        // You must do this:
        out.writeObject(s);
        out.writeInt(i);
    }

    @Override
    public void readExternal(ObjectInput in)
            throws IOException, ClassNotFoundException {
        print("Blip3.readExternal");
        // You must do this:
        s = (String) in.readObject();
        i = in.readInt();
    }

    public static void main(String[] args) throws IOException, ClassNotFoundException {
        print("Constructing objects:");
        Blip3 b3 = new Blip3("A String ", 47);
        print(b3);
        ObjectOutputStream o = new ObjectOutputStream(
                new FileOutputStream("Blip3.out"));
        print("Saving object:");
        o.writeObject(b3);
        o.close();
        // Now get it back:
        ObjectInputStream in = new ObjectInputStream(new FileInputStream("Blip3.out"));
        print("Recovering b3:");
        b3 = (Blip3) in.readObject();
        print(b3);
    }
} /*
Constructing objects:
Blip3(String x, int a)
A String 47
Saving object:
Blip3.writeExternal
Recovering b3:
Blip3 Constructor
Blip3.readExternal
A String 47
*/
```
s和i不在默认构造器初始化。意味着如果不在readExternal()中初始化s和i，s就是null，i=0。如果注释掉跟随与“You nust do this”后面的两行代码，然后运行会发现还原后的对象s是null，i=0。
如果从一个Externalizable对象继承，通常需要调用基类版本的writeExternal()和readExternal()来为基类组件提供恰当的存储和恢复功能。为正常运行，不仅需要在writeExternal()方法（没有任何默认行为来为Externalizable 对象写入任何成员对象）中将来自对象的重要信息写入，还必须在readExternal()方法中恢复数据。因为Externalizable对象的默认构造器行为使其看起来似乎像某种自动发生的存储与恢复操作。但实际上并非如此。

#####transient（瞬时）关键字
特定子对象不想让java的序列化自动保存与恢复，可以使用transient逐个字段地关闭序列化。
```
public class Logon implements Serializable {
    private Date date = new Date();
    private String username;
    private transient String password;

    public Logon(String name, String pwd) {
        username = name;
        password = pwd;
    }

    @Override
    public String toString() {
        return "logon info: \n   username: " + username +
                "\n   date: " + date + "\n   password: " + password;
    }

    public static void main(String[] args) throws Exception {
        Logon a = new Logon("Hulk", "myLittlePony");
        print("logon a = " + a);
        ObjectOutputStream o = new ObjectOutputStream(
                new FileOutputStream("Logon.out"));
        o.writeObject(a);
        o.close();
        TimeUnit.SECONDS.sleep(1); // Delay
        // Now get them back:
        ObjectInputStream in = new ObjectInputStream(
                new FileInputStream("Logon.out"));
        print("Recovering object at " + new Date());
        a = (Logon) in.readObject();
        print("logon a = " + a);
    }
} /*
logon a = logon info:
   username: Hulk
   date: Sat Nov 19 15:03:26 MST 2005
   password: myLittlePony
Recovering object at Sat Nov 19 15:03:28 MST 2005
logon a = logon info:
   username: Hulk
   date: Sat Nov 19 15:03:26 MST 2005
   password: null
*/
```
当对象被恢复时，transient的password域就会变成null。虽然toString()是用重载后的+运算符来连接String对象，但是null引用会被自动转换成字符串null。
由于Externalizable对象在默认情况下不保存任何字段，所以teansient关键字只能和Serializable对象一起使用。

#####Externalizable的替代方法
可以实现Serializable接口，并添加（非覆盖或实现）名为writeObject()和readObject()方法。这样一旦对象被序列化或者被反序列化还原，就会自动地分别调用这个方法。也就是说，只要提供这两个方法，就会使用它们而不是默认的序列化机制。
方法特征签名：
```
private void writeObject(ObjectOutputStream stream)throws IOException;
private void readObject(ObjectInputStream stream) throws IOException, ClassNotFoundException;
```
实际上并不是从这个类的其他方法中调用它们，而是ObjectOutputStream和ObjectInputStream对象的writeObject()和readObject()方法调用了对象的writeObject()和readObject()方法（类型信息章节展示了如何在类的外部访问private方法）。
接口中的所有东西自动定义为了public的，如果这两个方法必须是private的，那它们不会是接口的一部分。
在调用ObjectOutputStream.writeObject()时，会检查所传递的Serializable对象，看看是否实现了它自己的writeObject()。如果是，就跳过正常的序列化过程并调用writeObject()。
还有另一个技巧，可在自己的writeObject()内部，调用defaultWriteObject()来选择执行默认的writeObject()：
```
public class SerialCtl implements Serializable {
    private String a;
    private transient String b;

    public SerialCtl(String aa, String bb) {
        a = "Not Transient: " + aa;
        b = "Transient: " + bb;
    }

    @Override
    public String toString() {
        return a + "\n" + b;
    }

    private void writeObject(ObjectOutputStream stream)
            throws IOException {
        stream.defaultWriteObject();
        stream.writeObject(b);
    }

    private void readObject(ObjectInputStream stream) throws IOException, ClassNotFoundException {
        stream.defaultReadObject();
        b = (String) stream.readObject();
    }

    public static void main(String[] args) throws IOException, ClassNotFoundException {
        SerialCtl sc = new SerialCtl("Test1", "Test2");
        System.out.println("Before:\n" + sc);
        ByteArrayOutputStream buf = new ByteArrayOutputStream();
        ObjectOutputStream o = new ObjectOutputStream(buf);
        o.writeObject(sc);
        // Now get it back:
        ObjectInputStream in = new ObjectInputStream(
                new ByteArrayInputStream(buf.toByteArray()));
        SerialCtl sc2 = (SerialCtl) in.readObject();
        System.out.println("After:\n" + sc2);
    }
} /*
Before:
Not Transient: Test1
Transient: Test2
After:
Not Transient: Test1
Transient: Test2
*/
```
非transient字段由defaultWriteObject()方法保存，而transient字段必须在程序中明确保存和恢复。字段是在构造器内部而不是在定义外进行初始化的，以此可以证实他们在反序列化还原期间没有被一些自动化机制初始化。
如果打算使用默认机制写入对象的非transient部分，那么就必须调用defaultWriteObject()作为writeObject()中的第一个操作。并让defaultReadObject()作为readObject()中的第一个。
writeObject()方法必须检查sc，判断是否拥有自己的writeObject()方法（不是检查接口——这里根本就没有接口，也不是检查类的类型，而是利用反射来真正搜索方法）。

####使用“持久性”
```
class House implements Serializable {
}

class Animal implements Serializable {
    private String name;
    private House preferredHouse;

    Animal(String nm, House h) {
        name = nm;
        preferredHouse = h;
    }

    @Override
    public String toString() {
        return name + "[" + super.toString() +
                "], " + preferredHouse + "\n";
    }
}

public class MyWorld {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        House house = new House();
        List<Animal> animals = new ArrayList<Animal>();
        animals.add(new Animal("Bosco the dog", house));
        animals.add(new Animal("Ralph the hamster", house));
        animals.add(new Animal("Molly the cat", house));
        print("animals: " + animals);
        ByteArrayOutputStream buf1 = new ByteArrayOutputStream();
        ObjectOutputStream o1 = new ObjectOutputStream(buf1);
        o1.writeObject(animals);
        o1.writeObject(animals); // Write a 2nd set
        // Write to a different stream:
        ByteArrayOutputStream buf2 = new ByteArrayOutputStream();
        ObjectOutputStream o2 = new ObjectOutputStream(buf2);
        o2.writeObject(animals);
        // Now get them back:
        ObjectInputStream in1 = new ObjectInputStream(
                new ByteArrayInputStream(buf1.toByteArray()));
        ObjectInputStream in2 = new ObjectInputStream(
                new ByteArrayInputStream(buf2.toByteArray()));
        List
                animals1 = (List) in1.readObject(),
                animals2 = (List) in1.readObject(),
                animals3 = (List) in2.readObject();
        print("animals1: " + animals1);
        print("animals2: " + animals2);
        print("animals3: " + animals3);
    }
} /*
animals: [Bosco the dog[Animal@addbf1], House@42e816
, Ralph the hamster[Animal@9304b1], House@42e816
, Molly the cat[Animal@190d11], House@42e816
]
animals1: [Bosco the dog[Animal@de6f34], House@156ee8e
, Ralph the hamster[Animal@47b480], House@156ee8e
, Molly the cat[Animal@19b49e6], House@156ee8e
]
animals2: [Bosco the dog[Animal@de6f34], House@156ee8e
, Ralph the hamster[Animal@47b480], House@156ee8e
, Molly the cat[Animal@19b49e6], House@156ee8e
]
animals3: [Bosco the dog[Animal@10d448], House@e0e1c6
, Ralph the hamster[Animal@6ca1c], House@e0e1c6
, Molly the cat[Animal@1bf216a], House@e0e1c6
]
*/
```
创建一个Animal列表并将其两次序列化，分别送至不同的流。当其被反序列化还原并被打印时，可以看到每次运行时，对象将会出在不同的内存地址。
当然，我们期望反序列化还原后的对象地址与原来的地址不同。但是animals1和animals2中出现了相同的地址。当恢复animals3时，系统无法知道另一个流内的对象是第一个流内的对象的别名，因此它会产生出完全不同的对象网。
只要将任何对象序列化到单一流中，就可以恢复出与写出时一样的对象网，并且没有任何意外重复复制出的对象。

引入static字段的问题，如果发现Class是Serializable的，因此只需直接对Class对象序列化，就可以很容易地保存static字段：
```
abstract class Shape implements Serializable {
    public static final int RED = 1, BLUE = 2, GREEN = 3;
    private int xPos, yPos, dimension;
    private static Random rand = new Random(47);
    private static int counter = 0;

    public abstract void setColor(int newColor);

    public abstract int getColor();

    public Shape(int xVal, int yVal, int dim) {
        xPos = xVal;
        yPos = yVal;
        dimension = dim;
    }

    @Override
    public String toString() {
        return getClass() +
                "color[" + getColor() + "] xPos[" + xPos +
                "] yPos[" + yPos + "] dim[" + dimension + "]\n";
    }

    public static Shape randomFactory() {
        int xVal = rand.nextInt(100);
        int yVal = rand.nextInt(100);
        int dim = rand.nextInt(100);
        switch (counter++ % 3) {
            default:
            case 0:
                return new Circle(xVal, yVal, dim);
            case 1:
                return new Square(xVal, yVal, dim);
            case 2:
                return new Line(xVal, yVal, dim);
        }
    }
}

class Circle extends Shape {
    private static int color = RED;

    public Circle(int xVal, int yVal, int dim) {
        super(xVal, yVal, dim);
    }

    public void setColor(int newColor) {
        color = newColor;
    }

    public int getColor() {
        return color;
    }
}

class Square extends Shape {
    private static int color;

    public Square(int xVal, int yVal, int dim) {
        super(xVal, yVal, dim);
        color = RED;
    }

    public void setColor(int newColor) {
        color = newColor;
    }

    public int getColor() {
        return color;
    }
}

class Line extends Shape {
    private static int color = RED;

    public static void
    serializeStaticState(ObjectOutputStream os)
            throws IOException {
        os.writeInt(color);
    }

    public static void
    deserializeStaticState(ObjectInputStream os)
            throws IOException {
        color = os.readInt();
    }

    public Line(int xVal, int yVal, int dim) {
        super(xVal, yVal, dim);
    }

    public void setColor(int newColor) {
        color = newColor;
    }

    public int getColor() {
        return color;
    }
}

public class StoreCADState {
    public static void main(String[] args) throws Exception {
        List<Class<? extends Shape>> shapeTypes =
                new ArrayList<Class<? extends Shape>>();
        // Add references to the class objects:
        shapeTypes.add(Circle.class);
        shapeTypes.add(Square.class);
        shapeTypes.add(Line.class);
        List<Shape> shapes = new ArrayList<Shape>();
        // Make some shapes:
        for (int i = 0; i < 10; i++)
            shapes.add(Shape.randomFactory());
        // Set all the static colors to GREEN:
        for (int i = 0; i < 10; i++)
            ((Shape) shapes.get(i)).setColor(Shape.GREEN);
        // Save the state vector:
        ObjectOutputStream out = new ObjectOutputStream(
                new FileOutputStream("CADState.out"));
        out.writeObject(shapeTypes);
        Line.serializeStaticState(out);
        out.writeObject(shapes);
        // Display the shapes:
        System.out.println(shapes);
    }
} /*
[class Circlecolor[3] xPos[58] yPos[55] dim[93]
, class Squarecolor[3] xPos[61] yPos[61] dim[29]
, class Linecolor[3] xPos[68] yPos[0] dim[22]
, class Circlecolor[3] xPos[7] yPos[88] dim[28]
, class Squarecolor[3] xPos[51] yPos[89] dim[9]
, class Linecolor[3] xPos[78] yPos[98] dim[61]
, class Circlecolor[3] xPos[20] yPos[58] dim[16]
, class Squarecolor[3] xPos[40] yPos[11] dim[22]
, class Linecolor[3] xPos[4] yPos[83] dim[6]
, class Circlecolor[3] xPos[75] yPos[10] dim[42]
]
*/
```
Shap类实现了Seralizable，所以任何自Shape继承的类也都会自动是Serializable的。每个Shape都含有数据，而且每个派生自Shape的类都包含一个static 字段，用来确定各种Shape类型的颜色（如果将static字段置入基类，只会产生一个static字段，因为static字段不能在派生类中复制）。可对基类中的方法进行重载，以便为不同的类型设置颜色（static方法不会动态绑定，所以这些都是普通的方法）。每次调用randomFactory()方法时，它都会使用不同的随机数作为Shape的数据，从而创建不同的Shape。
在main()中，一个ArrayList用于保存对象，而另一个用于保存几何形状。
恢复对象相当直观：
```
public class RecoverCADState {
  @SuppressWarnings("unchecked")
  public static void main(String[] args) throws Exception {
    ObjectInputStream in = new ObjectInputStream(
      new FileInputStream("CADState.out"));
    // Read in the same order they were written:
    List<Class<? extends Shape>> shapeTypes =
      (List<Class<? extends Shape>>)in.readObject();
    Line.deserializeStaticState(in);
    List<Shape> shapes = (List<Shape>)in.readObject();
    System.out.println(shapes);
  }
} /*
[class Circlecolor[1] xPos[58] yPos[55] dim[93]
, class Squarecolor[0] xPos[61] yPos[61] dim[29]
, class Linecolor[3] xPos[68] yPos[0] dim[22]
, class Circlecolor[1] xPos[7] yPos[88] dim[28]
, class Squarecolor[0] xPos[51] yPos[89] dim[9]
, class Linecolor[3] xPos[78] yPos[98] dim[61]
, class Circlecolor[1] xPos[20] yPos[58] dim[16]
, class Squarecolor[0] xPos[40] yPos[11] dim[22]
, class Linecolor[3] xPos[4] yPos[83] dim[6]
, class Circlecolor[1] xPos[75] yPos[10] dim[42]
]
*/
```
可以看到，xPos、yPos以及dim的值都被成功地保存和恢复了，但是对static信息的读取却出现了问题。所有读回的颜色应该都是“3”， 但是真实情况却并非如此。Circle的值为1  (定义为RED),而Square的值为0 (记住，它们是在构造器中被初始化的)。看上去似乎static数据根本没有被序列化!确实如此一尽管Class类是Serializable的， 但它却不能按我们所期望的方式运行。所以假如想序列化static值，必须自己动手去实现。
这正是Line中的serializeStaticStateO和deserializeStaticState0两个static方法的用途。  可以看到，它们是作为存储和读取过程的一部分被显式地调用的。(注意必须维护写人序列化文件和从该文件中读回的顺序。)因此，为了使CADStatejava正确运转起来，我们必须:
1)为几何形状添加serializeStaticState()和deserializeStaticState().
2)移除ArrayList shapeTypes以及与之有关的所有代码。
3)在几何形状内添加对新的序列化和反序列化还原静态方法的调用。
另一个要注意的问题是安全，因为序列化也会将private数据保存下来。如果你关心安全问题，那么应将其标记成transient。  但是这之后，还必须设计一种安全的保存信息的方法，以便在执行恢复时可以复位那些private变量。
