一般的类和方法，只能使用具体的类型：要么是基本类型、要么是自定义的类。如果要编写可以应用于多种类型的代码，这种刻板的限制代码的束缚就会很大。
多态算是一种泛化机制。如果参数是接口，而不是一个类，基类如果是final类的限制就放松了很多。但一旦指明了接口就必须使用特定接口。
Java SE5引入了泛型的概念，实现了参数化类型的概念。

##简单泛型
容器类是促成泛型出现的一个原因。
明确指定其持有对象的类型：
```
public class Holder1 {
  private Automobile a;
  public Holder1(Automobile a) { 
    this.a = a; 
  }
  Automobile get() { 
    return a; 
  }
}
```
这个类无法持有其他类的任何对象。
Java SE5可以直接让这个类直接持有Object类型的对象：
```
public class Holder2 {
  private Object a;
  public Holder2(Object a) { 
    this.a = a; 
  }
  public void set(Object a) { 
    this.a = a; 
  }
  public Object get() { 
    return a; 
  }
  public static void main(String[] args) {
    Holder2 h2 = new Holder2(new Automobile());
    Automobile a = (Automobile)h2.get();
    h2.set("Not an Automobile");
    String s = (String)h2.get();
    h2.set(1);
    Integer x = (Integer)h2.get();
  }
}
```
现在Holder2可以存储任何类型的对象。但通常而言只会使用容器来存储一种类型的对象。泛型的主要目的之一就是用来指定容器要持有什么类型的对象，而且由编译器来保证类型的正确性。
因此与其使用Object，更喜欢暂时不指定类型。要达到这个目的，需要使用类型参数，用尖括号括住，放在类名后面。
```
public class Holder3<T> {
  private T a;
  public Holder3(T a) { 
    this.a = a; 
  }
  public void set(T a) { 
    this.a = a; 
  }
  public T get() { return a; }
  public static void main(String[] args) {
    Holder3<Automobile> h3 = new Holder3<Automobile>(new Automobile());
    Automobile a = h3.get();
    // h3.set("Not an Automobile"); // Error
    // h3.set(1); // Error
  }
}
```
现在创建Holder3对象时，必须指明想持有什么对象。并且，在Holder3中取出它持有的对象时，自动地就是正确的类型。
这就是Java泛型的核心概念：告诉编译器想使用什么类型，然后编译器帮你处理一切细节。

####一个元组类库
一个方法返回多个对象，可是return语句只允许返回单个对象。解决办法就是创建一个对象，用它来持有想要返回多个对象。这个概念成为元组(tuple)，它是将一组对象直接打包存储于其中的一个单一对象。这个容器对象允许读取其中元素，但是不允许向其他存放新的对象。
```
public class TwoTuple<A, B> {
    public final A first;
    public final B second;

    public TwoTuple(A a, B b) {
        this.first = a;
        this.second = b;
    }

    public String toString() {
        return "(" + this.first + ", " + this.second + ")";
    }
}
```
客户端程序可以读取first和second对象，然后可以随心所欲的使用这个两个对象。但是，它们却无法将其他值赋予first或second。
如果程序员想使用具有不同元素的元组，就强制要求它们创建一个新的TwoTuple对象。我们可以利用继承机制实现长度更长的元组：
```
public class ThreeTuple<A, B, C> extends TwoTuple<A, B> {
    public final C third;

    public ThreeTuple(A a, B b, C c) {
        super(a, b);
        this.third = c;
    }

    public String toString() {
        return "(" + this.first + ", " + this.second + ", " + this.third + ")";
    }
}
public class FourTuple<A, B, C, D> extends ThreeTuple<A, B, C> {
    public final D fourth;

    public FourTuple(A a, B b, C c, D d) {
        super(a, b, c);
        this.fourth = d;
    }

    public String toString() {
        return "(" + this.first + ", " + this.second + ", " + this.third + ", " + this.fourth + ")";
    }
}
public class FiveTuple<A, B, C, D, E> extends FourTuple<A, B, C, D> {
    public final E fifth;

    public FiveTuple(A a, B b, C c, D d, E e) {
        super(a, b, c, d);
        this.fifth = e;
    }

    public String toString() {
        return "(" + this.first + ", " + this.second + ", " + this.third + ", " + this.fourth + ", " + this.fifth + ")";
    }
}
```

为了使用元组，只需定义一个长度合适的元组，将其作为方法的返回值，然后在return语句中创建该元组，并返回即可：
```
class Amphibian {}
class Vehicle {}

public class TupleTest {
  static TwoTuple<String,Integer> f() {
    return new TwoTuple<String,Integer>("hi", 47);
  }
  static ThreeTuple<Amphibian,String,Integer> g() {
    return new ThreeTuple<Amphibian, String, Integer>(
      new Amphibian(), "hi", 47);
  }
  static FourTuple<Vehicle,Amphibian,String,Integer> h() {
    return new FourTuple<Vehicle,Amphibian,String,Integer>(new Vehicle(), new Amphibian(), "hi", 47);
  }
  static
  FiveTuple<Vehicle,Amphibian,String,Integer,Double> k() {
    return new FiveTuple<Vehicle,Amphibian,String,Integer,Double>(new Vehicle(), new Amphibian(), "hi", 47, 11.1);
  }
  public static void main(String[] args) {
    TwoTuple<String,Integer> ttsi = f();
    System.out.println(ttsi);
    // ttsi.first = "there"; // Compile error: final
    System.out.println(g());
    System.out.println(h());
    System.out.println(k());
  }
}/*
(hi, 47)
(generics.Amphibian@5cad8086, hi, 47)
(generics.Vehicle@610455d6, generics.Amphibian@511d50c0, hi, 47)
(generics.Vehicle@1d44bcfa, generics.Amphibian@266474c2, hi, 47, 11.1)
*/
```
ttsi.first = "there";语句错误，可以看出，final声明确实能保护public元素。

####一个堆栈类
net.mindview.util.Stack类，用一个LinledList实现堆栈。现在不用LinkedList来实现自己的内部链式存储机制：
```
public class LinkedStack<T> {
  private static class Node<U> {
    U item;
    Node<U> next;
    Node() { item = null; next = null; }
    Node(U item, Node<U> next) {
      this.item = item;
      this.next = next;
    }
    boolean end() { return item == null && next == null; }
  }
  private Node<T> top = new Node<T>(); // End sentinel
  public void push(T item) {
    top = new Node<T>(item, top);
  }	
  public T pop() {
    T result = top.item;
    if(!top.end()){
      top = top.next;
    }
    return result;
  }
  public static void main(String[] args) {
    LinkedStack<String> lss = new LinkedStack<String>();
    for(String s : "Phasers on stun!".split(" ")){
      lss.push(s);
    }
    String s;
    while((s = lss.pop()) != null){
      System.out.println(s);
    }

  }
}/*
stun!
on
Phasers
*/
```
这个例子使用了末端哨兵来判断堆栈何时为空。这个末端哨兵实在构造LinkedStack时创建的。然后每调用一次push方法，就会创建一个Node<T>对象，并将其连接到前一个Node<T>对象。当你调用pop方法时，总是返回top.item，然后丢弃当前top所指的Node<T>，并将top转移到下一个Node<T>，除非你已经碰到了末端哨兵，这时候就不在移动top了。如果已经到了末端，客户端程序还继续调用pop方法，他只能得到null，说明堆栈已经空啦。

####RandomList
需要一个特定持有类型对象的列表，每次调用其上的select方法时，它可以随机地选取一个元素。
```
public class RandomList<T> {
  private ArrayList<T> storage = new ArrayList<T>();
  private Random rand = new Random(47);
  public void add(T item) {
    storage.add(item);
  }
  public T select() {
    return storage.get(rand.nextInt(storage.size()));
  }
  public static void main(String[] args) {
    RandomList<String> rs = new RandomList<String>();
    for(String s: ("The quick brown fox jumped over the lazy brown dog").split(" ")) {
      rs.add(s);
    }
    for(int i = 0; i < 11; i++) {
      System.out.print(rs.select() + " ");
    }
  }
}/*
brown over fox quick quick dog brown The brown lazy brown
*/
```

##泛型接口
泛型也可以应用于接口。例如生成器，这是一种专门负责创建对象的类。无需额外的信息就知道如何创建新对象：
```
public interface Generator<T> {
    T next();
}
public class Coffee {
  private static long counter = 0;
  private final long id = counter++;
  @Override
  public String toString() {
    return getClass().getSimpleName() + " " + id;
  }
}
public class Latte extends Coffee {}
public class Cappuccino extends Coffee {}
public class Americano extends Coffee {}
public class Breve extends Coffee {}
public class Mocha extends Coffee {}
```

现在编写一个类，实现Generator<Coffee>接口，它能够随机生成不同类型的Coffee对象：
```
public class CoffeeGenerator implements Generator<Coffee>, Iterable<Coffee> {
  private Class[] types = { Latte.class, Mocha.class,
    Cappuccino.class, Americano.class, Breve.class, };
  private static Random rand = new Random(47);
  public CoffeeGenerator() {}
  // For iteration:
  private int size = 0;
  public CoffeeGenerator(int sz) { size = sz; }	
  @Override
  public Coffee next() {
    try {
      return (Coffee)
        types[rand.nextInt(types.length)].newInstance();
      // Report programmer errors at run time:
    } catch(Exception e) {
      throw new RuntimeException(e);
    }
  }
  class CoffeeIterator implements Iterator<Coffee> {
    int count = size;
    @Override
    public boolean hasNext() { return count > 0; }
    @Override
    public Coffee next() {
      count--;
      return CoffeeGenerator.this.next();
    }
    @Override
    public void remove() { // Not implemented
      throw new UnsupportedOperationException();
    }
  };	
  @Override
  public Iterator<Coffee> iterator() {
    return new CoffeeIterator();
  }
  public static void main(String[] args) {
    CoffeeGenerator gen = new CoffeeGenerator();
    for(int i = 0; i < 5; i++) {
      System.out.println(gen.next());
    }
    for(Coffee c : new CoffeeGenerator(5)) {
      System.out.println(c);
    }
  }
}/*
Americano 0
Latte 1
Americano 2
Mocha 3
Mocha 4
Breve 5
Americano 6
Latte 7
Cappuccino 8
Cappuccino 9
*/
```
参数化的Generator接口确保next的返回值是参数的类型。CoffeeGenerator同时还实现了Iterable接口，所以可以在循环语句中使用。
Fibonacci数列：
```
public class Fibonacci implements Generator<Integer> {
  private int count = 0;
  @Override
  public Integer next() { return fib(count++); }
  private int fib(int n) {
    if(n < 2) { return 1; }
    return fib(n-2) + fib(n-1);
  }
  public static void main(String[] args) {
    Fibonacci gen = new Fibonacci();
    for(int i = 0; i < 18; i++) {
      System.out.print(gen.next() + " ");
    }
  }
} /*
1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 1597 2584
*/
```
泛型的局限性：基本类型无法作为类型参数。但Java SE5具备了自动打包和自动拆包的功能，可以很方便地在基本类型和其相应的包装器类型之间进行转换。
更进一步，编写一个实现Iterable的Fibonacci生成器。创建一个适配器来实现所需的接口。可以通过继承来创建适配器类：
```
public class IterableFibonacci extends Fibonacci implements Iterable<Integer> {
  private int n;
  public IterableFibonacci(int count) { n = count; }
  @Override
  public Iterator<Integer> iterator() {
    return new Iterator<Integer>() {
      @Override
      public boolean hasNext() { return n > 0; }
      @Override
      public Integer next() {
        n--;
        return IterableFibonacci.this.next();
      }
      @Override
      public void remove() { // Not implemented
        throw new UnsupportedOperationException();
      }
    };
  }	
  public static void main(String[] args) {
    for(int i : new IterableFibonacci(18)) {
      System.out.print(i + " ");
    }
  }
} /* 
1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 1597 2584
*/
```
如果要在循环语句中使用IterableFibonacci，必须向IterableFibonacci的构造器提供一个边界值，然后hasNest方法才能知道何时应该返回false。

##泛型方法
泛型方法使得该方法能够独立于类而产生变化。要定义泛型方法，只需将泛型参数列表置于返回值之前：
```
public class GenericMethods {
  public <T> void f(T x) {
    System.out.println(x.getClass().getName());
  }
  public static void main(String[] args) {
    GenericMethods gm = new GenericMethods();
    gm.f("");
    gm.f(1);
    gm.f(1.0);
    gm.f(1.0F);
    gm.f('c');
    gm.f(gm);
  }
} /*
java.lang.String
java.lang.Integer
java.lang.Double
java.lang.Float
java.lang.Character
GenericMethods
*/
```
只有方法f拥有类型参数，这是由该方法的返回类型前面的类型参数列表指明的。
当使用泛型类时，必须在创建对象的时候指定类型参数的值，而使用泛型方法的时候，通常不必指明参数类型，因为编译器会为我们找出具体的类型，这称为类型参数推断。
调用f时传入基本类型，自动打包机制会介入其中，将基本类型的值包装为对应的对象。

####杠杆利用类型参数推断
通常使用泛型时需要向程序加入更多的代码：
```
Map<Person, List<? extends Pet>> petPeople = new HashMap<>();   
```
编译器本来应该能够从泛型参数列表中的一个参数推断出另一个参数，但是暂时做不到。然而在泛型方法中，类型参数推断可以为我们简化一部分工作。可以编写一个工具类，包含各式static方法，专门用来创建常用的容器类型。
```
public class New {
    public static <K,V> Map<K,V> map() {
        return new HashMap<K, V>();
    }
    public static <T> List<T> list() {
        return new ArrayList<T>();
    }
    public static <T> LinkedList<T> lList() {
        return new LinkedList<T>();
    }
    public static <T> Set<T> set() {
        return new HashSet<T>();
    }
    public static <T> Queue<T> queue() {
        return new LinkedList<T>();
    }

    public static void main(String[] args) {
        Map<String, List<String>> sls = New.map();
        List<String> ls = New.list();
        LinkedList<String> lls = New.lList();
        Set<String> ss = New.set();
        Queue<String> qs = New.queue();
    }
}
```
Map<Person, List<? extends Pet>> petPeople = New.map();
类型推断只对赋值操作有效，其他时候并不起作用。如果将一个泛型方法调用的结果作为参数传递给另一个方法，这时编译器不会执行类型推断。
```
public class LimitsOfInference {
  static void f(Map<Person, List<? extends Pet>> petPeople) {}
  public static void main(String[] args) {
    //不编译
    f(New.map());
  }
}
```

#####显式的类型说明
泛型方法中可以显式指明类型，必须在点操作符与方法名之间插入尖括号，然后把类型置于尖括号内。如果是在定义该方法的类的内部，必须在点操作符之前使用this关键字，如果是使用static的方法，必须在点操作符之前加上类名：
```
public class ExplicitTypeSpecification {
    static void f(Map<Person, List<Pet>> petPeople) {}

    public static void main(String[] args) {
        f(New.<Person, List<Pet>>map());
    }
}
```
当然这种语法抵消了New类为我们带来的好处（省去了大量的类型说明），不过只有在编写非赋值语句时，我们才需要这样的额外说明。

####可变参数与泛型方法
```
public class GenericVarargs {
  public static <T> List<T> makeList(T... args) {
    List<T> result = new ArrayList<T>();
    for(T item : args) {
      result.add(item);
    }
    return result;
  }
  public static void main(String[] args) {
    List<String> ls = makeList("A");
    System.out.println(ls);
    ls = makeList("A", "B", "C");
    System.out.println(ls);
    ls = makeList("ABCDEFFHIJKLMNOPQRSTUVWXYZ".split(""));
    System.out.println(ls);
  }
} /*
[A]
[A, B, C]
[, A, B, C, D, E, F, F, H, I, J, K, L, M, N, O, P, Q, R, S, T, U, V, W, X, Y, Z]
*/
```
makeList方法展示了与标准类库中java.util.Arrays.asList方法相同的功能。

####用于Generator的泛型方法
利用生成器，可以很方便地填充一个Collection，而泛型化这种操作是具有实际意义的：
```
public class Generators {
  public static <T> Collection<T> fill(Collection<T> coll, Generator<T> gen, int n) {
    for(int i = 0; i < n; i++) {
      coll.add(gen.next());
    }
    return coll;
  }	
  public static void main(String[] args) {
    Collection<Coffee> coffee = fill(
      new ArrayList<Coffee>(), new CoffeeGenerator(), 4);
    for(Coffee c : coffee) {
      System.out.println(c);
    }
    Collection<Integer> fnumbers = fill(new ArrayList<Integer>(), new Fibonacci(), 12);
    for(int i : fnumbers) {
      System.out.print(i + ", ");
    }
  }
} /*
Americano 0
Latte 1
Americano 2
Mocha 3
1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144,
*/
```
fill方法是如何透明地应用于Coffee和Integer的容器和生成器。

####一个通用的Generator
为任何类构造一个Generator，只要该类具有默认构造器。为了减少类型声明，它提供了一个泛型方法，用以生成BasicGenerator：
```
public class BasicGenerator<T> implements Generator<T> {
    private Class<T> type;

    public BasicGenerator(Class<T> type) {
        this.type = type;
    }

    public T next() {
        try {
            return this.type.newInstance();
        } catch (Exception var2) {
            throw new RuntimeException(var2);
        }
    }

    public static <T> Generator<T> create(Class<T> type) {
        return new BasicGenerator(type);
    }
}
```
这个类提供了一个基本实现，用以生成某个类的对象：必须声明为public，必须具备默认构造器。
下面是一个具有默认构造器的简单的类：
```
public class CountedObject {
  private static long counter = 0;
  private final long id = counter++;
  public long id() { return id; }
  @Override
  public String toString() { 
    return "CountedObject " + id;
  }
}
```
CountedObject类能够记录下它创建多少个CountedObject实例，并通过toString方法告诉其编号。使用BasicGenerator，可以很容易的为CountedObject创建一个Generator：
```
public class BasicGeneratorDemo {
  public static void main(String[] args) {
    Generator<CountedObject> gen = BasicGenerator.create(CountedObject.class);
    for(int i = 0; i < 5; i++) {
      System.out.println(gen.next());
    }
  }
} /*
CountedObject 0
CountedObject 1
CountedObject 2
CountedObject 3
CountedObject 4
*/
```

####简化元组的使用
有了类型参数推断，再加上static方法，可以重新编写之前看到的元组工具，使其成为更通用的工具类库。在这个类中，通过重载static方法创建元组：
```
public class Tuple {
    public Tuple() {
    }

    public static <A, B> TwoTuple<A, B> tuple(A a, B b) {
        return new TwoTuple(a, b);
    }

    public static <A, B, C> ThreeTuple<A, B, C> tuple(A a, B b, C c) {
        return new ThreeTuple(a, b, c);
    }

    public static <A, B, C, D> FourTuple<A, B, C, D> tuple(A a, B b, C c, D d) {
        return new FourTuple(a, b, c, d);
    }

    public static <A, B, C, D, E> FiveTuple<A, B, C, D, E> tuple(A a, B b, C c, D d, E e) {
        return new FiveTuple(a, b, c, d, e);
    }
}
```

下面是修改后的TupleTest.java，用来测试Tuple.java：
```
public class TupleTest2 {
  static TwoTuple<String,Integer> f() {
    return tuple("hi", 47);
  }
  static TwoTuple f2() { return tuple("hi", 47); }
  static ThreeTuple<Amphibian,String,Integer> g() {
    return tuple(new Amphibian(), "hi", 47);
  }
  static
  FourTuple<Vehicle,Amphibian,String,Integer> h() {
    return tuple(new Vehicle(), new Amphibian(), "hi", 47);
  }
  static
  FiveTuple<Vehicle,Amphibian,String,Integer,Double> k() {
    return tuple(new Vehicle(), new Amphibian(),
      "hi", 47, 11.1);
  }
  public static void main(String[] args) {
    TwoTuple<String,Integer> ttsi = f();
    System.out.println(ttsi);
    System.out.println(f2());
    System.out.println(g());
    System.out.println(h());
    System.out.println(k());
  }
} /*
(hi, 47)
(hi, 47)
(Amphibian@7d772e, hi, 47)
(Vehicle@757aef, Amphibian@d9f9c3, hi, 47)
(Vehicle@1a46e30, Amphibian@3e25a5, hi, 47, 11.1)
*/
```
方法f返回一个参数化的TwoTuple对象，而f2返回的是非参数化的TwoTuple对象。例子中，编译器并没有关于f2的警告信息，因为我们并没有将其返回值作为参数化对象使用。在某种意义上，它被向上转型为一个非参数化的TwoTuple。然而如果试图将f2的返回值转型为参数化的TwoTuple，编译器就会发出警告。

####一个Set实用工具
用Set来表示数学中的关系式：
```
public class Sets {
    public Sets() {
    }

    public static <T> Set<T> union(Set<T> a, Set<T> b) {
        Set<T> result = new HashSet(a);
        result.addAll(b);
        return result;
    }

    public static <T> Set<T> intersection(Set<T> a, Set<T> b) {
        Set<T> result = new HashSet(a);
        result.retainAll(b);
        return result;
    }

    public static <T> Set<T> difference(Set<T> superset, Set<T> subset) {
        Set<T> result = new HashSet(superset);
        result.removeAll(subset);
        return result;
    }

    public static <T> Set<T> complement(Set<T> a, Set<T> b) {
        return difference(union(a, b), intersection(a, b));
    }
}

```
前三个方法都将第一个set复制一份，将set中的所有引用都存入一个新的HashSet对象中，因此并未直接修改参数中的set。返回值是一个全新的set对象。
union返回一个Set，它将两个参数合并在一起；intersection返回的set只包含两个参数共有的部分；difference方法从superset中移除subset包含的元素；complement返回的set包含除了交集之外的所有元素。
enum，包含各种水彩画的颜色：
```
public enum Watercolors {
  ZINC, LEMON_YELLOW, MEDIUM_YELLOW, DEEP_YELLOW, ORANGE,
  BRILLIANT_RED, CRIMSON, MAGENTA, ROSE_MADDER, VIOLET,
  CERULEAN_BLUE_HUE, PHTHALO_BLUE, ULTRAMARINE,
  COBALT_BLUE_HUE, PERMANENT_GREEN, VIRIDIAN_HUE,
  SAP_GREEN, YELLOW_OCHRE, BURNT_SIENNA, RAW_UMBER,
  BURNT_UMBER, PAYNES_GRAY, IVORY_BLACK
}
```
为了方便起见，下面的示例以static的方式引入Watercolors。这个示例使用EnumSet，这是Java SE5的新工具，用来从enum直接创建Set。在这里，向static方法EnumSet.range传入某个范围的第一个元素与最后一个元素，然后它将返回一个set，其中包含该范围内的所有元素：
```
public class WatercolorSets {
  public static void main(String[] args) {
    Set<Watercolors> set1 =
      EnumSet.range(BRILLIANT_RED, VIRIDIAN_HUE);
    Set<Watercolors> set2 =
      EnumSet.range(CERULEAN_BLUE_HUE, BURNT_UMBER);
    print("set1: " + set1);
    print("set2: " + set2);
    print("union(set1, set2): " + union(set1, set2));
    Set<Watercolors> subset = intersection(set1, set2);
    print("intersection(set1, set2): " + subset);
    print("difference(set1, subset): " +
      difference(set1, subset));	
    print("difference(set2, subset): " +
      difference(set2, subset));
    print("complement(set1, set2): " +
      complement(set1, set2));
  }	
} /*
set1: [BRILLIANT_RED, CRIMSON, MAGENTA, ROSE_MADDER, VIOLET, CERULEAN_BLUE_HUE, PHTHALO_BLUE, ULTRAMARINE, COBALT_BLUE_HUE, PERMANENT_GREEN, VIRIDIAN_HUE]
set2: [CERULEAN_BLUE_HUE, PHTHALO_BLUE, ULTRAMARINE, COBALT_BLUE_HUE, PERMANENT_GREEN, VIRIDIAN_HUE, SAP_GREEN, YELLOW_OCHRE, BURNT_SIENNA, RAW_UMBER, BURNT_UMBER]
union(set1, set2): [SAP_GREEN, ROSE_MADDER, YELLOW_OCHRE, PERMANENT_GREEN, BURNT_UMBER, COBALT_BLUE_HUE, VIOLET, BRILLIANT_RED, RAW_UMBER, ULTRAMARINE, BURNT_SIENNA, CRIMSON, CERULEAN_BLUE_HUE, PHTHALO_BLUE, MAGENTA, VIRIDIAN_HUE]
intersection(set1, set2): [ULTRAMARINE, PERMANENT_GREEN, COBALT_BLUE_HUE, PHTHALO_BLUE, CERULEAN_BLUE_HUE, VIRIDIAN_HUE]
difference(set1, subset): [ROSE_MADDER, CRIMSON, VIOLET, MAGENTA, BRILLIANT_RED]
difference(set2, subset): [RAW_UMBER, SAP_GREEN, YELLOW_OCHRE, BURNT_SIENNA, BURNT_UMBER]
complement(set1, set2): [SAP_GREEN, ROSE_MADDER, YELLOW_OCHRE, BURNT_UMBER, VIOLET, BRILLIANT_RED, RAW_UMBER, BURNT_SIENNA, CRIMSON, MAGENTA]
*/
```

可以从输出中看到各种关系运算的结果。
使用Sets.difference打印出java.util包中各种Collection类与Map类之间的方法差异：
```
public class ContainerMethodDifferences {
    static Set<String> object = methodSet(Object.class);

    static {
        object.add("clone");
    }

    public ContainerMethodDifferences() {
    }

    static Set<String> methodSet(Class<?> type) {
        Set<String> result = new TreeSet();
        Method[] var5;
        int var4 = (var5 = type.getMethods()).length;

        for(int var3 = 0; var3 < var4; ++var3) {
            Method m = var5[var3];
            result.add(m.getName());
        }

        return result;
    }

    static void interfaces(Class<?> type) {
        System.out.print("Interfaces in " + type.getSimpleName() + ": ");
        List<String> result = new ArrayList();
        Class[] var5;
        int var4 = (var5 = type.getInterfaces()).length;

        for(int var3 = 0; var3 < var4; ++var3) {
            Class<?> c = var5[var3];
            result.add(c.getSimpleName());
        }

        System.out.println(result);
    }

    static void difference(Class<?> superset, Class<?> subset) {
        System.out.print(superset.getSimpleName() + " extends " + subset.getSimpleName() + ", adds: ");
        Set<String> comp = Sets.difference(methodSet(superset), methodSet(subset));
        comp.removeAll(object);
        System.out.println(comp);
        interfaces(superset);
    }

    public static void main(String[] args) {
        System.out.println("Collection: " + methodSet(Collection.class));
        interfaces(Collection.class);
        difference(Set.class, Collection.class);
        difference(HashSet.class, Set.class);
        difference(LinkedHashSet.class, HashSet.class);
        difference(TreeSet.class, Set.class);
        difference(List.class, Collection.class);
        difference(ArrayList.class, List.class);
        difference(LinkedList.class, List.class);
        difference(Queue.class, Collection.class);
        difference(PriorityQueue.class, Queue.class);
        System.out.println("Map: " + methodSet(Map.class));
        difference(HashMap.class, Map.class);
        difference(LinkedHashMap.class, HashMap.class);
        difference(SortedMap.class, Map.class);
        difference(TreeMap.class, Map.class);
    }
}
```

##匿名内部类
泛型还可以应用于内部类以及匿名内部类，使用匿名内部类实现了Generator接口：
```
class Customer {
  private static long counter = 1;
  private final long id = counter++;
  private Customer() {}
  @Override
  public String toString() { return "Customer " + id; }
  // A method to produce Generator objects:
  public static Generator<Customer> generator() {
    return new Generator<Customer>() {
      @Override
      public Customer next() { return new Customer(); }
    };
  }
}	

class Teller {
  private static long counter = 1;
  private final long id = counter++;
  private Teller() {}
  @Override
  public String toString() { return "Teller " + id; }
  // A single Generator object:
  public static Generator<Teller> generator = new Generator<Teller>() {
      @Override
      public Teller next() { return new Teller(); }
    };
}	

public class BankTeller {
  public static void serve(Teller t, Customer c) {
    System.out.println(t + " serves " + c);
  }
  public static void main(String[] args) {
    Random rand = new Random(47);
    Queue<Customer> line = new LinkedList<Customer>();
    Generators.fill(line, Customer.generator(), 15);
    List<Teller> tellers = new ArrayList<Teller>();
    Generators.fill(tellers, Teller.generator, 4);
    for(Customer c : line) {
      serve(tellers.get(rand.nextInt(tellers.size())), c);
    }
  }	
} /*
Teller 3 serves Customer 1
Teller 2 serves Customer 2
Teller 3 serves Customer 3
Teller 1 serves Customer 4
Teller 1 serves Customer 5
Teller 3 serves Customer 6
Teller 1 serves Customer 7
Teller 2 serves Customer 8
Teller 3 serves Customer 9
Teller 3 serves Customer 10
Teller 2 serves Customer 11
Teller 4 serves Customer 12
Teller 2 serves Customer 13
Teller 1 serves Customer 14
Teller 1 serves Customer 15
*/
```
Customer和Teller类都只有private的构造器，这可以强制你必须使用Generator对象。Customer有一个generator方法，每次执行它都会生成一个新的Generator<Customer>对象。我们其实不需要多个Generator方法，Teller就之创建一个public的generator对象。在main方法中可以看到，这两种创建Generator的方式都在fill中用到了。
