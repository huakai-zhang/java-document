# Java 内存模型

## 计算机结构

冯诺依曼，提出计算机由五大组成部分，输入设备，输出设备存储器，控制器，运算器。

![image-20200917204615406](Java内存模型.assets/image-20200917204615406.png)

**CPU**

中央处理器，是计算机的控制和运算的核心，我们的程序最终都会变成指令让CPU去执行，处理程序中的数据。

**内存**

我们的程序都是在内存中运行的，内存会保存程序运行时的数据，供CPU处理。

**缓存**

CPU的运算速度和内存的访问速度相差比较大。这就导致CPU每次操作内存都要耗费很多等待时间。内存的读写速度成为了计算机运行的瓶颈。于是就有了在CPU和主内存之间增加缓存的设计。最靠近CPU的缓存称为L1，然后依次是 L2，L3和主内存，CPU缓存模型如图下图所示。

<img src="Java内存模型.assets/image-20200917205201269.png" alt="image-20200917205201269" style="zoom:50%;" />

CPU Cache分成了三个级别: L1， L2， L3。级别越小越接近CPU，速度也更快，同时也代表着容量越小。

1. L1是最接近CPU的，它容量最小，例如32K，速度最快，每个核上都有一个L1 Cache。 

2. L2 Cache 更大一些，例如256K，速度要慢一些，一般情况下每个核上都有一个独立的L2 Cache。 

3. L3 Cache是三级缓存中最大的一级，例如12MB，同时也是缓存中最慢的一级，在同一个CPU插槽之间的核共享一个L3 Cache。

![image-20200917205323071](Java内存模型.assets/image-20200917205323071.png)

Cache 的出现是为了解决 CPU 直接访问内存效率低下问题的，程序在运行的过程中，CPU 接收到指令后，它会最先向CPU 中的一级缓存（L1 Cache）去寻找相关的数据，如果命中缓存，CPU 进行计算时就可以直接对 CPU Cache 中的数据进行读取和写入，当运算结束之后，再将 CPU Cache 中的最新数据刷新到主内存当中，CPU 通过直接访问 Cache 的方式替代直接访问主存的方式极大地提高了CPU 的吞吐能力。但是由于一级缓存（L1 Cache）容量较小，所以不可能每次都命中。这时 CPU 会继续向下一级的二级缓存（L2 Cache）寻找，同样的道理，当所需要的数据在二级缓存中也没有的话，会继续转向L3 Cache、内存(主存)和硬盘。

## Java 内存模型概念

``JMM(Java内存模型Java Memory Model,简称JMM)``本身是一种``抽象的概念 并不真实存在``。

Java 内存模型是 Java 虚拟机规范中所定义的一种内存模型，是标准化的，屏蔽掉了底层不同计算机的区别。

Java内存模型是一套规范，它描述了Java程序中各种变量(包括实例字段,静态字段和构成数组对象的元素等线程共享变量)的访问规则，以及在JVM中将变量存储到内存和从内存中读取变量这样的底层细节。

> Java 并发采用的是共享内存模型，线程之间共享程序的公共状态，通过写-读内存中的公共状态进行隐式通信。

JMM有以下规定：

* ``主内存``是所有线程都共享的，所有的共享变量都存储于主内存，这里所说的变量指的是``成员变量(即实例变量和类变量)``，不包含局部变量，因为局部变量是线程私有的，因此不存在竞争问题
* 每一个线程还存在自己的``工作内存``，线程的工作内存(有些地方称为栈空间)，工作内存是每个线程的私有数据区域，保留了被线程使用的变量的工作副本
* 线程对变量的`所有操作（读，取）都必须在工作内存中完成`，而不能直接读写主内存中的变量
* 不同线程之间也不能直接访问对方工作内存中的变量，线程间变量的值的传递需要`通过主内存中转来完成`

![image-20200915211552107](Java内存模型.assets/Image.bmp)

JMM关于同步规定:

1. 线程解锁前,必须把共享变量的值刷新回主内存

2. 线程加锁前,必须读取主内存的最新值到自己的工作内存

3. 加锁解锁是同一把锁 

## Java内存模型作用

Java内存模型是一套在多线程读写共享数据时，对共享数据的可见性、有序性、和原子性的规则保障。 

synchronized,volatile

## 计算机结构和 JMM 的关系

通过对前面的CPU硬件内存架构、Java内存模型以及Java多线程的实现原理的了解，我们应该已经意识到，多线程的执行最终都会映射到硬件处理器上进行执行。 

但Java内存模型和硬件内存架构并不完全一致。对于硬件内存来说只有寄存器、缓存内存、主内存的概念，并没有工作内存和主内存之分，也就是说Java内存模型对内存的划分对硬件内存并没有任何影响，因为JMM只是一种抽象的概念，是一组规则，不管是工作内存的数据还是主内存的数据，对于计算机硬件来说都会存储在计算机主内存中，当然也有可能存储到CPU缓存或者寄存器中，因此总体上来说， Java内存模型和计算机硬件内存架构是一个相互交叉的关系，是一种抽象概念划分与真实物理硬件的交叉。 

![1600332304429](Java内存模型.assets/1600332304429.png)

## 主内存与工作内存之间的交互

Java内存模型中定义了以下8种操作来完成，主内存与工作内存之间具体的交互协议，即一个变量如何从主内存拷贝到工作内存、如何从工作内存同步回主内存之类的实现细节，虚拟机实现时必须保证下面提及的每一种操作都是原子的、不可再分的。 

对应如下的流程图：

![image-20200917203657182](Java内存模型.assets/image-20200917203657182.png)

注意: 

1. 对共享变量的操作加了synchronized，才会有lock 和 unlock 操作

2. 如果对一个变量执行lock操作，将会清空工作内存中此变量的值 

3. 对一个变量执行unlock操作之前，必须先把此变量同步到主内存中 

主内存与工作内存之间的数据交互过程：

lock -> read -> load -> use -> assign -> store -> write -> unlock

------

