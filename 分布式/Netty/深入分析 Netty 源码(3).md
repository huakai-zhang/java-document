## EventLoop


一个Netty 程序启动时，至少要指定一个EventLoopGroup(如果使用到的是NIO, 那么通常是NioEventlLoopGroup)，那么这个NioEventLoopGroup 在Netty中到底扮演着什么角色呢?我们知道，Netty是Reactor模型的一个实现，那么首先从Reactor的线程模型开始吧.

### 关于Reactor的线程模型

首先我们来看一下Reactor 的线程模型. Reactor的线程模型有三种:

1. 单线程模型 
2. 多线程模型 
3. 主从模型



首先看单线程模型： ![img](https://img-blog.csdnimg.cn/20200606185811434.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) 所谓单线程，即acceptor 处理和handler 处理都在一个线程中处理.这个模型的坏处显而易见:当其中某个handler 阻塞时，会导致其他所有的client 的handler 都得不到执行，并且更严重的是，handler 的阻塞也会导致整个服务不能接收新的client 请求(因为acceptor也被阻塞了).因为有这么多的缺陷，因此单线程Reactor模型用的比较少. 那么什么是多线程模型呢? Reactor的多线程模型与单线程模型的区别就是acceptor 是一个单独的线程处理，并且有一组特定的NIO线程来负责各个客户端连接的IO操作. Reactor 多线程模型如下: ![img](https://img-blog.csdnimg.cn/2020060618595897.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) Reactor多线程模型有如下特点:

1. 有专门一个线程，即Acceptor 线程用于监听客户端的TCP连接请求. 
2. 客户端连接的IO操作都是由一个特定的NIO线程池负责.每个客户端连接都与一个特定的NIO线程绑定，因此在这个客户端连接中的所有IO操作都是在同一个线程中完成的. 
3. 客户端连接有很多，但是NIO线程数是比较少的，因此一个NIO 线程可以同时绑定到多个客户端连接中.


接下来我们再来看一下Reactor的主从多线程模型. 一般情况下，Reactor 的多线程模式已经可以很好的工作了，但是我们考虑一下如下情况:如果我们的服务器需要同时处理大量的客户端连接请求或我们需要在客户端连接时，进行一些权限的检查，那么单线程的Acceptor 很有可能就处理不过来，造成了大量的客户端不能连接到服务器. Reactor的主从多线程模型就是在这样的情况下提出来的，它的特点是:服务器端接收客户端的连接请求不再是一个线程，而是由一个独立的线程池组成.它的线程模型如下: ![img](https://img-blog.csdnimg.cn/20200606190336591.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) 可以看到，Reactor 的主从多线程模型和Reactor多线程模型很类似，只不过Reactor 的主从多线程模型的acceptor 使用了线程池来处理大量的客户端请求.

### NioEventLoopGroup 与Reactor 线程模型的对应

我们介绍了三种Reactor 的线程模型，那么它们和NioEventLoopGroup 又有什么关系呢?其实，不同的设置NioEventLoopGroup的方式就对应了不同的Reactor 的线程模型. **单线程模型** 来看一下下面的例子:

```java
EventLoopGroup bossGroup = new NioEventLoopGroup(1);
ServerBootstrap server = new ServerBootstrap();
server.group( bossGroup);
```

注意，我们实例化了一个NioEventLoopGroup， 然后接着我们调用server.group (bossGroup)设置了服务器端的EventLoopGroup.有人可能会有疑惑:我记得在启动服务器端的Netty 程序时，是需要设置bossGroup 和workerGroup 的，为什么这里就只有一个bossGroup?其实很简单，ServerBootstrap重写了group方法:

```java
public ServerBootstrap group(EventLoopGroup group) {
    return group(group, group);
}
```

因此当传入一个group 时，那么bossGroup和workerGroup 就是同一个NioEventLoopGroup了. 这时候呢，因为bossGroup 和workerGroup就是同一个NioEventLoopGroup， 并且这个NioEventLoopGroup只有一个线程，这样就会导致Netty中的acceptor 和后续的所有客户端连接的IO操作都是在一个线程中处理的.那么对应到Reactor 的线程模型中，我们这样设置NioEventLoopGroup时，就相当于Reactor 单线程模型. **多线程模型** 同理，再来看一下下面的例子:

```java
EventLoopGroup bossGroup = new NioEventLoopGroup(128) ;
ServerBootstrap server = new ServerBootstrap();
server.group(bossGroup);
```

将bossGroup的参数就设置为大于1的数,其实就是Reactor 多线程模型. **主从线程模型** 实现主从线程模型的例子如下:

```java
EventLoopGroup bossGroup = new NioEventLoopGroup();
EventLoopGroup workerGroup = new NioEventLoopGroup();
ServerBootstrap server = new ServerBootstrap();
server.group(bossGroup, workerGroup)
```

bossGroup为主线程，而workerGroup中的线程是CPU 核心数乘以2，因此对应的到Reactor线程模型中，我们知道，这样设置的NioEventLoopGroup其实就是Reactor 主从多线程模型.

### NioEventLoopGroup 类层次结构


![img](https://img-blog.csdnimg.cn/2020060619291521.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

### NioEventLoopGroup 实例化过程


![img](https://img-blog.csdnimg.cn/2020060619513977.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) 即:

1. EventLoopGroup(其实是MultithreadEventExecutorGroup) 内部维护一个类型为EventExecutorchildren数组，其大小是nThreads， 这样就构成了一个线程池 
2. 如果我们在实例化NioEventLoopGroup时，如果指定线程池大小，则nThreads 就是指定的值，反之是处理器核心数*2 
3. MultithreadEventExecutorGroup中会调用newChild 抽象方法来初始化children数组 
4. 抽象方法newChild 是在NioEventLoopGroup 中实现的，它返回一个NioEventLoop 实例 
5. NioEventLoop 属性: SelectorProvider provider 属性:NioEventLoopGroup 构造器通过SelectorProvider. provider() 获取一个 SelectorProvider Selector selector 属性: NioEventLoop 构造器中通过调用 selector = provider. openSelector() 获取一个 selector 对象

### NioEventLoop类层次结构


NioEventLoop继承于SingleThreadEventLoop，而SingleThreadEventLoop 又继承于SingleThreadEventExecutor. SingleThreadEventExecutor 是Netty 中对本地线程的抽象，它内部有一个Thread thread 属性，存储了一个本地Java 线程。因此我们可以认为，一个NioEventLoop其实和一个特定的线程绑定，并且在其生命周期内，绑定的线程都不会再改变. ![img](https://img-blog.csdnimg.cn/20200606200910585.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) NioEventLoop的类层次结构图还是比较复杂的，不过我们只需要关注几个重要的点即可.首先NioEventLoop的继承链如下: NioEventLoop-&gt;SingleThreadEventLoop-&gt;SingleThreadEventExecutor-&gt;AbstractScheduledEventExecutor 在AbstractScheduledEventExecutor中，Netty 实现了NioEventLoop 的schedule 功能，即我们可以通过调用一个NioEventLoop 实例的schedule 方法来运行一些定时任务。而在SingleThreadEventLoop中，又实现了任务队列的功能，通过它，我们可以调用一个NioEventLoop实例的execute 方法来向任务队列中添加一个task，并由NioEventlLoop 进行调度执行. 通常来说，NioEventLoop肩负着两种任务，第一个是作为IO线程，执行与Channel 相关的IO操作，包括调用select 等待就绪的I0事件、读写数据与数据的处理等;而第二个任务是作为任务队列，执行taskQueue 中的任务，例如用户调用eventLoop. schedule提交的定时任务也是这个线程执行的.

### NioEventLoop的实例化过程


![img](https://img-blog.csdnimg.cn/20200606201235650.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) 从上图可以看到，SingleThreadEventExecutor 有一个名为 thread 的 Thread 类型字段，这个字段就代表了与 SingleThreadEventExecutor 关联的本地线程。

```java
private void doStartThread() {
    assert thread == null;
    executor.execute(new Runnable() {
        @Override
        public void run() {
            thread = Thread.currentThread();
            if (interrupted) {
                thread.interrupt();
            }
            boolean success = false;
            updateLastExecutionTime();
            try {
                SingleThreadEventExecutor.this.run();
                success = true;
            } catch (Throwable t) {
                logger.warn("Unexpected exception from an event executor: ", t);
            } finally {
                for (;;) {
                    int oldState = STATE_UPDATER.get(SingleThreadEventExecutor.this);
                    if (oldState >= ST_SHUTTING_DOWN || STATE_UPDATER.compareAndSet(
                            SingleThreadEventExecutor.this, oldState, ST_SHUTTING_DOWN)) {
                        break;
                    }
                }
                // Check if confirmShutdown() was called at the end of the loop.
                if (success && gracefulShutdownStartTime == 0) {
                    logger.error("Buggy " + EventExecutor.class.getSimpleName() + " implementation; " +
                            SingleThreadEventExecutor.class.getSimpleName() + ".confirmShutdown() must be called " +
                            "before run() implementation terminates.");
                }
                try {
                    // Run all remaining tasks and shutdown hooks.
                    for (;;) {
                        if (confirmShutdown()) {
                            break;
                        }
                    }
                } finally {
                    try {
                        cleanup();
                    } finally {
                        STATE_UPDATER.set(SingleThreadEventExecutor.this, ST_TERMINATED);
                        threadLock.release();
                        if (!taskQueue.isEmpty()) {
                            logger.warn(
                                    "An event executor terminated with " +
                                            "non-empty task queue (" + taskQueue.size() + ')');
                        }
                        terminationFuture.setSuccess(null);
                    }
                }
            }
        }
    });
}
```

### EventLoop 与 Channel 的关联


Netty 中，每个 Channel 都有且仅有一个 EventLoop 与之关联，它们的关联过程如下： ![img](https://img-blog.csdnimg.cn/20200606205206757.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) 从上图中我们可以看到，当调用了AbstractChannel$AbstractUnsafe.register后，就完成了Channel和EventLoop 的关联. register 实现如下:

```java
public final void register(EventLoop eventLoop, final ChannelPromise promise) {
    ...
    AbstractChannel.this.eventLoop = eventLoop;
    if (eventLoop.inEventLoop()) {
        register0(promise);
    } else {
        try {
            eventLoop.execute(new Runnable() {
                @Override
                public void run() {
                    register0(promise);
                }
            });
        } catch (Throwable t) {
            ...
        }
    }
}
```

在 AbstractChannel¥AbstractUnsafe.register中，会将一个EventLoop赋值给AbstractChannel 内部的 eventLoop 字段，到这里就完成了 EventLoop 与 Channel 的关联过程。

### EventLoop 的启动

在前面我们已经知道了，NioEventLoop本身就是一个SingleThreadEventExecutor， 因此NioEventLoop的启动，其实就是NioEventLoop所绑定的本地Java线程的启动. 依照这个思想，我们只要找到在哪里调用了SingleThreadEventExecutor 的thread 字段的start()方法就可以知道是在哪里启动的这个线程了. 从代码中搜索，thread.start()被封装到SingleThreadEventExecutor.startThread() 方法中了:

```java
private void startThread() {
    if (STATE_UPDATER.get(this) == ST_NOT_STARTED) {
        if (STATE_UPDATER.compareAndSet(this, ST_NOT_STARTED, ST_STARTED)) {
            doStartThread();
        }
    }
}
```

STATE_ UPDATER是SingleThreadEventExecutor内部维护的一个属性，它的作用是标识当前的thread的状态，在初始的时候，STATE_UPDATER == ST_NOT_STARTED，因此第一次调用startThread()方法时，就会进入到if语句内，进而调用到thread.start(). 而这个关键的startThread() 方法又是在哪里调用的呢?经过方法调用关系搜索，我们发现，startThread是在SingleThreadEventExecutor.exqcute方法中调用的:

```java
public void execute(Runnable task) {
    if (task == null) {
        throw new NullPointerException("task");
    }
    boolean inEventLoop = inEventLoop();
    if (inEventLoop) {
        addTask(task);
    } else {
    	// 调用 startThread 方法，启动EventLoop线程
        startThread();
        addTask(task);
        if (isShutdown() && removeTask(task)) {
            reject();
        }
    }
    if (!addTaskWakesUp && wakesUpForTask(task)) {
        wakeup(inEventLoop);
    }
}
```

既然如此，那现在我们的工作就变为了寻找在哪里第一次调用了SingleThreadEventExecutor. execute()方法. 在前面EventLoop 与Channel 的关联这一小节时，有提到到在注册channel 的过程中，会在AbstractChannelSAbstractUnsafe.register中调用eventlLoop.execute方法，在EventLoop 中进行Channel注册代码的执行，AbstractChannel$AbstractUnsafe.register部分代码如下:

```java
public final void register(EventLoop eventLoop, final ChannelPromise promise) {
    ...
    AbstractChannel.this.eventLoop = eventLoop;
    if (eventLoop.inEventLoop()) {
        register0(promise);
    } else {
        try {
            eventLoop.execute(new Runnable() {
                @Override
                public void run() {
                    register0(promise);
                }
            });
        } catch (Throwable t) {
            ...
        }
    }
}
```

很显然，一路从Bootstrap.bind方法跟踪到AbstractChannel$AbstractUnsafe.register 方法，整个代码都是在主线程中运行的，因此上面的eventLoop.inEventLoop()就为false, 于是进入到else分支，在这个分支中调用了eventLoop.execute.eventLoop , 是一个NioEventLoop 的实例，而NioEventLoop 没有实现execute方法，因此调用的是SingleThreadEventExecutor.execute。 我们已经分析过了，inEventLoop == false， 因此执行到else分支，在这里就调用了startThread()方法来启动SingleThreadEventExecutor内部关联的Java 本地线程了. 总结一句话，当EventLoop.execute第一次被调用时，就会触发startThread() 的调用，进而导致了EventLoop 所对应的Java线程的启动.


java.util.concurrent.Future是Java提供的接口，表示异步执行的状态，Future 的get方法会判断任务是否执行完成，如果完成就返回结果，否则阻塞线程，直到任务完成。 Netty扩展了Java的Future,最主要的改进就是增加了监听器Listener接口,通过监听器可以让异步执行更加有效率，不需要通过get来等待异步执行结束，而是通过监听器回调来精确地控制异步执行结束的时间点。

```java
public interface Future<V> extends java.util.concurrent.Future<V> {
    boolean isSuccess();
    boolean isCancellable();
    Throwable cause();
    Future<V> addListener(GenericFutureListener<? extends Future<? super V>> listener);
    Future<V> addListeners(GenericFutureListener<? extends Future<? super V>>... listeners);
    Future<V> removeListener(GenericFutureListener<? extends Future<? super V>> listener);
    Future<V> removeListeners(GenericFutureListener<? extends Future<? super V>>... listeners);
    Future<V> sync() throws InterruptedException;
    Future<V> syncUninterruptibly();
    Future<V> await() throws InterruptedException;
    Future<V> awaitUninterruptibly();
    boolean await(long timeout, TimeUnit unit) throws InterruptedException;
    boolean await(long timeoutMillis) throws InterruptedException;
    boolean awaitUninterruptibly(long timeout, TimeUnit unit);
    boolean awaitUninterruptibly(long timeoutMillis);
    V getNow();

    @Override
    boolean cancel(boolean mayInterruptIfRunning);
}
```

ChannelFuture 接口扩展了Netty 的Future接口，表示一种没有返回值的异步调用，同时关联了 Channel，跟一个 Channel 绑定：

```java
public interface ChannelFuture extends Future<Void> {
    Channel channel();
    ChannelFuture addListener(GenericFutureListener<? extends Future<? super Void>> listener);
    ChannelFuture addListeners(GenericFutureListener<? extends Future<? super Void>>... listeners);
    ChannelFuture removeListener(GenericFutureListener<? extends Future<? super Void>> listener);
    ChannelFuture removeListeners(GenericFutureListener<? extends Future<? super Void>>... listeners);
    ChannelFuture sync() throws InterruptedException;
    ChannelFuture syncUninterruptibly();
    ChannelFuture await() throws InterruptedException;
    ChannelFuture awaitUninterruptibly();
    boolean isVoid();
}
```

Promise 接口也扩展了Future接口，它表示一种可写的Future，就是可以设置异步执行的结果：

```java
public interface Promise<V> extends Future<V> {
    Promise<V> setSuccess(V result);
    boolean trySuccess(V result);
    Promise<V> setFailure(Throwable cause);
    boolean tryFailure(Throwable cause);
    boolean setUncancellable();
    Promise<V> addListener(GenericFutureListener<? extends Future<? super V>> listener);
    Promise<V> addListeners(GenericFutureListener<? extends Future<? super V>>... listeners);
    Promise<V> removeListener(GenericFutureListener<? extends Future<? super V>> listener);
    Promise<V> removeListeners(GenericFutureListener<? extends Future<? super V>>... listeners);
    Promise<V> await() throws InterruptedException;
    Promise<V> awaitUninterruptibly();
    Promise<V> sync() throws InterruptedException;
    Promise<V> syncUninterruptibly();
}
```

ChannelPromise接口扩展了Promise和ChannelFuture，绑定了Channel，又可写异步执行结构，又具备了监听者的功能，是Netty实际编程使用的表示异步执行的接口：

```java
public interface ChannelPromise extends ChannelFuture, Promise<Void> {
    Channel channel();
    ChannelPromise setSuccess(Void result);
    ChannelPromise setSuccess();
    boolean trySuccess();
    ChannelPromise setFailure(Throwable cause);
    ChannelPromise addListener(GenericFutureListener<? extends Future<? super Void>> listener);
    ChannelPromise addListeners(GenericFutureListener<? extends Future<? super Void>>... listeners);
    ChannelPromise removeListener(GenericFutureListener<? extends Future<? super Void>> listener);
    ChannelPromise removeListeners(GenericFutureListener<? extends Future<? super Void>>... listeners);
    ChannelPromise sync() throws InterruptedException;
    ChannelPromise syncUninterruptibly();
    ChannelPromise await() throws InterruptedException;
    ChannelPromise awaitUninterruptibly();
    ChannelPromise unvoid();
}
```

DefaultChannelPromise是ChannelPromise的实现类，它是实际运行时的Promoise实例。 Netty使用addListener的方式来回调异步执行的结果。 看一下DefaultPromise的addListener方法，它判断异步任务执行的状态，如果执行完成，就理解通知监听者，否则加入到监听者队列通知监听者就是找一个线程来执行调用监听的回调函数。

```java
public Promise<V> addListener(GenericFutureListener<? extends Future<? super V>> listener) {
    checkNotNull(listener, "listener");
    synchronized (this) {
        addListener0(listener);
    }
    if (isDone()) {
        notifyListeners();
    }
    return this;
}
private void addListener0(GenericFutureListener<? extends Future<? super V>> listener) {
    if (listeners == null) {
        listeners = listener;
    } else if (listeners instanceof DefaultFutureListeners) {
        ((DefaultFutureListeners) listeners).add(listener);
    } else {
        listeners = new DefaultFutureListeners((GenericFutureListener<? extends Future<V>>) listeners, listener);
    }
}
private void notifyListeners() {
    EventExecutor executor = executor();
    if (executor.inEventLoop()) {
        final InternalThreadLocalMap threadLocals = InternalThreadLocalMap.get();
        final int stackDepth = threadLocals.futureListenerStackDepth();
        if (stackDepth < MAX_LISTENER_STACK_DEPTH) {
            threadLocals.setFutureListenerStackDepth(stackDepth + 1);
            try {
                notifyListenersNow();
            } finally {
                threadLocals.setFutureListenerStackDepth(stackDepth);
            }
            return;
        }
    }
    safeExecute(executor, new Runnable() {
        @Override
        public void run() {
            notifyListenersNow();
        }
    });
}
```

再来看监听者接口，就是个方法，即等异步任务执行完后，拿到Future结果，执行回调逻辑：

```java
public interface GenericFutureListener<F extends Future<?>> extends EventListener {
    void operationComplete(F future) throws Exception;
}
```


## ChannelHandlerContext


每个 ChannelHandler 被添加到 ChannelPipeline 后, 都会创建一个 ChannelHandlerContext 并与之创建的ChannelHandler关联绑定。ChannelHandlerContext允许ChannelHandler与其他的ChannelHandler实现进行交互。ChannelHandlerContext不会改变添加到其中的ChannelHandler，因此它是安全的。 下图显示 ChannelHandlerContext，ChannelHandler，ChannelPipeline 的关系: ![img](https://img-blog.csdnimg.cn/20200606220332446.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

## Channel 的状态模型

Netty有一个简单但强大的状态模型，并完美映射到Channel InboundHandler的各个方法。下面是Channel生命周期四个不同的状态:

1. channelUnregistered 
2. channelRegistered 
3. channelActive 
4. channelInactive


Channel 的状态在其生命周期中变化，因为状态变化需要触发，下图显示了Channel状态变化： ![img](https://img-blog.csdnimg.cn/20200606220456510.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

## ChannelHandler 和其子类


先看一张 Handler 的类继承图 ![img](https://img-blog.csdnimg.cn/20200607135555577.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

## ChannelHandler中的方法

Netty定义了良好的类型层次结构来表示不同的处理程序类型，所有的类型的父类是ChanneHandler。ChannelHandler 提供了在其生命周期内添加或从ChannelPipeline中删除的方法。

1. handlerAdded, ChannelHandler 添加到实际，上下文中准备处理事件 
2. handlerRemoved, 将ChannelHandler从实际上下文中删除，不再处理事件 
3. exceptionCaught, 处理抛出的异常

Netty还提供了一个实现了ChannelHandler的抽象类ChannelHandlerAdapter。ChannelHandlerAdapter 实现了父类的所有方法, 基本上就是传递事件到 ChannelPipeline 中的下一个 ChannelHandler 直到结束。也可以直接继承于 ChannelHandlerAdapter, 然后重写里面的方法。

## ChannelInboundHandler

ChannelInboundHandler 提供了一些方法再接收数据或 Channel 状态改变时被调用. 下面是 ChannelInboundHandler 的一些方法:

1. channelRegistered, ChannelHandlerContext 的 Channel 被注册到 EventLoop 
2. channelUnregistered, ChannelHandlerContext 的 Channel 从 EventLoop 中注销 
3. channelActive, ChannelHandlerContext 的 Channel 已激活 
4. channelInactive，ChannelHanderContext 的 Channel 结束生命周期 
5. channelRead, 从当前 Channel 的对端读取消息 
6. channelReadComplete, 消息读取完成后执行 
7. userEventTriggered, 一个用户事件被触发 
8. channelWritabilityChanged, 改变通道的可写状态，可以使用 Channel.isWritable() 检查 
9. exceptionCaught, 重写父类 ChannelHandler 的方法，处理异常

Netty提供了一个实现了ChannelInboundHandler接口并继承ChannelHandlerAdapter的类:ChannelInboundHandlerAdapter ChannelInboundHandlerAdapter 实现了 ChannelInboundHandler 的所有方法,作用就是处理消息并将消息转发到 ChannelPipeline 中的下一个 ChannelHandler. ChannelInboundHandlerAdapter 的 channelRead 方法处理完消息后会自动释放消息，若想自动释放收到的消息，可以使用SimpleChannelInboundHandler.看下面的代码:

```java
public class UnreleaseHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        // 手动释放消息
        ReferenceCountUtil.release(msg);
    }
}
```

SimpleChannelInboundHandler 会自动释放消息：

```java
public class ReleaseHandler extends SimpleChannelInboundHandler<Object> {
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, Object msg) throws Exception {
        // 不需要手动释放
    }
}
```

ChannelInitializer 用来初始化 ChannelHandler，将自定义的各种 ChannelHandler 添加到 ChannelPipeline 中。


## TCP 黏包/拆包

TCP是一个“流”协议，所谓流，就是没有界限的一长串二进制数据。TCP作为传输层协议并不不了解上层业务数据的具体含义，它会根据TCP缓冲区的实际情况进行数据包的划分，所以在业务上认为是一个完整的包，可能会被TCP拆分成多个包进行发送，也有可能把多个小的包封装成一个大的数据包发送，这就是所谓的TCP粘包和拆包问题。

## 粘包问题的解决策略

由于底层的TCP无法理解上层的业务数据，所以在底层是无法保证数据包不被拆分和重组的，这个问题只能通过。上层的应用协议栈设计来解决。业界的主流协议的解决方案，可以归纳如下:

1. 消息定长，报文大小固定长度，例如每个报文的长度固定为200字节，如果不够空位补空格 
2. 包尾添加特殊分隔符，例如每条报文结束都添加回车换行符(例如FTP协议)或者指定特殊字符作为报文分隔符，接收方通过特殊分隔符切分报文区分 
3. 将消息分为消息头和消息体，消息头中包含表示信息的总长度(或者消息体长度)的字段 
4. 更复杂的自定义应用层协议。

## 编、解码技术

通常我们也习惯将编码(Encode) 称为序列化(serialization) ，它将对象序列化为字节数组，用于网络传输、数据持久化或者其它用途。 反之，解码(Decode) /反序列化(deserialization) 把从网络、磁盘等读取的字节数组还原成原始对象(通常是原始对象的拷贝)，以方便后续的业务逻辑操作。进行远程跨进程服务调用时(例如RPC调用)，需要使用特定的编解码技术，对需要进行网络传输的对象做编码或者解码，以便完成远程调用。

## Netty为什么要提供编解码框架


作为一个高性能的异步、NIO通信框架，编解码框架是Netty的重要组成部分。尽管站在微内核的角度看，编解码框架并不是Netty微内核的组成部分，但是通过ChannelHandler定制扩展出的编解码框架却是不可或缺的。 然而，我们已经知道在Netty中，从网络读取的Inbound消息，需要经过解码，将二进制的数据报转换成应用层协议消息或者业务消息，才能够被上层的应用逻辑识别和处理;同理，用户发送到网络的Outbound业务消息，需要经过编码转换成二进制字节数组(对于Netty就是ByteBuf)才能够发送到网络对端。编码和解码功能是NIO框架的有机组成部分，无论是由业务定制扩展实现，还是NIO框架内置编解码能力，该功能是必不可少的。 为了降低用户的开发难度，Netty对常用的功能和API做了装饰，以屏蔽底层的实现细节。编解码功能的定制，对于熟悉Netty底层实现的开发者而言，直接基于ChannelHandler扩展开发，难度并不是很大。但是对于大多数初学者或者不愿意去了解底层实现细节的用户，需要提供给他们更简单的类库和API,而不是ChannelHandler. Netty在这方面做得非常出色，针对编解码功能，它既提供了通用的编解码框架供用户扩展，又提供了常用的编解码类库供用户直接使用。在保证定制扩展性的基础之上，尽量降低用户的开发工作量和开发门槛，提升开发效率。 Netty预置的编解码功能列表如下: base64、 Protobuf、 JBoss Marshalling、spdy 等。 ![img](https://img-blog.csdnimg.cn/20200607143954203.png)

## Netty粘包和拆包解决方案

### Netty中常用的解码器

Netty提供了多个解码器，可以进行分包的操作，分别是:

- LineBasedFrameDecoder 
- DelimiterBasedFrameDecoder (添加特殊分隔符报文来分包) 
- FixedLengthFrameDecoder (使用定长的报文来分包) 
- LengthFieldBasedFrameDecoder

#### LineBasedFrameDecoder解码器

LineBasedFrameDecoder是回车换行解码器，如果用户发送的消息以回车换行符作为消息结束的标识,则可以直接使用Netty的LineBasedFrameDecoder对消息进行解码,只需要在初始化Netty服务端或者客户端时将LineBasedFrameDecoder正确的添加到ChannelPipeline中即可,不需要自己重新实现一套换行解码器。 LineBasedFrameDecoder的工作原理是它依次遍历ByteBuf中的可读字节，判断看是否有“\n”或者“\r\n”，如果有，就以此位置为结束位置，从可读索引到结束位置区间的字节就组成了一行。它是以换行符为结束标志的解码器，支持携带结束符或者不携带结束符两种解码方式，同时支持配置单行的最大长度。如果连续读取到最大长度后仍然没有发现换行符，就会抛出异常，同时忽略掉之前读到的异常码流。防止由于数据报没有携带换行符导致接收到ByteBuf无限制积压，引起系统内存溢出。 它的使用效果如下：

```java
# 解码之前：
+------------------------------------------------------------------+
                        接收到的数据报
“This is a netty example for using the nio framework.\r\n When you“
+------------------------------------------------------------------+
# 解码之后的ChannelHandler接收到的Object如下：
+------------------------------------------------------------------+
                        解码之后的文本消息
“This is a netty example for using the nio framework.“
+------------------------------------------------------------------+
```

通常情况下，LineBasedFrameDecoder会和StringDecoder配合使用， 组合成按行切换的文本解码器，对于文本类协议的解析，文本换行解码器非常实用，例如对HTTP消息头的解析、FTP协议消息的解析等。 下面我们简单给出文本换行解码器的使用示例:

```java
@Override
public void initChannel(SocketChannel ch) throws Exception {
    ch.pipeline().addLast(new LineBasedFrameDecoder(1024));
    ch.pipeline().addLast(new StringDecoder());
    ch.pipeline().addLast(new MyHandler());
}
```

初始化Channel的时候，首先将LineBasedFrameDecoder添加到ChannelPipeline中，然后再依次添加字符串解码器StringDecoder，业务Handler。

### DelimiterBasedFrameDecoder解码器

DelimiterBasedFrameDecoder是分隔符解码器，用户可以指定消息结束的分隔符，它可以自动完成以分隔符作为码流结束标识的消息的解码。回车换行解码器实际上是一种特殊的DelimiterBasedFrameDecoder解码器。 分隔符解码器在实际工作中也有很广泛的应用，很多简单的文本私有协议，都是以特殊的分隔符作为消息结束的标识，特别是对于那些使用长连接的基于文本的私有协议。分隔符的指定:与大家的习惯不同，分隔符并非以char或者string作为构造参数，而是ByteBuf,下面我们就结合实际例子给出它的用法。假如消息以“$_”作为分隔符，服务端或者客户端初始化ChannelPipeline的代码实例如下:

```java
@Override
public void initChannel(SocketChannel ch) throws Exception {
    ByteBuf delimiter = Unpooled.copiedBuffer("$_".getBytes());
    ch.pipeline().addLast(new DelimiterBasedFrameDecoder(1024, delimiter));
    ch.pipeline().addLast(new StringDecoder());
    ch.pipeline().addLast(new MyHandler());
}
```

首先将“$_”转换成ByteBuf对象，作为参数构造DelimiterBasedFrameDecoder,将其添加到ChannelPipeline中，然后依次添加字符串解码器(通 常用于文本解码)和用户Handler,请注意解码器和Handler的添加顺序，如果顺序颠倒，会导致消息解码失败。 DelimiterBasedFrameDecoder原理分析:解码时，判断当前已经读取的ByteBuf中是否包含分隔符ByteBuf,如果包含，则截取对应的ByteBuf返回，源码如下:

```java
protected Object decode(ChannelHandlerContext ctx, ByteBuf buffer) throws Exception {
    if (lineBasedDecoder != null) {
        return lineBasedDecoder.decode(ctx, buffer);
    }
    // Try all delimiters and choose the delimiter which yields the shortest frame.
    int minFrameLength = Integer.MAX_VALUE;
    ByteBuf minDelim = null;
    for (ByteBuf delim: delimiters) {
        int frameLength = indexOf(buffer, delim);
        if (frameLength >= 0 && frameLength < minFrameLength) {
            minFrameLength = frameLength;
            minDelim = delim;
        }
    }
    ...
}
```

详细分析下indexOf(buffer, delim)方法的实现，代码如下：

```java
private static int indexOf(ByteBuf haystack, ByteBuf needle) {
    for (int i = haystack.readerIndex(); i < haystack.writerIndex(); i ++) {
        int haystackIndex = i;
        int needleIndex;
        for (needleIndex = 0; needleIndex < needle.capacity(); needleIndex ++) {
            if (haystack.getByte(haystackIndex) != needle.getByte(needleIndex)) {
                break;
            } else {
                haystackIndex ++;
                if (haystackIndex == haystack.writerIndex() &&
                    needleIndex != needle.capacity() - 1) {
                    return -1;
                }
            }
        }
        if (needleIndex == needle.capacity()) {
            // Found the needle from the haystack!
            return i - haystack.readerIndex();
        }
    }
    return -1;
}
```

该算法与Java String中的搜索算法类似，对于原字符串使用两个指针来进行搜索，如果搜索成功，则返回索引位置，否则返回-1。

### FixedLengthFrameDecoder解码器

FixedLengthFrameDecoder是固定长度解码器，它能够按照指定的长度对消息进行自动解码，开发者不需要考虑TCP的粘包/拆包等问题，非常实用。 对于定长消息，如果消息实际长度小于定长，则往往会进行补位操作，它在一定程度上导致了空间和资源的浪费。但是它的优点也是非常明显的，编解码比较简单，因此在实际项目中仍然有一定的应用场景。 利用FixedLengthFrameDecoder解码器，无论一次接收到多少数据报， 它都会按照构造函数中设置的固定长度进行解码，如果是半包消息，FixedLengthFrameDecoder会缓存半包消息并等待下个包到达后进行拼包，直到读取到一个完整的包。假如单条消息的长度是20字节，使用FixedLengthFrameDecoder解码器的效果如下:

```java
# 解码之前：
+------------------------------------------------------------------+
                        接收到的数据报
“HELLO NETTY FOR USER DEVELOPER“
+------------------------------------------------------------------+
# 解码后：
+------------------------------------------------------------------+
                        解码之后的文本消息
“HELLO NETTY FOR USER“
+------------------------------------------------------------------+
```

### LengthFieldBasedFrameDecoder解码器

了解TCP通信机制的读者应该都知道TCP底层的粘包和拆包，当我们在接收消息的时候，显示不能认为读取到的报文就是个整包消息，特别是对于采用非阻塞I/O和长连接通信的程序。 如何区分一个整包消息，通常有如下4种做法:

1. 固定长度，例如每120个字节代表一个整包消息，不足的前面补位。解码器在处理这类定常消息的时候比较简单，每次读到指定长度的字节后再进行解码 
2. 通过回车换行符区分消息，例如HTTP协议。 这类区分消息的方式多用于文本协议 
3. 通过特定的分隔符区分整包消息 
4. 通过在协议头/消息头中设置长度字段来标识整包消息。

前三种解码器之前的章节已经做了详细介绍，下面让我们来一起学习最后一种通用解码器LengthFieldBasedFrameDecoder. 大多数的协议(私有或者公有)，协议头中会携带长度字段，用于标识消息体或者整包消息的长度，例如SMPP、 HTTP协议等。由于基于长度解码需求的通用性，以及为了降低用户的协议开发难度，Netty提供 了LengthFieldBasedFrameDecoder,自动屏蔽TCP底层的拆包和粘包问题，只需要传入正确的参数，即可轻松解决“读半包“问题。 下面我们看看如何通过参数组合的不同来实现不同的“半包”读取策略。第一种常用的方式是消息的第一一个字段是长度字段，后面是消息体，消息头中只包含一个长度字段。它的消息结构定义如下所示:

```java
# 解码前的字节缓冲区（14字节）
+--------+----------------+
| Length | Actual Conyent |
| 0x000C | "HELLO, WORLD" |
+--------+----------------+
```

使用以下参数组合进行解码:

1. lengthFieldOffset = 0 
2. lengthFieldLength = 2 
3. lengthAdjustment = 0 
4. initialBytesToStrip= 0

解码后的字节缓冲区内容如下所示:

```java
# 解码后的字节缓冲区（14字节）
+--------+----------------+
| Length | Actual Conyent |
| 0x000C | "HELLO, WORLD" |
+--------+----------------+
```

通过ByteBuf.readableBytes(方法我们可以获取当前消息的长度，所以解码后的字节缓冲区可以不携带长度字段，由于长度字段在起始位置并且长度为2，所以将initialBytesToStrip设置为2，参数组合修改为:

1. lengthFieldOffset = 0 
2. lengthFieldLength = 2 
3. lengthAdjustment = 0 
4. initialBytesToStrip= 2 解码后的字节缓冲区内容如下所示:

```java
# 跳过长度字段解码后的字节缓冲区（12字节）
+----------------+
| Actual Conyent |
| "HELLO, WORLD" |
+----------------+
```

解码后的字节缓冲区丢弃了长度字段，仅仅包含消息体，对于大多数的协议，解码之后消息长度没有用处，因此可以丢弃。 在大多数的应用场景中，长度字段仅用来标识消息体的长度，这类协议通常由消息长度字段+消息体组成，如. 上图所示的几个例子。但是，对于某些协议，长度字段还包含了消息头的长度。在这种应用场景中，往往需要使用lengthAdjustment进行修正。由于整个消息(包含消息头)的长度往往大于消息体的长度，所以，lengthAdjustment为负数。图2-6展示了通过指定lengthAdjustment字段来包含消息头的长度:

1. lengthFieldOffset = 0 
2. lengthFieldLength = 2 
3. lengthAdjustment = -2 
4. initialBytesToStrip = 0

```java
# 解码之前的码流,包含长度字段自身的码流
+--------+----------------+
| Length | Actual Conyent |
| 0x000E | "HELLO, WORLD" |
+--------+----------------+
# 解码之后的码流
+--------+----------------+
| Length | Actual Conyent |
| 0x000E | "HELLO, WORLD" |
+--------+----------------+
```

由于协议种类繁多，并不是所有的协议都将长度字段放在消息头的首位，当标识消息长度的字段位于消息头的中间或者尾部时，需要使用lengthFieldOffset字段进行标识，下面的参数组合给出了如何解决消息长度字段不在首位的问题:

1. lengthFieldOffset = 2 
2. lengthFieldLength= 3 
3. lengthAdjustment = 0; 
4. initialBytesToStrip= 0 其中lengthFieldOffset表示长度字段在消息头中偏移的字节数，lengthFieldLength表示长度字段自身的长度，解码效果如下:

```java
# 解码之前，长度字段偏移的原始码流
+----------+----------+----------------+
| Header 1 |  Length  | Actual Conyent |
|  0xCAFE  | 0x00000E | "HELLO, WORLD" |
+----------+----------+----------------+
# 解码之后，长度字段偏移解码后的码流
+----------+----------+----------------+
| Header 1 |  Length  | Actual Conyent |
|  0xCAFE  | 0x00000E | "HELLO, WORLD" |
+----------+----------+----------------+
```

由于消息头1的长度为2，所以长度字段的偏移量为2;消息长度字段Length为3，所以lengthFieldL ength值为3。由于长度字段仅仅标识消息体的长度，所以lengthAdjustment和initialBytes’ ToStrip都为0。 最后一种场景是长度字段夹在两个消息头之间或者长度字段位于消息头的中间，前后都有其它消息头字段，在这种场景下如果想忽略长度字段以及其前面的其它消息头字段，则可以通过initialBytes ToStrip参数来跳过要忽略的字节长度，它的组合配置示意如下:

1. lengthFieldOffset= 1 
2. lengthFieldLength = 2 
3. lengthAdjustment= 1 
4. initialBytesToStrip= 3 解码之前的码流(16字节) :

```java
# 长度字段夹在消息头中间的原始码流（16字节）
+------+--------+------+----------------+
| HDR1 | Length | HDE2 | Actual Conyent |
| 0xCA | 0x000C | 0XFE | "HELLO, WORLD" |
+------+--------+------+----------------+
# 解码之后,13字节
+------+----------------+
| HDE2 | Actual Conyent |
| 0XFE | "HELLO, WORLD" |
+------+----------------+
```

由于HDR1的长度为1，所以长度字段的偏移量lengthFieldOffset为1;长度字段为2个字节，所以lengthFieldL ength为2。由于长度字段是消息体的长度，解码后如果携带消息头中的字段，则需要使用lengthAdjustment进行调整，此处它的值为1,代表的是HDR2的长度，最后由于解码后的缓冲区要忽略长度字段和HDR1部分，所以lengthAdjustment为3。解码后的结果为13个字节，HDR1和Length字段被忽略。 事实上，通过4个参数的不同组合，可以达到不同的解码效果，用户在使用过程中可以根据业务的实际情况进行灵活调整。 由于TCP存在粘包和组包问题，所以通常情况下用户需要自己处理半包消息。利用LengthFieldBasedFrameDecoder解码器可以自动解决半包问题，它的习惯用法如下:

```java
ch.pipeline().addLast("freameDecoder", new LengthFieldBasedFrameDecoder(65536, 0, 2));
ch.pipeline().addLast("UserDecoder", new UserDecoder());
```

在pipeline中增加engthFieldBasedFrameDecoder解码器，指定正确的参数组合，它可以将Netty的ByteBuf解码成整 包消息，后面的用户解码器拿到的就是个完整的数据报，按照逻辑正常进行解码即可，不再需要额外考虑“读半包”问题，降低了用户的开发难度。

## 常用的编码器

Netty并没有提供与前面介绍的匹配的编码器，原因如下:

1. 4种常用的解码器本质都是解析一一个完整的数据报给后端，主要用于解决TCP底层粘包和拆包;对于编码，就是将POJO对象序列化为ByteBuf,不需要与TCP层面打交道，也就不存在半包编码问题。从应用场景和需要解决的实际问题角度看，双方是非对等的 
2. 很难抽象出合适的编码器，对于不同的用户和应用场景，序列化技术不尽相同，在Netty底层统一 抽象封装也并不合适。

Netty默认提供了丰富的编解码框架供用户集成使用，本文对较常用的Java序列化编码器进行讲解。其它的编码器，实现方式大同小异。

### ObjectEncoder编码器

ObjectEncoder是Java序列化编码器，它负责将实现Serializable接口的对象序列化为byte []，然后写入到ByteBuf中用于消息的跨网络传输。 下面我们一起分析下它的实现: 首先，我们发现它继承自MessageToByteEncoder,它的作用就是将对象编码成ByteBuf，如果要使用Java序列化，对象必须实现Serializable接口，因此，它的泛型类型为Serializable。MessageToByteEncoder的子类只需要实现encode(ChannelHandlerContext ctx,Serializable msg, ByteBuf out)方法即可，下面我们重点关注encode方法的实现:

```java
public class ObjectEncoder extends MessageToByteEncoder<Serializable> {
    private static final byte[] LENGTH_PLACEHOLDER = new byte[4];
    @Override
    protected void encode(ChannelHandlerContext ctx, Serializable msg, ByteBuf out) throws Exception {
        int startIdx = out.writerIndex();
        ByteBufOutputStream bout = new ByteBufOutputStream(out);
        bout.write(LENGTH_PLACEHOLDER);
        ObjectOutputStream oout = new CompactObjectOutputStream(bout);
        oout.writeObject(msg);
        oout.flush();
        oout.close();
        int endIdx = out.writerIndex();
        out.setInt(startIdx, endIdx - startIdx - 4);
    }
}
```

首先创建ByteBufOutputStream和0bjectOutputStream，用于将0bjec对象序列化到ByteBuf中，值得注意的是在writeObject之前需要先将长度字段(4个字节) 预留，用于后续长度字段的更新。 依次写入长度占位符(4字节) 、序列化之后的0bject对象，之后根据ByteBuf的writelndex计算序列化之后的码流长度，最后调用ByteBuf的setInt(int index, int value)更新长度占位符为实际的码流长度。有个细节需要注意，更新码流长度字段使用了setInt方法而不是writeInt，原因就是setInt方法只更新内容，并不修改readerIndex和writerIndex。


尽管Netty预置了丰富的编解码类库功能，但是在实际的业务开发过程中，总是需要对编解码功能做一些定制。使用Netty的编解码框架，可以非常方便的进行协议定制。本章节将对常用的支持定制的编解码类库进行讲解，以期让读者能够尽快熟悉和掌握编解码框架。

## 解码器

### ByteToMessageDecoder抽象解码器


使用NIO进行网络编程时，往往需要将读取到的字节数组或者字节缓冲区解码为业务可以使用的POJO对象。为了方便业务将ByteBuf解码成业务POJO对象，Netty提供了ByteToMessageDecoder抽象工具解码类。 用户自定义解码器继承Byte ToMessageDecoder,只需要实现void decode (ChannelHandler Context ctx, ByteBufin, List out)抽象方法即可完成ByteBuf到POJO对象的解码。 由于ByteToMessageDecoder并没有考虑TCP粘包和拆包等场景，用户自定义解码器需要自己处理“读半包”问题。正因为如此，大多数场景不会直接继承ByteToMessageDecoder,而是继承另外一些更 高级的解码器来屏蔽半包的处理。实际项目中，通常将L engthFieldBasedFrameDecoder和ByteToMessageDecoder组合使用，前者负责将网络读取的数据报解码为整包消息，后者负责将整包消息解码为最终的业务对象。 除了和其它解码器组合形成新的解码器之外，ByteToMessageDecoder也是很多基础解码器的父类，它的继承关系如下图所示: ![img](https://img-blog.csdnimg.cn/20200607162519341.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

### MessageToMessageDecoder抽象解码器


MessageToMessageDecoder实际上是Netty的二次解码器，它的职责是将一个对象二次解码为其它对象。 为什么称它为二次解码器呢?我们知道，从SocketChannel读取到的TCP数据报是ByteBuffer,实际就是字节数组。我们首先需要将ByteBuffer缓冲区中的数据报读取出来，并将其解码为Java对象; 然后对Java对象根据某些规则做二次解码，将其解码为另一个POJO对象。 因为MessageToMessageDecoder在Byte ToMessageDecoder之后，所以称之为二次解码器。 二次解码器在实际的商业项目中非常有用，以HTTP+XML协议栈为例，第一次解码往往是将字节数组解码成HttpRequest对象，然后对HttpRequest消 息中的消息体字符串进行二次解码，将XML 格式的字符串解码为POJO对象，这就用到了二次解码器。类似这样的场景还有很多，不再一一枚举。 事实上，做一个超级复杂的解码器将多个解码器组合成- -个大而全的MessageToMessageDecoder解码器似乎也能解决多次解码的问题，但是采用这种方式的代码可维护性会非常差。例如，如果我们打算在HTTP+XML协议栈中增加一-个打印码流的功能，即首次解码获取HttpRequest对象之后打印XML格式的码流。如果采用多个解码器组合，在中间插入一个打印消息体的Handler即可，不需要修改原有的代码;如果做一个大而全的解码器，就需要在解码的方法中增加打印码流的代码，可扩展性和可维护性都会变差。 用户的解码器只需要实现void decode(ChannelHandlerContext ctx, I msg, List&lt; Object &gt; out)抽象方法即可，由于它是将一个POJO解码为另一个POJO，所以一般不会涉及到半包的处理，相对于ByteToMessageDecoder更加简单些。它的继承关系图如下所示: ![img](https://img-blog.csdnimg.cn/20200607162813384.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

## 编码器

### MessageToByteEncoder抽象编码器

MessageToByteEncoder负责将POJO对象编码成ByteBuf,用户的编码器继承MessageToByteEncoder，实现void encode(ChannelHandlerContext ctx, | msg, ByteBuf out)接口接口，示例代码如下:

```java
public class IntegerEncoder extends MessageToByteEncoder<Integer> {
    @Override
    protected void encode(ChannelHandlerContext ctx, Integer msg, ByteBuf out) throws Exception {
     out.writeInt(msg);   
    }
}
```

它的实现原理如下:调用write操作时，首先判断当前编码器是否支持需要发送的消息，如果不支持则直接透传;如果支持则判断缓冲区的类型，对于直接内存分配ioBuffer (堆外内存)，对于堆内存通过heapBuffer方法分配，源码如下:

```java
public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
    ByteBuf buf = null;
    try {
        if (acceptOutboundMessage(msg)) {
            @SuppressWarnings("unchecked")
            I cast = (I) msg;
            buf = allocateBuffer(ctx, cast, preferDirect);
            try {
                encode(ctx, cast, buf);
            } finally {
                ReferenceCountUtil.release(cast);
            }
            if (buf.isReadable()) {
                ctx.write(buf, promise);
            } else {
                buf.release();
                ctx.write(Unpooled.EMPTY_BUFFER, promise);
            }
            buf = null;
        } else {
            ctx.write(msg, promise);
        }
    } catch (EncoderException e) {
        throw e;
    } catch (Throwable e) {
        throw new EncoderException(e);
    } finally {
        if (buf != null) {
            buf.release();
        }
    }
}
```

编码使用的缓冲区分配完成之后，调用encode抽象方 法进行编码，方法定义如下:它由子类负责具体实现。

```java
protected abstract void encode(ChannelHandlerContext ctx, I msg, ByteBuf out) throws Exception;
```

编码完成之后，调用ReferenceCountUtil的release方 法释放编码对象msg。对编码后的ByteBuf进行以下判断:

1. 如果缓冲区包含可发送的字节，则调用ChannelHandlerContext的write方法发送ByteBuf 
2. 如果缓冲区没有包含可写的字节，则需要释放编码后的ByteBuf，写入一个空的ByteBuf到ChannelHandlerContext中。

发送操作完成之后，在方法退出之前释放编码缓冲区ByteBuf对象。

### MessageToMessageEncoder抽象编码器

将一个POJO对象编码成另一个对象，以HTTP+XML协议为例， 它的一种实现方式是:先将POJO对象编码成XML字符串，再将字符串编码为HTTP请求或者应答消息。对于复杂协议，往往需要经历多次编码，为了便于功能扩展，可以通过多个编码器组合来实现相关功能。 用户的解码器继承MessageToMessageEncoder解码器，实现void encode(Channel HandlerContext ctx,I msg, List out)方法即可。注意，它与MessageToByteEncoder 的区别是输出是对象列表而不是ByteFBuf：

```java
public class IntegerToStringEncoder extends MessageToMessageEncoder<Integer> {
    @Override
    protected void encode(ChannelHandlerContext ctx, Integer msg, List<Object> out) throws Exception {
        out.add(msg.toString());
    }
}
```

MessageToMessageEncoder编码器的实现原理与之前分析的MessageToByteEncoder相似，唯一的差 别是它编码后的输出是个中间对象，并非最终可传输的ByteBuf。 简单看下它的源码实现:创建RecyclableArrayList对象， 判断当前需要编码的对象是否是编码器可处理的类型，如果不是，则忽略，执行下一个ChannelHandler的write方法。 具体的编码方法实现由用户子类编码器负责完成，如果编码后的RecyclableArrayList为空，说明编码没有成功，释放RecyclableArrayList引用。 如果编码成功，则通过遍历RecyclableArrayList,循环发送编码后的POJO对象，代码如下所示:

```java
public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
    CodecOutputList out = null;
    
        if (acceptOutboundMessage(msg)) {
            out = CodecOutputList.newInstance();
            @SuppressWarnings("unchecked")
            I cast = (I) msg;
            try {
                encode(ctx, cast, out);
            } finally {
                ReferenceCountUtil.release(cast);
            }
            if (out.isEmpty()) {
                out.recycle();
                out = null;
                throw new EncoderException(
                        StringUtil.simpleClassName(this) + " must produce at least one message.");
            }
        } else {
            ctx.write(msg, promise);
        }
}
```

### LengthFieldPrepender编码器

如果协议中的第一个字段为长度字段，Netty提供了LengthFieldPrepender编码器，它可以计算当前待发送消息的二进制字节长度，将该长度添加到ByteBuf的缓冲区头中，如图所示:

```java
# 编码前(12 bytes)              编码后(14 bytes)
+---------------+        +--------+---------------+
| "HELLO,WORLD" | -----> + 0x000E | "HELLO,WORLD" | 
+---------------+        +--------+---------------+
```

通过LengthFieldPrepender可以将待发送消息的长度写入到ByteBuf的前2个字节，编码后的消息组成为长度字段+原消息的方式。通过设置L engthFieldPrepender为true,消息长度将包含长度本身占用的字节数，打开LengthFieldPrepender后， 编码结果如下图所示:

```java
protected void encode(ChannelHandlerContext ctx, ByteBuf msg, List<Object> out) throws Exception {
    int length = msg.readableBytes() + lengthAdjustment;
    if (lengthIncludesLengthFieldLength) {
        length += lengthFieldLength;
    }
    if (length < 0) {
        throw new IllegalArgumentException(
                "Adjusted frame length (" + length + ") is less than zero");
    }
    switch (lengthFieldLength) {
    case 1:
        if (length >= 256) {
            throw new IllegalArgumentException(
                    "length does not fit into a byte: " + length);
        }
        out.add(ctx.alloc().buffer(1).order(byteOrder).writeByte((byte) length));
        break;
    case 2:
        if (length >= 65536) {
            throw new IllegalArgumentException(
                    "length does not fit into a short integer: " + length);
        }
        out.add(ctx.alloc().buffer(2).order(byteOrder).writeShort((short) length));
        break;
    case 3:
        if (length >= 16777216) {
            throw new IllegalArgumentException(
                    "length does not fit into a medium integer: " + length);
        }
        out.add(ctx.alloc().buffer(3).order(byteOrder).writeMedium(length));
        break;
    case 4:
        out.add(ctx.alloc().buffer(4).order(byteOrder).writeInt(length));
        break;
    case 8:
        out.add(ctx.alloc().buffer(8).order(byteOrder).writeLong(length));
        break;
    default:
        throw new Error("should not reach here");
    }
    out.add(msg.retain());
}
```

LengthFieldPrepender工作原理分析如下:首先对长度字段进行设置，如果需要包含消息长度自身，则在原来长度的基础之.上再加上lengthFieldLength的长度。如果调整后的消息长度小于0，则抛出参数非法异常。对消息长度自身所占的字节数进行判断，以便采用正确的方法将长度字段写入到ByteBuf中，共有以下6种可能:

1. 长度字段所占字节为1:如果使用1个Byte字节代表消息长度，则最大长度需要小于256个字节。对长度进行校验，如果校验失败，则抛出参数非法异常;若校验通过，则创建新的ByteBuf并 通过writeByte将长度值写入到ByteBuf中 
2. 长度字段所占字节为2:如果使用2个Byte字节代表消息长度，则最大长度需要小于65536个字节，对长度进行校验，如果校验失败，则抛出参数非法异常;若校验通过，则创建新的ByteBuf并 通过writeShort将长度值写入到ByteBuf中 
3. 长度字段所占字节为3:如果使用3个Byte字节代表消息长度，则最大长度需要小于16777216个字节，对长度进行校验，如果校验失败，则抛出参数非法异.常;若校验通过，则创建新的ByteBuf并通过writeMedium将长度值写入到ByteBuf中 
4. 长度字段所占字节为4:创建新的ByteBuf，并通过writelnt将长度值写 入到ByteBuf中 
5. 长度字段所占字节为8:创建新的ByteBuf,并通过writel ong将长度值写入到ByteBuf中 
6. 其它长度值:直接抛出Error

最后将原需要发送的ByteBuf复制到List&lt; Object &gt; out中，完成编码。

