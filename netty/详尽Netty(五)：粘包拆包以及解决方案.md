详尽Netty(五)：粘包拆包以及解决方案

1.简介：

在RPC框架中，粘包和拆包问题是必须解决一个问题，因为RPC框架中，各个微服务相互之间都是维系了一个TCP长连接，比如dubbo就是一个全双工的长连接。由于微服务往对方发送信息的时候，所有的请求都是使用的同一个连接，这样就会产生粘包和拆包的问题。

2.产生粘包和拆包的原因：

操作系统在发送TCP数据的时候，底层会有一个缓冲区，例如1024个字节大小，如果一次请求发送的数据量比较小，没达到缓冲区大小，TCP则会将多个请求合并为同一个请求进行发送，这就形成了粘包问题；如果一次请求发送的数据量比较大，超过了缓冲区大小，TCP就会将其拆分为多次发送，这就是拆包，也就是将一个大的包拆分为多个小包进行发送。

如下图展示了粘包和拆包的一个示意图：

![image-20210619164013779](https://gitee.com/fking86/images4typora/raw/master/20210619164113.png)

- A和B两个包都刚好满足TCP缓冲区的大小，或者说其等待时间已经达到TCP等待时长，从而还是使用两个独立的包进行发送；
- A和B两次请求间隔时间内较短，并且数据包较小，因而合并为同一个包发送给服务端；
- B包比较大，因而将其拆分为两个包B_1和B_2进行发送，而这里由于拆分后的B_2比较小，其又与A包合并在一起发送。

3.常见解决方案：

- 客户端在发送数据包的时候，每个包都固定长度，比如1024个字节大小，如果客户端发送的数据长度不足1024个字节，则通过补充空格的方式补全到指定长度；
- 客户端在每个包的末尾使用固定的分隔符，例如\r\n，如果一个包被拆分了，则等待下一个包发送过来之后找到其中的\r\n，然后对其拆分后的头部部分与前一个包的剩余部分进行合并，这样就得到了一个完整的包；
- 将消息分为头部和消息体，在头部中保存有当前整个消息的长度，只有在读取到足够长度的消息之后才算是读到了一个完整的消息；
- 通过自定义协议进行粘包和拆包的处理。

4.Netty粘包拆包解决方案:

  1.固定长度的拆包器 FixedLengthFrameDecoder，每个应用层数据包的都拆分成都是固定长度的大小



server:

```
public class EchoServer {

  public void bind(int port) throws InterruptedException {
    EventLoopGroup bossGroup = new NioEventLoopGroup();
    EventLoopGroup workerGroup = new NioEventLoopGroup();
    try {
      ServerBootstrap bootstrap = new ServerBootstrap();
      bootstrap.group(bossGroup, workerGroup)
        .channel(NioServerSocketChannel.class)
        .option(ChannelOption.SO_BACKLOG, 1024)
        .handler(new LoggingHandler(LogLevel.INFO))
        .childHandler(new ChannelInitializer<SocketChannel>() {
          @Override
          protected void initChannel(SocketChannel ch) throws Exception {
            // 这里将FixedLengthFrameDecoder添加到pipeline中，指定长度为20
            ch.pipeline().addLast(new FixedLengthFrameDecoder(20));
            // 将前一步解码得到的数据转码为字符串
            ch.pipeline().addLast(new StringDecoder());
            // 这里FixedLengthFrameEncoder是我们自定义的，用于将长度不足20的消息进行补全空格
            ch.pipeline().addLast(new FixedLengthFrameEncoder(20));
            // 最终的数据处理
            ch.pipeline().addLast(new EchoServerHandler());
          }
        });

      ChannelFuture future = bootstrap.bind(port).sync();
      future.channel().closeFuture().sync();
    } finally {
      bossGroup.shutdownGracefully();
      workerGroup.shutdownGracefully();
    }
  }

  public static void main(String[] args) throws InterruptedException {
    new EchoServer().bind(8080);
  }
}
```

FixedLengthFrameEncoder:

```
public class FixedLengthFrameEncoder extends MessageToByteEncoder<String> {
  private int length;

  public FixedLengthFrameEncoder(int length) {
    this.length = length;
  }

  @Override
  protected void encode(ChannelHandlerContext ctx, String msg, ByteBuf out)
      throws Exception {
    // 对于超过指定长度的消息，这里直接抛出异常
    if (msg.length() > length) {
      throw new UnsupportedOperationException(
          "message length is too large, it's limited " + length);
    }

    // 如果长度不足，则进行补全
    if (msg.length() < length) {
      msg = addSpace(msg);
    }

    ctx.writeAndFlush(Unpooled.wrappedBuffer(msg.getBytes()));
  }

  // 进行空格补全
  private String addSpace(String msg) {
    StringBuilder builder = new StringBuilder(msg);
    for (int i = 0; i < length - msg.length(); i++) {
      builder.append(" ");
    }

    return builder.toString();
  }
}
```



EchoServerHandler:

```
public class EchoServerHandler extends SimpleChannelInboundHandler<String> {

  @Override
  protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {
    System.out.println("server receives message: " + msg.trim());
    ctx.writeAndFlush("hello client!");
  }
}
```



EchoClient:

```
public class EchoClient {

  public void connect(String host, int port) throws InterruptedException {
    EventLoopGroup group = new NioEventLoopGroup();
    try {
      Bootstrap bootstrap = new Bootstrap();
      bootstrap.group(group)
        .channel(NioSocketChannel.class)
        .option(ChannelOption.TCP_NODELAY, true)
        .handler(new ChannelInitializer<SocketChannel>() {
          @Override
          protected void initChannel(SocketChannel ch) throws Exception {
            // 对服务端发送的消息进行粘包和拆包处理，由于服务端发送的消息已经进行了空格补全，
            // 并且长度为20，因而这里指定的长度也为20
            ch.pipeline().addLast(new FixedLengthFrameDecoder(20));
            // 将粘包和拆包处理得到的消息转换为字符串
            ch.pipeline().addLast(new StringDecoder());
            // 对客户端发送的消息进行空格补全，保证其长度为20
            ch.pipeline().addLast(new FixedLengthFrameEncoder(20));
            // 客户端发送消息给服务端，并且处理服务端响应的消息
            ch.pipeline().addLast(new EchoClientHandler());
          }
        });

      ChannelFuture future = bootstrap.connect(host, port).sync();
      future.channel().closeFuture().sync();
    } finally {
      group.shutdownGracefully();
    }
  }

  public static void main(String[] args) throws InterruptedException {
    new EchoClient().connect("127.0.0.1", 8080);
  }
}
```



EchoClientHandler:

```
public class EchoClientHandler extends SimpleChannelInboundHandler<String> {

  @Override
  protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {
    System.out.println("client receives message: " + msg.trim());
  }

  @Override
  public void channelActive(ChannelHandlerContext ctx) throws Exception {
    ctx.writeAndFlush("hello server!");
  }
}
```

  

2.分隔符拆包器 DelimiterBasedFrameDecoder，每个应用层数据包，都通过自定义的分隔符，进行分割拆分

EchoServer（替换上面得initChannel）: 

```
@Override
protected void initChannel(SocketChannel ch) throws Exception {
    String delimiter = "_$";
    // 将delimiter设置到DelimiterBasedFrameDecoder中，经过该解码一器进行处理之后，源数据将会
    // 被按照_$进行分隔，这里1024指的是分隔的最大长度，即当读取到1024个字节的数据之后，若还是未
    // 读取到分隔符，则舍弃当前数据段，因为其很有可能是由于码流紊乱造成的
    ch.pipeline().addLast(new DelimiterBasedFrameDecoder(1024,
        Unpooled.wrappedBuffer(delimiter.getBytes())));
    // 将分隔之后的字节数据转换为字符串数据
    ch.pipeline().addLast(new StringDecoder());
    // 这是我们自定义的一个编码器，主要作用是在返回的响应数据最后添加分隔符
    ch.pipeline().addLast(new DelimiterBasedFrameEncoder(delimiter));
    // 最终处理数据并且返回响应的handler
    ch.pipeline().addLast(new EchoServerHandler());
}
```



DelimiterBasedFrameEncoder：

```
public class DelimiterBasedFrameEncoder extends MessageToByteEncoder<String> {

  private String delimiter;

  public DelimiterBasedFrameEncoder(String delimiter) {
    this.delimiter = delimiter;
  }

  @Override
  protected void encode(ChannelHandlerContext ctx, String msg, ByteBuf out) 
      throws Exception {
    // 在响应的数据后面添加分隔符
    ctx.writeAndFlush(Unpooled.wrappedBuffer((msg + delimiter).getBytes()));
  }
}
```



EchoClient (替换上面得 EchoClient  中得initChannel):

```
@Override
protected void initChannel(SocketChannel ch) throws Exception {
    String delimiter = "_$";
    // 对服务端返回的消息通过_$进行分隔，并且每次查找的最大大小为1024字节
    ch.pipeline().addLast(new DelimiterBasedFrameDecoder(1024, 
        Unpooled.wrappedBuffer(delimiter.getBytes())));
    // 将分隔之后的字节数据转换为字符串
    ch.pipeline().addLast(new StringDecoder());
    // 对客户端发送的数据进行编码，这里主要是在客户端发送的数据最后添加分隔符
    ch.pipeline().addLast(new DelimiterBasedFrameEncoder(delimiter));
    // 客户端发送数据给服务端，并且处理从服务端响应的数据
    ch.pipeline().addLast(new EchoClientHandler());
}
```





  3.行拆包器 LineBasedFrameDecoder，每个应用层数据包，都以换行符作为分隔符，进行分割拆分 

EchoServer：

```
@Override
protected void initChannel(SocketChannel ch) throws Exception {
	// 以下两行代码为了解决半包读问题
	ch.pipeline().addLast(new LineBasedFrameDecoder(1024));
	ch.pipeline().addLast(new StringDecoder());
	ch.pipeline().addLast(new TimeServerHandler());

}
```



TimeServerHandler：

```java
public class TimeServerHandler extends ChannelInboundHandlerAdapter {
    private int counter;

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        String body = (String) msg;
        System.out.println("The time server receive order:" + body + ";the counter is:" + (++counter));
        String currentTime = "QUERY TIME ORDER".equalsIgnoreCase(body) ? new Date(System.currentTimeMillis()).toString()
                : "BAD ORDER";
        currentTime = currentTime + System.getProperty("line.separator");
        ByteBuf resp = Unpooled.copiedBuffer(currentTime.getBytes());
        ctx.write(resp);
        ReferenceCountUtil.release(msg);
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        ctx.flush();
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}
```



EchoClient：

```java
@Override
protected void initChannel(SocketChannel ch) throws Exception {
    // 代码的关键部分，添加行解析器，添加stringDecoder，再添加业务类
    ch.pipeline().addLast(new LineBasedFrameDecoder(1024));
    ch.pipeline().addLast(new StringDecoder());
    ch.pipeline().addLast(new TimeClientHandler());
}
```



TimeClientHandler:

```java
public class TimeClientHandler extends ChannelInboundHandlerAdapter {
    private static final Logger logger = Logger.getLogger(LineTimeClientHandler.class.getName());

    private int counter;
    private byte[] req;

    public LineTimeClientHandler() {
        req = ("QUERY TIME ORDER" + System.getProperty("line.separator")).getBytes();
    }

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        ByteBuf message = null;
        for (int i = 0; i < 100; i++) {
            message = Unpooled.buffer(req.length);
            message.writeBytes(req);
            ctx.writeAndFlush(message);
        }
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        String body = (String) msg;
        System.out.println("Now is:" + body + "; the counter is:" + (++counter));
        ReferenceCountUtil.release(msg);
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        logger.warning("Unexcepted exception from downstream:" + cause.getMessage());
        ctx.close();
    }
}
```



  4.基于数据包长度的拆包器 LengthFieldBasedFrameDecoder，将应用层数据包的长度，作为接收端应用层数据包的拆分依据。按照应用层数据包的大小，拆包。这个拆包器，有一个要求，就是应用层协议中包含数据包的长度



![image-20210619193846517](https://gitee.com/fking86/images4typora/raw/master/20210619193848.png)

关于LengthFieldBasedFrameDecoder，这里需要对其构造函数参数进行介绍：

- maxFrameLength：指定了每个包所能传递的最大数据包大小；
- lengthFieldOffset：指定了长度字段在字节码中的偏移量；
- lengthFieldLength：指定了长度字段所占用的字节长度；
- lengthAdjustment：对一些不仅包含有消息头和消息体的数据进行消息头的长度的调整，这样就可以只得到消息体的数据，这里的lengthAdjustment指定的就是消息头的长度；
- initialBytesToStrip：对于长度字段在消息头中间的情况，可以通过initialBytesToStrip忽略掉消息头以及长度字段占用的字节。



EchoServer：

```
public class EchoServer {

  public void bind(int port) throws InterruptedException {
    EventLoopGroup bossGroup = new NioEventLoopGroup();
    EventLoopGroup workerGroup = new NioEventLoopGroup();
    try {
      ServerBootstrap bootstrap = new ServerBootstrap();
      bootstrap.group(bossGroup, workerGroup)
        .channel(NioServerSocketChannel.class)
        .option(ChannelOption.SO_BACKLOG, 1024)
        .handler(new LoggingHandler(LogLevel.INFO))
        .childHandler(new ChannelInitializer<SocketChannel>() {
          @Override
          protected void initChannel(SocketChannel ch) throws Exception {
            // 这里将LengthFieldBasedFrameDecoder添加到pipeline的首位，因为其需要对接收到的数据
            // 进行长度字段解码，这里也会对数据进行粘包和拆包处理
            ch.pipeline().addLast(new LengthFieldBasedFrameDecoder(1024, 0, 2, 0, 2));
            // LengthFieldPrepender是一个编码器，主要是在响应字节数据前面添加字节长度字段
            ch.pipeline().addLast(new LengthFieldPrepender(2));
            // 对经过粘包和拆包处理之后的数据进行json反序列化，从而得到User对象
            ch.pipeline().addLast(new JsonDecoder());
            // 对响应数据进行编码，主要是将User对象序列化为json
            ch.pipeline().addLast(new JsonEncoder());
            // 处理客户端的请求的数据，并且进行响应
            ch.pipeline().addLast(new EchoServerHandler());
          }
        });

      ChannelFuture future = bootstrap.bind(port).sync();
      future.channel().closeFuture().sync();
    } finally {
      bossGroup.shutdownGracefully();
      workerGroup.shutdownGracefully();
    }
  }

  public static void main(String[] args) throws InterruptedException {
    new EchoServer().bind(8080);
  }
}
```



JsonDecoder：

```
public class JsonDecoder extends MessageToMessageDecoder<ByteBuf> {

  @Override
  protected void decode(ChannelHandlerContext ctx, ByteBuf buf, List<Object> out) 
      throws Exception {
    byte[] bytes = new byte[buf.readableBytes()];
    buf.readBytes(bytes);
    User user = JSON.parseObject(new String(bytes, CharsetUtil.UTF_8), User.class);
    out.add(user);
  }
}
```



JsonEncoder：

```
public class JsonEncoder extends MessageToByteEncoder<User> {

  @Override
  protected void encode(ChannelHandlerContext ctx, User user, ByteBuf buf)
      throws Exception {
    String json = JSON.toJSONString(user);
    ctx.writeAndFlush(Unpooled.wrappedBuffer(json.getBytes()));
  }
}
```



EchoServerHandler：

```
public class EchoServerHandler extends SimpleChannelInboundHandler<User> {

  @Override
  protected void channelRead0(ChannelHandlerContext ctx, User user) throws Exception {
    System.out.println("receive from client: " + user);
    ctx.write(user);
  }
}
```



EchoClient:

```
@Override
protected void initChannel(SocketChannel ch) throws Exception {
    ch.pipeline().addLast(new LengthFieldBasedFrameDecoder(1024, 0, 2, 0, 2));
    ch.pipeline().addLast(new LengthFieldPrepender(2));
    ch.pipeline().addLast(new JsonDecoder());
    ch.pipeline().addLast(new JsonEncoder());
    ch.pipeline().addLast(new EchoClientHandler());
}
```



EchoClientHandler:

```
public class EchoClientHandler extends SimpleChannelInboundHandler<User> {

  @Override
  public void channelActive(ChannelHandlerContext ctx) throws Exception {
    ctx.write(getUser());
  }

  private User getUser() {
    User user = new User();
    user.setAge(27);
    user.setName("zhangxufeng");
    return user;
  }

  @Override
  protected void channelRead0(ChannelHandlerContext ctx, User user) throws Exception {
    System.out.println("receive message from server: " + user);
  }
}
```



5.自定义粘包与拆包器

继承 ByteToMessageDecoder 解码

```
public abstract class ByteToMessageDecoder extends ChannelInboundHandlerAdapter {
    protected abstract void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) 
        throws Exception;
}
```



继承MessageToByteEncoder 编码

```
public abstract class MessageToByteEncoder<I> extends ChannelOutboundHandlerAdapter {
    protected abstract void encode(ChannelHandlerContext ctx, I msg, ByteBuf out) 
        throws Exception;
}
```