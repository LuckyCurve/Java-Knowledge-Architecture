# 序



起因只是想将Netty作为工具，来实现一个简单的RPC框架，后来发现困难重重，还是应该系统的进行Netty的学习了。

这本书的序让我感觉到了人类的悲喜或许是相通的，我也曾经认为仅依靠Web服务器和HTTP协议可以解决绝大多数的问题，但是随着流量的增加，传统三层架构不支持了，需要进行水平扩展，水平扩展自然涉及到机器通信的问题，如果是依照HTTP协议来进行通信，那么消耗会非常大，涉及到封包拆包等复杂操作，若直接使用Java提供的java.net下面的类来编写，写出来的程序是阻塞的不说，繁杂的异常处理就够我们学习非常多的编程范式了，自然是不划算的。

于是在这种场景下，遇到了Netty，高度可订制化，提供了传输层和应用层绝大多数通信协议的实现，集成了安全加密等功能，并且本身是基于Reactor模型来进行开发的，拥有活跃的开源社区，虽然可能Bug是有亿点点多（不得不吐槽下了，对新手来说使用起来确实没有Spring那样友好，会报一些莫名其妙的错，而这些错误很有可能涉及到泛型等问题，很容易让我们怀疑自己的Java功底）





# 第一部分：Netty的概念及体系结构



Netty是用于**创建**高性能网络应用程序的高级框架，造轮子的首选





## 第一章、Netty——异步和事件驱动



Wikipedia对Netty的定义：Netty是一款异步的，事件驱动的网络应用程序框架，支持快速地开发可维护的高性能的**面向协议**的服务器和客户端。

就高性能而言，Netty已经帮我们承担了非常多了，可能是根据他良好的架构，严密的技术选型......总之这些对一个Netty初学者来说都是不能确定的。

早期网络编程是非常困难的，是基于C语言的套接字库的，然而C语言没有做跨平台的处理，因此对于不同的平台可能遇到各种各样的问题，在Java出现后情况得以好转，但是仍然需要使用大量的代码模板才能使得整个项目运行起来，并且只支持阻塞方式（BIO方式）。

存在几个地方的阻塞：

1、ServerSocket#accept方法是阻塞的，直到建立起一个连接

2、IO是阻塞的，需要全部操作完成之后才会返回

因为是阻塞方式的，所以当请求数量特别大的时候，会存在非常多的线程处于阻塞状态，即使JVM可以支持这么多的线程数，但是线程的上下文切换会有非常大的开销。



在1.4的时候引入了java.nio库，这种模型较之于传统的IO，可以具有更好的资源管理

BIO：

![image-20210428194630875](https://gitee.com/LuckyCurve/img/raw/master//img/image-20210428194630875.png)

NIO：

![image-20210428194636307](https://gitee.com/LuckyCurve/img/raw/master//img/image-20210428194636307.png)

当线程没有IO任务的时候，可以去进行其他工作，或者是处理别的Socket的IO操作，资源管理方式自然是更好了。

NIO好是好，但是编程嘛，讲究的是一个平衡，很可能我们无法平衡安全性和高负载情况下的高效性，所幸我们可以将这个任务交给Netty进行完成，Netty可以让你将原本的精力从学习底层繁杂的编码中摆脱出来，更专注于你想要去完成自己想要做的事儿。

:warning:**Netty对异步是有支持的**：一直将Netty当做是NIO的一个最佳实践，Netty其实更像只是将NIO当做了一个工具而已，提供了非常多的其他特性。



Netty核心组件：

- Channel

借用于NIO的一个概念，可以理解成是传入或者传出数据的载体，相当于通过Channel我们可以进行IO操作，可以被打开或者是关闭，连接或者是断开连接

- 回调

回调其实就是一个指向方法的引用，特别是在1.8之后，方法可以作为一等公民，和基本数据类型和对象一起被当做参数进行传递，在指定的时间点执行回调方法

- Future

完成异步执行完成后结果的占位符，借用的是JUC包下面的Future这个类，但是JDK的这个Future使用起来比较繁琐，需要手动检查时候完成`boolean isDone()`或者是尝试获取预期结果`V get()`如果获取不到就会一直阻塞，当然提供了超时方法`V get(long timeout, TimeUnit unit)`

Netty提供了ChannelFuture，通过ChannelFuture我们可以完成Listener的注册，这些Listener会在执行完成时候被调用，这就是异步最直观的体现形式了，代替了原本JUC下笨拙的Future去检查时候完成的必要了。

> Netty最明显的两大特性：异步和事件驱动

- 事件和ChannelHandler

因为Netty是基于事件驱动的，事件大体上可以分为入站事件和出站事件，都可以被我们实现的ChannelHandler所捕获

![image-20210428202012611](https://gitee.com/LuckyCurve/img/raw/master//img/image-20210428202012611.png)

可以理解成响应事件而被执行的回调操作

- 选择器、事件和EventLoop

Selector更像是一个抽象模型，具体的实现还是基于事件驱动的，在内部，每个Channel都会被分配一个EventLoop，用以处理所有事件，包括：

1、注册感兴趣的事件

2、将事件派发给ChannelHandler

3、安排进一步的动作

EventLoop有点像是Redis中的文件事件管理器，只不过这个文件事件管理器是对一个Channel来说的



小结：

了解了Netty重要特性与核心组件，更重要的是了解到了Netty是干什么的，这才是我认为学习最大乐趣之所在。







## 第二章、你的第一款Netty应用程序



Netty服务器必不可少的两部分：

ChannelHandler：用于保存服务器从客户端接收的数据的处理

引导：配置服务器的启动代码，至少需要绑定到服务器指定的端口上



选用ChannelHandler非常重要，一般不会直接实现ChannelHandler接口，而是选用他的子接口如ChannelInboundHandler，但是接口有一个不好的地方，需要你实现其中所有的方法，Netty想到了这一点，提供了ChannelInboundHandlerAdaptor，我们可以继承这个类，他提供了ChannelInboundHandler的默认实现，直接用吧，方法的作用在代码注释当中。

```java
/**
 * Server Handler实现
 *
 * @author LuckyCurve
 */
@ChannelHandler.Sharable        // 标记当前ChannelHandler可以安全的被多个Channel共享
public class EchoServerHandler extends ChannelInboundHandlerAdapter {
    /**
     * 对每个传入的消息都要调用
     *
     * @param ctx
     * @param msg
     * @throws Exception
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf buf = (ByteBuf) msg;
        System.out.println("Server received：" + buf.toString(CharsetUtil.UTF_8));
        // 将消息重新发给发送者，不冲刷出站消息
        ctx.write(buf);
    }

    /**
     * 对当前ChannelHandler最后一次channelRead调用会重定向到当前方法上来
     *
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        // 在最后一次处理接收到的消息的时候将所有消息冲刷给发送者，
        ctx.writeAndFlush(Unpooled.EMPTY_BUFFER)
                // 在冲刷完成之后关闭Channel
                .addListener(ChannelFutureListener.CLOSE);
    }

    /**
     * 读取操作期间有异常抛出时候会进行调用
     *
     * @param ctx
     * @param cause
     * @throws Exception
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        // 关闭该Channel
        ctx.close();
    }
}
```

Netty的异常处理也比较有意思，是通过这里的exceptionCaught方法来完成对抛出的Throwable捕获并处理的，如果ChannelHandler没有捕获异常会怎么样呢，这得从ChannelHandler的调用链路说起，Channel内部是持有一个ChannelPipeline，在这之中是记录着了一个ChannelHandler的调用链，如果当前ChannelHandler没有实现异常捕获，那么这个异常会一直沿着这个链路向下传递。

ChannelHandler是一个非常重要的接口了，可以理解成通过ChannelHandler我们可以完成对事件处理逻辑的织入，具有众多的子接口和实现需要我们去了解。



下面来编写服务器端的引导，主要如下两个步骤：

1、绑定到指定端口上去

2、配置Channel，将入站消息发送到我们自定义的ChannelHandler里面去

```java
/**
 * @author LuckyCurve
 */
public class EchoServer {

    private final Integer port;

    public EchoServer(Integer port) {
        this.port = port;
    }

    public void start() throws InterruptedException {
        final EchoServerHandler serverHandler = new EchoServerHandler();
        // 只使用一个线程组
        NioEventLoopGroup group = new NioEventLoopGroup();

        // 经典编程范式
        try {
            // 创建引导
            ServerBootstrap bootstrap = new ServerBootstrap();
            // 属性设置
            bootstrap.group(group)
                    // 指定传输Channel
                    .channel(NioServerSocketChannel.class)
                    // 使用指定的端口设置套接字地址，主要是绑定端口号，不需要在下面的bind指定Port了
                    .localAddress(new InetSocketAddress(port))
                    // 完成对新Channel的创建，将EchoServerHandler指定为ChannelPipeline
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(serverHandler);
                        }
                    });

            // 异步的完成绑定，然后调用sync方法阻塞直到绑定完成
            ChannelFuture future = bootstrap.bind().sync();

            // 阻塞直到服务器的Channel关闭
            future.channel().closeFuture().sync();

        } finally {
            group.shutdownGracefully();
        }
    }


    /**
     * 完成开启
     *
     * @param args
     */
    public static void main(String[] args) throws InterruptedException {
        if (args.length != 1) {
            System.err.println("Usage:" + EchoServer.class.getSimpleName() + " <port>");
            System.exit(-1);
        }

        int port = Integer.parseInt(args[0]);
        new EchoServer(port).start();
    }
}
```





客户端部分

ChannelHandler实现：

```java
public class EchoClientHandler extends SimpleChannelInboundHandler<ByteBuf> {

    /**
     * 到服务器的连接建立之后调用
     *
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        // 连接建立之后，发送消息
        ctx.writeAndFlush(Unpooled.copiedBuffer("Netty rocks!", CharsetUtil.UTF_8));
    }

    /**
     * 从服务器接收到消息时候调用
     *
     * @param ctx
     * @param msg
     * @throws Exception
     */
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, ByteBuf msg) throws Exception {
        System.out.println("Client received:" + msg.toString(CharsetUtil.UTF_8));
    }

    /**
     * 处理过程中被调用
     *
     * @param ctx
     * @param cause
     * @throws Exception
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }


}
```

使用到了SimpleChannelInboundHandler，之所以没有像服务器那样选择扩展ChannelInboundHandlerAdaptor，原因如下：

![image-20210429161958761](https://gitee.com/LuckyCurve/img/raw/master//img/image-20210429161958761.png)

主要是SimpleChannelInboundHandler在读取完数据之后可以立马自动释放内存当中的数据了，具有这个高级特性。

客户端引导类：

```java
public class EchoClient {

    private final String host;

    private final Integer port;

    public EchoClient(String host, Integer port) {
        this.host = host;
        this.port = port;
    }

    public void start() throws InterruptedException {
        NioEventLoopGroup group = new NioEventLoopGroup();
        try {
            Bootstrap bootstrap = new Bootstrap();

            // 常规设置
            bootstrap.group(group)
                    .channel(NioSocketChannel.class)
                    .remoteAddress(host, port)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new EchoClientHandler());
                        }
                    });

            // 依旧是阻塞到启动
            ChannelFuture future = bootstrap.connect().sync();

            // 阻塞到关闭
            future.channel().closeFuture().sync();
        } finally {
            group.shutdownGracefully();
        }
    }

    public static void main(String[] args) {
        if (args.length != 2) {
            System.err.println("Usage:" + EchoClient.class.getSimpleName() + " <host> <port>");
        }

        new EchoClient(args[0], Integer.parseInt(args[1]));
    }

}
```

可以允许客户端和服务器不使用相同的IO模型

完成字符串的传输，可以直接使用阿里的fastjson组件作为序列化工具来完成其他对象的传输



小结：

了解了大体的一个开发流程了，还是能让业务和引导分开来的，使用起来初步感受还是蛮不错的，可能对ByteBuf这种容器没有太多了解，使用起来始终感觉束手束脚。







## 第三章、Netty的组件和设计



对接触过的概念和组件进行一个简单的总结

- Channel

基本IO操作（bind、connect、read、write）原语。基本的构造就是java.net.Socket，并且使用Channel可以大大降低使用Socket的复杂性问题，并且提供了非常多的实现用于解决其中遇到的问题。

- EventLoop

![image-20210429220555145](https://gitee.com/LuckyCurve/img/raw/master//img/image-20210429220555145.png)

处理注册进EventLoop当中Channel所发生的事件

之间存在的关系是：

1. EventLoopGroup可以包含一个或者是多个EventLoop
2. 一个EventLoop可以和一个Thread绑定
3. 所有EventLoop捕捉到的IO事件必然在这个绑定了的Thread上执行
4. 一个Channel只能注册进一个EventLoop当中去
5. 一个EventLoop可以包含一个或者是多个Channel

其实对于特定的Channel，他会将IO事件交给到给定的EventLoop，EventLoop会指定唯一的一个Thread来进行事件的执行，因此Channel对应Thread是唯一的。

- ChannelFuture

在我看来这个接口是Netty对异步的支持，因为不会立马返回，JDK提供的Future之事后需要我们去手动进行，因此是同步的，但是Netty提供的ChannelFuture允许我们对其添加ChannelFutureListener来完成当某项操作完成了的时候可以得到通知。

- ChannelHandler

从开发角度看的主要组件，大部分的处理逻辑都是封装在这里的，当某些事件发生之后，可以对其中的方法进行回调，其实主要使用的就是ChannelInBoundHandler这个子接口，你的业务逻辑通常驻留在一个或者多个ChannelInBoundHandler中。

Netty提供了大量的ChannelHandler子接口，主要功能都取决于这些子接口，直接使用子接口是非常麻烦的，因此Netty官方以适配器的模式给出了大量的默认实现，使用这些默认实现，内部已经通过对ChannelHandlerContext的默认处理来将消息发送给下一个ChannelHandler，因此我们使用的时候只需要重写需要重写的方法即可，主要的几个适配器类有：

1. ChannelHandlerAdapter
2. ChannelInboundHandlerAdaptor
3. ChannelOutboundHandlerAdaptor
4. ChannelDuplexHandler

- ChannelPipeline

Channel在创建时候，内部会默认分配一个ChannelPipeline，ChannelPipeline其中就包含着ChannelHandler调用链，在ChannelPipeline中注册ChannelHandler的大体过程通常是：将ChannelInitializer通过handler方法完成向bootstrap的注册，在创建Channel的时候会调用我们注册的ChannelInitializer的initChannel方法，来完成对ChannelPipeline中的ChannelHandlerChain的设置。

实际上，被我们称为ChannelPipeline的是这些ChannelHandler的编排顺序，也就是ChannelHandler调用链路。

在ChannelHandler中大多数的方法参数包含ChannelHandlerContext，这个ChannelHandlerContext可以让我们完成直接对下一个ChannelHandler的调用，这个逻辑是默认封存在ChannelInboundHandlerAdaptor和ChannelOutboundHandlerAdaptor当中的，因此我们可以直接重写这些类的方法来完成对处理逻辑的织入。

ChannelHandlerContext代表了ChannelHandler和ChannelPipeline之间的绑定，虽然这个对象可以用来获取底层的Channel，但是他主要还是用于写出站数据。

Netty支持两种消息发送的方式：直接将数据写入到Channel中，这种方式会导致消息从ChannelPipeline的尾部开始流动，或者是将数据写入到ChannelHandlerContext当中，这样在ChannelPipeline的下一个ChannelHandler就可以开始数据的流动了。

> 第一种其实是非常类似于数据库读取时候的MVCC机制，可以保证在数据流动时候不能直接响应内部的消息，保存接收数据时候的现状。





- 对ChannelHandler常用3个子类型的讨论

编码器、解码器、`SimpleChannelInboundHandler<T>`，ChannelInboundHandlerAdaptor的一个子类

编码器和解码器存在的必要：

在网络传输过程中都是字节流，因此在出站时候需要将消息转换为字节码，在入站时候完成字节码到消息的转换，通常消息的存放形式为Java对象。

当然Netty提供了非常多的抽象类来完成编解码器的实现，因为有可能你不会直接将消息直接转换成字节，你可能会将其以某种中间格式暂存，而不是立刻转换成字符流，当然Netty也是支持这个的。

对所有Netty内置的编码器和解码器，都实现了ChannelInboundHandler或者是ChannelOutboundHandler接口，在我们实现编解码器的时候命名应该基于ByteToMessageDecoder或MessageToByteEncoder来，或者是ProtobufEncoder或ProtobufDecoder这样的名称，然后在引导类当中完成绑定即可。

当然使用起来是比较麻烦的，我们最常见的情况是需要这样一个ChannelHandler，通过解码器来完成消息的解码，并可以完成业务逻辑的书写， 这里就需要使用到``SimpleChannelInboundHandler<T>`这样一个ChannelHandler，T为转换过后的Java类型。

在这种类型的ChannelHandler中，最重要的是channelRead0，因为它的入参是ChannelHandlerContext和T，使用起来还是非常方便的。



- 引导

Netty引导为应用程序的网络层配置提供了容器，完成进程到端口的绑定，或者完成进程到另一个正在运行在指定主机的端口号上的连接。

前者为引导一个服务器，后者为引导一个客户端，决定使用Bootstrap或者是ServerBootstrap，仅仅只是取决于使用的是服务端还是客户端。

引导一个客户端只需要一个EventLoopGroup，引导一个服务器需要两个（也可以是同一实例）

之所以服务器端需要两组不同的Channel，通过EventLoopGroup来进行一个承载，第一组只包含一个ServerChannel完成对特定端口的绑定和监听，第二组则是完成监听每一个客户端连接的Channel。



小结：

总结了一些核心抽象，如ChannelHandler、ChannelPipeline、编/解码器的概念和使用的主要实现类。





## 第四章、传输



Netty在传输方式当中可以进行无缝切换，与Java提供的BIO和NIO的类库使用上差距很大截然不同，Netty屏蔽了底层复杂的细节处理，我们可以将时间更多的花在有成效的事情上面。

本章的主要目的就是讲述Netty对JDK底层的通信机制的封装

举了一个客户端编码，返回Hi的示例，从Java BIO切换到NIO会感觉到代码决然不同

但是使用Netty仅仅需要改变的就是将EventLoopGroup的类型由OioEventLoopGroup（现在直接标注了`@Deprecated`注解表示不推荐使用）。

所有的传输都依赖Channel、ChannelPipeline和ChannelHandler来完成

- Channel

层次结构为：

![image-20210503105843905](https://gitee.com/LuckyCurve/img/raw/master//img/image-20210503105843905.png)

> 虚线表示继承关系，实线表示持有关系

每个Channel持有唯一的对象ChannelPipeline和ChannelConfig，ChannelConfig不仅只是包含该Channel的所有配置，还支持热更新。

ChannelPipeline持有入站和出站数据以及事件的ChannelHandler实例

ChannelHandler的典型用途：

- 将数据从一种格式转换为另一种格式（数据处理）
- 异常通知
- 对Channel进行改变
- 提供当Channel注册到EventLoop或者从EventLoop注销时的通知
- 提供用户自定义事件的通知

ChannelHandler支持动态的向ChannelPipeline中插入或者是删除，如面对特定请求时候，我们需要加入特定的ChannelHandler来完成对数据的处理

Channel常见方法：

![image-20210507212350590](https://gitee.com/LuckyCurve/img/raw/master//img/image-20210507212350590.png)

> 对flush方法的简单理解：只有调用了flush方法才能将消息发送出去

简单的Channel写入数据的方法实现：

```java
@Override
public void channelActive(ChannelHandlerContext ctx) throws Exception {
    Channel channel = ctx.channel();
    ByteBuf buf = Unpooled.copiedBuffer("message info", CharsetUtil.UTF_8);
    // 完成对数据的写入
    ChannelFuture future = channel.writeAndFlush(buf);
    // 添加监视器
    future.addListener(new ChannelFutureListener() {
        @Override
        public void operationComplete(ChannelFuture future) throws Exception {
            if (future.isSuccess()) {
                System.out.println("write success");
            } else {
                System.out.println("write error");
                future.cause().printStackTrace();
            }
        }
    });
}
```

> 算是搞明白了为什么传输对象的时候都使用ByteBuf了，自定义Data想要直接传输，直接报错：
>
> `java.lang.UnsupportedOperationException: unsupported message type: Data (expected: ByteBuf, FileRegion)`

并且这些Channel都是可重用的，因此可以实现多个线程共同使用一个Channel的方式。



Netty默认支持很多协议，在交互时候需要指定使用到的协议，简而言之，协议需要配套实现，这些协议更多的是一种种IO操作，而不是一些网络协议：

- NIO：基于Java NIO的Selector模式进行封装实现。

核心就是一个Selector，在Netty中可以把这个Selector看做一个Channel的注册表，当注册在该注册表中的Channel发生变化时候（创建Channel、Channel连接完成、Channel有待读入数据、Channel进行数据写）立马就会得到通知，然后基于Reactor模型来完成一些预定义的代码逻辑的执行。

> 执行NIO操作可以实现零拷贝，利用直接内存的方式来实现对数据的直接操作，而不用等待将数据从内核空间复制到用户空间。

- Epoll：有且仅能在Linux下使用，比NIO传输更快，是完全非阻塞的。

Linux下的非阻塞网络编程的事实标准，Java的NIO在Linux上的实现就采用了这种方式，Netty是直接完成对Epoll更一致的封装。如果使用的是Linux这个平台可以优先考虑这个，较之于Java的NIO有更加精致的封装。高负载下的性能也会更好。

- OIO：基于阻塞流的IO操作

也是有用武之地的，如果有老旧的遗留系统我们需要对其进行使用IO操作来完成交互，那么OIO也就是BIO会是一个好的选择，因为到遗留系统里面去也是会走阻塞的。

事实上OIO的Netty实现不是直接基于Java1.0提出的BIO的，因为Netty是一个异步框架，需要实现对API一致性的整合，实现方式还是通过指定IO操作完成的最大毫秒数，如果在这个制定的毫秒数之内没有完成，那么就会抛出一个异常，然后EventLoop完成对异常的捕获并继续进行处理和等待操作，如此方式实现，在外部看来也是一直阻塞在这几个任务上了。

> 几乎是异步框架实现OIO的唯一方式了

- Local：在VM内部通过管道进行通信的本地传输

JVM内部的服务器与客户端之间的通信，难怪SOFA RPC支持这种通信机制，原来是底层偷的Netty的，会自动完成对当前JVM服务端的识别并且调用，不需要绑定IP地址端口

- Embedded：一般用于测试你的ChannelHandler实现

核心是Channel的一个实现——EmbeddedChannel，主要是可以完成将一组ChannelHandler作为包装类直接嵌入到其他的ChannelHandler内部，但不需要修改内部代码的特性，不是通过代码层面的AOP来实现的，而是通过传输层级的调用实现的。

使用用例：

- 非阻塞代码库：如果代码库中不涉及到阻塞调用（或者阻塞调用可以控制在一个可以接受的范围），那么无论是大量并发连接或者是少量连接都是非常好的方式。
- 阻塞代码库：如果代码库严重依赖阻塞IO，而且已经有了稳定的设计，不要想着直接使用NIO，而是应该使用BIO，然后完成项目代码一个个模块的NIO迁移，最后再切换Netty到NIO的模式
- JVM内部通信：不想暴露外部端口号，避免因为网络波动对应用压测产生影响，可以使用这种模式，后期再转换为NIO或者是BIO模式
- Embedded通信：测试ChannelHandler实现



小结

完成了Netty的传输、实现和使用。并针对不同的传输给出了不同的最佳实践。







## 第五章、ByteBuf



网络传输的基本单位是字节，Java NIO使用ByteBuffer作为自己的字节容器，但是由于JDK需要提供全方位功能支持，难免使用起来会比较繁琐和复杂，Netty就另起炉灶，效仿ByteBuffer的处理逻辑推出了自己的字节容器——ByteBuf。

> 千万不要抱有捧一踩一的观点，只是在易用性上ByteBuf或许确实较之于ByteBuffer更加的好，后文也会介绍非常多ByteBuf较之于ByteBuffer的优点，但很有可能这是作者刻意为之，为了保证读者对ByteBuf的长久兴趣，永远要带着辩证的观点去思考和学习。



ByteBuf优点：

- 可以指定是在堆内存分配还是在直接内存分配
- 实现零拷贝（仅限于直接内存）
- 容器可以按需增长（性能不知如何）
- 在读写期间不需要flip方法（之所以ByteBuffer需要flip是因为他内部仅仅持有一个指针，而ByteBuf内部是持有读写指针的）
- 支持ByteBuf池化资源的管理



可以将ByteBuf的读写操作理解成一个字节数组存储数据，读写指针分别完成读操作和写操作

如果读指针跑到了写指针的前面，会触发IndexOutOfBoundsException异常，因为ByteBuf支持动态扩展（当然也可以限定长度），理论上写指针不会触发边界，但是由于内部使用Integer记录数组长度，因此写指针的上限Index为Integer.MAX_VALUE。

read和write操作会推进相应的指针，但get和set开头的方法则不会完成推进。



具体说下ByteBuf的数据结构和使用特点吧：

1、分配在堆缓冲区的ByteBuf

支持快速的进行分配和释放（毕竟较之于直接内存，JVM对堆内存的管控力更强），其实就相当于是将数据保存在一个堆内存的字节数组当中，然后通过ByteBuf的API完成对字节数组的操控。

完成对堆缓冲区上的ByteBuf的基本操作：

```java
ByteBuf buf = Unpooled.buffer();
// 一个字节表示范围：-128 ~ 127
buf.writeByte(128);

// 是基于堆内存的ByteBuf
if (buf.hasArray()) {
    byte[] array = buf.array();
    int offset = buf.arrayOffset();
    int length = buf.readableBytes();
    // 特殊处理，CopyOfRange是[)区间
    byte[] copy = Arrays.copyOfRange(buf.array(), offset, offset + length);
}
```

为什么要特殊处理呢？结合ByteBuf的数据结构：

0| ---	可废弃字节	--- |readIndex| ---	可读字节	--- |writeIndex| ---	可写字节	--- |capacity

capacity默认是Integer.MAX_VALUE

可以手动调用discardReadBytes方法完成对废弃空间的回收，这时候废弃空间就到了可写空间里面去了，非常类似于一个循环队列。

无论是对基于堆内存的还是基于直接内存的都是这样，只是基于直接内存的ByteBuf我们无法直接获取数组，还是可以通过readByte方法获取一个个字节的。



2、分配在直接缓冲区的ByteBuf

好处是在网络传输过程中可以直接实现发送功能，如果是在堆上的ByteBuf则需要完成到直接内存的复制才能完成数据发送

缺点也非常明显：如果想要操作在直接内存上的ByteBuf则需要完成数据到堆内存的复制过程：

```java
ByteBuf buf = Unpooled.directBuffer();
if (!buf.hasArray()) {
    int length = buf.readableBytes();
    byte[] array = new byte[length];
    buf.getBytes(length, array);
}
```

非常明显，需要自己new一片Byte数组。



3、复合缓冲区

ByteBuf的一个子类：CompositeByteBuf

算是自己的一个创新点：完成对多个ByteBuf的逻辑抽象，在外看类似于实现了对ByteBuf的合并

> 如果其中都是hasArray最后会返回true，如果其中有一个false就返回false了

```java
public static ByteBuf merge(ByteBuf... bufs) {
    CompositeByteBuf res = Unpooled.compositeBuffer();
    for (ByteBuf buf : bufs) {
        res.addComponent(buf);
    }
    res.removeComponent(0);
    return res;
}
```

因为可能不支持堆内存方式，因此还是老老实实复制一遍到堆上吧

```java
CompositeByteBuf buf = Unpooled.compositeBuffer();
int length = buf.readableBytes();
byte[] array = new byte[length];
buf.getBytes(length, array);
```

以上三种就是ByteBuf基本数据存储方式了。



存储完成后，自然该学习的是基于ByteBuf暴露的API完成字节级操作了，直接粘贴一些Sample吧

- 随机访问索引

```java
ByteBuf buf = Unpooled.buffer();
for (int i = 0; i < buf.capacity(); i++) {
    System.out.println(buf.getByte(i));
}
```

> 对基于堆内存的ByteBuf指的是readIndex还没有移动时候的顺序访问

- 可丢弃字节

```java
ByteBuf buf = Unpooled.buffer();
// 普通ByteBuf
ByteBuf byteBuf = buf.discardReadBytes();
// 组合ByteBuf
ByteBuf byteBuf1 = buf.discardSomeReadBytes();
```

> test了一下，都是直接进行的就地修改，不需要使用返回值了

- 获取所有字节

自己写的通用方法

```java
ByteBuf buf = Unpooled.directBuffer();
buf.writeBytes("hello world".getBytes(CharsetUtil.UTF_8));

while (buf.isReadable()) {
    System.out.println(((char) buf.readByte()));
}
```

- 所有数据写入

```java
ByteBuf buf = Unpooled.buffer(5);
while (buf.writableBytes() >0) {
    buf.writeByte(100);
}
```

- 索引管理

直接操作reader和writer索引

```java
buf.writerIndex(4);
buf.readerIndex(1);
```

还有一组mark和reset操作：markReaderIndex、markWriterIndex、resetReaderIndex、resetWriterIndex来完成标记和复位操作。

clear方法会将write和read都置为零，但是不会清除掉数据，仍可以以`getByte(index)`来访问到

```java
ByteBuf buf = Unpooled.buffer();
buf.writeBytes("hello world".getBytes(CharsetUtil.UTF_8));
buf.clear();
for (int i = 0; i < buf.capacity(); i++) {
    System.out.println(buf.getByte(i));
}
```

无论怎么操作都必须保证reader<writer<capacity，否则会报异常IndexOutOfBoundsException

clear方法较之于discardReadBytes会轻量得多，只涉及到指针的移动，discardReadBytes会涉及到内存复制问题。

- 查找方法

```java
ByteBuf buf = Unpooled.buffer();
// 普遍查找方法
buf.writeBytes("hello world".getBytes(CharsetUtil.UTF_8));
System.out.println(buf.indexOf(0, buf.capacity(), ((byte) 'o')));

// 常见值查找方法
int index = buf.forEachByte(ByteProcessor.FIND_ASCII_SPACE);
```

- 派生缓冲区

相当于是为ByteBuf提供一个单独的视图，非常类似于copy了一个新的ByteBuf，这个ByteBuf具有自己的读索引，写索引以及标记索引，但是与原来派生的ByteBuf共享内部存储，因此修改时候是会一起修改的。

派生操作（记录我认为重要的两个API）：`duplicate()`，`slice()`，`slice(int, int)`

复制操作：`copy()`，`copy(int, int)`

- 读写操作

分两大块：get/set（保持索引位置不变）、read/write（推进索引）



- 其他操作

isReadable、isWritable、readableBytes、writableBytes、capacity、maxCapacity都可以见名知意了





ByteBufHolder接口

ByteBufHolder为ByteBuf提供了多种高级特性如缓冲区池化等

提供的几个方法：

`content()`：返回ByteBufHolder底层所持有的ByteBuf

`copy()`：返回该ByteBufHolder的一个深拷贝

`duplicate`：返回ByteBufHolder浅拷贝，内部ByteBuf共享



ByteBuf分配（创建）的几种方式：

- 按需分配：ByteBufAllocator接口

Netty通过这个接口可以实现ByteBuf的池化，提供的API：

![image-20210530165251937](https://gitee.com/LuckyCurve/img/raw/master//img/image-20210530165251937.png)

使用起来ByteBufAllocator是一个接口，可以直接调用Channel或者是ChannelHandlerContext的alloc方法来完成对ByteBufAllocator的获取。

- Unpooled缓冲区

也是最常见的方式了，因为并不是所有时候你都可以获取到ByteBufAllocator的引用

![](https://gitee.com/LuckyCurve/img/raw/master//img/image-20210530170231829.png)

都是静态方法，非常适用于并不需要Netty的其他组件的非网络项目。



Netty的ByteBuf和ByteBufHolder实现了ReferenceCounted接口从而具有了引用计数的特征，如果引用计数为0那么就会被释放，如果试图去访问一个已经被释放了的对象。会报异常：IllegalReferenceCountException。



小结：

这一章完成了对ByteBuf的创建，使用，聚合视图，数据访问，读写操作，池化资源的使用等操作。

下一章将专注ChannelHandler，因为ByteBuf仅仅提供一个数据载体，而在ChannelHandler中定义了数据处理规则，ChannelHandler会将前面的各个组件串起来。





## 第六章、ChannelHandler和ChannelPipeline



ByteBuf是数据装载容器，这一章开始了解数据容器的处理流程以及处理组件，主要是在ChannelPipeline当中完成对ChannelHandler链接的组装。

因为数据是基于Channel来进行传输的，在学习ChannelHandler和ChannelPipeline之前有必要了解基本的生命周期：

- ChannelUnregistered：被创建，但未注册到EventLoop当中去
- ChannelRegistered：已经被注册进了EventLoop
- ChannelActive：已经连接到远程节点，可以收发数据
- ChannelInactive：没有连接到远程节点

当发生状态转变时候会发送相应的事件到ChannelPipeline中的ChannelHandler



ChannelHandler的生命周期发生变化时，主要指的是当ChannelHandler被添加到ChannelPipeline或者是从ChannelPipeline当中移除时候会调用这些操作，这些操作会接受一个入参ChannelHandlerContext，通过ChannelHandlerContext可以完成对ChannelHandler上下文信息的获取，或者是获取对应的Channel来完成对应的操作：

- handlerAdded：当把ChannelHandler添加到ChannelPipeline中被调用
- handlerRemoved：当把ChannelHandler从ChannelPipeline移除时候调用
- exceptionCaught：在处理过程中ChannelPipeline中产生错误时候调用

主要两个重要的ChannelHandler子接口：

- ChannelInboundHandler：处理入站数据以及各种状态变化
- ChannelOutboundHandler：处理出站数据并且允许拦截所有的操作



ChannelHandler就是观察者模式的一种实现，通过完成对对应方法的回调来处理事件，入参均为ChannelHandlerContext，ChannelHandler提供的监听方法有：

- handlerAdded：当ChannelHandler被加入到ChannelPipeline后完成回调
- handlerRemoved：当ChannelHandler从ChannelPipeline当中移除时候调用
- 已废弃，建议在ChannelInboundHandler当中使用 exceptionCaught：当CHannelPipeline发生异常时候会将此异常和ChannelHandlerContext以入参的形式传入到该方法当中来

ChannelInboundHandler在接收数据时候或者与其对应的Channel状态发生变化时候调用，只有这两种情况下会完成对ChannelInboundHandler的调用，和Channel的生命周期密切相关，提供的特有方法有：

- channelRegistered
- channelUnRegistered
- channelActive
- channelInactive
- channelRead
- channelReadComplete
- userEventTriggered：当完成对`ChannelInboundInvoker fireUserEventTriggered(Object event);`调用时候会将当前event和ChannelHandlerContext传入到这个方法当参数传入
- channelWritabilityChanged：当Channel的可写状态发生改变时被调用，可以调用channel.isWritable方法来判断Channel的可写性
- exceptionCaught：发生异常时候捕获，向后面的ChannelInboundHandler传递

> ~~只有在发生变化的时候才会激活，比如最开始是UnRegistered，只有在最后资源释放的时候重新变回了UnRegistered才会触发方法，否则不会~~
>
> 状态流转：Register——Active——Inactive（离开活动状态，即断开连接）——unRegister（Channel取消注册到EventLoop上了）



之所以`SimpleChannelInboundHandler`可以自动自动释放，是因为

```java
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
    boolean release = true;
    try {
        if (acceptInboundMessage(msg)) {
            @SuppressWarnings("unchecked")
            I imsg = (I) msg;
            channelRead0(ctx, imsg);
        } else {
            release = false;
            ctx.fireChannelRead(msg);
        }
    } finally {
        if (autoRelease && release) {
            ReferenceCountUtil.release(msg);
        }
    }
}
```

只是自动帮我们加上了ReferenceCountUtil的release方法调用





ChannelOutboundHandler接口

出站操作和数据将被该实例处理，强大的功能是按需推迟操作或者是事件，可以通过一些复杂的方法来处理请求，如：远程节点写入暂停了，可以推迟flush操作并稍后继续。

核心方法：

- bind：当Channel绑定到远程地址时候调用
- connect：当Channel连接到远程地址时候调用
- disconnect：当Channel断开远程连接
- close：当关闭Channel时候调用
- deregister：从EventLoop注销时候调用
- read：从Channel读取数据时候调用
- flush：当Channel调用了flush时候调用
- write：向Channel当中写数据时候调用

绝大多数的ChannelOutboundHandler都需要一个ChannelPromise作为入参，ChannelFuture是ChannelFuture的子类，提供了setSuccess和setFailure方法完成回调函数的设置，相当于扩展提供了Future的部分写方法。



ChannelHandlerAdapter存在方法isSharable，如果实现类当中被注解@Sharable标注了，那么该方法就返回true，表示该Handler可以同时被注册进多个ChannelPipeline当中



看到一个非常好的总结：**ChannelInboundHandler和ChannelOutboundHandler的区别主要是对ChannelPipeline而言的，如果事件是传播出ChannelPipeline，那么则是outBound的，如果是传入ChannelPipeline，那么是inBound的**



ChannelOutboundHandler并没有类似于SimpleChannelInboundHandler，当消费完成之后需要手动完成资源释放：`ReferenceCountUtil.release(obj);`，此外还必须通知ChannelPromise，否则ChannelFuture上注册的ChannelFutureListener会失效。

如果一个消息被消费了或者是被丢弃了，没有被传入到下一个ChannelOutboundHandler，那么用户就需要回收掉它（为什么不是ChannelHandler？）。如果消息到达了实际的传输层，那么在写入或者是Channel关闭时候，资源会自动释放（Channel关闭不会释放不在传输层的资源）。

Channel绑定唯一的ChannelPipeline，并且不能修改，不能分离，通过ChannelHandlerContext实现，会将消息转发到同一超类型的下一个ChannelHandler

通过ChannelHandlerContext可以实现和ChannelPipeline以及其他的ChannelHandler交互，实现通知下一个ChannelHandler或者动态修改所属的ChannelPipeline。

![image-20211126155041990](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211126155041990.png)

**一个ChannelPipeline可以包含多种类型的ChannelHandler，具体的Handler的执行顺序直接上ChannelPipeline注：**

```
例如，假设我们创建了以下管道：
   ChannelPipeline p = ...;
   p.addLast("1", new InboundHandlerA());
   p.addLast("2", new InboundHandlerB());
   p.addLast("3", new OutboundHandlerA());
   p.addLast("4", new OutboundHandlerB());
   p.addLast("5", new InboundOutboundHandlerX());

在上面的示例中，名称以Inbound开头的类表示它是一个入站处理程序。 名称以Outbound开头的类表示它是一个出站处理程序。
在给定的示例配置中，当事件进入时，处理程序评估顺序为 1、2、3、4、5。 当事件出站时，顺序为 5, 4, 3, 2, 1。 在此原则之上， ChannelPipeline跳过某些处理程序的评估以缩短堆栈深度（根据事件类型来完成跳过）：
3 和 4 没有实现ChannelInboundHandler ，因此入站事件的实际评估顺序将是：1、2 和 5。
1 和 2 没有实现ChannelOutboundHandler ，因此出站事件的实际评估顺序将是： ChannelOutboundHandler和 3。
如果 5 同时实现ChannelInboundHandler和ChannelOutboundHandler ，则入站和出站事件的评估顺序可能分别为 125 和 543。
```

ChannelHandler可以在运行过程当中动态更改从属的Pipeline，主要方式为通过ChannelHandlerContext来获取到Pipeline来调用Pipeline的addFirst/Last、addBefore/After，甚至将自身从ChannelPipeline当中移除，实现灵活的逻辑



```
建议可以使用带名称的Handler，因为在后续对ChannelPipeline的remove或者replace的时候可以指定唯一名字来完成，Netty会保证名字唯一
```



ChannelHandler的执行：因为ChannelHandler是从属于一个特定的Channel的，一个Channel有且仅能从属一个EventLoop，一个EventLoop绑定唯一的Thread，所以当前Channel当中的所有ChannelHandler的IO操作，甚至别的共用这一个EventLoop的Channel当中的ChannelHandler的IO操作，都会直接使用到这个IO线程，至关重要的就是不要阻塞住这个线程，否则整体IO都会受影响。

以上是尽量避免在ChannelHandler当中使用阻塞IO，但是难免的有时候业务处理逻辑就需要和老的阻塞IO打交道，那么此时ChannelPipeline在完成ChannelHandler的组装的时候（如addLast等方法时候）这时候我们需要传入一个EventExecutorGroup参数，这时候当中的阻塞IO操作会被这个特定的EventExecutorGroup当中的某个EventExecutor所执行，而不会去阻塞整个EventLoop当中默认的DefaultEventExecutorGroup的执行。

几个Pipeline访问ChannelHandler的操作：

```
ChannelPipeline pipeline = ctx.pipeline();

List<String> list = pipeline.names();

pipeline.get(name / Class.ChannelHandler);

pipeline.context(name / ChannelHandler / ChannelHandler.class);
```



ChannelPipeline触发入站操作：

```java
@Override
ChannelPipeline fireChannelRegistered();

@Override
ChannelPipeline fireChannelUnregistered();

@Override
ChannelPipeline fireChannelActive();

@Override
ChannelPipeline fireChannelInactive();

@Override
ChannelPipeline fireExceptionCaught(Throwable cause);

@Override
ChannelPipeline fireUserEventTriggered(Object event);

@Override
ChannelPipeline fireChannelRead(Object msg);

@Override
ChannelPipeline fireChannelReadComplete();

@Override
ChannelPipeline fireChannelWritabilityChanged();
```

都是调用head的对应事件，而head调用完对应事件必然会接连下去调用后面的事件了，入站感觉好理解，着重看一下出站的：

![image-20211126220918630](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211126220918630.png)

也会调用对应的ChannelOutboundHandler当中对应的方法，以及完成后续的链式调用。



实际上这些方法都在ChannelInboundInvoker和ChannelOutboundInvoker当中定义了。



ChannelHandlerContext当中也实现了这两个接口，与Channel或者ChannelPipeline当中不同的是传播的起点是不同的（这两者本身是等价的），**前者是当前ChannelHandlerconContext所对应的下一个ChannelHandler开始传播**，后者是从head开始传播或者tail开始传播（根据入站事件和出站事件的不同而定）。

使用ChannelHandlerContext的两个要点：

- ChannelHandlerContext与ChannelHandler之间的绑定永远不会改变
- ChannelHandlerContext会产生更短的事件流，依据这个特性可以获得更大的性能



因为ChannelHandlerContext负责的是当前ChannelHandler与下一个ChannelHandler之间的交互，因此在ChannelHandlerContext当中发起事件调用的时候只能将该事件发送给下一个ChannelHandler，从代码层面上也可以看出来：

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



非常重要的一点：ChannelHandler是可以实现复用的，但ChannelHandlerContext相当于是ChannelHandler在这个Channel当中的上下文信息，因此是不可能复用的，从源代码和注解当中可以窥见一二。

```java
/**
     * Gets called after the {@link ChannelHandler} was added to the actual context and it's ready to handle events.
     */
void handlerAdded(ChannelHandlerContext ctx) throws Exception;
```

如果一个ChannelHandler可以添加到多个ChannelPipeline当中，需要在该ChannelHandler头上加上注解@Sharable标注，非常明显的错误标注为：

![image-20211127123416076](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211127123416076.png)

一般标注了@Sharable的需要是无状态的或者是线程安全的才可以

> 一种非常常见的ChannelHandler的共享使用为：收集跨越多个Channel的统计信息



高级用法：

- 可以通过动态修改ChannelPipeline来实现动态的协议切换，如上一个ChannelHandler完成的是对消息的协议判断，根据判断出来的协议动态切换下一个ChannelHandler的协议解析模式
- 缓存ChannelHandlerContext的引用，然后通过该引用来发送消息



Netty当中的异常处理：

可以分为两部分：处理入站异常和处理出站异常

如果ChannelHandler抛出了异常，那么首先会被当前ChannelHandler当中的exceptionCaught捕获到，所以我们一般都会在方法当中完成两件事情：异常堆栈打印、ctx关闭，至于具体需要执行什么其他的业务逻辑，就自己添加了。

往往都是在最后一个ChannelHandler当中完成异常处理，因为异常信息也会沿着入站方向流动，这样可以确保所有的异常信息都会被处理掉，无论该异常是发生在ChannelPipeline的什么位置，默认实现是转发给下一个ChannelHandler，如果到达最后还是没人处理，那么netty会将该异常标注为未处理，如下：

```
11月 27, 2021 12:53:02 下午 io.netty.channel.DefaultChannelPipeline onUnhandledInboundException
警告: An exceptionCaught() event was fired, and it reached at the tail of the pipeline. It usually means the last handler in the pipeline did not handle the exception.
```





出站异常：

无论是操作成功或者是出现异常，都是基于以下的通知机制：

- 根据出站操作返回的ChannelFuture，然后在ChannelFuture注册ChannelFutureListener来完成处理成功还是失败
- 成功或者失败是出站操作时候对传入的ChannelPromise参数决定的，主要两个方法：`setSuccess();`  `setFailure(Throwable cause)`

也是最常见的处理方式：

![image-20211127125843013](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211127125843013.png)

还有另一种写法是直接将回调函数在ChannelOutboundHandler当中就注册了

![image-20211127130017589](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211127130017589.png)

两种效果完全一致，只是编码风格不同，个人推荐使用第一种，更接近回调的思想





## 第七章、EventLoop和线程模型



Java当中提供了Thread和1.5引入的Thread优化技术Executor来利用多线程的优势，虽然池化技术可以很好的完成Thread重量级资源的重用，但是随着线程数的增加，上下文切换的次数也会增加的很明显，Netty显然也是认识到了这种线程模型所带来的问题，Netty的主旨是：**简化应用程序代码，同时最大限度的提高性能和可维护性**



EventLoop：事件循环，整个模型的核心，包含两大部分API：并发处理和网络通信。并且其上的事件和任务都是FIFO的，可以保证字节内容总是顺序消费。

其中只有一个方法parent，用来获取对应的EventLoopGroup

所有的IO操作和事件都被分配给了EventLoop对应的Thread来进行处理，因此一定不能有阻塞操作，这是最新的Netty4的模型。

Netty3原来只有入站事件会被EventLoop处理，出站事件都由调用线程处理，因此可能是EventLoop的线程也可能是别的线程，乍看上去是好的，但有种情况：无法保证多个线程不会再同一时刻尝试访问出站事件，也就是两个线程同时调用channel.write方法。



任务调度API：

JDK的：Timer和ScheduledExecutorService

![image-20211127163657727](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211127163657727.png)

好用的，但是存在一个性能瓶颈：如果大量任务紧密的调度，为了保证执行，会有大量的额外线程被创建

![image-20211127164445107](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211127164445107.png)

![image-20211127164506310](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211127164506310.png)

Netty的EventLoop实际上是扩展了ScheduledExecutorService，这几个方法都是直接@Override了ScheduledExecutorService的

取消定时调度操作：

![image-20211127164956375](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211127164956375.png)



前面看到的都是Netty的性能提升表现，真正的线程模型实现细节从现在开始。

EventLoop会将所有内部发生的事件（对应的Channel的事件）都在Thread内部解决掉

执行逻辑如下：

1、判断当前线程是否是EventLoop所持有的线程

```
Channel channel = ...;
channel.eventLoop().inEventLoop(thread);
```

2、如果是的，那么当前线程直接执行任务

3、如果不是的，把任务放入队列等EventLoop下一次来处理

因此，**永远不要将一个长时间运行的任务放入到执行队列中，因为它将阻塞需要在同一线程上执行的任何其他任务**，如果必须的话就使用一个单独的EventExecutor来完成

```java
static final EventExecutorGroup group = new DefaultEventExecutorGroup(16);

ChannelPipeline pipeline = ch.pipeline();

pipeline.addLast("decoder", new MyProtocolDecoder());

pipeline.addLast("encoder", new MyProtocolEncoder());

// Tell the pipeline to run MyBusinessLogicHandler's event handler methods
// in a different thread than an I/O thread so that the I/O thread is not blocked by
// a time-consuming task.
// If your business logic is fully asynchronous or finished very quickly, you don't
// need to specify a group.
pipeline.addLast(group, "handler", new MyBusinessLogicHandler());
```



根据传输方式的不同，可以分为异步传输和阻塞传输：

- 异步传输

通过尽可能少的Thread来支撑大量的Channel

![image-20211127171144385](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211127171144385.png)

这时候多个Channel就无法使用ThreadLocal来做状态追踪了，因为都是共用一个ThreadLocal的，但是使用ThreadLocal来完成在Channel之间传递对象还是很好用的

- 阻塞传输

![image-20211127171655310](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211127171655310.png)

最常见的传输模型了，一个任务对应一个Thread



Netty支持多种NIO实现，有：

- COMMON：NioEventLoopGroup
- Linux：EpollEventLoopGroup
- macOS：KQueueEventLoopGroup

其实COMMON就是直接使用的JDK提供的NIO，JDK提供的NIO在Linux环境下使用的也是Epoll技术，Netty在这里之所有要重新写一个是因为Netty作者有自信自己写出来的Epoll比JDK当中的要好，好的地方是以下两点：

- JDK的NIO只支持水平触发，Netty Epoll支持水平触发和边缘触发（默认）

> 水平触发，当文件描述符的读缓冲区非空，有数据可读的时候，就一直发送可读信号，当文件描述符的写缓冲区非满，可写的时候，一直发出可写信号。
>
> 边缘触发：当文件描述符的读缓冲区由空转为非空的时候，代表有数据可读，发送一次可读信号，当文件描述符的写缓冲区由满转为非满的时候，代表有数据可写，发送一次可写信号。





## 第八章、引导



引导一个应用程序是指对他进行配置，并使他运行起来的过程

类图如下：

![image-20211127230208241](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211127230208241.png)

之所以会有这个区分，是因为致力方向不同：

- 服务器致力于使用一个父Channel来接收来自客户端的连接，并创建子Channel以用于他们之间的通信
- 客户端往往只需要一个单独的，没有父Channel的Channel来用于所有的网络交互

之所以implements了Cloneable是因为需要完成配置引导类的拷贝，其中的EventLoopGroup是浅拷贝的。



AbstractBootstrap的签名为：`public abstract class AbstractBootstrap<B extends AbstractBootstrap<B, C>, C extends Channel> implements Cloneable `

之所以要传入BC两个泛型（B就是当前子类的Class，C是子类的Channel对象），是为了支持链式调用，将本体或者Channel对象返回出去，这样就需要指明具体的数据类型



先从结构简单的客户端API说起（Bootstrap往往只需要单独的Channel即可）：

- `public Bootstrap group(EventLoopGroup group);`

处理Channel事件的EventLoopGroup

- `public Bootstrap channel(Class<? extends C> channelClass);`

指定Channel实现类，会调用其中Channel的无参构造函数来创建Channel

- `public B localAddress(SocketAddress localAddress);`

指定Channel应该绑定到的本地地址，或者可以在bind和connect（就默认连本地了）方法当中去指定

- `public <T> Bootstrap option(ChannelOption<T> option, T value);`

设置ChannelOptional，在bind或者connect时候完成ChannelConfig属性设置，如果Channel已经被创建完成了再去调用，则不会有任何的效果

- `public <T> Bootstrap attr(AttributeKey<T> key, T value);`

:question:感觉就是上面option方法的翻版，也是在bind或者connect执行时候属性设置

- `public Bootstrap handler(ChannelHandler handler);`

设置ChannelHandler

- `public Bootstrap remoteAddress(SocketAddress remoteAddress);` 

指明远程地址，可以使用connect方法指明



在connect方法被调用后，Bootstrap类会创建一个新的Channel



不能混用Channel和EventLoopGroup，必须使用前缀搭配的，如NIO、OIO、Epoll等



在引导调用bind或者connect方法之前，必须完成以下设置：

- group
- channel或者channelFactory
- handler





服务器引导

操作非常多，比如从ServerChannel的子Channel中引导一个客户端这样的特殊情况

主要的API区别是增加了一些child开头的方法，主要有：option、attr、handler、

这里存在两个抽象：ServerChannel和子Channel，ServerChannel负责创建子Channel，而这些子Channel代表已经建立的连接，因此，ServerChannel的创建时期是在ServerBootstrap调用bind方法时候创建的，而子Channel创建时机是有客户端connect进来的时候创建的

![image-20211128222515631](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211128222515631.png)



Channel属性设置：可以在Bootstrap当中设置ChannelOption，这样在创建Channel的时候会自动应用，主要几种方式：

1、`bootstrap#option(k, v)`，这里的k都是ChannelOption当中预设的属性，因此可用度不是很高

2、`bootstrap#attr(k, v)`，这里的k是`AttributeKey<T>`，这里的v就是T，Channel当中取出值的话也是非常容易取出的，直接`channel#attr(k).get()`即可



前面大多数都是TCP协议的，其实Bootstrap也可以引导无连接的协议，唯一的区别就是调用bind去接收消息。



完成bootstrap的关闭：只需要调用`shutdownGracefully`方法即可，EventLoopGroup将处理任何挂起的事件和任务，并且随后释放所有的线程，但是是个异步操作，需要我们使用sync阻塞或者是向Future注册监听器以完成通知。





## 第九章、单元测试



单元测试的基本思想是：以尽可能小的区块测试你的代码，并且尽可能的和其他的代码模块以及运行时的依赖（如数据库和网络）相隔离

Netty当中的单元测试主体为ChannelHandler，Netty专门针对单元测试提供了EmbeddedChannel，使用Junit4来完成。

EmbeddedChannel可以认为是特殊的Channel实现，将入站事件或者是出站事件写入到EmbeddedChannel当中，然后检查Channel的尾部即可

核心API：

![image-20211130154015997](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211130154015997.png)

书中给出了很明确的说法，关于入站数据和出站数据：

- 入站数据由ChannelInboundHandler处理，代表从远程节点读取的数据
- 出站数据由ChannelOutboundHandler处理，代表将要写到远程节点的数据

![image-20211130154920283](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211130154920283.png)

可以就把EmbeddedChannel看做一个简易的客户端，使用起来感觉应该会非常棒



一些断言方法都在`org.junit.Assert`当中，并且是静态测试方法，因此建议使用`import static org.junit.Assert.*`来高效使用





# 第二部分：编解码器



在数据和网络字节流之间相互转换是最常见的编程任务之一



## 第十章、编解码器框架



开发基础组件非常有用的工具，如你正在编写邮件服务器，那么会发现Netty对一些SMTP协议的编解码支持有多么的好用了。

编码器是完成消息到适合传输的格式（最有可能是字节流）的转换，解码器是字节流到消息的转换

解码器，主要有两个：

- 字节转换为消息：抽象类ByteToMessageDecoder和抽象类ReplayingDecoder
- 将一种消息类型转换为另一种：MessageToMessageDecoder

都是直接extends了ChannelInboundHandlerAdaptor，常规

甚至可以将多个解码器连接在一起，这也是Netty的模块化和复用的一个很好的例子



ByteToMessageDecoder：由于不知道是否一次性发送完，所以ByteToMessageDecoder会对入站数据进行缓冲

![image-20211201231228276](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211201231228276.png)

decode方法只有List不为空的时候才会去走下面的

> 消息被编解码完成之后会立马被release掉，所以如果我们需要留下，需要调用ReferenceCountUtil.retain方法来增加计数信息防止消息被释放掉

可以看到ReplayingDecoder有extends我们的ByteToMessageDecoder

![image-20211201231954020](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211201231954020.png)

原理非常暴力，比如我们读一个int，没有数据会抛出Exception，然后DeplayingDecoder会生吞这个异常，并且不是所有操作都支持，然后下次有数据再调用，因此速率上会低于ByteToMessageDecoder。

使用基准：如果使用ByteToMessageDecoder不会引入太多的复杂度，可以使用他，否则是哦那个DeplayingDecoder





MessageToMessageDecoder：完成消息到消息间的转换

![image-20211201232556427](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211201232556427.png)

指定的参数T即为输入参数msg的泛型



一个异常：TooLongFrameException：因为Netty是异步的，所以需要使用缓冲区来缓存大量的入站消息，为了防止缓冲大量数据导致内存耗尽，在缓冲区扩张到指定大小时候会抛出这个异常，这个异常会顺着Channel往下走，被下游的exceptionCaught捕获到。需要我们在解码器当中手动抛出：

![image-20211201233221433](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211201233221433.png)





编码器，主要也就两种：

- MessageToByteEncoder
- MessageToMessageEncoder

都是直接extends了ChannelOutboundHnadlerAdaptor



MessageToByteEncoder核心API

![image-20211202155218654](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211202155218654.png)

对比ByteToMessageDecoder做逆向的事儿，



MessageToMessageEncoder

感觉之所以有这个，是对上面的补充，因为字节流只是一种网络传输的方式（或者字符串编码），还会有其他的方式，也就是该方法所做的补充了



抽象编解码器：在同一个类当中完成对入站和出站数据的编解码操作，这些类同时实现了ChannelInboundHandler和ChannelOutboundHandler接口。其实Netty没有那么优先于这种编解码器，因为将编码和解码功能分开，可以最大化代码的可重用性和可扩展性



1、ByteToMessageCoderc

他是extends了ChannelInboundHandlerAdaptor然后再implements了ChannelOutboundHandler，可见入站操作普遍会比出站操作复杂

![image-20211202165602831](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211202165602831.png)

完完全全的简单组合。



2、MessageToMessageCodec

![image-20211202170223665](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211202170223665.png)

decode操作是将in转成out，encode操作是将out转换成in

因此ByteToMessageCoderc可以理解成MessageToMessageCodec<ByteBuf, Object>，将第一个参数IN看做是网络上的数据即可





上面提到的Codec会降低代码的重用性，所以Netty也是通过组合的方式来解决了：

`public class CombinedChannelDuplexHandler<I extends ChannelInboundHandler, O extends ChannelOutboundHandler>`

可以实现各自的编解码器，然后再组装到这个Handler当中来即可。

提供给第三方使用会比较好，感觉自己使用感觉还是挺麻烦的，个人偏好问题吧。







## 第十一章、预置的ChannelHandler和编解码器



Netty内置的通用协议编解码器和处理器，和一些性能调优点



使用SSL/TLS安全协议来保证Netty应用程序的数据安全，不一定绑定HTTP协议使用，它们叠加在其他协议之上，如SMTP（SMTPS）甚至是关系型数据库系统。

Netty提供了SslHandler：`public class SslHandler extends ByteToMessageDecoder implements ChannelOutboundHandler`来完成的

> 果然如前面预想的，都是extends Decoder，说明Decoder更加复杂

其实是依赖于Javax的SSLEngine来实现的，使用起来也非常简单。但是Netty也提供了自己的实现，认为性能更好吧，OpenSslEngine，默认使用的是Netty自身的，如果没有提供就退化到JDK的SSLEngine【在Netty中是jdkSslEngine】当中去。

记得将SslHandler添加到最头边去就好。

核心API：

![image-20211203153701418](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211203153701418.png)





使用HTTP/HTTPS协议

Netty对HTTP请求和响应的数据模型：

![image-20211203154115555](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211203154115555.png)

![image-20211203154123254](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211203154123254.png)

图中所有的数据抽象都实现了HttpObject接口。

![image-20211203154242076](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211203154242076.png)

HttpClientCodec和HttpServerCodec，聚合的类



因为Http报文可能被划分成很多段，所以我们需要聚合器来将其聚合成Full的，就和我们之前写的：ByteToIntegerDecoder一样罢了，特征值够了就转换即可。

Main Class：`HttpObjectAggregator(512 * 1024)`，接收最大的碎片大小为512K的Http段



建议开启HTTP压缩来减小传输数据的大小，

![image-20211204194327037](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211204194327037.png)



使用HTTPS：

![image-20211204194411223](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211204194411223.png)



WebSocket：在一个TCP连接上提供双向的通信，因为HTTP始终是应答机制，算是对代替HTTP轮询的方案。

感觉和HTTP2.0有点类似，但是都是应用层的协议，应该不会淘汰HTTP协议的。

![image-20211204195342122](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211204195342122.png)

更全面的内容会留在12章



网络传输过程当中对空闲以及超时的连接的处理：

![image-20211204195626285](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211204195626285.png)

**只有添加了前面的Handler才会抛出后面对应的异常**





Netty协议解码，通常有两种：

- 基于分隔符的协议
- 基于长度的协议



![image-20211204202941504](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211204202941504.png)

Example：`cn.luckycurve.character11.LineBasedChannelInitializer`



![image-20211204211716802](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211204211716802.png)

定长容易，变长模型：

![image-20211204211814724](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211204211814724.png)

Example：`cn.luckycurve.character11.LengthBasedInitializer`





写大型数据，最主要的一点就是客户端网速无法估计，因此如果传输一个较大的文件到慢的客户端，会有很大的内存占用

![image-20211204214127538](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211204214127538.png)

如果仅仅直接将文件进行传输，使用NIO零拷贝技术可以轻易完成。但如果我们需要将文件加载到内存当中进行处理，建议使用ChunkedWriteHandler，很好的支持了异步写大型数据流，而不会导致大量的内存消耗。

传输的数据需要使用ChunkedInput实现类来进行封装，常见的ChunkedInput实现有：

![image-20211204215828924](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211204215828924.png)

零拷贝代码：cn.luckycurve.character11.ZeroCopyChannelHandler

减缓内存压力代码：cn.luckycurve.character11.MemoryChannelInitializer



序列化机制：

JDK序列化

![image-20211205162745467](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211205162745467.png)

前两个在Netty4当中已经被废弃了，很有可能是JDK根据Netty完成了重写，直接使用后两个就行



JBoss Marshalling

![image-20211205163007124](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211205163007124.png)

也对JDK序列化进行了兼容



Protocol Buffers

也就是大名鼎鼎的谷歌的Protobuf了，高效而紧凑的方式对结构化数据进行编解码，并且提供了很多语言的支持，很适合跨语言

![image-20211205163430221](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211205163430221.png)

![image-20211205163528945](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211205163528945.png)

后面两者来避免粘包半包问题的







## 第十二章、WebSocket



实现实时Web的功能：实时Web利用技术和实践，使用户在信息的作者发布信息之后就能够立即收到信息，而不需要他们或者其他的软件周期性的检查信息源以获取更新（JQuery）

WebSocket协议在这方面迈出了坚实的一步

WebSocket协议是完全重新设计的协议，旨在为Web上的双向数据传输问题提供一个坚实可行的解决方案，使得客户端和服务器之间可以在任一时刻传输消息。

因此要求异步处理消息回执（如果同步处理，无法保证任一时刻都在传输消息）

用Netty使用WebSocket实现以下实时聊天功能：

![image-20211206225154099](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211206225154099.png)



WebSocket协议需要从标准的HTTP进行升级握手，因此WebSocket的应用程序都是以HTTP开头

因此可以指定如下规则：如果请求以/ws结尾就升级，否则不升级

![image-20211206225528005](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211206225528005.png)





Netty自带注解：

1、@Sharable：标注handler可共享，如果不标注，那么当一个Handler被共享的时候就会报错

2、@Skip：跳过，可标注在handler当中的METHOD上，不能用，只支持内部使用

3、







# Netty当中使用到的设计模式



|  设计模式  |                            出现类                            |                           核心代码                           |
| :--------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|  单例模式  | ReadTimeoutException【在Idle检测的时候避免异常被频繁的创建】 | `public static final ReadTimeoutException INSTANCE = PlatformDependent.javaVersion() >= 7 ? new ReadTimeoutException(true) : new ReadTimeoutException();` |
|  工厂模式  | ReflectiveChannelFactory【反射加工厂的方式来完成Channel和Childchannel的创建】 | `constructor = clazz.getConstructor();` `return constructor.newInstance();` |
|  策略模式  | EventExecutorChooser【从NioEventLoopGroup当中选择一个EventLoop，两个具体策略：GenericEventExecutorChooser、PowerOfTwoEventExecutorChooser】 |                   `EventExecutor next();`                    |
|  装饰模式  | DuplicateByteBuf【创建一个与原ByteBuf共享源缓冲区的ByteBuf，只存在自己的read和write指针】 |                                                              |
| 责任链模式 |                       ChannelPipeline                        |                                                              |
| 建造者模式 |         SslContextBuilder【完成对SslContext的构建】          |                                                              |
| 观察者模式 |                  channelFuture#addListener                   |                                                              |
|            |                                                              |                                                              |
|            |                                                              |                                                              |

