---
layout:  post
title:   Java核心技术 接口、lambda表达式与内部类2
date:   2019-11-25 14:16:25
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-Java核心技术

---



## 4.内部类

==内部类（inner class）是定义在另一个类中的类。为什么需要内部类，有三点： 1.内部类方法可以访问该类定义所在的作用域中的数据，包括私有的数据 2.内部类可以对同一个包中的其他类隐藏起来 3.当想要定义一个回调函数且不想编写大量代码时，使用匿名（anonymous）==内部类比较便捷

### 使用内部类访问对象状态

抽象一个TalkingClock类，构造一个语音时钟需要提供两个参数：发布通告的间隔和开关铃声的标志。

```java
public class TalkingClock {
    private int interval;
    private boolean beep;
    
    public TalkingClock(int interval, boolean beep) {...}
    public void start() {...}
    // 内部类
    public class TimePrinter implements ActionListener {
       System.out.println("时间是：" + new Date());
       if (beep) {
           Toolkit.getDefaultToolkit().beep();
       }
    }
}
```

这里的TimePrinter类位于TalkingClock类内部。这并不意味着每个TalkingClock都有一个TimerPrinter类的详细内容。TimerPrinter对象是由TalkingClock类的方法构造。 内部类即可以访问自身的数据域，也可以访问创建它的外围类对象的数据域。 内部类的对象总有一个隐式引用，它指向了创建它的外部类对象。这个引用在内部类的定义中是不可见的。

```java
if (TalkingClock.this.beep) {
    Toolkit.getDefaultToolkit().beep();
}
```

外围类的引用在构造器中设置。编译器修改了所有的内部类的构造器，添加了一个外围类引用的参数。因为TimerPrinter没有定义构造器，所以编译器生成了一个默认构造器：

```java
public TimePrinter(TalkingClock clock) {
	outer = clock;
}
```

```java
public class InnerClassTest {
    public static void main(String[] args) {
        TalkingClock clock = new TalkingClock(1000, false);
        clock.start();
        JOptionPane.showMessageDialog(null, "退出程序？");
        System.exit(0);
    }
}
class TalkingClock {
    private int interval;
    private boolean beep;
    public TalkingClock(int interval, boolean beep) {
        this.interval = interval;
        this.beep = beep;
    }
    public void start() {
        ActionListener listener = new TimePrinter();
        Timer t = new Timer(interval, listener);
        t.start();
    }
    // 内部类
    public class TimePrinter implements ActionListener {
        @Override
        public void actionPerformed(ActionEvent e) {
            System.out.println("时间是：" + new Date());
            if (beep) {
                Toolkit.getDefaultToolkit().beep();
            }
        }
    }
}
```

### 内部类的特殊语法规则

OuterClass.this表示外围类引用，TalkingClock.this.beep。 可以利用下列语法更加明确地编写内部对象的构造器： outerObject.new InnerClass(); 例如：

```java
ActionListener listener = this.new TimePrinter();
```

最新构造的TimerPrinter对象的外围类引用被设置为创建内部类对象的方法中的this引用。this通常是多余的。 可以通过显示地命名将外围类引用设置为其他的对象：

```java
TalkingClock jabberer = new TalkingClock(1000, true);
TalkingClock.TimePrinter listener = jabberer.new TimePrinter();
```

在外围类的作用域之外，可以这样引用内部类： OuterClass.InnerClass

1.内部类中声明的所有静态域都必须是final。我们希望一个静态域只有一个实例，不过对于每个外部对象，会分别有一个单独的内部类实例。如果这个域不是final，它可能不是唯一的。 2.内部类不能有static方法。也可以有静态方法，但只能访问外围类的静态域和方法。

### 内部类是否有用、必要和安全

内部类是一种编译器现象，与虚拟机无关。编译器将会把内部类翻译成分 隔 外 部 类 名 与 内 部 类 名 的 常 规 类 文 件 ， 而 虚 拟 机 则 对 此 一 无 所 知 。 T i m e r P r i n t e r 类 将 被 翻 译 成 类 文 件 T a l k i n g C l o c k 分隔外部类名与内部类名的常规类文件，而虚拟机则对此一无所知。 TimerPrinter类将被翻译成类文件TalkingClock 分隔外部类名与内部类名的常规类文件，而虚拟机则对此一无所知。TimerPrinter类将被翻译成类文件TalkingClockTimePrinter.class。

javap -private TalkingClock.TimePrinter(class文件名)

```java
public class TalkingClock$TimePrinter implements java.awt.event.ActionListener {
  final chapter6.section4.TalkingClock this$0;
  public chapter6.section4.TalkingClock$TimePrinter(chapter6.section4.TalkingClock);
  public void actionPerformed(java.awt.event.ActionEvent);
}
```

编译器为了引用外围类，生成了一个附加的实例域this$0(名字this$0是由编译器合成的，在自己编写的代码中不能引用它)。

能不能自己编写程序实现这种机制呢？

```java
class TalkingClock {
	public void start()
	{
		ActionListener listener = new TalkingClock.TimePrinter();
        Timer t = new Timer(this.interval, listener);
        t.start();
	}
}

class TimePrinter implements ActionListener {
	private TalkingClock outer;
	public TimePrinter (TalkingClock clock) {
		outer = clock;
	}
}
```

调用outer.beep会报错，内部类可以访问外围类的私有数据，但是这里的TimePrinter则不行。所以内部类比常规类比较起来功能更加强大。

内部类被翻译成名字很古怪的常规类，怎么管理那些额外的访问特权？ 查看TalkingClock.class：

```java
public class TalkingClock {
  private int interval;
  private boolean beep;
  public TalkingClock(int, boolean);
  public void start();
  static boolean access$000(TalkingClock);
}
```

编译器在外围类添加了静态方法access$000，它将返回作为参数传递给它的对象域beep。 内部内将调用这个方法： if(TalkingClock.access$000(outer)) 这样存在风险，任何人都可以通过access$000方法很容易地读取到私有域beep。当然access$000不是一个合法的方法名，但是熟悉类文件结构的黑客可以使用十六进制编译器轻松地创建一个用虚拟机指令调用那个方法的类文件。由于隐秘地访问方法需要包可见性，所以攻击代码需要与被攻击类放在同一个包中。 总而言之，如果内部类访问了私有数据域，就有可能通过附加在外围类所在包中的其他类访问它们。

### 局部内部类

TimePrinter这个类名字只在start方法中创建了这个类型的对象时使用了一次，这时可以在方法中定义局部类：

```java
public void start() {
    // 内部类
    class TimePrinter implements ActionListener {
        @Override
        public void actionPerformed(ActionEvent e) {
            System.out.println("时间是：" + new Date());
            if (beep) {
                Toolkit.getDefaultToolkit().beep();
            }
        }
    }
    
    ActionListener listener = new TimePrinter();
    Timer t = new Timer(interval, listener);
    t.start();
}
```

局部类不用public或private访问说明符进行声明。它的作用域被限定在声明这个局部类的块中。 局部类的优势：即对外部世界可以完全地隐藏起来。即使TalkingClock类中的其他代码也不能访问它。除了start方法之外，没有任何方法知道TimePrinter的存在。

#### 由外部方法访问变量

局部内部类可以访问事实上为final的局部变量。

```java
public void start(int interval, boolean beep) {
        // 内部类
        class TimePrinter implements ActionListener {
            @Override
            public void actionPerformed(ActionEvent e) {
                System.out.println("时间是：" + new Date());
                if (beep) {
                    Toolkit.getDefaultToolkit().beep();
                }
            }
        }

        ActionListener listener = new TimePrinter();
        Timer t = new Timer(interval, listener);
        t.start();
    }
```

TalkingClock类不再需要存储实例变量beep，它只是引用start方法中的beep参数变量。 1.调用start方法 2.调用内部类TimePrinter构造器，以便初始化listener 3.将listener引用传递给Timer，开始计时，start方法结束。start方法的beep参数变量不复存在 4.然后actionPerformed方法执行fi(beep) TimePrinter在beep域释放之前将beep域用start方法的局部变量进行备份。

```java
class TalkingClock$1TimePrinter {
	TalkingClock$1TimePrinter(TalkingClock, boolean);
	public actionPerformed(java.awt.event.ActionEvent);
	final boolean val$beep;
	final TalkingClock this$0;
}
```

当创建一个对象的时候，beep就会被传递给构造器，并存储在val$beep域中。编译器必须检测对局部变量的访问，为每一个变量建立相应的数据域，并将局部变量拷贝到构造器中，以便将这些数据域初始化为局部变量的副本。 局部类的方法只可以引用定义为final的局部变量。将beep参数声明为final，对它进行初始化后不能够再进行修改。因此，使得局部变量与在局部类内建立的拷贝保持一致。

final限制显得不是太方便：

```java
int counter = 0;
Date[] dates = new Date[100];
for (int i = 0; i < dates.length; i++) {
    dates[i] = new Date() {
        @Override
        public int compareTo(Date other) {
            counter++;
            return super.compareTo(other);
        }
    };
}
Arrays.sort(dates);
System.out.println(counter + " 比较。");
```

清楚地知道counter需要更新，所以不能将counter声明为final。Integer对象是不可变的，也不能用Integer代替它。补救方法是使用一个长度为1的数组：

```java
int[] counter = new int[1];
Date[] dates = new Date[100];
for (int i = 0; i < dates.length; i++) {
    dates[i] = new Date() {
        @Override
        public int compareTo(Date other) {
            counter[0]++;
            return super.compareTo(other);
        }
    };
}
Arrays.sort(dates);
System.out.println(counter + " 比较。");
```

### 匿名内部类

只创建类的一个对象，而不命名，这种类被称为匿名内部类（anonymous inner class）。

```java
public void start(int interval, boolean beep) {
    ActionListener listener = new ActionListener() {
        @Override
        public void actionPerformed(ActionEvent e) {
            System.out.println("时间是：" + new Date());
            if (beep) {
                Toolkit.getDefaultToolkit().beep();
            }
        }
    };
    Timer t = new Timer(interval, listener);
    t.start();
}
```

创建一个实现ActionListener接口的类的新对象，需要实现的方法actionPerformed定义在括号｛｝内。 通常语法格式： new SuperType(construction paramethers) { inner class methods and data } SuperType可以是接口，于是内部类就要实现接口。也可以是一个类，于是内部类就要扩展它。

由于构造器的名字必须与类名相同，而匿名类没有类名，所以匿名类不能有构造器，取而代之的是将构造器参数传递给超类构造器。 使用匿名内部类的解决方案比较简短，更切实际，更易于理解。 过去Java程序员习惯用匿名内部类实现事件监听器和其他回调。如今最好使用lambda表达式。

```java
public void start(int interval, boolean beep) {
    Timer t = new Timer(interval, e -> {
        System.out.println("时间是：" + new Date());
        if (beep) {
            Toolkit.getDefaultToolkit().beep();
        }
    });
    t.start();
}
```

双括号初始化：

```java
public static void main(String[] args) {
    ArrayList<String> friends = new ArrayList<>();
    friends.add("Harry");
    friends.add("Tony");
    invite(friends);
    invite(new ArrayList<String>() {
        {
            add("Tom");
            add("Mack");
        }
    });
}
private static void invite(ArrayList<String> friends) {
    System.out.println(friends);
}
```

外层括号简历ArrayList的一个匿名子类。内层括号是一个对象的构造块。

建立一个与超类大体类似的匿名子类通常会很方便。但是对于equals方法要当心。 if(getClass != other.getClass()) return false;对匿名子类做这个测试时会失败。

```java
public class InnerClassTest {
    public static void main(String[] args) {
        invite();
    }

    public static void invite() {
        System.out.println(new Object(){}.getClass().getEnclosingClass());
    }
}
```

静态方法无法使用getClass方法，因为调用getClass时调用的是this.getCalss()，而静态方法没有this。 所以可以使用 new Object(){}.getClass().getEnclosingClass()，在这里new Object(){}会建立Object的一个匿名子类的一个匿名对象，getEnclosingClass则得到其外围类。

### 静态内部类

有时候，使用内部类只是为了把一个类隐藏在另一个类内部，并不需要内部类引用外围类对象。此时可以将内部类声明为static，以便取消产生的引用。 只遍历一次数组，取出最大最小值：

```java
public class StaticInnerClassTest {
    public static void main(String[] args) {
        double[] d = new double[20];
        for (int i = 0; i < d.length; i++) {
            d[i] = 100 * Math.random();
        }
        ArrayAlg.Pair p = ArrayAlg.minmax(d);
        System.out.println("最小：" + p.getFirst());
        System.out.println("最大：" + p.getSecond());
    }
}
class ArrayAlg {
    public static class Pair {
        private double first;
        private double second;
        public Pair(double f, double s) {
            first = f;
            second = s;
        }
        public double getFirst() {
            return first;
        }
        public double getSecond() {
            return second;
        }
    }
    public static Pair minmax(double[] values) {
        double min = Double.POSITIVE_INFINITY;
        double max = Double.NEGATIVE_INFINITY;
        for (double v : values) {
            if (min > v) {
                min = v;
            }
            if (max < v) {
                max = v;
            }
        }
        return new Pair(min, max);
    }
}
```

Pair是一个大众化的名字，可以将其定义为内部类，同时它不需要引用任何其他的对象，为此声明为static。静态内部类的对象除了没有对生成它的外围类对象的引用特权外，与其他所有内部类完全一样。

如果Pair类没有声明为static：

```java
public static Pair minmax(double[] values) {
        ...
        return new Pair(min, max);
    }
```

编译器会给出错误报告：没有可用的隐式ArrayAlg类型对象初始化内部类对象。

与常规内部类不同，静态内部类可以有静态域和方法。 声明在接口中的内部类自动成为public和static类。

## 代理

利用代理（proxy），可以在运行时创建一个实现了一组给定接口的新类。这种功能只有在编译时无法确定需要实现哪个接口时才有必要使用。

### 如何使用代理

代理类可以在运行时创建全新的类，这样的代理类能够实现指定的接口： 1.指定接口所需的全部方法 2.Object类的全部方法 然而不能在运行时定义这些方法的新代码，而是要提供一个调用处理器（invocation handler）。调用处理器是实现了InvocationHandler接口的类对象。这个接口只有一个方法： Object invoke(Object proxy, Method method, Object[] args) 无论何时调用代理对象的方法，调用处理器的invoke方法都会被调用，并向其传递Method对象和原始的调用参数。调用处理器必须给出处理调用的方式。

### 创建代理对象

要想创建代理对象，需要使用Proxy类的newProxyInstance方法，这个方法有三个参数： 1.一个类加载器 2.一个Class对象数组，每个元素都是需要实现的接口 3.一个调用处理器

使用代理和调用处理器跟踪方法调用，并且定义了一个TraceHander包装器类存储包装对象。其中invoke方法打印出被调用方法的名字和参数，随后用包装好的对象作为隐式参数调用这个方法。

```java
public class ProxyTest {
    public static void main(String[] args) {
        Object[] elements = new Object[1000];

        for (int i = 0; i < elements.length; i++) {
            Integer value = i + 1;
            // 构造用于跟踪方法调用的代理对象
            InvocationHandler handler = new TraceHandler(value);
            Object proxy = Proxy.newProxyInstance(null, new Class[] {Comparable.class}, handler);
            // 无论何时用 proxy 调用某个方法， 这个方法的名字和参数就会打印出来， 之后再用 value 调用它
            elements[i] = proxy;
        }

        Integer key = new Random().nextInt(elements.length) + 1;

        int result = Arrays.binarySearch(elements, key);
        if (result >= 0) {
            System.out.println(elements[result]);
        }
    }
}

class TraceHandler implements InvocationHandler {

    private Object target;

    public TraceHandler(Object t) {
        target = t;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.print(target);
        System.out.print("." + method.getName() + "(");
        if (args != null) {
            for (int i = 0; i < args.length; i++) {
                System.out.print(args[i]);
                if (i < args.length - 1) {
                    System.out.print(", ");
                }
            }
        }
        System.out.println(")");
        return method.invoke(target, args);
    }
}
```

Integer类实现了Comparable接口。代理对象属于在运行时定义的类，这个类也实现了Comparable接口，然而它的compareTo方法调用了代理对象处理器的invoke方法。在运行时，所有的泛型类都被取消，代理将它们构造为原Comparable类的类对象。

Arrays类的binarySearch方法是使用二分查找，在数组中的一个元素。

binarySearch方法调用： if (elements[i].compareTo(key) &lt; 0)… 由于数组中填充了代理对象， 所以compareTo调用了TraceHander类中的invoke方法。这个方法打印方法名和参数，之后用包装好的Integer对象调用compareTo。

### 代理类的特性

代理类在程序运行过程中创建，一旦被创建，就变成了常规类，与虚拟机中的其他任何类没有什么区别。 所有代理类都扩展于Proxy类。一个代理类只有一个实力域——调用处理器。它定义在Proxy的超类中。为了履行代理对象的职责，所需要的任何附加数据都必须存储在调用处理器中。 所有代理类都覆盖了Object类的方法toString、equals和hashCode。这些方法仅仅调用了调用处理器的invoke。Object类中的其他方法（clone和getClass）没有被重新定义。 没有定义代理类的名字，Sun虚拟机中的Proxy类将生成一个以字符串$Proxy开头的类名。

代理类一定是public和final。如果代理类实现的所有接口都是public，代理类就不属于某个特定的包。否则，所有非公有的接口都必须属于同一个包，同时代理类也属于这个包。

