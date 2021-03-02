# Netty知识点



以前仅仅只是听说过Netty，还没有系统的学习过，借助这个机会领略下Netty的风采



Netty的兴起主要是随着网络的普及，Java的网络编程愈发的重要，面对高并发的场景，Java1.4提供的NIO是非常难用的，Netty相当于是对NIO进行了一次封装，简化了用法

目前使用到了Netty的项目非常多，如ElasticSearch、Spring Cloud Gateway、Dubbo



BIO、NIO和AIO的区别

BIO：传统的阻塞IO，在1.0的时候就已经存在了，是比较传统的IO方式，单个线程在进行IO操作的时候会直接阻塞，属于同步阻塞IO，在QPS不是很大的时候，每个线程各司其职，井井有条，是非常好的IO模型，但是随着请求数量的增加，线程重量级的特性体现了出来，无法应付，于是1.4提出了NIO

NIO：同步非阻塞IO，线程通过和Selector打交道，从而可以同时操作多个Channel，每个Channel分别管理着一份buffer，NIO是面向Buffer的，可以实现更高的并发量，但是在并发量较小的时候还是推荐直接使用BIO，易维护，可以通过池化资源来减小线程开销

AIO：1.7提出的异步非阻塞IO模型，是基于事件和回掉机制来完成的，NIO在调用线程发出指令后，可以进行其他线程，但是到最后还是需要进行同步操作，等待IO的执行完成，但是AIO是不需要的，可以直接返回，AIO提供的回掉机制可以帮忙完成同步操作，但是使用起来非常困难。Netty之前尝试过使用，但是放弃了。



Netty是基于NIO之上的基于Client-Server的网络编程模型，优化了TCP，UDP套接字，并且还支持多种应用层协议如HTTP，FTP等



Netty和NIO的一个比较（Netty的优势）

- 成熟稳定，很多大型项目如ElasticSearch、Dubbo都使用到了Netty
- 在保证安全性和性能的前提下做到了易开发和维护
- 统一的API，直接支持阻塞和非阻塞的操作
- 安全性有保障，自带SSL/TLS支持



具体使用场景：Netty真是一个造轮子的好帮手，可以说是一个造框架的框架了，如实现RPC远程调用，自定义HTTP服务器，通信系统，消息推送系统



Netty的核心组件：

ByteBuf：字节容器，Netty操作数据的基本单位，相当于是NIO当中的ByteBuffer

BootStrap和ServerBootStrap：客户端和服务端的启动引导类

Channel：作为数据传输的一个通道，类似于Java BIO当中的Stream的概念

EventLoop：事件循环，说白了NIO是基于事件驱动的，那么Netty也是，绝对的核心，负责监听网络事件并调用相关操作



下面就是一些编程模型的知识点了

Reactor编程模型（反映式编程模型）：局域事件驱动，特别适合处理海量请求

Reactor可以细分为单线程Reactor，多线程Reactor以及主从多线程模型

单线程主要是所有请求都交给一个线程去调用Selector，来完成对应事件匹配到事件处理器上，优点是消耗特别少，缺点也非常明显，因为只有一个线程，可能无法处理表现出高延迟

多线程Reactor就是解决单线程Reactor在并发量大的时候的高延迟问题，引入了线程池，每个线程都持有一个Selector完成事件的匹配，由一个父线程来完成对事件的监听，但是监听到事件后不是直接进行匹配，而是交给线程池中的子线程去完成匹配工作，这样就提高了吞吐量，降低了延迟（主要耗费的时间还是在匹配Handler上），但是如果是上百万的请求，这个模型还是顶不住，因为始终只有一个线程进行事件的监听，哪怕调用子线程池去处理费时操作，延迟依旧会上来

此时需要使用主从多线程Reactor，一组线程池中的线程负责监听事件并将事件发送给另一组线程池，另一组线程池中的线程进行事件与事件处理器Handler的匹配



Netty的线程模型

Netty也是基于Reactor线程模型开发的，支持以上三种模型，使用分别为：

```java
// 如果是无参的化默认大小是CPU可用线程数*2，这里指定大小为1，模拟单线程情况
EventLoopGroup group = new NioEventLoopGroup(1);
ServerBootstrap server = new ServerBootstrap();
server.group(group, group);

// 多线程
EventLoopGroup boss = new NioEventLoopGroup(1);
EventLoopGroup worker = new NioEventLoopGroup();
ServerBootstrap server = new ServerBootstrap();
server.group(boss, worker);

// 主从多线程
EventLoopGroup boss = new NioEventLoopGroup();
EventLoopGroup worker = new NioEventLoopGroup();
ServerBootstrap server = new ServerBootstrap();
server.group(boss, worker);
```



因此，具体的执行逻辑即为：创建两个EventLoopGroup，分别作为客户端的boss和worker，通过NioEventLoopGroup的构造方法指定线程个数，如果没有指定则是CPU线程数（通过Runtime中的信息获取到的）*2 。

然后创建服务器的引导类ServerBootstrap，通过ServerBootstrap的group方法实现对boss和child的装载，此时有三种状态：单线程，多线程，主从多线程。然后书写对应的业务处理逻辑即可。