

# Netty框架教程(二):环境搭建

## 1. 依赖工具

Maven
Git
JDK
IntelliJ IDEA
SourceTree



## 2.源码拉取

netty源码git地址:https://github.com/netty/netty

选择的是4.1分支，如图：



最好是Fork到自己的仓库里，这样可以自己做点笔记，写一些注释，并且提交。

然后用sourcetree来clone

![image-20200501131326939](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200501131326939.png)





3. maven 配置：



如果不选jdk8，则会报错：

```
java.lang.NoSuchMethodError:
java.nio.ByteBuffer.clear()Ljava/nio/ByteBuffer
```

   4.

```
import io.netty.util.collection.LongObjectHashMap;
import io.netty.util.collection.LongObjectMap;
```

如果是这个引用有问题，则必须common包要重新编译一下，原因可以参见：

https://github.com/netty/netty/issues/7518

或者

https://github.com/netty/netty/issues/5447

5. ###  **运行 example**

   netty源码中，有很多example例子程序，在example目录下。

6. EchoServer

   执行 io.netty.example.echo.EchoServer 的 #main(args) 方法，启动服务端。目录如图：

   

   输出日志如下：

   ```
   00:20:17.078 [nioEventLoopGroup-2-1] INFO  i.n.handler.logging.LoggingHandler - [id: 0xbfd8f792] REGISTERED
   00:20:17.092 [nioEventLoopGroup-2-1] INFO  i.n.handler.logging.LoggingHandler - [id: 0xbfd8f792] BIND: 0.0.0.0/0.0.0.0:8007
   00:20:17.100 [nioEventLoopGroup-2-1] INFO  i.n.handler.logging.LoggingHandler - [id: 0xbfd8f792, L:/0:0:0:0:0:0:0:0:8007] ACTIVE
   ```

   

   7.EchoClient

  	   执行 io.netty.example.echo.EchoClientr 的 #main(args) 方法，启动客户端，目录如图：

​	

​		不输出任何日志。

​		EchoServer输出：

```
00:23:35.880 [nioEventLoopGroup-2-1] INFO  i.n.handler.logging.LoggingHandler - [id: 0xbfd8f792, L:/0:0:0:0:0:0:0:0:8007] READ: [id: 0xee5c60d5, L:/127.0.0.1:8007 - R:/127.0.0.1:49598]
00:23:35.885 [nioEventLoopGroup-2-1] INFO  i.n.handler.logging.LoggingHandler - [id: 0xbfd8f792, L:/0:0:0:0:0:0:0:0:8007] READ COMPLETE
```



