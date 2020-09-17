# Biglnteger 
• Biglnteger add(Biglnteger other)
• Biglnteger subtract(Biglnteger other)
• Biglnteger multipiy(Biginteger other)
• Biglnteger divide(Biglnteger other)
• Biglnteger mod(Biglnteger other)
返冋这个大整数和另一个大整数 other•的和、 差、 积、 商以及余数。
• int compareTo(Biglnteger other)
如果这个大整数与另一个大整数 other 相等， 返回 0; 如果这个大整数小于另一个大整数 other, 返回负数； 否则， 返回正数。
• static Biglnteger valueOf(1 ong x)
返回值等于 x 的大整数。

# BigDecimal 
• BigDecimal add(BigDecimal other)
• BigDecimal subtract(BigDecimal other)
• BigDecimal multipiy(BigDecimal other)
• BigDecimal divide(BigDecimal other RoundingMode mode) 
返回这个大实数与另一个大实数 other 的和、 差、 积、 商。要想计算商， 必须给出舍入方式 （ rounding mode。) RoundingMode.HALF UP 是在学校中学习的四舍五入方式( BP , 数值 0 到 4 舍去， 数值 5 到 9 进位）。它适用于常规的计算。
• BigDecimal divide(BigDecimal divisor, int scale, RoundingMode roundingMode)
scale保留位数
• int compareTo(BigDecimal other)
如果这个大实数与另一个大实数相等， 返回 0 ; 如果这个大实数小于另一个大实数，返回负数；否则，返回正数。
• static BigDecimal valueOf(long x)
• static BigDecimal valueOfl ong x ,int scale)
返回值为 X 或 x / 10^scale的一个大实数。

## BigDecimal 舍入模式（Rounding mode）介绍
#### UP
远离零方向舍入
public final static int ROUND_UP = 0;
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190415173829834.png)
#### DOWN
向零方向舍入
public final static int ROUND_DOWN = 1;
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190415173948528.png)
#### CEILING
向正无限大方向舍入
public final static int ROUND_CEILING = 2;
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190415174029182.png)
####  FLOOR
向负无限大方向舍入
public final static int ROUND_FLOOR = 3;
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190415174106890.png)

#### HALF_UP 
四舍五入
public final static int ROUND_HALF_UP = 4;
####  HALF_DOWN
向最接近的数字方向舍入，如果与两个相邻数字的距离相等，则向下舍入。
public final static int ROUND_HALF_DOWN = 5;
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190415174444712.png)

#### HALF_EVEN
向最接近数字方向舍入，如果与两个相邻数字的距离相等，则向相邻的偶数舍入。
public final static int ROUND_HALF_EVEN = 6;
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190415174522537.png)

#### UNNECESSARY
用于断言请求的操作具有精确结果，因此不发生舍入。
public final static int ROUND_UNNECESSARY =  7;
