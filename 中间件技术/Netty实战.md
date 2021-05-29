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
    // 特殊处理，CopyOfRange是[]区间
    byte[] copy = Arrays.copyOfRange(buf.array(), offset, offset + length);
}
```

为什么要特殊处理呢？结合ByteBuf的数据结构：

0| ---	可废弃字节	--- |readIndex| ---	可读字节	--- |writeIndex| ---	可写字节	--- |capacity

capacity默认是Integer.MAX_VALUE

可以手动调用discardReadBytes方法完成对废弃空间的回收，这时候废弃空间就到了可写空间里面去了，非常类似于一个循环队列。



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