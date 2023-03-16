I/O源端与之通信的接收端：文件、控制台、网络链接等。
通信方式：顺序、随机存取、缓冲、二进制、按字符、按行、按字等。

##File类
File（文件）既能代表一个特定文件名称，又能代表一个目录下的一组文件的名称。如果是文件集，可以对此集合调用list()方法，返回一个字符数组。

####目录列表器
```
// {Args: "D.*\.java"}
public class DirList {
    public static void main(String[] args) {
        File path = new File("./src/io");
        String[] list;
        if (args.length == 0) {
            list = path.list();
        } else {
            list = path.list(new DirFilter(args[0]));
        }
        Arrays.sort(list, String.CASE_INSENSITIVE_ORDER);
        for (String dirItem : list) {
            System.out.println(dirItem);
        }
    }
}

class DirFilter implements FilenameFilter {
    private Pattern pattern;

    public DirFilter(String regex) {
        pattern = Pattern.compile(regex);
    }

    @Override
    public boolean accept(File dir, String name) {
        return pattern.matcher(name).matches();
    }
} /*
DirectoryDemo.java
DirList.java
DirList2.java
DirList3.java
*/
```
DirFilter类实现了FilenameFilter接口：
```
public interface FilenameFilter {
    boolean accept(File dir, String name);
}
```
DirFilter这个类存在的唯一原因就是accept()方法。创建这个类的目的在于把accept()方法提供给list()使用，使list()可以回调accept()，进而以决定哪些文件包含在列表中。因此，这种结构也常常成为回调。
accept()会使用一个正则表达式的matcher对象，来查看此正则表达式regex是否匹配这个文件的名字。通过accept，list()方法会返回一个数组。

#####匿名内部类
使用匿名内部类改进：
```
public class DirList2 {
    public static FilenameFilter filter(final String regex) {
        return new FilenameFilter() {
            private Pattern pattern = Pattern.compile(regex);

            @Override
            public boolean accept(File dir, String name) {
                return pattern.matcher(name).matches();
            }
        };
    }

    public static void main(String[] args) {
        File path = new File(".");
        String[] list;
        if (args.length == 0) {
            list = path.list();
        } else {
            list = path.list(filter(args[0]));
        }
        Arrays.sort(list, String.CASE_INSENSITIVE_ORDER);
        for (String dirItem : list) {
            System.out.println(dirItem);
        }
    }
} /*
DirectoryDemo.java
DirList.java
DirList2.java
DirList3.java
*/
```
传向filter()的参数必须是final的。这在匿名内部类中是必需的，这样它才能够使用来自该类范围之外的对象。

进一步修改，定义一个作为list()参数的匿名内部类：
```
public class DirList3 {
    public static void main(final String[] args) {
        File path = new File(".");
        String[] list;
        if (args.length == 0) {
            list = path.list();
        } else {
            list = path.list(new FilenameFilter() {
                private Pattern pattern = Pattern.compile(args[0]);
                @Override
                public boolean accept(File dir, String name) {
                    return pattern.matcher(name).matches();
                }
            });
        }
        Arrays.sort(list, String.CASE_INSENSITIVE_ORDER);
        for (String dirItem : list) {
            System.out.println(dirItem);
        }
    }
} /*
DirectoryDemo.java
DirList.java
DirList2.java
DirList3.java
*/
```
既然匿名内部类直接使用args[0]，那么传递给main()方法的参数就是final的。

####目录实用工具
产生文件集的工具：
```
public final class Directory {
    public Directory() {
    }

    public static File[] local(File dir, final String regex) {
        return dir.listFiles(new FilenameFilter() {
            private Pattern pattern = Pattern.compile(regex);

            public boolean accept(File dir, String name) {
                return this.pattern.matcher((new File(name)).getName()).matches();
            }
        });
    }

    public static File[] local(String path, String regex) {
        return local(new File(path), regex);
    }

    public static Directory.TreeInfo walk(String start, String regex) {
        return recurseDirs(new File(start), regex);
    }

    public static Directory.TreeInfo walk(File start, String regex) {
        return recurseDirs(start, regex);
    }

    public static Directory.TreeInfo walk(File start) {
        return recurseDirs(start, ".*");
    }

    public static Directory.TreeInfo walk(String start) {
        return recurseDirs(new File(start), ".*");
    }

    static Directory.TreeInfo recurseDirs(File startDir, String regex) {
        Directory.TreeInfo result = new Directory.TreeInfo();
        File[] var6;
        int var5 = (var6 = startDir.listFiles()).length;

        for(int var4 = 0; var4 < var5; ++var4) {
            File item = var6[var4];
            if(item.isDirectory()) {
                result.dirs.add(item);
                result.addAll(recurseDirs(item, regex));
            } else if(item.getName().matches(regex)) {
                result.files.add(item);
            }
        }

        return result;
    }

    public static void main(String[] args) {
        if(args.length == 0) {
            System.out.println(walk("."));
        } else {
            String[] var4 = args;
            int var3 = args.length;

            for(int var2 = 0; var2 < var3; ++var2) {
                String arg = var4[var2];
                System.out.println(walk(arg));
            }
        }

    }

    public static class TreeInfo implements Iterable<File> {
        public List<File> files = new ArrayList();
        public List<File> dirs = new ArrayList();

        public TreeInfo() {
        }

        public Iterator<File> iterator() {
            return this.files.iterator();
        }

        void addAll(Directory.TreeInfo other) {
            this.files.addAll(other.files);
            this.dirs.addAll(other.dirs);
        }

        public String toString() {
            return "dirs: " + PPrint.pformat(this.dirs) + "\n\nfiles: " + PPrint.pformat(this.files);
        }
    }
}
```
local()方法使用被称为listFile()的File.list()的变体来产生File数组。
walk()方法将开始目录的名字转换为File对象，然后调用recurseDirs()，该方法将递归地遍历目录，并在每次递归中都手机更多的信息。为了区分普通文件和目录，返回值实际上是一个对象元组——一个List持有所有普通文件，另一个持有目录。
TreeInfo.toString()方法使用了一个灵巧打印机类，一个可以添加新行并缩排所有元素的工具：
```
public class PPrint {
    public PPrint() {
    }

    public static String pformat(Collection<?> c) {
        if(c.size() == 0) {
            return "[]";
        } else {
            StringBuilder result = new StringBuilder("[");

            Object elem;
            for(Iterator var3 = c.iterator(); var3.hasNext(); result.append(elem)) {
                elem = var3.next();
                if(c.size() != 1) {
                    result.append("\n  ");
                }
            }

            if(c.size() != 1) {
                result.append("\n");
            }

            result.append("]");
            return result.toString();
        }
    }

    public static void pprint(Collection<?> c) {
        System.out.println(pformat(c));
    }

    public static void pprint(Object[] c) {
        System.out.println(pformat(Arrays.asList(c)));
    }
}
```
pformat()方法可以从Collection中产生格式化的String，而pprint()方法使用pformat()来执行其任务。
```
public class DirectoryDemo {
    public static void main(String[] args) {
        PPrint.pprint(Directory.walk(".").dirs);
        for (File file : Directory.local(".", "T.*")) {
            print(file);
        }
        print("----------------------");
        for (File file : Directory.walk(".", "T.*\\.java")) {
            print(file);
        }
        print("======================");
        for (File file : Directory.walk(".", ".*[Zz].*\\.class")) {
            print(file);
        }
    }
} /*
[.\xfiles]
.\TestEOF.class
.\TestEOF.java
.\TransferTo.class
.\TransferTo.java
----------------------
.\TestEOF.java
.\TransferTo.java
.\xfiles\ThawAlien.java
======================
.\FreezeAlien.class
.\GZIPcompress.class
.\ZipCompress.class
*/
```

创建一个工具，它可以在目录中穿行，并且根据Strategy对象来处理这些目录中的文件：
```
public class ProcessFiles {
    private ProcessFiles.Strategy strategy;
    private String ext;

    public ProcessFiles(ProcessFiles.Strategy strategy, String ext) {
        this.strategy = strategy;
        this.ext = ext;
    }

    public void start(String[] args) {
        try {
            if(args.length == 0) {
                this.processDirectoryTree(new File("."));
            } else {
                String[] var5 = args;
                int var4 = args.length;

                for(int var3 = 0; var3 < var4; ++var3) {
                    String e = var5[var3];
                    File fileArg = new File(e);
                    if(fileArg.isDirectory()) {
                        this.processDirectoryTree(fileArg);
                    } else {
                        if(!e.endsWith("." + this.ext)) {
                            e = e + "." + this.ext;
                        }

                        this.strategy.process((new File(e)).getCanonicalFile());
                    }
                }
            }

        } catch (IOException var7) {
            throw new RuntimeException(var7);
        }
    }

    public void processDirectoryTree(File root) throws IOException {
        Iterator var3 = Directory.walk(root.getAbsolutePath(), ".*\\." + this.ext).iterator();

        while(var3.hasNext()) {
            File file = (File)var3.next();
            this.strategy.process(file.getCanonicalFile());
        }

    }

    public static void main(String[] args) {
        (new ProcessFiles(new ProcessFiles.Strategy() {
            public void process(File file) {
                System.out.println(file);
            }
        }, "java")).start(args);
    }

    public interface Strategy {
        void process(File var1);
    }
}
```
Strategy接口内嵌在ProcessFiles中，使得如果希望实现它，就必须实现ProcessFiles.Strategy，为读者提供了更多的上下文信息。ProcessFiles执行了查找具有特定扩展名的文件所需的全部工作，并且当它找到匹配的文件时，将直接把文件传递给Strategy对象。
如果没有提供任何参数，那么ProcessFiles就遍历当前目录下的所有目录。可以指定特定文件，或者一个或多个目录。

####目录的检查及创建
```
public class MakeDirectories {
    private static void usage() {
        System.err.println(
                "Usage:MakeDirectories path1 ...\n" +
                        "Creates each path\n" +
                        "Usage:MakeDirectories -d path1 ...\n" +
                        "Deletes each path\n" +
                        "Usage:MakeDirectories -r path1 path2\n" +
                        "Renames from path1 to path2");
        System.exit(1);
    }

    private static void fileData(File f) {
        System.out.println(
                "Absolute path: " + f.getAbsolutePath() +
                        "\n Can read: " + f.canRead() +
                        "\n Can write: " + f.canWrite() +
                        "\n getName: " + f.getName() +
                        "\n getParent: " + f.getParent() +
                        "\n getPath: " + f.getPath() +
                        "\n length: " + f.length() +
                        "\n lastModified: " + f.lastModified());
        if (f.isFile()) {
            System.out.println("It's a file");
        } else if (f.isDirectory()) {
            System.out.println("It's a directory");
        }
    }

    public static void main(String[] args) {
        if (args.length < 1) {
            usage();
        }
        if (args[0].equals("-r")) {
            if (args.length != 3) {
                usage();
            }
            File
                    old = new File(args[1]),
                    rname = new File(args[2]);
            old.renameTo(rname);
            fileData(old);
            fileData(rname);
            return;
        }
        int count = 0;
        boolean del = false;
        if (args[0].equals("-d")) {
            count++;
            del = true;
        }
        count--;
        while (++count < args.length) {
            File f = new File(args[count]);
            if (f.exists()) {
                System.out.println(f + " exists");
                if (del) {
                    System.out.println("deleting..." + f);
                    f.delete();
                }
            } else { // Doesn't exist
                if (!del) {
                    f.mkdirs();
                    System.out.println("created " + f);
                }
            }
            fileData(f);
        }
    }
} /*
created MakeDirectoriesTest
Absolute path: d:\aaa-TIJ4\code\io\MakeDirectoriesTest
 Can read: true
 Can write: true
 getName: MakeDirectoriesTest
 getParent: null
 getPath: MakeDirectoriesTest
 length: 0
 lastModified: 1101690308831
It's a directory
*/
```
在fileData()中，可以看到了多种不同的文件特征查询方法来显示方法或目录路径的信息。main()方法首先调用的是renameTo()，用来把一个文件重命名（或移动）到由参数所指示的另一个完全不同的新路径。

##输入和输出
编程语言的I/O库经常使用<font color=red>流</font>，它代表任何有能力产生数据的数据源对象或者有能力接受数据的接收端对象。
Java类库中的I/O类分成输入和输出两部分。通过继承，任何自Inputstream或Reader派生而来的类都含有名为read的基本方法，用于读取单个字节或者字节数组。同样的OutputStream或者Writer派生出来的类都含有名为write的基本方法，用于写单个字节或者字节数组。但是不会用到，它们之所以存在是因为别的类可以使用它们以便提供更有用的接口。因此，很少使用单一的类来创建流对象，而是通过叠合多个对象来提供所期望的功能。

####InputStream类型
InputStream的作用是用来表示从不同数据源产生输出的类，这些数据源包括：
1.字节数组
2.String对象
3.文件
4.管道，工作方式与实际管道相似，即，从一段输入，从另一端输出
5.一个由其他种类的流组成的序列，以便可以将它们收集合并到一个流内
6.其他数据源，如Internet链接等
每一种数据源都有相应的InputStream子类。另外，FilterInputStream也属于一种InputStream，为装饰器类提供的基类，其中，装饰器类可以把属性或有用的接口与输入流连接在一起。
![这里写图片描述](https://img-blog.csdn.net/20180518115140228?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

####OutputStream类型
该类别的类决定了输出所要去往的目标：字节数组、文件或管道。
另外，FilterOutputStream为装饰器类提供了一个基类，装饰器类把属性或者有用的接口与输出流连接了起来：
![这里写图片描述](https://img-blog.csdn.net/20180518115640960?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

##添加属性和有用的接口
Java I/O类库里存在filter（过滤器）类的原因所在抽象类filter是所有装饰器类的基类。
FilterInputStream和FilterOutputStream是用来提供装饰类器接口以及控制特定输入流和输出流的两个类。FilterInputStream和FilterOutputStream分别自I/O类库中的基类InputStream和OutputStream派生而来，这两个类是装饰器的必要条件。

####通过FilterInputStream从InputStream读取数据
FilterInputStream类能够完成完全不同的事情，其中，DateInputStream允许读取不同的基本类型数据以及String对象。
其他FilterInputStream类则在内部修改InputStream的行为方式：是否缓冲，是否保留它所读过的行（允许查询行数或设置行数），以及是否把单一字符推回输入流。
![这里写图片描述](https://img-blog.csdn.net/20180518135346507?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

####通过FilterOutputStream向OutputStream导入
![这里写图片描述](https://img-blog.csdn.net/20180518135602654?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

##Reader和Writer
InputStream和OutputStream在以面向字节形式的IO中可以提供极有价值的功能，Reader和Writer（Java 1.1对基础IO流类库进行了重大修改，可能会以为是用来替换InputStream和OutputStream的）则提供兼容Unicode和面向字符的IO功能。
1.Java 1.1向InputStream和OutputStream继承层次中添加了一些新类，所以这两个类不会被取代
2.有时必须把来自于字节层次结构中的类和字符层次中的类结合起来。为了实现这个目的，要用到适配器类：InputStreamReader可以吧InputStream转换为Reader，而OutputStreamWriter可以吧OutputStream转换为Writer。

设计Reader和Writer继承层次结构只要是为了国际化。老的IO流继承层次结构仅支持8位字节流，并且不能很好地处理16位的Unicode字符。所以Reader和Writer继承层次结构就是为了在所有IO操作中都支持Unicode。

####数据的来源和去处
几乎所有的原始IO刘类都有相应的Reader和Writer类来提供天然的Unicode操作。因此尽量尝试使用Reader和Writer，一旦程序代码无法成功编译，在不得不去使用面向字节的类库。
![这里写图片描述](https://img-blog.csdn.net/20180518141023170?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

####更改流的行为
对于InputStream和OutputStream来说，有装饰器子类来修改流以满足需要。Reader和Writer的类继承层次结构继续沿用相同的思想——但不完全相同。
下表中，相对于前一表来说，左右之间的对应关系更加粗略一些。造成这种差别的原因是因为类的组织形式不同。尽管BufferedOutputStream是FilterOutputStream的子类，但是BufferedWriter并不是FilterWriter的子类。
![这里写图片描述](https://img-blog.csdn.net/2018051814141218?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

无论何时使用readLine()，都不应该使用DataInputStream，而应该使用BufferedReader。
为了更容易地过渡到使用PrintWriter，它提供了一个既接受Writer对象又能接受任何OutputStream对象的构造器。PrintWriter的格式化接口实际上与PrintStream相同。
在Java SE5中添加了PrintWriter构造器，以简化在讲输出写入时的文件创建过程。
有一种PrintWriter构造器还有一个选项，就是自动执行清空选项。如果构造器设置此选项，则在每个Println()执行之后，便会自动清空。

####未发生变化的类
一些类在Java 1.0和java 1.1之间未做变化，DataOutputStream,File,RandomAccessFile,SequenceInputStream。

##自我独立的类：RandomAccessFile
RandomAccessFile适用于由大小已知的记录组成的文件，所以可以使用seek()将记录从一处转移到另一处，然后读取或者修改记录。文件中记录的大小不一定都相同，只要能够确定那些记录有多大以及它们在文件中的位置即可。
  最初，可能难以相信RandomAcessFile不是InputStream或者OutputStream继承层次结构中的一部分。除了实现了DataInput和DataOutput接口(DatalnputStream和DataOutputStream也实现了这两个接口)之外，它和这两个继承层次结构没有任何关联。它甚至不使用InputStream和OutputStream类中已有的任何功能。它是一个完全独立的类，从头开始编写其所有的方法(大多数都是本地的)。这么做是因为RandomccessFile拥有和别的I/O类型本质不同的行为，因为可以在一个文件内向前和向后移动。在任何情况下，它都是自我独立的，直接从Object派生而来。
 从本质上来说，RandomAccessFile的工作方式类似于把DataInputStream和DataOutStream组合起来使用，还添加了一些方法。其中方法getFilePointer()用于查找当前所处的文件位置，seek()用于在文件内移至新的位置，length()用于判断文件的最大尺寸。另外，其构造器还需要第二个参数(和C中的fopen()相同)用来指示我们只是“随机读”(r)还是“既读又写”(rw)。它并不支持只写文件，这表明RandomAccessFile若是从DataInputStream继承而来也可能会运行得很好。
只有RandonccessFile支持搜寻方法，并且只适用于文件。BufferedInputStream却能允许标往(mark0) 位置(其值存储于内部某个简单变量内)和重新设定位置(reset0), 但这些功能很有限，不是非常有用。 
在JDK 1.4中，RandomAccessFile的大多数功能(但不是全部)由nio存 储映射文件所取代，
