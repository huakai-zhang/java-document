## 1.面向对象程序设计概述

面向对象程序设计（简称OOP），Java是完全面向对象的。

面向对象的程序是由对象组成的，每个对象包含对用户公开的特定功能部分和隐藏的实现部分。

在 OOP 中，不必关心对象的具体实现，只要能够满足用户的需求即可。

### 类

`类（class）`是构造对象的模版或蓝图。<font color=red>由类构造（construct）对象的过程被称为创建类的实例（instance）。</font>

`封装（encapsulation，有时称为数据隐藏）`，将数据和行为组合在一个包中，并对对象的使用者隐藏了数据的实现方式，对象的数据被称为`实例域（instance field）`，操纵数据的过程成为`方法（method）`。

实现封装的关键在于绝对不能让类中的方法直接地访问其他类的实例域，程序仅通过对象的方法与对象数据进行交互。

OOP 的另一个原则，可以通过扩展一个类来建立另一个新的类。事实上所有 Java 类都源自于一个超类，Object。扩展后的类具有所扩展的类的全部属性和方法。

### 对象

对象的三个特征：

对象行为（behavior），可以对对象施加哪些操作，或可以对对象施加哪些方法

状态（state），当施加那些方法时，对象如何响应

标识（identity），如何辨别具有相同行为与状态的不同对象

每个对象都保存着描述当前特征的信息，这就是对象的状态。对象的状态并不能完全描述一个对象，每个对象都有一个唯一的身份（identity）。做为一个类的实例，每个对象的标识永远是不同的，状态常常也存在着差异。

### 识别类

识别类的简单规则是在分析问题的过程中寻找名词，而方法对应着动词。

### 类之间的关系

常见关系有：

依赖（“uses-a”），一个类的方法操作另一个类的对象，我们就说一个类依赖于另一个类，尽可能将相互依赖的类减至最少，如果类A不知道B的存在，就不用关心B的任何变化，用软件工程术语来说就是，让类之间的耦合度最小。

聚合（“has-a”），聚合关系意味着类A包含类B的对象

继承（“is-a”） &nbsp;

## 2.使用预定义类

Math 类只封装了功能，不需要也不必隐藏数据，由于没有数据，因此也不必担心生成对象以及初始化实例域。

### 对象与对象变量

想使用对象就必须构造对象，并制定其初始化状态。

Java中使用构造器（constructor）构造新实例，构造器是一种特殊的方法，用来构造并初始化对象。

在对象与对象变量之间存在着一个重要区别：

```java
Data deadline;
```

定义了一个对象变量 deadline，它可以引用 Date 类型的对象。但是变量 deadline 不是一个对象，实际上也没有引用对象，此时不能将任何 Date 方法应用于这个变量上。

必须初始化变量 deadline，有两个选择：

1. 用构造器的对象初始化这个变量

2. 引用一个已存在的对象

一个对象变量并没有实际上包含一个对象，而仅仅引用一个对象。

在 Java 中，任何对象变量的值都是对存储在另外一个地方的一个对象的引用。

> new 操作符的返回值也是一个引用。

```java
// 表达式 new Date() 构造了一个 Date 类型的对象
// 并且它的值是对新创建对象的引用，这个引用存储在变量 deadline 中
Data deadline = new Date();
```

可以显示地将对象变量设置为 null，表明对象变量没有任何引用，如果将一个方法应用于一个值为 null 的对象上，那么就会产生运行时错误。

```java
Date deadline = new Date();
deadline = null;
String s = deadline.toString(); // NullPointerException
```

### Java 变量分类

按照数据类型分：`基本数据类型`和`引用数据类型`

按照类中位置分：

* `成员变量`：在使用前，都经过默认初始化赋值

  - 类变量（静态变量，被 static 修饰的成员变量）：

    linking 的 prepare 阶段给变量默认赋值

    initial 阶段给变量显示赋值即静态代码块赋值

  - 实例变量（没有被 static 修饰的成员变量）：随着对象的创建，会在堆空间分配实例变量空间，进行默认赋值

* `局部变量`：使用前必须显示赋值，否则编译不通过。

### Java类库中的LocalDate类

Date类是用距离一个固定时间点的毫米数（可正可负）表示，这个点就是所谓的纪元（epoch），它是UTC时间1970年1月1日00：00：00。UTC是Coordinated Universal Time的缩写，是一种具有实践意义的科学标准时间。

Date可以满足大多数地区的阳历表示法，但同一时间点采用中国的农历就不一样了，所以类库的设计者决定将保存时间与给时间点命名分开。一个表示时间点的Date类，另一个采用日历表示法LocalDate类。

不要使用构造器构造LocalDate类的对象，应当使用静态工厂方法（factory method）代表调用了构造器：

```java
LocalDate.now();
```

可提供年、月、日来构造对应一个特定日期的对象：

```java
LocalDate newYear = LocalDate.of(2019, 10, 23);
System.out.println(newYear.getYear());
System.out.println(newYear.getMonthValue());
System.out.println(newYear.getDayOfMonth());
```

看起来没多大意义，因为这正是构造对象使用的值，不过有时某个日期是计算出来的：

```java
LocalDate aThousandDaysLater = newYear.plusDays(1000);
```

LocalDate类封装了实例域来维护所设置的日期。如果不查看源代码，就不可能知道类内部的日期表示。当然，封装的意义在于，类对外提供的方法。

实际上Date类页游getDay等方法，然而并不推荐使用，当类库设计者意识到某些方法不应该存在时，就会把它标记为不鼓励使用。

&nbsp;

### 更改器方法与访问器方法

plusDays方法生成了一个新的LocalDate对象，然后赋值给aThousandDaysLater变量，原来的对象不做任何变动。我们说plusDays没有更改调用这个方法的对象。

```java
GregorianCalendar someDay = new GregorianCalendar(2019, 10, 23);
someDay.add(Calendar.DAY_OF_MONTH, 1000);
System.out.println(someDay.get(Calendar.YEAR));
System.out.println(someDay.get(Calendar.MONTH) + 1);
System.out.println(someDay.get(Calendar.DAY_OF_MONTH));
```

GregorianCalendar.add方法是一个更改器方法（mutator method）。调用这个方法后，someDay对象的状态会改变。

相反的，只访问对象而不修改对象的方法称为访问器方法（accessor method）。

```java
public class ConstructorTest {
    public static void main(String[] args) {
        LocalDate date = LocalDate.now();
        int month = date.getMonthValue();
        int today = date.getDayOfMonth();
        date = date.minusDays(today - 1);
        DayOfWeek weekday = date.getDayOfWeek();
        int value = weekday.getValue();
        System.out.println("Mon Tue Web Thu Fri Sat Sun");
        for (int i = 1; i < value; i++) {
            System.out.print("    ");
        }
        while (date.getMonthValue() == month) {
            System.out.printf("%3d", date.getDayOfMonth());
            if (date.getDayOfMonth() == today) {
                System.out.print("*");
            } else {
                System.out.print(" ");
            }
            date = date.plusDays(1);
            if (date.getDayOfWeek().getValue() == 1) {
                System.out.println();
            }
        }
        if (date.getDayOfWeek().getValue() != 1) {
            System.out.println();
        }
    }
}
Mon Tue Web Thu Fri Sat Sun
      1   2   3   4   5   6 
  7   8   9  10  11  12  13 
 14  15  16  17  18  19  20 
 21  22  23* 24  25  26  27 
 28  29  30  31
```

&nbsp;

## 3.用户自定义类

主力类（workhorse class），这些类没有main方法，却有自己的实例域和实例方法。

### Employee类

```java
public class EmployeeTest {
    public static void main(String[] args) {
        Employee[] staff = new Employee[3];
        staff[0] = new Employee("Carl Cracker", 75000, 1987, 12, 15);
        staff[1] = new Employee("Harry Hacker", 50000, 1989, 10, 1);
        staff[2] = new Employee("Tony Tester", 40000, 1990, 3, 15);
       
        for (Employee e : staff) {
            e.raiseSalary(5);
        }
        
        for (Employee e : staff) {
            System.out.println("name=" + e.getName() + ",salary=" + e.getSalary() + ",hi reDay="
                    + e.getHireDay());
        }
    }
}

class Employee {
    private String name;
    private double salary;
    private LocalDate hireDay;
    
    public Employee(String name, double salary, int year, int month, int day) {
        this.name = name;
        this.salary = salary;
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
}
```

源文件名是EmployleeTest.java，这是因为文件名必须与public类的名字相匹配。在一个源文件中，只能有一个公有类，但可以有任意数目的非公共类。

当编译这段源代码的时候，编译器将在目录下创建两个类文件：EmployeeTest.class和Employee.class。

将程序中包含main方法的类名提供给字节码解释器，以便启动程序：**java EmployeeTest** 字节码解释器开始运行EmployeeTest类的main方法中的代码。在这段代码中，先后构造了三个新Employee对象，并显示它们的状态。

&nbsp;

### 多个源文件的使用

许多程序员习惯于将每一个类存在一个单独的源文件中。

这样可以有两张编译源程序的方法：

1.使用通配符调用Java编译器：javac Employee*.java

2.键入下列命令：javac EmployeeTest.java

当Java编译器发现EmployeeTest.java使用了Employee类时会查找名为Employee.class的文件。如果没有找到这个文件，就会自动搜索Employee.java，然后进行编译。更重要的是，如果Employee.java版本较已有的Employee.class文件版本新，Java编译器会自动地重新编译这个文件。

&nbsp;

### 剖析Employee类

方法都被标记为public，意味着任何类的任何方法都可以调用这些方法。

三个实例域关键字private确保只有Employee类自身的方法能够访问。可以用public标记实例域，但是一种极为不提倡的做法，public数据域允许任何程序的方法访问和修改，这就完全破坏了封装。

类通常包括类型属于某个类类型的实例域。

&nbsp;

### 从构造器开始

1.构造器与类同名

2.每个类可以有一个以上的构造器

3.构造器可以有0个、1个或多个参数

4.构造器没有返回值

5.构造器总是伴随着new操作一起调用

不要再构造器中定义与实例域重名的局部变量。这些变量只能在构造器的内部访问，屏蔽了同名的实例域。

&nbsp;

### 隐式参数与显式参数

```java
e.raiseSalary(5);
```

raiseSalary方法有两个参数，第一个参数为隐式（implicit）参数，是出现在方法名钱的Employee类对象。第二个参数方法后面括号的数值，是一个显式（explicit）参数。

在每一个方法中，关键字this表示隐式参数：

```java
public void raiseSalary(double byPercent) {
    double raise = this.salary * byPercent / 100;
    this.salary += raise;
}
```

有些人偏爱这种风格，因为可以将实例域与局部变量明显区分开来。

&nbsp;

### 封装的优点

1.可以改变内部实现，除了该类的方法之外，不影响其他代码

2.更改器方法可以执行错误检查，然而直接对域进行赋值将不会进行这些处理。

注意不要编写返回引用可变对象的访问器方法：

```java
class Employee {
    
   private Date hireDay;
   ...
   public Date getHireDay() {
       return hireDay;
   }
   ...
}
```

```java
Employee harry = new Employee("Carl Cracker", 75000);
// 时间使用Date(Date对象是可变的)
System.out.println(harry.getHireDay());
Date d = harry.getHireDay();
double t = 10 * 365.25 *24 * 60 * 60 * 1000;
d.setTime(d.getTime() - (long)t);
System.out.println(harry.getHireDay());
// Mon Oct 28 20:19:18 CST 2019
// Wed Oct 28 08:19:18 CST 2009
```

Date对象是可变的，这点破坏了封装。d和harry.hireDay引用同一对象。对d调用更改器方法就可以自动地改变这个雇员对象的私有状态。

如果需要返回一个可变对象的引用，应该首先对它进行克隆（clone），**对象clone是指存放在另一个位置上的对象副本**。

修改后的代码：

```java
public Date getHireDay() {
    return (Date) hireDay.clone();
}
```

如果需要返回一个可变数据域的拷贝，就应该使用clone。

&nbsp;

### 基于类的访问权限

Employee类的方法可以访问Employee类的任何一个对象的私有域。

&nbsp;

### 私有方法

在java中为了实现一个私有方法，只需将关键字public改为private即可。

只要方法是私有的，它就不会被外部的其他类操作调用，可以将其删去。如果是共有的，就不能删去，因为其他的代码可能依赖它。

&nbsp;

### final实例域

可以将实例域定义为final。构建对象时必须初始化这样的域。也就是说，必须确保在每一个构造器执行之后，这个域的值被设置，并且在后面的操作中，不能够再对它进行修改。

final修饰符大多都应用于基本类型域或不可变类的域（String类就是一个不可变类）。

对于可变的类，会造成混乱：

```java
private final StringBuilder evaluations;
```

构造器中会初始化为：

```java
evaluations = new StringBuilder();
```

final关键字只是表示存储在evaluations变量中的对象引用不会再指示其他StringBuilder对象，不过这个对象可以更改：

```java
public void giveGoldStar () {
    evaluations.append(LocalDate.now() + ": Gold star!\n");
}
```

## 4.静态域与静态方法

### 静态域

如果将域定义为static，每个类中只有一个这样的域。而每一个对象对于所有的实例域却都有一份自己的拷贝。

```java
class Employee {
    private static int nextId = 1;
    private int id;
}
```

每一个雇员对象都有一个自己的id域，但这个类的所有实例将共享一个nextId域。即使没有一个雇员对象，静态域nextId也存在。它属于类，不属于任何一个独立的对象。

```java
class Employee {
    private static int nextId = 1;
    private int id;
    public void setId() {
        this.id = nextId;
        nextId++;
    }
    public int getId() {
        return id;
    }
    public static void main(String[] args
        Employee e1 = new Employee();
        e1.setId();
        
        Employee e2 = new Employee();
        e2.setId();
        System.out.println(e1.getId());
        System.out.println(e2.getId());
    }
    // 1
    // 2
}
```

&nbsp;

### 静态常量

```java
public static final double PI = 3.14159265358979323846;
```

程序中可以采用Math.PI获取这个常量。如果关键字static被省略，PI就变成了Math类的一个实例域。需要通过Math类的对象访问PI，并且每一个Math对象都有它自己的一份PI拷贝。

```java
public class System {
    ...
    public static final PrintStream out = ...;
    ...
}
```

前面说最好不要将域设计为public，然而对于公有常量（final域）却没问题。因为out被声明为final，所以，不允许在将其他打印流赋给它：

```java
System.out = new PrintStream(); // 错误
```

&nbsp;

### 静态方法

静态方法是一种不能向对象实施操作的方法。换句话说就是没有隐式参数。

可以认为静态方法是没有this参数的方法（在一个非静态方法中，this参数表示这个方法的隐式参数。）

Employee类的静态方法不能访问Id实例域，因为它不能操作对象。但是静态方法可以访问自身类中的静态域。

```java
public static int getNextId() {
    return nextId;
}

int n = Employee.getNextId();
```

这个方法可以省略掉static。但是需要通过Employee类对象的引用调用这个方法。

在下面两种情况下使用静态方法：

1.一个方法不需要访问对象状态，其所需参数都是通过显示参数提供（Math.pow）

2.一个方法只需要访问类的静态域（Employee.getNextId）

&nbsp;

### 工厂方法

静态方法还有另一个常见的用途。类似LocalDate和NumberFormat的类使用静态工厂方法类构造对象。

NumberFormat类使用工厂方法生成不同风格的格式化对象：

```java
NumberFormat currencyFormatter = NumberFormat.getCurrencyInstance();
NumberFormat percentFormatter = NumberFormat.getPercentInstance();
double x = 0.1;
System.out.println(currencyFormatter.format(x));
System.out.println(percentFormatter.format(x));
// ￥0.10
// 10%
```

NumberFormat不使用构造器只要有两个原因：

1.无法命名构造器，构造器的名称必须与类名相同。但是这里希望得到货币和百分比实例采用不同的名字。

2.当使用构造器时，无法改变所构造对象类型。而Factory方法将返回一个DecimaFormat类对象，他是NumberFormat的子类。

&nbsp;

### main方法

不需要使用对象调用静态方法。

main方法是一个静态方法，不对任何对象进行操作。事实上，在启动程序时还没有任何一个对象。静态的main方法将执行并创建程序所需的对象。

每个类可以有一个main方法，这是一个常用于对类进行单元测试的技巧。

&nbsp;

## 5.方法参数

**按值调用（call by value）**，表示方法接收的是调用者提供的值。

**按引用调用（call by reference）**，表示方法接收的是调用者提供的变量地址。

一个方法可以修改传递引用所对应的变量值，而不能修改传递值调用所对应的变量值。

Java程序设计语言总是采用按值调用。也就是说，方法得到的是所有参数值的一个拷贝，方法不能修改传递给它的任何参数变量的内容。

```java
public static void main(String[] args) {
    double percent = 10;
    tripleValue(percent);
    System.out.println(percent);
}
static void tripleValue(double x) {
    x = 3 * x;
}
```

方法参数共有两种类型：

1.基本数据类型

2.对象引用

一个方法不可能修改一个基本数据类型的参数。而对象引用作为参数就不同了。

```java
public static void main(String[] args) {
    Employee harry = new Employee("Carl Cracker", 75000, 1987, 12, 15);
    tripleSalary(harry);
    System.out.println(harry.getSalary());
}
static void tripleSalary(Employee x) {
    x.raiseSalary(200);
}
// 225000.0
```

具体执行过程为：

1.x被初始化为harry值的拷贝，这里是一个对象的引用

2.raiseSalary方法应用于这个对象引用。x和harry同时引用的那个雇员对象的薪水提高了200%

3.方法结束后，参数变量x不再使用。当然对象变量harry继续引用那个薪水增至3倍的雇员对象

方法得到的是对象引用的拷贝，对象引用及其他的拷贝同时引用同一个对象。

&nbsp;

Java对象并不是采用引用调用：

```java
public static void main(String[] args) {
    Employee a = new Employee("Carl Cracker", 75000, 1987, 12, 15);
    Employee b = new Employee("Harry Hacker", 50000, 1989, 10, 1);
    swap(a, b);
    System.out.println(a.getName());
    System.out.println(b.getName());
}

static void swap(Employee x, Employee y) {
    Employee temp = x;
    x = y;
    y = temp;
}
// Carl Cracker
// Harry Hacker
```

如果Java使用的是按引用调用，那么这个方法应该可以实现交换数据的效果。但方法并没有改变存储在变量a和b中的对象引用。swap方法的参数x和y被初始化为两个对象引用的拷贝。这个方法交换的是这个拷贝。

这个过程说明：Java程序设计语言对对象采用的不是引用调用，实际上，对象引用是按值传递的。

&nbsp;

Java中方法参数的使用情况：

1.一个方法不能修改一个基本数据类型的参数（即数值型和布尔型）

2.一个方法可以改变一个对象参数的状态

3.一个方法不能让对象参数引用一个新的对象

&nbsp;

## 6.对象构造

### 重载

有些类有多个构造器：

```java
StringBuilder messages = new StringBuilder();
StringBuilder todoList = new StringBuilder("To do:\n");
```

这种特性叫做**重载（overloading）**，如果多个方法有相同的名字、不同的参数，便产生了重载。

重载解析（overloading resolution），编译器通过各个方法给出的参数类型与特定方法调用所使用的值类型进行匹配来挑选出响应的方法。如果找不到匹配的参数，就会产生编译错误。

Java允许任何方法重载。要描述一个方法需要指出方法名以及参数，这叫做方法的**签名（signature）**。返回类型不是方法签名的一部分，也就是说，不能有两个方法名字相同参数类型也相同却返回不同类型值的方法。

&nbsp;

### 默认域初始化

如果构造器没有显式地给域赋值初值，那么就会被自动赋值为默认值：数值为0、布尔值为false、对象引用为null。

这是域与局部变量的主要不同点，必须明确地初始化方法中的局部变量。

```java
Employee harry = new Employee();
LocalDate h = harry.getHireDay();
int year = h.getYear(); // 空指针异常
```

这并不是一个好习惯，getHireDay()方法会得到一个null引用。

&nbsp;

### 无参数的构造器

对象由无参数构造器函数创建时，其状态会设置为适当的默认值。

如果在编写一个类时没有编写构造器，那么系统会提供一个无参数构造器。

如果类中提供了至少一个构造器，但没有提供无参数构造器，则在构造对象时如果没有提供参数就会被视为不合法。

&nbsp;

### 显式域初始化

每个实例域都可以被设置为一个有意义的初值，这是一种很好的习惯。

可以在类定义中，直接将一个值赋值给任何域：

```java
class Employee {
    private String name = "";
    ...
}
```

在执行构造器之前，先执行赋值操作。当一个类的所有构造器都希望把相同的值赋予某个特定实例域时，特别好用。

初始值不一定是常量：

```java
class Employee {
    private static int nextId;
    private int id = assignId();
    
    private static int assignId() {
        int r = nextId;
        nextId++;
        return r;
    }
}
```

&nbsp;

### 参数名

```java
public Employee(String n, double s) {
    name = n;
    salary = s;
}
```

这样过于简单，只有阅读代码才能了解n和s的含义。

有些习惯前面加a：

```java
public Employee(String aName, double aSalary) {
    name = aName;
    salary = aSalary;
}
```

还有一种技巧：参数变量用同样的名字将实例域屏蔽起来，例如参数命名为salary，salary将引用这个参数，而不是实例域，但可以采用this.salary的形式访问实例域，this指示隐式参数，也就是所构造的对象：

```java
public Employee(String name, double salary) {
    this.name = name;
    this.salary = salary;
}
```

&nbsp;

### 调用另一个构造器

关键字this引用方法的隐式参数。

如果构造器的第一个语句形如this(...)，这个构造器将调用同一个类的另一个构造器：

```java
public Employee(double s) {
    this("Employee #" + nextId, s);
    nextId++;
}
```

当调用new Employee(60000)时，Employee(double)构造器将调用Employee(String, double)即可。

&nbsp;

### 初始化块

前面提到两张初始化数据域的方法：

1.在构造器中设置值

2.在声明中赋值

Java还有第三种机制，**初始化块（initialization block）**，在一个类的声明中，可以包含多个代码块，只要构造类的对象，这些块就会被执行：

```java
class Employee {
    private int id;
    private String name;
    private double salary;
    private static int nextId;
    
    {
        id = nextId;
        nextId++;
    }
    
    public static void main(String[] args) {
        Employee e1 = new Employee();
        Employee e2 = new Employee();
        System.out.println(e1.getId());
        System.out.println(e2.getId());
        // 0
        // 1
    }
}
```

无论使用哪个构造器构造对象，id域都在对象初始化块中被初始化，首先运行初始化块，然后执行构造器部分。

&nbsp;

调用构造器的具体步骤：

* 类初始化过程，只执行一次
  * 父类依次执行所有域静态初始化语句和静态初始化块
  * 依次执行所有域静态初始化语句和静态初始化块

* 实例初始化过程

  * 父类所有数据域被初始化默认值
  * 父类按照在类声明中出现的次序，依次执行父级所有域初始化语句和初始化块
  * 实例初始化构造器中调用了父级构造器，执行对应父级构造器，未调用则执行默认父级构造器

  * 所有数据域被初始化默认值

  * 按照在类声明中出现的次序，依次执行所有域初始化语句和初始化块

  * 如果构造器第一行调用了第二个构造器，则执行第二个构造器主体

  * 执行这个构造器主体

如果让类的构造器行为依赖于数据域声明的顺序，那就显得奇怪并且容易引起错误，可以通过提供一个初始值：

```java
private static int nextId = 1;
```

或使用**静态初始化块**，将代码放到一个块中，并标记为static：

```java
static {
    Random generator = new Random();
    nextId = generator.nextInt(10000);
}
```

在类第一次加载的时候，将会进行静态域的初始化。与实力域一样，除非将它们显式设置，否则是默认的初始值。所有的静态初始化语句以及静态初始化块都依照类定义的顺序执行。

&nbsp;

### 对象析构与finalize方法

Java有自动的垃圾回收器，不需要人工回收内存，所有Java不支持析构器。

当然有些对象是有了内存之外的其他资源，当此资源不再需要时，将其回收或再利用将显得十分重要。可以为任何一个类添加**finalize**方法，在垃圾回收器清除对象之前调用，在实际应用中，不要依赖于使用finalize方法回收任何短缺的资源，因为很难知道这个方法什么时候调用。

某个资源在使用完毕后立刻被关闭，那么需要人工管理，对象用完时，可以用于一个close方法来清理。



