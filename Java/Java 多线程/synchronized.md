# synchronized

## 1 synchronized 保证三大特性

### 1.1 synchronized 与原子性

```java
/**
* 5个线程各执行1000次 i++
*/
public class AtomicityTest {

    private static int number;
    private static Object obj = new Object();

    public static void add() {
        synchronized (obj) {
            number++;
        }
    }

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    add();
                }
            }).start();
        }
        Thread.sleep(2000);
        System.out.println(number);
    }
}
```

![image-20200917212101947](synchronized.assets/image-20200917212101947.png)

**synchronized 保证原子性的原理**

对 number++ 增加同步代码块后，保证同一时间只有一个线程拿到锁能够进入同步代码块操作 number++，就不会出现安全问题。

### 1.2 synchronized 与可见性

```java
public class VisibilityTest {
    private static boolean flag = true;

    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            while (flag) {
                // 增加对象共享数据的打印，println是同步方法 
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("flag = " + flag);
                // System.out.println 源码
                /* public void println(String x) {
                    synchronized (this) {
                        print(x);
                        newLine();
                    }
                } */
            }
        }).start();

        Thread.sleep(2000);

        new Thread(() -> {
            flag = false;
            System.out.println("线程修改了变量的值为false");
        }).start();
    }
}
```

synchronized保证可见性的原理，执行synchronized时，会对应lock原子操作会刷新工作内存中共享变量的值(参考 Java内存模型——主内存与工作内存之间的交互)。

### 1.3 synchronized 与有序性

synchronized保证有序性的原理，我们加synchronized后，依然会发生重排序，只不过，我们有同步代码块，可以保证只有一个线程执行同步代码中的代码，保证有序性。

## 2 synchronized 的特性

### 2.1 可重入特性

一个线程可以多次执行synchronized，重复获取同一把锁。

```java
public class ReentrantDemo {
    public static void main(String[] args) {
        new MyThread().start();
        new MyThread().start();
    }

    static void test() {
        synchronized (MyThread.class) {
            System.out.println(Thread.currentThread().getName() + "进入了同步代码块2");
        }
    }
}

class MyThread extends Thread {
    @Override
    public void run() {
        synchronized (MyThread.class) {
            System.out.println(getName() + "进入了同步代码块1");
            ReentrantDemo.test();
          	/* synchronized (MyThread.class) {
            		System.out.println(getName() + "进入了同步代码块2");
        		} */
        }
    }
}
```

可重入原理：synchronized的锁对象中有一个计数器（recursions变量）会记录线程获得几次锁。

可重入的好处：

1. 可以避免死锁

2. 可以让我们更好的来封装代码

synchronized是可重入锁，内部锁对象中会有一个计数器记录线程获取几次锁啦，在执行完同步代码块时，计数器的数量会-1，知道计数器的数量为0，就释放这个锁。

