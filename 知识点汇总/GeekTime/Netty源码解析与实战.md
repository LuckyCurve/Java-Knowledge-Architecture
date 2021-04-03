# Netty源码解析与实战



Netty对三种IO模式都有所支持，但是现在BIO和AIO大都已经废弃了，因为BIO可扩展性不强，AIO维护成本较高，NIO模式成为了Netty唯一推荐的IO模式了，针对NIO模式下，我们使用的开发方式称为Reactor。



Reactor的简单总结：

Reactor三种模式：

1、Reactor单线程模式：一个线程包揽所有的工作【单个线程循环利用】

2、Reactor多线程模式：一组线程包含所有的工作【使用线程池技术对一组线程进行循环利用】

3、主从Reactor多线程模式：每一组线程都干特定的事情