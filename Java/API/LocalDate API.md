• static LocalTime now( )
构造一个表示当前日期的对象。
• static LocalTime of(int year, int month , int day )
构造一个表示给定日期的对象。
•int getYear( )
•int getMonthValue( )
•int getDayOfMonth( )
得到当前日期的年、 月和曰。
•DayOfWeek getDayOfWeek
得到当前日期是星期几， 作为 DayOfWeek 类的一个实例返回。 调用 getValue 来得到1 ~ 7 之间的一个数， 表示这是星期几， 1 表示星期一， 7 表示星期日。
•Local Date piusDays(int n )
•Local Date minusDays(int n)
生成当前日期之后或之前 n 天的日期。
