Dubbo-rpc调用流程





RPC 工作原理

RPC的设计由Client，Client stub，Network ，Server stub，Server构成。 其中Client就是用来调用服务的，Cient stub是用来把调用的方法和参数序列化的（因为要在网络中传输，必须要把对象转变成字节），Network用来传输这些信息到Server stub， Server stub用来把这些信息反序列化的，Server就是服务的提供者，最终调用的就是Server提供的方法。



![image-20200928110517958](https://gitee.com/fking86/images4typora/raw/master/imgs/20200928110518.png)



1、Client像调用本地服务似的调用远程服务；
2、Client stub接收到调用后，将方法、参数序列化
3、客户端通过sockets将消息发送到服务端
4、Server stub 收到消息后进行解码（将消息对象反序列化）
5、Server stub 根据解码结果调用本地的服务
6、本地服务执行(对于服务端来说是本地执行)并将结果返回给Server stub
7、Server stub将返回结果打包成消息（将结果消息对象序列化）
8、服务端通过sockets将消息发送到客户端
9、Client stub接收到结果消息，并进行解码（将结果消息反序列化）
10、客户端得到最终结果。



Dubbo中的RPC调用流程图

![image-20200928110707902](https://gitee.com/fking86/images4typora/raw/master/imgs/20200928110708.png)

Dubbo官网
![image-20200928111207796](https://gitee.com/fking86/images4typora/raw/master/imgs/20200928111208.png)



**一、客户端执行流程**

执行main(String[]) 

首先服务消费者通过代理对象 Proxy 发起远程调用，即demoService是一个代理bean，执行proxy0.sayHello(String) ，在sayHello方法中当执行到**handler**.invoke(this, methods[0], args)时

1.调用InvocationHandler

​     执行**InvokerInvocationHandler**.invoke(Object, Method, Object[])

2.调用invoker
     执行MockCluster**Invoker**<T>.invoke(Invocation)
     执行**服务降级**FailoverCluster**Invoker**<T>(AbstractClusterInvoker<T>).invoke(Invocation)

> 执行RegistryDirectory$**Invoker**Delegate<T>(InvokerWrapper<T>).invoke(Invocation) 

3.过滤器链处理

> **ConsumerContextFilter**.invoke(Invoker<?>, Invocation) 
>
> **FutureFilter**.invoke(Invoker<?>, Invocation) 
>
> **MonitorFilter**.invoke(Invoker<?>, Invocation)

4.调用invoker

> Listener**Invoker**Wrapper<T>.invoke(Invocation)
>
> Dubbo**Invoker**<T>(AbstractInvoker<T>).invoke(Invocation)

5.调用ExchangeClient

> ReferenceCount**ExchangeClient**.request(Object, int) 
>
> Header**ExchangeClient**.request(Object, int) 

6.执行ExchangeChannel

​     Header**ExchangeChannel**.request(Object, int)，执行channel.send方法向服务端发起请求

**二、服务提供者执行流程**

1.当服务端接收到来自客户端的请求后，开始执行ChannelEventRunnable中的run() 方法

2.执行ChannelHandler

> Decode**Handler**.received(Channel, Object) 
>
> HeaderExchange**Handler**.received(Channel, Object)
>
> HeaderExchange**Handler**.handleRequest(ExchangeChannel, Request) 

3.执行Protocol

​    Dubbo**Protocol**$1.reply(ExchangeChannel, Object) 

4.过滤器链处理

> **EchoFilter**.invoke(Invoker<?>, Invocation)
>
> **ClassLoaderFilter**.invoke(Invoker<?>, Invocation) 
>
> **GenericFilter**.invoke(Invoker<?>, Invocation) 
>
> **ContextFilter**.invoke(Invoker<?>, Invocation)
>
> **TraceFilter**.invoke(Invoker<?>, Invocation) 
>
> **TimeoutFilter**.invoke(Invoker<?>, Invocation) 
>
> **MonitorFilter**.invoke(Invoker<?>, Invocation)
>
> **ExceptionFilter**.invoke(Invoker<?>, Invocation) 

5.执行Invoker调用

> RegistryProtocol$**Invoker**Delegete<T>(InvokerWrapper<T>).invoke(Invocation)
>
> DelegateProviderMetaData**Invoker**<T>.invoke(Invocation) 

6.代理类调用

> Javassist**ProxyFactory**$1(AbstractProxyInvoker<T>).invoke(Invocation) 

7.Wrapper调用方法

​    **Wrapper**1.invokeMethod(Object, String, Class[], Object[]) 

8.执行Impl的调用

​    DemoServiceImpl.sayHello(String)

 

说明：

服务消费者执行如下代码时

```
DemoService demoService = (DemoService) context.getBean("demoService"); 
String hello = demoService.sayHello("world"); 
```

 

获得demoService代理类，执行hello方法直接调用其代理类的sayHello方法

```
public java.lang.String sayHello(java.lang.String arg0){
    Object[] args = new Object[1]; 
    args[0] = ($w)$1; 
    Object ret = handler.invoke(this, methods[0], args); 
    return (java.lang.String)ret;
}
```

handler是一个InvokerInvocationHandler类型，InvocationHandler是其接口

![image-20200928111320281](https://gitee.com/fking86/images4typora/raw/master/imgs/20200928111320.png)