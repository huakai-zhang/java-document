# java.util.Locale
Locale(String language)
Locale(String language, String country)
Locale(String language, String country, String variant)
用给定的语言、国家和变量创建一个Locale。在新代码中不要使用变体，应该使用IETF BCP 47语言标签。
static Locale forLanguageTag(String languageTag)
构建与给定的语言标签相对应的Locale对象
static Locale getDefault()
返回默认的Locale
static void setDefault(Locale loc)
设置默认的Locale
String getDisplayName()
返回一个在当前的Locale中所表示的用来描述Locale的名字
String getDisplayName(Locale loc)
返回一个在给定的Locale中所表示的用来描述Locale的名字
String getLanguage()
返回语言代码，它是两个小写字母组成的ISO-639代码
String getDisplayLanguage()
返回在Locale中所表示的语言名称
String getCountry()
返回国家代码，它是由两个大写字母组成的ISO-3166代码
String getDisplayCountry()
返回在当前Locale中所表示的国家名
String getDisplayCountry(Locale loc)
返回在当前Locale中所表示的国家名
String toLanguageTag()
返回该Locale对象的语言标签，例如“de-CH”
String toString()
返回Locale的描述，包括语言和国家，用下划线分割“de_CH”。

# java.text.NumberFormat
static Locale[] getAvailableLocales()
返回一个Locale对象的数组，其成员包含有可用的NumberFormat格式器
static NumberFormat getNumberInstance()
static NumberFormat getNumberInstance(Locale l)
static NumberFormat getCurrencyInstance()
static NumberFormat getCurrencyInstance(Locale l)
static NumberFormat getPercentInstance()
static NumberFormat getPercentInstance(Locale l)
为当前的或给定的Locale提供处理数字。货币或百分比的格式器
String format(double x)
String format(long x)
对给定的浮点数或整数进行格式化并以字符串的形式返回结果
Number parse(String s)
解析给定的字符串并返回数字值，如果输入字符串描述了一个浮点数，返回类型就是Double，否则返回类型就是Long。字符串必须以一个数字开头；以空白字符开头是不允许的。数字之后可以跟随其他字符，但是将被忽略。解析失败会抛出ParseException异常。
void setParseIntegerOnly(boolean b)
boolean isParseIntegerOnly()
设置或获取一个标志，该标志指示这个格式器是否应该只解析整数值
void setGroupingUsed(boolean b)
boolean isGroupingUsed()
设置或获取一个标志，该标志指示这个格式器是否会添加和识别十进制分隔符
void setMinimumIntegerDigits(int n)
int getMinimumIntegerDigits()
void setMaximumIntegerDigits(int n)
int getMaximumIntegerDigits()
void setMinimumFractionDigits(int n)
int getMinimumFractionDigits()
void setMaximumFractionDigits(int n)
int getMaximumFractionDigits()
设置或获取整数或小数部分所允许的最大或最小位数

# java.util.Currency
static Currency getInstance(String currencyCode)
static Currency getInstance(Locale locale)
返回给定的ISO 4217货币代号或给定的Locale中的国家相对应的Curreny对象
String toString()
String getCurrencyCode()
获取该货币的ISO 4217代码
String getSymbol()
String getSymbol(Locale locale)
根据默认或给定的Locale得到该货币的格式化符号。比如美元的格式化符号可能是$或US，具体是哪种形式取决于Locale
int getDefaultFractionDigits()
获取该货币小数点后的默认位数
static Set< Currency > getAvailableCurrencies()
获得所有可用的货币

# java.time.format.DateTimeFormatter
static DateTimeFormatter ofLocalizedDate(FormatStyle dateStyle)
static DateTimeFormatter ofLocalizedTime(FormatStyle dateStyle)
static DateTimeFormatter ofLocalizedDateTime(FormatStyle dateTimeStyle)
static DateTimeFormatter ofLocalizedDate(FormatStyle dateStyle, FormatStyle timeStyle)
返回用指定的风格格式化日期、时间或日期和时间的DateTimeFormatter实例
DateTimeFormatter withLocale(Locale locale)
返回当前格式器的具有给定Locale副本
String format(TemporalAccessor temporal)
返回格式化给定日期/时间所产生的字符串

### java.time.LocalDate
### java.time.LocalTime
### java.time.LocalDateTime
### java.time.ZonedDateTime
static Xxx parse(CharSequence text, DateTimeFormatter formatter)
解析给定的字符串并返回其中描述的LocalDate、LocalTime、LocalDateTime或ZonedDateTime。如果解析不成功，则抛出DateTimeParseException异常。

# java.text.Collator
static Locale[] getAvailableLocales()
返回Locale对象的一个数组，该Collator对象可用于这些对象
static Collator getInstance()
static Collator getInstance(Locale l)
为默认或给定的locale返回一个排序器
int compare(String s, String b)
如果a在b之前，则返回负值；如果它们等价，则返回0，否则返回正值。
boolean equals(String a, String b)
如果它们等价，则返回true，否则返回false
void setStrength(int strength)
int getStrength()
设置或获取排序器的强度。更强的排序器可以区分更多的词。强度的值可以是Collator.PRIMARY、Collator.TENTIARY、Collator.TERTIARY。
void setDecomposition(int decomp)
int getDecomposition(0
设置或获取排序器的分解模式。分解越细，判断两个字符串是否相等时就越严格。分解的等级值可以是Collator.NO_DECOMPOSITION、Collator.CANONICAL_DECOMPOSITION、Collator.FULL_DECOMPOSITION。
CollationKey getCollationKey(String a)
返回一个排序器键，这个键包含一个对一组字符按特定格式分解的结果，可以快速地和其他排序器键进行比较。

# java.text.CollationKey
int compareTo(CollationKey b)
如果这个键在b之前，则返回一个负值；如果两者等价，则返回0，否则返回正值。

# java.text.Normalizer
static String normalizer(CharSequence str, Normalizer.Form form)
返回str的范化形式，form的值是ND、NKD、NC或NKC之一

# java.text.MessageFormat
MessageFormat(String pattern)
MessageFormat(String pattern, Locale loc)
用给定的模式和locale构建一个消息格式对象
void applyPattern(String pattern)
给消息格式对象设置特定的模式
void setLocale(Locale loc)
Locale getLocale()
设置或获取消息中占位符所使用的locale。这个locale仅仅被通过调用applyPattern方法所设置的后续模式所使用
static String format(String pattern, Object... args)
通过使用args[i]作为占位符{i}的输入来格式化pattern字符串。
StringBuffer format(Object args, StringBuffer result, FieldPosition pos)
格式化MessageFormat的模式。args参数必须是一个对象数组。被格式化的字符串会被附加到result末尾，并返回result。如果pos等于new FieldPosition(MessageFormat.Field.ARGUMENT)，就用它的beginIndex和endIndex属性值来设置替换占位符｛1｝的文本位置。如果不关心位置信息，可以将它设为null。

# java.text.Format
String format(Object obj)
按照格式器的规则格式化给定的对象，这个方法将调用format(obj, new StringBuffer(), new FieldPosition(1)).toString()。

# java.util.ResourceBundle
static ResourceBundle getBundle(String baseName, Locale loc)
static ResourceBundle getBundle(String baseName)
在给定的locale或默认的locale下以给定的名字加载资源绑定类和它的父类。如果资源包类位于一个Java包中，那么类的名字必须包含完整的包名，例如“intl.ProgramResources”。资源包必须是public的，这样getBundle方法才能访问它们
Object getObject(String name)
从资源包或它的父包中查找一个对象
String getString(String name)
从资源包或它的父包中查找一个对象并把它转型成字符串
String[] getStringArray(String name)
从资源包或它的父包中查找一个对象并把它转型成字符串数组
Enumeration< String > getKeys()
返回一个枚举对象，枚举出资源包中的所有键，也包括父包中的键
Object handleGetObject(String key)
如果要定义自己的资源查找机制，那么这个方法就需要被复写，用来查找与给定键相关联的资源的值。

