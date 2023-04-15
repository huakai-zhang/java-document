## 7.包

Java允许使用**包（package）**将类组织起来。借助包可以方便地组织自己的代码，并将自己的代码与别人提供的代码库分开管理。

使用包的主要原因是确保类名的唯一性。比如两个包都有Employee类，只要放在不同的包中，就不会产生冲突。

为了保证包名唯一，Sun建议将公司因特网域名的倒序做为包名。

&nbsp;

### 类的导入

一个类可以使用所属包中的所有类，以及其他包中的公有类。

两种方式访问其他包中的公有类：

```java
java.time.LocalDate today = java.time.LocalDate.now();
```

另一种方式，使用**import**语句：

```java
import java.util.*;
LocalDate today = LocalDate.now();
```

使用java.time.*的语法比较简单，对代码的大小也没有任何负面影响。当然如果可以明确java.time.LocalDate将会使代码的读者更加准确地知道加载了哪些类。

只能使用星号（*）导入一个包，不能使用java.*或者java.*.*导入以java为前缀的所有包。

java.util和java.sql包都有Date类，只能使用完整的包名使用这两个类

在包中定位类是编译器的工作。类文件中的字节码肯定使用完整的包名来引用其他类。

&nbsp;

### 静态导入

import增加了导入静态方法和静态域的功能：

```java
import static java.lang.System.*;
import static java.lang.Math.*;

out.println(sqrt(4));
```

&nbsp;

### 将类放入包中

要想将一个类放入包中，就必须将包的名字放在源文件的开头，包中定义类的代码之前。

```java
package com.spring.corejava;

public class Employee {

}
```

如果没有在源文件中防止package，这个类就被放置在一个默认包中（defaulf package）。默认包没有名字。

将包中文件放到与完整包名匹配的子目录中，编译器将类文件也放在相同的目录结构中。

类分别放在不同的包中，在这种情况下，仍然要从基目录编译和运行类：

```java
javac com/mycompany/PayrollApp.java
java com.mycompany.PayrollApp
```

需要注意，编译器对文件（带有文件分割符和扩展名.java文件）进行操作。而Java解释器加载类（带有.分隔符）

&nbsp;

### 包作用域


![img](https://img-blog.csdn.net/20170204131531938?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

变量没有显式的标为private，默认为包可见，这显然破坏了封装性。在java.awt包中的Window类就是一个典型的示例。

```java
public class Window extends Container {
    String      warningString;
}
```

但这视情况而定，不一定会出问题。如果不把类文件放置在类路径某处的java/awt子目录下，就不能访问java.awt的内部。

从1.2版本开始，JDK的实现者修改了类加载器。明确的禁止加载用户自定义的包名以java.开始的类。可以通过包密封机制来解决将各种包混杂在一起的问题。如果一个包密封起来，就不会再向这个包添加类了。

&nbsp;

## 8.类路径

类存储在文件系统的子目录中，类的路径必须与包名匹配。

类文件也可以存储在**JAR（Java归档）**文件中，在一个JAR中，可以包含多个压缩形式的类文件和子目录，这样既可以节省又可以改善性能。在程序中用到第三方的库文件时，通常会给出一个或多个需要包含的JAR文件。JDK也提供了许多JAR文件。

JAR文件使用**ZIP格式**组织文件和子目录。

&nbsp;

## 9.文档注释

JDK包含了一个很有用的工具，可以有源文件生成一个HTML文档。

&nbsp;

### 注释的插入

**javadoc实用程序（utility）**从下面几个特性中抽取信息：

包、公有类与接口、公有的和受保护的构造器及方法、公有的和受保护的域

注释应该放置在所描述特性的前面，注释以/**开始，并以*/结束。

每个/**...*/文档注释在标记之后紧跟着**自由格式文本（free-form text）**，标记由@开始，如@author或@param。

自由格式文本的第一句应该是一个概要性的句子。javadoc实用程序自动地将这些句子抽取出来形成概要页。在自由格式文本中，可以使用HTML修饰符。要想键入等宽代码，需使用{@code ... }而不是&lt;code&gt;...&lt;/code&gt;这样一来，就不用操心对代码中的&lt;字符转义了。

&nbsp;

### 类注释

类注释必须放在import语句之后，类定义之前。没必要每行开始都是用星号*。

&nbsp;

### 方法注释

每个方法注释必须在所描述方法之前：

@param 变量描述，一个方法的所有@param必须放在一起

@return 描述

@throws 类描述

&nbsp;

### 域注释

只需要对公有域（通常指静态常量）建立文档。

&nbsp;

### 通用注释

类文档注释：

@author 作者姓名

@version 版本

所有文档注释：

@since 产生一个始于条目，对引入特性的版本描述

@deprecated 对类、方法或变量添加一个不再使用的注释，给出取代的建议

@see 引用，超级链接

package.class#method

```java
@see com.mycompany.corejava.Employee#raiseSalary(double)
```

可以省略包名，甚至类名，此时，链接将定位于当前包或当前类，一定要使用**井号#**。

另一种方式：

{@link package.class#method}

&nbsp;

### 包与概述注释

想产生包注释，就需要每个包目录中添加一个单独的文件：

1.提供一个以package.html命名的HTML文件。在标记&lt;body&gt;...&lt;/body&gt;之间的所有文本都会被抽取出来。

2.提供一个以package-info.java命名的Java文件。这个文件必须包含一个初始的以/**和*/界定的Javadoc注释。

&nbsp;

### 注释的抽取

假设HTML文件将被存在docDirectory

1.切换到包含想要生成文档的源文件目录

2.如果是一个包，应该运行：

javadoc -d docDirectory nameOfPackage

或多个包生成文档

javadoc -d docDirectory nameOfPackage1&nbsp;nameOfPackage2 ...

如果在默认包

javadoc -d docDirectory *.java

如果省去-d&nbsp;docDirectory，就将被提取到当前目录，不提倡。

可以使用多种形式的命令行选项对javadoc程序进行跳转。例如，可以使用-author和-version选项在文档中包含@author和@version标记（默认这些标记会被省略）

&nbsp;

### 类设计技巧

1.一定要保证数据私有

2.一定要对数据初始化

3.不要在类中使用过多的基础类型

4.不是所有的域都需要独立的域访问器和域更改器

5.将职责过多的类进行分解

6.类名和方法名要能够体现他们的职责

7.优先使用不可变的类

