正则表达式（regular expression）用于指定字符串的模式，可以在任何需要定位匹配某种特定模式的字符串的情况下使用正则表达式。

简单示例：

[Jj]vaa.+

匹配下列形式的所有字符串：

1.第一个字母是J或j

2.接下来三个字母是ava

3.字符串的其余部分有一个或多个任意字符构成

大多数情况下，一小部分很直观的语法结构就足够了：

1.字符类（character class）是一个括在括号内的可选择字符集。例如[Jj]、[0-9]、[A-Za-z]、[^0-9]，这里“-”表示一个范围，而^表示补集（除了指定字符之外的所有字符）。

2.如果字符类在包含“-”，那么它必须是第一项或最后一项；如果要包含“[”，那么它必须是第一项；如果要包含“^”，那么它可以是除开始位置以外的任何位置。其中“[”和“\”需要转移。

3.有许多预定的字符类，例如\d(数字)和\p{Sc}（Unicode货币符号）。


![img](https://img-blog.csdnimg.cn/20191227152713673.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)


![img](https://img-blog.csdnimg.cn/20191227152801647.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

1.大部分字符都可以与他们自身匹配

2.&nbsp; .符号可以匹配任何字符（可能不包括行终止符，这取决于标志的设置）

3.使用\作为转义字符，例如，\.匹配句号而\\匹配反斜线

4.^和$分别匹配一行的开头和结尾

5.如果X和Y是正则表达式，那么XY表示“任何X的匹配后面跟随Y的匹配”，X|Y表示“任何X或Y的匹配”

6.可以将两次运用到表达式X:X+(1个或多个)、X*(0个或多个)、X?(0个或多个)

7.默认情况下，量词要匹配能够使整个匹配成功的最大可能的重复次数。可以修改这种行为，方法是使用后缀？（使用勉强或吝啬匹配，也就是匹配最小的重复次数）或使用后缀+（使用占有符或贪婪匹配，也就是即使让整个匹配失败，也要匹配最大的重复次数）。

例如，字符串cab匹配[a-z]*ab，但是不匹配[a-z]*+ab。第一种情况，表达式[a-z]*只匹配字符c，使得字符ab匹配该模式的剩余部分；但是贪婪模式[a-z]*+将匹配字符cab，模式的剩余部分将无法匹配。

8.使用群组来定义子表达式，其中群组用括号括起来。例如([+-]?)([0-9]+)，然后可以询问模式匹配器，让其返回每个组的匹配，或者用\n来引用某个群组，其中n是群组号(从\1开始)

例如，下面的正则表达式，描述了十进制和十六进制的整数：

[+-]?[0-9]+|0[Xx][0-9A-Fa-f]+

正则表达式的最简单用法就是测试某个特定的字符串是否与它匹配。首先用表示正则表达式的字符串构建一个Pattern对象。然后从这个模式中获得一个Matcher，并调用它的matches方法：

```java
Pattern pattern = Pattern.compile("");
Matcher matcher = pattern.matcher(input);
if (matcher.matches()) {...}
```

这个匹配器的输入可以是任何实现了CharSequence接口的类的对象，例如String、StringBuilder和CharBuffer。

在编译这个模式时，可以设置一个或多个标志：

```java
Pattern pattern = Pattern.compile(expression, Pattern.CASE_INSENSITIVE + Pattern.UNICODE_CASE);
```

或者在模式中指定它们：

```java
String regex = "(?iU:expression)";
```

下面是各个标志：

1.Pattern.CASE_INSENSITIVE或r：匹配字符时忽略字母的大小，默认情况下，这个标志只会考虑US ASCII字符

2.Pattern.UNICODE_CASE或u：当与CASE_INSENSITIVE组合使用时，用Unicode字母的大小来匹配

3.Pattern.UNICODE_CHARACTER——CLASS或U：选择Unicode字符类代替POSIX，其中蕴含了UNICODE_CASE

4.Pattern.MULYILINE或m：^和$匹配行的开头和结尾，而不是整个输入的开头和结尾

5.Pattern.UNIX_LINES或d：在多行模式中匹配^和$时，只有‘\n’被识别成终止符

6.Pattern.DOTALL或s：当使用这个标志时，.符号匹配所有字符，包括行终止符

7.Pattern.COMMENTS或x：空白字符和注释（从#到行末尾）将被忽略

8.Pattern.LITERAL：该模式将被逐字地采纳，必须精确匹配，因字母大小而造成的差异除外

9.Pattern.CANON_EQ：考虑Unicode字符规范的等价性，例如u后面跟随‘’（分音符号）匹配ü

最后两个标志不能在正则表达式内部指定。

如果想在集合或流中匹配元素，可将模式转换为谓词：

```java
Stream<String> strings = ...;
Stream<String> result = strings.filter(pattern.asPredicate());
```

其结果中包含了匹配正则表达式的所有字符串。

如果正则表达式包含群组，那么Matcher对象可以揭示群组的边界：int start(int groupIndex)，int end(int groupIndex)将产生指定群组的开始索引和结束之后的索引。

可以直接通过调用String group(int groupIndex)抽取匹配的字符串。

群组0是整个输入，而用于第一个实际群组的群组索引是1.调用groupCount方法可以获得全部群组的数量。对于具名的组，使用

int start(String groupName)、int end(String groupName)、String group(String groupName)。

嵌套群组是按照前括号排序的，假如有下面模式：

(([1-9]|1[0-2]):([0-5][0-9]))[ap]m

和下面输出：11:59am

那么匹配器会报告下面的群组：


![img](https://img-blog.csdnimg.cn/20191227171412378.png)

下面程序，输入模式和字符串，如果输入匹配模式，并且模式包含群组，那么将用括号打印出群组边界。

```java
public class RegexTest {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        System.out.println("输入模式：");
        String patternString = in.nextLine();

        Pattern pattern = Pattern.compile(patternString);

        while (true) {
            System.out.println("输入字符串匹配：");
            String input = in.nextLine();
            if (input == null || input.equals("")) {
                return;
            }
            Matcher matcher = pattern.matcher(input);
            if (matcher.matches()) {
                System.out.println("Match");
                int g = matcher.groupCount();
                if (g > 0) {
                    for (int i = 0; i < input.length(); i++) {
                        // 打印任何群组
                        for (int j = 1; j <= g; j++) {
                            if (i == matcher.start(j) && i == matcher.end(j)) {
                                System.out.print("()");
                            }
                        }
                        // 打印（对于从此处开始的非空组
                        for (int j = 1; j <= g; j++) {
                            if (i == matcher.start(j) && i != matcher.end(j)) {
                                System.out.print("(");
                            }
                        }
                        System.out.print(input.charAt(i));
                        for (int j = 1; j <= g; j++) {
                            if (i + 1 != matcher.start(j) && i + 1 == matcher.end(j)) {
                                System.out.print(")");
                            }
                        }
                    }
                    System.out.println();
                }
            } else {
                System.out.println("No match");
            }
        }
    }
}
// 输入模式：
// (([1-9]|1[0-2]):([0-5][0-9]))[ap]m
// 输入字符串匹配：
// 11:59am
// Match
// ((11):(59))am
```

想找出输入中一个或多个匹配的子字符串，可以使用Matcher类的find方法来查找匹配内容，如果返回true，在使用start和end方法来查找匹配的内容，或使用不带引元的group方法来获取匹配的字符串。

```java
public class HrefMatch {
    public static void main(String[] args) {
        try {
            String urlString = "http://horstmann.com";
            InputStreamReader in = new InputStreamReader(new URL(urlString).openStream(), StandardCharsets.UTF_8);
            StringBuilder input = new StringBuilder();
            int ch;
            while ((ch = in.read()) != -1) {
                input.append((char) ch);
            }
            String patternString = "<a\\s+href\\s*=\\s*(\"[^\"]*\"|[^\\s>]*)\\s*>";
            Pattern pattern = Pattern.compile(patternString, Pattern.CASE_INSENSITIVE);
            Matcher matcher = pattern.matcher(input);
            while (matcher.find()) {
                String match = matcher.group();
                System.out.println(match);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

&nbsp;

Matcher类的replaceAll方法将正则表达式出现的所有地方都用替换字符串来替换。例如，下面将所有数字序列都替换成#字符：

```java
Pattern pattern = Pattern.compile("[0-9]+");
Matcher matcher = pattern.matcher("a78w68s6f7r2388cv65a7s6de5f7se");
String output = matcher.replaceAll("#");
```

替换字符串可以包含对模式中群组的引用：$n表示替换成第n个群组，${name}被替换为具有给定名字的组，因此需要用\$来表示在替换文本中包含的一个$字符。

如果字符串中包含$和\，但是又不希望他们被解释成群组的替换符，那么就可以调用matcher.replaceAll(Matcher.quoteReplacement(str))。

replaceFirst方法将只替换模式的第一次出现。

&nbsp;

Pattern类有一个split方法，它可以用正则表达式来匹配边界，从而将输入分割成字符串数组。例如，下面代码可以将输入分割成标记，其中分隔符是由可选的空白字符包围的标点符号。

```java
Pattern pattern = Pattern.compile("\\s*\\p{Punct}\\s*");
String[] tokens = pattern.split("");
// 如果有多个标记，可以惰性的获取它们
Stream<String> stream = pattern.splitAsStream("");
// 如果不关心预编译模式和惰性获取，可以使用String.split方法
String[] tokens = input.split("\\s*,\\s*");
```

**抓取网页中的Email地址**

```java
public class EmainSpider {
    public static void main(String[] args) {
        try {
            BufferedReader br = new BufferedReader(new FileReader("D:\\email.html"));
            String line = "";
            while ((line = br.readLine()) != null) {
                parse(line);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static void parse(String line) {
        Pattern p = Pattern.compile("[\\w[.-]]+@[\\w[.-]]+\\.[\\w]+");
        Matcher m = p.matcher(line);
        while (m.find()) {
            System.out.println(m.group());
        }
    }
}
```

(\\w+)(\\.|_)?(\\w*)@(\\w+)(\\.(\\w+))+

**代码统计小程序**

```java
public class CodeCounter {
    static long normalLines = 0;
    static long commentLines = 0;
    static long whiteLines = 0;

    public static void main(String[] args) {
        File f = new File("E:\\JAVA");
        File[] codeFiles = f.listFiles();
        for (File child : codeFiles) {
            if (child.getName().matches(".*\\.java$")) {
                parse(child);
            }
        }
        System.out.println("normalLines:" + normalLines);
        System.out.println("commentLines:" + commentLines);
        System.out.println("whiteLines:" + whiteLines);
    }

    private static void parse(File f) {
        BufferedReader br = null;
        boolean comment = false;
        try {
            br = new BufferedReader(new FileReader(f));
            String line = "";
            while ((line = br.readLine()) != null) {
                line = line.trim();
                if (line.matches("^[\\s&&[^\\n]]*$")) {
                    whiteLines++;
                } else if (line.startsWith("")) {
                    commentLines++;
                    comment = true;
                } else if (line.startsWith("")) {
                    commentLines++;
                } else if (true == comment) {//区别在于容易查错，当误把==号写作=号时，if ($i=true)不会报错，而且无论$i为何值都会成立，但是写成if (true=$i) 会报错，因为常量无法被赋值。在涉及==的逻辑表达式中，常量写在前面可以有效利用编译器查错机制避免类似 if ($i == true)这样的错误。至于实际功能上，没有任何区别commentLines ++;
                    if (line.endsWith("*/")) {
                        comment = false;
                    }
                } else if (line.startsWith("//")) {
                    commentLines++;
                } else {
                    normalLines++;
                }
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (br != null) {
                try {
                    br.close();
                    br = null;
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

结果是： normalLines:384 commentLines:2 whiteLines:37

**1.Qulifiers** Greedy quantifiers &nbsp; X? X* X+ X{n} X{n,} X{n,m}​ Reluctant &nbsp;quantifiers &nbsp;X?? &nbsp;后多跟一个问号​ Possessive quantifiers &nbsp;X?+ &nbsp; 后多跟一个加号​

&nbsp;

```java
public class Test {
    public static void main(String args[]) {
        Pattern p = Pattern.compile(".{3,10}[0-9]");
        String s = "aaaa5b65bbb6a1";
        Matcher m = p.matcher(s);
        if (m.find()) {
            p(m.start() + "-" + m.end());
        } else {
            p("not match!");
        }
    }
}
```

 结果是：0-10 &nbsp; 此种方式是先吞入最多的字符既是10个字符，发现后面没有跟数字，然后释放以为，检测符合正则表达式​ 在{3,10}后加问号，结果是：0-5，此种方法是先读入最少的3个字符，检测不符合在吞入一个字符，符号所以结果是0-5​ 在{3,10}后加+，结果是not match,这种方式是直接吞入10个字符并且不吐出，所以不匹配​**2.non-capturing** 不捕获字符串​ (?X) (?:X) (?idmsuxU-idmsuxU) (?idmsux-idmsux:X) (?=X) (?!X) (?&lt;=X) (?X)​

&nbsp;

```java
public class Test {
    public static void main(String args[]) {
        Pattern p = Pattern.compile(".{3}(?=a)");
        String s = "444a66b";
        Matcher m = p.matcher(s);
        while(m.find()) {
            p(m.group());
        }
    }
}
```

结果：444，.{3}(?=a)表示，有3个字符以a结尾，打印该3个字符，不包括a，(?=a).{3}时，结果为a66​​(放前面包括) .{3}(?！a)，结果是44a &nbsp;66b &nbsp; &nbsp;(?!a).{3}结果是444 &nbsp;66b​ .{3}(?&lt;！a)，结果是444 &nbsp;a66(从后往前数不是a的) &nbsp; .{3}(?&lt;=a)，结果是44a，反向等同于(?=a).{3}**3.back refenrences向前引用**

&nbsp;

```java
public class Test {
    public static void main(String args[]) {
        Pattern p = Pattern.compile("(\\d(\\d))\\2");
        String s = "122";
        Matcher m = p.matcher(s);
        p(m.matches());
    }
}
```

结果：true​ 表示第3个数是不是和第二组的数匹配，(\\d\\d)\\1，1212结果是true，\\1表示第一个组组过之后的字符串和第一组比较​**4.flags的简写**

&nbsp;

```java
public class Test {
    public static void main(String args[]) {
        Pattern p = Pattern.compile("java", Pattern.CASE_INSENSITIVE);
        p("Java".matches("(?i)(java)"));
    }
}
```

结果：true​​ (?!)=Pattern p = Pattern.compile("java", Pattern.CASE_INSENSITIVE);​ (?i)表示开启忽略大小写，而(?-i)表示关闭忽略大小写

