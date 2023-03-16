# 1.时间线
闰秒，许多计算机系统使用“平滑”方式来人为地紧邻闰秒之前让时间变慢或变快，以保证每天都是86400秒。
Java的Date和Time API规范要求Java使用的时间尺度为：
1.每天86400秒
2.每天正午与官方时间精确匹配
3.在其他时间点上，以精确定义的方式与官方时间接近匹配
Java中，Instant表示时间线的某个点。被称为新纪元的时间线原点被设置为穿过伦敦格林尼治皇家天文台的本初子午线所在时区的1970年1月1日的午夜。从该原点开始，时间按照每天86400秒向前或向后度量，精确到纳秒。最大值是Instant.MAX是公元1 000 000 000年的12月31日。
静态方法调用Instant.now()会给出当前的时刻。
equals和compareTo方法用来比较两个Instant对象，因此可以将Instant对象用作时间戳。
静态方法Duration.between，可以得到两个时刻的时间差。

Duration是两个时刻之间的时间量。可以调用ToNanos、toMillis、getSeconds、toMinutes、toHours和toDays来获得Duration按照传统单位度量的时间长度。
Duration对象的内部存储所需的空间超过了一个long值，所以秒数存储在一个long中，而纳秒数存储在一个额外的int中。如果计算要精确到纳秒级，实际上需要整个Duration的存储内容（下图）。如果不要求这个高的精度，可以用long值执行计算，然后调用toNanos。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200102115358131.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200102115525429.png)
Instant和Duration类都是不可修改的类，所以诸如multipliedBy和minus都会返回一个新的实例。

```java
public class TimeLine {
    public static void main(String[] args) {
        Instant start = Instant.now();
        runAlgorithm();
        Instant end = Instant.now();
        Duration timeElapsed = Duration.between(start, end);
        long millis = timeElapsed.toMillis();
        System.out.printf("%d milliseconds\n", millis);
        Instant start2 = Instant.now();
        runAlgorithm2();
        Instant end2 = Instant.now();
        Duration timeElapsed2 = Duration.between(start2, end2);
        System.out.printf("%d milliseconds\n", timeElapsed2.toMillis());
        boolean overTenTimesFaster = timeElapsed.multipliedBy(10).
                minus(timeElapsed2).isNegative();
        System.out.printf("The first algorithm is %smore than ten times faster", overTenTimesFaster ? "" : "not ");
    }
    public static void runAlgorithm() {
        int size = 10;
        List<Integer> list = new Random().ints().map(i -> i % 100).limit(size)
                .boxed().collect(Collectors.toList());
        Collections.sort(list);
        System.out.println(list);
    }
    public static void runAlgorithm2() {
        int size = 10;
        List<Integer> list = new Random().ints().map(i -> i % 100).limit(size)
                .boxed().collect(Collectors.toList());
        while (!IntStream.range(1, list.size()).allMatch(
             i -> list.get(i - 1).compareTo(list.get(i)) <= 0)) {
            Collections.sort(list);
        }
        System.out.println(list);
    }
}
```
# 2.本地时间
Java API有两种人类时间，本地日期/时间和时区时间。1903年6月14日就是一个本地日期。1969年7月16日 09:32:00 EDT是一个时区时间，表示的是时间线上的一个精确时刻。
API的设计者不推荐使用时区时间，除非确实想要表示绝对时间的实例。生日、假日、计划时间等通常最好表示成本地日期和时间。
LocalDate是带有年、月、日的日期。为了构建LocalDate对象，可以使用now或of静态方法。
与UNIX和java.util.Date中使用的月从0开始计算，而年从1900开始计算的不规则惯用法不同，LocalDate通常使用月份的数字，或者使用Month枚举。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020010213100289.png)![在这里插入图片描述](https://img-blog.csdnimg.cn/20200102131007652.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)
计算程序员日，每年的第256天：

```java
LocalDate programmersDay = LocalDate.of(2019, 1, 1).plusDays(255);
```
本地时间的Period，它表示的是流逝的年、月或日的数量。
until方法会产生两个本地日期之间的时长，但每个月的天数不尽相同，为了确定到底有多少天：

```java
independenceDay.until(christmas, ChronoUnit.DAYS);
```
LocalDate的有些方法可能会创建并不存在的时间，比如1月31日加上1个月，不会产生2月31日，也不会抛出异常，而是会返回该月有效的最后一天：

```java
LocalDate.of(2020, 1, 31).plusMonths(1);
LocalDate.of(2020, 3, 31).minusMonths(1);
//都产生2020年2月29日
```
getDayOfWeek会产生星期日期，即DayOfWeek枚举的某个值。DayOfWeek.SUNDAY=7
DayOfWeek枚举具有便捷的plus和minus，以7位模进行计算星期日期。例如DayOfWeek.SATURDAY.plus(3)会产生DayOFWeek.TUESDAY。
除了LoaclDate之外，还有MonthDay、YearMonth和Year可以描述部分日期。12月25日（没有年）可以表示成MonthDay对象。

```java
public class LocalDates {
    public static void main(String[] args) {
        LocalDate today = LocalDate.now();
        System.out.println("today: " + today);
        LocalDate alonzosBirthday = LocalDate.of(1903, 4, 14);
        alonzosBirthday = LocalDate.of(1903, Month.JUNE, 14);
        System.out.println("alonzosBirthday: " + alonzosBirthday);
        LocalDate programmersDay = LocalDate.of(2018, 1, 1).plusDays(255);
        System.out.println("programmersDay: " + programmersDay);
        LocalDate nationalDay = LocalDate.of(2019, Month.NOVEMBER, 1);
        LocalDate midAutumnFestival = LocalDate.of(2019, Month.SEPTEMBER, 13);
        System.out.println("Until nationalDay: " + midAutumnFestival.until(nationalDay));
        System.out.println("Until nationalDay: " + midAutumnFestival.until(nationalDay, ChronoUnit.DAYS));
        System.out.println(LocalDate.of(2020, 1, 31).plusMonths(1));
        System.out.println(LocalDate.of(2020, 3, 31).minusMonths(1));
        DayOfWeek startOfLastMillennium = LocalDate.of(1900, 1, 1).getDayOfWeek();
        System.out.println("startOfLastMillennium: " + startOfLastMillennium);
        System.out.println(startOfLastMillennium.getValue());
        System.out.println(DayOfWeek.SATURDAY.plus(3));
    }
}
```
# 3.日期调整器
TemporalAdjusters类提供了大量用于常见调整的静态方法。某个月的第一个星期二：

```java
LocalDate firstTuesday = LocalDate.of(2020, 1, 1).with(
        TemporalAdjusters.nextOrSame(DayOfWeek.TUESDAY));
```
一如既往，with方法会返回一个新的LocalDate对象，而不会修改原来的对象。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200102135325808.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)
还可以通过实现TemporalAdjuster接口来创建自己的调整器，用于计算下一个工作日的调整器：

```java
TemporalAdjuster NEXT_WORKDAY = w -> {
	// lambda的参数类型为Temporal，必须强制转型
    LocalDate result = (LocalDate) w;
    do {
        result = result.plusDays(1);
    } while (result.getDayOfWeek().getValue() >= 6);
    return result;
};
LocalDate today = LocalDate.now();
LocalDate backToWork = today.with(NEXT_WORKDAY);
System.out.println(backToWork);
```
或者可以使用ofDateAdjuster方法来避免强转，该方法期望得到的参数为UnaryOperator< LocalDate >的lambda表达式：

```java
TemporalAdjuster NEXT_WORKDAY = TemporalAdjusters.ofDateAdjuster(w -> {
    LocalDate result = w;
    do {
        result = result.plusDays(1);
    } while (result.getDayOfWeek().getValue() >= 6);
    return result;
});
```
# 4.本地时间
LocalTime表示当日时刻，例如15:30:00。可以用now或of方法创建其实例：

```java
LocalTime rightNow = LocalTime.now();
LocalTime bedtime = LocalTime.of(22, 30);
// plus和minus按照一天24小时操作
LocalTime wakeup = bedtime.plusHours(8);
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200102141424946.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)
还有一个表示日期和时间的LocalDateTime类。这个类适合存储固定时区的时间点。例如，排课或排程。但是如果需要处理不同时区的用户，那么就应该使用ZonedDateTime类。
# 5.时区时间
互联网编码分配管理机构（Internet Assigned Numbers Authority，IANA）保存着一个数据库，里面存储着世界上已知的时区。Java使用了IANA数据库。
每个时区都有一个ID，例如America/New_York。ZoneId.getAvailableZoneIds()查看所有可用的时区。
给定一个时区ID，静态方法ZonedId.of(id)可以产生一个ZonedId对象。可以调用local.atZone(zoneId)用找个对象将LocalDateTime对象转换为ZonedDateTime对象，或者：

```java
ZonedDateTime zonedDateTime = ZonedDateTime.of(2020, 1, 2, 14, 39, 0, 0, ZoneId.of("Asia/Shanghai"));
// 获得Instant对象
System.out.println(zonedDateTime.toInstant());
// 2020-01-02T14:39+08:00[Asia/Shanghai]
// 2020-01-02T06:39:00Z
```
反过来，如果有一个时刻对象，调用instant.atZone(ZoneId.of("UTC"))可以获得格林尼治的ZonedDateTime对象，或者使用其他的ZoneId获得地球上其他地方的ZoneId。
UTC代表协调世界时，这是英语“Coordinated Universal Time”和法文“Temps Universal Coordine”首字母缩写的折中。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200102144734284.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200102144750562.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)
如果将时间设置在下个星期，不要直接加上一个7天的Duration，而是使用Period类。

```java
public class ZonedTimes {
    public static void main(String[] args) {
        ZonedDateTime apollo11launch = ZonedDateTime.of(2020, 1, 2, 14, 39, 0, 0, ZoneId.of("Asia/Shanghai"));
        System.out.println("apollo11launch: " + apollo11launch);
        Instant instant = apollo11launch.toInstant();
        System.out.println("instant: " + instant);
        ZonedDateTime zonedDateTime = instant.atZone(ZoneId.of("UTC"));
        System.out.println("zonedDateTime: " + zonedDateTime);
        // 中欧地区在3月31日2:00切换夏令时，试图构建2:30，实际得到的是3:30
        ZonedDateTime skipped = ZonedDateTime.of(LocalDate.of(2013, 3, 31),
                LocalTime.of(2, 30), ZoneId.of("Europe/Berlin"));
        System.out.println("skipped: " + skipped);
        // 反过来，时钟会回拨慢一个小时，同一个本地时间就会出现两次
        ZonedDateTime ambiguous = ZonedDateTime.of(LocalDate.of(2013, 10, 27),
                LocalTime.of(2, 30), ZoneId.of("Europe/Berlin"));
        ZonedDateTime anHourLater = ambiguous.plusHours(1);
        System.out.println("ambiguous: " + ambiguous);
        System.out.println("anHourLater: " + anHourLater);
        ZonedDateTime meeting = ZonedDateTime.of(LocalDate.of(2013, 10, 31),
                LocalTime.of(14, 30), ZoneId.of("America/Los_Angeles"));
        System.out.println("meeting: " + meeting);
        // 小心！夏令时不起作用
        ZonedDateTime nextMeeting = meeting.plus(Duration.ofDays(7));
        System.out.println("nextMeeting: " + nextMeeting);
        nextMeeting = meeting.plus(Period.ofDays(7));
        System.out.println("nextMeeting: " + nextMeeting);
    }
}
```

# 6.格式化和解析
DataTimeFormatter类提供了用三种用于打印日期/时间值的格式器：
1.预定义的格式器
2.Locale相关的格式器
3.带有定制模式的格式器
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200102184254975.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)
要使用标准的格式器，可以直接调用其format方法：

```java
String formatted = DateTimeFormatter.ISO_OFFSET_DATE_TIME.format(apollo11launch);
```
标准格式器只要是为了机器刻度的时间戳设计的。为了向人类读者表示日期和时间，可以使用Locale相关的格式器。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200102184734952.png)
静态方法ofLocalizedDate、ofLocalizedTime和ofLocalizedDateTime可以创建这种格式器。
这些方法使用了默认的Locale。为了切换到不同的Locale，可以直接使用withLocale方法。
DayofWeek和Month枚举都有getDisplayName方法，可以按照不同的Locale和格式给出星期日期和月份的名字。

java.time.format.DateTimeFormatter类被设置用来替代java.util.DateFormat。如果为了向后兼容性而需要使用后者，那么可以调用formatter.toFormat()。
可以指定模式，来定制自己的日期格式：
DateTimeFormatter.ofPattern("E yyyy-MM-dd HH:mm");
每个字母表示一个不同的时间域，而字母重复的次数对应于所选择的特定格式。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200102185629888.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200102185733606.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)
为了解析字符串中的日期/时间值，可以使用众多的静态parse方法之一。

```java
public class Formatting {
    public static void main(String[] args) {
        ZonedDateTime apollo11launch = ZonedDateTime.of(1969, 7, 16, 9, 32, 0, 0,
                ZoneId.of("America/New_York"));

        String formatted = DateTimeFormatter.ISO_OFFSET_DATE_TIME.format(apollo11launch);
        System.out.println(formatted);

        DateTimeFormatter formatter = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.LONG);
        formatted = formatter.format(apollo11launch);
        System.out.println(formatted);

        formatted = formatter.withLocale(Locale.FRENCH).format(apollo11launch);
        System.out.println(formatted);

        formatter = DateTimeFormatter.ofPattern("E yyyy-MM-dd HH:mm");
        formatted = formatter.format(apollo11launch);
        System.out.println(formatted);

        // 调用使用了标准的ISO_LOCAL_DATE格式器
        LocalDate churchsBirthday = LocalDate.parse("1903-06-14");
        System.out.println("churchsBirthday: " + churchsBirthday);
        // 使用一个定制的格式器
        apollo11launch = ZonedDateTime.parse("1969-07-16 03:32:00-0400",
                DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ssxx"));
        System.out.println("apollo11launch: " + apollo11launch);

        for (DayOfWeek w : DayOfWeek.values()) {
            System.out.print(w.getDisplayName(TextStyle.SHORT, Locale.ENGLISH) + " ");
        }
    }
}
```
# 7.与遗留代码的互操作
Instant类近似于java.util.Date。Java SE 8中，Date有两个额外方法：将Date转换为Instant的toInstant方法，以及反方向的转换的静态的from方法。
类似地，ZonedDateTime近似于java.util.GregorianCalendar，在Java SE 8中，这个类有细粒度的转换方法。toZonedDateTime方法可以将GregorianCalendar转换为ZonedDateTime，而静态的from方法可以执行方向转换。
另一个可以用于日期和时间类的转换集位于java.sql包中。还可以传递一个DateTimeFotmatter给使用java.text.Format的遗留代码。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200102191450161.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200102191456791.png)
