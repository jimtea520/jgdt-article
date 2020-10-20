## 1、首先redis是单线程的，为什么redis会是单线程的呢？

1. 从redis的性能上进行考虑，单线程避免了上下文频繁切换问题，效率高；
2. 从redis的内部结构设计原理进行考虑，redis是基于**Reactor模式**开发了自己的网络事件处理器： 这个处理器被称为**文件事件处理器**（file event handler）。而这个文件事件处理器是单线程的，所以才叫redis的**单线程模型**，这也决定了redis是单线程的。



## 2、聊一聊redis的单线程模型构造部分？

redis单线程模型中最为核心的就是文件事件处理器

而文件事件处理器结构包含5个部分，**其实真正包含为4个部分（不包含socket队列，加上主要方便后面理解）**：多个socket、IO多路复用程序、socket队列、文件事件分派器、以及事件处理器。而事件处理器又分为3个部分为：连接应答处理器、命令请求处理器、命令回复处理器。如图：

![image-20200925170038382](https://gitee.com/fking86/images4typora/raw/master/imgs/20200925170038.png)

## 3、redis单线程模型的大致工作流程及原理

客户端与redis进行通信大致流程：
1、首先在redis启动初始化的时候，redis会先将事件处理器中的连接应答处理器和AE_READABLE事件关联起来;
2、如果客户端向redis发起连接，会产生AE_READABLE事件(步骤A)，产生该事件后会被IO多路复用程序监听到(步骤B)，然后IO多路复用程序会把监听到的socket信息放入到队列中(步骤C)，事件分配器每次从队列中取出一个socket(步骤D)，然后事件分派器把socket给对应的事件处理器(步骤E)。由于连接应答处理器和AE_READABLE事件在redis初始化的时候已经关联起来，所以由连接应答处理器来处理跟客户端建立连接，然后通过ServerSocket创建一个与客户端一对一对应的socket，如叫socket01，同时将这个socket01的AE_READABLE事件和命令请求处理器关联起来。

![image-20200925170313501](https://gitee.com/fking86/images4typora/raw/master/imgs/20200925170313.png)

3、当客户端向redis发生请求时(读、写操作)，首先就会在对应的socket如socket01上会产生AE_READABLE事件(步骤A)，产生该事件后会被IO多路复用程序监听到(步骤B)，然后IO多路复用程序会把监听到的socket信息放入到队列中(步骤C)，事件分配器每次从队列中取出一个socket(步骤D)，然后事件分派器把socket给对应的事件处理器(步骤E)。由于命令处理器和socket01的AE_READABLE事件关联起来了，然后对应的命令请求处理器来处理。这个命令请求处理器会从事件分配器传递过来的socket01上读取相关的数据，如何执行相应的读写处理。操作执行完之后，redis就会将准备好相应的响应数据(如你在redis客户端输入 set a 123回车时会看到响应ok)，并将socket01的AE_WRITABLE事件和命令回复处理器关联起来。

![image-20200925171341458](https://gitee.com/fking86/images4typora/raw/master/imgs/20200925171341.png)

4、当客户端会查询redis是否完成相应的操作，就会在socket01上产生一个AE_WRITABLE事件，会由对应的命令回复处理器来处理，就是将准备好的相应数据写入socket01(由于socket连接是双向的),返回给客户端，如读操作，客户端会显示ok。

![image-20200925171508599](https://gitee.com/fking86/images4typora/raw/master/imgs/20200925171508.png)

5、如果命令回复处理器执行完成后，就会删除这个socket01的AE_WRITABLE事件和命令回复处理器的关联。
6、这样客户端就和redis进行了一次通信。由于连接应答处理器执行一次就够了，如果客户端在次进行操作就会由命令请求处理器来处理，反复执行。