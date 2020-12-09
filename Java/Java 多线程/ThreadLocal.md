![image-20201030135147366](ThreadLocal.assets/image-20201030135147366.png)

#### 内存泄露是不是弱引用的锅？

从表面上看内存泄漏的根源在于使用了弱引用，但是另一个问题也同样值得思考：为什么ThreadLocalMap使用弱引用而不是强引用？

翻看官网文档的说法：

> To help deal with very large and long-lived usages, the hash table entries use WeakReferences for keys. 
>
> 为了处理非常大和长期的用途，哈希表条目使用weakreference作为键。

分两种情况讨论：

**（1）key 使用强引用**

引用ThreadLocal的对象被回收了，但是ThreadLocalMap还持有ThreadLocal的强引用，如果没有手动删除，ThreadLocal不会被回收，导致Entry内存泄漏。

**（2）key 使用弱引**

引用ThreadLocal的对象被回收了，由于ThreadLocalMap持有ThreadLocal的弱引用，即使没有手动删除，ThreadLocal也会被回收。value在下一次ThreadLocalMap调用set、get、remove的时候会被清除。

比较两种情况，我们可以发现：由于ThreadLocalMap的生命周期跟Thread一样长，如果都没有手动删除对应key，都会导致内存泄漏，但是使用弱引用可以多一层保障：弱引用ThreadLocal在GC后被清理key为null，对应的value在下一次ThreadLocalMap调用set、get、remove的时候可能会被清除。

因此，ThreadLocal内存泄漏的根源是：由于ThreadLocalMap的生命周期跟Thread一样长，如果没有手动删除对应key就会导致内存泄漏，而不是因为弱引用。

#### ThreadLocal 最佳实践

- 当需要存储线程私有变量的时候（例如在拦截器中获取的用户信息）
- 当需要实现线程安全的变量时（例如线程不安全的工具类**SimpleDateFormat**）
- 当需要减少线程资源竞争的时候