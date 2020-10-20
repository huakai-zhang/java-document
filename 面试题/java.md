### Java SE

#### Q1 两个对象值相同 (x.equals(y) == true) ，hashCode应当相同；但hashCode相同，(x.equals(y) == true)不一定成立。



#### Q2 是否可以继承 String？

String 类是 final 类，不可以被继承。



#### Q3 为什么函数不能根据返回类型来区分重载？

因为调用时不能指定类型信息，编译器不知道你要调用哪个函数。 



#### Q4 ==和 equals 的区别？

equals 方法不能用于基本数据类型的变量，在未重写 equals 方法时，和 == 一样。

== 如果比较的对象是基本数据类型，则比较的是数值是否相等；如果比较的是引用数据类型，则比较的是对象的地址值是否相等。 



#### Q5 String s = "Hello";s = s + " world!";代码执行后，原始的 String 对象中的内容到底变了没有？

没有。因为 String 被设计成不可变(immutable)类，所以它的所有对象都是不可变对象。在这段代码中，s 原先指向一个 String 对象，内容是 "Hello"，然后我们对 s 进行了“+”操作，这时 s 不指向原来那个对象了，而指向了另一个 String 对象，内容为"Hello world!"，原来那个对象还存在于内存之中，只是 s 这个引用变量不再指向它了。 



#### Q6 error 和 exception 的区别？

Error和Exception都继承自Throwable类，区别如下：

Error一般指与虚拟机相关的问题，如内存溢出，栈溢出等。对于这类错误导致的应用程序中断，仅靠程序本身无法恢复和预防，遇到这种错误，建议让程序终止。

Exception，编译时异常（也叫强制性异常，CheckedException ），运行时异常（也叫非强制性异常，RuntimeException）



#### Q7 final、finally、finalize 的区别？

``final`` 用于声明属性，方法和类，分别表示属性不可变，方法不可覆盖，被其修饰的类不可继承。 

``finally`` 异常处理语句结构的一部分，表示总是执行。 

``finalize`` Object 类的一个方法，在垃圾回收器执行的时候会调用被回收对象的此方法，可以覆盖此方法，提供垃圾收集时的其他资源回收，例如关闭文件等。该方法更像是一个对象生命周期的临终方法，当该方法被系统调用则代表该对象即将“死亡”，但是需要注意的是，我们主动行为上去调用该方法并不会导致该对象“死亡”，这是一个被动的方法（其实就是回调方法），不需要我们调用。



#### Q8 switch 是否能作用在 byte 上，是否能作用在 long 上，是否能作用在 String上?

JDK5之前，只能作用在byte，short，char，int（包括其包装类型）。JDK5中引入了枚举类型，JDK7开始，支持String类型。



#### Q9 String 、StringBuilder 、StringBuffer 的区别？

Java 平台提供了两种类型的字符串：String 和 StringBuffer/StringBuilder，它们都可以储存和操作字符串，区别如下: 

``String`` 是只读字符串，也就意味着 String 引用的字符串内容是不能被改变的（引用发生改变）。

``StringBuffer`` 表示的字符串对象可以直接进行修改。 

``StringBuilder`` 是 Java5 中引入的，它和 StringBuffer 的方法完全相同，区别在于它是在单线程环境下使用的， 因为它的所有方法都没有被 synchronized 修饰，因此它的效率理论上也比 StringBuffer 要高。 

> 在Java中无论使用何种方式进行``字符串``连接，实际上都使用的是StringBuilder。但 + 的方式，如果在for 语句内部，会每执行一次循环创建一个 StringBuilder 对象。
>
>  ``字符串常量``相加，不会用到StringBuilder对象（String s = "Program" + "ming";在被编译器优化成了String s = "Programming"; ）。
>
> String 对象的``intern()``方法会得到字符串对象在常量池中对应的版本的引用（如果常量池中有一个字符串与 String 对象的 equals 结果是 true），如果常量池中没有对应的字符串，则该字符串将被添加到常量池中，然后返 回常量池中字符串的引用；



#### Q10 在java 中 wait 和 sleep 方法的不同？  

最大的不同是在等待时 wait 会释放锁，而 sleep 一直持有锁。wait 通常被用于线程间交互，sleep 通常被用于暂停执行。



#### Q11 如何控制某个方法允许并发访问线程的个数？

Semaphore



#### Q12 同一个类中的 2 个方法都加了同步锁，多个线程能同时访问同一个类中的这两个方法吗？

Lock 可以让等待锁的线程响应中断，Lock 获取锁，之后需释放锁。多个线程不可访问同一个类中的 2 个加了 Lock 锁的方法。

使用 synchronized 时，当我们访问同一个类对象的方法（ synchronized (this) 加锁），是同一把锁，所以不可以访问该对象的其他 synchronized 方法。

但是访问同一个类实例对象的两个Runnable对象的run方法时，多个线程是能同时访问同个类的两个同步方法的。这是因为
`synchronized(this){ //... }`中锁住的不是代码块，即这个锁在run方法中，但是并不是同步了这个run方法，而是括号中的对象this,也就是说，多个线程会拿到各自的锁，就能够同时执行run方法。(在run方法前声明synchronized也是同样的效果)

```java
public Runnable runnable1 = new Runnable() {
      @Override
      public void run() {
         synchronized (this) {
            //同步锁
            while (number < 1000) {
               try {
                  //打印是否执行该方法
                  System.out.println(Thread.currentThread().getName() + " run1: " + number++);
               } catch (Exception e) {
                  e.printStackTrace();
               }
            }
            System.out.println(this);
         }
      }
   };

new Thread(example.runnable1).start(); //同步方法1
new Thread(example.runnable2).start(); //同步方法2
```

打印出这个this对象，是两个不同的类实例对象。

