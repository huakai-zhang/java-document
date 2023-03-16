---
layout:  post
title:   Java核心技术 接口、lambda表达式与内部类
date:   2019-11-19 19:04:45
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-Java核心技术

---



## 1.接口

### 接口概念

接口（interface）技术，主要描述类具有什么内容，而并不给每个功能的具体实现。一个类可以实现（implement）一个或多个接口。 接口不是类，而是对类的一组需求描述。 如果类遵从某个特定接口，那么就履行这项服务。 Arrays类中的sort方法承诺可以对对象数组进行排序，但要求满足下列前提：对象所属的类必须实现Comparable接口。

```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```

接口的所有方法自动地属于public，所以声明方法时不必提供关键字public。 在调用x.compareTo(y)的时候，这个compareTo方法必须确实比较两个对象的内容，并返回比较结果。当x小于y时，返回一个负数；当x等于y时，返回0；否则返回一个正数。

接口可以定义常量，不能含有实例域，Java SE 8之后可以在接口中提供简单的方法。 提供实例域和方法实现的任务应该由实现接口的那个类来完成。因此，可以将接口看成是没有实例域的抽象类。

为了让类实现一个接口： 1.将类声明为实现给的接口 2.对接口中的所有方法进行定义

```java
public class Employee implements Comparable {
    @Override
    public int compareTo(Object otherObject) {
        Employee other = (Employee) otherObject;
        return Double.compare(salary, other.salary);
    }
}
```

在实现接口时，必须声明为public，否则编译器认为这个方法的访问属性是包可见的。

但为什么不能在Employee类直接提供一个compareTo方法，而必须实现Comparable接口呢？ 原因在于Java程序设计语言是一种强类型（strongly typed）语言，在调用方法的时候，编译器将会检查这个方法是否存在。在sort方法中可能存在这样的语句：

```java
if (a[i].compareTo(a[i]) > 0) {
	...
}
```

编译器必须确认a[i]一定有compareTo方法，如果a是一个Comparable对象的数组，就可以确保拥有compareTo方法，因为每个实现compareTo接口的类型都必须提供这个方法的定义。

### 接口的特性

接口不能使用new运算符实例化，却能声明接口的变量，接口变量必须引用实现了接口的类对象：

```java
Comparable x;
x = new Employee(...);
```

可以使用instance检查一个对象是否实现了某个特定的接口：

```java
if (anObject instanceof Comparable) {}
```

与可以建立类的继承关系一样，接口也可以被扩展。允许存在多条从具有较高通用性的接口到较高专用性的接口的链。 虽然在接口中不能包含实例域或静态方法，但却可以包含常量。

```java
public interface Powered extends Moveable {
	double milesPerGallon();
	double SPEED_LIMIT = 95;
}
```

与接口中的方法都自动地被设置为public一样，接口中的域将被自动设置public static final； 有些接口只定义了常量，而没有定义方法。任何实现了这种接口的类都自动继承这些常量，并且在方法中可以直接引用，而不必采用（Object.常量）这样的繁琐书写形式。然而这样应用接口似乎有点偏离了接口概念的初衷，最好不要这样使用它。

尽管每个类只能够拥有一个超类，但却可以实现多个接口。这就为定义类的行为提供了极大的灵活性。

### 接口与抽象类

为什么不将Comparable直接设计成抽象类？ 使用抽象类存在一个问题，每个类只能扩展于一个类。假设Employee类可以扩展于一个类，它就不能扩展第二个类了，但每个类可以实现多个接口。 Java的设计者选择了不支持多继承，其只要原因是多继承会让语言本身变得非常复杂，效率也会降低。实际上，接口可以提供多重继承的大多数好处，同时还避免了多重继承的复杂性和低效性。

### 静态方法

Java SE 8中，允许接口增加静态方法，理论上，没有任何理由认为这是不合法的，只是这有违于将接口做为抽象规范的初衷。 目前为止，通常的做法是将静态方法放在伴随类中。在标准库中，可以看到成对出现的接口和实现工具类：Collection/Collections或Path/Paths。 Paths类中包含两个工厂方法，可以由一个字符串序列构造一个文件或目录的路径，如Paths.get(“jdk1.8.0”, “jre”, “bin”)。 在Java SE 8中可以为Path接口添加静态方法：

```java
public interface Path {
    public static Path get(String first, String... more) {
        return FileSystems.getDefault().getPath(first, more);
    }
}
```

这么一来Paths就没有存在的必要了。 不过整个Java路都以这种方式重构也不太可能，但是在实现自己的几口时，不再需要为实用工具方法另外提供伴随类。

### 默认方法

可以为接口方法提供一个默认实现，必须用default修饰符标记，默认方法无需实现：

```java
public interface Comparable<T>{
	default int compareTo(T other) {
		return 0;
	}
}
```

这并没有太大用处，因为Comparable的每一个实际实现都要覆盖这个方法。 不过有些情况默认方法可能有用，比如一个下面MouseListener接口大多数情况下，只关心1，2个事件类型，在Java SE 8中，可以把所有方法声明为默认方法：

```java
public interface MouseListener {
    default void mouseClicked(MouseEvent e){}

    default void mousePressed(MouseEvent e){}

    default void mouseReleased(MouseEvent e){}

    default void mouseEntered(MouseEvent e){}

    default void mouseExited(MouseEvent e){}
}
```

实现接口的程序员只需要为他们关心的事件覆盖相应的监听器，默认方法可以调用任何其他方法。

默认方法的一个重要用法是接口演化（interface evolution）。 在新版本中为接口新增方法，继承它的类将不能编译，因为它没有实现新方法，为接口增加一个非默认方法不能保证“源代码兼容”。 不过，假设不重新编译这个类，而只是使用原先一个包含这个类的JAR文件，程序仍能加载，可以正常构造Bag实例。不过如果程序在一个Bag实例上调用新方法，就会出现一个AbstractMethodError。 将方法实现为一个默认方法就可以解决这两个问题。

### 解决默认方法冲突

如果在一个接口中将一个方法定义为默认方法，然后又在超类或另一个接口定义了同样的方法： 1.超类优先 2.接口冲突，超接口提供一个默认方法，另一个接口提供同名而且参数类型相同的方法（二义性），Java编译器会报告一个错误，让程序员解决二义性。只需在子类中提供一个同名方法，覆盖这个方法来解决冲突。

## 2.接口示例

### 接口与回调

回调（callback）是一种常见的程序设计模式。可以指定某个特定事件发生时应该采取的动作。

java.swing包中有一个Timer类，可以使用它在给定时间间隔时发出通告。在构造定时器时，需要设置一个时间间隔，并告知定时器，到达时间应该做什么。 如何告知定时器？Java标准类库采用面向对象方法，将某个类的对象传递给定时器，然后定时器调用这个对象的方法。

```java
public class TimerTest {
    public static void main(String[] args) {
        ActionListener listener = new TimePrinter();

        Timer t = new Timer(11000, listener);
        t.start();

        JOptionPane.showMessageDialog(null, "退出程序？");
        System.exit(0);
    }
}

class TimePrinter implements ActionListener {

    @Override
    public void actionPerformed(ActionEvent e) {
        System.out.println("目前时间是：" + new Date());
        Toolkit.getDefaultToolkit().beep();
    }
}
```

上述程序给出了定时器和监听器的操作操作行为，在定时器启动后，会弹出一个消息框，等待用户点击确认来终止程序，在未终止程序前，每个10秒会显示一次当前的时间。

### Comparator接口

String类实现了Comparable&lt; String &gt;，而且String.compareTo方法可以按照字典顺序比较字符串。 假设我们希望按长度递增对字符串进行排序，不能让String用两种方式实现compareTo方法，更何况String类也不应有程序员修改。 Arrays.sort方法还有第二个版本，有一个数组和一个比较器作为参数，比较器实现了Comparator接口：

```java
public interface Comparator<T> {
	int compare(T o1, T o2);
}
```

具体比较时需要建立一个实例：

```java
Comparator<string> comp = new LengthComparator();
if (comp.compare(words[i], words[j]) > 0) ...
```

这个compare方法要在比较器对象上调用，而不是字符串本身上调用。

```java
public class LengthComparator implements Comparator<String> {
    @Override
    public int compare(String first, String second) {
        return first.length() - second.length();
    }
    public static void main(String[] args) {
        String[] friends = { "Peter", "Paul", "Mary" };
        Arrays.sort(friends, new LengthComparator());
        System.out.println(Arrays.toString(friends));
        // [Paul, Mary, Peter]
    }
}
```

### 对象克隆

Cloneable接口，指示一个类提供一个安全的clone方法。 如果希望copy是一个新对象，它的初始状态与original相同，但是之后各自会有自己不同的状态，这种情况下可以使用clone方法。

clone方法是Object的一个protected方法，说明代码不能直接调用这个方法。只有Employee对象可以克隆Employee对象。 默认克隆操作是“浅拷贝”，并没有克隆对象中引用的其他对象。 浅拷贝的影响：如果原对象和浅克隆对象的子对象是不可变的，如String，或者在对象的生命期中，子对象一直包含不可变的常量，没有更改器方法给改变它，也没有方法会生成它的引用，这种情况下是安全的。 通常子对象是可变的，必须重新定义clone方法来建立一个深拷贝，同时克隆左右子对象。

浅克隆（Employee类的hireDay属性这里使用可变类型Date）：

```java
public class Employee implements Cloneable  {
	...
    @Override
    public Employee clone() throws CloneNotSupportedException {
        return (Employee) super.clone();
    }
    ...
}
```

```java
Employee original = new Employee("John Public", 50000);
Employee copy = original.clone();
copy.setName("Jack");
copy.setHireDay(1998, 10, 01);
copy.setSalary(40000);
System.out.println(original.toString());
System.out.println(copy.toString());
// Employee[name=John Public,salary: 50000.0,hireDay=Thu Oct 01 00:00:00 CST 1998]
// Employee[name=Jack,salary: 40000.0,hireDay=Thu Oct 01 00:00:00 CST 1998]
```

Cloneable接口是Java提供的一组标记接口（tagging interface）之一，它的唯一作用是允许在类型查询中使用instanceof，它的的出现与接口的正常使用没有关系。具体来说，它没有制定clone方法，这个方法是从Object类继承的。这个接口只是作为一个标记，指示类设计者了解克隆过程。如果一个对象请求克隆，但没有实现这个接口，就会生成一个受检异常。

即使clone的默认实现能够满足要求，还是需要实现Cloneable接口，将clone重新定义为public，再调用super.clone()。

如果在一个对象上调用了clone，但这个对象的类没有实现Cloneable接口，Object类的clone方法就会抛出CloneNotSupportedException。当然Employee和Date都实现了Cloneable接口，所以不会抛出异常，但是编译器并不知道这一点。

深拷贝：

```java
public Employee clone() throws CloneNotSupportedException {
    Employee cloned = (Employee) super.clone();
    cloned.hireDay = (Date) hireDay.clone();
    return cloned;
}

Employee original = new Employee("John Public", 50000);
Employee copy = original.clone();
copy.setName("Jack");
copy.setHireDay(1998, 10, 01);
copy.setSalary(40000);
System.out.println(original.toString());
System.out.println(copy.toString());
// Employee[name=John Public,salary: 50000.0,hireDay=Sun Nov 17 17:02:51 CST 2019]
// Employee[name=Jack,salary: 40000.0,hireDay=Thu Oct 01 00:00:00 CST 1998]
```

所有数组类型都有一个public的clone方法，而不是protected。可以使用这个方法建立一个新数组，包含原数组的所有元素的副本。

```java
int[] luckNumbers = { 2, 3, 5, 7, 11, 13};
int[] cloned = luckNumbers.clone();
cloned[5] = 12;
System.out.println(Arrays.toString(luckNumbers));
System.out.println(Arrays.toString(cloned));
// [2, 3, 5, 7, 11, 13]
// [2, 3, 5, 7, 11, 12]
```

## 3.lambda表达式

### 为什么引入lambda表达式

lambda表达式是一个可传递的代码块，可以在以后执行一次或多次。 ActionListener，LengthComparator都是将一个代码块传递到某个对象（一个定时器或一个sort方法），这个代码块将会在将来的某个时间调用。 到目前为止，在Java中传递一个代码段并不容易，不能直接传递代码段。Java是一种面向对象语言，所以必须构造一个对象，这个对象的类需要有一个方法能够包含所需的代码。 就现在来说，问题已经不是是否增强Java来支持函数式编程，而是要如何做到这一点，设计者们做了多年的尝试，终于找到了一种适合Java的设计。

### lambda表达式的语法

1.就上诉字符串按长度排序的问题，Java是强类型语言，要指定他们的类型：

```java
(String first, String second) -> first.length() - second.length()
```

这就是一个lambda表达式，就是一个代码块，以及必须传入代码得变量规范。 逻辑学家Alonzo Church想要形式化地表示能有效计算的数学函数。受权威的数学原理一书使用重音符^来表示自由变量，Church使用大写lambda（大写Λ）表示参数。不过后来改为了小写的lambda（小写λ）。从此之后，带参数变量的表达式就被成为lambda表达式。

lambda表达式形式：参数，箭头（->）以及一个表达式。 2.如果代码要完成的计算无法放在一个表达式中，可以像写方法一样，放在{}中并包含显式的return语句：

```java
(String first, String second) -> {
	if (first.length() > second.length()) {return -1;}
	else if (first.length() > second.length()) {return 1;}
	else return 0;
}
```

3.如果没有参数：

```java
() -> {
	for (int i = 100; i >= 0; i--) {
		System.out.println(i);
	}
}
```

4.如果可以推导lambda表达式的参数类型，则可以忽略其类型：

```java
Comparator<String> comp = (first, second) -> first.length() - first.length();
```

5.如果方法只有一个参数，而且参数类型可以推导得出，可以省略小括号：

```java
ActionListener listener = event -> System.out.println(new Date());
```

```java
public class LambdaTest {
    public static void main(String[] args) {
        String[] planets = new String[] { "Mercury", "Venus", "Earth", "Mars",
            "Jupiter", "Saturn", "Uranus", "Neptune" };
        System.out.println(Arrays.toString(planets));
        System.out.println("按字典顺序排序：");
        Arrays.sort(planets);
        System.out.println(Arrays.toString(planets));
        System.out.println("按长度排序：");
        Arrays.sort(planets, (first, second) -> first.length() - second.length());
        System.out.println(Arrays.toString(planets));
        Timer t = new Timer(1000, event -> System.out.println("时间是：" + new Date()));
        t.start();
        JOptionPane.showConfirmDialog(null, "退出程序？");
        System.exit(0);
    }
}
```

### 函数式接口

对于只有一个抽象方法的接口，需要这种接口的对象时，就可以提供一个lambda表达式。这种接口成为函数式接口（functional interface）。

Arrays.sort方法，第二个参数需要一个Comparator实例，Comparator就是只有一个方法的接口，所以可以提供lambda表达式：

```java
Arrays.sort(words, (first, second) -> first.length() - second.length());
```

在底层，Arrays.sort方法会接收实现了Comparator&lt; String &gt;的某个类的对象。在这个对象上调用compare方法会执行这个lambda表达式的体。这些对象和类的管理完全取决于具体实现，与使用传统的内联类相比，这样可能要高效得多。最好把lambda表达式看作是一个函数，而不是一个对象，另外要接受lambda表达式可以传递到函数式接口。

Java API在java.util.function包中定义了很多通用的函数式接口。不过很多类似Comparator的接口往往有一个特定的用途，而不只是提供一个有指定参数和返回类型的方法。

java.util.function包中有一个尤其有用的接口Predicate：

```java
public interface Predicate<T> {
    boolean test(T t);
}
```

ArrayList类有一个removeIf方法，它的参数就是一个Predicate。这个接口专门用来传递lambda表达式。例如，从一个数组删除所有null值：

```java
list.removeIf(e -> e == null);
```

### 方法引用

有时，可能已经有现成的方法可以完成想要传递到代码的某个动作：

```java
Timer t = new Timer(1000, event -> System.out.println(event));
```

但是，如果直接把println方法传递到Timer构造器就更好了：

```java
Timer t = new Timer(1000, System.out::println);
```

表达式System.out::println是一个方法引用（method reference），它等价于lambda表达式x -&gt;System.out.println(x)。

假设对字符串排序，而不考虑字母的大小写：

```java
Arrays.sort(strings, String::compareToIgnoreCase);
```

要使用::操作符分割方法名与对象或类名，主要有三种情况： 1.object::instanceMethod // 实例方法 2.Class::staticAMethod // 静态方法 3.Class：：instanceMethod 前两种方法引用等价于提供方法参数的lambda表达式，例如System.out::println等价于System.out.println(x)，Math::pow等价于(x, y) -&gt; Math.pow(x, y)。 对于第三种情况，第一个参数会成为方法的目标。例如，String::compareToIgnoreCase等同于(x, y) -&gt; x.compareToIgnoreCase(y)。

可以在方法引用中使用this，super参数：

```java
public class Greeter {
    public void greet() {
        System.out.println("Hello, world!");
    }
}

public class TimedGreeter extends  Greeter {
    @Override
    public void greet() {
        Timer t = new Timer(100, super::greet);
        t.start();
    }
}
```

### 构造器引用

构造器引用和方法引用类似，只不过方法名为new，Person::new是Person构造器的一个引用，哪个构造器呢？这取决于上下文。 假设有一个字符串列表，可以把它转换为一个Person对象数组：

```java
ArrayList<String> names = ...;
Stream<Person> stream = names.stream().map(Person::new);
List<Person> people = stream.collect(Collectors.toList());
```

map方法会为各个列表元素调用Person(String)构造器。

可以用数组类型构建构造器引用，int[]::new是一个构造器引用，它有一个参数：即数组的长度，这等价于lambda表达式x -&gt; new int[x]。 Java有一个限制，无法构造泛型类型T的数组。数组构造器引用对于克服这个限制很有用。表达式new T[n]会产生错误，因为会改为new Object[n]。 流库利用构造器引用解决这个问题，可以把Person[]::new传入toArray方法： Person[] people = stream.toArray(Person[]::new); toArray方法调用这个构造器来得到正确的数组。然后填充这个数组并返回。

### 变量作用域

```java
public static void repeatMessage(String text, int delay) {
    ActionListener listener = event -> {
        System.out.println(text);
        Toolkit.getDefaultToolkit().beep();
    };
    new Timer(delay, listener).start();
}
```

lambda表达式有3个部分： 1.一个代码块 2.参数 3.自由变量的值，这是指非参数而且不在代码中定义的变量

自由变量text，表示lambda表达式的数据结构必须存储自由变量的值，也可以说自由变量被lambda表达式捕获（captured）。 关于代码块以及自由变量值有一个术语叫闭包（closure），Java也有闭包，lambda表达式就是闭包。

在lambda表达式中，只能引用值不会改变的变量：

```java
public static void countDown(int start, int delay) {
    ActionListener listener = event -> {
        start--; //无法改变捕获的变量，不合法
        System.out.println(start);
    };
    new Timer(delay, listener).start();
}
```

如果在lambda表示中改变变量，并发执行多个动作时就会不安全。

如果lambda表达式引用变量，而变量可能在外部改变，这也不合法：

```java
public static void repeat(String test, int count) {
    for (int i = 1; i <= count; i++) {
        ActionListener listener = event -> {
            System.out.println(i + ": " + test);  // 不能改变i，不合法
        };
        new Timer(1000, listener).start();
    }
}
```

lambda表达式中捕获的变量实际上是最终变量（effectively final）。这个变量初始化之后就不会在为它赋新值。

lambda表达式的体与嵌套体有相同的作用域，同样适用命名冲突和遮蔽的有关规则。

```java
Path first = Paths.get("/usr/bin");
Comparator<String> comp =
(first, second) -> first.length() - second.length(); // 错误，变量已定义
```

lambda表达式中使用this关键字时，是指创建这个lambda表达式的方法的this参数：

```java
public class Application(){
	public void init(){
		ActionListener listener = event ->{
			System.out.println(this.toStringO);
			...
		}
		...
	}
}
```

this.toString()会调用Application对象的toString方法，而不是ActionListener实例的方法。

### 处理lambda表达式

使用lambda表达式的重点是延迟执行（deferred execution）。之所以希望以后在执行代码有很多原因： 1.在一个单独的线程中运行代码 2.多次运行代码 3.在算法的适当位置运行代码 4.发生某种情况时运行代码 5.只在必要时才运行代码

```java
public static void main(String[] args) {
       repeat(10, () -> System.out.println("Hello,World!"));
}

public static void repeat(int n, Runnable action) {
    for (int i = 0; i < n; i++) {
        action.run();
    }
}
```


要接受这个lambda表达式，需要选择一个函数式接口： ![img](https://img-blog.csdnimg.cn/20191119174132450.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) 需要说明，调用action.run()时会执行这个lambda表达式主体。

自定义函数式接口：

```java
public interface IntConsumer {
    void accept(int value);
}

public static void main(String[] args) {
    repeat(10, i -> System.out.println("倒计时：" + (9 - i)));
}

public static void repeat(int n, IntConsumer action)
{
    for (int i = 0; i < n; i++) {
        action.accept(i);
    }
}
```


基本类型的函数式接口： ![img](https://img-blog.csdnimg.cn/20191119174235983.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) 如果是自己的接口，其中只有一个抽象方法，可以用@FunctionalInterface注解来标记这个接口。有两个优点，一是如果你无意中增加了另一个非抽象方法，编译器会产生错误消息，另外javadoc会指出这个接口是一个函数式接口。 并不是必须使用注解。根据定义，任何有一个抽象方法的几口都是函数式接口，不过使用@FunctionalInterface是一个很好的做法。

### 再谈Comparator

静态方法comparing方法取一个“键提取器”函数，它将类型T映射为一个可比较的类型。对要比较的对象应用这个函数，然后对返回的键完成比较。

```java
Arrays.sort(people, Comparator.comparing(Person::getName));
```

可以把比较器和thenComparing方法串联起来：

```java
Arrays.sort(people, Comparator.comparing(Person::getLastName).thenComparing(Person::getFirstName));
```

这些方法有很多变体形式，comparing和thenComparing方法提取的键制定一个比较器：

```java
Arrays.sort(people, Comparator.comparing(Person::getName, (s, t) -> Integer.compare(s.length(), t.length())));
```

还有一些变体形式，可以避免int,long或double值的装箱：

```java
Arrays.sort(people, Comparator.comparingInt(p -> p.getName().length()));
```

如果键函数可以返回null，可能就要用到nullsFirst和nullsLast适配器。这些静态方法会修改现有的比较器，从而在遇到null时不会抛出异常，而是将这个值标记为小于或大于正常值。nullsFirst需要一个比较器，naturalOrder方法可以为任何实现了Comparable的类建立一个比较器。 需要静态导入java.util.Comparator.*：

```java
import static java.util.Comparator.*;

Arrays.sort(employees, comparing(Employee::getName, nullsFirst(naturalOrder())));
```

静态reverseOrder方法会提供自然顺序的逆序，等同于naturalOrder().reversed()；

