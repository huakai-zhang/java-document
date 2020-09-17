# java.util.stream.Stream< T >
Stream< T > filter(Predicate< ? super T > p)
产生一个流，其中包含当前流中满足P的所有元素
long count()
产生当前流中元素的数量。这是一个终止操作。

static < T > Stream< T > of(T... values)
产生一个元素为给定值的流
static < T > Stream< T > empty()
产生一个不包含任何元素的流
static < T > Stream< T > generate(Supplier< T > s)
产生一个无限流，它的值是通过反复函数s而构建的
static < T > Stream< T > iterate(T seed, UnaryOperator< T > s)
产生一个无限流，它的元素包含种子、在种子上调用f产生的值、在前一个元素上调用f产生的值，等等。

< R > Stream< R > map(Function< ? supper T, ? extends R> mapper)
产生一个流，它包含将mapper应用于当前流中所有元素所产生的结果。
< R > Stream< R > flatMap(Function< ? supper T, ? extends R> mapper)
产生一个流，它包含将mapper应用于当前流中所有元素所产生的结果连接到一起而获得的（每个结果都是一个流）。

Stream< T > limit(long maxSize)
产生一个流，其中包含了当前流中最初的maxSize个元素
Stream< T > skip(long n)
产生一个流，它的元素是当前流中除了前n个元素之外的所有元素
static < T > Stream< T > concat(Stream< ? extends T > a, Stream< ? extends T > b)
产生一个流，它的元素是a的元素后面跟着b的元素

Stream< T > distinct()
产生一个流，包含当前流中所有不同的元素。
Stream< T > sorted()
Stream< T > sorted(Comparator< ? super T > comparator)
产生一个流，它的元素时当前流中的所有元素按照顺序排列的。第一个方法要求元素是实现了Comparable的类的实例
Stream< T > peek(Consumer< ? super T > action)
产生一个流，它与当前流中的元素相同，在获取其中每个元素时，会将其传递给action

Optional< T > max(Comparator< ? super T > comparator)
Optional< T > min(Comparator< ? super T > comparator)
分别产生这个流的最大元素和最小元素，使用有给定比较器定义的排序规则，如果流为空，会产生一个空的Optional对象，这些操作都是终结操作
Optional< T > findFirst()
Optional< T > findAny()
分别产生这个流的第一个和任意一个元素，如果流为空，会产生一个空的Optional对象，这些操作都是终结操作
boolean anyMatch(Predicate< ? super T > predicate)
boolean allMatch(Predicate< ? super T > predicate)
boolean noneMatch(Predicate< ? super T > predicate)
分别在这个流中任意元素、所有元素和没有任何元素匹配给定断言时返回true。这些操作都是终结操作

void forEach(Consumer< ? super T > action)
在流的每个元素上调用action，这是一个终结操作
Object[] toArray()
< A > A[] toArray(IntFuction< A[] > generator)
产生一个对象数组，或者在将引用A[]::new传递给构造器时，返回一个A类型的数组，这些操作都是终结操作。
< R ,A > R collect(Collector< ? super T,A,R > collector)
使用给定的收集器来收集当前流中的元素。Collectors类有用于多种收集器的工厂方法。

Optional< T > reduce(BinaryOperator< T > accumulator)
T reduce(T identity, BinaryOperator< T > accumulator)
< U > U reduce(T identity, BiFunction< U, ? super T, U > accumulator, BinaryOperator< U > combiner)
用给定的accmulator函数产生流中元素的累积总和。如果提供了幺元，那么第一个被累积的元素就是该幺元。如果提供了组合器，那么它可以用来分别累积的各个部分整合成总和。
< R > Rcollect(Supplier< R > supplier, BiConsumer< R, ? super T > accumulator, BiConsumer< R, R > combiner)
将元素收集到类型R的结果中。在每个部分上，都会调用supplier来提供初始结果，调用accmulator来交替地将元素添加到结果中，并调用combiner来整合两个结果。

# java.util.Stream.Collectors
static < T > Collector< T,?,List< T > > toList()
static < T > Collector< T,?,Set< T > > toSet()
产生一个将元素收集到列表或集中的收集器
static < T,C extends Collection< T > > Collector< T,?,C > toCollection(Supplier< C > collectionFactory)
产生一个将元素收集到任意集合中的收集器。可以传递一个诸如TreeSet:new的构造器引用
static Collector< CharSequence,?,String > joining()
static Collector< CharSequence,?,String > joining(CharSequence delimiter)
static Collector< CharSequence,?,String > joining(CharSequence delimiter, CharSequence prefix, CharSequence suffix)
产生一个链接字符串的收集器。分隔符会置于字符串之间，而第一个字符串之前可以有前缀，最后一个字符串之后可以有后缀，如果没有指定，那么它们都为空。
static < T > Collector< T,?,IntSummaryStatistics > summarizingInt(ToIntFunction< ? super T > mapper)
static < T > Collector< T,?,LongSummaryStatistics > summarizingLong(ToLongFunction< ? super T > mapper)
static < T > Collector< T,?,DoubleSummaryStatistics > summarizingDouble(ToDoubleFunction< ? super T > mapper)
产生能够生成（Int|Long|Double）SummaryStatistics对象的收集器，通过它可以获得将mapper应用于每个元素后所产生的结果的个数、总和、平均值、最大值和最小值

static< T,K,U > Collector< T,?,Map< K,U > > toMap(Function< ? super T, ? extends K > keyMapper, Function< ? super T, ? extends U > valueMapper)
static< T,K,U > Collector< T,?,Map< K,U > > toMap(Function< ? super T, ? extends K > keyMapper, Function< ? super T, ? extends U > valueMapper, BinaryOperator< U > mergeFunction)
static< T,K,U > Collector< T,?,Map< K,U > > toMap(Function< ? super T, ? extends K > keyMapper, Function< ? super T, ? extends U > valueMapper, BinaryOperator< U > mergeFunction, Supplier< M > mapSupplier)
static< T,K,U > Collector< T,?,Map< K,U > > toConcurrentMap(Function< ? super T, ? extends K > keyMapper, Function< ? super T, ? extends U > valueMapper)
static< T,K,U > Collector< T,?,Map< K,U > > toConcurrentMap(Function< ? super T, ? extends K > keyMapper, Function< ? super T, ? extends U > valueMapper, BinaryOperator< U > mergeFunction)
static< T,K,U > Collector< T,?,Map< K,U > > toConcurrentMap(Function< ? super T, ? extends K > keyMapper, Function< ? super T, ? extends U > valueMapper, BinaryOperator< U > mergeFunction, Supplier< M > mapSupplier)
产生一个收集器，它会产生一个映射表或并发映射表。keyMapper和valueMapper函数会应用于每个收集到的元素上，从而在所产生的映射表中生成一个键/值项。默认情况下，当两个元素产生相同的键时，会抛出一个IllegalStateException异常。可以提供一个mergeFunction来合并具有相同键的值。默认情况下，其结果是一个HashMap或ConcurrentHashMap。可以提供一个mapSupplier，它会产生所期望的映射表实例。

static< T,K > Collector< T,?,Map< K,List< T > > > groupingBy(Function< ? super T , ? extends K> classifier)
static< T,K > Collector< T,?,Map< K,List< T > > > groupingByConcurrent(Function< ? super T , ? extends K> classifier)
产生一个收集器，它会产生一个映射表或并发映射表，其键是将classifier应用于所有收集到的元素上所产生的结果，而值是有具有相同键的元素构成的一个个列表
static < T > Collector<T,?,Map< Boolean,List< T>>> partitioningBy(Predicate< ? super T > predicate)
产生一个收集器，它会产生一个映射表，其键是true/false，而值是由满足/不满足断言的元素构成的列表。

static < T > Collector< T,?,Long > counting()
产生一个可以对收集到的元素进行计数的收集器
static < T > Collector< T,?,Integer > () summingInt(ToIntFunction< ? super T > mapper)
static < T > Collector< T,?,Long > () summingLong(ToLongFunction< ? super T > mapper)
static < T > Collector< T,?,Double> () summingDouble(ToDoubleFunction< ? super T > mapper)
产生一个收集器，对将mapper应用到收集到的元素上之后产生的值计算总和
static < T > Collector< T,?,Optional< T > > maxBy(Comparator< ? super T > comparator)
static < T > Collector< T,?,Optional< T > > minBy(Comparator< ? super T > comparator)
产生一个收集器，使用comparator指定的排序方法，计算收集到的元素中的最大值和最小值
static < T,U,A,R > Collector< T,?,R > mapping(Function< ? super T,? extends U > mapper, Collector< ? super U,A,R > downstream)
产生一个收集器，它会产生一个映射表，其键是将mapper应用到收集到的数据上面产生的，其值是使用downstream收集器收集到的具有相同键的元素

# java.util.stream.IntStream
static IntStream range(int startInclusive, int endExclusive)
static IntStream rangeClosed(int startInclusive, int endExclusive)
产生一个由给定范围内的整数构成的IntStream
static IntStream of(int... values)
产生一个由给定元素构成的IntStream
int[] toArray()
产生一个由当前流中的元素构成的数组
int sum()
OptionalDouble average()
OptionalInt max()
OptionalInt min()
IntSummaryStatistics summaryStatistics()
产生当前流中的元素的总和、平均值、最大值和最小值，或者从中可以获得这些结果的所有四种值的对象
Stream< Integer > boxed()
产生用于当前流中的元素的包装器对象流

# java.util.stream.LongStream
static LongStreamrange(int startInclusive, int endExclusive)
static LongStreamrangeClosed(int startInclusive, int endExclusive)
产生一个由给定范围内的整数构成的LongStream
static LongStreamof(long... values)
产生一个由给定元素构成的LongStream
long[] toArray()
产生一个由当前流中的元素构成的数组
long sum()
OptionalDouble average()
OptionalLong max()
OptionalLong min()
LongSummaryStatistics summaryStatistics()
产生当前流中的元素的总和、平均值、最大值和最小值，或者从中可以获得这些结果的所有四种值的对象
Stream< Long > boxed()
产生用于当前流中的元素的包装器对象流

# java.util.stream.DoubleStream
static DoubleStream(double... values)
产生一个由给定元素构成的DoubleStream
double[] toArray()
产生一个由当前流中的元素构成的数组
double sum()
OptionalDouble average()
OptionalDouble max()
OptionalDouble min()
DoubleSummaryStatistics summaryStatistics()
产生当前流中的元素的总和、平均值、最大值和最小值，或者从中可以获得这些结果的所有四种值的对象
Stream< Double > boxed()
产生用于当前流中的元素的包装器对象流

# java.util.CharSequence
IntStream codePoints()
产生由当前字符串的所有Unicode码点构成的流

# java.util.Random
IntStream ints()
IntStream ints(int randomNumberOrigin, int randomNumberBound)
IntStream ints(long streamSize)
IntStream ints(long streamSize, int randomNumberOrigin, int randomNumberBound)
LongStream ints()
LongStream ints(long randomNumberOrigin, long randomNumberBound)
LongStream ints(long streamSize)
LongStream ints(long streamSize, long randomNumberOrigin, long randomNumberBound)
DoubleStream ints()
DoubleStream ints(double randomNumberOrigin, double randomNumberBound)
DoubleStream ints(long streamSize)
DoubleStream ints(long streamSize, double randomNumberOrigin, double randomNumberBound)
产生随机数流。如果提供了streamSize，这个流就是具有给定数量元素的有限流。当提供了边界时，其元素将位于randomNumberOrigin（包含）和randomNumberBound（不包含）的区间内。

# java.util.stream.BaseStream
Iterator< T > iterator()
产生一个用于获取流中各个元素的迭代器，这是一个终结操作

# java.util.regex.Pattern
Stream< String > splitAsStream(CharSequence input)
产生一个流，它的元素是输入中由该模式界定的部分。

# java.nio.file.Files
static Stream< String > lines(Path path)
static Stream< String > lines(Path path, Charset cs)
产生一个流，它的元素是指定文件中的行，该文件的字符集为UTF-8或为指定的字符集

# java.util.function.Supplier< T >
提供一个值

# java.util.Optional
T orElse(T other)
产生这个Optional的值，或者在该Optional为空时，产生other
T orElseGet(Supplier< ? extends T > other)
产生这个Optional的值，或者在该Optional为空时，产生调用other的结果
< X extends Throwable > T orElseThrow(Supplier< ? extends X > exceptionSupplier)
产生这个Optional的值，或者在该Optional为空时，抛出调用exceptionSupplier的结果
void ifPresent(Consumer< ? super T > consumer)
如果该Optional不为空，那么就将它的值传递给consumer
< U > Optional< U > map(Function< ? super T, ? extends U > mapper)
产生将该Optional的值传递给mapper后的结果，只要这个Optional不为空且结果不为null，否则产生一个空Optional

T get()
产生这个Optional值，或者在该Optional为空时，抛出一个NoSuchELementException对象
boolean isPresent()
如果该Optional不为空，则返回true

static < T > Optional< T > of(T value)
static < T > Optional< T > ofNullable(T value)
产生一个具有给定值的Optional。如果value为空，那么第一个方法会抛出一个NullPointerException对象，而第二个方法将产生一个空Optional
static < T > Optional< T > empty()
产生一个空Optional

< U > Optional< U > flatMap(Function< ? super T, Optional< U > > mapper)
产生将mapper应用于当前的Optional值所产生的结果，或者在当前Optional为空时返回一个空Optional

# IntSummaryStatistics、LongSummaryStatistics、DoubleSummaryStatistics
long getCount()
产生汇总后的元素的个数
(int|long|double) getSum()
double getAverage()
产生汇总后的元素的总和或平均值，或者在没有任何元素时返回0
(int|long|double) getMax()
(int|long|double) getMin()
产生汇总后的元素的最大值和最小值，或者在没有任何元素时，产生（Int|Long|Double）.(MAX|MIN)_VALUE

# java.util.Optional(Int|Long|Double)
static Optional(Int|Long|Double) of ((int|long|double) value)
用所提供的基本类型值产生一个可选对象
(int|long|double) getAs(Int|Long|Double)()
产生当前可选对象的值，或者在其为空时抛出一个NoSuchElementException异常
(int|long|double) orElse((int|long|double) other)
(int|long|double) orElseGet((Int|Long|Double)Supplier other)
产生当前可选对象的值，或者在这个对象为空时产生可替代的值
void ifPresent((Int|Long|Double)Consumer consumer)
如果当前可选对象为空，则将其值传递给consumer

# java.util.stream.BaseStream< T, S extends BaseStream< T,S > >
S parallel()
产生一个与当前流中元素相同的并行流
S unordered()
产生一个与当前流中元素相同的无序流

# java.util.Collection< E >
Stream< E > parallelStream()
用当前集合中的元素产生一个并行流
