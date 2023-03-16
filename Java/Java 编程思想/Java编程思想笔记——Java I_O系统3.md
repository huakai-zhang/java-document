##新IO
JDK 1.4的java.nio.*包中引入了新的IO类库，其目的在于提高速度。实际上，旧的IO包已经使用nio重新实现过，以便充分利用这种速度提高。
速度提高源自于所使用的结构更接近于操作系统执行IO的方式：通道和缓冲器。
唯一直接与通道交互的缓冲器是ByteBuffer，可以存储未加工字节的缓冲器。java.nio.ByteBuffer是相当基础的类：通过稿子分配多少存储空间来创建一个ByteBuffer对象，并且还有一个方法选择集，用于以原始的字节形式或基本数据类型输出和读取数据。但是，没办法输出或读取对象，即使是字符串对象也不行。这种处理虽然很低级，但却正好，因为这是大多数草走系统中更有效的映射方式。
旧IO类库有三个类被修改了，用以产生FileChannel。这三个被修改类是FileInputStream、FileOutputStream以及用于既读又写的RandomAccessFile。这些都是字节操作流，与底层nio性质一致。Reader和Writer这些字符模式类不能用于产生通道；但是java.nio.channels.Channels类提供了使用方法，用于在通道中产生Reader和Writer。
```
public class GetChannel {
    private static final int BSIZE = 1024;

    public static void main(String[] args) throws Exception {
        FileChannel fc = new FileOutputStream("data.txt").getChannel();
        fc.write(ByteBuffer.wrap("Some text ".getBytes()));
        fc.close();
        fc = new RandomAccessFile("data.txt", "rw").getChannel();
        fc.position(fc.size());
        fc.write(ByteBuffer.wrap("Some more".getBytes()));
        fc.close();
        fc = new FileInputStream("data.txt").getChannel();
        ByteBuffer buff = ByteBuffer.allocate(BSIZE);
        fc.read(buff);
        buff.flip();
        while (buff.hasRemaining()) {
            System.out.print((char) buff.get());
        }
    }
} /* 
Some text Some more
*/
```
getChannel()将会产生一个FileChannel。通道是一种相当基础的：可以向它传送用于读写的ByteBuffer，并且可以锁定文件的某些区域用于独占式访问。
使用warp()方法将已存在的字节数组包装到ByteBuffer中。
data.txt文件用RandomAccessFile被再次打开。注意可以在文件内随处移动FileChanel；这里，把它移动到最后，以便附加其他写操作。
对于只读访问，必须显式地使用静态的allocate()方法来分配ByteBuffer。nio的目的就是快速移动大量数据，因此ByteBuffer的大小显得尤为重要——实际上，使用1K可能比通常使用的小一点（必须通过实际运行应用程序来找到最佳尺寸）。甚至叨叨更高速度，使用allocateDirect()，以产生一个与操作系统有更高耦合性的直接缓冲器（但分配的开支会更大）。
一旦调用read()来告知FileChannel向ByteBuffer存储字节，就必须调用缓冲器上的flip()，让它做好让别人读取字节的准备。如果打算使用缓冲器执行进一步read()操作，也必须得使用clear()来为每个read()做好准备。在下面简单文件复制程序可以看到：
```
// {Args: ChannelCopy.java test.txt}

public class ChannelCopy {
  private static final int BSIZE = 1024;
  public static void main(String[] args) throws Exception {
    if(args.length != 2) {
      System.out.println("arguments: sourcefile destfile");
      System.exit(1);
    }
    FileChannel
      in = new FileInputStream(args[0]).getChannel(),
      out = new FileOutputStream(args[1]).getChannel();
    ByteBuffer buffer = ByteBuffer.allocate(BSIZE);
    while(in.read(buffer) != -1) {
      buffer.flip();
      out.write(buffer);
      buffer.clear();
    }
  }
}
```
打开两个FileChannel一个用于读，一个用于写。ByteBuffer被分配了空间，当FileChannel.read()返回-1时，表示已到达了输入的末尾。每次read()操作之后，就会将数据输入到缓冲器中，flip()则是准备缓冲器以便它的信息可以有write()提取。write()操作之后，信息仍在缓冲器中，接着clear()操作则对所有的内部指针重新安排，以便缓冲器在另一个read()操作期间能够做好接收数据的准备。
特殊方法transferTo()和transferFrom()则允许将一个通道和另一个通道直接相连：
```
//Args: TransferTo.java TransferTo.txt
public class TransferTo {
  public static void main(String[] args) throws Exception {
    if(args.length != 2) {
      System.out.println("arguments: sourcefile destfile");
      System.exit(1);
    }
    FileChannel
      in = new FileInputStream(args[0]).getChannel(),
      out = new FileOutputStream(args[1]).getChannel();
    in.transferTo(0, in.size(), out);
    // Or:
    // out.transferFrom(in, 0, in.size());
  }
}
```

####转换数据
在GetChannel.java中，必须每次只读取一个字节的数据，然后将每个byte类型强制转换成char类型。而java.nio.CharBuffer有一个toString方法：返回一个包含缓冲器中所有字符的字符串。既然ByteBuffer可以看作是具有asCharBuffer()方法的CharBuffer，为什么不用？
```
public class BufferToText {
  private static final int BSIZE = 1024;
  public static void main(String[] args) throws Exception {
    FileChannel fc =
      new FileOutputStream("data2.txt").getChannel();
    fc.write(ByteBuffer.wrap("Some text".getBytes()));
    fc.close();
    fc = new FileInputStream("data2.txt").getChannel();
    ByteBuffer buff = ByteBuffer.allocate(BSIZE);
    fc.read(buff);
    buff.flip();
    // Doesn't work:
    System.out.println(buff.asCharBuffer());
    // Decode using this system's default Charset:
    buff.rewind();
    String encoding = System.getProperty("file.encoding");
    System.out.println("Decoded using " + encoding + ": "
      + Charset.forName(encoding).decode(buff));
    // Or, we could encode with something that will print:
    fc = new FileOutputStream("data2.txt").getChannel();
    fc.write(ByteBuffer.wrap(
      "Some text".getBytes("UTF-16BE")));
    fc.close();
    // Now try reading again:
    fc = new FileInputStream("data2.txt").getChannel();
    buff.clear();
    fc.read(buff);
    buff.flip();
    System.out.println(buff.asCharBuffer());
    // Use a CharBuffer to write through:
    fc = new FileOutputStream("data2.txt").getChannel();
    buff = ByteBuffer.allocate(24); // More than needed
    buff.asCharBuffer().put("Some text");
    fc.write(buff);
    fc.close();
    // Read and display:
    fc = new FileInputStream("data2.txt").getChannel();
    buff.clear();
    fc.read(buff);
    buff.flip();
    System.out.println(buff.asCharBuffer());
  }
} /*
卯浥⁴數
Decoded using UTF-8: Some text
Some text
Some text   
*/
```
如输出语句的第一行所见，这种方法并不能解决问题。
调用rewind()返回到数据开始部分，接着使用平台默认字符集对数据进行decode()。使用System.getProperty("file.encoding")发现默认字符集。把该字符串传送给Charset.forName()用以产生Charset对象，可以用它对字符串解码。
另一选择，在读取文件时使用能够产生打印的输出的字符集进行encode()，这里UTF-16BE可以把文本写到文件中，当读取时，只需要把它转换成CharBuffer。
最后，为ByteBuffer分配了24个字节，一个字符需要2个字节，那么ByteBuffer足可容纳12个字符，但是“Some text”只有9个字符，剩余的内容为0的字节让出现在由他的toString()所产生的CarBuffer的表示在。

缓冲器容纳的是普通字节，为了把它们转换成字符，要么在输入它们时候对其进行编码，要么在将其从缓冲器输出时对它们进行解码。可以使用java.nio.charset.Charset类实现这些功能，该类提供了把数据编码成多种不同类型的字节集的工具：
```
public class AvailableCharSets {
    public static void main(String[] args) {
        SortedMap<String, Charset> charSets = Charset.availableCharsets();
        Iterator<String> it = charSets.keySet().iterator();
        while (it.hasNext()) {
            String csName = it.next();
            printnb(csName);
            Iterator aliases =
                    charSets.get(csName).aliases().iterator();
            if (aliases.hasNext()) {
                printnb(": ");
            }
            while (aliases.hasNext()) {
                printnb(aliases.next());
                if (aliases.hasNext()) {
                    printnb(", ");
                }
            }
            print();
        }
    }
} /*
Big5: csBig5
Big5-HKSCS: big5-hkscs, big5hk, big5-hkscs:unicode3.0, big5hkscs, Big5_HKSCS
EUC-JP: eucjis, x-eucjp, csEUCPkdFmtjapanese, eucjp, Extended_UNIX_Code_Packed_Format_for_Japanese, x-euc-jp, euc_jp
EUC-KR: ksc5601, 5601, ksc5601_1987, ksc_5601, ksc5601-1987, euc_kr, ks_c_5601-1987, euckr, csEUCKR
GB18030: gb18030-2000
GB2312: gb2312-1980, gb2312, EUC_CN, gb2312-80, euc-cn, euccn, x-EUC-CN
GBK: windows-936, CP936
...
*/
```

####获取基本类型
尽管ByteBuffer只能保存字节类型数据，但是它可以从其所容纳的字节中产生出各种不同的基本类型值的方法：
```
public class GetData {
    private static final int BSIZE = 1024;

    public static void main(String[] args) {
        ByteBuffer bb = ByteBuffer.allocate(BSIZE);
        // Allocation automatically zeroes the ByteBuffer:
        int i = 0;
        while (i++ < bb.limit()) {
            if (bb.get() != 0) {
                print("nonzero");
            }
        }
        print("i = " + i);
        bb.rewind();
        // Store and read a char array:
        bb.asCharBuffer().put("Howdy!");
        char c;
        while ((c = bb.getChar()) != 0) {
            printnb(c + " ");
        }
        print();
        bb.rewind();
        // Store and read a short:
        bb.asShortBuffer().put((short) 471142);
        print(bb.getShort());
        bb.rewind();
        // Store and read an int:
        bb.asIntBuffer().put(99471142);
        print(bb.getInt());
        bb.rewind();
        // Store and read a long:
        bb.asLongBuffer().put(99471142);
        print(bb.getLong());
        bb.rewind();
        // Store and read a float:
        bb.asFloatBuffer().put(99471142);
        print(bb.getFloat());
        bb.rewind();
        // Store and read a double:
        bb.asDoubleBuffer().put(99471142);
        print(bb.getDouble());
        bb.rewind();
    }
} /*
i = 1025
H o w d y !
12390
99471142
99471142
9.9471144E7
9.9471142E7
*/
```

####视图缓冲器
视图缓冲器（view buffer）可以让我们通过某个特定的基本数据类型的视窗查看其底层的ByteBuffer。对视图的任何修改都会映射成为对ByteBuffer中数据的修改。
```
public class IntBufferDemo {
  private static final int BSIZE = 1024;
  public static void main(String[] args) {
    ByteBuffer bb = ByteBuffer.allocate(BSIZE);
    IntBuffer ib = bb.asIntBuffer();
    ib.put(new int[]{ 11, 42, 47, 99, 143, 811, 1016 });
    System.out.println(ib.get(3));
    ib.put(3, 1811);
    ib.flip();
    while(ib.hasRemaining()) {
      int i = ib.get();
      System.out.println(i);
    }
  }
} /*
99
11
42
47
1811
143
811
1016
*/
```
先用重载后的put()方法存储一个整数数组。接着get()和put()方法调用直接访问底层ByteBuffer中的某个整数位置。注意，这些通过直接与ByteBuffer对话访问绝对位置的方式也同样适用于基本类型。
一旦底层的ByteBuffer通过视图缓冲器填满了整数或其他基本类型时，就可以直接被写到通道中。正像从通道中读取那样容易，然后使用视图缓冲器可以把任何数据都转化为某一特定的基本类型。
```
public class ViewBuffers {
    public static void main(String[] args) {
        ByteBuffer bb = ByteBuffer.wrap(new byte[]{0, 0, 0, 0, 0, 0, 0, 'a'});
        bb.rewind();
        printnb("Byte Buffer ");
        while (bb.hasRemaining()) {
            printnb(bb.position() + " -> " + bb.get() + ", ");
        }
        print();
        CharBuffer cb =
                ((ByteBuffer) bb.rewind()).asCharBuffer();
        printnb("Char Buffer ");
        while (cb.hasRemaining()) {
            printnb(cb.position() + " -> " + cb.get() + ", ");
        }
        print();
        FloatBuffer fb =
                ((ByteBuffer) bb.rewind()).asFloatBuffer();
        printnb("Float Buffer ");
        while (fb.hasRemaining()) {
            printnb(fb.position() + " -> " + fb.get() + ", ");
        }
        print();
        IntBuffer ib =
                ((ByteBuffer) bb.rewind()).asIntBuffer();
        printnb("Int Buffer ");
        while (ib.hasRemaining()) {
            printnb(ib.position() + " -> " + ib.get() + ", ");
        }
        print();
        LongBuffer lb =
                ((ByteBuffer) bb.rewind()).asLongBuffer();
        printnb("Long Buffer ");
        while (lb.hasRemaining()) {
            printnb(lb.position() + " -> " + lb.get() + ", ");
        }
        print();
        ShortBuffer sb =
                ((ByteBuffer) bb.rewind()).asShortBuffer();
        printnb("Short Buffer ");
        while (sb.hasRemaining()) {
            printnb(sb.position() + " -> " + sb.get() + ", ");
        }
        print();
        DoubleBuffer db =
                ((ByteBuffer) bb.rewind()).asDoubleBuffer();
        printnb("Double Buffer ");
        while (db.hasRemaining()) {
            printnb(db.position() + " -> " + db.get() + ", ");
        }
    }
} /*
Byte Buffer 0 -> 0, 1 -> 0, 2 -> 0, 3 -> 0, 4 -> 0, 5 -> 0, 6 -> 0, 7 -> 97,
Char Buffer 0 ->  , 1 ->  , 2 ->  , 3 -> a,
Float Buffer 0 -> 0.0, 1 -> 1.36E-43,
Int Buffer 0 -> 0, 1 -> 97,
Long Buffer 0 -> 97,
Short Buffer 0 -> 0, 1 -> 0, 2 -> 0, 3 -> 97,
Double Buffer 0 -> 4.8E-322,
*/
```
ByteBuffer通过一个被包装过的8字节数组产生，然后通过各种不同的基本类型的视图缓冲器显示出来。
在下图中看到，当从不同类型的缓冲器读取时，数据显示的方式也不同：
![这里写图片描述](https://img-blog.csdn.net/20180521125423415?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

#####字节存放次序
不同的机器可能会使用不同的字节排序方法来存储数据。big endian高位优先将最重要的字节存放在地址最低的存储器单元。而little endian低位优先相反。ByteBuffer是以告慰优先的形式存储数据的。可以使用参数ByteOrder.Big_ENDIAN和ByteOrder.LITTER_ENDIAN的order()方法来改变排序方式。
考虑包含下面两个字节的ByteBuffer：
![这里写图片描述](https://img-blog.csdn.net/20180521130327158?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

如果以short形式读取数据，得到数字97，但如果改为低位优先，得到的是24832（01100001 00000000）；
```
public class Endians {
  public static void main(String[] args) {
    ByteBuffer bb = ByteBuffer.wrap(new byte[12]);
    bb.asCharBuffer().put("abcdef");
    print(Arrays.toString(bb.array()));
    bb.rewind();
    bb.order(ByteOrder.BIG_ENDIAN);
    bb.asCharBuffer().put("abcdef");
    print(Arrays.toString(bb.array()));
    bb.rewind();
    bb.order(ByteOrder.LITTLE_ENDIAN);
    bb.asCharBuffer().put("abcdef");
    print(Arrays.toString(bb.array()));
  }
} /*
[0, 97, 0, 98, 0, 99, 0, 100, 0, 101, 0, 102]
[0, 97, 0, 98, 0, 99, 0, 100, 0, 101, 0, 102]
[97, 0, 98, 0, 99, 0, 100, 0, 101, 0, 102, 0]
*/
```
ByteBuffer有足够的空间，以存储作为外部缓冲器的charArray的所有字节，因此可以调用array()方法显示视图底层的字节。array()方法是可选的，并且只能对由数组支持的缓冲器调用此方法。否则，将会抛出UnsupportedOperationException。

####用缓冲器操纵数据
![这里写图片描述](https://img-blog.csdn.net/20180521132803196?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

此图阐明了nio类之间的关系，便于理解怎么移动和转换数据。如果想把一个字节数组写到文件中去，那么就应该使用ByteBuffer.wrap()方法把字节数组包装起来，然后用getChannel()方法在FileOutputStream上打开一个通道，接着将来自于ByteBuffer的数据写到FileChannel。
注意：BytBuffer是将数据移进移出通道的唯一方式，并且只能创建一个独立的基本类型缓冲器，或者使用as方法从ByteBuffer中获得。也就是说，不能把基本类型的缓冲器转换成ByteBuffer。

####缓冲器的细节
Buffer有数据和可以高效地访问及操作这些数据的四个索引组成，mark（标记）、position（位置）、limit（界限）和capacity（容量）。
![这里写图片描述](https://img-blog.csdn.net/20180521133647661?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

在缓冲器中插入和提取数据的方法会更新索引，用于反应所发生的变化。
交换相邻字符，以对CharBuffer中的字符进行编码和译码：
```
public class UsingBuffers {
  private static void symmetricScramble(CharBuffer buffer){
    while(buffer.hasRemaining()) {
      buffer.mark();
      char c1 = buffer.get();
      char c2 = buffer.get();
      buffer.reset();
      buffer.put(c2).put(c1);
    }
  }
  public static void main(String[] args) {
    char[] data = "UsingBuffers".toCharArray();
    ByteBuffer bb = ByteBuffer.allocate(data.length * 2);
    CharBuffer cb = bb.asCharBuffer();
    cb.put(data);
    print(cb.rewind());
    symmetricScramble(cb);
    print(cb.rewind());
    symmetricScramble(cb);
    print(cb.rewind());
  }
} /*
UsingBuffers
sUniBgfuefsr
UsingBuffers
*/
```
CharBuffer只是一个视图而已，总是在操纵ByteBuffer，因为它可以和通道进行交互。
进入symmetricScramble()方法时缓冲器的样子：
position指针指向缓冲器第一个元素，capacity和limit指向最后一个元素。迭代while循环直到position等于limit。
![这里写图片描述](https://img-blog.csdn.net/20180521134237519?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
当操纵到while循环时，使用mark()来设置mark的值：
![这里写图片描述](https://img-blog.csdn.net/2018052113424877?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
调用完两个get()方法后，缓冲器如下：
![这里写图片描述](https://img-blog.csdn.net/20180521134259749?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
为了实现交换，要在position=0时写入c2，position=1时写入c1。使用reset()把position的值设为mark的值：
![这里写图片描述](https://img-blog.csdn.net/20180521134308394?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
写完之后的状态：
![这里写图片描述](https://img-blog.csdn.net/20180521134317430?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
在下一次循环迭代期间，将mark设置成position的当前值：
![这里写图片描述](https://img-blog.csdn.net/20180521134325899?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
在while循环的最后，position指向缓冲器的末尾。如果想打印缓冲器，只能打印position和limit之间的字符。因此必须使用rewind设置缓冲器开始位置：
![这里写图片描述](https://img-blog.csdn.net/20180521134332456?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

####内存映射文件
内存映射文件允许创建和修改因为太大而不能放入内存的文件。可以假定整个文件都放在内存中，而且可以完全把它当作非常大的数组访问：
```
public class LargeMappedFiles {
    static int length = 0x8FFFFFF; // 128 MB
    public static void main(String[] args) throws Exception {
        MappedByteBuffer out =
                new RandomAccessFile("test.dat", "rw").getChannel()
                        .map(FileChannel.MapMode.READ_WRITE, 0, length);
        for (int i = 0; i < length; i++) {
            out.put((byte) 'x');
        }
        print("Finished writing");
        for (int i = length / 2; i < length / 2 + 6; i++) {
            printnb((char) out.get(i));
        }
    }
}
```
使用map()产生MappedByteBuffer，一个特殊类型的直接缓冲器，必须指定映射文件初始位置和映射区域长度。
MappedByteBuffer由ByteBuffer继承而来，因此它具有ByteBuffer的所有方法。
创建了128MB，这可能比操作系统所允许一次载入内存的空间大。但似乎可以一次访问整个文件。因为只有一部分文件放入了内存，其他被交换了出去。
底层操作系统的文件映射工具是用来最大化地提高性能的。

#####性能
nio实现后性能有所提高，但是映射文件访问往往可以更加显著地加快速度。
```
public class MappedIO {
  private static int numOfInts = 4000000;
  private static int numOfUbuffInts = 200000;
  private abstract static class Tester {
    private String name;
    public Tester(String name) { this.name = name; }
    public void runTest() {
      System.out.print(name + ": ");
      try {
        long start = System.nanoTime();
        test();
        double duration = System.nanoTime() - start;
        System.out.format("%.2f\n", duration/1.0e9);
      } catch(IOException e) {
        throw new RuntimeException(e);
      }
    }
    public abstract void test() throws IOException;
  }
  private static Tester[] tests = {
    new Tester("Stream Write") {
      public void test() throws IOException {
        DataOutputStream dos = new DataOutputStream(
          new BufferedOutputStream(
            new FileOutputStream(new File("temp.tmp"))));
        for(int i = 0; i < numOfInts; i++)
          dos.writeInt(i);
        dos.close();
      }
    },
    new Tester("Mapped Write") {
      public void test() throws IOException {
        FileChannel fc =
          new RandomAccessFile("temp.tmp", "rw")
          .getChannel();
        IntBuffer ib = fc.map(
          FileChannel.MapMode.READ_WRITE, 0, fc.size())
          .asIntBuffer();
        for(int i = 0; i < numOfInts; i++)
          ib.put(i);
        fc.close();
      }
    },
    new Tester("Stream Read") {
      public void test() throws IOException {
        DataInputStream dis = new DataInputStream(
          new BufferedInputStream(
            new FileInputStream("temp.tmp")));
        for(int i = 0; i < numOfInts; i++)
          dis.readInt();
        dis.close();
      }
    },
    new Tester("Mapped Read") {
      public void test() throws IOException {
        FileChannel fc = new FileInputStream(
          new File("temp.tmp")).getChannel();
        IntBuffer ib = fc.map(
          FileChannel.MapMode.READ_ONLY, 0, fc.size())
          .asIntBuffer();
        while(ib.hasRemaining())
          ib.get();
        fc.close();
      }
    },
    new Tester("Stream Read/Write") {
      public void test() throws IOException {
        RandomAccessFile raf = new RandomAccessFile(
          new File("temp.tmp"), "rw");
        raf.writeInt(1);
        for(int i = 0; i < numOfUbuffInts; i++) {
          raf.seek(raf.length() - 4);
          raf.writeInt(raf.readInt());
        }
        raf.close();
      }
    },
    new Tester("Mapped Read/Write") {
      public void test() throws IOException {
        FileChannel fc = new RandomAccessFile(
          new File("temp.tmp"), "rw").getChannel();
        IntBuffer ib = fc.map(
          FileChannel.MapMode.READ_WRITE, 0, fc.size())
          .asIntBuffer();
        ib.put(0);
        for(int i = 1; i < numOfUbuffInts; i++)
          ib.put(ib.get(i - 1));
        fc.close();
      }
    }
  };
  public static void main(String[] args) {
    for(Tester test : tests)
      test.runTest();
  }
} /*
Stream Write: 0.31
Mapped Write: 0.04
Stream Read: 0.46
Mapped Read: 0.04
Stream Read/Write: 7.26
Mapped Read/Write: 0.01
*/
```
尽管映射写似乎要用到FileOurputStream，但是映射文件中的所有输出必须使用RandomAccessFile。
test()方法包括初始化各种IO对象的时间，因此，即使建立映射文件的花费很大，但是整体受益比起IO流来说还是很显著的。

####文件加锁
JDK 1.4引入了文件加锁机制，允许同步访问某个做为共享资源的文件。文件锁对其他的操作系统进程是可见的，因为Java的文件加锁直接映射到本地操作系统的加锁工具。
```
public class FileLocking {
  public static void main(String[] args) throws Exception {
    FileOutputStream fos= new FileOutputStream("file.txt");
    FileLock fl = fos.getChannel().tryLock();
    if(fl != null) {
      System.out.println("Locked File");
      TimeUnit.MILLISECONDS.sleep(100);
      fl.release();
      System.out.println("Released Lock");
    }
    fos.close();
  }
} /*
Locked File
Released Lock
*/
```
通过对FileChannel调用tryLock()或lock()，就可以获得整个文件的FileLock。SocketChannel、DatagramChannel和ServerSocketChannel不需要加锁，因为它们是从单进程实体继承而来，通常不在两个进程之间共享socket。tryLock()是非阻塞式的，它设法获取锁，如果不能得到（当其他一些进程已经持有相同的锁，并且不共享时），它将直接从方法调用返回。lock()是阻塞式的，它要阻塞进程直至锁可以获得，或调用lock()的线程中断，或调用lock()的通道关闭。使用FileLock.release()可以释放锁。
也可以对文件部分上锁：
tryLock(long position, long size, boolean shared)或者lock(long position, long size, boolean shared)
其中加锁区域由size-position决定，第三个参数指定是否共享锁。
尽管无参的加锁方法将根据文件尺寸变化而变化，但是具有固定尺寸的锁不随文件变化而变化。
对于独占锁或者共享锁的支持必须有底层的操作系统提供。如操作系统不支持共享锁并未每一个请求都创建锁，那么它就会使用独占锁。锁的类型可以通过FileLock.isShared()进行查询。

#####对映射文件的部分加锁
```
public class LockingMappedFiles {
    static final int LENGTH = 0x8FFFFFF; // 128 MB
    static FileChannel fc;

    public static void main(String[] args) throws Exception {
        fc =
                new RandomAccessFile("test.dat", "rw").getChannel();
        MappedByteBuffer out =
                fc.map(FileChannel.MapMode.READ_WRITE, 0, LENGTH);
        for (int i = 0; i < LENGTH; i++) {
            out.put((byte) 'x');
        }
        new LockAndModify(out, 0, 0 + LENGTH / 3);
        new LockAndModify(out, LENGTH / 2, LENGTH / 2 + LENGTH / 4);
    }

    private static class LockAndModify extends Thread {
        private ByteBuffer buff;
        private int start, end;

        LockAndModify(ByteBuffer mbb, int start, int end) {
            this.start = start;
            this.end = end;
            mbb.limit(end);
            mbb.position(start);
            buff = mbb.slice();
            start();
        }

        public void run() {
            try {
                // Exclusive lock with no overlap:
                FileLock fl = fc.lock(start, end, false);
                System.out.println("Locked: " + start + " to " + end);
                // Perform modification:
                while (buff.position() < buff.limit() - 1) {
                    buff.put((byte) (buff.get() + 1));
                }
                fl.release();
                System.out.println("Released: " + start + " to " + end);
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        }
    }
}/*
Locked: 0 to 50331647
Locked: 75497471 to 113246206
Released: 75497471 to 113246206
Released: 0 to 50331647
*/
```
线程类LocakAndModify创建了缓冲区和用于修改的slice()，然后在run中，获得文件通道上的锁。lock()调用类似于获得一个对象的线程锁——处于临界区，即对该部分的文件具有独占访问权。
如果有Java虚拟机，它会自动释放锁，或者关闭加锁通道。
