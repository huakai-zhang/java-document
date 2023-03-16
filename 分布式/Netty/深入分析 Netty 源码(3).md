# 1 Pipeline 设计原理

## 1.1 Channel 与 ChannelPipeline

在 Netty 中每个 Channel 都有且仅有一个 ChannelPipeline 与之对应，它们的组成关系如下： 

![image-20210107180635605](深入分析 Netty 源码(3).assets/image-20210107180635605.png)

通过上图我们可以看到，一个Channel 包含了一个ChannelPipeline， 而ChannelPipeline 中又维护了一个由ChannelHandlerContext组成的双向链表。这个链表的头是HeadContext，链表的尾是TailContext，并且每个ChannelHandlerContext 中又关联着一个 ChannelHandler。

上面的图示给了我们一个对ChannelPipeline 的直观认识，但是实际上Netty 实现的Channel是否真的是这样的呢?我们继续用源码说话。在前我们已经知道了一个Channel 的初始化的基本过程，下面我们再回顾一下. 下面的代码是AbstractChannel 构造器:

```java
protected AbstractChannel(Channel parent) {
    this.parent = parent;
    id = newId();
    unsafe = newUnsafe();
    pipeline = newChannelPipeline();
}
```

AbstractChannel有一个pipeline 字段，在构造器中会初始化它为DefaultChannelPipeline的实例。这里的代码就印证了一点：`每个Channel都有一个ChannelPipeline`。

接着我们跟踪一下DefaultChannelPipeline的初始化过程。

首先进入到 DefaultChannelPipeline构造器中:

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

在DefaultChannelPipeline构造器中，首先将与之关联的Channel 保存到字段channel 中，然后实例化两个ChannelHandlerContext，一个是HeadContext 实例head， 另一个是TailContext实例tail.接着将head和tail互相指向，构成一个双向链表。

特别注意到，我们在开始的示意图中，head和tail并没有包含ChannelHandler, 这是`因为HeadContext和TailContext 继承于AbstractChannelHandlerContext的同时也实现了ChannelHandler接口了，因此它们有Context 和Handler的双重属性`。

## 1.2 ChannelPipeline 的初始化

在实例化一个 Channel 时，会伴随着一个 ChannelPipeline 的实例化，并且相互关联，这点可以通过 NioSocketChannel 的父类 AbstractChannel 的构造器予以佐证。 

当实例化一个 Channel（这里以 EchoClient 为例，那么 Channel 就是 NioSocketChannel），其 pipeline 字段就是我们新创建的 DefaultChannelPipeline 对象，那么来看一下DefaultChannelPipeline的构造方法：

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

可以看到，在 DefaultChannelPipeline 的构造方法中，将传入的 channel 赋值给字段 this.channel，接着又实例化了两个特殊的字段：tail与head。

这两个字段是一个双向链表的头和尾。其实在DefaultChannelPipeline中，维护了一个以AbstractChannelHandlesContext为节点的双向链表，这个链表是Netty 实现Pipeline 机制的关键。

再回顾一下head 和tail的类层次结构：

![image-20210107181257710](深入分析 Netty 源码(3).assets/image-20210107181257710.png)

![image-20210107181352216](深入分析 Netty 源码(3).assets/image-20210107181352216.png)

从类层次结构图中可以很清楚地看到，head 实现了ChannelInboundHandler，而tail 实现了ChannelOutboundHandler 接口，并且它们都实现了 ChannelHandlerContext 接口, 因此可以说head 和 tail 既是一个ChannelHandler, 又是一个ChannelHandlerContext。接着看一下HeadContext的构造器:

```java
HeadContext(DefaultChannelPipeline pipeline) {
    super(pipeline, null, HEAD_NAME, false, true);
    unsafe = pipeline.channel().unsafe();
    setAddComplete();
}
```

它调用了父类AbstractChannelHandlerContext 的构造器，并传入参数inbound = false, outbound = true。TailContext的构造器与HeadContext的相反，它调用了父类AbstractChannelHandlerContext的构造器，并传入参数inbound = true, outbound = false。即header是一个outboundHandler, 而tail 是一个inboundHandler，关于这一点，大家要特别注意，因为在后面的分析中，我们会反复用到inbound 和outbound 这两个属性。

## 1.3 ChannelInitializer 的添加

前面我们已经分析了Channel 的组成，其中我们了解到，最开始的时候ChannelPipeline中含有两个ChannelHandlerContext(同时也是ChannelHandler)， 但是这个Pipeline 并不能实现什么特殊的功能，因为我们还没有给它添加自定义的ChannelHandler。

通常来说，我们在初始化Bootstrap，会调用 handler添加自定义的ChannelHandler，同时传入了 ChannelInitializer 对象，它提供了一个 initChannel 方法来初始化 ChannelHandler。 ChannelInitializer实现了ChannelHandler, 那么它是在什么时候添加到ChannelPipeline 中的昵?通过代码跟踪，发现它是在 Bootstrap.init 方法中添加到 ChannelPipeline 中的，其代码如下:

```java
void init(Channel channel) throws Exception {
    ChannelPipeline p = channel.pipeline();
    p.addLast(config.handler());
    ...
}
```

上面的代码将handler() 返回的ChannelHandler 添加到Pipeline 中，而handler() 返回的是handler其实就是我们在初始化Bootstrap 调用handler 设置的ChannelInitializer实例，因此这里就是将ChannelInitializer插入到了Pipeline 的末端。此时Pipeline 的结构如下图所示:

![image-20210107182500985](深入分析 Netty 源码(3).assets/image-20210107182500985.png)

> Netty 提供了一个特殊的 ChannelInboudHandlerAdapter 子类：
>
> public abstract class ChannelInitializer< C extends Channel > extends ChannelInboundHandlerAdapter
>
> 它定义了下面的方法：
>
> protected abstract void initChanel(C ch);
>
> 这个方法提供了一种将多个 ChannelHandler 添加到一个 ChannelPipeline 中的简便方法。你只需要简单地向 Bootstrap 或 ServerBootstrap 的实例提供 ChannelInitializer 实现即可，并且一旦 Channel 被注册到了它的 EventLoop 之后，就会调用你的 initChannel 版本。在该方法返回之后，ChannelInitializer 的实例将会从 ChannelPipeline 中移除它自己。(Bootstrap注册过程中)

明明插入的是一个ChannelInitializer 实例，为什么在ChannelPipeline中的双向链表中的元素却是一个ChannelHandlerContext? 为了解答这个问题，我们继续在代码中寻找答案吧。刚提到，在 Bootstrap.init 中会调用 p.addLast()方法，将 ChannelInitializer 插入到链表尾端：

```java
public final ChannelPipeline addLast(EventExecutorGroup group, String name, ChannelHandler handler) {
    final AbstractChannelHandlerContext newCtx;
    synchronized (this) {
        checkMultiplicity(handler);
        newCtx = newContext(group, filterName(name, handler), handler);
        addLast0(newCtx);
        
            return this;
        }
    }
    callHandlerAdded0(newCtx);
    return this;
}
private AbstractChannelHandlerContext newContext(EventExecutorGroup group, String name, ChannelHandler handler) {
    return new DefaultChannelHandlerContext(this, childExecutor(group), name, handler);
}
```

addLast有很多重载的方法，我们关注这个比较重要的方法就可以了。上面的addLast 方法中，首先检查这个ChannelHandler 的名字是否是重复的，如果不重复的话，则调用newContext方法为这个Handler 创建一个对应的DefaultChannelHandlerContext实例，并与之关联起来(Context中有一个handler 属性保存着对应的Handler 实例)。`为了添加一个handler 到pipeline 中，必须把此handler 包装成ChannelHandlerContext。`

因此在上面的代码中我们可以看到新实例化了一个newCtx 对象，并将handler 作为参数传递到构造方法中，那么我们来看一下实例化的DefaultChannelHandlerContext到底有什么玄机. 首先看它的构造器:

```java
DefaultChannelHandlerContext(
        DefaultChannelPipeline pipeline, EventExecutor executor, String name, ChannelHandler handler) {
    super(pipeline, executor, name, isInbound(handler), isOutbound(handler));
    if (handler == null) {
        throw new NullPointerException("handler");
    }
    this.handler = handler;
}
```

DefaultChannelHandlerContext的构造器中，调用了两个很有意思的方法：isInbound 与 isOutbound：

```java
private static boolean isInbound(ChannelHandler handler) {
    return handler instanceof ChannelInboundHandler;
}
private static boolean isOutbound(ChannelHandler handler) {
    return handler instanceof ChannelOutboundHandler;
}
```

从源码中可以看到，当一个handler 实现了ChannelInboundHandler接口，则isInbound 返回真;相似地，当一个handler 实现了ChannelOutboundHandler 接口，则isOutbound 就返回真. 而这两个boolean变量会传递到父类AbstractChannelHandlerContext中，并初始化父类的两个字段: inbound与outbound. 那么这里的ChannelInitializer 所对应的DefaultChannelHandlerContext的inbound 与inbound字段分别是什么呢?那就看一下ChannelInitializer 到底实现了哪个接口不就行了? 如下是ChannelInitializer的类层次结构图: 

![image-20210107182722266](深入分析 Netty 源码(3).assets/image-20210107182722266.png)

可以清楚地看到，ChannelInitializer 仅仅实现了ChannelInboundHandler接口，因此这里实例化的DefaultChannelHandlerContext的inbound = true， outbound = false。

不就是inbound 和outbound 两个字段嘛，为什么需要这么大费周章地分析一番?其实这两个字段关系到pipeline的事件的流向与分类，因此是十分关键的,后面我们再来详细分析这两个字段所起的作用，在这里，我暂且只需要记住，ChannelInitializer所对应的DefaultChannelHandlerContext的inbound = true, outbound = false 即可。

当创建好Context 后，就将这个Context 插入到Pipeline 的双向链表中:

```java
private void addLast0(AbstractChannelHandlerContext newCtx) {
    AbstractChannelHandlerContext prev = tail.prev;
    newCtx.prev = prev;
    newCtx.next = tail;
    prev.next = newCtx;
    tail.prev = newCtx;
}
```

![image-20210107182500985](深入分析 Netty 源码(3).assets/pipeline-initchannel.png)

## 1.4 自定义 ChannelHandler 的添加过程

前面我们已经分析了一个Channellnitializer 如何插入到Pipeline 中的，接下来就来探讨一下Channellnitializer 在哪里被调用，Channellnitializer 的作用，以及我们自定义的ChanneHandler是如何插入到Pipeline 中的. 现在我们再简单地复习一下Channel的注册过程:

1. 首先在AbstractBootstrap.initAndRegister中，通过group().register(channel),调用MultithreadEventLoopGroup.register方法 
2. MultithreadEventLoopGroup.register 中,通过 next() 获取下一个可用的 SingleThreadEventLoop, 然后调用它的 register 
3. 在 SingleThreadEventLoop.register 中,通过 channel. unsafe (). register(this,promise) 来获取 channel 的 unsafe() 底层操作对象, 然后调用它的 register. 
4. 在 AbstractUnsafe.register 中, 调用 register0 方法注册 Channel 
5. 在 AbstractUnsafe.register0 中, 调用 AbstractNioChannel.doRegister 方法 
6. AbstractNioChannel.doRegister 方法通过javaChannel ().register(eventLoop().seledtor, 0, this) 将 Channel 对应的 Java NIO SockerChannel 注册到一个 eventLoop 的 Selector 中,并且将当前 Channel 作为 attachment.

而自定义的 ChannelHandler 的添加过程，发生在AbstractUnsafe.register0中，在这个方法调用来 pipeline.fireChannelRegistered() 方法:

```java
public final ChannelPipeline fireChannelRegistered() {
    AbstractChannelHandlerContext.invokeChannelRegistered(head);
    return this;
}
```

在看AbstractChannelHandlerContext.invokeChannelRegistered方法：

```java
static void invokeChannelRegistered(final AbstractChannelHandlerContext next) {
    EventExecutor executor = next.executor();
    if (executor.inEventLoop()) {
        next.invokeChannelRegistered();
    } else {
        executor.execute(new Runnable() {
            @Override
            public void run() {
                next.invokeChannelRegistered();
            }
        });
    }
}
```

很显然，这个代码会从head开始遍历Pipeline 的双向链表，然后找到第一个属性inbound为true 的ChannelHandlerContext 实例.想起来了没?我们在前面分析ChannelInitializer时，花了大量的笔墨来分析了inbound 和outbound 属性，你看现在这里就用上了.回想一下，ChannelInitializer实现了Channel InboudHandler,因此它所对应的ChannelHandlerContext的inbound属性就是true，因此这里返回就是ChannelInitializer实例所对应的ChannelHandlerContext.即: 

![image-20210107182816685](深入分析 Netty 源码(3).assets/image-20210107182816685.png)

当获取到 inbound 的 Context 后，就会调用它的 invokeChannelRegistered 方法：

```java
private void invokeChannelRegistered() {
    if (invokeHandler()) {
        try {
            ((ChannelInboundHandler) handler()).channelRegistered(this);
        } catch (Throwable t) {
            notifyHandlerException(t);
        }
    } else {
        fireChannelRegistered();
    }
}
```

我们已经强调过了，每个ChannelHandler都与一个ChannelHandlerContext 关联，我们可以通过ChannelHandlerContext获取到对应的ChannelHandler. 因此很显然了，这里handler()返回的，其实就是一开始我们实例化的ChannelInitializer对象，并接着调用了ChannelInitializer.channelRegistered方法。看到这里，是否会觉得有点眼熟呢?ChannelInitializer.channelRegistered这个方法我们在一开始的时候已经大量地接触了，但是我们并没有深入地分析这个方法的调用过程，那么在这里读者朋友应该对它的调用有了更加深入的了解了吧. 那么这个方法中又有什么玄机呢?继续看代码:

```java
public final void channelRegistered(ChannelHandlerContext ctx) throws Exception {
    if (initChannel(ctx)) {
        ctx.pipeline().fireChannelRegistered();
    } else {
        ctx.fireChannelRegistered();
    }
}
private boolean initChannel(ChannelHandlerContext ctx) throws Exception {
    if (initMap.putIfAbsent(ctx, Boolean.TRUE) == null) {
        try {
            initChannel((C) ctx.channel());
        } catch (Throwable cause) {
            exceptionCaught(ctx, cause);
        } finally {
          	// 此处会将原 ChannelInitializer 移除
            remove(ctx);
        }
        return true;
    }
    return false;
}
```

initChannel((C) ctx.channel()); 这个方法就是在初始化Bootstrap时，调用 handler 方法传入的匿名内部类所实现的方法：

```java
.handler(new ChannelInitializer<SocketChannel>() {
                @Override
                public void initChannel(SocketChannel ch) throws Exception {
                   
                }
            });
```

因此当调用了这个方法后，自定义的 ChannelHandler 就插入到了 Pipeline 了，此时Pipeline 如下图所示：

![image-20210107182849075](深入分析 Netty 源码(3).assets/image-20210107182849075.png)

<font color=red>当添加了自定义的 ChannelHandler 后，就会删除 ChannelInitializer 这个 ChannelHandler</font>，即 “ctx.pipeline().remove(this)”，因此最后的 Pipeline 如下：

![image-20210107182912594](深入分析 Netty 源码(3).assets/image-20210107182912594.png)

## 1.5 ChannelHandler 的名字

pipeline.addXXX 都有一个重载的方法，例如 addLast，它有一个重载的版本是：

```java
ChannelPipeline addLast(String name, ChannelHandler handler);
```

第一个参数指定了所添加的handler 的名字(更准确地说是Channe lHandlerContext的名字，不过我们通常是以handler 作为叙述的对象，因此说成handler 的名字便于理解).那么handler的名字有什么用呢?如果我们不设置name，那么handler会有怎样的名字? 为了解答这些疑惑，老规矩，依然是从源码中找到答案. 我们还是以addLast方法为例:

```java
public final ChannelPipeline addLast(String name, ChannelHandler handler) {
    return addLast(null, name, handler);
}
```

这个方法会调用重载的 addLast 方法：

```java
public final ChannelPipeline addLast(EventExecutorGroup group, String name, ChannelHandler handler) {
    final AbstractChannelHandlerContext newCtx;
    synchronized (this) {
        checkMultiplicity(handler);
        newCtx = newContext(group, filterName(name, handler), handler);
        addLast0(newCtx);
        ...
    }
    return this;
}
```

第一个参数被设置为null，我们不关心它.第二参数就是这个handler 的名字.看代码可知，在添加一个handler 之前，需要调用checkMultiplicity 方法来确定此handler 的名字是否和已添加的handler 的名字重复.

## 1.6 自动生成 handler 的名字

如果调用 ChannelPipeline addLast(ChannelHandler… handlers); 那么 Netty 会调用 generateName 为handler 自动生成一个名字：

```java
private String filterName(String name, ChannelHandler handler) {
    if (name == null) {
        return generateName(handler);
    }
    checkDuplicateName(name);
    return name;
}
private String generateName(ChannelHandler handler) {
    Map<Class<?>, String> cache = nameCaches.get();
    Class<?> handlerType = handler.getClass();
    String name = cache.get(handlerType);
    if (name == null) {
        name = generateName0(handlerType);
        cache.put(handlerType, name);
    }
    if (context0(name) != null) {
        String baseName = name.substring(0, name.length() - 1); // Strip the trailing '0'.
        for (int i = 1;; i ++) {
            String newName = baseName + i;
            if (context0(newName) == null) {
                name = newName;
                break;
            }
        }
    }
    return name;
}
```

而 generatename 会接着调用 generateName0 来实际产生一个 handler 的名字：

```java
private static String generateName0(Class<?> handlerType) {
    return StringUtil.simpleClassName(handlerType) + "#0";
    // ChatClientHandler  => ChatClientHandler#0
}
```

# 2 Pipeline 的事件传输机制

前面章节中，我们知道AbstractChannelHandlerContext中有inbound 和outbound 两个boolean变量，分别用于标识Context所对应的handler 的类型，即:

1. inbound 为真时，表示对应的ChannelHandler 实现了Channel InboundHandler 方法. 
2. outbound 为真时，表示对应的ChannelHandler 实现了Channel0utboundHandler 方法.

这里大家肯定很疑惑了吧:那究竟这两个字段有什么作用呢?其实这还要从ChannelPipeline的传输的事件类型说起. Netty的事件可以分为Inbound 和Outbound 事件.

```java
*                                I/O Request
 *                              via Channel or
 *                           ChannelHandlerContext
 *                                                      |
 *  +---------------------------------------------------+---------------+
 *  |                           ChannelPipeline         |               |
 *  |                                                  \|/              |
 *  |    +---------------------+            +-----------+----------+    |
 *  |    | Inbound Handler  N  |            | Outbound Handler  1  |    |
 *  |    +----------+----------+            +-----------+----------+    |
 *  |              /|\                                  |               |
 *  |               |                                  \|/              |
 *  |    +----------+----------+            +-----------+----------+    |
 *  |    | Inbound Handler N-1 |            | Outbound Handler  2  |    |
 *  |    +----------+----------+            +-----------+----------+    |
 *  |              /|\                                  .               |
 *  |               .                                   .               |
 *  | ChannelHandlerContext.fireIN_EVT() ChannelHandlerContext.OUT_EVT()|
 *  |        [ method call]                       [method call]         |
 *  |               .                                   .               |
 *  |               .                                  \|/              |
 *  |    +----------+----------+            +-----------+----------+    |
 *  |    | Inbound Handler  2  |            | Outbound Handler M-1 |    |
 *  |    +----------+----------+            +-----------+----------+    |
 *  |              /|\                                  |               |
 *  |               |                                  \|/              |
 *  |    +----------+----------+            +-----------+----------+    |
 *  |    | Inbound Handler  1  |            | Outbound Handler  M  |    |
 *  |    +----------+----------+            +-----------+----------+    |
 *  |              /|\                                  |               |
 *  +---------------+-----------------------------------+---------------+
 *                  |                                  \|/
 *  +---------------+-----------------------------------+---------------+
 *  |               |                                   |               |
 *  |       [ Socket.read() ]                    [ Socket.write() ]     |
 *  |                                                                   |
 *  |  Netty Internal I/O Threads (Transport Implementation)            |
 *  +-------------------------------------------------------------------+
```

从上图可以看出，inbound事件和outbound 事件的流向是不一样的，inbound事件的流行是从下至上，而outbound 刚好相反，是从上到下.并且inbound 的传递方式是通过调用相应的ChanneHandlerContext.fireIN_ EVT()方法，而outbound 方法的的传递方式是通过调用Channe lHandlerContext.OUT_ EVT()方法。 例如ChannelHandlerContext.fireChannelRegistered()调用会发送一个ChannelRegistered 的inbound给下一个ChannelHandlerContext，而ChannelHandlerContext.bind调用会发送一个bind 的outbound 事件给下一个ChannelHandlerContext. Inbound事件传播方法有:

```java
public interface ChannelInboundHandler extends ChannelHandler {
    void channelRegistered(ChannelHandlerContext ctx) throws Exception;
    void channelUnregistered(ChannelHandlerContext ctx) throws Exception;
    void channelActive(ChannelHandlerContext ctx) throws Exception;
    void channelInactive(ChannelHandlerContext ctx) throws Exception;
    void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception;
    void channelReadComplete(ChannelHandlerContext ctx) throws Exception;
    void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception;
    void channelWritabilityChanged(ChannelHandlerContext ctx) throws Exception;
    void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception;
}
```

Outbound事件传播方法有:

```java
public interface ChannelOutboundHandler extends ChannelHandler {
    void bind(ChannelHandlerContext ctx, SocketAddress localAddress, ChannelPromise promise) throws Exception;
    void connect(
            ChannelHandlerContext ctx, SocketAddress remoteAddress,
            SocketAddress localAddress, ChannelPromise promise) throws Exception;
    void disconnect(ChannelHandlerContext ctx, ChannelPromise promise) throws Exception;
    void close(ChannelHandlerContext ctx, ChannelPromise promise) throws Exception;
    void deregister(ChannelHandlerContext ctx, ChannelPromise promise) throws Exception;
    void read(ChannelHandlerContext ctx) throws Exception;
    void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception;
    void flush(ChannelHandlerContext ctx) throws Exception;
}
```

注意，如果捕获了一个事件，并且想继续传递下去，那么需要调用 Context 相应的传播方法：

```java
public class MyInboundHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("连接成功");
        ctx.fireChannelActive();
    }
}
public class MyOutboundHandler extends ChannelOutboundHandlerAdapter {
    @Override
    public void close(ChannelHandlerContext ctx, ChannelPromise promise) throws Exception {
        System.out.println("客户端关闭");
        ctx.close(promise);
    }
}
```

上面例子，MyInboundHandler收到一个 channelActive 事件，它在处理后，希望事件继续传播下去，就需要接着调用ctx.fireChannelActive()。

## 2.1 Outbound 事件传播方式

![image-20210107183105454](深入分析 Netty 源码(3).assets/pipeline-channelread.png)

Outbound事件都是请求事件(request event)， 即请求某件事情的发生，然后通过Outbound事件进行通知. Outbound事件的传播方向是tail -&gt; customContext -&gt; head。

我们接下来以connect 事件为例，分析一下Outbound 事件的传播机制。

首先，当用户调用了Bootstrap.connect方法时，就会触发一个Connect请求事件，此调用会触发如下调用链:

![image-20210107183105454](深入分析 Netty 源码(3).assets/image-20210107183105454.png)

继续跟踪的话，我们就发现，AbstractChannel.connect其实由调用了DefaultChannelPipeline.connect方法:

```java
public ChannelFuture connect(SocketAddress remoteAddress, ChannelPromise promise) {
    return pipeline.connect(remoteAddress, promise);
}
// 而 pipeline.connect 的实现如下
public final ChannelFuture connect(
        SocketAddress remoteAddress, SocketAddress localAddress, ChannelPromise promise) {
    return tail.connect(remoteAddress, localAddress, promise);
}
```

可以看到，当outbound 事件(这里是connect 事件)传递到Pipeline 后，它其实是以tail为起点开始传播的. 而tail. connect其实调用的是AbstractChannelHandlerContext.connect方法:

```java
public ChannelFuture connect(
        final SocketAddress remoteAddress, final SocketAddress localAddress, final ChannelPromise promise) {
    final AbstractChannelHandlerContext next = findContextOutbound();
    EventExecutor executor = next.executor();
    next.invokeConnect(remoteAddress, localAddress, promise);
    return promise;
}
```

findContextOutbound() 顾名思义，它的作用是以当前 Context 为起点，向 Pipeline 中的 Context 双向链表的前端寻找第一个outbound属性为真的Context(即关联着ChannelOutboundHandler 的 Context), 然后返回，它的实现如下:

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

如果用户没有重写 ChannelHandler 的 connect 方法，那么就会调用ChannnelOutboundHandlerAdapter 所实现的方法：

```java
public void connect(ChannelHandlerContext ctx, SocketAddress remoteAddress,
        SocketAddress localAddress, ChannelPromise promise) throws Exception {
    ctx.connect(remoteAddress, localAddress, promise);
}
```

ChannnelOutboundHandlerAdapter.connect 仅仅调用了 ctx.connect，而这个调用又回到了： Context.connect -&gt; Connect.findContextOutbound -&gt; next.invokeConnect- &gt;handler. connect -&gt; Context.connect 这样的循环中，直到connect 事件传递到DefaultChannelPipeline的双向链表的头节点，即 head中，为什么会传递到head中呢?回想一下，head实现了ChannelOutboundHandler，因此它的outbound 属性是true. 因为head 本身既是一个ChannelHandlerContext，又实现了ChannelOutboundHandler接口，因此当connect 消息传递到head后，会将消息转递到对应的ChannelHandler中处理，而恰好，head的handler() 返回的就是head 本身:

```java
public ChannelHandler handler() {
    return this;
}
```

因此最终 connect 事件在 head 中处理的，head 的 connect 事件处理方式如下：

```java
public void connect(
        ChannelHandlerContext ctx,
        SocketAddress remoteAddress, SocketAddress localAddress,
        ChannelPromise promise) throws Exception {
    unsafe.connect(remoteAddress, localAddress, promise);
}
```


到这里，整个 Connect 请求事件就结束了： ![img](https://img-blog.csdnimg.cn/20200606183637811.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70) 我们仅仅以Connect 请求事件为例，分析了Outbound 事件的传播过程，但是其实所有的outbound的事件传播都遵循着一样的传播规律,同学们可以试着分析一下其他的outbound 事件，体会一下它们的传播过程.

## 2.2 Inbound 事件

Inbound 事件和 Outbound 事件的处理过程是类似的，只是传播方向不同。

Inbound事件是一个通知事件，即某件事已经发生了，然后通过Inbound 事件进行通知. Inbound通常发生在Channel 的状态的改变或IO事件就绪。

Inbound的特点是它传播方向是head -&gt; customContext -&gt; tail. 既然上面我们分析了Connect 这个Outbound 事件，那么接着分析Connect事件后会发生什么Inbound 事件，并最终找到Outbound 和Inbound 事件之间的联系. 当Connect 这个Outbound 传播制unsafe 后，其实是在AbstractNioUnsafe.connect 方法中进行处理的：

```java
public final void connect(final SocketAddress remoteAddress, final SocketAddress localAddress, final ChannelPromise promise) {
	if (doConnect(remoteAddress, localAddress)) {
        fulfillConnectPromise(promise, wasActive);
    } else {...}
}
```

在AbstractNioUnsafe.connect中，首先调用doConnect方法进行实际上的 Socket连接,当连接上后，会调用fulfillConnectPromise 方法:

```java
private void fulfillConnectPromise(ChannelPromise promise, boolean wasActive) {
    if (!wasActive && active) {
        pipeline().fireChannelActive();
    }
}
```

我们看到，在fulfillConnectPromise中，会通过调用pipeline().fireChannelActive()将通道激活的消息(即Socket 连接成功)发送出去. 而这里，当调用pipeline.fireXXX后，就是Inbound事件的起点. 因此当调用了pipeline().fireChannelActive()后，就产生了一个ChannelActive Inbound事件，我们就从这里开始看看这个Inbound 事件是怎么传播的吧.

```java
public final ChannelPipeline fireChannelActive() {
    AbstractChannelHandlerContext.invokeChannelActive(head);
    return this;
}
```

在fireChannelActive方法中，调用的是head.invokeChannelActive，因此可以证明了，Inbound 事件在Pipeline 中传输的起点是head.那么，在head. invokeChannelActive()中又做了什么呢?

```java
static void invokeChannelActive(final AbstractChannelHandlerContext next) {
    EventExecutor executor = next.executor();
    if (executor.inEventLoop()) {
        next.invokeChannelActive();
    } else {
        executor.execute(new Runnable() {
            @Override
            public void run() {
                next.invokeChannelActive();
            }
        });
    }
}
```

上面的代码应该很熟悉了吧.回想一下在Outbound 事件(例如Connect 事件)的传输过程中时，我们也有类似的操作:

1. 首先调用findContextInbound，从Pipeline 的双向链表中中找到第一个属性inbound为真的Context， 然后返回 
2. 调用这个Context 的invokeChannelActive方法如下:

```java
private void invokeChannelActive() {
    if (invokeHandler()) {
        try {
            ((ChannelInboundHandler) handler()).channelActive(this);
        } catch (Throwable t) {
            notifyHandlerException(t);
        }
    } else {
        fireChannelActive();
    }
}
```

这个方法和Outbound 的对应方法(例如invokeConnect) 如出一辙.同Outbound 一样，如果用户没有重写channelActive 方法，那么会调用ChannelInboundHandlerAdapter的channelActive方法:

```java
public void channelActive(ChannelHandlerContext ctx) throws Exception {
    ctx.fireChannelActive();
}
```

同样地，在 ChannelInboubdHandlerAdapter.channelActive中，仅仅调用了ctx.fireChannelActive方法，就因此会有如下循环： Context.fireChannelActive -&gt; Connect. findContextInbound -&gt;nextContext.invokeChannelActive -&gt; nextHandler.channelActive - &gt;nextContext. fireChannelActive 这样的循环中，同理，tail 本身既实现了ChannelInboundHandler接口，又实现了ChannelHandlerContext接口，因此当channelActive 消息传递到tail 后，会将消息转递到对应的ChannelHandler中处理，而恰好，tail的handler()返回的就是tail 本身:

```java
public ChannelHandler handler() {
    return this;
}
```

因此 channelActive Inbound 事件最终在 tail 中处理，看一下它的处理方法：

```java
public void channelActive(ChannelHandlerContext ctx) throws Exception { }
```


TailContext.channelActive方法是空的.如果读者自行查看TailContext 的Inbound 处理方法时，会发现，它们的实现都是空的.可见，如果是Inbound， 当用户没有实现自定义的处理器时，那么默认是不处理的. 用一幅图来总结一下Inbound 的传输过程: ![img](https://img-blog.csdnimg.cn/20200606183657865.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

# 3 总结

对于Outbound 事件:

1. Outbound 事件是请求事件(由Connect 发起一个请求，并最终由unsafe 处理这个请求) 
2. Outbound 事件的发起者是Channel . 
3. Outbound事件的处理者是unsafe 
4. Outbound 事件在Pipeline 中的传输方向是tail -&gt; head. 
5. 在ChannelHandler中处理事件时，如果这个Handler 不是最后一个Handler, 则需要调用ctx. xxx (例如ctx. connect)将此事件继续传播下去.如果不这样做，那么此事件的传播会提前终止. 
6. Outbound事件流: Context.OUT_ EVT -&gt; Connect.findContextOutbound - &gt;nextContext. invokeOUT_ EVT -&gt; nextHandler. OUT_ EVT -&gt; nextContext. OUT_ EVT

对于Inbound 事件:

1. Inbound 事件是通知事件，当某件事情已经就绪后，通知上层. 
2. Inbound 事件发起者是unsafe 
3. Inbound 事件的处理者是Channel, 如果用户没有实现自定义的处理方法，那么Inbound事件默认的处理者是TailContext， 并且其处理方法是空实现. 
4. Inbound事件在Pipeline 中传输方向是head -&gt; tail 
5. 在ChannelHandler中处理事件时，如果这个Handler 不是最后一个Handler， 则需要调用ctx.fireIN_ EVT (例如ctx.fireChannelActive)将此事件继续传播下去.如果不这样做,那么此事件的传播会提前终止. 
6. Outbound 事件流: Context.fireIN_ EVT - &gt; Connect. findContext Inbound- &gt;nextContext. invokeIN EVT -&gt; nextHandler. IN EVT -&gt; nextContext. fireIN EVT

outbound和inbound 事件十分的像，并且Context 与Handler 直接的调用关系是否容易混淆，因此我们在阅读这里的源码时，需要特别的注意.

------

