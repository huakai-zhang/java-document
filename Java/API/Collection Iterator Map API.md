# java.util.Collection< E >
Iterator< E > iterator()
返回一个用于访问集合中每个元素的迭代器
int size()
返回当前存储在集合中的元素个数
boolean isEmpty()
如果集合中没有元素，返回true
boolean contains(Object obj)
如果集合中包括了一个与obj相等对象，返回true
boolean containsAll(Collection< ? > other)
如果这个集合包含other集合中的所有元素，返回true
boolean add(Object element)
将一个元素添加到集合中。如果由于这个调用改变了集合，返回true
boolean addAll(Collection< ? extends E > other)
将 other 集合中的所有元素添加到这个集合。如果由于这个调用改变了集合，返回 true。
boolean remove(Object obj)
从这个集合中删除等于obj 的对象。如果有匹配的对象被删除， 返回 true。
boolean removeAll(Collection< ? > other)
从这个集合中删除 other 集合中存在的所有元素。如果由于这个调用改变了集合，返回 true。
default boolean removelf(Predicate< ? super E > filter)
从这个集合删除 filter 返回 true 的所有元素。如果由于这个调用改变了集合， 则返回 true。
void clear()
从这个集合中删除所有的元素。
boolean retainAll(Collection< ? > other)
从这个集合中删除所有与 other 集合中的元素不同的元素。如果由于这个调用改变了集合， 返回 true。
Object[] toArray()
返回这个集合的对象数组。
< T > T[] toArray(T[] arrayToFill)
返回这个集合的对象数组。如果 arrayToFill 足够大， 就将集合中的元素填入这个数组中。剩余空间填补 null ; 否则， 分配一个新数组， 其成员类型与 arrayToFill 的成员类型相同， 其长度等于集合的大小， 并填充集合元素。
default boolean removeIf(Predicate< ? super E > filter)
删除所有匹配元素。

dafault Stream< E > stream()
default Stream< E > parallelStream()
产生当前集合中所有元素的顺序流或并行流

# java.util.Iterator< E >
boolean hasNext()
如果存在可以访问的元素，返回true
E next()
返回将要访问的下一个对象。如果已经到达了集合的尾部，将抛出一个NoSuchElementException
void remove()
删除上次访问的对象。这个方法必须紧跟在访问一个元素之后执行。如果上次访问之后，集合已经发生了变化，这个方法将抛出一个IllegalStateException

# java.util.List< E >
ListIterator< E > listIterator()
返回一个列表迭代器，以便用来访问列表中的元素
ListIterator< E > listIterator(int index)
返回一个列表迭代器，以便用来访问列表中的元素，这个元素是第一次调用next返回的给定索引的元素
void add(int i, E element)
在给定位置添加一个元素
void addAll(int i, Collection< ? extends E > elements)
将某个集合中的所有元素添加到给定位置
E remove(int i)
删除给定位置的元素并返回这个元素
E get(int i )
获取给定位置的元素
E set(int i)
用新元素取代给定位置的元素，并返回原来那个元素
int indexOf(Object element)
返回与指定元素相等的元素在列表中第一次出现的位置，如果没有这样的元素将返回-1
int lastIndexOf(Object element)
返回与指定元素相等的元素在列表中最后一次出现的位置，如果没有这样的元素将返回-1
List< E> subList(int firstlncluded，int firstExcluded)
返回给定位置范围内的所有元素的列表视图。
default void sort(Comparator< ? super T > comparator)
使用给定比较器对列表排序。
default void replaceAll(UnaryOperator< E > op)
对这个列表的所有元素应用这个操作。

# java.util.ListIterator< E >
void add(E newElement)
在当前位置前添加一个元素
void set(E newElement)
用新元素替代next或previous上次访问的元素。如果在next或previous上次调用之后列表结构被修改了，将抛出一个IllegaStateException
boolean hasPrevious()
当反向迭代列表时，还有可供访问的元素，返回true
E previous()
返回下一次调用next方法时将返回的元素索引
int previousIndex()
返回下一次调用previous方法时将返回的元素索引

# java.util.LinkedList< E >
LinkedList()
构造一个空链表
LinkedList(Collection< ? extends E > elements)
构造一个链表，并将集合中所有的元素添加到这个链表中
void addFirst(E element)
void addLast(E element)
将某个元素添加到列表的头部或尾部
E getFirst()
E getLast()
返回列表头部或尾部的元素
E removeFirst()
E removeLast()
删除并返回列表头部或尾部的元素

# java.util.HashSet< E >
HashSet()
构造一个空散列表
HashSet(Collection< ? extends E > elements)
构造一个散列集，并将集合中的所有元素添加到这个散列集中
HashSet(int initialCapacity, float loadFactor)
构造一个具有指定容量和装填因子（0.0~1.0之间）的空散列集

# java.util.TreeSet< E >
TreeSet()
TreeSet(Comparator< ? super E > comparator)
构造一个空树集
TreeSet(Collection< ? extends E > elements)
TreeSet(SortedSet< E > s)
构造一个树集，并增加一个集合或有序集中的所有元素（对于后一种情况，要使用相同的顺序）

# java.util.SortedSet< E >
Comparator< ? super E > comparator()
返回用于对元素进行排序的比较器。如果元素用Comparable接口的compareTo方法进行比较则返回null
E first()
E last()
返回有序集中的最小元素或最大元素
SortedSet< E> subSet(E firstlncluded, E firstExcluded)
SortedSet< E> headSet(E firstExcluded)
SortedSet< E> tailSet(E firstlncluded)
返回给定范围内的元素视图。
# java.util.NavigableSet< E >
E higher(E value)
E lower(E value)
返回大于value的最小元素或小于value的最大元素，如果没有这样的元素就返回null
E ceiling(E value)
E floor(E value)
返回大于等于value的最小元素或小于等于value的最大元素，如果没有这样的元素就返回null
E pollFirst()
E pollLast()
删除并返回这个集中的最大元素或最小元素，这个集为空时返回null
Iterator< E > descendingIterator()
返回一个按照递减顺序遍历集中元素的迭代器
NavigableSet< E> subSet(E from , boolean fromlncluded, E to, boolean tolncluded)
NavigableSet< E> headSet(E to, boolean tolncluded)
NavigableSet< E> tailSet(E from, boolean fromlncluded)
返回给定范围内的元素视图。boolean 标志决定视图是否包含边界。

# java.util.Queue< E >
boolean add(E element)
boolean offer(E element)
如果队列没有满，将给定的元素添加到这个双端队列的尾部并返回true。如果队列满了（==存在一些固定长度的队列类==），第一个方法将抛出一个IllegalStateException，而第二个方法返回false。
E remove（）
E poll()
如果队列不空，删除并返回这个队列头部元素。如果队列为空，第一个方法抛出NoSuchElementException，而第二个方法返回null。
E element()
E peek()
如果队列不空，返回这个队列头部的元素，但不删除。如果队列为空，第一个方法抛出NoSuchElementException，而第二个方法返回null。

# java.util.Deque< E >
void addFirst(E element)
void addLast(E element)
boolean offerFirst(E element)
boolean offerLast(E element)
将给定的对象添加到双端队列的头部或尾部。如果队列满了，前两个方法将抛出一个IllegaStateException，而后两个方法返回false。
E removeFirst()
E removeLast()
E pollFirst()
E pollLast()
如果队列不空，删除并返回队列的头部或尾部元素。如果队列为空，前两个方法将抛出一个NoSuchElementException，而后两个方法返回null。
E getFirst()
E getLast()
E peekFIrst()
E peekLast()
如果队列不空，返回这个队列头部或尾部的元素，但不删除。如果队列为空，前两个方法抛出NoSuchElementException，而后两个方法返回null。

# java.util.ArrayDeque< E >
ArrayDeque()
ArrayDeque(int initialCapacity)
用初始化容量16或给定的原始容量构造一个无限双端队列

# java.util.PriorityQueue< E >
PriorityQueue()
PriorityQueue(int initialCapacity)
PriorityQueue(int initialCapacity, Comparator< ? super E > c)
构造一个优先级队列，并用知道的比较器对元素进行排序。

# java.util.Map< K, V >
• V get(Object key)
获取与键对应的值；返回与键对应的对象， 如果在映射中没有这个对象则返回 null。键可以为 null。
• default V getOrDefault(Object key, V defaultValue)
获得与键关联的值；返回与键关联的对象， 或者如果未在映射中找到这个键， 则返回defaultValue。
• V put(K key, V value)
将键与对应的值关系插入到映射中。如果这个键已经存在， 新的对象将取代与这个键对应的旧对象。这个方法将返回键对应的旧值。如果这个键以前没有出现过则返回null。键可以为 null， 但值不能为 null。
• void putAl 1(Map< ? extends K , ? extends V > entries)
将给定映射中的所有条目添加到这个映射中。
• boolean containsKey(Object key)
如果在映射中已经有这个键， 返回 true。
• boolean containsValue(Object value)
如果映射中已经有这个值， 返回 true。
•default void forEach(BiConsumer<  ? super K ,? super V  > action)
对这个映射中的所有键 / 值对应用这个动作。
•default V merge(K key, V value, BiFunction<? super V ,? super V ,?
extends V> remappingFunctlon)
如果 key 与一个非 null 值 v 关联， 将函数应用到 v 和 value, 将 key 与结果关联， 或者如果结果为 null, 则删除这个键。否则， 将 key 与 value 关联， 返回 get(key)
• default V compute(K key, BiFunction< ? super K,? super V ,? extends V > remappingFunction)
将函数应用到 key 和 get(key。) 将 key 与结果关联， 或者如果结果为 mill， 则删除这个键。返回 get(key)
•default V computeIfPresent(K key , BiFunction< ? super K , ? super V , ? extends V > remappingFunction ) 
如果 key 与一个非 null 值 v 关联，将函数应用到 key 和 v， 将 key 与结果关联， 或者如果结果为 null, 则删除这个键。返回 get(key)
•default V computeIfAbsent(K key , Function< ? super K , ? extends V > mappingFunction ) 
将函数应用到 key, 除非 key 与一个非 mill 值关联。将 key 与结果关联， 或者如果结果为 null, 则删除这个键。返回 get(key)
•default void repl aceAl 1 (BiFunction< ? super K ,? super V , ? extends V > function) 
在所有映射项上应用函数。将键与非 mill 结果关联， 对于 null 结果， 则将相应的键删除。

•Set< Map.Entry< K, V > > entrySet()
返回 Map.Entry 对象（映射中的键 / 值对）的一个集视图。可以从这个集中删除元素，它们将从映射中删除，但是不能增加任何元素。
•Set< K > keySet()
返回映射中所有键的一个集视图。可以从这个集中删除元素，键和相关联的值将从映射中删除， 但是不能增加任何元素。
• Collection< V > values()
返回映射中所有值的一个集合视图。可以从这个集合中删除元素， 所删除的值及相应的键将从映射中删除， 不过不能增加任何元素。

# java.utii.HashMap< K,V >
•HashMap()
•HashMap(int initialCapacity)
•HashMap(int initialCapacity, float loadFactor)
用给定的容量和装填因子构造一个空散列映射（装填因子是一个 0.0 〜 1.0 之间的数值。这个数值决定散列表填充的百分比。一旦到了这个比例， 就要将其再散列到更大的表中）。默认的装填因子是 0.75。

# java.util.TreeMap< K,V >
•TreeMap()
为实现 Comparable 接口的键构造一个空的树映射。
•TreeMap(Comparator< ? super K > c)
构造一个树映射， 并使用一个指定的比较器对键进行排序。
•TreeMap(Map< ? extends K, ? extends V > entries)
构造一个树映射， 并将某个映射中的所有条目添加到树映射中。
•TreeMap(SortedMap< ? extends K, ? extends V > entries)
构造一个树映射， 将某个有序映射中的所有条目添加到树映射中， 并使用与给定的有序映射相同的比较器。

# java.util.SortedMap< K, V >
•Comparator< ? super K > comparator()
返回对键进行排序的比较器。如果键是用 Comparable 接口的 compareTo 方法进行比较的，返回 null。
•K firstKey()
•K lastKey()
返回映射中最小元素和最大元素。
•SortedMap<K , V> subMap(K firstlncluded , K firstExcluded)
•SortedMap<K , V > headMap(K firstExcluded)
•SortedMap<K , V> tailMap(K firstlncluded)
返回在给定范围内的键条目的映射视图。

# java.util.Map.Entry< k, v >
K getKey()
V getValue()
返回这一条目的键或值
V setValue(V newValue)
将相关映射中的值改为心智，并返回原来的值

# java.util.WeakHashMap< K, V >
• WeakHashMap()
• WeakHashMapCint initialCapacity)
• WeakHashMap(int initialCapacity, float loadFactor)
用给定的容量和填充因子构造一个空的散列映射表。

# java.util.LinkedHashSet< E >
• LinkedHashSet()
• LinkedHashSet(int initialCapacity)
• LinkedHashSet(int initialCapacity, float loadFactor)
用给定的容量和填充因子构造一个空链接散列集。

# java.utif.LinkedHashMap< K, V > 
• LinkedHashMap()
• LinkedHashMap(int initialCapacity)
• LinkedHashMap(int initialCapacity, float loadFactor)
• LinkedHashMap(int initialCapacity, float loadFactor, booleanaccessOrder)
用给定的容量、 填充因子和顺序构造一个空的链接散列映射表。accessOrder 参数为true 时表示访问顺序， 为 false 时表示插入顺序。
• protected boolean removeEldestEntry(Map.Entry< K, V > eldest)
如果想删除 eldest 元素， 并同时返回 true, 就应该覆盖这个方法。eldest 参数是预期要删除的条目。这个方法将在条目添加到映射中之后调用。其默认的实现将返回 false。即在默认情况下，旧元素没有被删除。然而，可以重新定义这个方法， 以便有选择地返回 true。例如， 如果最旧的条目符合一个条件， 或者映射超过了一定大小，则返回true。

# java.util.EnumSet<E extends Enum< E >
• static < E extends Enum< E» EnumSet< E> allOf(Class< E > enumType)
返回一个包含给定枚举类型的所有值的集。
• static < E extends Enum< E» EnumSet< E > noneOf(Class< E > enumType)
返回一个空集，并有足够的空间保存给定的枚举类型所有的值。
• static < E extends Enum< E >> EnumSet< E > range(E from, E to)
返回一个包含 from 〜 to 之间的所有值（包括两个边界元素）的集。
• static < E extends Enum< E» EnumSet< E > of(E value)
• static < E extends Enum< E» EnumSet< E > of(E value, E... values)
返回包括给定值的集。

# java.util.EnumMap<K extends Enum<K>, V> 
• EnumMap(Class< K > keyType)
构造一个键为给定类型的空映射。

# java.util.ldentityHashMap< K >
• IdentityHashMap()
• IdentityHashMap(int expectedMaxSize)
构造一个空的标识散列映射集，其容量是大于 1.5 * expectedMaxSize 的 2 的最小次幂( expectedMaxSize 的默认值是 21 )。

# java.lang.System
• static int identityHashCode(Object obj) 
返回 ObjecUiashCode 计算出来的相同散列码（根据对象的内存地址产生，) 即使obj所属的类已经重新定义了 hashCode 方法也是如此。

# java.util.Collections
• static < E> Collection unmodifiableCollectlon(Collection< E> c )
•static < E> List unmodifiableLIst(L1st< E> c )
•static < E> Set unmodifiab1eSet(Set< E> c )
•static < E> SortedSet unmodif1ableSortedSet(SortedSet< E> c)
•static < E> SortedSet unmodif1ableNavigab1eSet(Navigab1eSet< E> c ) 
•static <K , V> Map unmodifiableMap(Map<K , V> c )
•static <K , V> SortedMap unmodifiableSortedMap(SortedMap<K , V> c)
•static <K , V> SortedMap unmodifiableNavigab eMap(NavigableMap<K , V> c) 
构造一个集合视图；视图的更改器方法抛出一个 UnsupportedOperationException。

•static < E> Collection< E> synchronizedCollection(Collection< E> c )
•static < E> List synchronizedList(List< E> c )
•static < E> Set synchronizedSet(Set< E> c )
•static < E> SortedSet synchronizedSortedSet(SortedSet< E> c )
•static < E> NavigabeSet synchronizedNavigabeSet(NavigableSet< E> c) 
•static <K , V > Map<K, V > synchronizedMap(Map<K , V > c )
•static <K , V > SortedMap<K, V> synchronizedSortedMap(SortedMap<K, V> c )
•static <K , V > NavigableMap<K , V > synchronizedNavigab1eMap(NavigableMap<K, V > c ) 
构造一个集合视图；视图的方法同步。

•static < E> Collection checkedCollection(Collection< E> c, Class< E> elementType)
•static < E> List checkedList(List< E> c , Class< E> elementType)
•static < E> Set checkedSet(Set< E> c, Class< E> elementType)
•static < E> SortedSet checkedSortedSet(SortedSet< E> c , Class< E> elementType)
•static < E> NavigableSet checkedNavigableSet(NavigableSet< E> c ,Class< E> elementType) 
•static <K , V > Map checkedMap(Map<K , V > c , Class< K > keyType,Class< V > valueType)
•static <K , V > SortedMap checkedSortedMap( SortedMap<K , V > c ,Class< K> keyType, Class< V> valueType)
•static <K , V > NavigableMap checkedNavigableMap(NavigableMap<K , V > c, Class< K> keyType, Cl ass< V > valueType) 
•static < E > Queue< E > checkedQueue ( Queue< E > queue , Class < E > elementType) 
构造一合视图；如果插入一个错误类型的元素，视图的方法拋出 ClassCastException。

•static < E> List< E> nCopies(int n, E value)
•static < E> Set< E> singleton(E value)
•static < E> List< E> singletonList(E value)
•static <K , V> Map<K, V> singletonMap(K key, V value)
构造一个对象视图， 可以是一个包含 n 个相同元素的不可修改列表， 也可以是一个单元素集、 列表或映射。

•static < E> List< E> emptyList( )
•static < T> Set< T> emptySet( )
•static < E> SortedSet< E> emptySortedSet( )
•static NavigableSet< E> emptyNavigableSet()
•static <K ,V> Map<K ,V> emptyMap()
•static <K ,V> SortedMap<K,V > emptySortedMap()
•static <K ,V> NavigableMap<K,V > emptyNavigableMap()
•static < T> Enumeration< T> emptyEnumeration()
•static < T> Iterator< T> emptyIterator()
•static < T> ListIterator< T> emptyListIterator()
生成一个空集合、映射或迭代器。

static < T extends Comparable< ? super T>> void sort(List< T > elements)
使用稳定的排序算法，对列表中的列表进行排序。这个算法的时间复杂度是O(n logn)，其中n为列表的长度。
static void shuffle(List< ? > elements)
static void shuffle(List< ? > elements, Random r)
随机地打乱列表中的元素。这个算法的时间复杂度是O(n a(n))，n是列表的长度，a(n)是访问元素的平均时间。

static < T extends Comparable<? super T > > int binarySearch(List< T > elements, T key)
static < T > int binarySearch(List< T > elements, T key, Conparator< ? super T> c)
从有序列表中搜索一个键， 如果元素扩展了 AbstractSequentialList 类， 则采用线性查找，否则将采用二分查找。这个方法的时间复杂度为 0 (a(n) log n), n 是列表的长度，a(n) 是访问一个元素的平均时间。这个方法将返回这个键在列表中的索引，如果在列表中不存在这个键将返回负值i 。在这种情况下，应该将这个键插人到列表索引-i-1的位置上，以保持列表的有序性。

static < T extends Comparab1e< ? super T>> T min(Collection< T> elements )
static <T extends Comparable<? super T>> T max(Col 1ection< T> elements )
static < T> min(Collection< T> elements, Comparator ? super T> c )
static < T> max (Collection< T> elements, Comparator ? super T> c )
返回集合中最小的或最大的元素（为清楚起见， 参数的边界被简化了）。
static < T> void copy(List<? super T> to, List< T> from)
将原列表中的所有元素复制到目辱列表的相应位1上。目标列表的长度至少与原列表一样。
static < T> void fill(List<? super T> l， T value)
将列表中所有位置设置为相同的值。
static < T> boolean addAll(Collection<? super T> c, T . .. values ) 
将所有的值添加到集合中。如果集合改变了， 则返回 true。
static < T> boolean replaceAll(List< T> l, T oldValue, T newValue) 
用 newValue 取代所有值为 oldValue 的元素。
static int indexOfSubList(List<?> l , List<?> s ) 
static int lastlndexOfSubList(List<?> l , List<?> s ) 
返回 1 中第一个或最后一个等于s子列表的索引。如果l中不存在等于s的子列表， 则返回 -1。例如，1 为[s,t a，r] , s 为[t, a, r], 两个方法都将返回索引 1。
static void swap(List<?> l, int i, int j) 
交换给定偏移量的两个元素。
static void reverse(List<?> l)
逆置列表中元素的顺序。例如， 逆置列表 [t a, r] 后将得到列表 [r, a, t。] 这个方法的时间复杂度为 O (n) ，n为列表的长度。
static void rotate(List<?> l, int d) 
旋转列表中的元素， 将索引 i 的条目移动到位置（i + d) % l.size() 。例如， 将列表 [t ，a, r] 旋转移 2 个位置后得到 [a,，r ，t]。 这个方法的时间复杂度为 O(n), n 为列表的长度。
static int frequency(Collection<?> c, Object o) 
返回 c 中与对象 o 相同的元素个数。
boolean disjoint(Collection<?> cl, Collection<?> c2 ) 
如果两个集合没有共同的元素， 则返回 true。

# java.util.NavigableMap<K, V>
• NavigableMap<K , V > subMap(K from, boolean fromlncluded , K to,boolean tolncluded)
• NavigableMap< K, V > headMap(K from, boolean fromlncluded)
• NavigableMap<K, V > tailMap(K to, boolean tolncluded)
返回在给定范围内的键条目的映射视图。boolean 标志决定视图是否包含边界。

# java.util.Enumeration< E >
boolean hasMoreElements()
如果还有更多的元素可以查看，则返回true
E nextElement()
返回被检测的下一个元素。如果hasMoreElements()返回false，则不要调用这个方法。

# java.util.Hashtable< K, V >
Enumeration< K > keys()
返回一个遍历散列表中键的枚举对象。
Enumeration< V > elements()
返回一个遍历散列表中元素的枚举对象。

# java.util.Vector< E >
返回遍历向量中元素的枚举对象。

# java.util.Properties
Properties()
创建一个空的属性映射。
Properties(Properties defaults)
创建一个带有一组默认值的空的属性映射。
String getProperty(String key)
获得属性的对应关系；返回与键对应的字符串。如果在映射中不存在，返回默认表中与这个键对应的字符串。
String getProperty(String key, String defaultValue)
获得在键没有找到时具有的默认值属性；它将返回与键对应的字符串，如果在映射中不存在，就返回默认的字符串。
void load(InputStream in)
从InputStream加载属性映射。
void store(OutputStream out, String commentString)
把属性映射存储到OutputStream。

# java.util.Stack< E >
E push(E item)
将item压入栈并返回item。
E pop()
弹出并返回栈顶的item。如果栈为空，请不要调用这个方法。
E peek()
返回栈顶元素，但不弹出。如果栈为空，请不要调用这个方法。

# java.util.BitSet
BitSet(int initialCapcity)
创建一个位集。
int length()
返回位集的“逻辑长度”，即1加上位集的最高设置位的索引。
boolean get(int bit)
获得一个位。
void set(int bit)
设置一个位。
void clear(int bit)
清除一个位。
void and(BitSet set)
这个位集与另一个位集进行逻辑 “AND”。
void or(BitSet set)
这个位集与另一个位集进行逻辑 “OR”。
void xor(BitSet set)
这个位集与另一个位集进行逻辑 “XOR”。
void andNot(BitSet set)
清除这个位集中对应另一个位集中设置的所有位。

