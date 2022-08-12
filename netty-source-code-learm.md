### Netty

#### 总体描述

Netty 是一个异步事件驱动网络应用（框架），快速开发可维护的高性能协议服务器和客户端

#### 详细描述

Netty 是一个基于NIO的客户、服务器端的编程框架，使用Netty 可以确保你快速和简单的开发出一个网络应用，例如实现了某种协议的客户、服务端应用。Netty相当于简化和流线化了网络应用的编程开发过程，例如：基于TCP和UDP的socket服务开发。

“快速”和“简单”并不用产生维护性或性能上的问题。Netty 是一个吸收了多种协议（包括FTP、SMTP、HTTP等各种二进制文本协议）的实现经验，并经过相当精心设计的项目。最终，Netty 成功的找到了一种方式，在保证易于开发的同时还保证了其应用的性能，稳定性和伸缩性

#### 特性

Ease of use (简单易用),
Performance（高性能）,
Security（安全）,
Community（社区服务）

#### 设计

Unified API for various transport types - blocking and non-blocking socket
Based on a flexible and extensible event model which allows clear separation of concerns
Highly customizable thread model - single thread, one or more thread pools such as SEDA
True connectionless datagram socket support (since 3.1)

各种传输类型的统一 API - 阻塞和非阻塞套接字
基于灵活且可扩展的事件模型，允许明确分离关注点
高度可定制的线程模型——单线程、一个或多个线程池，例如 SEDA
真正的无连接数据报套接字支持（自 3.1 起）

#### 简单例子

##### DiscardServer

1. 根据系统变量获取SSL，默认没有
2. 根据系统变量获取端口号，默认8080
3. 配置SSL
4. 创建主事件循环组
5. 创建从事件循环组
6. 创建服务端启动器
7. 把事件循环组放到启动器中
8. 设置服务端ServerSocket 通道为NioServerSocketChannel
9. 设置处理ServerSocket通道接收到的客户端事件处理器，即日志打印器
10. 设置socket客户端请求的处理器
11. 将处理器放入socket通道对象流水线对象中处理
12. 向流水线末尾添加一个自定义的数据处理器
13. 绑定端口，并接受传入进来的连接
14. 等待 server socket 关闭
15. 优雅的关闭 事件循环组

```
/**
 * Discards any incoming data.
 * 丢弃任何传入数据。
 */
public final class DiscardServer {

    // 根据系统变量获取SSL，默认没有
    static final boolean SSL = System.getProperty("ssl") != null;
    // 根据系统变量获取端口号，默认8080
    static final int PORT = Integer.parseInt(System.getProperty("port", "8009"));

    public static void main(String[] args) throws Exception {
        // Configure SSL.
        // 配置SSL
        final SslContext sslCtx;
        if (SSL) {
            SelfSignedCertificate ssc = new SelfSignedCertificate();
            sslCtx = SslContextBuilder.forServer(ssc.certificate(), ssc.privateKey()).build();
        } else {
            sslCtx = null;
        }

        // 创建主事件循环组
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        // 创建从事件循环组
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            // 创建服务端启动器
            ServerBootstrap b = new ServerBootstrap();
            // 把事件循环组放到启动器中
            b.group(bossGroup, workerGroup)
                    // 设置服务端ServerSocket 通道为NioServerSocketChannel
             .channel(NioServerSocketChannel.class)
                    // 设置处理ServerSocket通道接收到的客户端事件处理器，即日志打印器
             .handler(new LoggingHandler(LogLevel.INFO))
                    // 设置socket客户端请求的处理器
             .childHandler(new ChannelInitializer<SocketChannel>() {
                 @Override
                 // 将处理器放入socket通道对象流水线对象中处理
                 public void initChannel(SocketChannel ch) {
                     ChannelPipeline p = ch.pipeline();
                     if (sslCtx != null) {
                         p.addLast(sslCtx.newHandler(ch.alloc()));
                     }
                     // 向流水线末尾添加一个自定义的数据处理器
                     p.addLast(new DiscardServerHandler());
                 }
             });

            // Bind and start to accept incoming connections.
            // 绑定端口，并接受传入进来的连接
            ChannelFuture f = b.bind(PORT).sync();

            // Wait until the server socket is closed.
            // In this example, this does not happen, but you can do that to gracefully
            // shut down your server.
            // 等待 server socket 关闭
            f.channel().closeFuture().sync();
        } finally {
            // 优雅的关闭 事件循环组
            workerGroup.shutdownGracefully();
            bossGroup.shutdownGracefully();
        }
    }
}

```


##### DiscardServerHandler

```
/**
 * Handles a server-side channel.
 * 处理服务器端通道。
 */
public class DiscardServerHandler extends SimpleChannelInboundHandler<Object> {

    /**
     * 通道读取到数据回调的方法，什么也不做，只用来丢弃数据
     *
     * @param ctx           the {@link ChannelHandlerContext} which this {@link SimpleChannelInboundHandler}
     *                      belongs to
     * @param msg           the message to handle
     * @throws Exception
     */
    @Override
    public void channelRead0(ChannelHandlerContext ctx, Object msg) throws Exception {
        // discard
    }

    /**
     * Calls {@link ChannelHandlerContext#fireExceptionCaught(Throwable)} to forward
     * to the next {@link ChannelHandler} in the {@link ChannelPipeline}.
     *
     * 发生异常时的回调方法
     *
     * Sub-classes may override this method to change behavior.
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        // Close the connection when an exception is raised.
        // 打印堆栈日志
        cause.printStackTrace();
        // 关闭连接
        ctx.close();
    }
}
```

##### DiscardClient

```
/**
 * Keeps sending random data to the specified address.
 * 不断向指定地址发送随机数据。
 */
public final class DiscardClient {

    static final boolean SSL = System.getProperty("ssl") != null;
    static final String HOST = System.getProperty("host", "127.0.0.1");
    static final int PORT = Integer.parseInt(System.getProperty("port", "8009"));
    static final int SIZE = Integer.parseInt(System.getProperty("size", "256"));

    public static void main(String[] args) throws Exception {
        // Configure SSL.
        final SslContext sslCtx;
        if (SSL) {
            sslCtx = SslContextBuilder.forClient()
                .trustManager(InsecureTrustManagerFactory.INSTANCE).build();
        } else {
            sslCtx = null;
        }

        EventLoopGroup group = new NioEventLoopGroup();
        try {
            Bootstrap b = new Bootstrap();
            b.group(group)
             .channel(NioSocketChannel.class)
             .handler(new ChannelInitializer<SocketChannel>() {
                 @Override
                 protected void initChannel(SocketChannel ch) throws Exception {
                     ChannelPipeline p = ch.pipeline();
                     if (sslCtx != null) {
                         p.addLast(sslCtx.newHandler(ch.alloc(), HOST, PORT));
                     }
                     p.addLast(new DiscardClientHandler());
                 }
             });

            // Make the connection attempt.
            ChannelFuture f = b.connect(HOST, PORT).sync();

            // Wait until the connection is closed.
            f.channel().closeFuture().sync();
        } finally {
            group.shutdownGracefully();
        }
    }
}
```

##### DiscardClientHandler

```
/**
 * Handles a client-side channel.
 * 处理客户端通道。
 */
public class DiscardClientHandler extends SimpleChannelInboundHandler<Object> {

    private ByteBuf content;
    private ChannelHandlerContext ctx;

    @Override
    public void channelActive(ChannelHandlerContext ctx) {
        this.ctx = ctx;

        // Initialize the message.
        content = ctx.alloc().directBuffer(DiscardClient.SIZE).writeZero(DiscardClient.SIZE);

        // Send the initial messages.
        generateTraffic();
    }

    @Override
    public void channelInactive(ChannelHandlerContext ctx) {
        content.release();
    }

    @Override
    public void channelRead0(ChannelHandlerContext ctx, Object msg) throws Exception {
        // Server is supposed to send nothing, but if it sends something, discard it.
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        // Close the connection when an exception is raised.
        cause.printStackTrace();
        ctx.close();
    }

    long counter;

    private void generateTraffic() {
        // Flush the outbound buffer to the socket.
        // Once flushed, generate the same amount of traffic again.
        ctx.writeAndFlush(content.retainedDuplicate()).addListener(trafficGenerator);
    }

    private final ChannelFutureListener trafficGenerator = new ChannelFutureListener() {
        @Override
        public void operationComplete(ChannelFuture future) {
            if (future.isSuccess()) {
                generateTraffic();
            } else {
                future.cause().printStackTrace();
                future.channel().close();
            }
        }
    };
}
```
