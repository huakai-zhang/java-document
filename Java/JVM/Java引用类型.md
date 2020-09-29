# Java 引用类型

## 整体架构

![1601369123545](Java引用类型.assets/1601369123545.png)

## 强引用

当内存不足，JVM开始垃圾回收，对于强引用的对象，``就算是出现了OOM也不会对该对象进行回收``，死都不收。

强引用是最常见的普通对象引用，只要还有强引用指向一个对象，就表明对象还活着，垃圾收集器不会碰这种对象。在Java中最常见的就是强引用，把一个对象赋给一个引用变量，这个引用变量就是一个强引用。当一个对象被强引用变量引用时，它处于可达状态，它是不可能被垃圾回收机制回收的，``即使该对象以后永远都不会被用到JVM也不会回收``。因此强引用是造成Java内存泄漏的主要原因之一。

对于一个普通对象，如果没有其他的引用关系，只要超过了引用的作用域或者显式地将相应（强）引用赋值为null，一般认为就是可以被垃圾收集了。

```java
public class StrongReferenceDemo {
    public static void main(String[] args) {
        // 强引用
        Object obj1 = new Object();
        // obj2引用赋值
        Object obj2 = obj1;
        obj1 = null;
        System.gc();
        System.out.println(obj2);
    }
}
// java.lang.Object@2ff4acd0
```

## 软引用

软引用是一种相对强引用弱化了一些的引用，需要用java.lang.ref.SoftReference类来实现，可以让对象豁免一些垃圾收集。

对于只有软引用的对象来说，当系统内存充足时，不会被回收，当系统内存不足时，会被回收。

软引用通常用在对内存敏感的程序中，比如高速缓存就有用到软引用，内存够用时就保留，不够用就回收。

```java
public class SoftReferenceDemo {

    public static void softRef_Memory_Enough() {
        Object o1 = new Object();
        SoftReference<Object> softReference = new SoftReference<>(o1);
        System.out.println(o1);
        System.out.println(softReference.get());

        o1 = null;
        System.gc();

        System.out.println(o1);
        System.out.println(softReference.get());
    }

    // -Xms5m -Xmx5m -XX:+PrintGCDetails
    public static void softRef_Memory_NotEnough() {
        Object o1 = new Object();
        SoftReference<Object> softReference = new SoftReference<>(o1);
        System.out.println(o1);
        System.out.println(softReference.get());

        o1 = null;

        try {
            // 故意产生大对象
            byte[] bytes = new byte[30 * 1024 *1024];
        } catch (Throwable e) {
            e.printStackTrace();
        } finally {
            System.out.println(o1);
            System.out.println(softReference.get());
        }
    }

    public static void main(String[] args) {
        //softRef_Memory_Enough();
        softRef_Memory_NotEnough();
    }
}
```

