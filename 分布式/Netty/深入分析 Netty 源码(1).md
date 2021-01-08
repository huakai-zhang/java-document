1 客户端 BootStrap

BootStrap 是 Netty 提供的一个便利的工厂类，可以通过它来完成 Netty 的客户端或服务端的 Netty 初始化。 从客户端方面启动 Netty：

```java
EventLoopGroup workerGroup = new NioEventLoopGroup();
Bootstrap bootstrap = new Bootstrap();
bootstrap.group(workerGroup) // 指定 EventLoopGroup
.channel(NioSocketChannel.class) // 指定 Channel 的类型
.option(ChannelOption.SO_KEEPALIVE, true)
.handler(new ChannelInitializer<SocketChannel>() { // 设置数据的处理器
    @Override
    public void initChannel(SocketChannel ch) throws Exception {
        ch.pipeline().addLast(new IMDecoder());
        ch.pipeline().addLast(new IMEncoder());
        ch.pipeline().addLast(clientHandler);
    }
});
ChannelFuture f = b.connect(this.host, this.port).sync();
f.channel().closeFuture().sync();
```

1.1 EventLoop 的初始化

最开始的 Client 用户代码中，我们在一开始就实例化了一个 `NioEventLoopGroup` 对象，因此我们就从它的构造器中追踪一下 EventLoop 的初始化过程。首先来看一下 NioEventLoopGroup 的类继承层次：

![image-20210106112732248](深入分析 Netty 源码(1).assets/image-20210106112732248.png)

NioEventLoop 有几个重载的构造器，不过内容都没有太大的区别，从 new NioEventLoopGroup() 开始，其中确定了线程数，selectorProvider ，选择器策略工厂 DefaultSelectStrategyFactory.INSTANCE 最终都是调用的父类MultithreadEventLoopGroup的构造器：

```java
// NioEventLoopGroup.java
public NioEventLoopGroup() {
    this(0);
}
public NioEventLoopGroup(int nThreads) {
    this(nThreads, (Executor) null);
}
// MultithreadEventLoopGroup.java
protected MultithreadEventLoopGroup(int nThreads, Executor executor, Object... args) {
    super(nThreads == 0 ? DEFAULT_EVENT_LOOP_THREADS : nThreads, executor, args);
}
```

其中有个有意思的地方是，如果我们传入的线程数nThreads 是0，那么Netty 会为我们设置默认的线程数`DEFAULT_EVENT_LOOP_THREADS`，而这个默认的线程数是怎么确定的呢?其实很简单，在静态代码块中，会首先确定DEFAULT_EVENT_LOOP_THREADS 的值：

```java
static {
    DEFAULT_EVENT_LOOP_THREADS = Math.max(1, SystemPropertyUtil.getInt(
            "io.netty.eventLoopThreads", Runtime.getRuntime().availableProcessors() * 2));
}
```

Netty 首先会从系统属性中获取 `io.netty.eventLoopThreads` 的值，如果我们没有设置的话，那么就返回默认值：`即处理器核心数* 2`。回到 MultithreadEventLoopGroup 构造器中会继续调用父类MultithreadEventExecutorGroup 的构造器：

```java
protected MultithreadEventExecutorGroup(int nThreads, Executor executor, Object... args) {
    this(nThreads, executor, DefaultEventExecutorChooserFactory.INSTANCE, args);
}
```

其中 DefaultEventExecutorChooserFactory 就是事件执行器选择策略的创建工厂，继续调用重载方法：

```java
protected MultithreadEventExecutorGroup(int nThreads, Executor executor, EventExecutorChooserFactory chooserFactory, Object... args) {
    if (executor == null) {
        // 默认初始化 executor 为 null
    	executor = new ThreadPerTaskExecutor(newDefaultThreadFactory());
	}
    ...
    children = new EventExecutor[nThreads];
    for (int i = 0; i < nThreads; i ++) {
        ...
        children[i] = newChild(executor, args);　　　　　
    }
    ...
    chooser = chooserFactory.newChooser(children);
　　 //去掉以下代码
}
```

这里其实就是创建了一个大小为 nThreads 的事件执行器数组，第二步是初始化该数组，紧接着通过选择策略工厂 EventExecutorChooserFactory 创建选择策略，首先看一下他的初始化，调用newChhild()方法初始化 children 数组。

根据上面的代码，我们也能知道：`MultithreadEventExecutorGroup 内部维护了一个 EventExecutor 数组`，而Netty 的EventLoopGroup 的实现机制其实就建立在MultithreadEventExecutorGroup 之上。每当Netty 需要一个 EventLoop 时,会调用next()方法获取一个可用的 EventLoop。newChild() 方法是一个抽象方法，它的任务是实例化 EventLoop 对象。我们跟踪一下它的代码。可以发现。这个方法在 NioEventLoopGroup 类中有实现，其内容很简单：

```java
protected EventLoop newChild(Executor executor, Object... args) throws Exception {
    return new NioEventLoop(this, executor, (SelectorProvider) args[0],
        ((SelectStrategyFactory) args[1]).newSelectStrategy(), (RejectedExecutionHandler) args[2]);
}
```

在此构造方法中调用其父类 `SingleThreadEventLoop` 的构造方法，再调用 SingleThreadEventExecutor 的构造方法。

数组初始化结束，紧接着是初始化选择器，我们可以继续跟踪到 newChooser 方法里面看看其实现逻辑，具体代码如下：

```java
public final class DefaultEventExecutorChooserFactory implements EventExecutorChooserFactory {

    public static final DefaultEventExecutorChooserFactory INSTANCE = new DefaultEventExecutorChooserFactory();

    private DefaultEventExecutorChooserFactory() { }

    @SuppressWarnings("unchecked")
    @Override
    public EventExecutorChooser newChooser(EventExecutor[] executors) {
        //判断是否二次幂
        if (isPowerOfTwo(executors.length)) {
            return new PowerOfTowEventExecutorChooser(executors);
        } else {
            return new GenericEventExecutorChooser(executors);
        }
    }

    private static boolean isPowerOfTwo(int val) {
        return (val & -val) == val;
    }

    private static final class PowerOfTowEventExecutorChooser implements EventExecutorChooser {
        private final AtomicInteger idx = new AtomicInteger();
        private final EventExecutor[] executors;

        PowerOfTowEventExecutorChooser(EventExecutor[] executors) {
            this.executors = executors;
        }

        @Override
        public EventExecutor next() {
             // 索引自增 & 执行器数组长度-1
            return executors[idx.getAndIncrement() & executors.length - 1];
        }
    }

    private static final class GenericEventExecutorChooser implements EventExecutorChooser {
        private final AtomicInteger idx = new AtomicInteger();
        private final EventExecutor[] executors;

        GenericEventExecutorChooser(EventExecutor[] executors) {
            this.executors = executors;
        }

        @Override
        public EventExecutor next() {
            // 索引自增 % 执行器数组长度后取绝对值
            return executors[Math.abs(idx.getAndIncrement() % executors.length)];
        }
    }
}
```

上面的代码逻辑主要表达的意思是：即如果 nThreads 是2 的幂，则使用 PowerOfTwoEventExecutorChooser，否则使用 GenericEventExecutorChooser。这两个 Chooser 都重写 next() 方法。next() 方法的主要功能就是将数组索引循环位移，如下图所示：

![image-20210106115932368](深入分析 Netty 源码(1).assets/image-20210106115932368.png)

当索引移动最后一个位置时，再调用next()方法就会将索引位置重新指向0。

![image-20210106115951762](深入分析 Netty 源码(1).assets/image-20210106115951762.png)

这个运算逻辑其实很简单，就是每次让索引自增后和数组长度取模：idx.getAndIncrement() % executors.length。但是就连一个非常简单的数组索引运算，Netty 都帮我们做了优化。因为在计算机底层，& 与比 % 运算效率更高。

分析到这里我们应该已经非常清楚 MultithreadEventExecutorGroup 中的处理逻辑，最后总结一下整个EventLoopGroup 的初始化过程：

![image-20210106133107473](深入分析 Netty 源码(1).assets/image-20210106133107473.png)

1. EventLoopGroup(其实是MultithreadEventExecutorGroup)内部维护一个类型为 EventExecutor 名为 children 的数组，是`大小为 nThreads 的 SingleThreadEventExecutor`，这样就构成了一个线程池。

2. 如果我们在实例化 NioEventLoopGroup 时，如果指定线程池大小，则 nThreads 就是指定的值，反之是处理器核心数* 2。

3. MultithreadEventExecutorGroup 中会调用 newChild() 抽象方法来初始化 children 数组。

4. 抽象方法 newChild() 是在 NioEventLoopGroup 中实现的，它返回一个 NioEventLoop 实例。

5. NioEventLoop 属性赋值：

   provider：在NioEventLoopGroup 构造器中通过SelectorProvider.provider()获取一个SelectorProvider。

   selector：在NioEventLoop 构造器中通过调用通过provider.openSelector()方法获取一个selector 对象。

6. 根据nThreads 的大小，创建不同的Chooser，即如果nThreads 是2 的幂，则使用PowerOfTwoEventExecutorChooser，反之使用GenericEventExecutorChooser。不论使用哪个Chooser，它们的功能都是一样的，即从children 数组中选出一个合适的EventExecutor 实例。

1.2 Channel 简介

在Netty 中，Channel 是一个Socket 的抽象，它为用户提供了关于Socket 状态(是否是连接还是断开)以及对Socket的读写等操作。每当Netty 建立了一个连接后, 都创建一个对应的Channel 实例。除了TCP 协议以外，Netty 还支持很多其他的连接协议, 并且每种协议还有NIO(非阻塞IO)和OIO(Old-IO, 即传统的阻塞IO)版本的区别。不同协议不同的阻塞类型的连接都有不同的Channel 类型与之对应下面是一些常用的Channel类型：

- NioSocketChannel ：异步非阻塞的客户端TCP Socket 连接。
- NioServerSocketChannel ：异步非阻塞的服务器端TCP Socket 连接
- NioDatagramChannel ：异步非阻塞的UDP 连接。
- NioSctpChannel ：异步的客户端Sctp（Stream Control Transmission Protocol，流控制传输协议）连接。
- NioSctpServerChannel ：异步的Sctp 服务器端连接。
- OioSocketChannel ：同步阻塞的客户端TCP Socket 连接。
- OioServerSocketChannel ：同步阻塞的服务器端TCP Socket 连接。
- OioDatagramChannel ：同步阻塞的UDP 连接。
- OioSctpChannel ：同步的Sctp 服务器端连接。
- OioSctpServerChannel： 同步的客户端TCP Socket 连接。

![image-20210106134621498](深入分析 Netty 源码(1).assets/image-20210106134621498.png)

1.3 NioSocketChannel 的创建

从上面的客户端代码虽然简单, 但是却展示了 Netty 客户端初始化时所需的所有内容：

1. EventLoopGroup：不论是服务端还是客户端，都必须指定 EventLoopGroup，在例子中，指定了 NioEventLoopGroup，表示一个 NIO 的 EventLoopGroup 
2. ChannelType：指定 Channel 的类型，因为是客户端，因此使用了 NioSocketChannel 
3. Handler：设置数据的处理器

下面我们继续深入代码，看一下客户端通过Bootstrap 启动后，都做了哪些工作？我们看一下NioSocketChannel 的类层次结构如下:

![image-20210106134942764](深入分析 Netty 源码(1).assets/image-20210106134942764.png)

回到我们在客户端连接代码的初始化 Bootstrap 中调用了一个 `channel()` 方法，传入的参数是`NioSocketChannel.class`，在这个方法中其实就是初始化了一个 ReflectiveChannelFactory 的对象：

```java
public B channel(Class<? extends C> channelClass) {
    if (channelClass == null) {
        throw new NullPointerException("channelClass");
    }
    return channelFactory(new ReflectiveChannelFactory<C>(channelClass));
}
```

而 ReflectiveChannelFactory 实现了ChannelFactory 接口，它提供了唯一的方法，即newChannel()方法，ChannelFactory。顾名思义，就是创建Channel 的工厂类。进入到ReflectiveChannelFactory 的 newChannel()方法中，我们看到其实现代码如下:

```java
public class ReflectiveChannelFactory<T extends Channel> implements ChannelFactory<T> {
    private final Class<? extends T> clazz;
    public ReflectiveChannelFactory(Class<? extends T> clazz) {
        if (clazz == null) {
            throw new NullPointerException("clazz");
        }
        this.clazz = clazz;
    }
// bootstrap.connect() -> Bootstrap.doResolveAndConnect() -> AbstractBootstrap.initAndRegister() -> newChannel()
    @Override
    public T newChannel() {
        try {
            return clazz.newInstance();
        } catch (Throwable t) {
            throw new ChannelException("Unable to create Channel from class " + clazz, t);
        }
    }
}
```

根据上面代码的提示，我们就可以得出：

1. Bootstrap 中的 ChannelFactory 实现类是 ReflectiveChannelFactory
2. 通过 channel() 方法创建的 Channel 具体类型是 NioSocketChannel

Channel 的实例化过程其实就是调用ChannelFactory 的newChannel()方法，而实例化的Channel 具体类型又是和初始化Bootstrap 时传入的channel()方法的参数相关。因此对于客户端的Bootstrap 而言，创建的Channel 实例就是NioSocketChannel。

![image-20210106140921622](深入分析 Netty 源码(1).assets/image-20210106140921622.png)

1.4 客户端Channel 的初始化

前面我们已经知道了如何设置一个Channel 的类型，并且了解到Channel 是通过ChannelFactory 的newChannel()方法来实例化的, 那么ChannelFactory 的newChannel()方法在哪里调用呢?继续跟踪, 我们发现其调用链如下:

![image-20210106142226525](深入分析 Netty 源码(1).assets/image-20210106142226525.png)

在 AbstractBootstrap.initAndRegister 中调用了 channelFactory().newChannel() 来获取一个新的NioSocketChannel 实例，其源码如下:

```java
final ChannelFuture initAndRegister() {
	// 去除非关键代码
    Channel channel = channelFactory.newChannel();
    init(channel);
    ChannelFuture regFuture = config().group().register(channel);
	return regFuture;
}
```

在 newChannel 中，通过类对象的 newInstance 来获取一个新 Channel 实例，因而会调用 NioSocketChannel 的默认构造器：

```java
public NioSocketChannel() {
	this(DEFAULT_SELECTOR_PROVIDER);
}
```

这个构造器会调用 newSocket 来打开一个新的 Java NIO SocketChannel：

```java
private static SocketChannel newSocket(SelectorProvider provider) {
    return provider.openSocketChannel();
}
```

接着调用父类，即 AbstractNioByteChannel 的构造器，并传入参数 parent 为 null，ch 为刚才使用的 newSocket 创建的 Java NIO SocketChannel，因此生成的 NioSocketChannel 的 parent channel 是空的：

```java
protected AbstractNioByteChannel(Channel parent, SelectableChannel ch) {
    super(parent, ch, SelectionKey.OP_READ);
}
```

接着会调用父类 AbstractNioChannel 的构造器，并传入了参数 readInterestOp = SelectionKey.OP_READ：

```java
// AbstractNioByteChannel.java
protected AbstractNioByteChannel(Channel parent, SelectableChannel ch) {
    super(parent, ch, SelectionKey.OP_READ);
}
// AbstractNioChannel.java
protected AbstractNioChannel(Channel parent, SelectableChannel ch, int readInterestOp) {
    super(parent);
    this.ch = ch;
    this.readInterestOp = readInterestOp;
    // 设置非阻塞
    ch.configureBlocking(false);
}
```

然后继续调用父类 AbstractChannel 的构造器：

```java
protected AbstractChannel(Channel parent) {
    this.parent = parent;
    id = newId();
    unsafe = newUnsafe();
    pipeline = newChannelPipeline();
}
```

到这里，一个完整的NioSocketChannel就初始化完成了，稍微总结一下构造一个 NioSocketChannel 所需要做的工作:

1. 调用 NioSocketChannel.newSocket (DEFAULT_  *SELECTOR*  PROVIDER) 打开一个新的Java NIOSocketChannel 

2. AbstractChannel (Channel parent) 中初始化AbstractChannel 的属性: 

  `id` 每个 Channel 都拥有一个唯一的id

  `parent` 属性置为null 

  `unsafe` 通过newUnsafe() 实例化一个unsafe 对象，它的类型是AbstractNioByteChannel.NioByteUnsafe内部类 

  `pipeline` 是new DefaultChannelPipeline(this)新创建的实例

  这里体现了: Each channel has its own pipeline and it is created automatically when a new channel is created. 

3. AbstractNioChannel 中的属性: 

   `ch` 被设置为 Java SocketChannel,即NioSocketChannel.newSocket 返回 Java NIO SocketChannel. 

   `readInterestOp` 被设置成 SelectionKey. OP_ READ

   `ch` 被配置为非阻塞 ch.configureBlocking(false)

4. NioSocketChannel 中的属性： 

   SocketChannelConfig config = newNioSocketChannelConfig(this,socket. socket())

1.5 unsafe 字段的初始化

在实例化 NioSocketChannel 的过程中，会在父类AbstractChannel 的构造器中，调用newUnsafe()来获取一个unsafe 实例. 其实unsafe封装了对Java底层Socket 的操作，因此实际上是沟通Netty 上层和Java底层的重要的桥梁. 那么看一下Unsafe接口所提供的方法:

```java
interface Unsafe {
    RecvByteBufAllocator.Handle recvBufAllocHandle();
    SocketAddress localAddress();
    SocketAddress remoteAddress();
    void register(EventLoop eventLoop, ChannelPromise promise);
    void bind(SocketAddress localAddress, ChannelPromise promise);
    void connect(SocketAddress remoteAddress, SocketAddress localAddress, ChannelPromise promise);
    void disconnect(ChannelPromise promise);
    void close(ChannelPromise promise);
    void closeForcibly();
    void deregister(ChannelPromise promise);
    void beginRead();
    void write(Object msg, ChannelPromise promise);
    void flush();
    ChannelPromise voidPromise();
    ChannelOutboundBuffer outboundBuffer();
}
```

一看便知，这些方法其实都会对应到相关的Java 底层的Socket 的操作. 回到AbstractChannel 的构造方法中，在这里调用了newUnsafe() 获取一个新的unsafe 对象，而newUnsafe方法在NioSocketChannel 中被重写了:

```java
protected AbstractNioUnsafe newUnsafe() {
    return new NioSocketChannelUnsafe();
}
```

NioSocketChannel.newUnsafe 方法会返回一个 `NioSocketChannelUnsafe` 实例，可以确定在实例化的 NioSocketChannel 中的 unsafe 字段，其实是一个 NioSocketChannelUnsafe 的实例。

1.6 关于 pipeline 的初始化

上面我们分析了NioSocketChannel 的大体初始化过程，但是我们漏掉了一个关键的部分，即 ChannelPipeline 的初始化。

> 在Pipeline 的注释说明中写到：
>
> Each channel has its own pipeline and it is created automatically when a new channel is created.

可知在实例化一个Channel 时，必然伴随着实例化一个ChannelPipeline。而确实在AbstractChannel 的构造器看到了 pipeline 字段被初始化为DefaultChannelPipeline的实例：

```java
protected DefaultChannelPipeline(Channel channel) {
    this.channel = ObjectUtil.checkNotNull(channel, "channel");
    succeededFuture = new SucceededChannelFuture(channel, null);
    voidPromise =  new VoidChannelPromise(channel, true);
    tail = new TailContext(this);
    head = new HeadContext(this);
    head.next = tail;
    tail.prev = head;
}
```

调用 DefaultChannelPipeline 的构造器，传入了一个channel, 而这个channel 其实就是实例化的NioSocketChannel, DefaultChannelPipeline会将这个NioSocketChannel 对象保存在channel 字段中。

DefaultChannelPipeline 中，还有两个特殊的字段，即 `head` 和 `tail`， 而这两个字段是一个双向链表的头和尾.其实在 DefaultChannelPipeline 中，维护了一个以 AbstractChannelHandlerContext 为节点的双向链表，这个链表是 Netty 实现Pipeline 机制的关键。关于DefaultChannelPipeline 中的双向链表以及它所起的作用。 先看看HeadContext跟TailContext的类继承层次结构如下所示：

![image-20210106144109690](深入分析 Netty 源码(1).assets/image-20210106144109690.png)  

再来看看二者的构造器：

```java
HeadContext(DefaultChannelPipeline pipeline) {
    super(pipeline, null, HEAD_NAME, false, true);
    unsafe = pipeline.channel().unsafe();
    setAddComplete();
}
TailContext(DefaultChannelPipeline pipeline) {
    super(pipeline, null, TAIL_NAME, true, false);
    setAddComplete();
}
```

我们可以看到，链表中 head 实现了ChannelOutboundHandler 跟 ChannelInboundHandler，它调用了父类 AbstractChannelHandlerContext 的构造器，并传入参数 inbound = false，outbound = true。 TailContext 的构造器与 HeadContext 的相反，它调用了父类 AbstractChannelHandlerContext的构造器，并传入参数inbound = true, outbound = false。 即header 是一个outboundHandler, 而tail 是一个inboundHandler。

关于这一点，要特别注意，因为在分析到Netty Pipeline 时，会反复用到inbound 和outbound 这两个属性。

1.7 Channel 注册到 Selector

在前面的分析中，我们提到 Channel 会在 Bootstrap 的 initAndRegister() 中进行初始化，但是这个方法还会将初始化好的 Channe 注册到NioEventLoop 的selector 中。接下来我们来分析一下Channel 注册的过程。再回顾一下AbstractBootstrap 的initAndRegister()方法：

```java
final ChannelFuture initAndRegister() {
	...
    channel = channelFactory.newChannel();
    init(channel);
    // channel 作为 ChannelFuture 的参数
    ChannelFuture regFuture = config().group().register(channel);
    ...
}
// MultithreadEventLoopGroup.java
public EventLoop next() {
    return (EventLoop) super.next();
}
public ChannelFuture register(Channel channel) {
    return next().register(channel);
}
// 最终 next() 调用的是 newChooser 中定义的两个 EventExecutorChooser 的其中一个
// 实际返回的是 new EventExecutor[nThreads] 数组中的下一个
```

当 Channel 初始化后，会紧接着调用 group.register() 方法来注册Channel，其调用链如下: AbstractBootstrap. initAndRegister-&gt;MultithreadEventLoopGroup. register-&gt;SingleThreadEventLoop.register -&gt; AbstractChannel$AbstractUnsafe.register

通过跟踪调用链，最终发现是调用到了unsafe 的 register 方法，那么接下来就仔细看一下`AbstractChannel$AbstractUnsafe.register` 方法中到底做了什么:

```java
public final void register(EventLoop eventLoop, final ChannelPromise promise) {
    ...
    AbstractChannel.this.eventLoop = eventLoop;
    if (eventLoop.inEventLoop()) {
    	register0(promise);
    } else {...}
}
```

首先，`将 eventLoop赋值给 Channel 的eventLoop 属性`，而我们知道这个 eventLoop 对象其实是 MultithreadEventLoopGroup.next() 方法获取的，根据我们前面的小节中，我们可以确定 next() 方法返回的 eventLoop 对象是 NioEventLoop 实例。register方法接着调用了 register0 方法，register0 又调用了 AbstractNioChannel.doRegister：

```java
protected void doRegister() throws Exception {
	boolean selected = false;
		for (;;) {//javaChannel()是channel初始化的时候设置的值
			selectionKey = javaChannel().register(eventLoop().unwrappedSelector(), 0, this);
			return;
		}
}
```

看到 javaChannel() 这个方法在前面已经知道了，它返回的是一个Java NIO的 SocketChannel，这里将这个SocketChannel 注册到与 eventLoop 关联的selector上了。总结一下Channel 的注册过程:

1. 首先AbstractBootstrap.initAndRegister 中, 通过group().register(channel),调用MultithreadEventLoopGroup.register 方法 
2. MultithreadEventLoopGroup.register 中，通过 next() 获取一个可用的 SingleThreadEventLoop, 然后调用它的 register 
3. 在 SingleThreadEventLoop.register 中 , 通过 channel.unsafe().register(this,promise) 来获取 channel 的 unsafe() 底层操作对象，然后调用它的 register 
4. 在 AbstractUnsafe.register 方法中，调用 register0 方法注册 Channel 
5. 在 AbstractUnsafe.register0 方法中，调用 AbstarctNioChannel.doRegister 方法 
6. AbstarctNIioChannel.doRegister 方法通过 javaChannel().register(eventLoop().selector, 0, this)将 Channel 对应的 Java NIO SocketChannel 注册到一个 eventLoop 的 Selector 中，并将当前 Channel 作为 attachment

总的来说，Channel 注册过程所做的工作就是将Channel与对应的EventLoop 关联，因此这也体现了，在Netty中，每个Channel 都会关联一个特定的EventLoop， 并且这个Channel中的所有IO操作都是在这个EventLoop 中执行的;当关联好Channel 和EventLoop 后，会继续调用底层的 Java NIO SocketChannel 的 register 方法，将底层的 Java NIO SocketChannel 注册到指定的 selector 中，通过这两步，就完成了 Netty Channel 的注册过程。

1.8 Handler 的添加过程

Netty的一个强大和灵活之处就是基于Pipeline的自定义handler 机制。基于此，我们可以像添加插件一样自由组合各种各样的handler 来完成业务逻辑。例如我们需要处理HTTP数据，那么就可以在pipeline 前添加一个Http的编解码的Handler， 然后接着添加我们自己的业务逻辑的handler， 这样网络上的数据流就向通过一个管道一样，从不同的handler 中流过并进行编解码，最终在到达我们自定义的handler 中。

既然说到这里，有些同学肯定会好奇，既然这个pipeline 机制是这么的强大，那么它是怎么实现的呢?在这里我还不打算详细讲解，在这一小节中，我们从简单的入手，展示一下我们自定义的handler 是如何以及何时添加到ChannelPipeline 中的。首先让我们看一下如下的代码片段:

```java
.handler(new ChannelInitializer<SocketChannel>() {
                @Override
                public void initChannel(SocketChannel ch) throws Exception {
                    ch.pipeline().addLast(new IMDecoder());
                    ch.pipeline().addLast(new IMEncoder());
                    ch.pipeline().addLast(clientHandler);
                }
            });
// AbstractBootstrap.java
public B handler(ChannelHandler handler) {
    if (handler == null) {
        throw new NullPointerException("handler");
    }
    this.handler = handler;
    return (B) this;
}
```

```java
// AbstractBootstrap 的initAndRegister()方法会调用init()方法
// 将 handle 方法 添加的 ChannelInitializer 放入 Pipeline
void init(Channel channel) throws Exception {
    ChannelPipeline p = channel.pipeline();
    p.addLast(config.handler());
    ...
}
// AbstractBootstrapConfig.java
public final ChannelHandler handler() {
    return bootstrap.handler();
}
// AbstractBootstrap.java
final ChannelHandler handler() {
    return handler;
}
```

这个代码片段就是实现了handler 的添加功能。我们看到，Bootstrap.handler方法接收一个ChannelHandler，而我们传递的是一个派生于 ChannelInitializer 的匿名类，它正好也实现了ChannelHandler 接口.我们来看一下，Channellnitializer 类内到底有什么玄机:

```java
public abstract class ChannelInitializer<C extends Channel> extends ChannelInboundHandlerAdapter {

    private static final InternalLogger logger = InternalLoggerFactory.getInstance(ChannelInitializer.class);
    private final ConcurrentMap<ChannelHandlerContext, Boolean> initMap = PlatformDependent.newConcurrentHashMap();

    protected abstract void initChannel(C ch) throws Exception;

    @Override
    @SuppressWarnings("unchecked")
    public final void channelRegistered(ChannelHandlerContext ctx) throws Exception {
        // 调用 initChannel(ChannelHandlerContext ctx)
        if (initChannel(ctx)) {
            ctx.pipeline().fireChannelRegistered();
        } else {
            // Called initChannel(...) before which is the expected behavior, so just forward the event.
            ctx.fireChannelRegistered();
        }
    }
...
    @SuppressWarnings("unchecked")
    private boolean initChannel(ChannelHandlerContext ctx) throws Exception {
        if (initMap.putIfAbsent(ctx, Boolean.TRUE) == null) { // Guard against re-entrance.
            try {
                // 调用自定义重写initChannel(C ch)方法
                initChannel((C) ctx.channel());
            } catch (Throwable cause) {
                // Explicitly call exceptionCaught(...) as we removed the handler before calling initChannel(...).
                // We do so to prevent multiple calls to initChannel(...).
                exceptionCaught(ctx, cause);
            } finally {
                remove(ctx);
            }
            return true;
        }
        return false;
    }
...
}
```

ChannelInitializer是一个抽象类，它有一个抽象的方法initChannel, 正是实现了这个方法，并在这个方法中添加的自定义的handler的。那么initChannel 是哪里被调用的呢? 

答案是ChannelInitializer. channelRegistered方法中。 来关注一下 channelRegistered方法，从上面的源码中，我们可以看到，在 channelRegistered 方法中，会调用 initChannel 方法，将自定义的handler添加到ChannelPipeline中，然后调用ctx. pipeline(). remove(this)将自己从ChannelPipeline中删除。

上面的分析过程，可以用如下图片展示: 一开始，ChannelPipeline 中只有三个handler，head, tail 和我们添加的ChannelInitializer。 

![image-20210106151453069](深入分析 Netty 源码(1).assets/image-20210106151453069.png)

接着 initChannel 方法调用后，添加了自定义的 handler：

![image-20210106151515816](深入分析 Netty 源码(1).assets/image-20210106151515816.png)

最后将 ChannelInitializer 删除：

![image-20210106151538931](深入分析 Netty 源码(1).assets/image-20210106151538931.png)

分析到这里，我们已经简单了解了自定义的 handler 是如何添加到 ChannelPipeline 中的，后续我们再进行深入的探讨。

1.9 客户端连接分析

首先，客户端会通过调用 Bootstrap 的 connect 方法进行连接。在 connect 中，会进行一些参数检查后，最终调用的是 doConnect 方法，其实现如下：

```java
private static void doConnect(
        final SocketAddress remoteAddress, final SocketAddress localAddress, final ChannelPromise connectPromise) {
    final Channel channel = connectPromise.channel();
    channel.eventLoop().execute(new Runnable() {
        @Override
        public void run() {
            if (localAddress == null) {
                channel.connect(remoteAddress, connectPromise);
            } else {
                channel.connect(remoteAddress, localAddress, connectPromise);
            }
            connectPromise.addListener(ChannelFutureListener.CLOSE_ON_FAILURE);
        }
    });
}
```

在 doConnect 中，会在 event loop 线程中调用 Channel 的 connect 方法，而这个 Channel 的具体类型是什么呢？这里 channel 的类型就是 NioSocketChannel。 进行跟踪到 channel.connect 中，发现它调用的是 DefaultChannelPipeline.connect，而 pipeline 的 connect 代码如下：

```java
public final ChannelFuture connect(SocketAddress remoteAddress, ChannelPromise promise) {
    return tail.connect(remoteAddress, promise);
}
```

而tail 字段，我们已经分析过了，是一个TailContext 的实例，而TailContext 又是AbstractChannelHandlerContext的子类，并且没有实现connect 方法，因此这里调用的其实是AbstractChannelHandlerContext. connect,我们看一下这个方法的实现:

```java
public ChannelFuture connect(
        final SocketAddress remoteAddress, final SocketAddress localAddress, final ChannelPromise promise) {
    if (remoteAddress == null) {
        throw new NullPointerException("remoteAddress");
    }
    if (!validatePromise(promise, false)) {
        // cancelled
        return promise;
    }
    final AbstractChannelHandlerContext next = findContextOutbound();
    EventExecutor executor = next.executor();
    if (executor.inEventLoop()) {
        next.invokeConnect(remoteAddress, localAddress, promise);
    } else {
        safeExecute(executor, new Runnable() {
            @Override
            public void run() {
                next.invokeConnect(remoteAddress, localAddress, promise);
            }
        }, promise, null);
    }
    return promise;
}
```

上面的代码中有一个关键的地方，即`final AbstractChannelHandlerContext next = findContextOutbound()`，这里调用findContextOutbound 方法，从DefaultChannelPipeline内的双向链表的tail开始，不断向前寻找第一个outbound 为true 的 AbstractChannelHandlerContext，然后调用它的invokeConnect 方法，其代码如下:

```java
private void invokeConnect(SocketAddress remoteAddress, SocketAddress localAddress, ChannelPromise promise) {
    if (invokeHandler()) {
        try {
            ((ChannelOutboundHandler) handler()).connect(this, remoteAddress, localAddress, promise);
        } catch (Throwable t) {
            notifyOutboundHandlerException(t, promise);
        }
    } else {
        connect(remoteAddress, localAddress, promise);
    }
}
```

前面我们提到，在 DefaultChannelPipeline 的构造器中，会实例化两个对象: head和tail,并形成了双向链表的头和尾，head 是HeadContext 的实例，它实现了ChannelOutboundHandler接口，并且它的outbound字段为true。

因此在findContextOutbound中，找到的AbstractChannelHandlerContext对象其实就是head. 进而在invokeConnect 方法中，我们向上转换为ChannelOutboundHandler 就是没问题的了。而又因为HeadContext 重写了connect 方法，因此实际上调用的是HeadContext. connect.我们接着跟踪到HeadContext. connect，其代码如下:

```java
public void connect(
        ChannelHandlerContext ctx,
        SocketAddress remoteAddress, SocketAddress localAddress,
        ChannelPromise promise) throws Exception {
    unsafe.connect(remoteAddress, localAddress, promise);
}
```

这个connect方法很简单，仅仅调用了unsafe 的connect 方法。而unsafe 又是什么呢? 回顾一下HeadContext的构造器，我们发现unsafe 是pipeline.channel().unsafe()返回的是Channel 的unsafe字段，在这这里，我们已经知道了，其实是AbstractNioByteChannel.NioByteUnsafe内部类.兜兜转转了一大圈，我们找到了创建Socket连接的关键代码。进行跟踪 NioByteUnsafe -&gt; AbstractNioUnsafe.connect:

```java
public final void connect(
        final SocketAddress remoteAddress, final SocketAddress localAddress, final ChannelPromise promise) {
  
        boolean wasActive = isActive();
        if (doConnect(remoteAddress, localAddress)) {
            fulfillConnectPromise(promise, wasActive);
        } else {...}
}
```

AbstractNioUnsafe.connect 的实现代码如上，在这个 connect 方法中，调用了 doConnect 方法，注意，这个方法并不是AbstractNioUnsafe 的方法，而是 AbstractNioChannel 的抽象方法。 doConnect 是在 NioSocketChannel 中实现的，因此进入到 NioSocketChannel.doConnect 中 :

```java
protected boolean doConnect(SocketAddress remoteAddress, SocketAddress localAddress) throws Exception {
    if (localAddress != null) {
        doBind0(localAddress);
    }
    boolean success = false;
    try {
        boolean connected = javaChannel().connect(remoteAddress);
        if (!connected) {
            selectionKey().interestOps(SelectionKey.OP_CONNECT);
        }
        success = true;
        return connected;
    } finally {
        if (!success) {
            doClose();
        }
    }
}
```

我们终于看到的最关键的部分了，首先是获取Java NIO 的SocketChannel，获取NioSocketChannel 的newSocket() 返回的SocketChannel 对象；然后调用SocketChannel 的connect()方法完成JavaNIO 底层的Socket 的连接。最后总结一下，客户端BootStrap 发起连接请求的流程可以用如下时序图直观地展示：

![image-20210106133107473](深入分析 Netty 源码(1).assets/NIO.png)

2 服务端 ServerBootstrap

在分析客户端的代码时，我们已经对Bootstrap 启动Netty 有了一个大致的认识，那么接下来分析服务器端时，就会相对简单一些了。首先还是来看一下服务器端的启动代码：

```java
public class ChatServer {
    private int port = 80;
    public void start() throws Exception {
        // Boss 线程
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        // Worker 线程
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {
            // 启动引擎
            ServerBootstrap server = new ServerBootstrap();
            // 主从模型
            server.group(bossGroup, workerGroup).channel(NioServerSocketChannel.class)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            // 所有自定义的业务从这开始
                            ChannelPipeline pipeline = socketChannel.pipeline();

                            // ------------------------- 支持自定义Socket协议 -------------------------
                            pipeline.addLast(new IMDecoder());
                            pipeline.addLast(new IMEncoder());
                            pipeline.addLast(new SocketHandler());

                            // ------------------------- 支持HTTP协议 -------------------------
                            // 解码和编码HTTP请求
                            pipeline.addLast(new HttpServerCodec());
                            pipeline.addLast(new HttpObjectAggregator(64 * 1024));
                            // 用于处理文件流的一个Handler
                            pipeline.addLast(new ChunkedWriteHandler());
                            // 用来拦截Http协议
                            pipeline.addLast(new HttpHandler());

                            // ------------------------- 支持WebSocket协议 -------------------------
                            pipeline.addLast(new WebSocketServerProtocolHandler("/im"));
                            pipeline.addLast(new WebSocketHandler());
                        }
                    }).option(ChannelOption.SO_BACKLOG, 128)
                    .childOption(ChannelOption.SO_KEEPALIVE, true);

            // 等待客户端连接
            ChannelFuture f = server.bind(this.port).sync();

            System.out.println("服务已启动，端口号：" + this.port);

            f.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }

    public static void main(String[] args) {
        try {
            new ChatServer().start();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

和客户端的代码相比，没有很大的差别，基本上也是进行了如下几个部分的初始化:

1. EventLoopGroup:不论是服务器端还是客户端，都必须指定EventLoopGroup。在这个例子中，指定了NioEventLoopGroup， 表示一个NIO的EventLoopGroup， 不过服务器端需要指定两个EventLoopGroup，一个是bossGroup，用于处理客户端的连接请求；另一个是workerGroup，用于处理与各个客户端连接的IO操作。
2. ChannelType: 指定Channel 的类型.因为是服务器端，因此使用了NioServerSocketChannel. 
3. Handler:设置数据的处理器.

2.1 bossGroup 与 workerGroup

在客户端的时候，我们初始化了一个EventLoopGroup 对象，而在服务端的初始化时，我们设置了两个EventLoopGroup，一个是bossGroup，另一个是workerGroup。那么这两个EventLoopGroup 都是干什么用的呢? 接下来我们详细探究一下。其实，bossGroup 只用于服务端的 accept，也就是用于处理客户端新连接接入请求。我们可以把 Netty 比作一个餐馆，bossGroup 就像一个大堂经理，当客户来到餐馆吃时，大堂经理就会引导顾客就坐，为顾客端茶送水等。而 workerGroup 就是实际上干活的厨师，它们负责客户端连接通道的IO 操作：当大堂经历接待顾客后，顾客可以稍做休息, 而此时后厨里的厨师们(workerGroup)就开始忙碌地准备饭菜了。关于bossGroup 与workerGroup 的关系，我们可以用如下图来展示：

<img src="深入分析 Netty 源码(1).assets/image-20210106172130516.png" alt="image-20210106172130516" style="zoom: 80%;" />

首先，服务端的bossGroup 不断地监听是否有客户端的连接，当发现有一个新的客户端连接到来时，bossGroup 就会为此连接初始化各项资源，然后从 workerGroup 中选出一个 EventLoop 绑定到此客户端连接中。那么接下来的服务器与客户端的交互过程就全部在此分配的 EventLoop 中完成。

口说无凭，我们还是以源码说话吧。首先在ServerBootstrap 初始化时，调用了b.group(bossGroup, workerGroup)设置了两个EventLoopGroup，我们跟踪进去以后会看到：

```java
public ServerBootstrap group(EventLoopGroup parentGroup, EventLoopGroup childGroup) {
    super.group(parentGroup);
    if (childGroup == null) {
        throw new NullPointerException("childGroup");
    }
    if (this.childGroup != null) {
        throw new IllegalStateException("childGroup set already");
    }
    this.childGroup = childGroup;
    return this;
}
// AbstractBootstrap.java
public B group(EventLoopGroup group) {
    if (group == null) {
        throw new NullPointerException("group");
    }
    if (this.group != null) {
        throw new IllegalStateException("group set already");
    }
    this.group = group;
    return (B) this;
}
```

显然，这个方法初始化了两个字段，一个是group = parentGroup。它是在 super.group(parentGroup) 中完成初始化的，另一个是childGroup = childGroup。接着从应用程序的启动代码来看调用了 b.bind() 方法来监听一个本地端口。bind()方法会触发如下调用链：AbstractBootstrap.bind() -> AbstractBootstrap.doBind() -> AbstractBootstrap.initAndRegister()源码看到到这里为止，我们发现AbstractBootstrap 的initAndRegister()方法已经是我们的老朋友了，我们在分析客户端程序时和它打过很多交道，现在再来回顾一下这个方法吧：

```java
final ChannelFuture initAndRegister() {
    channel = channelFactory.newChannel();
    init(channel); // ServerBootstrap
    // 此处使用的是parentGroup
    ChannelFuture regFuture = config().group().register(channel);
    return regFuture;
}
// AbstractBootstrapConfig.java
// config().group()
public final EventLoopGroup group() {
    return bootstrap.group();
}
// AbstractBootstrap.java
public final EventLoopGroup group() {
    return group;
}
```

这里group() 方法返回的是上面我们提到的bossGroup， 而这里的channel 我们也已经分析过了，它是NioServerSocketChannl 实例，因此可以知道, group().register (channel) 将 bossGroup 和 NioServerSocketChannl 关联起来了。 那么workerGroup是在哪里与NioServerSocketChannel 关联的呢? 继续看看init (channel)方法:

```java
void init(Channel channel) throws Exception {
    ChannelPipeline p = channel.pipeline();
    final EventLoopGroup currentChildGroup = childGroup;
    final ChannelHandler currentChildHandler = childHandler;
    final Entry<ChannelOption<?>, Object>[] currentChildOptions;
    final Entry<AttributeKey<?>, Object>[] currentChildAttrs;
    ...
    p.addLast(new ChannelInitializer<Channel>() {
        @Override
        public void initChannel(Channel ch) throws Exception {
            final ChannelPipeline pipeline = ch.pipeline();
            ChannelHandler handler = config.handler();
            if (handler != null) {
                pipeline.addLast(handler);
            }
            ch.eventLoop().execute(new Runnable() {
                @Override
                public void run() {
                    pipeline.addLast(new ServerBootstrapAcceptor(
                            currentChildGroup, currentChildHandler, currentChildOptions, currentChildAttrs));
                }
            });
        }
    });
}
```

init方法在 ServerBootstrap 中重写了，从上面的代码片段中我们看到，它为pipeline 中添加了一个ChannelInitializer，而这个 ChannelInitializer 中添加了一个关键的 ServerBootstrapAcceptor handler。关于handler 的添加与初始化的过程，我们留待下一小节中分析，我们现在关注一下ServerBootstrapAcceptor类。 ServerBootstrapAcceptor 中重写了 channelRead 方法，主要代码如下:

```java
public void channelRead(ChannelHandlerContext ctx, Object msg) {
	final Channel child = (Channel) msg;
    child.pipeline().addLast(childHandler);
    ...
        childGroup.register(child).addListener(...);
 ...
}
```

ServerBootstrapAcceptor 中的 childGroup 是构造此对象时传入的 currentChildGroup, 即我们的workerGroup, 而 Channel 是一个 NioSocketChannel 的实例, 因此这里的 childGroup.register 就是将workerGroup 中的某个EventLoop 和NioSocketChannel 关联了。既然这样，那么现在的问题是，ServerBootstrapAcceptor.channelRead 方法是怎么被调用的呢？其实当一个 client 连接到 server 时，Java 底层的 NIO ServerSocketChannel 会有一个 SelectionKey.OP_ACCEPT 就绪事件，接着就会调用到 NioServerSocketChannel.doReadMessages:

```java
protected int doReadMessages(List<Object> buf) throws Exception {
    SocketChannel ch = javaChannel().accept();
    buf.add(new NioSocketChannel(this, ch));
    return 1;
}
```

在 doReadMessages 中，通过 javaChannel().accept() 获取到客户端新连接的 SocketChannel，接着就实例化一个 NioSocketChannel，并传入 NioServerSocketChannel 对象（即 this），由此可知，创建的这个 NioSocketChannel 的父Channel 就是 NioServerSocketChannel 实例。 接下来就经由 Netty 的 ChannelPipeline机制，将读取事件逐级发送到各个handler中，于是就会触发前面提到的 ServerBootstrapAcceptor.channelRead 方法。

2.2 服务端 Selector 事件轮询

再回到服务端ServerBootStrap 的启动代码，是从bind()方法开始的。ServerBootStrapt 的bind()方法实际上就是其父类AbstractBootstrap 的bind()方法，来看代码：

```java
private static void doBind0(
        final ChannelFuture regFuture, final Channel channel,
        final SocketAddress localAddress, final ChannelPromise promise) {
    // This method is invoked before channelRegistered() is triggered.  Give user handlers a chance to set up
    // the pipeline in its channelRegistered() implementation.
    channel.eventLoop().execute(new Runnable() {
        @Override
        public void run() {
            if (regFuture.isSuccess()) {
                channel.bind(localAddress, promise).addListener(ChannelFutureListener.CLOSE_ON_FAILURE);
            } else {
                promise.setFailure(regFuture.cause());
            }
        }
    });
}
```

在doBind0()方法中，调用的是EventLoop 的execute()方法，我们继续跟进去：

```java
public void execute(Runnable task) {
    if (task == null) {
        throw new NullPointerException("task");
    }
    boolean inEventLoop = inEventLoop();
    if (inEventLoop) {
        addTask(task);
    } else {
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

在execute()主要就是创建线程，将线程添加到EventLoop 的无锁化串行任务队列。我们重点关注 startThread() 方法，继续看源代码：

```java
private void startThread() {
    doStartThread();
}
private void doStartThread() {
    executor.execute(new Runnable() {
　　　　　.....
       SingleThreadEventExecutor.this.run();
 　　　　.......
　　});
}
```

我们发现startThread()最终调用的是SingleThreadEventExecutor.this.run()方法，这个this 就是NioEventLoop 对象：

```java
        try {
            switch (selectStrategy.calculateStrategy(selectNowSupplier, hasTasks())) {
                case SelectStrategy.CONTINUE:
                    continue;
                case SelectStrategy.SELECT:
                    // 此方法中调用了 selector.select(timeoutMillis);
                    select(wakenUp.getAndSet(false));
                    if (wakenUp.get()) {
                        selector.wakeup();
                    }
                default:
                    // fallthrough
            }
            cancelledKeys = 0;
            needsToSelectAgain = false;
            final int ioRatio = this.ioRatio;
            if (ioRatio == 100) {
                try {
                    processSelectedKeys();
                } finally {
                    // Ensure we always run tasks.
                    runAllTasks();
                }
            ...
}
private void processSelectedKeys() {
    if (selectedKeys != null) {
        processSelectedKeysOptimized(selectedKeys.flip());
    } else {
        processSelectedKeysPlain(selector.selectedKeys());
    }
}
private void processSelectedKeysPlain(Set<SelectionKey> selectedKeys) {
    // check if the set is empty and if so just return to not create garbage by
    // creating a new Iterator every time even if there is nothing to process.
    // See https://github.com/netty/netty/issues/597
    if (selectedKeys.isEmpty()) {
        return;
    }
    Iterator<SelectionKey> i = selectedKeys.iterator();
    for (;;) {
        final SelectionKey k = i.next();
        final Object a = k.attachment();
        i.remove();
        if (a instanceof AbstractNioChannel) {
            // 接口类型为 AbstractNioChannel 时调用 processSelectedKey
            processSelectedKey(k, (AbstractNioChannel ) a);
        } else 
        ...
    }
}
```

终于看到似曾相识的代码。上面代码主要就是用一个死循环，在不断地轮询 SelectionKey.select() 方法，主要用来解决JDK 空轮询 Bug，而 processSelectedKeys() 就是针对不同的轮询事件进行处理。如果客户端有数据写入，最终也会调用AbstractNioMessageChannel 的doReadMessages()方法。那么我们先来看一下是哪里调用的，通过追踪processSelectedKeys()方法，最后会调用到NioEventLoop的processSelectedKey 方法：

```java
private void processSelectedKey(SelectionKey k, AbstractNioChannel ch) {
    final AbstractNioChannel.NioUnsafe unsafe = ch.unsafe();
    if (!k.isValid()) {
        final EventLoop eventLoop;
        try {
            eventLoop = ch.eventLoop();
        } catch (Throwable ignored) {
            // If the channel implementation throws an exception because there is no event loop, we ignore this
            // because we are only trying to determine if ch is registered to this event loop and thus has authority
            // to close ch.
            return;
        }
        // Only close ch if ch is still registerd to this EventLoop. ch could have deregistered from the event loop
        // and thus the SelectionKey could be cancelled as part of the deregistration process, but the channel is
        // still healthy and should not be closed.
        // See https://github.com/netty/netty/issues/5125
        if (eventLoop != this || eventLoop == null) {
            return;
        }
        // close the channel if the key is not valid anymore
        unsafe.close(unsafe.voidPromise());
        return;
    }
    try {
        int readyOps = k.readyOps();
        // We first need to call finishConnect() before try to trigger a read(...) or write(...) as otherwise
        // the NIO JDK channel implementation may throw a NotYetConnectedException.
        if ((readyOps & SelectionKey.OP_CONNECT) != 0) {
            // remove OP_CONNECT as otherwise Selector.select(..) will always return without blocking
            // See https://github.com/netty/netty/issues/924
            int ops = k.interestOps();
            ops &= ~SelectionKey.OP_CONNECT;
            k.interestOps(ops);
            unsafe.finishConnect();
        }
        // Process OP_WRITE first as we may be able to write some queued buffers and so free memory.
        if ((readyOps & SelectionKey.OP_WRITE) != 0) {
            // Call forceFlush which will also take care of clear the OP_WRITE once there is nothing left to write
            ch.unsafe().forceFlush();
        }
        // Also check for readOps of 0 to workaround possible JDK bug which may otherwise lead
        // to a spin loop
        if ((readyOps & (SelectionKey.OP_READ | SelectionKey.OP_ACCEPT)) != 0 || readyOps == 0) {
            unsafe.read();
            if (!ch.isOpen()) {
                // Connection already closed - no need to handle write.
                return;
            }
        }
    } catch (CancelledKeyException ignored) {
        unsafe.close(unsafe.voidPromise());
    }
}
```

这里可以看到熟悉的代码，当有链接进来的时候，便会走 unsafe.read()：这个里面就调用了doReadMessages

1. Connect, 即连接事件(TCP 连接), 对应于SelectionKey.OP_CONNECT.int值为16.
2. Accept, 即确认事件, 对应于SelectionKey.OP_ACCEPT.int值为8.
3. Read, 即读事件, 对应于SelectionKey.OP_READ, 表示 buffer 可读.int值为1
4. Write, 即写事件, 对应于SelectionKey.OP_WRITE, 表示 buffer 可写.int值为4

```java
public void read() {
   ...
   int localRead = doReadMessages(readBuf);
}
```

2.3 Netty 解决JDK 空轮询 Bug

各位应该早有耳闻臭名昭著的Java NIO epoll 的bug，它会导致Selector 空轮询，最终导致CPU 100%。官方声称在JDK1.6 版本的 update18 修复了该问题，但是直到JDK1.7 版本该问题仍旧存在，只不过该BUG 发生概率降低了一些而已，它并没有被根本解决。出现此Bug 是因为当 Selector 的轮询结果为空，也没有wakeup 或新消息处理，则发生空轮询，CPU 使用率达到100%。我们来看下这个问题在 issue 中的原始描述：

> This is an issue with poll (and epoll) on Linux. If a file descriptor for a connected socket is polled with a request event mask of 0, and if the connection is abruptly terminated (RST) then the poll wakes up with the POLLHUP (and maybe POLLERR) bit set in the returned event set. The implication of this behaviour is that Selector will wakeup and as the interest set for the SocketChannel is 0 it means there aren't any selected events and the select method returns 0.
>
> 具体解释为：在部分Linux 的2.6 的kernel 中，poll 和epoll 对于突然中断的连接socket 会对返回的eventSet 事件集合置为POLLHUP，也可能是POLLERR，eventSet 事件集合发生了变化，这就可能导致Selector 会被唤醒。这是与操作系统机制有关系的，JDK 虽然仅仅是一个兼容各个操作系统平台的软件，但很遗憾在JDK5 和JDK6 最初的版本中（严格意义上来将，JDK 部分版本都是），这个问题并没有解决，而将这个帽子抛给了操作系统方，这也就是这个bug 最终一直到2013 年才最终修复的原因。

在Netty 中最终的解决办法是：创建一个新的Selector，将可用事件重新注册到新的Selector 中来终止空轮询。前面我们有提到 select()方法解决了JDK 空轮询的Bug，它到底是如何解决的呢？下面我们来一探究竟，进入select()方法的源码：

```java
private void select(boolean oldWakenUp) throws IOException {
    Selector selector = this.selector;
    try {
        int selectCnt = 0;
        long currentTimeNanos = System.nanoTime();
        long selectDeadLineNanos = currentTimeNanos + delayNanos(currentTimeNanos);
        for (;;) {
            ...
            int selectedKeys = selector.select(timeoutMillis);
            selectCnt ++;
            if (selectedKeys != 0 || oldWakenUp || wakenUp.get() || hasTasks() || hasScheduledTasks()) {
                break;
            }
            if (Thread.interrupted()) {
                ...
                selectCnt = 1;
                break;
            }
            long time = System.nanoTime();
            if (time - TimeUnit.MILLISECONDS.toNanos(timeoutMillis) >= currentTimeNanos) {
                // timeoutMillis elapsed without anything selected.
                selectCnt = 1;
            } else if (SELECTOR_AUTO_REBUILD_THRESHOLD > 0 &&
                    selectCnt >= SELECTOR_AUTO_REBUILD_THRESHOLD) {
                rebuildSelector();
                selector = this.selector;
                // Select again to populate selectedKeys.
                selector.selectNow();
                selectCnt = 1;
                break;
            }
            currentTimeNanos = time;
        }
        ...
    }
}
```

从上面的代码中可以看出,Selector 每一次轮询都计数selectCnt++，开始轮询会计时赋值给timeoutMillis，轮询完成会计时赋值给time，这两个时间差会有一个时间差，而这个时间差就是每次轮询所消耗的时间。从上面的的逻辑看出，如果每次轮询消耗的时间为0，且重复次数超过512 次，则调用rebuildSelector()方法，即重构Selector。我们跟进到源码中就会发现：

```java
public void rebuildSelector() {
    final Selector oldSelector = selector;
    final Selector newSelector;
    
    newSelector = openSelector();
    int nChannels = 0;
    for (;;) {
        try {
            for (SelectionKey key: oldSelector.keys()) {
                Object a = key.attachment();
                try {
                    if (!key.isValid() || key.channel().keyFor(newSelector) != null) {
                        continue;
                    }
                    int interestOps = key.interestOps();
                    key.cancel();
                    SelectionKey newKey = key.channel().register(newSelector, interestOps, a);
                   ...
}
```

在rebuildSelector()方法中，主要做了三件事情：

1. 创建一个新的Selector。
2. 将原来Selector 中注册的事件全部取消。
3. 将可用事件重新注册到新的Selector 中，并激活。

2.4 NioServerSocktChannel 的创建

我们在分析客户端Channel 初始化过程时已经提到，Channel 是对Java 底层Socket 连接的抽象，并且知道了客户端Channel 的具体类型是NioSocketChannel，那么，自然的服务端的Channel 类型就是NioServerSocketChannel 了。通过前面的分析, 我们已经知道了,在客户端中，Channel 类型的指定是在初始化时，通过Bootstrap 的 channel() 方法设置的，服务端也是同样的方式。再看服务端代码，我们调用了ServerBootstarap 的 `channel(NioServerSocketChannel.class)` 方法，传的参数是NioServerSocketChannel.class 对象。如此，按照客户端代码同样的流程，我们可以确定NioServerSocketChannel 的实例化也是通过ReflectiveChannelFactory 工厂类来完成的，而ReflectiveChannelFactory 中的clazz 字段被赋值为NioServerSocketChannel.class，因此当调ReflectiveChannelFactory 的newChannel()方法，就能获取到一个NioServerSocketChannel 的实例。

最后我们也来总结一下:

1. ServerBootstrap 中的ChannelFactory 的实现类是 ReflectiveChannelFactory 类
2. 创建的Channel 具体类型是 NioServerSocketChannel

2.5 Channel 的实例化过程

其实就是调用ChannelFactory 的newChannel()方法，而实例化的Channel 具体类型就是初始化ServerBootstrap 时传给channel()方法的实参。因此，上面代码案例中的服务端ServerBootstrap, 创建的Channel实例就是NioServerSocketChannel 的实例。

2.6 服务端Channel 的初始化

我们来分析NioServerSocketChannel 的实例化过程，先看一下NioServerSocketChannel 的类层次结构图：

![image-20210106181927736](深入分析 Netty 源码(1).assets/image-20210106181927736.png)


首先，我们来看一下它的默认的构造器和NioSocketChannel 类似，构造器都是调用了newSocket 来打开一个Java 的NIO Socket， 不过需要注意的是，客户端的newSocket 调用的是openSocketChannel,而服务器端的newSocket 调用的是openServerSocketChannel. 顾名思义，一个是客户端的Java SocketChannel，一个是服务器端的Java ServerSocketChannel.

```java
private static ServerSocketChannel newSocket(SelectorProvider provider) {
    return provider.openServerSocketChannel();
}
public NioServerSocketChannel() {
    this(newSocket(DEFAULT_SELECTOR_PROVIDER));
}
```

接下来会调用重载的构造器：

```java
public NioServerSocketChannel(ServerSocketChannel channel) {
    super(null, channel, SelectionKey.OP_ACCEPT);
    config = new NioServerSocketChannelConfig(this, javaChannel().socket());
}
```

这个构造器中，调用父类构造器时，传入的参数是SelectionKey.OP_ACCEPT.作为对比，我们回想一下，在客户端的Channel初始化时，传入的参数是SelectionKey.OP_READ.我前面讲过，Java NIO 是一种Reactor模式，我们通过selector 来实现I/O 的多路复用，在一开始时，服务器端需要监听客户端的连接请求，因此在这里我们设置了SelectionKey. OP_ ACCEPT，即通知selector 我们对客户端的连接请求感兴趣. 接着和客户端的分析一下，会逐级地调用父类的构造器NioServerSocketChannel -&gt;AbstractNioMessageChannel -&gt; AbstractNioChannel -&gt; AbstractChannel. 同样的，在AbstractChannel中会实例化一个unsafe 和pipeline:

```java
protected AbstractChannel(Channel parent) {
    this.parent = parent;
    id = newId();
    unsafe = newUnsafe();
    pipeline = newChannelPipeline();
}
```

这里客户端的 unsafe 是一个 AbstractNioByteChannle#NioByteUnsafe 的实例，而在服务端时，因为 AbstractNioMessageChannel 重写了 newUnsafe 方法：

```java
protected AbstractNioUnsafe newUnsafe() {
    return new NioMessageUnsafe();
}
```

因此在服务器端，unsafe 字段其实是一个 AbstractNioMessageChannel.AbstractNioUnsafe 的实例。 总结一下，在 NioServerSocketChannel 实例化过程中，所需要做的工作：

1. 调用 NioServerSocketChannel.newSocket(DEFAULT_SELECTOR_PrOVIDER) 打来一个新的 Java NIO ServerSocketChannel 

2. AbstractChannel(Channel parent)中初始化 AbstractChannel 的属性：

  `parent` 属性置为 null

  `unsafe` 通过 newUnsafe() 实例化一个 unsafe 对象，它的类型是 AbstractNioMessageChannel#AbstractNioUnsafe 内部类

  `pipeline` 是 new DefaultChannelPipeline(this) 新创建的实例 

3. AbstractNioChannel 中的属性：

   `ch` 被设置为 Java ServerSocketChannel, 即 NioServerSocketChannel#newSocket 返回的 Java NIO ServerSocketChannel

   `readInterest0p` 被设置为 SelectionKey. OP_ACCEРT

   `ch` 被配置为非阻塞的 ch.configureBlocking (false)

4. NioServerSocketChannel 中的属性： 

   ServerSocketChannelConfig config = new NioServerSocketChannelConfig(this, javaChannel (). socket())

![image-20210106181927736](深入分析 Netty 源码(1).assets/Service-NIO.png)

2.7 ChannelPipeline 初始化

服务器端和客户端的ChannelPipeline的初始化一致，因此就不再单独分析了

2.8 Channel的注册

服务器端和客户端的Channel的注册过程一致，因此就不再单独分析了

2.9 Handler 的添加过程

服务器端的 handler 的添加过程和客户端的有点区别，和 EventLoopGroup 一样，服务器端的handler也有两个，一个是通过handler() 方法设置handler 字段，另一个是通过childHandler()设置childHandler字段。通过前面的bossGroup 和workerGroup 的分析，其实我们在这里可以大胆地猜测:handler字段与accept 过程有关，即这个handler 负责处理客户端的连接请求;而childHandler就是负责和客户端的连接的I0交互. 那么实际上是不是这样的呢?

我们继续用代码来证明。在前面章节我们已经了解ServerBootstrap 重写了init()方法，在这个方法中也添加了handler：

```java
void init(Channel channel) throws Exception {
    ChannelPipeline p = channel.pipeline();
    final EventLoopGroup currentChildGroup = childGroup;
    final ChannelHandler currentChildHandler = childHandler;
    ...
    p.addLast(new ChannelInitializer<Channel>() {
        @Override
        public void initChannel(Channel ch) throws Exception {
            final ChannelPipeline pipeline = ch.pipeline();
            ChannelHandler handler = config.handler();
            if (handler != null) {
                pipeline.addLast(handler);
            }
            ch.eventLoop().execute(new Runnable() {
                @Override
                public void run() {
                    pipeline.addLast(new ServerBootstrapAcceptor(
                            currentChildGroup, currentChildHandler, currentChildOptions, currentChildAttrs));
                }
            });
        }
    });
}
```

在上面代码的initChannel()方法中，首先通过handler()方法获取一个handler，如果获取的handler 不为空，则添加到pipeline 中。然后接着，添加了一个ServerBootstrapAcceptor 的实例。那么这里的handler()方法返回的是哪个对象呢? 其实它返回的是handler 字段，而这个字段就是我们在服务器端的启动代码中设置的：那么这个时候, pipeline 中的handler 情况如下:

![image-20210106181435833](深入分析 Netty 源码(1).assets/image-20210106181435833.png)

根据我们原来分析客户端的经验，我们指定，当channel 绑定到eventLoop后(在这里是NioServerSocketChannel绑定到bossGroup) 中时，会在pipeline中发出fireChannelRegistered事件，接着就会触发ChannelInitializer. initChannel方法的调用.因此在绑定完成后，此时的pipeline 的内如下: 

![image-20210106181450375](深入分析 Netty 源码(1).assets/image-20210106181450375.png)

前面我们在分析bossGroup和workerGroup时，已经知道了在ServerBootstrapAcceptor. channelRead中会为新建的Channel 设置handler 并注册到一个eventLoop中，即:

```java
public void channelRead(ChannelHandlerContext ctx, Object msg) {
	final Channel child = (Channel) msg;
    child.pipeline().addLast(childHandler);
    ...
        childGroup.register(child).addListener(...);
 ...
}
```

而这里的 childHandler 就是在服务器端启动代码中设置的 handler：

```java
...
.childHandler(new ChannelInitializer<SocketChannel>() {
    @Override
    protected void initChannel(SocketChannel socketChannel) throws Exception {
        ...
    }
})
```

后续的步骤就没有什么好说的了，当这个客户端连接Channel注册后，就会触发ChannelInitializer.initChannel方法的调用。 最后我们来总结一下服务器端的handler 与childHandler的区别与联系:

1. 在服务器NioServerSocketChannel 的pipeline 中添加的是handler 与ServerBootstrapAcceptor. 
2. 当有新的客户端连接请求时，ServerBootstrapAcceptor.channelRead 中负责新建此连接的NioSocketChannel 并添加childHandler到NioSocketChannel 对应的pipeline 中，并将此channel 绑定到workerGroup 中的某个eventLoop中. 
3. handler 是在accept 阶段起作用，它处理客户端的连接请求. 
4. childHandler 是在客户端连接建立以后起作用，它负责客户端连接的IO交互.


![img](深入分析 Netty 源码(1).assets/20200606152510118.png)

------

