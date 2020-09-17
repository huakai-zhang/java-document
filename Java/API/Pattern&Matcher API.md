# java.util.regex.Pattern
static Pattern compile(String expression)
static Pattern compile(String expression, int flags)
把正则表达式字符串编译到一个用于快速处理匹配的模式对象中。
参数 expression    正则表达式
		flags		      CASE_INSENSITIVE,UNICODE_CASE,MULYILINE,UNIX_LINES,DOTALL,CANON_EQ
Matcher matcher(CharSequence input)
返回一个matcher对象，可以用它在输入中定位模式的匹配
String[] split(CharSequence input)
String[] split(CharSequence input, int limit)
Stream< String > splitAsStream(CharSequence input)
将输入分割成标记，其中模式指定了分隔符的形式。返回标记数组，分隔符并非标记的一部分。
参数 input    要分割成标记的字符串
		limit     所产生的字符串的最大数量。如果已经发现了limit-1个匹配的分隔符，那么返回的数组中的最后一项就包含所有剩下未分割的输入。如果limit<=0，那么输入都被分割；如果limit为0，那么坠尾的空字符串将不会置于返回的数组中。

# java.util.regex.Matcher
boolean matchers()
如果输入匹配模式，则返回true
boolean lookingAt()
如果输入的开头匹配模式，则返回true
boolean find()
boolean find(int start)
尝试查找下一个匹配，如果找到了另一个匹配，则返回true，start：开始查找的索引位置
int start(int groupIndex)
int end(int groupIndex)
返回当前匹配的开始索引和结尾索引之后的索引位置，groupIndex：群组索引（从1开始），或者表示整个匹配的0
String group(int groupIndex)
返回匹配给定群组的字符串，groupIndex：群组索引（从1开始），或者表示整个匹配的0
String replaceAll(String replacement)
String replaceFirst(String replacement)
返回从匹配器输入获得的通过将所有匹配或第一个匹配用替换字符串替换之后的字符串，replacement：替换字符串，它可以包含用$n表示的对群组的引用，这时需要用 \ $来表示字符串中包含一个 $的符号
static String quoteReplacement(String str)
引用str中所有\和 $
Matcher reset()
Matcher reset(CharSequence input)
复位匹配器状态。第二个方法将使匹配器作用于另一个不同的输入。这两个方法都返回this。
