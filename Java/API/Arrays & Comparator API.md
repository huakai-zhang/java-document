•static String toString(type[] a) 
返回包含 a 中数据元素的字符串， 这些数据元素被放在括号内， 并用逗号分隔。
参数： a 类型为 int、long、short、 char、 byte、boolean、float 或 double 的数组。
• static type copyOf(type[]a, int length)
• static type copyOfRange(type[]a , int start, int end)
返回与 a 类型相同的一个数组， 其长度为 length 或者 end-start， 数组元素为 a 的值。
参数：a 类型为 int、 long、 short、 char、 byte、boolean、 float 或 double 的数组。
start 起始下标（包含这个值）0
end 终止下标（不包含这个值）。这个值可能大于 a.length。 在这种情况下，结果为 0 或 false。
length 拷贝的数据元素长度c 如果 length 值大于 a.length， 结果为 0 或 false ;否则， 数组中只有前面 length 个数据元素的拷贝值。
•static void sort(Object [] a)
采用优化的快速排序算法对数组进行排序。
参数：a 类型为 int、long、short、char、byte、boolean、float 或 double 的数组。使用 mergesort 算法对数组 a 中的元素进行排序。要求数组中的元素必须属于实现了Comparable 接口的类， 并且元素之间必须是可比较的。
•static int binarySearch(type[]a , type v)
• static int binarySearch(type[]a, int start, int end, type v) 
采用二分搜索算法查找值 v。如果查找成功， 则返回相应的下标值； 否则， 返回一个负数值 。r -r-1 是为保持 a 有序 v 应插入的位置。
参数：a 类型为 int、 long、 short、 char、 byte、 boolean 、 float 或 double 的有序数组。
start 起始下标（包含这个值）。
end 终止下标（不包含这个值。)
v 同 a 的数据元素类型相同的值。
• static void fill(type[]a , type v)
将数组的所有数据元素值设置为 V。
参数：a 类型为 int、 long、short、 char、byte、boolean、float 或 double 的数组。
v 与 a 数据元素类型相同的一个值。
• static boolean equals(type[]a, type[]b)
如果两个数组大小相同， 并且下标相同的元素都对应相等， 返回 true。
参数：a、 b 类型可以是 Object、 int、 long、 short、 char、 byte、 boolean、 float 或 double。
•static int hashCode(type[] a )
计算数组 a 的散列码。组成这个数组的元素类型可以是 object，int，long, short, char,byte, boolean, float 或 double。

• static < E> List< E> asList(E... array)
返回一个数组元素的列表视图。这个数组是可修改的， 但其大小不可变。

• static < T > Stream< T > stream(T[] array, int startInclusive, int endExclusive)
产生一个流它的元素有数组中指定范围内的元素构成的。

 # java.lang.Comparable< T> 
• int compareTo(T other)
用这个对象与 other 进行比较。如果这个对象小于 other 则返回负值； 如果相等则返回0；否则返回正值。
static < T extends Comparable< ? super T > > Comparator< T > reverseOrder()
生成一个比较器，将逆置Comparable接口提供的顺序。
default Comparator< T > reversed()
生成一个比较器，将逆置这个比较器提供的顺序。

