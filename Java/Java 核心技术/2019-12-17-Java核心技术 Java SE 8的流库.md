---
layout:  post
title:   Java核心技术 Java SE 8的流库
date:   2019-12-17 19:14:13
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-Java核心技术

---



## 1.从迭代到流的操作

在处理集合时，通常迭代遍历它的元素：

```java
String contents = new String(Files.readAllBytes(Paths.get("alice30.txt")), StandardCharsets.UTF_8);
List<String> words = Arrays.asList(contents.split("\\PL+"));
long count = 0;
for (String w: words) {
    if (w.length() > 12) {
        count++;
    }
}
```

在使用流时：

```java
long count = words.stream().filter(w -> w.length() > 12).count();
```

流的版本比循环版本要更易于阅读，因为不必扫描整个代码去查找过滤和计数操作，方法名就可以直接看到代码意欲何为。 循环需要非常详细地指定操作的顺序，而流却能够以其想要的任何方式来调度这些操作，只要结果正确即可。 将stream修改为parallelStream就可以让流库以并行方式来执行过滤和计数。 流遵循了做什么而非怎么做的原则。

流和集合的显著差异： 1.流并不存储其元素。这些元素可能存储在底层的集合中，或者是按需生成。 2.流的操作不会修改其数据源。 3.流的操作尽可能惰性执行的。这意味着直至需要其结果时，操作才会执行。

实例代码：stream会产生一个用于words列表的stream，filter方法会返回另一个流，其中只包含长度大于12的单词。count方法会将这个流化简为一个结果。

操作流时典型的三个阶段的操作管道： 1.创建一个流 2.指定将初始化流转换为其他流的中间操作，可能包含多个步骤 3.应用终止操作，从而产生结果。这个操作会强制执行之前的惰性操作。从此之后，这个流就再也不能用了。

## 2.流的创建

数组可以用静态的Stream.of方法：

```java
Stream<String> words = Stream.of(contents.split("\\PL+"));
```

of方法具有可变长参数，因此可以具有任意数量引元的流：

```java
Stream<String> song = Stream.of("gently", "down", "the", "stream");
```

使用Array.stream(arry, from, to)可以数组中位于from(包括)和to(不包括)的元素中创建一个流。 Stream.empty创建不包含任何元素的流。

Stream接口有两个用于创建无限流的静态方法。generate方法会接受一个不包含任何引元的函数（从技术上讲，是一个Supplier&lt; T &gt;接口的对象）。只需要一个流类型的值，该函数就会调用用以产生一个这样的值。获取一个常量值的流：

```java
Stream<String> echos = Stream.generate(() -> "Echo"); // 无限个Echo
```

获取一个随机数的流：

```java
Stream<Double> randoms = Stream.generate(Math::random);
```

为了产生无限序列（0 1 2 3）可以使用iterate方法。它会接受一个种子值，以及一个函数（OnaryOperation&lt; T &gt;），并反复将该函数应用到之前的结果：

```java
Stream<BigInteger> integers = Stream.iterate(BigInteger.ZERO, n -> n.add(BigInteger.ONE));
```

第一个元素是种子BigInteger.ZERO，第二个是f(seed)，第三个是f(f(seed))。

Patteran类有一个splitAsStream方法，它会按照某个正则表达式来分割一个CharSequence对象：

```java
Stream<String> ws = Pattern.compile("\\PL+").splitAsStream(contents);
```

静态的Files.lines方法会返回一个包含了文件所有行的Stream：

```java
Stream<String> lines = Files.lines(Paths.get("alice30.txt"));
```

```java
public class CreatingStreams {
    public static <T> void show(String title, Stream<T> stream) {
        final int SIZE = 10;
        List<T> firstElements = stream.limit(SIZE + 1).collect(Collectors.toList());
        System.out.println(title + ": ");
        for (int i = 0; i < firstElements.size(); i++) {
            if (i > 0) {
                System.out.print(", ");
            }
            if (i < SIZE) {
                System.out.print(firstElements.get(i));
            } else {
                System.out.print("...");
            }
        }
        System.out.println();
    }

    public static void main(String[] args) throws IOException {
        Path path = Paths.get("alice30.txt");
        String contents = new String(Files.readAllBytes(path), StandardCharsets.UTF_8);

        Stream<String> words = Stream.of(contents.split("\\PL+"));
        show("words", words);
        Stream<String> song = Stream.of("gently", "down", "the", "stream");
        show("song", song);

        Stream<String> echos = Stream.generate(() -> "Echo");
        show("echos", echos);

        Stream<Double> randoms = Stream.generate(Math::random);
        show("randoms", randoms);

        Stream<BigInteger> integers = Stream.iterate(BigInteger.ZERO, n -> n.add(BigInteger.ONE));
        show("integers", integers);

        Stream<String> wordsAnotherWay = Pattern.compile("\\PL+").splitAsStream(contents);
        show("wordsAnotherWay", wordsAnotherWay);

        Stream<String> lines = Files.lines(path, StandardCharsets.UTF_8);
        show("lines", lines);
    }
}
```

## 3.filter、map和faltMap方法

filter转换会产生一个流，它的元素与某种条件想匹配，filter的引元是Predicate&lt; T &gt;，即从T到boolean的函数：

```java
list.stream().filter(w -> w.length() > 12);
```

按照某种方式来转换流中的值，可以使用map方法并传递执行该转换的函数，该函数应用到每个元素上，并且其结果是包含了应用该函数后所产生的所有结果的流：

```java
// 包含所有单词的首字母
Stream<String> firstLetters = list.stream().map(s -> s.substring(0, 1));
```

```java
public static void main(String[] args) {
    List<String> words = new ArrayList<>();
    words.add("Amy");
    words.add("Mack");
    words.add("boat");
    // 得到一个包含流的流
    Stream<Stream<String>> result = words.stream().map(w -> letters(w));
    
}
// 返回包含值的流
public static Stream<String> letters(String s) {
    List<String> result = new ArrayList<>();
    for (int i = 0; i < s.length(); i++) {
        result.add(s.substring(i, i + 1));
    }
    return result.stream();
}
```

为了得到字母流，可以使用flatMap方法而不是map方法：

```java
Stream<String> result = words.stream().flatMap(w -> letters(w));
```

## 4.抽取子流和连接流

调用stream.limit(n)会返回一个新的流，它在n个元素之后结束（如果原来的流更短，那么就会在流结束时结束）。

```java
Stream<Double> randoms = Stream.generate(Math::random).limit(100);
```

调用stream.skip(n)正好相反，它会丢弃前n个元素。 Stream类的静态的concat方法可以将两个流连接起来，当然第一个流不应该是无限的，否则第二个流永远都不会得到处理的机会：

```java
Stream<String> combined = Stream.concat(letters("Hello"), letters("World"));
```

## 5.其他的流转换

distinct方法会返回一个流，它的元素是从原有流中产生的，即原来的元素按照同样的顺序剔除重复元素后产生，这个流显然能够记住它已经看过的元素：

```java
Stream<String> uniqueWords = Stream.of("merrily", "merrily", "merrily", "merrily").distinct();
```

流的排序，一种用于操作Comparable元素的流，另一种可以接受一个Comparator。

```java
Stream<String> longestFirst = words.stream().sorted(Comparator.comparing(String::length).reversed());
```

当然对集合排序可以不使用流，但是当排序处理是流管道的一部分时，sorted方法就很有用了。

peek方法会产生另一个流，它的元素与原来的元素相同，但是每次在获取元素时，都会调用一个函数（对调试很方便）：

```java
Object[] powers = Stream.iterate(1.0, p -> p * 2).peek(e -> System.out.println("Fetching " + e)).limit(20).toArray();
```

当实际访问一个元素时，就会打印一条消息，通过这种方式，可以验证iterate返回的无限流是被惰性处理的。

## 6.简单约简

约简是一种终结操作（terminal operation），它们会将流约简为可以在程序中使用的非流值。 count方法返回流中的数量，就是一种简单约简。

简单约简还有max和min，会返回最大值和最小值。返回的是一个类型Optional&lt; T &gt;的值，它要么在其中包装了答案，要么表示没有任何值。碰到返回null是很常见的，如果直接返回null会导致未做完备测试的程序产生空指针异常。Optional类型是一种更好的表示缺少返回值的方式。

```java
Optional<String> largest = words.stream().max(String::compareToIgnoreCase);
System.out.println(largest.orElse(""));
```

findFirst返回非空集合中的第一个值，通常与filter组合使用时显得很有用：

```java
Optional<String> startWithQ = words.stream().filter(s -> s.startsWith("Q")).findFirst();
```

如果不强调使用第一个匹配，而是用任意的匹配，可以使用findAny方法。 如果想知道是否存在匹配，那么可以使用anyMatch。这个方法可以接受一个断言引元，不需要使用filter。

```java
boolean aWordStartWithQ = words.stream().parallel().anyMatch(s -> s.startsWith("Q"));
```

还有allMatch和noneMatch方法，它们分别会在所有元素和没有任何元素匹配断言的情况下返回true。

## 7.Optional类型

Optional&lt; T &gt;对象是一种包装器对象，要么包装了类型T的对象，要么没有包装任何对象（要么引用某个对象，要么为null）。

### 如何使用Optional值

有效使用Optional的关键：它在值不存在的情况下会产生一个可替代物，而存在的情况下才会使用这个值。 默认值为空字符串：

```java
String result = optionalString.orElse("");
```

计算默认值：

```java
String result = optionalString.orElseGet(() -> Locale.getDefault().getDisplayName());
```

抛出异常：

```java
String result = optionalString.orElseThrow(IllegalAccessException::new);
```

上述是不存在任何值的情况下产生相应的替代物，另一条策略是只有存在的情况下才消费该值：

```java
optionalString.ifPresent(v -> results.add(v));
optionalString.ifPresent(results::add);
```

ifPresent不会返回任何值，如果想要得到处理结果：

```java
Optional<Boolean> added = optionalString.map(results::add);
```

added有三种值：true或false，以及在optionalString不存在的情况下的空Optional。

### 不适合使用Optional值的方式

get方法会在Optional值存在的情况下获得其中包装的元素，或者不存在时抛出一个NoSuchElementException对象，因此：

```java
Optional<T> optionalValue = ...;
optionalValue.get().method();
// Optional并不比下面方式安全
T value = ...;
value.method()

if (optionalValue.isPresent()) {
	optionalValue.get().method();
} 
// 并不比下面的方式更易处理
if (value != null) {
	value.method();
}
```

### 创建Optional值

```java
public static Optional<Double> inverse(Double x) {
    return x == 0 ? Optional.empty() : Optional.of(1 / x);
}
```

Optional.ofNullable(obj)会在obj不为nul的情况下返回Optional.of(obj)，否则返回Optional.empty()

### 用flatMap来构建Optional值的函数

可以产生Optional&lt; T &gt;对象的方法f，目标对象T具有一个可以产生Optional&lt; U &gt;对象的方法g。 s.f().g()这种组合没法工作，因为s.f()的类型为Optional&lt; T &gt;而不是T，因此需要：

```java
Optional<U> result = s.f().flatMap(T::g);
```

如果s.f()值存在，那么g就可以应用到上面，否则会返回一个空的Optional&lt; U &gt; 。

```java
public class OptionalTest {
    public static void main(String[] args) throws IOException {
        String contents = new String(Files.readAllBytes(Paths.get("alice30.txt")), StandardCharsets.UTF_8);
        List<String> wordList = Arrays.asList(contents.split("\\PL+"));
        Optional<String> optionalValue = wordList.stream().filter(s -> s.contains("fred")).findFirst();
        System.out.println(optionalValue.orElse("No word") + " contains fred");
        Optional<String> optionalString = Optional.empty();
        String result = optionalString.orElse("N/A");
        System.out.println("result: " + result);
        result = optionalString.orElseGet(() -> Locale.getDefault().getDisplayName());
        System.out.println("result: " + result);
        try {
            result = optionalString.orElseThrow(IllegalStateException::new);
            System.out.println("result: " + result);
        } catch (Throwable t) {
            t.printStackTrace();
        }
        optionalValue = wordList.stream().filter(s -> s.contains("red")).findFirst();
        optionalValue.ifPresent(s -> System.out.println(s + " contains red"));
        Set<String> results = new HashSet<>();
        optionalValue.ifPresent(results::add);
        Optional<Boolean> added = optionalValue.map(results::add);
        System.out.println(results);
        System.out.println(added);
        System.out.println(inverse(4.0).flatMap(OptionalTest::squareRoot));
        System.out.println(inverse(-1.0).flatMap(OptionalTest::squareRoot));
        System.out.println(inverse(0.0).flatMap(OptionalTest::squareRoot));
        Optional<Double> result2 = Optional.of(-4.0).flatMap(OptionalTest::inverse).flatMap(OptionalTest::squareRoot);
        System.out.println(result2);
    }
    public static Optional<Double> inverse(Double x) {
        return x == 0 ? Optional.empty() : Optional.of(1 / x);
    }
    public static Optional<Double> squareRoot(Double x) {
        return x < 0 ? Optional.empty() : Optional.of(Math.sqrt(x));
    }
}
```

## 8.收集结果

iterator会产生可以用来访问元素的旧式风格迭代器。 调用forEach方法，将某个函数应用于每个元素：

```java
stream.forEach(System.out::println);
```

并行流上，forEach会以任意顺序遍历各个元素。如果想按照流中顺序来处理，可以调用forEachOrdered方法，这个方法会丧失并行处理的优势。

toArray获得有流元素构成的数组，会返回一个Object[]，如果要具有正确的类型，可以将其传递给数组构造器：

```java
String[] result = stream.toArray(String[]::new);
```

collect，会接受一个Collector接口的实例。Collectors类提供了大量用于生产公共收集器的工厂方法。为了将流收集到列表或集中：

```java
List/Set<String> result = stream.collect(Collectors.toList()/toSet());
```

如果想要控制集的种类：

```java
TreeSet<String> result = stream.collect(Collectors.toCollection(TreeSet::new));
```

通过连接操作来收集流中所有字符串：

```java
String result = stream.collect(Collectors.joining(", "));
```

流中包含除字符串以外其他对象，需要将其转化为字符串：

```java
String result = stream.map(Object::toString).collect(Collectors.joining(", "));
```

将流的结果约间为总和、平均值、最大值或最小值，可以使用summarizing(Int|Long|Double)方法中的某一个。这些方法会接受一个将流对象映射为数据的函数，会产生（Int|Long|Double）SummaryStatistics的结果，同时计算总和、数量、平均值、最小值和最大值：

```java
IntSummaryStatistics summary = stream.collect(Collectors.summarizingInt(String::length));
double averageWordLength = summary.getAverage();
double maxWordLength = summary.getMax();
```

```java
public class CollectingResults {
    public static Stream<String> noVowels() throws IOException {
        String contents = new String(Files.readAllBytes(Paths.get("alice30.txt")), StandardCharsets.UTF_8);
        List<String> wordList = Arrays.asList(contents.split("\\PL+"));
        Stream<String> words = wordList.stream();
        return words.map(s -> s.replaceAll("[aeiouAEIOU]", ""));
    }
    public static <T> void show(String label, Set<T> set) {
        System.out.print(label + ": " + set.getClass().getName());
        System.out.println("["
            + set.stream().limit(10).map(Object::toString)
                .collect(Collectors.joining(", ")) + "]");
    }
    public static void main(String[] args) throws IOException {
        Iterator<Integer> iter = Stream.iterate(0, n -> n + 1).limit(10).iterator();
        while (iter.hasNext()) {
            System.out.println(iter.next());
        }
        Object[] numbers = Stream.iterate(0, n -> n + 1).limit(10).toArray();
        System.out.println("Object array:" + Arrays.toString(numbers));
        try {
            Integer number = (Integer) numbers[0];
            System.out.println("number:" + number);
            System.out.println("The following statement throws an exception:");
            Integer[] number2 = (Integer[]) numbers;
        } catch (ClassCastException ex) {
            System.out.println(ex);
        }
        Integer[] number3 = Stream.iterate(0, n-> n + 1).limit(10).toArray(Integer[]::new);
        System.out.println("Integer array:" + Arrays.toString(number3));
        Set<String> noVowelSet = noVowels().collect(Collectors.toSet());
        show("noVowelSet", noVowelSet);
        TreeSet<String> noVowelTreeSet = noVowels().collect(Collectors.toCollection(TreeSet::new));
        show("noVowelTreeSet", noVowelTreeSet);
        String result = noVowels().limit(10).collect(Collectors.joining());
        System.out.println("Joining: " + result);
        result = noVowels().limit(10).collect(Collectors.joining(", "));
        System.out.println("Joining with commas: " + result);
        IntSummaryStatistics summary = noVowels().collect(Collectors.summarizingInt(String::length));
        double averageWordLength = summary.getAverage();
        double maxWordLength = summary.getMax();
        System.out.println("Average word length: " + averageWordLength);
        System.out.println("Max word length: " + maxWordLength);
        System.out.println("forEach:");
        noVowels().limit(10).forEach(System.out::println);
    }
}
```

