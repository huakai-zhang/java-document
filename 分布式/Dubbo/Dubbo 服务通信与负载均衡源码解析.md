

至此Reference在关联了所有application、module、consumer、registry、monitor、service、protocol后调用对应Protocol类的refer方法生成InvokerProxy。当用户调用service时dubbo会通过InvokerProxy调用Invoker的invoke的方法向服务端发起请求。客户端就这样完成了自己的初始化。

这个代理实例中仅仅包含一个handler对象（InvokerInvocationHandler类的实例），handler中则包含了RPC调用中非常核心的一个接口Invoker<T>的实现，Invoker接口的的的定义如下：

```java
public interface Invoker<T> extends Node {     
	Class<T> getInterface();  //调用过程的具体表示形式      
	Result invoke(Invocation invocation) throws RpcException;
}
```

Invoker<T>接口的核心方法是invoke(Invocation invocation)，方法的参数Invocation是一个调用过程的抽象，也是Dubbo框架的核心接口，该接口中包含如何获取调用方法的名称、参数类型列表、参数列表以及绑定的数据，定义代码如下：

```java
public interface Invocation {
    String getTargetServiceUniqueName();
    // 调用的方法名字
    String getMethodName();
    // 调用方法的参数的类型列表
    Class<?>[] getParameterTypes();
    // 调用方法的参数列表
    Object[] getArguments();
    // 调用时附加的数据，用map存储
    Map<String, String> getAttachments();
    // 根据key来获取附加的数据
    String getAttachment(String key);
    // getAttachment扩展，支持默认值获取
    String getAttachment(String key, String defaultValue);
    // 获取真实的调用者实现
    Invoker<?> getInvoker();
}
```

代理中的handler实例中包含的Invoker<T>接口实现者是MockClusterInvoker，其中MockClusterInvoker仅仅是一个Invoker的包装，并且也实现了接口Invoker<T>，其只是用于实现Dubbo框架中的mock功能，我们可以从他的invoke方法的实现中看出。


Dubbo的插件化实现非常类似于原生的JAVA的SPI：它只是提供一种协议，并没有提供相关插件化实施的接口。用过的同学都知道，它有一种java原生的支持类：ServiceLoader，通过声明接口的实现类，在META-INF/services中注册一个实现类，然后通过ServiceLoader去生成一个接口实例，当更换插件的时候只需要把自己实现的插件替换到META-INF/services中即可。