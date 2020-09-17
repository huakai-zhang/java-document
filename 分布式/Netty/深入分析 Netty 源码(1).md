## 客户端 BootStrap

BootStrap 是 Netty 提供的一个便利的工厂类，可以通过它来完成 Netty 的客户端或服务端的 Netty 初始化。 从客户端方面启动 Netty：

```java
Bootstrap bootstrap = new Bootstrap();
bootstrap.group(workerGroup)
.channel(NioSocketChannel.class)
.option(ChannelOption.SO_KEEPALIVE, true)
.handler(new ChannelInitializer<SocketChannel>() {
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

Netty 客户端初始化时所需的所有内容：

1. EventLoopGroup：不论是服务端还是客户端，都必须指定 EventLoopGroup，在例子中，指定了 NioEventLoopGroup，表示一个 NIO 的 EventLoopGroup 
2. ChannelType：指定 Channel 的类型，因为是客户端，因此使用了 NioSocketChannel 
3. Handler：设置数据的处理器

### NioSocketChannel 的初始化过程


在 Netty 中，Channel 是一个 Socket 的抽象，它为用户提供了关于 Socket 状态（是否是连接还是断开）以及对 Socket 的读写等操作。每当 Netty 建立了一个连接后，都会有一个对应的 Channel 实例。 NioSocketChanel 的类层次结构如下： ![img](https://img-blog.csdnimg.cn/20200605202613623.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

### ChannelFactory 和 Channel 类型的确定

除了 TCP 协议以外，Netty 还支持很多其他的连接协议，并且每种协议还有 NIO（非阻塞 IO）和 OIO（Old-IO，即传统的阻塞 IO）版本的区别，不同协议不同的阻塞类型的连接都有不同的 Channel 类型与之对应下面常用的 Channel 类型： NioSocketChannel，异步的客户端TCP Socket 连接 NioServerSocketChannel, 异步的服务端 TCP Socket 连接 NioDatagramChannel, 异步的 UDP 连接 NioSctpChannel, 异步的客户端 Sctp 连接 NioSctpServerChannel, 异步的 Sctp 服务器端连接 OioSocketChannel, 同步的客户端TCP Socket 连接 OioServerSocketChannel, 同步的服务端 TCP Socket 连接 OioDatagramChannel, 同步的 UDP 连接 OioSctpChannel, 同步的客户端 Sctp 连接 OioSctpServerChannel, 同步的 Sctp 服务器端连接

通过 channel() 方法的调用设置所需要的 Channel 类型。传入 NioSocketChannel.class，这个方法其实就是初始化一个 ReflectiveChannelFactory：

```java
public B channel(Class<? extends C> channelClass) {
    if (channelClass == null) {
        throw new NullPointerException("channelClass");
    }
    return channelFactory(new ReflectiveChannelFactory<C>(channelClass));
}
```

而 ReflectiveChannelFactory 实现了 ChannelFactory 接口，它提供了唯一的方法，即 newChannel. ChannelFactory, 顾名思义，就是产生Channel的工厂类.进入到ReflectiveChannelFactory. newChannel中，我们看到其实现代码如下:

```java
public T newChannel() {
    return clazz.newInstance();
}
```

根据上面代码的提示，可以确定:

1. Bootstrap 中的ChannelFactory 的实现是ReflectiveChannelFactory 
2. 生成的Channel 的具体类型是NioSocketChannel Channel的实例化过程，其实就是调用的ChannelFactory. newChannel方法，而实例化的Channel的具体的类型又是和在初始化Bootstrap 时传入的channel() 方法的参数相关.因此对于我们这个例子中的客户端的Bootstrap而言，生成的的Channel实例就是NioSocketChannel.

### Channel 的实例化

ChannelFactory. newChannel()方法在哪里调用呢?继续跟踪，发现其调用链是: Bootstrap.connect-> Bootstrap.doResolveAndConnect->AbstractBootstrap.initAndRegister 在AbstractBootstrap.initAndRegister中调用了channelFactory().newChannel()来获取一个新的NioSocketChannel 实例，其源码如下:

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
protected AbstractNioChannel(Channel parent, SelectableChannel ch, int readInterestOp) {
    super(parent);
    this.ch = ch;
    this.readInterestOp = readInterestOp;
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

到这里，一个完整的NioSocketChannel就初始化完成了，稍微总结一下构造一个NioSocketChannel所需要做的工作:

1. 调用NioSocketChannel.newSocket (DEFAULT_  *SELECTOR*  PROVIDER) 打开一个新的Java NIO SocketChannel 
2. AbstractChanne l (Channel parent) 中 初始化AbstractChannel 的属性: parent属性置为null unsafe通过newUnsafe() 实例化一个unsafe 对象，它的类型是AbstractNioByteChannel.NioByteUnsafe内部类 pipeline是new DefaultChannelPipeline(this)新创建的实例.这里体现了: Each channel has its own pipeline and it is created automatically when a new channel is created. 
<li>AbstractNioChannel 中的属性: SelectableChannel，ch 被设置为 Java SocketChannel,即NioSocketChannel.newSocket 返回 Java NIO SocketChannel. readInterestOp 被设置成 SelectionKey. OP_ READ SelectableChannel，ch 被配置为非阻塞 ch.configureBlocking(false)</li> 
<li>NioSocketChannel 中的属性： SocketChannelConfig config = newNioSocketChannelConfig(this,socket. socket())</li>

### 关于 unsafe 字段的初始化

在实例化NioSocketChannel的过程中，会在父类AbstractChannel 的构造器中，调用newUnsafe()来获取一个unsafe 实例. 其实unsafe封装了对Java底层Socket 的操作，因此实际上是沟通Netty 上层和Java底层的重要的桥梁. 那么看一下Unsafe接口所提供的方法:

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

NioSocketChannel.newUnsafe 方法会返回一个 NioSocketChannelUnsafe 实例，可以确定在实例化的 NioSocketChannel 中的 unsafe 字段，其实是一个 NioSocketChannelUnsafe 的实例。

### 关于 pipeline 的初始化

根据Each channel has its own pipeline and it is created automatically when a new channel is created.可知在实例化一个Channel 时，必然伴随着实例化一个ChannelPipeline. 而确实在AbstractChannel 的构造器看到了pipeline 字段被初始化为DefaultChannelPipeline的实例：

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



调用DefaultChannelPipeline的构造器，传入了一个channel, 而这个channel 其实就是实例化的NioSocketChannel, DefaultChannelPipeline会将这个NioSocketChannel 对象保存在channel 字段中. DefaultChannelPipeline 中，还有两个特殊的字段，即head和tail， 而这两个字段是一个双向链表的头和尾.其实在DefaultChannelPipeline中，维护了一个以AbstractChannelHandlerContext为节点的双向链表，这个链表是Netty 实现Pipeline 机制的关键.关于DefaultChannelPipeline 中的双向链表以及它所起的作用。 先看看HeadContext的继承层次结构如下所示: ![img](https://img-blog.csdnimg.cn/20200605214504426.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) TailContext的继承层次结构如下所示： ![img](https://img-blog.csdnimg.cn/20200605214917623.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) 链表中 head 是一个 ChannelOutboundHandler，而 tail 则是一个 ChannelInboundHandler。 接着看一下HeadContext 的构造器：

```java
HeadContext(DefaultChannelPipeline pipeline) {
    super(pipeline, null, HEAD_NAME, false, true);
    unsafe = pipeline.channel().unsafe();
    setAddComplete();
}
```

它调用了父类 AbstractChannelHandlerContext 的构造器，并传入参数 inbound = false，outbound = true。 TailContext 的构造器与 HeadContext 的相反，它调用了父类 AbstractChannelHandlerContext的构造器，并传入参数inbound = true, outbound = false。 即header 是一个outboundHandler, 而tail 是一个inboundHandler,关于这一点，要特别注意，因为在分析到Netty Pipeline 时，会反复用到inbound 和outbound 这两个属性。

### 关于EventLoop的初始化


回到最开始的ChatClient.java代码中，在一开始就实例化了一个NioEventLoopGroup对象，因此就从它的构造器中追踪一下EventLoop 的初始化过程。首先来看一下NioEventLoopGroup的类继承层次: ![img](https://img-blog.csdnimg.cn/20200605215831162.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) NioEventLoop 有几个重载的构造器，最终调用了父类 MultithreadEventLoopGroup 构造器：

```java
protected MultithreadEventLoopGroup(int nThreads, Executor executor, Object... args) {
    super(nThreads == 0 ? DEFAULT_EVENT_LOOP_THREADS : nThreads, executor, args);
}
```

其中有一点有意思的地方是，如果传入的线程数nThreads 是0，那么Netty 会设置默认的线程数DEFAULT_  *EVENT*   *L0OP THREADS，在静态代码块中，会首先确定DEFAULT EVENT*  L0OP_ THREADS 的值:

```java
static {
    DEFAULT_EVENT_LOOP_THREADS = Math.max(1, SystemPropertyUtil.getInt(
            "io.netty.eventLoopThreads", Runtime.getRuntime().availableProcessors() * 2));
    if (logger.isDebugEnabled()) {
        logger.debug("-Dio.netty.eventLoopThreads: {}", DEFAULT_EVENT_LOOP_THREADS);
    }
}
```

Netty会首先从系统属性中获取io. netty. eventLoopThreads的值，如果没有设置它的话，那么就返回默认值:处理器核心数* 2. 回到 MultithreadEventLoopGroup 构造器中，这个构造器会继续调用父类 MultithreadEventExecutorGroup 的构造器:

```java
protected MultithreadEventExecutorGroup(int nThreads, Executor executor,
                                        EventExecutorChooserFactory chooserFactory, Object... args) {
    children = new EventExecutor[nThreads];
    for (int i = 0; i < nThreads; i ++) {
         children[i] = newChild(executor, args);
    }
    chooser = chooserFactory.newChooser(children);
    ...
}
```

根据代码，很清楚 MultithreadEventExecutorGroup 中的处理逻辑：

1. 创建一个大小为 nThreads 的 SingleThreadEventExcutor 数组 
2. 根据 nThreads 的大小，创建不同的 Chooser , 即如果 nThreads 是2的幂，则使用 PowerOfTwoEventExecutorChooser，反之使用GenericEventExecutorChooser. 不论使用哪个Chooser，它们的功能都是一样的，即从children 数组中选出一个合适的EventExecutor 实例.

```java
public EventExecutorChooser newChooser(EventExecutor[] executors) {
    if (isPowerOfTwo(executors.length)) {
        return new PowerOfTowEventExecutorChooser(executors);
    } else {
        return new GenericEventExecutorChooser(executors);
    }
}
```

1. 调用newChhild 方法初始化children 数组.根据上面的代码，我们知道，MultithreadEventExecutorGroup 内部维护了一个 EventExecutor数组，Netty 的EventLoopGroup 的实现机制其实就建立在MultithreadEventExecutorGroup之上，每当Netty 需要一个EventLoop 时，会调用next()方法获取一个可用的EventLoop.

上面代码最后一部分是 newChild 方法，这是一个抽象方法，它的任务是实例化 EventLoop 对象，跟踪代码可以发现，这个方法在 NioEventLoopGroup 类中实现了：

```java
protected EventLoop newChild(Executor executor, Object... args) throws Exception {
    return new NioEventLoop(this, executor, (SelectorProvider) args[0],
        ((SelectStrategyFactory) args[1]).newSelectStrategy(), (RejectedExecutionHandler) args[2]);
}
```

其实就是实例化一个NioEventLoop对象，然后返回它. 最后总结一下整个EventLoopGroup 的初始化过程:

1. EventLoopGroup(其实是MultithreadEventExecutorGroup) 内部维护一个类型为EventExecutor children数组，其大小是nThreads, 这样就构成了一个线程池 
2. 如果我们在实例化NioEventLoopGroup 时，如果指定线程池大小，则nThreads 就是指定的值，质之是处理器核心数* 2 
3. MultithreadEventExecutorGroup中会调用newChild抽象方法来初始化children 数组 
4. 抽象方法newChild 是在NioEventLoopGroup 中实现的，它返回一个NioEventLoop实例. 
5. NioEventLoop属性: SelectorProvider provider 属性: NioEventLoopGroup 构造器中通过SelectorProvider. provider()获取一个SelectorProvider Selector selector 属性: NioEventLoop 构造器中通过调用通过selector = provider. openSelector()获取一个selector 对象.

### Channel 的注册过程

channel 会在 Bootstrap.initAndRegister 中初始化，但是这个方法还会将初始化好的 Channel 注册到 EventGroup 中，回顾一下 AbstractBootStrap.initAndRegister 方法：

```java
final ChannelFuture initAndRegister() {
	...
    channel = channelFactory.newChannel();
    init(channel);
    ChannelFuture regFuture = config().group().register(channel);
    ...
    return regFuture;
}
```

当 Channel 初始化后，会紧接着调用 group.register()方法来注册Channel， 其调用链如下: AbstractBootstrap. initAndRegister-&gt;MultithreadEventLoopGroup. register-&gt;SingleThreadEventLoop.register -&gt; AbstractChannel$AbstractUnsafe.register

通过跟踪调用链，最终发现是调用到了unsafe的register 方法，那么接下来就仔细看一下AbstractChannel$AbstractUnsafe. register方法中到底做了什么:

```java
public final void register(EventLoop eventLoop, final ChannelPromise promise) {
    AbstractChannel.this.eventLoop = eventLoop;
    register0(promise);
}
```

首先，将eventLoop赋值给Channel 的eventLoop 属性，而我们知道这个eventLoop 对象其实是MultithreadEventLoopGroup.next()方法获取的，根据我们前面的小节中，我们可以确定next()方法返回的eventLoop 对象是NioEventLoop 实例.register方法接着调用了register0 方法:

```java
private void register0(ChannelPromise promise) {
    try {
        boolean firstRegistration = neverRegistered;
        doRegister();
        neverRegistered = false;
        registered = true;
        pipeline.invokeHandlerAddedIfNeeded();
        safeSetSuccess(promise);
        pipeline.fireChannelRegistered();
        if (isActive()) {
            if (firstRegistration) {
                pipeline.fireChannelActive();
            } else if (config().isAutoRead()) {
                beginRead();
            }
        }
    } catch (Throwable t) {
        closeForcibly();
        closeFuture.setClosed();
        safeSetFailure(promise, t);
    }
}
```

register0 有调用了 AbstractNioChannel.goRegister：

```java
protected void doRegister() throws Exception {
   selectionKey = javaChannel().register(eventLoop().selector, 0, this);
}
```

javaChannel() 这个方法在前面已经知道了，它返回的是一个Java NIO SocketChannel,这里将这个SocketChannel 注册到与eventLoop 关联的selector上了. 总结一下Channel 的注册过程:

1. 首先AbstractBootstrap.initAndRegister 中, 通过group().register(channel),调用MultithreadEventLoopGroup.register 方法 
2. MultithreadEventLoopGroup.register 中，通过 next() 获取一个可用的 SingleThreadEventLoop, 然后调用它的 register 
3. 在 SingleThreadEventLoop.register 中 , 通过 channel.unsafe().register(this,promise) 来获取 channel 的 unsafe() 底层操作对象，然后调用它的 register 
4. 在 AbstractUnsafe.register 方法中，调用 register0 方法注册 Channel 
5. 在 AbstractUnsafe.register0 方法中，调用 AbstarctNioChannel.doRegister 方法 
6. AbstarctNIioChannel.doRegister 方法通过 javaChannel().register(eventLoop().selector, 0, this)将 Channel 对应的 Java NIO SocketChannel 注册到一个 eventLoop 的 Selector 中，并将当前 Channel 作为 attachment

总的来说，Channel 注册过程所做的工作就是将Channel与对应的EventLoop 关联，因此这也体现了，在Netty中，每个Channel 都会关联一个特定的EventLoop， 并且这个Channel中的所有IO操作都是在这个EventLoop 中执行的;当关联好Channel 和EventLoop 后，会继续调用底层的 Java NIO SocketChannel 的 register 方法，将底层的 Java NIO SocketChannel 注册到指定的 selector 中，通过这两步，就完成了 Netty Channel 的注册过程。

### Handler 的添加过程

Netty的一个强大和灵活之处就是基于Pipeline的自定义handler 机制.基于此，我们可以像添加插件一样自由组合各种各样的handler 来完成业务逻辑.例如我们需要处理HTTP数据,那么就可以在pipeline 前添加一个Http的编解码的Handler， 然后接着添加我们自己的业务逻辑的handler， 这样网络上的数据流就向通过一个管道一样，从不同的handler 中流过并进行编解码，最终在到达我们自定义的handler 中. 既然说到这里，有些同学肯定会好奇，既然这个pipeline 机制是这么的强大，那么它是怎么实现的呢?在这里我还不打算详细讲解，在这一小节中，我们从简单的入手，展示一下我们自定义的handler 是如何以及何时添加到ChannelPipeline 中的. 首先让我们看一下如下的代码片段:

```java
.handler(new ChannelInitializer<SocketChannel>() {
                @Override
                public void initChannel(SocketChannel ch) throws Exception {
                    ch.pipeline().addLast(new IMDecoder());
                    ch.pipeline().addLast(new IMEncoder());
                    ch.pipeline().addLast(clientHandler);
                }
            });
```

这个代码片段就是实现了handler 的添加功能.我们看到，Bootstrap.handler方法接收一个ChannelHandler，而我们传递的是一个派生于 ChannelInitializer 的匿名类，它正好也实现了ChannelHandler 接口.我们来看一下，Channellnitializer 类内到底有什么玄机:

```java
public abstract class ChannelInitializer<C extends Channel> extends ChannelInboundHandlerAdapter {

    private static final InternalLogger logger = InternalLoggerFactory.getInstance(ChannelInitializer.class);
    private final ConcurrentMap<ChannelHandlerContext, Boolean> initMap = PlatformDependent.newConcurrentHashMap();

    protected abstract void initChannel(C ch) throws Exception;

    @Override
    @SuppressWarnings("unchecked")
    public final void channelRegistered(ChannelHandlerContext ctx) throws Exception {
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




ChannelInitializer是一个抽象类，它有一个抽象的方法initChannel, 正是实现了这个方法，并在这个方法中添加的自定义的handler的.那么initChannel 是哪里被调用的呢? 答案是ChannelInitializer. channelRegistered方法中. 我们来关注一下(channelRegistered方法，从上面的源码中，我们可以看到，在channelRegistered方法中，会调用initChannel方法，将自定义的handler添加到ChannelPipeline中，然后调用ctx. pipeline(). remove(this)将自己从ChannelPipeline中删除.上面的分析过程，可以用如下图片展示: 一开始，ChannelPipeline 中只有三个handler，head, tail和我们添加的ChannelInitializer. ![img](https://img-blog.csdnimg.cn/20200606111651920.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) 接着 initChannel 方法调用后，添加了自定义的 handler： ![img](https://img-blog.csdnimg.cn/20200606111727186.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) 最后将 ChannelInitializer 删除： ![img](https://img-blog.csdnimg.cn/20200606111840248.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

### 客户端连接分析

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

上面的代码中有一个关键的地方，即final AbstractChannelHandlerContext next = findContextOutbound()，这里调用findContextOutbound 方法，从DefaultChannelPipeline内的双向链表的tail开始，不断向前寻找第一个outbound 为true 的 AbstractChannelHandlerContext，然后调用它的invokeConnect 方法，其代码如下:

```java
private void invokeConnect(SocketAddress remoteAddress, SocketAddress localAddress, ChannelPromise promise) {
	...
    ((ChannelOutboundHandler) handler()).connect(this, remoteAddress, localAddress, promise);
}
```

前面我们提到，在 DefaultChannelPipeline 的构造器中，会实例化两个对象: head和tail,并形成了双向链表的头和尾，head 是HeadContext 的实例，它实现了ChannelOutboundHandler接口，并且它的outbound字段为true. 因此在findContextOutbound中，找到的AbstractChannelHandlerContext对象其实就是head. 进而在invokeConnect 方法中，我们向上转换为ChannelOutboundHandler 就是没问题的了. 而又因为HeadContext 重写了connect 方法，因此实际上调用的是HeadContext. connect.我们接着跟踪到HeadContext. connect，其代码如下:

```java
public void connect(
        ChannelHandlerContext ctx,
        SocketAddress remoteAddress, SocketAddress localAddress,
        ChannelPromise promise) throws Exception {
    unsafe.connect(remoteAddress, localAddress, promise);
}
```

这个connect方法很简单，仅仅调用了unsafe 的connect 方法.而unsafe 又是什么呢? 回顾一下HeadContext的构造器，我们发现unsafe 是pipeline.channel().unsafe()返回的是Channel 的unsafe字段，在这这里，我们已经知道了，其实是AbstractNioByteChannel.NioByteUnsafe内部类.兜兜转转了一大圈，我们找到了创建Socket连接的关键代码. 进行跟踪NioByteUnsafe -&gt; AbstractNioUnsafe.connect:

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


上面的代码不用多说，首先是获取Java NIO SocketChannel,从NioSocketChannel.newSocket返回的SocketChannel 对象;然后是调用SocketChannel. connect方法完成Java NIO层面上的Socket 的连接. 最后，上面的代码流程可以用如下时序图直观地展示: ![img](https://img-blog.csdnimg.cn/20200606120148330.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

## 服务端 ServerBootstrap

服务端的启动代码：

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

1. EventLoopGroup:不论是服务器端还是客户端，都必须指定EventLoopGroup. 在这个例子中，指定了NioEventLoopGroup， 表示一个NIO的EventLoopGroup， 不过服务器端需要指定两个EventLoopGroup，一个是bossGroup, 用于处理客户端的连接请求;另一个是workerGroup，用于处理与各个客户端连接的IO操作. 
2. ChannelType: 指定Channel 的类型.因为是服务器端，因此使用了NioServerSocketChannel. 
3. Handler:设置数据的处理器.

### Channel 的初始化过程

我们在分析客户端的 Channel 初始化过程时，已经提到，Channel 是对Java 底层Socket 连接的抽象,并且知道了客户端的Channel 的具体类型是NioSocketChannel， 那么自然的，服务器端的Channel 类型就是NioServerSocketChannel了 .

### Channel 类型的确定

同样的分析套路，已经知道了，在客户端中，Channel 的类型其实是在初始化时，通过Bootstrap. channel()方法设置的，服务器端自然也不例外. 在服务器端，我们调用了ServerBootstarap.channel(NioServerSocketChannel.class)， 传递了一个NioServerSocketChannel Class 对象.这样的话，按照和分析客户端代码一样的流程，我们就可以确定，NioServerSocketChannel的实例化是通过ReflectiveChannelFactory工厂类来完成的，而ReflectiveChannelFactory中的clazz 字段被设置为了NioServerSocketChannel.class，因此当调用ReflectiveChannelFactory.newChannel()时:

```java
public T newChannel() {
    return clazz.newInstance();
}
```

就获取到了一个NioServerSocketChannel的实例. 最后我们也来总结一下:

1. ServerBootstrap 中的 ChannelFactory 的实现是 ReflectiveChannelFactory 
2. 生成的Channel 的具体类型是 NioServerSocketChannel.

Channel的实例化过程，其实就是调用的ChannelFactory.newChannel方法，而实例化的Channel的具体的类型又是和在初始化ServerBootstrap时传入的channel() 方法的参数相关。因此对于我们这个例子中的服务器端的ServerBootstrap 而言，生成的的Channel 实例就是NioServerSocketChannel.

### NioServerSocketChannel 的实例化过程


首先还是来看一下NioServerSocketChannel的实例化过程. 下面是NioServerSocketChannel的类层次结构图: ![img](https://img-blog.csdnimg.cn/20200606140514547.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) 首先，我们来看一下它的默认的构造器和NioSocketChannel 类似，构造器都是调用了newSocket 来打开一个Java 的NIO Socket， 不过需要注意的是，客户端的newSocket 调用的是openSocketChannel,而服务器端的newSocket 调用的是openServerSocketChannel. 顾名思义，一个是客户端的Java SocketChannel，一个是服务器端的Java ServerSocketChannel.

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
2. AbstractChannel(Channel parent)中初始化 AbstractChannel 的属性： parent 属性置为 null unsafe 通过 newUnsafe() 实例化一个 unsafe 对象，它的类型是 AbstractNioMessageChannel#AbstractNioUnsafe 内部类 pipeline 是 new DefaultChannelPipeline(this) 新创建的实例 
<li>AbstractNioChannel 中的属性： SelectableChannel ch 被设置为 Java ServerSocketChannel, 即 NioServerSocketChannel#newSocket 返回的 Java NIO ServerSocketChannel. readInterest0p 被设置为 SelectionKey. OP_ACCEРT SelectableChannel ch 被配置为非阻塞的 ch.configureBlocking (false)</li> 
<li>NioServerSocketChannel 中的属性： ServerSocketChannelConfig config = new NioServerSocketChannelConfig(this, javaChannel (). socket())</li>

### ChannelPipeline 初始化

服务器端和客户端的ChannelPipeline的初始化一致，因此就不再单独分析了.

### Channel的注册

服务器端和客户端的Channel的注册过程一致，因此就不再单独分析了.

### 关于bossGroup 与workerGroup


在客户端的时候，我们只提供了一个EventLoopGroup 对象，而在服务器端的初始化时，我们设置了两个EventLoopGroup，一个是bossGroup, 另一个是workerGroup. 那么这两个EventLoopGroup 都是干什么用的呢?其实呢，bossGroup是用于服务端的accept 的，即用于处理客户端的连接请求，我们可以把Netty比作一个饭店，bossGroup 就像一个像一个前台接待，当客户来到饭店吃时，接待员就会引导顾客就坐，为顾客端茶送水等.而workerGroup，其实就是实际上干活的啦，它们负责客户端连接通道的IO操作:当接待员招待好顾客后，就可以稍做休息，而此时后厨里的厨师们(workerGroup)就开始忙碌地准备饭菜了. 关于bossGroup与workerGroup 的关系，我们可以用如下图来展示: ![img](https://img-blog.csdnimg.cn/20200606142921430.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) 首先，服务器端bossGroup不断地监听是否有客户端的连接，当发现有一个新的客户端连接到来时，bossGroup就会为此连接初始化各项资源，然后从workerGroup中选出一个EventLoop 绑定到此客户端连接中，那么接下来的服务器与客户端的交互过程就全部在此分配的EventLoop 中了. 首先在ServerBootstrap 初始化时，调用了b.group (bossGroup,workerGroup) 设置了两个EventlLoopGroup，我们跟踪进去看一下:

```java
public ServerBootstrap group(EventLoopGroup parentGroup, EventLoopGroup childGroup) {
    super.group(parentGroup);
    this.childGroup = childGroup;
    return this;
}
```

显然，这个方法初始化了两个字段，一个是group=parentGroup，它是在super.group(parentGroup)中初始化的，另一个是childGroup = childGroup.接着我们启动程序调用了b.bind 方法来监听一个本地端口。bind方法会触发如下的调用链: AbstractBootstrap.bind -&gt; AbstractBootstrap.doBind -&gt;AbstractBootstrap.initAndRegister 源码看到到这里为止，AbstractBootstrap.initAndRegister已经是我们的老朋友了，我们在分析客户端程序时，和它打过很多交到了，现在再来回顾一下这个方法吧:

```java
final ChannelFuture initAndRegister() {
    channel = channelFactory.newChannel();
    init(channel);
    ChannelFuture regFuture = config().group().register(channel);
    return regFuture;
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

init方法在 ServerBootstrap 中重写了，从上面的代码片段中我们看到，它为pipeline 中添加了一个ChannelInitializer,而这个 ChannelInitializer 中添加了一个关键的 ServerBootstrapAcceptor handler.关于handler 的添加与初始化的过程，我们留待下一小节中分析，我们现在关注一下ServerBootstrapAcceptor类. ServerBootstrapAcceptor 中重写了 channelRead 方法，主要代码如下:

```java
public void channelRead(ChannelHandlerContext ctx, Object msg) {
	final Channel child = (Channel) msg;
    child.pipeline().addLast(childHandler);
    ...
        childGroup.register(child).addListener(...);
 ...
}
```

ServerBootstrapAcceptor 中的 childGroup 是构造此对象时传入的 currentChildGroup, 即我们的workerGroup, 而 Channel 是一个 NioSocketChannel 的实例, 因此这里的 childGroup.register 就是将workerGroup 中的某个EventLoop 和NioSocketChannel 关联了.既然这样，那么现在的问题是，ServerBootstrapAcceptor.channelRead 方法是怎么被调用的呢？其实当一个 client 连接到 server 时，Java 底层的 NIO ServerSocketChannel 会有一个 SelectionKey.OP_ACCEPT 就绪事件，接着就会调用到 NioServerSocketChannel.doReadMessages:

```java
protected int doReadMessages(List<Object> buf) throws Exception {
    SocketChannel ch = javaChannel().accept();
    buf.add(new NioSocketChannel(this, ch));
    return 1;
}
```

在 doReadMessages 中，通过 javaChannel().accept() 获取到客户端新连接的 SocketChannel，接着就实例化一个 NioSocketChannel，并传入 NioServerSocketChannel 对象（即 this），由此可知，创建的这个 NioSocketChannel 的父Channel 就是 NioServerSocketChannel 实例。 接下来就经由 Netty 的 ChannelPipeline机制，将读取事件逐级发送到各个handler中，于是就会触发前面提到的 ServerBootstrapAcceptor.channelRead 方法。

### Handler 的添加过程

服务器端的handler的添加过程和客户端的有点区别，和EventLoopGroup 一样，服务器端的handler也有两个，一个是通过handler() 方法设置handler 字段，另一个是通过chi ldHandler()设置childHandler字段.通过前面的bossGroup 和workerGroup 的分析，其实我们在这里可以大胆地猜测:handler字段与accept 过程有关，即这个handler 负责处理客户端的连接请求;而chi ldHandler就是负责和客户端的连接的I0交互. 那么实际上是不是这样的呢?来，我们继续通过代码证明. 在关于bossGroup 与workerGroup 小节中，我们提到，ServerBootstrap 重写了init方法，在这个方法中添加了handler。init方法中的 initChannel 方法，首先通过 handler() 方法获取一个 handler，如果获取的 handler 不为空,则添加到pipeline 中，然后接着，添加了一个ServerBootstrapAcceptor 实例.那么这里handler()方法返回的是哪个对象呢?其实它返回的是handler 字段，而这个字段就是我们在服务器端的启动代码中设置的:

```java
b.group(bossGroup, workerGroup);
```



这个时候，pipeline 中的 handler 情况如下： ![img](https://img-blog.csdnimg.cn/20200606151756758.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) 根据我们原来分析客户端的经验，我们指定，当channel 绑定到eventLoop后(在这里是NioServerSocketChannel绑定到bossGroup) 中时，会在pipeline中发出fireChannelRegistered事件，接着就会触发ChannelInitializer. initChannel方法的调用.因此在绑定完成后，此时的pipeline 的内如下: ![img](https://img-blog.csdnimg.cn/20200606151847666.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) 前面我们在分析bossGroup和workerGroup时，已经知道了在ServerBootstrapAcceptor. channelRead中会为新建的Channel 设置handler 并注册到一个eventLoop中，即:

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


![img](https://img-blog.csdnimg.cn/20200606152510118.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

