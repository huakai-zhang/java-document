•ArrayList<E> ( )
构造一个空数组列表。
•ArrayList<E> ( int initialCapacity)
用指定容量构造一个空数组列表。
参数：initalCapacity 数组列表的最初容量
•boolean add( E obj )
在数组列表的尾端添加一个元素。 永远返回 true。
参数：obj 添加的元素
•int size( )
返回存储在数组列表中的当前元素数量。（这个值将小于或等于数组列表的容量。)
•void ensureCapacity( int capacity)
确保数组列表在不重新分配存储空间的情况下就能够保存给定数量的元素。
参数：capacity 需要的存储容量
•void trimToSize( )
将数组列表的存储容量削减到当前尺寸。
• void set(int index，E obj)
设置数组列表指定位置的元素值， 这个操作将覆盖这个位置的原有内容。
参数： index 位置（必须介于 0 ~ size()-1 之间）
obj 新的值
• E get(int index)
获得指定位置的元素值。
参数：index 获得的元素位置（必须介于 0 ~ size()-l 之间）
• void add(int index,E obj)
向后移动元素，以便插入元素。
参数：index 插入位置（必须介于 0 〜 size()-l 之间）
obj 新元素
• E remove (int index)
删除一个元素，并将后面的元素向前移动。被删除的元素由返回值返回。
参数：index 被删除的元素位置（必须介于 0 〜 size()-1之间）
