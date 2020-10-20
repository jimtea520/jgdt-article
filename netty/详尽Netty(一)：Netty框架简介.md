# Netty框架解析(一):简介

# 概念

​		Netty 是一个高性能、异步事件驱动的 NIO 框架，它提供了对 TCP、UDP 和文件传输的支持，作为一个异步 NIO 框架，Netty 的所有 IO 操作都是异步非阻塞的，通过 Future-Listener 机制，用户可以方便的主动获取或者通过通知机制获得 IO 操作结果。

作为当前最流行的 NIO 框架，Netty 在互联网领域、大数据分布式计算领域、游戏行业、通信行业等获得了广泛的应用，一些业界著名的开源组件也基于 Netty 的 NIO 框架构建。

本系列学习基于 [Netty 4.1](https://netty.io/) 展开介绍相关理论模型，使用场景，基本组件、整体架构，知其然且知其所以然，希望给大家在实际开发实践、学习开源项目方面提供参考。



# JDK 原生 NIO 程序的问题

JDK 原生也有一套网络应用程序 API，但是存在一系列问题，主要如下：

1）NIO 的类库和 API 繁杂，使用麻烦：你需要熟练掌握 Selector、ServerSocketChannel、SocketChannel、ByteBuffer 等。

2）需要具备其他的额外技能做铺垫：例如熟悉 Java 多线程编程，因为 NIO 编程涉及到 Reactor 模式，你必须对多线程和网路编程非常熟悉，才能编写出高质量的 NIO 程序。

3）可靠性能力补齐，开发工作量和难度都非常大：例如客户端面临断连重连、网络闪断、半包读写、失败缓存、网络拥塞和异常码流的处理等等。NIO 编程的特点是功能开发相对容易，但是可靠性能力补齐工作量和难度都非常大。

4）JDK NIO 的 Bug：例如臭名昭著的 Epoll Bug，它会导致 Selector 空轮询，最终导致 CPU 100%。官方声称在 JDK 1.6 版本的 update 18 修复了该问题，但是直到 JDK 1.7 版本该问题仍旧存在，只不过该 Bug 发生概率降低了一些而已，它并没有被根本解决。



# Netty 的特点

Netty 对 JDK 自带的 NIO 的 API 进行了封装，解决了上述问题。

Netty的主要特点有：

   设计：

​    针对多种传输类型的同一接口-阻塞和非阻塞；

​    简单但更强大的线程模型；

​    真正的无连接的数据报套接字支持；

​    连接逻辑支持复用。

易用性：

​    大量的javadoc和代码实例；

​    除了在JDK1.6+额外的限制。

性能：

​    比核心Java API更好的吞吐量，较低的延时；

​    资源消耗更少，这个得益于于共享池和重用；

​    减少内存拷贝。

健壮性：

​    消除由于慢、快、活重载连接产生的OutOfMemoryError；

​    消除经常发现在NIO在告诉网络中的应用中的不公平的读/写比。

安全：

​    完整的SSL/TLS和StartTLS的支持；

​    运行在受限的环境例如Applet活OSGI。

社区：

​    发布的更早和更频繁；

​    社区驱动。



## 各种io介绍

**BIO**

同步阻塞IO，阻塞整个步骤，如果连接少，他的延迟是最低的，因为一个线程只处理一个连接，适用于少连接且延迟低的场景，比如说数据库连接。

**NIO**

同步非阻塞IO，阻塞业务处理但不阻塞数据接收，适用于高并发且处理简单的场景，比如聊天软件。

**多路复用IO**

他的两个步骤处理是分开的，也就是说，一个连接可能他的数据接收是线程a完成的，数据处理是线程b完成的，他比BIO能处理更多请求。

**信号驱动IO**

这种IO模型主要用在嵌入式开发，不参与讨论。

**异步IO**

他的数据请求和数据处理都是异步的，数据请求一次返回一次，适用于长连接的业务场景。

# Netty中几个主要组件 

- Channel：代表了一个链接，与EventLoop一起用来参与IO处理。 
- ChannelHandler：为了支持各种协议和处理数据的方式，便诞生了Handler组件。Handler主要用来处理各种事件，这里的事件很广泛，比如可以是连接、数据接收、异常、数据转换等。 
- ChannelHandlerContext，用于传输业务数据。
- ChannelPipeline：提供了 ChannelHandler 链的容器，并定义了用于在该链上传播入站 
和出站事件流的 API。 用于保存处理过程需要用到的ChannelHandler和ChannelHandlerContext。
- EventLoop：Channel处理IO操作，一个EventLoop可以为多个Channel服务。 
- EventLoopGroup：会包含多个EventLoop。



# Netty中核心部分

- Channels
- Buffers
- Selectors

### Channel 和 Buffer

基本上，所有的 IO 在NIO 中都从一个Channel 开始。Channel 有点象流。 数据可以从Channel读到Buffer中，也可以从Buffer 写到Channel中。这里有个图示：

![img](E:\技术帖子\笔记\netty\图片资源\netty介绍1.png)

Channel和Buffer有好几种类型。下面是JAVA NIO中的一些主要Channel的实现：

- **FileChannel**
- **DatagramChannel**
- **SocketChannel**
- **ServerSocketChannel**

正如你所看到的，这些通道涵盖了UDP 和 TCP 网络IO，以及文件IO。

与这些类一起的有一些有趣的接口，但为简单起见，我尽量在概述中不提到它们。本教程其它章节与它们相关的地方我会进行解释。

以下是Java NIO里关键的Buffer实现：

- **ByteBuffer**
- **CharBuffer**
- **DoubleBuffer**
- **FloatBuffer**
- **IntBuffer**
- **LongBuffer**
- **ShortBuffer**

这些Buffer覆盖了你能通过IO发送的基本数据类型：byte, short, int, long, float, double 和 char。

Java NIO 还有个 MappedByteBuffer，用于表示内存映射文件， 我也不打算在概述中说明。



## Selector

Selector允许单线程处理多个 Channel。如果你的应用打开了多个连接（通道），但每个连接的流量都很低，使用Selector就会很方便。例如，在一个聊天服务器中。

这是在一个单线程中使用一个Selector处理3个Channel的图示：

![img](E:\技术帖子\笔记\netty\图片资源\netty介绍2.png)

要使用Selector，得向Selector注册Channel，然后调用它的select()方法。这个方法会一直阻塞到某个注册的通道有事件就绪。一旦这个方法返回，线程就可以处理这些事件，事件的例子有如新连接进来，数据接收等。



## Netty异步和事件驱动

所有的网络应用程序需要被设计为可扩展性，可以被界定为“一个系统，网络能力，或过程中能够处理越来越多的工作方式或可扩大到容纳增长的能力”,Netty 利用非阻塞 I/O 完成这一目标，通常称为“异步 I/O”



### 为什么Netty受欢迎？

1. **并发高**

2. **传输快**

3. **封装好**

   

### Netty为什么并发高?

Selector, 当一个Socket建立好之后，Thread并不会阻塞去接受这个Socket，而是将这个请求交给Selector，Selector会不断的去遍历所有的Socket，一旦有一个Socket建立完成，他会通知Thread，然后Thread处理完数据再返回给客户端——**这个过程是不阻塞的**，这样就能让一个Thread处理更多的请求了。

### Netty为什么传输快？

零拷贝，当它需要接收数据的时候，它会在堆内存之外开辟一块内存，数据就直接从IO读到了那块内存中去，在netty里面通过ByteBuf可以直接对这些数据进行直接操作，从而加快了传输速度。

### Netty和Tomcat有什么区别？

Netty和Tomcat最大的区别就在于通信协议，Tomcat是基于Http协议的，他的实质是一个基于http协议的web容器，但是Netty不一样，他能通过编程自定义各种协议，因为netty能够通过codec自己来编码/解码字节流，完成类似redis访问的功能，这就是netty和tomcat最大的不同。

有人说netty的性能就一定比tomcat性能高，其实不然，tomcat从6.x开始就支持了nio模式，并且后续还有APR模式——一种通过jni调用apache网络库的模式，相比于旧的bio模式，并发性能得到了很大提高，特别是APR模式，而netty是否比tomcat性能更高，则要取决于netty程序作者的技术实力了。



# Netty 常见使用场景

1）**互联网行业：**在分布式系统中，各个节点之间需要远程服务调用，高性能的 RPC 框架必不可少，Netty 作为异步高性能的通信框架，往往作为基础通信组件被这些 RPC 框架使用。典型的应用有：阿里分布式服务框架 Dubbo 的 RPC 框架使用 Dubbo 协议进行节点间通信，Dubbo 协议默认使用 Netty 作为基础通信组件，用于实现各进程节点之间的内部通信。

2）**游戏行业：**无论是手游服务端还是大型的网络游戏，Java 语言得到了越来越广泛的应用。Netty 作为高性能的基础通信组件，它本身提供了 TCP/UDP 和 HTTP 协议栈。

非常方便定制和开发私有协议栈，账号登录服务器，地图服务器之间可以方便的通过 Netty 进行高性能的通信。

3）**大数据领域：**经典的 Hadoop 的高性能通信和序列化组件 Avro 的 RPC 框架，默认采用 Netty 进行跨界点通信，它的 Netty Service 基于 Netty 框架二次封装实现。