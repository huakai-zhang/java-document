## 1.类、超类和子类

### 定义子类

**关键字extends**表示继承。

```java
public class Manager extends Employee {
    添加方法和域
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

关键字extends表明正在构建的新类派生于一个已存在的类。已存在的类被称为**超类、基类或父类**。新类称为**子类、派生类或孩子类**。Java程序员更喜欢用超类和子类。

子类比超类拥有的功能更加丰富，Manager比员工多一份奖金。

在通过扩展超类定义子类的时候，仅需要指出子类与超类的不同之处。



### 覆盖方法

超类中的方法对于子类并不一定适用（Manager的getSalary方法应该返回薪水和奖金的总和）。需要提供新的方法来**覆盖（override）**超类中的这个方法：

```java
public class Manager extends Employee {
    private double bonus;

    @Override
    public double getSalary() {
        return salary+ bonus;
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

然而这个方法并不能运行。**子类不能直接访问超类的私有域**。

```java
public double getSalary() {
    double baseSalary = getSalary();
    return baseSalary + bonus;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

仍不能运行，因为子类也有一个getSalary方法（就是正在实现的这个方法），所以这条语句会导致无限次地调用自己，直到整个程序崩溃。

为此，需要使用特定的super解决这个问题：

```java
@Override
public double getSalary() {
    double baseSalary = super.getSalary();
    return baseSalary + bonus;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

super并不像this一样，super不是一个对象的引用，不能将super赋给另一个对象变量，它只是一个指示编译器调用超类方法的特殊关键字。

在子类中可以增加域、增加方法或覆盖超类的方法，然而不能删除继承的任何域和方法。



### 子类构造器

```java
public Manager(String name, double salary, int year, int month, int day) {
    super(name, salary, year, month, day);
    bonus = 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

这里的super具有不同的含义，语句super(name, salary, year, month, day)是调用超类中含有name,salary,year,day参数的构造器的简写形式。由于子类不能访问超类的私有域，所以必须利用超类构造器对这部分私有域进行初始化。**super调用构造器的语句必须是子类构造器的第一条语句**。

如果子类的构造器没有显式地调用超类构造器，则将自动调用超类默认构造器。如果超类没有不带参数的构造器，并且在子类构造器中又没有显式地调用超类的其他构造器，则Java编译器将报告错误。



this的两个用途：

1.引用隐式参数

2.调用该类其他的构造器

super的两个用途：

1.调用超类方法

2.调用超类构造器

两个关键字在调用构造器时候，有一个共同点，调用构造器的语句只能作为另一个构造器的第一条语句出现。构造参数既可以传递给本类（this）的其他构造器，也可以传递给超类（super）的构造器。



```java
public static void main(String[] args) {
    Manager boss = new Manager("Carl Cracker", 80000, 1987, 12, 15);
    boss.setBonus(5000);

    Employee[] staff = new Employee[3];
    staff[0] = boss;
    staff[1] = new Employee("Harry Hacker", 50000, 1989, 10, 1);
    staff[2] = new Employee("Tommy Tester", 40000, 1990, 3, 15);
    
    for (Employee e : staff) {
        System.out.println(e.getName() + " " + e.getSalary());
    }
    //Carl Cracker 85000.0
    //Harry Hacker 50000.0
    //Tommy Tester 40000.0
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

只有staff[0]计算了奖金加薪水，e.getSalary()调用能够确定应该执行哪个getSalary方法。尽管e声明为Employee类型，但实际上e既可以引用到Employee类型的对象，也可以引用Manager类型的对象。虚拟机知道e实际引用的对象类型，因此能够正确地调用响应的方法。



一个对象变量可以指示多种实际类型的现象被称为**多态（polymorphism）**。

在运行时能够自动地选择调用哪个方法的现象被称为**动态绑定（dynamic binding）**。



### 继承层次

继承不仅限于一个层次。**Java不支持多继承**，有关多多继承的功能Java通过接口实现。

由一个公共超类派生出来的所有类的集合被称为**继承层次（inheritance hierarchy）**。



### 多态

继承关系就是is-a规则，is-a规则的另一种表述法是置换法则。

在Java中，对象变量是多态的。一个Employee对象既可以引用一个Employee类对象，也可以引用一个Employee类的任何一个子类对象。

```java
Manager boss = new Manager(...);
Employee[] staff = new Employee[3];
staff[0] = boss;
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

这个例子中，变量staff[0]与boss引用同一个对象。但编译器将staff[0]看成Employee对象。这意味着可以调用boss.setBonus(5000)，但不能调用staff[0].setBonus(5000)。

然而，不能将一个超类的引用赋给子类变量。

```java
Manager m = staff[i]; // error
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

不是所有的雇员都是经理。

多态的条件：

1.要有继承  

2.要有重写   

3.父类引用指向子类对象



在Java中，子类数组的引用可以转换成超类数组的引用，而不需要采用强制类型转换。

```java
Manager[] managers = new Manager[10];
Employee[] staff = managers;
staff[0] = new Employee(...);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

编译器接纳这个赋值操作，但staff[0]与manager[0]引用的是同一个对象，似乎我们把一个普通雇员擅自归入经理行列中了。当调用manager[0].setBonus(1000)的时候，将会导致调用一个不存在的域，进而搅乱相邻存储空间的内容。

为了确保不发生这类错误，所有数组都要牢记他们的类型，并负责监督仅将类型兼容的引用存储到数组中。例如new manager[10]创建了一个经理数组，如果试图存储一个Employee类型的引用就会发生ArrayStoreException异常。



### 理解方法调用

x.f(args)，隐式参数x声明为C的一个对象。

1.编译器查看对象的声明类型和方法名，编译器会一一列举所有C类中名为f的方法和其超类中访问属性为public且名为f的方法（超类的私有方法不可访问）。

2.编译器将查看调用方法时提供的参数类型。如果在所有名为f的方法中存在一个与提供的参数类型完全匹配，就选择这个方法，这个过程被称为**重载解析（overloading resolution）**。由于允许参数类型转换（int转为double），如果编译器没有找到与参数类型匹配的方法，或者经过转换后有多个方法与之匹配，就会报告错误。

3.如果是private、static、final方法或者构造器，那个编译器将可以准确地知道应该调用哪个方法，这种调用方式称为**静态绑定（static binding）**。与之相对应的是，调用的方法依赖于隐式参数的实际类型，并且运行时实现**动态绑定**。

4.当采用动态绑定调用方法时，虚拟机一定调用与x所引用对象的实际类型最合适的那个类的方法。

假设x的实际类型是D，它是C的子类。如果D类定义了方法f，就直接调用它，否则在D的超类在寻找f，依次类推。

虚拟机预先为每个类创建一个**方法表（method table）**，列出所有的签名和实际调用的方法。如果调用super.f(param)，编译器将对隐式参数超类的方法表进行搜索。



动态绑定有一个重要特性：无需对现存的代码进行修改，就可以对程序进行扩展。



### 阻止继承：final类和方法

不允许扩展的类被称为**final类**。

```java
public final class Executive extends Manager {
    ...
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

类中特定方法也可以被声明为final。子类就不能覆盖这个方法（**final类中的所有方法自动的成为final方法，但不包括域**）。

将方法或类声明为final主要目的是：确保它们不会在子类中改变语义。



### 强制类型转换

进行类型转换的唯一原因是：在暂时忽略对象的实际类型之后，使用对象的全部功能。

```java
Manager boss1 = (Manager) staff[0];
Manager boss2 = (Manager) staff[1]; // error
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

第二句转换会产生一个ClassCastException异常。

应该养成一个良好的程序设计习惯，在转换之前先查看是否能够成功地转换：

```java
if (staff[1] instanceof Manager) {
    Manager boss2 = (Manager) staff[1];
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

x如果为null，x instanceof C不会产生异常，只是会返回false。因为null没有引用任何对象，当然也不会引用C类型的对象。

1.只能在继承层内进行类型转换。

2.在将超类转换成子类之前，应该使用instanceof进行检查。

###   抽象类

如果自上而下在类的继承层次结果中上移，位于上层的类更具有通用性，甚至可能更加抽象。我们只将它作为派生其他类的基类，而不作为想使用的特定实例类。可以使用**abstract关键字**，这样就不需要实现基类的方法了。

包含一个或多个抽象方法的类本身必须被声明为抽象的。

```java
public abstract class Person {
    public abstract String getDescription();
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

除了抽象方法之外，抽象类还可以包含具体数据和具体方法。

```java
public abstract class Person {
    private String name;
    
    public Person(String name) {
        this.name = name;
    }
    
    public abstract String getDescription();
    public String getName() {
        return name;
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

类即使不含抽象方法，也可以声明为抽象类。

抽象类不能实例化。

```java
new Person("Wu");
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

是错误的，可以定义一个抽象类的对象变量，但是它只能引用非抽象子类的对象：

```java
Person p = new Student("Wu", "Economics");
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



### 受保护访问

可以将Employee中的hireDay声明为proteced，而不是私有域，Manager中的方法就可以直接访问它，但Manager类中的方法只能够访问Manager对象中的hireDay域，而不能访问其他Employee对象中的这个域。这种限制有助于避免滥用受保护机制，使得子类只能获得访问受保护域的权利。在实际应用中，要谨慎使用protected属性。

受保护方法更具有实际意义。如果需要限制某个方法的使用，就可以将它声明为protected。这表明子类得到信任，可以正确地使用这个方法，而其他类则不行。这种方法一个最好的示例就是Object中的clone方法。



## 2.Object：所有类的超类

Object类是Java中所有类的始祖，在Java中每个类都是由它扩展而来的。

在Java中，只有基本类型（primitive types）不是对象。

所有数组类型，不过是对象数组还是基本类型的数组都扩展了Object类。



### equals类

在Object类中，这个方法将判断两个对象是否具有相同的引用。

```java
@Override
public boolean equals(Object otherObject) {
    if (this == otherObject) {
        return true;
    }
    if (otherObject == null) {
        return false;
    }
    if (getClass() != otherObject.getClass()) {
        return false;
    }
    Employee other = (Employee) otherObject;
    return Objects.equals(name, other.name) && salary == other.salary && Objects.equals(hireDay, other.hireDay);
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

getClass方法将返回一个对象所属的类。

由于null值调用equals方法会出现NullPointerException异常，所以这里使用Objects.equals方法，如果两个参数都为null，返回true；如果其中一个参数为null，返回false；均不为null，调用a.equals(b)。



在子类定义equals方法时，首先调用超类的equals，如果检查失败，对象就不可能相等。

```java
@Override
public boolean equals(Object otherObject) {
    if (!super.equals(otherObject)) {
        return false;
    }
    Manager other = (Manager) otherObject;
    return bonus == other.bonus;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



### 相等测试与继承

很多人喜欢使用instanceof进行检查类是否匹配。这样不但没有解决otherObject是子类的情况，并还还有可能招致一些麻烦。

java中，instanceof运算符的**前一个操作符是一个引用变量，后一个操作数通常是一个类（可以是接口）**，用于判断***前面的对象是否是后面的类，或者其子类、实现类的实例\***。如果是返回true，否则返回false。

```java
Person p = new Student("Wu", "Economics");
System.out.println(p instanceof Object);
System.out.println(p instanceof Student);
// true
// true
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

Java语言规范要求equals方法具有下面的特性：

1.自反性 x.equals(x) = true

2.对称性

3.传递性

4.一致性，如果x和y引用的对象没有发生变化，反复调用x.equals(y)应该返回同样的结果。

5.对于任意非空引用x.equals(null)应该返回false。



一个完美编写equals方法的建议：

1.显式参数命名为otherObject，稍后需要将它转换成另一个叫做other的变量

2.检测otherObject与this是否引用同一个对象

```java
if (this == otherObject) {
    return true;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

3.检测otherObject是否为null

```java
if (otherObject == null) {
    return false;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

4.比较this与otherObject是否属于同一个类，使用getClass或instanceof检测

●如果子类能够拥有自己的相等概念，则对称性需求将强制采用getClass进行检查

●如果由超类决定相等的概念，那么就可以使用instanceof进行检测，这样可以在不同的子类的对象之间进行相等比较。

5.将otherObject转换为相应的类类型变量

```java
ClassName other = (ClassName)otherObject;
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

6.对所有需要比较的域进行比较

对于数组类型的域，可以使用静态的Arrays.equals方法检测相应的数组元素是否相等。



### hashCode方法

**散列码（hash code）**是由对象导出的一个整型值。散列码是没有规律的，如果x和y是两个不同的对象，x.hashCode()与y.hashCode()基本上不会相同。

String类源码的hashCode方法：

```java
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;
        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```java
String s = "OK";
StringBuilder sb = new StringBuilder(s);
System.out.println(s.hashCode() + " " + sb.hashCode());
String t = new String("OK");
StringBuilder st = new StringBuilder(t);
System.out.println(t.hashCode() + " " + st.hashCode());
// 2524 356573597
// 2524 1735600054
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

sb和st有着不同的散列码，因为在StringBuilder类中没有定义hashCode方法，它的散列码是由Object类默认hashCode方法导出的对象存储地址。

hashCode方法应该返回一个整数数值（负数也可），并合理地自核实例域的散列码，以便能够让各个不同的对象产生的散列码更加均匀。

```java
@Override
public int hashCode() {
    return 7 * Objects.hashCode(name)
            + 11 * Double.hashCode(salary)
            + 13 * Objects.hashCode(hireDay);
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

最好使用null安全方法Objects.hashCode，如果参数为null，这个方法返回0。

还有更好的做法，组合多个散列值时，可使用Objects.hash：

```java
public int hashCode() {
    return Objects.hash(name, salary, hireDay);
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

Equals与hashCode的定义必须一致：如果x.equals(y)返回true，那么x.hashCode()就必须与y.hashCode()具有相同的值。

如果存在数组类型的域，可以使用静态的Arrays.hashCode方法计算一个散列码，这个散列码有数组元素的散列码组成。



### toString方法

toString方法，返回表示对象值的字符串。

绝大多数toString方法都遵循这样的格式：类的名字，随后是一对方括号括起来的域值。

```java
@Override
public String toString() {
    return getClass().getName() + "[name=" + name +
            "salary: " + salary
            + ",hireDay=" + hireDay
            + "]";
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

最好通过getClass().getName()获得类名的字符串，而不要讲类名硬加在toString方法中。

这样的好处是，子类只要调用super.toString方法就可以了

```java
@Override
public String toString () {
    return super.toString()
            + "[bonus=" + bonus +
            "]";
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

随处可见toString方法的主要原因是：只要对象与一个字符串通过操作符 “+”连接起来，Java编译就会自动调用toString方法。

在调用x.toString()的地方可以用 "" + x代替，与toString不同的是，如果x是基础类型，这句话也可以执行。

如果x是任意对象，调用System.out.println(x)，println方法就会直接地调用x.toString方法，并打印出得到的字符串。

Object类定义了toString方法，用来打印输出对象所属的类名和散列码。

```java
System.out.println(System.out);
// java.io.PrintStream@14ae5a5
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

PrintStream类的设计者没有覆盖toString方法。



但是素组继承了Object类的toString方法，数组类型将按照旧的格式打印：

```java
int[] numbers = { 2, 3, 5, 7 };
String ss = "" + numbers;
System.out.println(ss);
System.out.println(Arrays.toString(numbers));
// [I@7f31245a
// [2, 3, 5, 7]
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

[I表明是一个整型数组，修正的方式是调用静态方法Arrays.toString方法，多维数组则需要调用Arrays.deepToString方法。



## 3.泛型数组列表

一旦确定了数组的大小，改变它就不太容易。在Java中解决这个问题最简单的方法是使用Java中另一个被称为ArrayList的类，在添加和删除元素时，具有自动调节数组容量的功能。 ArrayList是一个采用了类型参数（type parameter）的泛型类（generic class）。

```java
ArrayList<Employee> staff = new ArrayList<>();
staff.add(new Employee("Carl Cracker", 75000, 1987, 12, 15));
staff.add(new Employee("Harry Hacker", 50000, 1989, 10, 1));
staff.add(new Employee("Tony Tester", 40000, 1990, 3, 15));
```

Java SE7中可以省去new ArrayList()的类型参数。 数组列表是管理着对象引用的一个内部数组。最终数组的全部空间有可能被用尽。如果调用add且内部数组已经满了，数组列表将自动地创建一个更大的数组，并将所有的对象从较小的数组中拷贝到较大的数组中。 如果已经清楚或可以估计出数组可能存储的元素数量，就可以在填充数组之前调用ensureCapscity方法。

```java
staff.ensureCapacity(100);
```

这个方法调用将分配一个包含100个对象的内部数组，然后调用100次add，而不用重新分配空间。 还可以把初始容量传递给ArrayList构造器：

```java
ArrayList<Employee> staff = new ArrayList<>(100);
Employee[] employees = new Employee[100];
System.out.println(staff.size());
System.out.println(employees.length);
// 0
// 100
```

如果数组分配100个元素的存储空间，数组就有100个空位置可以使用。而容量为100个元素的数组列表只是拥有保存100个元素的千里，但是在最初，甚至完成初始化构造后，数组列表根本不会有任何元素。

一旦能够确认数组列表的大小不再发生变化，就可以调用trimToSize()方法。这个方法将存储区域的大小跳转为当前元素数量所需的存储空间数目。垃圾回收器将回收多余的存储空间。

### 访问数组列表元素

ArrayList类并不是Java程序设计语言的一部分：它只是一个有某些人编写且放在标准库中的一个实用类。 使用get和set方法实现访问或改变数组元素的操作。

```java
staff.set(i, harry);
Employee e = staff.get(i);
```

```java
ArrayList<Employee> staff = new ArrayList<>(100); // capacity 100, size 0
staff.set(0, x); // 错误，还没有元素0
```

使用add方法为数组添加新元素，而不要使用set方法，它只能替换数组中已存在的元素。

除了在数组列表的尾部追加元素之外，还可以在数组列表的中间插入元素：

```java
int n = staff.size() / 2;
staff.add(n, e);
```

为了插入一个新元素，位于n之后的所有元素都要向后移动一个位置。插入元素后，数组列表大小超过了容量，就会被重新分配存储空间。 同样地，删除一个元素：

```java
Employee e = staff.remove(n);
```

位于n后的所有元素向前移动一个位置，并且数组大小减1。 对于数组插入和删除元素的操作其效率比较低。对于小型数组来说，这点不必担心。但是如果存储的元素数比较多，又经常在中间位置插入、删除元素，应该考虑链表。

for each遍历数组列表：

```java
for (Employee e : staff) {
	...
}
```

### 类型化与原始数组列表的兼容性

```java
public class EmployeeDB {
	public void update(ArrayList list) {...}
	public ArrayList find (String query) {...}
}
```

```java
ArrayList<Employee> staff = ...;
employeeDB.updatye(staff);
```

尽管编译器没有给出任何错误信息或警告，但是这样调用并不太安全。 相反的，将一个元素ArrayList赋给一个类型化ArrayList会得到一个警告。

```java
ArrayList<Employee> result = employeeDB.find(query); //警告
```

这就是Java中不尽人意的参数化类型的限制所带来的结果。鉴于兼容性的考虑，编译器在对类型转换进行检查之后，如果没有发现违反规则的现象，就讲所有类型化数组列表转换成原始ArrayList对象。在程序运行时，所有的数组列表都是一样的，即没有虚拟机中的类型参数。 因此，类型转换ArrayList和ArrayList将执行相同的运行时检查。 这种情形下，只要确保不会造成严重后果，可以使用@SuppressWarnings(“unckecked”)标注来标记这个变量能够接受类型转换：

```java
@SuppressWarnings("unckecked")
ArrayList<Employee> staff = (ArrayList<Employee>)employeeDB.find(query);
```

## 4.对象包装器与自动装箱

所有的基本类型都有一个与之对应的类，这些类称为包装器（wrapper）。 Integer、Long、Float、Double、Short、Byte、Character、Void、Boolean，前6个类派生于公共的超类Number。 对象包装器类是不可变的，即一旦构造了包装器，就不允许更改包装在其中的值。同时，对象包装器类还是final，因此不能定义它们的子类。

```java
public final class Integer extends Number implements Comparable<Integer> {

}
```

```java
ArrayList<Integer> list = new ArrayList<>();
list.add(3);
```

调用list.add(3)将自动地变换成lits.add(Integer.valueOf(3))，这种变换被称为自动装箱（autoboxing）。 相反的，将一个Integer对象赋给一个int值时，将会自动拆箱。

甚至在算术表达式中也能自动地装箱拆箱：

```java
Integer n = 3;
n++;
```

编译器将插入一条对象拆箱指令，然后进行自增计算，最后再将结果装箱。 在两个包装器对象比较时调用equals方法。

```java
Integer n = null;
System.out.println(2 * n);
```

包装器类引用可能为null，所以自动装箱有可能会抛出一个NullPointException。

```java
Integer n = 1;
Double x = 2.0;
System.out.println(true ? n : x);
```

如果在一个条件表达式中混合使用Integer和Double类型，Intefer值就会拆箱，提升为double，再装箱为Double。

```java
Integer n = Integer.valueOf(1);
Double x = 2.0D;
System.out.println((double)n.intValue());
```

上面是编译器生成.class文件，装箱和拆箱是编译器认可的，而不是虚拟机。编译器生成类的字节码时，插入必要的方法调用，虚拟机只是执行这些字节码。

包装器类中还有一些基本方法： 字符串转为整型 int x = Integer.parseInt(s); 这与Integer对象没有任何关系，parseInt是一个静态方法，但是Integer类是放置这些方法的好地方。

```java
public static void triple(Integer x) {
	x = 3 * x;
}
```

不能使用包装器类创建修改数值参数的方法。因为包装器类是不可变的：包含在包装器中的内容不会改变。

## 5.参数数量可变的方法

在Java SE 5.0以后的版本，提供了可以用可变的参数数量调用的方法。

```java
public class PrintStream {
	public PrintStream printf(String format, Object ... args) {
  	  	return format(format, args);
	}
}
```

这里的省略号…是Java代码的一部分，它表明这个方法可以接收任何数量的对象。

实际上printf方法接收两个参数，一个是格式字符串，另一个是Object[]数组。现在将扫描format字符串，并将第i个格式说明符与args[i]的值匹配起来。 用户可以自己定义可变参数方法，并将参数指定为任意类型，甚至是基本类型：

```java
public static void main(String[] args) {
    System.out.println(max(3.1, 40.4, -5));
}
public static double max(double... values) {
    double largest = Double.NEGATIVE_INFINITY;
    for (double v : values) {
        if (v > largest) {
            largest = v;
        }
    }
    return largest;
}
```

允许将一个数组传递给可变参数方法的最后一个参数。

```java
System.out.printf("%d %s", new Object[] { new Integer(1), "widgets"});
```

因此可以将已经存在且最后一个参数是数组的方法重新定义为可变参数的方法，而不会破坏已存在的代码。

## 6.枚举类

```java
public enum Size { SMALL, MEDIUM, LARGE, EXTRA_LARGE }
```

实际上，这个声明定义的类型是一个类，它刚好有4个实例，在此尽量不要构造新对象。 因此，在比较两个枚举类型值时，永远不需要调用equals，而直接使用==就可以了。

```java
public enum Size {
    SMALL("S"), MEDIUM("M"), LARGE("L"), EXTRA_LARGE("XL");
    
    private String abbreviation;
    
    Size(String abbreviation) {
        this.abbreviation = abbreviation;
    }
    public String getAbbreviation() {
        return abbreviation;
    }
}
```

可以在枚举类型中添加一些构造器、方法和域，当然构造器只是在构造枚举常量时候被调用。

所有枚举类型都是Enum类的子类。它们继承了这个类的许多方法。 toString，返回枚举常量名。 toString的逆方法是静态方法valueOf。 每个枚举类型都有一个静态的values方法，它将返回一个包含全部枚举值的数组： Size[] values = Size.values(); 返回包含元素Size.SMALL,Size. MEDIUM,Size. LARGE, Size.EXTRA_LARGE的数组。

```java
public static void main(String[] args) {
    Scanner in = new Scanner(System.in);
    System.out.print("输入一个尺寸：(SMALL, MEDIUM, LARGE, EXTRA_LARGE)");
    String input = in.next().toUpperCase();
    Size size = Enum.valueOf(Size.class, input);
    System.out.println("size=" + size);
    System.out.println("abbreviation=" + size.getAbbreviation());
    if (size == Size.EXTRA_LARGE) {
        System.out.println("good job--you paid attention to the _.");
    }
}
```

## 7.反射

能够分析类能力的程序成为反射（reflective）。反射机制可以用来： 1.在运行时分析类的能力 2.在运行时查看对象 3.实现通用的数组操作代码 4.利用Method对象

### Class类

在程序运行期间，Java运行时系统始终为所有对象维护一个被称为运行时的类型标识。这个信息跟踪着每个对象所属的类。虚拟机利用运行时类型信息选择相应的方法执行。可以通过专门的Java类访问这些信息，保存这些信息的类被称为Class，Object类中的getClass方法可以返回一个Class类型的实例：

```java
Employee e = new Employee("Carl Cracker", 75000, 1987, 12, 15);
Class cl = e.getClass();
System.out.println(cl.getName() + " " + e.getName());
// chapter4.section3.Employee Carl Cracker
```

Class方法getName()，返回类的名字，如果类在一个包里，包的名字也作为类名的一部分。 还可以调用静态方法forName获得类名对应Class对象：

```java
String className = "java.util.Random";
Class c = Class.forName(className);
```

这个方法只有在className是类名或者接口名的时候才能使用。否则forName方法将抛出一个已检查异常(checkedexception)，ClassNotFoundException。

如果T是任意的Java类型（或void关键字），T.class将代表匹配的类对象：

```java
Class cl1 = Random.class;
Class cl2 = int.class;
Class cl3 = Double[].class;
System.out.println(cl3.getName());
// [Ljava.lang.Double;   应用于数组会返回一个奇怪的名字
```

一个Class对象实际上表示一个类型，而未必是一种类。 Class类实际上是一个泛型类，Employee.class的类型是Class&lt; Employee &gt;。

利用==运算符实现两个类对象比较的操作：

```java
System.out.println(e.getClass() == Employee.class);
// true
```

newInstance方法调用默认构造器，创建一个与e具有相同类型的实例。如果没有默认构造器将会抛出异常。如果需要以这种方式向希望按名称创建的类的构造器提供参数，就必须使用Constructor类中的newInstance方法。

```java
String className = "java.util.Random";
Object m = Class.forName(className).newInstance();

Employee l = e.getClass().newInstance();
```

### 捕获异常

抛出异常比终止程序要灵活得多，因为可以提供一个捕获异常的==处理器（handler）==对异常进行处理。 异常有两种类型：未检查异常和已检查异常。对于已检查异常，编译器会检查是否提供了处理器。应精心编写程序避免未检查异常的出现。 一个简单实现的处理器：

```java
try {
    String name = "java.util.Random";
    Class cl = Class.forName(name);
} catch (Exception e) {
    e.printStackTrace();
}
```

如果类名不存在，则跳出try块，直接进入catch子句（Throwable类的printStackTrace方法打印出栈的轨迹，Throwable是Exception的超类）。如果没有异常，将会跳过catch子句。 如果调用一个方法抛出已检查异常，而又没有处理器，编译器会给出错误提示。

### 利用反射分析类的能力

检查类的结构： java.lang.reflect包中有三个类Field，Method，Constructor分别用于描述类的域、方法和构造器。 Class类中的getFields、getMethods、和getConstructors方法分别返回提供的public域、方法和构造器数组，其中包括超类的公有成员。Class类的getDeclareFields、getDeclareMethods、和getDeclareConstructors返回类中声明的全部域、方法和构造器，其中包括私有和受保护成员，但不包括超类成员。

```java
public class ReflectionTest {
    public static void main(String[] args) {
        // 读取类名
        String name;
        if (args.length > 0) {
            name = args[0];
        } else {
            Scanner in = new Scanner(System.in);
            System.out.println("输入类名(例如：java.util.Date)：");
            name = in.next();
        }
        try {
            Class cl = Class.forName(name);
            Class supercl = cl.getSuperclass();
            String modifiers = Modifier.toString(cl.getModifiers());
            if (modifiers.length() > 0) {
                System.out.print(modifiers + " ");
            }
            System.out.print("class " + name);
            if (supercl != null && supercl != Object.class) {
                System.out.print(" extends " + supercl.getName());
            }
            System.out.print("\n{\n");
            printConstructors(cl);
            System.out.println();
            printMethods(cl);
            System.out.println();
            printFields(cl);
            System.out.println("}");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        System.exit(0);
    }
    /**
     * 打印所有构造器
     * @param cl
     */
    public static void printConstructors(Class cl) {
        Constructor[] constructors = cl.getDeclaredConstructors();
        for (Constructor c : constructors) {
            String name = c.getName();
            System.out.print("    ");
            String modifiers = Modifier.toString(c.getModifiers());
            if (modifiers.length() > 0) {
                System.out.print(modifiers + " ");
            }
            System.out.print(name + "(");
            // 打印参数类型
            Class[] paramTypes = c.getParameterTypes();
            for (int j = 0; j < paramTypes.length; j++) {
                if (j > 0) {
                    System.out.print(", ");
                }
                System.out.print(paramTypes[j].getName());
            }
            System.out.println(");");
        }
    }
    /**
     * 打印所有方法
     */
    public static void printMethods(Class cl) {
        Method[] methods = cl.getDeclaredMethods();
        for (Method m : methods) {
            Class retType = m.getReturnType();
            String name = m.getName();
            System.out.print("    ");
            String modifiers = Modifier.toString(m.getModifiers());
            if (modifiers.length() > 0) {
                System.out.print(modifiers + " ");
            }
            System.out.print(retType.getName() + " " + name + "(");
            // 打印参数类型
            Class[] paramTypes = m.getParameterTypes();
            for (int j = 0; j < paramTypes.length; j++) {
                if (j > 0) {
                    System.out.print(", ");
                }
                System.out.print(paramTypes[j].getName());
            }
            System.out.println(");");
        }
    }
    /**
     * 打印所有的域
     */
    public static void printFields(Class cl) {
        Field[] fields = cl.getDeclaredFields();
        for (Field f : fields) {
            Class type = f.getType();
            String name = f.getName();
            System.out.print("    ");
            String modifiers = Modifier.toString(f.getModifiers());
            if (modifiers.length() > 0) {
                System.out.print(modifiers + " ");
            }
            System.out.println(type.getName() + " " + name + ";");
        }
    }
}
```

### 在运行时使用反射分析对象

利用反射机制可以查看在编译时还不清楚的对象域。 查看域的关键是Field类中的get方法，如果f是一个Field类型的对象，obj是某个包含f域的类的对象，f.get(obj)将返回一个对象，其值为obj域的当前值：

```java
public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
    Employee obj = new Employee("Carl Cracker", 75000, 1987, 12, 15);
    Class cl = obj .getClass();
    Field f = cl.getDeclaredField("name");
    Object v = f.get(obj );
}
```

上述方法，由于name是一个私有域，将会抛出一个IllegalAccessException。没有访问权限，Java安全机制只允许查看任意对象有哪些域，而不允许读取值。 反射机制默认行为受限于Java的访问控制。如果一个Java程序没有受到安全管理器的控制，就可以覆盖访问控制。为了达到这个目的，需要调用Field、Method或Constructor对象的setAccessible方法。

```java
f.setAccessible(true);
```

setAccessible是AccessibleObject类的一个方法，它是Field、Method和Constructor的公共超类。这个特性是为了调试、持久存储和相似机制提供的。 get方法如果处理数值类型时，如果是double类型，可以使用Field类的getDouble方法，也可调用get方法，此时，反射机制将会自动将这个域值打包到相应的对象包装器中。 调用f.set(obj，value)可以将obj对象的f域设置成新值。

下面程序提供一个可供任意类使用的同样toString方法。泛型toString方法需要解释几个复杂问题。循环引用将有可能导致无限递归，因此ObjectAnalyzer将记录已经被访问过的对象：

```java
public String toString() {
	return new ObjectAnalyzer().toString(this);
}
```

```java
public class ObjectAnalyzer {
    private ArrayList<Object> visited = new ArrayList<>();

    public String toString(Object obj) {
        if (obj == null) {
            return "null";
        }
        if (visited.contains(obj)) {
            return "...";
        }
        visited.add(obj);
        Class cl = obj.getClass();
        if (cl == String.class) {
            return (String) obj;
        }
        if (cl.isArray()) {
            String r = cl.getComponentType() + "[]{";
            for (int i = 0; i < Array.getLength(obj); i++) {
                if (i > 0) {
                    r += ",";
                }
                Object val = Array.get(obj, i);
                if (cl.getComponentType().isPrimitive()) {
                    r += val;
                } else {
                    r += toString(val);
                }
            }
            return r + "}";
        }
        String r = cl.getName();
        do {
            r += "[";
            Field[] fields = cl.getDeclaredFields();
            AccessibleObject.setAccessible(fields, true);

            for (Field f : fields) {
                if (!Modifier.isStatic(f.getModifiers())) {
                    if (!r.endsWith("[")) {
                        r += ",";
                    }
                    r += f.getName() + "=";
                    try {
                        Class t = f.getType();
                        Object val = f.get(obj);
                        if (t.isPrimitive()) {
                            r += val;
                        } else {
                            r += toString(val);
                        }
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }
            r += "]";
            cl = cl.getSuperclass();
        }
        while (cl != null);
        return r;
    }
}
```

```java
public static void main(String[] args) {
    ArrayList<Integer> squares = new ArrayList<>();
    for (int i = 1; i <= 5; i++) {
        squares.add(i * i);
    }
    System.out.println(new ObjectAnalyzer().toString(squares));
}
```

### 使用反射编写泛型数组代码

java.lang.reflect包中的Array类允许动态地创建数组。比如Array.copyOf，这个方法用于扩展已经填满的数组。 一个从开始就是Object[]的数组永不能转换成Employee[]数组。 Array类的静态方法newInstance，它能够构造新数组，在调用它时必须提供两个参数，一个是数组的元素类型，一个是数组的长度。 要获得新数组元素类型，就需要进行以下工作： 1.获得a数组的类对象 2.确认它是一个数组 3.使用Class类的getComponentType方法确定数组对应的类型

整型数组类型int[]可以被转换成Object，但不能转换成对象数组。所以goodCopyOf的参数声明为Object类型：

```java
public class CopyOfTest {
    public static void main(String[] args) {
        int[] a = { 1, 2, 3};
        a = (int[]) goodCopyOf(a, 10);
        System.out.println(Arrays.toString(a));
        String[] b = { "Tom", "Dick", "Harry" };
        b = (String[]) goodCopyOf(b, 10);
        System.out.println(Arrays.toString(b));
        System.out.println("下面将抛出一个异常");
        b = (String[]) badCopyOf(b, 10);
    }
    /**
     * 此方法试图通过分配新数组并复制所有元素来增大数组
     * 返回值进行转换将会抛出一个异常
     */
    public static Object[] badCopyOf(Object[] a, int newLength) {
        Object[] newArray = new Object[newLength];
        System.arraycopy(a, 0, newArray, 0, Math.min(a.length, newLength));
        return newArray;
    }
    /**
     * 此方法通过分配相同类型的新数组并复制所有元素来增大数组
     */
    public static Object goodCopyOf(Object a, int newLength) {
        Class cl = a.getClass();
        if (!cl.isArray()) {
            return null;
        }
        Class componentType = cl.getComponentType();
        int length = Array.getLength(a);
        Object newArray = Array.newInstance(componentType, newLength);
        System.arraycopy(a, 0, newArray, 0, Math.min(length, newLength));
        return newArray;
    }
}
```

### 调用任意方法

在Method类中有一个invoke方法，它允许调用包装在当前Method对象中的方法，invoke方法的签名是： Object invoke(Object obj, Object… args) 第一个参数是隐式参数，其余的对象提供了显示参数（在Java SE 5.0以前的版本中，必须传递一个对象数组，如果没有显示参数就传递一个null），对于静态方法，第一个参数可以被忽略，即可以将它设置为null。

假设ml代表Employee类的getName方法：

```java
String n = (String) ml.invoke(harry);
```

如果返回类型是基本类型，invoke方法会返回包装器类型。

Class类中的getMethod方法，有可能存在若干个相同名字的方法，还必须提供想要的方法的参数类型： Method getMethod(String name, Class… parameterTypes);

```java
public class MethodTableTest {
    public static void main(String[] args) throws Exception {
        Method square = MethodTableTest.class.getMethod("square", double.class);
        Method sqrt = Math.class.getMethod("sqrt", double.class);
        printTable(1, 10, 10, square);
        printTable(1, 10, 10, sqrt);
    }
    public static double square(double x) {
        return  x * x;
    }
    public static void printTable(double from, double to, int n, Method f) {
        System.out.println(f);
        double dx = (to - from) / (n - 1);
        for (double x = from; x <= to; x+= dx) {
            try {
                double y = (Double) f.invoke(null, x);
                System.out.printf("%10.4f | %10.4f%n", x, y);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```

invoke的参数和返回值必须是Object类型。这就意味着要进行多次类型转换。这就会使出错几率变大。 建议Java开发中不要使用Method对象的回调功能。使用接口回调使得代码的执行速度更快，更易于维护。

## 8.继承的技巧设计

1.将公共操作和域放在超类 2.不要使用受保护的域 3.使用继承实现is-a关系 4.除非所有继承的方法都有意义，否则不要使用继承 5.在覆盖方法时，不要改变预期的行为 6.使用多态，而非类型信息 7.不要过多地使用反射
