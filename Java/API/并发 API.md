# java.lang.Thread
static void sleep(long millis)
休眠给定的毫秒数
Thread(Runnable target)
构造一个新线程，用于调用给定目标的run()方法
void start()
启动这个线程，将引发调用run()方法。这个方法将立即返回，并且新线程将并发运行。
void run()
调用关联Runnable的run方法
void interrupt()
向线程发送中断请求。线程的中断状态将被设置为true。如果目前该线程被一个sleep调用阻塞，那么，InterruptedException异常被抛出
static boolean interrupted()
测试当前线程是否被中断。静态方法，这一调用会产生副作用—它将当前线程的中断状态重置为false
boolean isInterrupted()
测试线程是否被终止。不像静态的中断方法，这一调用不改变线程的中断状态
static Thread currentThread()
返回代表当前执行线程的Thread对象
void join()
等待终止指定的线程
void join(long millis)
等待指定的线程死亡或经过指定的毫秒数
Thread.State getState()
得到这一线程的状态，NEW、RUNNABLE、BLOCKED、WAITING、TIMED_WAITING、TERMINATED
void stop()
停止该线程，已过时
void setPriority(int new Priority)
设置线程的优先级。优先级必须在Thread.MIN_PRIORITY(1)与Thread.MAX_PRIORITY(10)之间，一般使用Thread.NORM_PRIORITY优先级
static int MIN_PRIORITY
线程的最小优先级，值为1
static int MAX_PRIORITY
线程的最高优先级，值为10
static int NORM_PRIORITY
线程的默认优先级，值为5
static void yield()
导致当前执行线程处于让步状态。如果有其他的可运行线程具有至少与此线程同样高的优先级，那么这些线程接下来会被调度。
void setDaemon(boolean isDaemon)
标识该线程为守护线程或用户线程。这一方法必须在线程启动之前调用
static void setDefaultUncaughtExceptionHandler(Thread.UncaughtExceptionHandler handler)
static Thread.UncaughtExceptionHandler getDefaultUncaughtExceptionHandler()
设置或获取未捕获异常的默认处理器
void setUncaughtExceptionHandler(Thread.UncaughtExceptionHandler handler)
Thread.UncaughtExceptionHandler getUncaughtExceptionHandler()
设置或获取未捕获异常的处理器，如果没有安装处理器，则将线程组对象作为处理器

# java.lang.Thread.static void  UncaughtExceptionHandler
void uncaughtException(Thread t, Throwable e)
当一个线程因未捕获异常而终止，按规定要将客户报告记录到日志中

# java.lang.ThreadGroup
void uncaughtExcaption(Thread t, Throwable e)
如果有父线程组，调用父线程组的这一方法；或者，如果Thread类有默认处理器，调用该处理器，否则，输出栈轨迹到标准错误流上（但是e是TreadDeath对象，栈轨迹是被禁用的，TreadDeath由stop产生，而该方法是过时的）。

# java.util.concurrent.locks.Lock
void lock()
获取这个锁，如果锁同时被另一个线程拥有则发生阻塞
void unlock()
释放这个锁
Condition newCondition()
返回一个与该锁相关的条件对象
boolean tryLock()
尝试获得锁而没有发生阻塞；如果成功返回真。这个方法会抢夺可用的锁，即使该锁有公平加锁策略，即便其他程序已经等待很久也是如此
boolean tryLock(long time, TimeUnit unit)
尝试获得锁，阻塞时间不会超过给定的值；如果成功返回true
void lockInterruptibly()
获得锁，但是会不确定地发生阻塞。如果线程被中断，抛出一个InterruptedException异常

# java.util.concurrent.locks.ReentrantLock
ReentrantLock()
构建一个可以被用来保护临界区的可重入锁
ReentrantLock(boolean fair)
构建一个带有公平策略的锁。一个公平锁偏爱等待时间最长的线程。但是，这一公平的保证将大大降低性能。所以默认情况下，锁没有被强制公平。

# java.util.concurrent.locks.Condition
void await()
将该线程放到条件的等待集中
void signalAll()
解除该条件的等待集中的所有线程的阻塞状态
void signal()
从该条件的等待集中随机地选择一个线程，解除其阻塞状态
boolean await(long time, TimeUnit unit)
进入该条件的等待集，直到线程从等待集中移出或等待了指定的时间之后才解除阻塞。如果因为等待时间到了而返回就返回false，否则返回true
void awaitUninterruptibly()
进入该条件的等待集，知道线程从等待集移除才解除阻塞。如果线程被中断，该方法不会抛出InterruptedException

# java.lang.Object
void notifyAll()
解除那些在该对象上调用wait方法的线程的阻塞状态。该方法只能在同步方法或同步块内部调用。如果当前线程不是对象锁的持有者，该方法抛出一个IllegalStateException异常
void notify()
随机选择一个在该对象调用wait方法的线程，解除其阻塞状态。该方法只能在同步方法或同步块内部调用。如果当前线程不是对象锁的持有者，该方法抛出一个IllegalStateException异常
void wait()
导致线程进入等待状态直到它被通知。该方法只能在一个同步方法中调用。如果当前线程不是对象锁的持有者，该方法抛出一个IllegalStateException异常
void wait(long millis)
void wait(long millis, int nanos)
导致线程进入等待状态直到它被通知或者经过指定的时间。该方法只能在一个同步方法中调用。如果当前线程不是对象锁的持有者，该方法抛出一个IllegalStateException异常
millis毫秒数
nanos纳秒数 <1 000 000

# java.lang.ThreadLocal< T >
T get()
得到这个线程的当前值。如果是首次调用get，会调用initialize来得到这个值
protected initialize()
应覆盖这个方法来提供一个初始值。默认情况下，这个方法返回null
void set(T t)
为这个线程设置一个新值
void remove()
删除对应这个线程的值
static < S > ThreadLocal< S > withInitial(Supplier< ? extends S > supplier)
创建一个线程局部变量，其初始值通过调用给定的supplier生成

# java.util.concurrent.ThreadLocalRandom
static ThreadLocalRandom current()
返回特定于当前线程的Random类实例

# java.util.concurrent.locks.ReentrantReadWriteLock
Lock readLock()
得到一个可以被多个读操作共用的读锁，但会排斥所有写操作
Lock writeLock()
得到一个写锁，排除所有其他读操作和写操作

# java.util.concurrent.ArrayBlockingQueue< E >
ArrayBlockingQueue(int capacity)
ArrayBlockingQueue(int capacity, boolean fair)
构造一个带有指定的容量和公平性设置的阻塞队列。该队列用循环数组实现
# java.util.concurrent.LinkedBlockingQueue< E >
# java.util.concurrent.LinkedBlockingDeque< E >
LinkedBlockingQueue()
LinkedBlockingDeque()
构造一个无上限的阻塞队列或双向队列，用链表实现
LinkedBlockingQueue(int capacity)
LinkedBlockingDeque(int capacity)
根据指定容量构建一个有限的阻塞队列或双向队列，用链表实现
# java.util.concurent.DelayQueue< E extends Delayed >
DelayQueue()
构造一个包含Delayed元素的无解的阻塞时间有限的阻塞队列。只有那些延迟已经超过时间的元素可以从队列移除
# java.util.concurrent.Delayed
long getDelay(TimeUnit unit)
得到该对象的延迟，用给定的时间单位进行度量
# java.util.concurrent.PriorityBlockingQueue< E >
PriorityBlockingQueue()
PriorityBlockingQueue(int initialCapacity)
PriorityBlockingQueue(int initialCapacity, Comparator< ? super E > comparator)
构造一个无边界阻塞优先队列，用堆实现
initialCapacity 优先队列的初始容量，默认为11
comparator  用来对元素进行比较的比较器，如果没有指定，则元素必须实现Comparable接口

# java.util.concurrent.BlockingQueue< E >
void put(E element)
添加元素，在必要时阻塞
E task()
移除并返回头元素，必要时阻塞
boolean offer(E element, long time, TimeUnit unit)
添加给定的元素，如果成功返回true，如果必要时阻塞，直至元素已经被添加或超时
E poll(long time, TimeUnit unit)
移除并返回头元素，必要时阻塞，直至元素可用或超时用完。失败时返回null

# java.util.concurrent.BlockingDeque< E >
void putFirst(E element)
void putLast(E element)
添加元素，必要时阻塞
E takeFirst()
E takeLast()
移除并返回头元素或尾元素，必要时阻塞
boolean offerFirst(E element, long time, TimeUnit unit)
boolean offerLast(E element, long time, TimeUnit unit)
添加给定的元素，成功时返回true，必要时阻塞直至元素被添加或超时
E pollFirst(long time, TimeUnit unit)
E pollLast(long time, TimeUnit unit)
移动并返回头元素或尾元素，必要时阻塞，直至元素可用或超时，失败时返回null

# java.util.concurrent.TransferQueue< E >
void transfer(E element)
boolean tryTransfer(E element, long time, TimeUnit unit)
传输一个值，或者尝试在给定的超时时间内传输这个值，这个调用将阻塞，直到另一个线程将元素删除。第二个方法会在调用成功时返回true。

# java.util.concurrent.ConcurrentLinkedQueue< E >
ConcurrentLinkedQueue< E >()
构造一个可以被多线程安全访问的无边界无阻塞的队列

# java.util.concurrent.ConcurrentSkipListSet< E >
ConcurrentSkipListSet< E >()
ConcurrentSkipListSet< E >(Comparator< ? super E > comp)
构造一个可被多线程安全访问的有序集。第一个构造器要求元素实现Comparable接口

### java.util.concurrent.ConcurrentHashMap< K, V >
### java.util.concurrent.ConcurrentSkipListMap< K, V >
ConcurrentHashMap< K, V >()
ConcurrentHashMap< K, V >(int initialCapacity)
ConcurrentHashMap< K, V >(int initialCapacity, float loadFactor, int concurrencyLevel)
构造一个可以被多线程安全访问的散列映射表
initialCapacity集合的初始容量，默认值为16
loadFactor控制调整：如果每一个桶的平均负载超过这个因子，表大小会被重新调整，默认0.75
concurrencyLevel并发写者线程的估计数目
ConcurrentSkipListMap< K, V >()
ConcurrentSkipListMap< K, V >(Comparator< ? super E > comp)
构造一个可以被多线程安全访问的有序的映射表，第一个构造器要求键实现Comparable接口

# java.util.Collections
static < E > Collection< E > synchronizedCollection(Collection< E > c)
static < E > List synchronizedList(List< E > c)
static < E > Set synchronizedSet(Set< E > c)
static < E > SortedSet synchronizedSortedSet(SortedSet< E > c)
static < K, V > Map<K, V> synchronizedMap(Map< K, V > c)
static < K, V > SortedMap< K, V > synchronizedSortedMap(SortedMap< K, V > c)
构建集合视图，该集合的方法是同步的

# java.util.concurrent.Callable< V >
V call()
运行一个将产生结果的任务

# java.util.concurrent.Future< V >
V get()
V get(long time, TimeUnit unit)
获取结果，如果没有结果可用，则阻塞直到真正得到结果超过指定的时间为止。如果不成功，第二个方法会抛出TimeoutException异常
boolean cancel(boolean mayInterrupt)
尝试取消这一任务的运行，如果任务已经开始，并且mayInterrupt为true，它就会被中断。如果成功执行了取消操作，返回true
boolean isCancelled()
如果任务在完成前被取消了，则返回true
boolean isDone()
如果任务结束，无论是正常结束、中途取消或发生异常，都返回true

# java.util.concurrent.FutureTask< V >
FutureTask(Callable< V > task)
FutureTask(Runnable task, V result)
构造一个既是Future< V >又是Runnable的对象

# java.util.concurrent.Executors
ExecutorService newCachedThreadPool()
返回一个带缓存的线程池，该池在必要的时候创建线程，在线程空闲60秒之后终止线程。
ExecutorService newFixedThreadPool(int threads)
返回一个线程池，该池中的线程数由参数指定
ExecutorService newSingleThreadExecutor()
返回一个执行器，它在一个单个的线程中依次执行各个任务
ScheduledExecutorService newScheduledThreadPool(int threads)
返回一个线程池，它使用给定的线程数来调度任务
ScheduledExecutorService newSingleThreadScheduledExecutor()
返回一个执行器，它在一个单独线程中调度任务

# java.util.concurrent.ExecutorService
Future< T > submit(Callable< T > task)
Future< T > submit(Runnable task, T result)
Future< ? > submit(Runnable task)
提交指定的任务去执行
void shutdown()
关闭服务，会先完成已经提交的任务而不再接受新的任务
T invokeAny(Collection< Callable< T>> tasks)
T invokeAny(Collection< Callable< T>> tasks, long timeout, TimeUnit unit)
执行给定的任务，返回其中一个任务的结果。第二个方法若发生超时，抛出一个TimeoutException异常
List< Future< T>> invokeAll(Collection< Callable< T>> tasks)
List< Future< T>> invokeAll(Collection< Callable< T>> tasks, long timeout, TimeUnit unit)
执行给定的任务，返回所有任务的结果，第二个方法若发生超时，抛出一个TimeoutException异常

# java.util.concurrent.ThreadPoolExecutor
int getLargestPoolSize()
返回线程池在该执行器生命周期的最大尺寸

# java.util.concurrent.ScheduledExecutorService
ScheduledFuture< V > schedule(Callable< V  > task, long time, TimeUnit unit)
ScheduledFuture< ? > schedule(Runnable task, long time, TimeUnit unit)
预定在指定的时间之后执行任务
ScheduledFuture< ? > scheduleAtFixedRate(Runnable task, long initialDelay, long period, TimeUnit unit)
预定在初始的延迟结束后，周期性地运行给定的任务，周期长度是period
ScheduledFuture< ? > scheduleWithFixedDelay(Runnable task, long initialDelay, long delay, TimeUnit unit)
预定在初始的延迟结束后周期性地运行给定的任务，在一次调用完成和下一次调用开始之间有长度为delay的延迟

# java.util.concurrent.ExecutorCompletionService< V >
ExecutorCompletionService(Executor e)
构建一个执行器完成服务来收集给定执行器的结果
Future< V> submit(Callable< V> task)
Future< V> submit(Runnable task, V result)
提交一个任务给底层的执行器
Future< V> take()
移除下一个已完成的结果，如果没有任何已完成的结果可用则阻塞
Future< V> poll()
Future< V> poll(long time, TimeUnit unit)
移除下一个已完成的结果，如果没有已完成结果则返回null。第二个方法将等待给定的时间
