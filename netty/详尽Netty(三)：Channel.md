## 概念

Channel 是java nio的一个基本构造。

​		它代表一个到实体(如一个硬件设备，一个文件、一个网络套接字或者一个能够之行一个或者多个不同的I/O操作的程序组件)的开放链接，如读操作和写操作。

可以把Channel 看做是传送(入站)或者传出(出站)数据的载体。可以被打开或者被关闭，链接或者断开连接。

其UML图：



## 分类

Channel：是对网络Socket的封装，抽象了网络I/O的读、写、连接与绑定。

AbstractChannel：实现了Channel接口的大部分功能，一次连接用到的Channel、ChannelId、eventLoop、pipelIne、unsafe都会保存在这里。

AbstractNioChannel：通过select的方式对读写事件进行监听。

客户端Channel：主要注册read与write事件，关注于具体数据的读写。

服务端Channel：主要注册accept事件，关注于具体连接的接入，这也是与客户端Channel的read事件最主要的区别。



所有方法,如图：



## Channel重要方法和参数

eventLoop： 返回分配给Channel 的EventLoop
pipeline： 返回分配给Channel 的ChannelPipeline
isActive： 如果Channel 是活动的，则返回true。活动的意义可能依赖于底层的传输。例如，一个Socket 传输一旦连接到了远程节点便是活动的，而一个Datagram 传输一旦被打开便是活动的。
localAddress： 返回本地的SokcetAddress
remoteAddress： 返回远程的SocketAddress
write： 将数据写到远程节点。这个数据将被传递给ChannelPipeline，并且排队直到它被冲刷
flush： 将之前已写的数据冲刷到底层传输，如一个Socket
writeAndFlush： 一个简便的方法，等同于调用write()并接着调用flush()

## 生命周期状态

ChannelUnregistered ：Channel 已经被创建，但还未注册到EventLoop
ChannelRegistered ：Channel 已经被注册到了EventLoop
ChannelActive ：Channel 处于活动状态（已经连接到它的远程节点）。它现在可以接收和发送数据了
ChannelInactive ：Channel 没有连接到远程节点
当这些状态发生改变时，将会生成对应的事件。这些事件将会被转发给ChannelPipeline 中的ChannelHandler，其可以随后对它们做出响应。

状态的切换：



## 相关源码

### 1. AbstractChannel

```java
public abstract class AbstractChannel extends DefaultAttributeMap implements Channel {
    // 父 Channel（NioServerSocketChannel 是没有父channel的）
    private final Channel parent;
    // Channel 唯一ID
    private final ChannelId id;
    // Unsafe 对象，封装 ByteBuf 的读写操作
    private final Unsafe unsafe;
    // 关联的 Pipeline 对象
    private final DefaultChannelPipeline pipeline;
    
    private final VoidChannelPromise unsafeVoidPromise = new VoidChannelPromise(this, false);
    private final CloseFuture closeFuture = new CloseFuture(this);
    // 本地地址
    private volatile SocketAddress localAddress;
    //远端地址
    private volatile SocketAddress remoteAddress;
    // EventLoop 封装的 Selector,channel所注册的eventLoop
    private volatile EventLoop eventLoop;
    // 是否注册
    private volatile boolean registered;
    private boolean closeInitiated;

    /** Cache for the string representation of this channel */
    private boolean strValActive;
    private String strVal;
```

构造函数

```java
protected AbstractChannel(Channel parent, ChannelId id) {
    this.parent = parent;
    this.id = id;
    unsafe = newUnsafe();
    pipeline = newChannelPipeline();
}

// Unsafe 实现交给子类实现
protected abstract AbstractUnsafe newUnsafe();

// 创建 DefaultChannelPipeline 对象
protected DefaultChannelPipeline newChannelPipeline() {
    return new DefaultChannelPipeline(this);
}
```

说明

1.Unsafe类里实现了具体的连接与写数据。比如：网络的读，写，链路关闭，发起连接等。之所以命名为unsafe是不希望外部使用，并非是不安全的。

2.DefaultChannelPipeline 只是一个 Handler 的容器，也可以理解为一个Handler链，具体的逻辑由Handler处理，而每个Handler都会分配一个EventLoop，最终的请求还是要EventLoop来执行，而EventLoop中又调用Channel中的内部类Unsafe对应的方法。
 新建一个channel会自动创建一个ChannelPipeline。

3.这里创建 DefaultChannelPipeline，构造中传入当前的 Channel，而读写数据都是在 ChannelPipeline 中进行的，ChannelPipeline 进行读写数据又委托给 Channel 中的 Unsafe 进行操作。

### 2.AbstractNioChannel 

```java
public abstract class AbstractNioChannel extends AbstractChannel {

    // 抽象了 SocketChannel 和 ServerSocketChannel 的公共的父类
    //Socketchannle和ServerSocketChannel的公共操作类，用来设置SelectableChannel相关参数和IO操作
    private final SelectableChannel ch;
    // SelectionKey.OP_READ 读事件
    protected final int readInterestOp;
    // 注册到 selector 上返回的 selectorKey
    volatile SelectionKey selectionKey;
    // 是否还有未读的数据
    boolean readPending;
    private final Runnable clearReadPendingRunnable = new Runnable() {
        @Override
        public void run() {
            clearReadPending0();
        }
    };

    /**
     * The future of the current connection attempt.  If not null, subsequent
     * connection attempts will fail.
     */
    // 连接操作的结果
    private ChannelPromise connectPromise;
    // 连接超时定时任务
    private ScheduledFuture<?> connectTimeoutFuture;
    // 客户端地址
    private SocketAddress requestedRemoteAddress;
   
    ....
```

 核心方法：

```
  //核心操作，注册操作

 //1) 如果当前注册返回的selectionKey已经被取消，则抛出CancelledKeyException异常，捕获该异常进行处理。
//2) 如果是第一次处理该异常，调用多路复用器的selectNow()方法将已经取消的selectionKey从多路复用器中删除掉。操作成功之后，将selected置为true， 说明之前失效的selectionKey已经被删除掉。继续发起下一次注册操作，如果成功则退出,
//3) 如果仍然发生CancelledKeyException异常,说明我们无法删除已经被取消的selectionKey,按照JDK的API说明，这种意外不应该发生。如果发生这种问题，则说明可能NIO的相关类库存在不可恢复的BUG,直接抛出CancelledKeyException异常到上层进行统一处理。

    protected void doRegister() throws Exception {
        boolean selected = false;
        for (;;) {
            try {
                selectionKey = javaChannel().register(eventLoop().unwrappedSelector(), 0, this);
                return;
            } catch (CancelledKeyException e) {
                if (!selected) {
                    // Force the Selector to select now as the "canceled" SelectionKey may still be
                    // cached and not removed because no Select.select(..) operation was called yet.
                    eventLoop().selectNow();
                    selected = true;
                } else {
                    // We forced a select operation on the selector before but the SelectionKey is still cached
                    // for whatever reason. JDK bug ?
                    throw e;
                }
            }
        }
    }

}
```

SelectionKey 常量值

```java

public abstract class SelectionKey {
    
	public static final int OP_READ = 1 << 0;   //读操作位
	public static final int OP_WRITE = 1 << 2;  //写操作位
	public static final int OP_CONNECT = 1 << 3;  //客户端连接操作位
	public static final int OP_ACCEPT = 1 << 4;  //服务端接受连接操作位

	//如果注册的操作位为0表示只是完成注册功能，说明对任何事件都不感兴趣
```

**doBeginRead()** 读之前的准备

```java
  @Override
    protected void doBeginRead() throws Exception {
        // Channel.read() or ChannelHandlerContext.read() was called
        final SelectionKey selectionKey = this.selectionKey;
        if (!selectionKey.isValid()) {
            return;   //key无效的话直接返回
        }

        readPending = true;  //表示读pending中

        final int interestOps = selectionKey.interestOps();
        if ((interestOps & readInterestOp) == 0) {  //表示当前没有读操作位
            selectionKey.interestOps(interestOps | readInterestOp);  //设置读操作位
        }

    
    }

  //SelectionKey中定义的是否可读操作
  public final boolean isReadable() {
        return (readyOps() & OP_READ) != 0;
    }
```

### 3.AbstractNioByteChannel

doWrite操作

配置中设置循环次数是避免半包中数据量过大，IO线程一直尝试写操作，此时IO线程无法处理其他IO操作或者定时任务，比如新的消息或者定时任务，如果网络IO慢或者对方读取慢等造成IO线程假死的状态.

```java
    @Override
    protected void doWrite(ChannelOutboundBuffer in) throws Exception {
        int writeSpinCount = -1; //        写自选次数

        boolean setOpWrite = false;  //写操作位为0
        for (;;) {
            Object msg = in.current();
            if (msg == null) {  //从环形数组ChannelOutboundBuffer弹出一条消息，如果为null，表示消息已经发送完成，
                // Wrote all messages.
                clearOpWrite();  //清除写标志位，退出循环
                // Directly return here so incompleteWrite(...) is not called.
                return;
            }

            if (msg instanceof ByteBuf) {
                ByteBuf buf = (ByteBuf) msg;
                int readableBytes = buf.readableBytes();
                if (readableBytes == 0) { //如果可读字节为0，则丢弃该消息，循环处理其他消息
                    in.remove();
                    continue;
                }

                boolean done = false;    //消息是否全部发送完毕表示
                long flushedAmount = 0;  //发送的字节数量
                if (writeSpinCount == -1) {
                    //如果为-1的时候从配置中获取写循环次数
                    writeSpinCount = config().getWriteSpinCount();
                }
                for (int i = writeSpinCount - 1; i >= 0; i --) {
                    int localFlushedAmount = doWriteBytes(buf);  //由子类实现写
                    if (localFlushedAmount == 0) {  //这里表示本次发送字节为0，发送TCP缓冲区满了，所以此时为了避免空循环一直发送，这里就将半包写表示设置为true并退出循环
                        setOpWrite = true;
                        break;
                    }
                    //发送成功就对发送的字节计数
                    flushedAmount += localFlushedAmount;
                    if (!buf.isReadable()) { //如果没有可读字节，表示已经发送完毕
                        done = true; //表示发送完成，并退出循环
                        break;
                    }
                }
                //通知promise当前写的进度
                in.progress(flushedAmount); 

                if (done) {  //如果发送完成，移除缓冲的数据
                    in.remove();
                } else {
                    如果没有完成会调用incompleteWrite方法
                    // Break the loop and so incompleteWrite(...) is called.
                    break;
                }
            } else if (msg instanceof FileRegion) {  //这个是文件传输和上面类似
                FileRegion region = (FileRegion) msg;
                boolean done = region.transferred() >= region.count();

                if (!done) {
                    long flushedAmount = 0;
                    if (writeSpinCount == -1) {
                        writeSpinCount = config().getWriteSpinCount();
                    }

                    for (int i = writeSpinCount - 1; i >= 0; i--) {
                        long localFlushedAmount = doWriteFileRegion(region);
                        if (localFlushedAmount == 0) {
                            setOpWrite = true;
                            break;
                        }

                        flushedAmount += localFlushedAmount;
                        if (region.transferred() >= region.count()) {
                            done = true;
                            break;
                        }
                    }

                    in.progress(flushedAmount);
                }

                if (done) {
                    in.remove();
                } else {
                    // Break the loop and so incompleteWrite(...) is called.
                    break;
                }
            } else {
                // Should not reach here.
                throw new Error();
            }
        }
        //如果没有完成写看看需要做的事情
        incompleteWrite(setOpWrite);
    }

//未完成写操作，看看操作
  protected final void incompleteWrite(boolean setOpWrite) {
        // Did not write completely.
        if (setOpWrite) {  //如果当前的写操作位true，那么当前多路复用器继续轮询处理
            setOpWrite();
        } else {  //否则重新新建一个task任务，让eventLoop后面点执行flush操作，这样其他任务才能够执行
            // Schedule flush again later so other tasks can be picked up in the meantime
            Runnable flushTask = this.flushTask;
            if (flushTask == null) {
                flushTask = this.flushTask = new Runnable() {
                    @Override
                    public void run() {
                        flush();
                    }
                };
            }
            eventLoop().execute(flushTask);
        }
    }
```

### 4.AbstractNioMessageChannel

doWrite操作

```java
    @Override
    protected void doWrite(ChannelOutboundBuffer in) throws Exception {
        final SelectionKey key = selectionKey();
        final int interestOps = key.interestOps();

        for (;;) {
            Object msg = in.current();
            if (msg == null) {
                // Wrote all messages.
                if ((interestOps & SelectionKey.OP_WRITE) != 0) {
                    key.interestOps(interestOps & ~SelectionKey.OP_WRITE);
                }
                break;
            }
            try {
                boolean done = false;
                for (int i = config().getWriteSpinCount() - 1; i >= 0; i--) {
                    if (doWriteMessage(msg, in)) {
                        done = true;
                        break;
                    }
                }

                if (done) {
                    in.remove();
                } else {
                    // Did not write all messages.
                    if ((interestOps & SelectionKey.OP_WRITE) == 0) {
                        key.interestOps(interestOps | SelectionKey.OP_WRITE);
                    }
                    break;
                }
            } catch (Exception e) {
                if (continueOnWriteError()) {
                    in.remove(e);
                } else {
                    throw e;
                }
            }
        }
    }
```

AbstractNioMessageChannel 和AbstractNioByteChannel的消息发送实现比较相似，

不同之处在于:一个发送的是ByteBuf或者FileRegion，它们可以直接被发送;另一个发送的则是POJO对象。