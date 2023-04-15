## 1.为什么要使用泛型程序设计

==泛型设计程序（Generic programming）==意味着编写的代码可以被不同类型的对象所重用。

### 类型参数的好处

在Java增加泛型类之前，泛型程序设计用继承来实现。ArrayList类只维护一个Object引用的数组：

```java
// 引入泛型之前的ArrayList
public class ArrayList {
	private Object[] elementData;
	...
	public Object get(int i) {...}
	public void add(Object o) {...}
}
```

这样有两个问题，当获取一个值时必须进行强制类型转换：

```java
ArrayList files = new ArrayList();
String filename = (String)files.get(0);
```

此外没有错误检查，可以向数组添加任何类的对象。编译和运行不会出错，如果get的结果强制类型转换成其他类型，就会产生一个错误。

泛型提供了更好的解决方案：类型参数（type parameters）：

```java
ArrayList<String> files = new ArrayList<String>();
```

在Java SE 7之后的版本构造函数中可以省略泛型类型new ArrayList&lt;&gt;()。 编译器知道ArrayList&lt; String &gt;add方法有一个类型为String的参数，避免插入错误的对象（无法通过编译）。 类型参数的魅力在于：使得程序具有更好的可读性和安全性。

### 谁想成为泛型程序员

一个泛型程序员的任务就是预测出所有类的未来可能有的所有用途。 可以将ArrayList&lt; Manager &gt;中的所有元素添加到ArrayList&lt; Employee &gt;中去，但是反过来就不行。Java语言有一个独创性的新概念通配符类型（wildcard type），解决了这个问题。

## 2.定义简单泛型类

==泛型类（generic class）==就是具有一个或多个类型变量的类。

```java
public class Pair<T> {
    private T first;
    private T second;
    
    public Pair() {
        first = null;
        second = null;
    }
    
	public Pair(T first, T second) {
        this.first = first;
        this.second = second;
    }
    
    public T getFirst() {
        return first;
    }
    public void setFirst(T first) {
        this.first = first;
    }
    public T getSecond() {
        return second;
    }
    public void setSecond(T second) {
        this.second = second;
    }
}
```

Pair类引入了一个类型变量T，用==尖括号（&lt; &gt;）==括起来，放在类名后面。泛型类可以引入多个类型变量：

```java
public class Pair<T, U> {...}
```

类定义中的类型变量指定方法的返回类型以及域和局部变量的类型。 类型变量使用大写形式，且比较短。Java库中，使用变量E表示集合的元素类型，K和V表示关键字和值的类型。T（有时是临近的U和S）便是任意类型。

用具体的类型可以替换掉类型变量就可以实例化泛型类型（带构造器和方法的普通类）： Pair&lt; String &gt; Pair(String, String) String getFirst() String getSecond() void setFirst(String) void setSecond(String) 泛型类可以看做普通类的工厂。

之前使用内部类实现的同时比较最大最小值的方法，可以使用泛型：

```java
public class PairTest1 {
    public static void main(String[] args) {
        String[] words = { "Mary", "had", "a", "little", "lamb" };
        Pair<String> mm = ArrayAlg.minmax(words);
        System.out.println("最小值：" + mm.getFirst());
        System.out.println("最大值：" + mm.getSecond());
    }
}
class ArrayAlg {
    public static Pair<String> minmax(String[] a) {
        if (a == null || a.length == 0) {
            return null;
        }
        String min = a[0];
        String max = a[0];
        for (int i = 0; i < a.length; i++) {
            if (min.compareTo(a[i]) > 0) {
                min = a[i];
            }
            if (max.compareTo(a[i]) < 0) {
                max = a[i];
            }
        }
        return new Pair<>(min, max);
    }
}
```

## 3.泛型方法

可以定义一个带有类型参数的简单方法：

```java
calss ArrayAlg {
	public static <T> T getMiddle(T... a) {
		return a[a.length / 2];
	}
}
```

类型变量放在修饰符的后面，返回类型的前面。泛型方法可以定义在普通类中，也可以定义在泛型类中。 调用时在方法名前面的尖括号中放入具体类型：

```java
String middle = ArrayAlg..<String>getMiddle("John", "Q", "public");
```

大多数情况下可省略&lt; String &gt;类型参数。编译器有足够的信息能够推断出所调用的方法。用Strintg[]与泛型类型T[]进行匹配并推断出T一定是String。 大多数情况下没有问题，偶尔编译器也会有提示错误：

```java
double m = ArrayAlg.getMiddle(3.14, 1729, 0);
```

错误信息：解释这句代码有两种方法，而且两种方法都是合法的。1个Double和2个Integer对象，而后寻找共同的超类型，找到了两个超类型：Number和Comparable接口，其本身也是一个泛型类型。这时候我们需要补救措施将所有参数写为double值。

## 4.类型变量的限定

有时，类或方法需要对类型遍历加以约束。

```java
public static <T extends Comparable> T min(T[] a) {
    if (a == null || a.length == 0) {
        return null;
    }
    T smallest = a[0];
    for (int i = 0; i < a.length; i++) {
        if (smallest.compareTo(a[i]) > 0) {
            smallest = a[i];
        }
    }
    return smallest;
}
```

为了确信变量T具有conpareTo方法，将T限制为实现了Comparable接口（只含一个方法compareTo方法）的类。实际上Comparable接口本身就是一个泛型类型。现在，泛型的min方法只能被实现了Comparable接口的类的数组调用，否则会产生编译错误。 ==&lt; T extends BoundingType &gt;==表示T应该是绑定类型的子类型。T和绑定类型可以是类，也可以是接口。选择extends关键字是更接近子类的概念，而非implements，并且Java设计者并不打算添加新的关键字。 一个类型变量或通配符可以有多个限定，限定类型用“&amp;”分隔，而逗号用来分隔类型变量：

```java
T extends Comparable & Serializable
```

## 5.泛型代码和虚拟机

虚拟机没有泛型类型对象——所有对象都属于普通类。

### 类型擦除

无论何时定义一个泛型类型，都自动提供了一个相应的原始类型（raw type）。原始类型的名字就是删去类型参数后的泛型类型名。擦除（erased）类型变量，并替换为限定类型（无限定的变量用Object）。

```java
public class Pair {
	private Object first;
	private Object second;
	...
}
```

因为T是一个无限定的变量，所以直接用Object替换。结果是一个普通的类，就好像泛型引入Java语言之前已经实现的那样。 在程序中可以包含不同类型的Pair。而擦除类型后就变成原始的Pair类型了。原始类型用第一个限定的类型变量来替换，如果没有给定限定就用Object替换。

```java
public class Interval <T extends Comparable & Serializable> implements Serializable {
    private T lower;
    private T upper;
    
    public Interval(T first, T second) {
        if (first.compareTo(second) <= 0) {
            lower = first;
            upper = second;
        } else {
            lower = second;
            upper = first;
        }
    }
}
```

原始类型Interval如下：

```java
public class Interval implements Serializable {
    private Comparable  lower;
    private Comparable  upper;
    
    public Interval(Comparable first, Comparable second) {...}
}
```

如果class Interval &lt;T extends Serializable &amp; Comparable&gt;，原始类型用Serializable替换T，而编译器在必要时要向Comparable插入强制类型转换。为了提高效率，应该将标签（tagging）接口（即没有方法的接口）放在边界列表的末尾。

### 翻译泛型表达式

当程序调用泛型方法时，如果擦除返回类，编译器插入强制类型转换。

```java
Pair<Employee> buddies = ...;
Employee buddy = buddies.getFirst();
```

擦除getFirst的返回类型后将返回Object类型。编译器自动插入Employee的强制类型转换。编译器将这个方法调用翻译为两条虚拟机指令： 1.对原始方法Pair.getFirst的调用 2.将返回的Object类型强制转换为Employee类型

当存取一个泛型域时也要插入强制类型转换。

### 翻译泛型方法

泛型方法看似是一个完整的方法族，而擦除方法之后，只剩下一个方法：

```java
public static Comparable min(Comparable[] a)
```

方法擦除带来了两个复杂问题：

```java
class DateInterval extends Pair<LocalDate> {
	public void setSecond(LocalDate second) {
		if (second.compareTo(getFirst()) >= 0) {
			super.setSecond(second);
		}
	}
}
```

擦除后变成：

```java
class DateInterval extends Pair {
	public void setSecond(LocalDate second){...}
}
```

存在另一个Pair继承来的setSecond方法： public void setSecond(Object second) 这显然是一个不同的方法，因为它有一个不同类型的参数——Object，而不是LocalDate。考虑下面语句：

```java
DateInterval interval = new DateInterval(...);
Pair<LocalDate> pair = interval;
pair.setSecond(aDate);
```

这里希望对setSecond的调用具有多态性，并调用最合适的那个方法。由于pair引用DateInterval对象，所以应该调用dateInterval.setSecond。问题在于类型擦除与多态发生了冲突。编译器在DateInterval类中生成了一个桥方法（bridge method）：

```java
public void setSecond(Object second) {setSecond((Date) second);}
```

变量pair已经声明为类型Pair&lt; LocalDate &gt;，并且这个类型只有一个简单方法叫setSecond(Object)。虚拟机用pair引用对象调用方法。这个对象是DateInterval类型，因而将会调用DateInterval.setSecond(Object)方法，即合成的桥方法。

假设：

```java
class DateInterval extends Pair<LocalDate> {
	public LocalDate getSecond() {
		return (Date)super.getSecond().clone();
	}
}
```

DateInterval类中有两个getSecond方法： LocalDate getSecond() Object getSecond() 在虚拟机中用参数类型和返回类型确定一个方法。因此编译器可能产生两个仅返回类型不同的方法字节码，虚拟机能够正确处理。但不能这么编写Java代码（具有相同参数类型的两个方法是不合法的），它们都没有参数。

桥方法不仅用于泛型类型。在一个方法覆盖另一个方法时可以指定一个更严格的返回类型：

```java
public class Employee implements Cloneable{
	public Employee clone() {...}
}
```

Object.clone和Employee.clone方法被说成具有协变的返回类型（covariant return types），实际上Employee类有两个克隆方法： Employee clone() Object Clone() 合成的桥方法调用了新定义的方法。

总之，需要记住有关Java泛型转换的事实： 1.虚拟机中没有泛型，只有普通类和方法 2.所有的类型参数都用他们的限定类型替换 3.桥方法被合成来保持多态 4.为保持类型安全性，必要时插入强制类型转换

### 调用遗留代码

设计Java泛型类型时，主要目标是允许泛型代码和遗留代码之间能够互操作。

```java
Dictionary<Integer, Component> labelTable = new Hashtable<>()
labelTable.put(0, new JLabel(new ImageIcon("nine.gif")));
labelTable.put(1, new JLabel(new ImageIcon("ten.gif")));
JSlider jSlider = new JSlider();
jSlider.setLabelTable(labelTable);
@SuppressWarnings("unchecked")
Dictionary<Integer, Component> lt = jSlider.getLabelTable();
```

调用setLabelTable时，比编译器会发出一个警告。这个警告不会产生什么影响，因此可以忽略。 相反的情形，由一个遗留的类得到一个原始类型的对象Dictionary&lt;Integer, Component&gt; lt = jSlider.getLabelTable()，或看到一个警告，确保标签表已经包含了Integer和Component对象，当然从来也不会有绝对的承诺，最差的情况将抛出一个异常。 在查看警告之后，可以利用==注解(annotation)==使之消失，@SuppressWarnings(“unchecked”)。 注解必须放在生成这个警告的代码所在方法之前，或者标注整个方法（关闭对方法所有代码的检查）。

## 6.约束与限制性

Java泛型时需要考虑的一些限制，大多数限制由类型擦除引起。

### 不能用基本类型实例化类型参数

不能用类型参数代替基本类型。因此没有Pair&lt; double &gt;，只有Pair&lt; Double &gt;。其原因是类型擦除之后，Pair类含有Object类型的域，而Object不能存储double值。

### 运行时类型查询只适用于原始类型

虚拟机中的对象有一个特定的非泛型类型。因此所有的类型查询只产生原始类型。

```java
if (a instanceof Pair<String>) // Error
if (a instanceof Pair<T>) // Error
Pair<String> p = (Pair<String>) a; // 警告，只能测试a是一个Pair
```

试图查询一个对象是否属于某个泛型类型时，使用instanceof会得到一个编译器错误，如果使用强制类型转换会得到一个警告。 同样的，getCalss方法总是返回原始类型：

```java
Pair<String> stringPair = ...;
Pair<Employee> employeePair = ...;
if (stringPair.getClass() == employeePair.getClass) // 相等
```

比较结果为true，这是因为两次调用getClass都返回Pair.class。

### 不能创建参数化类型的数组

```java
Pair<String>[] table = new Pair<String>[10]; // Error
```

擦除之后table类型是Pair[]，可以转换成Object[]：Object[] objarray = table; 数组会记住它的元素类型，如果试图存储其他类型的元素，就会抛出一个ArrayStoreException异常：

```java
Pair<String>[] table = new Pair[10];
objarray[0] = "Hello"; // Error,类型是Pair
```

不过对于泛型类型，擦除会使这种机制无效。

```java
objarray[0] = new Pair<Employee>();
```

能够通过数组存储检查，不过仍会导致一个类型错误。出于这个原因，不允许创建参数化类型的数组。而声明类型为Pair&lt; String &gt;的变量仍是合法的。不过不能用new Pair&lt; String &gt;[10]初始化这个变量。

可以声明通配符类型的数组，然后进行类型转换：

```java
Pair<String>[] table = (Pair<String>[]) new Pair<?>[10];
```

结果不安全，如果table[0]中存储一个Pair&lt; Employee &gt;，然后对table[0].getFirst()调用一个String方法，会得到一个ClassCastException异常。如果需要收集参数化类型的对象，使用ArrayList&lt; Pair&lt; String &gt; &gt;更加安全有效。

### Varargs警告

向参数个数可变的方法传递一个泛型类型的实例：

```java
public static <T> void addAll(Collection<T> coll, T... ts){
	for (t : ts) {coll.add(t);}
}
```

实际上参数ts是一个数组，为了调用这个方法，Java虚拟机必须建立一个Pair&lt; String &gt;数组，这违反了前面的规则。但这种情况只会得到一个警告，而不是错误。 有两种方法抑制警告，一种是注解@SupperssWarning(“unckecked”)。或者在Java SE 7中，还可以用@SafeVarargs直接标注。

### 不能实例化类型变量

不能使用像new T(…)，new T[…]或T.class这样的表达式中的类型变量。 在Java SE 8之后，最好的解决办法是让调用者提供一个构造器表达式：

```java
Pair<String> p = Pair.makePair(String::new);

public static <T> Pair<T> makePair(Supplier<T> constr) {
    return new Pair<>(constr.get(), constr.get());
}
```

Supplier&lt; T &gt;是一个函数式接口，表示一个无参数而且返回类型为T的函数。

比较传统的解决方法是反射调用Class.newInstance方法来构造泛型对象。

```java
Pair<String> p = Pair.makePair(String.class);

public static <T> Pair<T> makePair(Class<T> cl) {
    try {
        return new Pair<>(cl.newInstance(), cl.newInstance());
    } catch (Exception e) {
        return null;
    }
}
```

Class类本身是泛型。例如，String.class是一个Class&lt; String &gt;的实例（事实上，它是唯一的实例）。因此，makePair方法能够推断出pair 的类型。

### 不能构造泛型数组

就像不能实例化一个泛型实例一样，也不能实例化数组：

```java
public static <T extends Comparable> T[] minmax(T[] a) {
	T[] mm = new T[2]; // Error,类型参数“T”不能直接实例化
	...
}
```

类型擦除会让这个方法永远构造Comparable[2]数组。 如果数组仅仅做为一个类的私有域，就可以将数组声明为Object[]，并且在获取元素时进行类型转换。 自己实现ArrayList类：

```java
public class ArrayList<E> {
    private Object[] emements;
    
    @SuppressWarnings("unchecked")
    public E get(int n) {
        return (E) emements[n];
    }
    
    public void set(int n, E e) {
        emements[n] = e;
    }
}
```

实际上的ArrayList的实现没有这么清晰：

```java
public class ArrayList<E> {
	private E[] elements;
	public ArrayList() {
		elements = (E[]) new Object[10];
	}
}
```

这里的强制类型E[]是一个假象，而类型擦除使其无法察觉。

由于minmax方法返回T[]数组，使得这一技术无法施展，如果掩盖这个类型会有运行时错误结果：

```java
public static <T extends Comparable> T[] minmax(T... a) {
	Object[] mm = new Object[2];
	...
	return (T[]) mm; // 编译时警告
}
```

调用该方法时候，编译时不会有任何警告。当Object[]引用赋值给Comparable[]变量时，将会发生ClassCastException异常。

在这种情况下，最好让用户提供一个数组构造器表达式，构造器表达式String::new指示一个函数，给定所需的长度，会构造一个指定长度的String数组：

```java
String[] ss = ArrayAlg.minmax(String[]::new, "Tom", "Dick", "Harry");

public static <T extends Comparable> T[] minmax(IntFunction<T[]> constr, T... a) {
    T[] mm = constr.apply(2);
    return mm;
}
```

比较老式的方法是利用反射，调用Array.newInstance:

```java
public static <T extends Comparable> T[] minmax(T... a) {
    T[] mm = (T[]) Array.newInstance(a.getClass().getComponentType(), 2);
    return mm;
}
```

ArrayList类的toArray方法，需要生成一个T[]数组，但没有成分类型，因此有两种不同的形式： Object[] toArray() T[] toArray(T[] result) 第二个方法接收一个数组参数。如果数组足够大，就是用数组。否则，用result的成分类型构造一个足够大的新数组。

### 泛型类的静态上下文中类型变量无效

不能在静态域或方法中引用类型变量。

```java
public class Singleton<T> {
    private static T singleInstance; //Error
    
    public static T getSingleInstance() { //Error
        if (singleInstance == null) {
            return singleInstance;
        }
    }
}
```

类型擦除后，只剩下Singleton类，它只包含一个singleInstance域。因此，禁止使用带有类型变量的静态域和方法。

### 不能抛出或捕获泛型类的实例

既不能抛出也不能捕获泛型类对象。实际上，甚至泛型类扩展Throwable都是不合法的。 catch子句不能使用类型变量。

```java
try{
} catch (T e) { // Error,无法捕获类型变量
}
```

不过，在异常规范中使用类型变量是允许的：

```java
public static <T extends Throwable> void doWork(T t) throws T {
    try {
        
    } catch (Throwable realCause) {
        t.initCause(realCause);
        throw t;
    }
}
```

### 可以消除对受检异常的检查

Java异常处理的一个基本原则是，必须为所有受检异常提供一个处理器。不过可以利用泛型消除这个限制：

```java
public class Block {
@SuppressWarnings("unchecked")
public static <T extends Throwable> void throwAs(Throwable e) throws T {
    throw (T) e;
}
}
```

被调用该方法：

```java
try {
} catch(Throwable t) {
	Block.<RuntimeException>throwAs(t);
}
```

编译器会认为t是一个非受检异常，会把所有异常都转化为编译器所认为的非受检异常。

```java
public abstract class Block {
    public abstract void body() throws Exception;
    public Thread toThread() {
        return new Thread() {
            @Override
            public void run() {
                try {
                    body();
                } catch (Throwable t) {
                    Block.<RuntimeException>throwAs(t);
                }
            }
        };
    }
    @SuppressWarnings("unchecked")
    public static <T extends Throwable> void throwAs(Throwable e) throws T {
        throw (T) e;
    }
    public static void main(String[] args) {
        new Block() {
            @Override
            public void body() throws Exception {
                Scanner in = new Scanner(new File("ququx"), "UTF-8");
                while (in.hasNext()) {
                    System.out.println(in.next());
                }
            }
        }.toThread().start();
    }
}
```

正常情况下，必须捕获线程run方法中的所有受检异常，把他们包装到非受检异常中，因为run方法声明为不抛出任何受检异常。 不过这里并没有做包装，只是抛出异常，并哄骗编译器，让它认为这不是一个受检异常。

### 注意擦除后的冲突

假设将equals方法添加到Pair类中，方法擦除后boolean equals(T)就是boolean equals(Object)与Object.equals方法发生冲突。 补救的方法是重新命名引发错误的方法。要想支持擦除的转换，就需要强行限制一个类或类型变量不能同时成为两个接口类型的子类，而这两个接口是同一接口的不同参数化。

下述代码是非法的：

```java
class Employee implements Comparable<Employee> {...}
class Manager extends Employee implements Comparable<Manager> {...} //Error
```

Manager实现了同一接口的不同参数化。 实现了Comaprable&lt; X &gt;的类可以获得一个桥方法： public int comparaTo(Object ohter){return compareTo((x) other）;} 对于不同类型的X不能有两个这样的方法。

## 7.泛型类型的继承规则

Pair&lt; Manager &gt;并不是Pair&lt; Employee &gt;：

```java
Manager[] topHonchos = ...;
Pair<Employee> result = ArrayAlg.minmax(topHonchos );  // Error
```

无论S与T有什么联系，通常Pair&lt; S &gt;与Pair&lt; T &gt;没有什么联系。

```java
Pair<Manager> managerBuddies = new Pair(ceo, cfo);
Pair<Employee> employeeBuddies = managerBuddies; // 不合法的，假设可以
employeeBuddies.setFirst(lowlyEmployee);
```

显然最后一句是合法的，但是employeeBuddies和managerBuddies引用同样的对象。现在将CFO和一个普通员工组成一对，这对于Pair&lt; Manager &gt;来说不太可能。 对于数组来说，数组具有特别的保护，如果将一个低级别雇员存储到employeeBuddies[0]，虚拟机将会抛出ArrayStoreException：

```java
Manager[] managerBuddies = { ceo, cfo };
Employee[] employeeBuddies = managerBuddies; // OK
```

永远可以将参数化类型转换为一个原始类型。例如Pair&lt; Employee &gt;是原始类型Pair的一个子类型。在与遗留代码衔接时，这个转换非常必要。

```java
Pair<Manager> managerBuddies = new Pair<>(ceo, cfo);
Pair rawBuddies = managerBuddies; // OK
rawBuddies.setFirst(new File("...")); // 编译时警告
```

当使用getFirst获得外来对象并赋值给Manager变量时，与通常一样，会抛出ClassCastException异常。这里失去的只是泛型程序设计提供的附加安全性。


泛型类可以扩展或实现其他的泛型类。例如Arraylist&lt; T &gt;类实现List&lt; T &gt;接口，一个ArrayList&lt; Manager &gt;可以被转换为一个List&lt; Manager &gt;，但是ArrayList&lt; Manager &gt;不是一个ArrayList&lt; Employee&gt;或List&lt; Employee&gt;。![img](https://img-blog.csdnimg.cn/20191203133817800.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

## 8.通配符类型

### 通配符概念

通配符类型中，允许类型参数变化： Pair&lt; ? extends Employee &gt;表示任何泛型Pair类型，它的类型参数是Employee的子类。

```java
public static void printBuddies(Pair<Employee> p){...}
```

不能将Pair&lt; Manager &gt;传递给这个方法。解决方案很简单，使用通配符类型，类型&lt; Manager &gt;是Pair&lt; ? extends Employee &gt;的子类型：

```java
public static void printBuddies(Pair<? extends Employee> p){...}
```


![img](https://img-blog.csdnimg.cn/20191203152604680.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) 通配符不会引起破坏：

```java
Pair<Manager> managerBuddies = new Pair<>(ceo, cfo);
Pair<? extends Employee> wildcardBuddies = managerBuddies; // OK
wi1dcardBuddies.setFirst(1owlyEnployee); // 编译时错误
```

类型Pair&lt; ? extends Employee &gt;，其方法看起来是： ？ extends Employee getFirst() void setFirst(？ extends Employee) 这样将不能调用setFirst，编译器只知道需要某个Employee的子类型，但不知道具体是什么类型。它拒绝传递任何特定类型。 但将getFirst的返回值赋值给一个Employee的引用完全合法。

### 通配符的超类型限定

超类型限定（supertype bound） ? super Manager，这个通配符限制为Manager的所有超类型。 void setFirst(? super Manager) ? super Manager getFirst() setFirst不能接受类型为Employee或Object的参数，只能传递Manager类型的对象，或者某个子类型对象。另外如果调用getFirst，不能保证返回对象的类型，只能把它赋值给一个Object。

```java
public static void minmaxBonus(Manager[] a, Pair<? super Manager> result) {
    if (a.length == 0) {
        return;
    }
    Manager min = a[0];
    Manager max = a[0];
    for (int i = 1; i < a.length; i++)
    {
        if (min.getBonus() > a[i].getBonus()) {
            min = a[i];
        }
        if (max.getBonus() < a[i].getBonus()) {
            max = a[i];
        }
    }
    result.setFirst(min);
    result.setSecond(max);
}
```

直观地讲，带有超类型限定的通配符可以向泛型对象写入，带有子类型限定的通配符可以从泛型对象读取。


![img](https://img-blog.csdnimg.cn/20191203152626149.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

超类型限定的另一种应用，由于Comaprable接口本身也是一个泛型类型，那可以把ArrayAIg类的min方法声明为：

```java
public static <T extends Comparable<T>> T min(T[] a)
```

如果计算一个String数组的最小值，T就是String类型的，而String是Comparable&lt; String &gt;的子类型。但是LocalDate实现了ChronoLocalDate，而ChronoLocalDate扩展了Comparable&lt; ChronoLocalDate &gt;。因此Local实现的是Comparable&lt; ChronoLocalDate &gt;而不是Comparable&lt; LocalDate &gt;。 这种情况下，超类型可以：

```java
public static <T extends Conparable<? super T>> T min(T[] a) ...
```

现在compareTo方法写成int compareTo(? super T)，有可能被声明为使用类型T的对象，也有可能使用T的超类型。无论如何，传递一个T类型的对象给compareTo方法都是安全的。

子类型限定的另一个常见用法是作为一个函数式接口的参数类型。 Collection接口有一个方法：

```java
default boolean removeIf(Predicate<? super E> filter)
```

这个方法会删除所有满足给定谓词条件的元素。

```java
ArrayList<Employee> staff = ...;
Predicate<Object> oddHashCode = obj -> obj.hashCode() % 2 != 0;
staff.removeIf(oddHashCode );
```

希望传入一个Predicate&lt; Object &gt;，而不只是Predicate&lt; Employee &gt;。super通配符可以是愿望成真。

### 无限定通配符

Pair&lt; ? &gt;： ？ getFirst() void setFirst(?) getFirst的返回值只能给一个Object。setFirst方法不能被调用，甚至不能用Object调用，可以调用setFirst(null)。 Pair&lt; ? &gt;和Pair本质的不同在于：可以用任意Object对象调用原始Pair类的setObject方法。

它对于许多简单的操作非常有用，测试一个pair是否包含一个null引用，它不需要实际的类型：

```java
public static boolean hasNulls(Pair<?> p) {
	return p.getFirst() == null || p.getSecond() == null;
}
```

通过将hasNulls转换成泛型方法，可以避免使用通配符类型：

```java
public static <T> boolean hasNulls(Pair<T> p)
```

但是通配符的版本可读性更强。

### 通配符捕获

通配符不是类型变量，不能在代码中使用？做为一种类型：

```java
? t = p.getFirst(); // Error
```

需要一个辅助方法swapHelper：

```java
public static <T> void swapHelper(Pair<T> p) {
	T t = p.getFirst();
	p.setFirst(p.getSecond());
	p.setSecond(t);
}

public static void swap(Pair<?> p) { 
	swapHelper(p); 
}
```

swapHelper方法的参数T捕获通配符。它不知道是哪种类型的通配符，但是这是一个明确的类型，并且&lt; T &gt;swapHelper的定义在T指出类型时才有明确的含义。 通配符捕获只有在许多限制的情况下才是合法的。编译器必须能够确信通配符表达的是单个、确定的类型。例如ArrayList&lt; Pair&lt; T &gt; &gt;中的T永远不能捕获ArrayList&lt; Pair &lt; ? &gt; &gt;中的通配符。数组列表可以保存两个Pair&lt; ？ &gt;分别针对？的不同类型。

```java
public class PairTest3 {
    public static void main(String[] args) {
        Manager ceo = new Manager("Gus Greedy", 800000, 2019, 12, 03);
        Manager cfo = new Manager("Sid Sneaky", 600000, 2019, 12, 03);
        Pair<Manager> buddies = new Pair<>(ceo, cfo);
        printBuddies(buddies);
        ceo.setBonus(1000000);
        cfo.setBonus(500000);
        Manager[] managers = {ceo, cfo};
        Pair<Employee> result = new Pair<>();
        minmaxBonus(managers, result);
        System.out.println("第一：" + result.getFirst().getName() + "，第二：" + result.getSecond().getName());
        maxminBonus(managers, result);
        System.out.println("第一：" + result.getFirst().getName() + "，第二：" + result.getSecond().getName());
    }
    public static void printBuddies(Pair<? extends Employee> p) {
        Employee first = p.getFirst();
        Employee second = p.getSecond();
        System.out.println(first.getName() + " 和 " + second.getName() + "是朋友。");
    }
    public static void minmaxBonus(Manager[] a, Pair<? super Manager> result) {
        if (a.length == 0) {
            return;
        }
        Manager min = a[0];
        Manager max = a[0];
        for (int i = 1; i < a.length; i++) {
            if (min.getBonus() > a[i].getBonus()) {
                min = a[i];
            }
            if (max.getBonus() < a[i].getBonus()) {
                max = a[i];
            }
        }
        result.setFirst(min);
        result.setSecond(max);
    }
    public static void maxminBonus(Manager[] a, Pair<? super Manager> result) {
        minmaxBonus(a, result);
        PairAlg.swapHelper(result);
    }
}
class PairAlg {
    public static boolean hasNulls(Pair<?> p) {
        return p.getFirst() == null || p.getSecond() == null;
    }
    public static void swap(Pair<?> p) {
        swapHelper(p);
    }
    public static <T> void swapHelper(Pair<T> p) {
        T t = p.getFirst();
        p.setFirst(p.getSecond());
        p.setSecond(t);
    }
}
```

## 9.反射和泛型

### 泛型Class类

Class类是泛型的，String.class实际上是一个Class&lt; String &gt;类的对象（唯一对象）。

### 使用Class< T >参数进行类型匹配

```java
public static <T> Pair<T> makePair(Class<T> c) throws InstantiationException, IllegaAccessException {
	return new Pair<>(c.newInstance(), c.newInstance());
}
```

调用makePair(Employee.class)，Employee.class是类型Class&lt; Employee &gt;的一个对象。makePair方法的类型参数T同Employee匹配，并且编译器可以推断出这个方法将返回一个Pair&lt; Employee &gt;。

### 虚拟机中的泛型类型信息

Java泛型的卓越特性之一是在虚拟机中泛型类型的擦除。但擦除的类仍然保留一些泛型祖先的微弱记忆。

```java
public static <T extends Comparable<? super T>> T min (T[] a)
```

擦除为public static Comparable min(Comparable[] a) 可以使用反射API来确定： •这个泛型方法有一个叫做 T 的类型参数。 •这个类型参数有一个子类型限定， 其自身又是一个泛型类型。 •这个限定类型有一个通配符参数。 •这个通配符参数有一个超类型限定。 •这个泛型方法有一个泛型数组参数。


为了表达泛型类型声明，使用java.lang.reflect 包中提供的接口 Type。这个接口包含下列子类型： •Class 类，描述具体类型。 •TypeVariable 接口，描述类型变量（如 T extends Comparable&lt; ? super T &gt;) •WildcardType 接口， 描述通配符 （如？super T) •ParameterizedType 接口， 描述泛型类或接口类型（如 Comparable&lt; ? super T &gt;) •GenericArrayType 接口，描述泛型数组（如 T[ ]） 下图给出继承层次，最后4个子类型是接口，虚拟机将实例化实现这些接口的适当的类。 ![img](https://img-blog.csdnimg.cn/20191203171148994.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

```java
public class GenericReflectionTest {
    public static void main(String[] args) {
        String name;
        if (args.length > 0) {
            name = args[0];
        } else {
            try (Scanner in = new Scanner(System.in)) {
                System.out.println("请输入类名：");
                name = in.next();
            }
        }
        try {
            Class<?> cl = Class.forName(name);
            printClass(cl);
            for (Method m : cl.getDeclaredMethods()) {
                printMethod(m);
            }
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
    public static void printClass(Class<?> cl) {
        System.out.print(cl);
        printTypes(cl.getTypeParameters(), "<", ", ", ">", true);
        Type sc = cl.getGenericSuperclass();
        if (sc != null) {
            System.out.print(" extends ");
            printType(sc, false);
        }
        printTypes(cl.getGenericInterfaces(), " implements", ", ", "", false);
        System.out.println();
    }
    public static void printMethod(Method m) {
        String name = m.getName();
        System.out.print(Modifier.toString(m.getModifiers()));
        System.out.print(" ");
        printTypes(m.getTypeParameters(), "<", ", ", ">", true);
        printType(m.getGenericReturnType(), false);
        System.out.print(" ");
        System.out.print(name);
        System.out.print("(");
        printTypes(m.getGenericParameterTypes(), "", ", ", "", false);
        System.out.println(")");
    }
    public static void printTypes(Type[] types, String pre, String sep, String suf, boolean isDefinition) {
        if (pre.equals(" extends ") && Arrays.equals(types, new Type[] { Object.class })) {
            return;
        }
        if (types.length > 0) {
            System.out.print(pre);
        }
        for (int i = 0; i < types.length; i++) {
            if (i > 0) {
                System.out.print(sep);
            }
            printType(types[i], isDefinition);
        }
        if (types.length > 0) {
            System.out.print(suf);
        }
    }
    public static void printType(Type type, boolean isDefinition) {
        if (type instanceof Class) {
            Class<?> t = (Class<?>) type;
            System.out.print(t.getName());
        } else if (type instanceof TypeVariable) {
            TypeVariable<?> t = (TypeVariable<?>) type;
            System.out.print(t.getName());
            if (isDefinition) {
                printTypes(t.getBounds(), " extends "," & ", "", false);
            }
        } else if (type instanceof WildcardType) {
            WildcardType t = (WildcardType) type;
            System.out.print("?");
            printTypes(t.getUpperBounds(), " extends "," & ", "", false);
            printTypes(t.getLowerBounds(), " extends "," & ", "", false);
        } else if (type instanceof ParameterizedType) {
            ParameterizedType t = (ParameterizedType) type;
            Type owner = t.getOwnerType();
            if (owner != null) {
                printType(owner, false);
                System.out.print(".");
            }
            printType(t.getRawType(), false);
            printTypes(t.getActualTypeArguments(), "<", ", ", ">", false);
        } else if (type instanceof GenericArrayType) {
            GenericArrayType t = (GenericArrayType) type;
            System.out.print("");
            printType(t.getGenericComponentType(), isDefinition);
            System.out.print("[]");
        }
    }
}
请输入类名：
chapter8.section2.Pair
class chapter8.section2.Pair<T> extends java.lang.Object
public T getFirst()
public void setFirst(T)
public T getSecond()
public void setSecond(T)
```

