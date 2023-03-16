# String API
• char charAt (int index)
返回给定位置的代码单元。除非对底层的代码单元感兴趣， 否则不需要调用这个方法。
• int codePointAt(int Index)
返回从给定位置开始的码点。
• int offsetByCodePoints(int startlndex, int cpCount)
返回从 startlndex 代码点开始，位移 cpCount 后的码点索引。
• in t compareTo(String other)
按照字典顺序，如果字符串位于 other 之前， 返回一个负数；如果字符串位于 other 之后，返回一个正数；如果两个字符串相等，返回 0。
• IntStream codePoints()
将这个字符串的码点作为一个流返回。调用 toArray 将它们放在一个数组中。
• new String(int[] codePoints, int offset, int count)
用数组中从 offset 开始的 count 个码点构造一个字符串。
• boolean equals(0bject other)
如果字符串与 other 相等， 返回 true。
•boolean equalsIgnoreCase(String other )
如果字符串与 other 相等 （ 忽略大小写，) 返回 tme。
•boolean startsWith(String prefix )
•boolean endsWith(String suffix )
如果字符串以 suffix 开头或结尾， 则返回 true。
•int indexOf(String str)
•int indexOf(String str, int fromlndex )
•int indexOf(int cp)
•int indexOf(int cp, int fromlndex )
返回与字符串 str 或代码点 cp 匹配的第一个子串的开始位置。这个位置从索引 0 或fromlndex 开始计算。 如果在原始串中不存在 str，返回 -1。
•int lastIndexOf(String str)
•Int lastIndexOf(String str, int fromlndex )
•int lastindexOf(int cp)
•int lastindexOf(int cp, int fromlndex )
返回与字符串 str 或代码点 cp 匹配的最后一个子串的开始位置。 这个位置从原始串尾端或 fromlndex 开始计算。
•int length( )
返回字符串的长度。
•int codePointCount(int startlndex , int endlndex ) 
返回 startlndex 和 endludex-l之间的代码点数量。没有配成对的代用字符将计入代码点。
•String replace( CharSequence oldString,CharSequence newString)
返回一个新字符串。这个字符串用 newString 代替原始字符串中所有的 oldString。可以用 String 或 StringBuilder 对象作为 CharSequence 参数。
•String substring(int beginlndex )
•String substring(int beginlndex, int endlndex )
返回一个新字符串。这个字符串包含原始字符串中从 beginlndex 到串尾或 endlndex-l的所有代码单元。
•String toLowerCase( )
•String toUpperCase( )
返回一个新字符串。 这个字符串将原始字符串中的大写字母改为小写，或者将原始字符串中的所有小写字母改成了大写字母。
•String trim( )
返回一个新字符串。这个字符串将删除了原始字符串头部和尾部的空格。
•String join(CharSequence delimiter, CharSequence ... elements ) 
返回一个新字符串， 用给定的定界符连接所有元素。

#  StringBuilder API
• StringBuilder()
构造一个空的字符串构建器。
• int length()
返回构建器或缓冲器中的代码单元数量。
• StringBuilder  append(String str)
追加一个字符串并返回 this。
•StringBuilder  append(char c)
追加一个代码单元并返回 this。
• StringBuilder  appendCodePoint(int cp)
追加一个代码点，并将其转换为一个或两个代码单元并返回 this。
• void setCharAt(int i ,char c)
将第 i 个代码单元设置为 c。
• StringBuilder insert(int offset,String str)
在 offset 位置插入一个字符串并返回 this。
• StringBuilder insert(int offset,Char c)
在 offset 位置插入一个代码单元并返回 this。
• StringBuilder  delete(1 nt startindex,int endlndex)
删除偏移量从 startindex 到 -endlndex-1 的代码单元并返回 this。
• String toString()
返回一个与构建器或缓冲器内容相同的字符串
