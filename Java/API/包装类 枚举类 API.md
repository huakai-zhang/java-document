# java.lang.Integer
• int intValue( )
以 int 的形式返回 Integer 对象的值（在 Number 类中覆盖了 intValue方法）。
• static String toString(int i )
以一个新 String 对象的形式返回给定数值 i 的十进制表示。
參 static String toString(int i ,int radix )
返回数值 i 的基于给定 radix 参数进制的表示。
• static int parselnt(String s)
• static int parseInt(String s,int radix)
返回字符串 s 表示的整型数值， 给定字符串表示的是十进制的整数（第一种方法，)
或者是 radix 参数进制的整数（第二种方法 。)
• static Integer valueOf(String s)
•Static Integer value Of(String s, int radix)
返回用 s 表示的整型数值进行初始化后的一个新 Integer 对象， 给定字符串表示的是十
进制的整数（第一种方法，) 或者是 radix 参数进制的整数（第二种方法。)
• static int compare(int x , int y)
如果 x < y 返回一个负整数；如果 x 和 y 相等，则返回 0; 否则返回一个负整数。

# java.lang.Double
•static int compare(double x , double y)
如果 x < y 返回一个负数；如果 x 和 y 相等则返回 0; 否则返回一个负数

# java.text.NumberFormat
• Number parse(String s)
返回数字值，假设给定的 String 表示了一个数值。

 # java.Iang.Enum <E>
•static Enum valueOf(Class enumClass , String name )
返回指定名字、给定类的枚举常量。
•String toString( )
返回枚举常量名。
•int ordinal ( )
返回枚举常量在 enum 声明中的位置，位置从 0 开始计数。
•int compareTo( E other )
如果枚举常量出现在 Other 之前， 则返回一个负值；如果 this=other，则返回 0; 否则，返回正值。枚举常量的出现次序在 enum 声明中给出。
