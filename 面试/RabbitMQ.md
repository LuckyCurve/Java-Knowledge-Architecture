# RabbitQM面试常见问题



RabbitMQ的特点：提供图形化的管理界面、支持多语言客户端、相较于其他MQ虽然吞吐量不是最高的，但由于他是Erlang开发的，在并发度上非常优秀，有很低的延迟，达到微秒级，其他的都是毫秒级别的、消息的可靠性：通过持久化，传输确认以及发布确认等来保证可靠



RabbitMQ的模型：

![图1-RabbitMQ 的整体模型架构](http://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-12-16/96388546.jpg)

几个核心概念：

Product：消息生产者

Consumer：消息消费者

Excahnge：通过将消息直接发送给Exchange，然后Exchange负责将消息发送到各个Queue当中

Exchange有三种类似：Format、Direct、Topic

Format：相当于广播到所有相连的Queue中，不做任何判断，速度非常快，可以用来广播消息

Direct：进行消息头与Binding的Key的匹配，然后进行发送，一般用于优先级的区分，按照优先级Routing Key来将消息划分到不同队列当中

Topic：和Direct非常类似，但是允许进行模糊匹配

Binding：连接Exchange和Queue的纽带，通常会指定BindingKey来进行与RoutingKey的匹配

Queue：消息投放到Queue中，可能存在多个消费者同时订阅一个Queue的情况，这时候Queue收到消息后会让消费者轮询处理，而不是每个消费者都接受到消息，避免被重复消费

发送的消息包含消息头和消息体，消息体是不透明的，消息头是对MQ可见的，主要包含一些信息如routing-key等信息。



RabbitMQ保证消息的可靠性

RabbitMQ保证消息的可靠，我们只需要保证四个步骤的可靠：

1、生产者生产出的消息可以顺利到达RabbitMQ

2、RabbitMQ可以收到消息并且正确分发到Exchange上

3、Exchange消息传递到Queue上并保证消息的持久性

4、消费者可以顺利收到消息



1、保证消息顺利到RabbitMQ的解决办法：利用RabbitMQ提供的一个发送确认机制，和发送TCP数据包非常像，每次接收到之后都会回一个请求个RabbitMQ，如果没有接收到消息则会进行重发

2、可能RabbitMQ接收到消息之后无法找到Exchange，又或者是Exchange找不到Queue，此时为了保证消息的可靠性，需要将消息回退给客户端

3、可能突然宕机了，需要做持久化来保证消息的可靠性

4、消费者可能无法接收消息，和1的处理一样，发送确认，否则一直重发



因为上面涉及到了数据的重发，可能因为网络波动问题导致消息重复，为了避免消息重复，我们需要保证消息的幂等。

通过让消息持有一个全局唯一的ID即可，将ID存储于Redis当中，每次处理消息时候查询Redis是否包含，如果不包含则直接处理，并将ID写入到Redis当中，如果包含了则证明处理过了，直接丢弃消息。