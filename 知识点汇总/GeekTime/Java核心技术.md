# Java核心技术36讲



主要是站在整体的角度来审视Java语言





## 谈谈你对Java平台的理解

- 语言特性
- 基础类库
- JVM
- JDK提供的工具

![image-20200624160017106](images/image-20200624160017106.png)

JIT（即时编译器）中的C1对应着虚拟机的Client模式，适用于启动速度敏感的应用。C2对应着Server模式，他的优化是为了长期运行的程序设定的，默认采用所谓的分层编译



Java9提供了平台相关性更强的编译性工具AOT，直接将某个类或者模块编译成AOT库，避免虚拟机启动时候过长时间的预热（适应微服务的一种体现吧）

提供的工具，JDK里面就有了：jaotc，但是使用起来感觉一般，估计还得需要时间普及





## JVM的参数及其作用



|               参数               |                     意义                     |
| :------------------------------: | :------------------------------------------: |
|              -Xint               |            告诉JVM只进行解释执行             |
|              -Xcomp              |       告诉JVM关闭解释器，启动会非常慢        |
| -XX:SoftRefLRUPolicyMSPerMB=3000 | 设置软引用在最后一次引用过后的3000ms后被回收 |
|                                  |                                              |
|                                  |                                              |
|                                  |                                              |
|                                  |                                              |
|                                  |                                              |
|                                  |                                              |
|                                  |                                              |
|                                  |                                              |
|                                  |                                              |
|                                  |                                              |
|                                  |                                              |
|                                  |                                              |
|                                  |                                              |
|                                  |                                              |
|                                  |                                              |
|                                  |                                              |





## Java的异常处理



主要讨论的是Throwable类的两个实现：Exception和Error、以及运行时异常和一般异常有什么区别

Exception是在程序运行情况下可以预料到的意外情况，应该捕获并进行处理

Error是正常情况下不大可能出现的情况，会导致程序（JVM）处于非正常的，不可恢复状态，例如OutOfMemoryError类



Exception可以分为检查check异常和不检查uncheck异常，检查异常需要在编译器强制要求处理，不检查异常就是所谓的运行时异常，例如NullPointException、ArrayIndexOutOfBoundsException之类，不会在编译器强制要求处理

简单类图：

![image-20200625214059392](images/image-20200625214059392.png)



Java对异常处理的支持（应该是Java7之后的）：主要就是Try-with-resources和Multiple catch两个语法糖的加入

```java
try (BuferedReader br = new BuferedReader(…);
	BuferedWriter writer = new BuferedWriter(…)) {
    // Try-with-resources
	// do something
catch ( IOException | XEception e) {// Multiple catch
	// Handle it
}
```



几个常见的使用误区：

- 尽量不要捕获Exception这样的通用异常，而是需要将捕获异常具体化
- 不要生吞异常，就例如catch不作任何异常处理，诊断起来非常难
- e.printStackTrace();   不要在产品代码中使用如下代码，很难判断出到底输出到哪里去了，特别是对于分布式架构的系统更为如此
- 尽量遵循Throw early，Catch late原则

throw early：也就是对参数进行检查，避免最常规的空指针异常问题，例如以下的两段代码：

```java
public void readPreferences(String fleName){
    //...perform operations...
    InputStream in = new FileInputStream(fleName);
    //...read the preferences fle...
}
```

```java
public void readPreferences(String flename) {
    Objects. requireNonNull(flename);
    //...perform other operations...
    InputStream in = new FileInputStream(flename);
    //...read the preferences fle...
}
```

new FileInputStream都可能会因为传入的filename为null而报错，而1中生成的异常堆栈信息就不是很适合读取了，而2中可以直接定位到第二行代码报错，非常容易解决

catch late：中庸的办法：保存原有的cause信息，直接再抛出或者是构建新的异常再抛出



自定义异常：

- 自定义异常是否需要声明成checked Exception

几点不需要声明成checked Exception的例子：

1. Checked Exception的初衷就是为了从异常情况恢复，但是绝大多数情况不支持
2. 不兼容functional编程，lambda和Stream无法使用

> Spring、Hibernate就基本抛弃了Checked Exception，例如新的编程语言Scale就直接抛弃掉了Checked Exception

当然也不是绝对，在少数环境下异常是可以恢复的，如IO，网络的重复连接等等。

- 在保证诊断信息足够的前提下，避免敏感信息存储到了异常中去了，例如Java中的Connection refused异常就不会包含用户的IP，端口号，机器名的输出。





异常处理的性能开销：

1. try-catch会产生额外的性能开销，并往往会影响JVM对代码的优化，因此try代码块不要过大
2. 实例化Exception会生成一个栈的快照，重量级操作

第二点也是优势的地方，可以大大减少代码诊断的难度，尤其是在分布式项目和大型项目中





课后问题：对于反应式编程，因为其本身是异步，基于事件处理的，所以出现异常决不能简单的抛出去，另外，因为代码堆栈不是垂直调用形式，生成的异常和日志往往都是特定executor的堆栈，而不是业务方法调用关系，对此你有什么想法呢？

优质回答：

> 先说问题外的话，Java的checked exception总是被诟病，可我是从C#转到Java开发上来的，中间经历了go，体验过scala。我觉得Java这种机制并没有什么不好，不同的语言体验下来，错误与异常机制真是各有各的好处和槽点，而Java我觉得处在中间，不极端。当然老师提到lambda这确实是个问题...
> 至于响应式编程，我可以泛化为异步编程的概念嘛？一般各种异步编程框架都会对异常的传递和堆栈信息做处理吧？比如promise/future风格的。本质上大致就是把lambda中的异常捕获并封装，再进一步延续异步上下文，或者转同步处理时拿到原始的错误和堆栈信息
>
> 作者回复：
>
> > 是的，非常棒的总结，归根结底我们需要一堆人合作构建各种规模的程序，Java异常处理有槽点，但实践证明了其能力；
> > 类似第二点，我个人也觉得可以泛化为异步编程的概念，比如Future Stage之类使用ExecutionException的思路







## final，finally，finalize有什么不同



final可以修饰类，变量，方法。被final修饰的方法默认是不可重写的

final!=immutable：只能约束引用不被另外赋值，无法保证添加元素等操作是正常的，如被final修饰的List依然可以执行add操作，如果要保证绝对的immutable，可以借助List.of()方法等相应的类和方法来进行支持，Java没有提供原生的支持

如果要实现immutable的类，需要注意以下几点：

- 将class自身声明成final，避免别人使用扩展类来绕开限制
- 向所有成员变量定义成private和final的，不实现setter方法
- 构造函数进行赋值时候要使用深拷贝来代替浅拷贝，避免被final修饰的内部对象改变了的问题
- 如果需要实现getter方法，使用copy-on-write原则，复制出一个副本



finally保证重点代码一定会被执行，如资源的关闭，锁的unlock动作等

不过现在更加推荐Java7的try-with-resources语句

也有特例：如以下代码就不会产生输出：

```java
try {
    System.exit(1);
} finally {
    System.out.println("hello world");
}
```





finalize是Object的一个方法，保证在对象被垃圾回收前完成特定的动作，从JDK9开始不推荐使用

会导致对象回收变慢，大约是四十到五十倍的速率变慢

Java平台正在使用Cleaner来逐步替换掉原有的finalize实现，比finalize更加轻量和可靠。仍然是有缺陷的，要避免所有对对象的强引用，否则很容易造成内存泄漏





## 常见的引用类型

强引用，软引用，弱引用，虚引用（幻象引用）



不同的引用类型，主要影响的就是对象的可达性状态和垃圾收集的影响



强引用：无论如何都不会被回收

软引用：JVM会在确保抛出OutOfMemoryError之前清理软引用对象，常用于实现内存敏感的缓存

弱引用：无法使得对象豁免垃圾收集，仅仅提供对象的一种访问途径。例如维护一组非强制性的映射关系，如果对象还在就使用，否则就重新实例化，也是很多缓存的实现

虚引用：无法通过虚引用来访问对象，虚引用仅仅提供了确保对象被finalize以后，做某些事情的机制



对象引用的转换关系：

![image-20200628201039550](images/image-20200628201039550.png)







## String,StringBuffer,StringBuilder之间的区别

String：是一个经典的Immutable类，被声明成final class，所有属性也都是final的，导致字符串拼接，裁减等操作都会产生一个新的String对象，会对性能有明显的影响

StringBuffer：提供线程安全的方式来修改字符序列，也正是由于其线程安全，导致其效率不高，所以除非有线程安全的需要，还是建议使用StringBuilder（因为大部分的对象都可以是线程私有的，绝对的线程安全）

StringBuffer是直接将修改数据的方法加上Synchronized来实现的，非常直白，不必纠结于Synchronized性能之类的，有人说：“过早优化是万恶之源”，考虑可靠性，正确性和代码可读性才是大多数应用开发最重要的因素

StringBuilder：1.5以后新增的一个类，相当于去掉了线程安全的StringBuffer，是绝大多数情况下字符串拼接的首选



通常不用使用到以上这些类，在JDK8之前Javac操作会直接将其转换为StringBuilder的实现，在Java8之后则不会再Javac时期执行优化了，而是直接使用JVM的InvokeDynamic来进行优化



String的存储方式从JDK9之前的使用char数组变成使用一个byte数组加上一个标识编码的coder以减少Char占用两个字符所带来的额外的内存开销





## Java的动态代理



谈到动态代理就不得不谈到反射机制



反射机制是Java语言层面提供的一种机制，允许我们可以直接操作类或者是对象，例如：获取某个对象的类定义，获取类声明的属性和方法，调用方法或者是构造对象，甚至可以运行时修改类定义

动态代理是一种方便运行时动态构建代理，动态处理代理方法调用的机制。使用到的场景有：AOP面向切面编程，包装RPC调用等等



实现动态代理的方法有很多，例如JDK自身提供的动态代理，主要就是利用了反射机制，还有更高级别的字节码操作机制：类似cglib，ASM等

动态代理底层并不一定是由反射来实现的



Java语言通过本省语言的反射机制做到了灵活操作很多运行时才能确定的信息，而动态代理则是延伸出的一种广泛应用于产品开发中的技术，很多繁琐的重复操作可以通过动态代理优雅的解决



动态代理是一个代理机制，代理可以被看做是对调用目标的一个包装，这样对目标代码的调用不是直接发生的，而是通过代理完成，有点python装饰器的味道了

通过代理可以使得调用者与实现着之间的解耦，例如常规的序列化，反序列化对调用者来说是毫无意义的，通过代理可以提供更友善的界面，也为应用插入额外的逻辑提供了便利的入口

Java提供的动态代理使用：

```java
public class MyDynamicProxy {
    public static  void main (String[] args) {
        HelloImpl hello = new HelloImpl();
        MyInvocationHandler handler = new MyInvocationHandler(hello);
        // 构造代码实例
        Hello proxyHello = (Hello) Proxy.newProxyInstance(HelloImpl.class.getClassLoader(), HelloImpl.class.getInterfaces(), handler);
        // 调用代理方法
        proxyHello.sayHello();
    }
}
interface Hello {
    void sayHello();
}
class HelloImpl implements  Hello {
    @Override
    public void sayHello() {
        System.out.println("Hello World");
    }
}
 class MyInvocationHandler implements InvocationHandler {
    private Object target;
    public MyInvocationHandler(Object target) {
        this.target = target;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args)
            throws Throwable {
        System.out.println("Invoking sayHello");
        Object result = method.invoke(target, args);
        return result;
    }
}
```

仍然需要实例化Proxy对象，而不是真正的调用类型，带来了一定的不便



如果使用另一种方式（参考Spring AOP实现的两种方式） cglib就完全可以避开Proxy对象的使用，直接操作接口。cglib使用的是创建目标类的子类的方式来实现的，就可以直接通过Java的向上转型机制来完成很好的调用

且cglib拥有跟高的性能和更流畅的编码体验，但是在JDK版本升级的时候不如JDK Proxy那般过度平滑，且编写门槛要低得多







## Java的自动装箱/包装类

包装类都设有缓存，以防止太过频繁的创建对象导致性能的提前降低

只是包装类缓存了数值，对应的基本类型并没有缓存有相应的数据

包装类的缓存范围：

- Integer：-128~127
- Boolean：true/false
- Short：-128~127
- Byte：数值有限，全部缓存来了，表示范围 -128~127
- Character：缓存了从`'\u0000'~'\u007F'`的字符





在实战中，建议避免自动装箱和自动拆箱的行为

但还是以开发效率为先

之所以泛型不支持原始数据类型，主要是因为：在javac的过程当中需要将泛型全部都向上转型称为Object对象，而原始数据类型是无法转换成为Object对象的，如果每次转换都需要单独处理会显得过于慢了

其实使得Java的List无法存储原始的基本数据类型也是一件好事情，因为List存储的都是对象的引用，对象通常都分布在堆的各个区域当中，无法保证对CPU缓存的最大利用。而单独的原始类型数组则可以很好的利用CPU的缓存机制。

> 所有技术都是有利有弊的，例如这个存储对象的分布虽然降低了缓存的利用率，但也极大的避免了JVM的垃圾收集器工作过于频繁的问题。
>
> 当然，OpenJDK现在正在致力于解决这些问题。





## List集合框架



主要的成员有：Vector、ArrayList、LinkedList



Vector是早期的线程安全的动态数组，可以根据需要进行扩容，扩容时候会创建新的数组并拷贝原数组的数据



ArrayList是动态数组的实现，不是线程安全的，效率高很多，也存在扩容问题，与Vector不同，Vector是直接扩容1倍，而ArrayList仅仅只是扩容50%



LinkedList是双向链表，不存在扩容问题，不是线程安全的

![image-20200703233024189](images/image-20200703233024189.png)





利用JDK9提供的容器静态工厂方法可以很轻易的创建出不可变的集合对象，如：

```java
ArrayList<Integer> list = new ArraySlist();
list.add("hello");
list.add("world")
```

可以使用如下代码代替(不可变的)：

```java
List<String> list = List.of("hello","world");
```







## Map集合框架



最常见的Map的实现：HashMap，HashTable，Treemap，



HashMap和HashTable都是哈希表的实现，前者不是线程安全的，后者是同步容器

HashMap进行get和put操作，可以达到常数时间的性能，因此它是绝大部分利用键值对存储场景的首选

TreeMap则是基于红黑树访问的一种Map，他的操作的时间复杂度为O（log(n)），具体顺序可以指定Comparator来决定，或者根据键的自然顺序来判断



着重了解HashMap



HashMap的性能表现非常依赖于哈希码的有效性，以下是一些hashCode和equals的基本约定：

主要就是：hashCode相等不一定equals，equals了的hashCode一定相等



有序Map的分析：LinkedHashMap和TreeMap

- LinkedHashMap提供的是一种遍历顺序符合插入顺序的一个Map，顺序主要是插入顺序，一些特定场景例如空间占用敏感的资源池就可以使用LinkedHashMap来实现
- TreeMap是由键的顺序来实现的，因此键不能为null。通过Comparator或者Comparable来决定





HashMap的源码分析：

- 内部实现
- 容量和负载稀疏
- 树化



内部主要是有数组和链表来实现，数组被分为一个个桶，通过键的哈希值来决定这个键值对在数组的位置，对哈希值相同的键，则以链表方式存储。如果链表大小超过阈值（TREEIFY_THRESHOLD 8），图中的链表会被改造为树状结构便于访问时候的高效性

![img](data:image/png;base64,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)

HashMap具有懒加载的特性，数组并不会在你new一个HashMap的时候创建，而是在putVal中时候发生对size的判断再进行HashMap的初始化（其中后续的扩容，树化操作都与这个方法有关）



HashMap的key的哈希值并不是key的hashCode方法算出来的，而是通过以下方法处理：

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

这里为什么要将高位数据移到低位进行异或运算呢？主要就是为了防止某些计算出来的hashcode只有高位数据有效，这样可以有效地避免哈希冲突



resize方法也很重要，分配到扩容，很容易被面试官问道



树化改造的逻辑主要在treeifyBin中







## 如何保证集合是线程安全的？



Collections的synchronized方法，粗粒度的加锁，在高并发条件下性能比较地下

使用JUC包来代替 

主要的面试问题：

- 基本的线程安全工具
- 传统的同步容器Map存在的问题
- 梳理JUC包，尤其是ConcurrentHashMap采取了哪些方式来提高并发表现
- ConcurrentHashMap的历史演进，很多都是停留在早期的版本



ConcurrentHashMap的设计实现：在Java8发生了较大的变化

早期ConcurrentHashMap，其实现是基于分段锁来实现的，在进行并发操作时，只需要锁定相应的段（HashEntity，结构直接类似于HashMap，也直接以链表的形式存放数据），这样就可以有效避免了类似HashTable整体同步的问题，大大提高了性能

HashEntity的大小为concurrencyLevel，默认是16，可以通过构造方法来实现

ConcurrentHashMap的扩容不是对整体进行扩容，而是对每一个HashEntity进行扩容

当需要获取全部锁，例如计算所有HashEntity的总和的时候，或许会因为并发put导致获取结果后续被修改了，又或者是获取全部的锁，但代价果高

ConcurrentHashMap采取类似CAS的重试机制（RETRIES_BEFORE_LOCK）尝试获取可靠值，如果检测到了修改就需要重新获取了。





Java8和之后的版本中ConcurrentHashMap发生的变化

- 同步与HashMap的数据结构，当链表过长时候转换成红黑树
- 初始化操作改成懒加载模式，应该和HashMap一样是在putVal中分配内存
- 内部数据使用volatile来保证可见性
- 使用CAS操作，实现特定场景无锁并发







## Java提供的IO方式



包括NIO



传统的java.io，也是我们说的BIO，会一直阻塞在数据流的读取或写入阶段里，是同步阻塞的方式，他们之间提供了可靠的线性顺序

优点：代码简单直接

缺点：效率和扩展性存在局限性，容易成为应用的瓶颈

有时也将net包下的基于网络的操作也归类到了IO中



1.4提出了NIO框架，Channel、Selector、Buffer等新的抽象，是多路复用的，具有同步非阻塞的特点

同步非阻塞的由来（网上看到的，不一定准确）：对IO流，非阻塞：程序执行不会一直阻塞，而是去干别的了（具体是啥也没讲清楚，可能是向下执行代码），同步：该线程需要定时读取Stream，判断数据是否准备好。提供了更接近操作系统底层的高性能数据操作方式。



1.7的NIO2，异步非阻塞方案，也叫AIO（Asynchronous IO），直接将读取工作交给另外一个线程去执行



专栏里给的概念解释：

![image-20200803191505821](images/image-20200803191505821.png)







## Java的文件拷贝方式



使用IO库中的Stream或者是NIO库中FileChannel方法实现

```java
public static void copyFileByChannel(File source, File dest) throws
        IOException {
    try (FileChannel sourceChannel = new FileInputStream(source)
            .getChannel();
         FileChannel targetChannel = new FileOutputStream(dest).getChannel
                 ();){
        for (long count = sourceChannel.size() ;count>0 ;) {
            long transferred = sourceChannel.transferTo(
                    sourceChannel.position(), count, targetChannel);            sourceChannel.position(sourceChannel.position() + transferred);
            count -= transferred;
        }
    }
 }
```



Java在语言层面已经提供了Files.copy方法。





区别：



使用输入 流来进行文件操作时候，会进行多次用户态与内核态的状态切换

![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAqAAAAHxCAIAAAFKY6nLAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsMAAA7DAcdvqGQAAGDYSURBVHhe7d19UFTnvi9461ad2jV3Tp25mTlzz9yaPWdTNzV1qm6d/yb/3b2ryJw95a3MWOVcq3w5O3XCJEUuIVsCiiGiBt8J0ehW8RURDARxQ2SHYDAcFRWVYFAQo/KiDYja8iav0oKNzK95Hh8XT7/QL+t9fT/1q/Z5nl7dvVY//evfWm2zesGMg2HjnQobP59ly5bxlvZycnLoMjMzk3U1FWrjFyxYwFblzTffZCMMjfNWzBYuXEj3NjAwQO26ujrxiOwhmpubfQup+ohKmtypVWDjre/dTXkzbR9KkV+8lV8dhOU3funGr6VtluLCmbV8UT/Wn3m/rQ0QQWDjLe3VFrrqP9mwd4e7cSVF7rGtVX9ZJ67iS/rBzFuatJ0BIwhLbvydx+O8RZQb2Vs6Z1PFeBCGbfwf8m6HE3zpEMQWUow1zwydlwcpgrDSxicVtfKWkrSdASMIe818sAjCjBu/71yPaNOSj4aes5sEJm1nwAjCmI3vG50Smzf5YppGvr7qFiPK9pz3tkBaXE/kTZ0bi9cV8EX96L3xW6u6WENsXuhgC8+rse2RtM0UO0tn3/yC02/js3/gm81IGxks+NLa0GPjhyde8JaCtJHBgi+tDTO+4SmDL60NwzbeDAJvfF1L18yCBaHD/Td/y5dWaOwa5S0rCDLzfpsaNF5JLm7jLesIsPFvpx2VtzBEzMxU3OhnN7QcFTY+GLfbzVtmpeHGFxQE3bUyifA2/tmzmaIiX2NgYM44RXBZWVm8ZVZhzzxtP10S5SAbCcJGGx8sgsvOzuYts9Jw481PhY231o6NUpCpk7YwRARXX1/PW2al/ut21ckO1tDoP9VVpNX6PZv0xsXF8Y4f6dAtRPAbaEPDyWEbP+V9ybpK0haGCH4DbWi48ampqbw1M5N4fM6nztIWhgh+A20Yk5bSFoYIfgNtGLDx9CqQtpCCxrsHPdS43jUqjWtHw43v7OzkLT/KzQsdtHCDa4TdKpgVGwN8J4VF5an1fKFANNz4EEd10haGCH6DEPw2OEAEYaWNzzx1n7eUpO0MGEFouPG1tbW85UfawhDBbxBC24cNZ9fM2VRFpOTs9jWC0HDje3p6eEtt3mnFvsPcrQ0cQWi48Vq73DHs+0fazoARhIU3nhRecc/ZQulSRBDW3ngfsYVjzTPecd83U4jn/utxiiBstPEUbJulQTYSiL02PlgEgY23NGk7A0YQ1t94Im2qFMHZYuNfabjbs2LTcYrKK3f5UEi22vhIYeOdChvvVNh4p8LGOxU2PiT2Z80aCfg/ubr9xXrgjad1klbrrbfeopGBgQG6PH36NB9VA92heH7Fg7I/LWddMag6re43RrTxvKUlk268Puyy8dIx/Gw0dTzm1wZh/Y3322Y5grP/xvu+XBaE5Td+6YYD0tYqo/HcGjtvvNjOxG17m2pXU4MuF312SIzbc+P7Rqd8/7zayGBhw41//R+1flsrhSNe9q9j7rjpNl76BkKI4Dd4ZeePD3hLENs81swbc/+X0j4bH4BiOwOG5Tfe9//wwfhtrRSY+cDMsvE0GOWXEP22Vo7grD/zs9J375e2Ob94M78uOJtsfHQ03/iuAQ9vKUhbGCL4DbSh4cavKbvHW36kLQwR/Aba0GTj8y494i1zC7zxDXH/OLPA72+J5sa7Kfv40goB/67CtAJtvN92BouO//l/4zexppg2nmJXjd/OtnXEuvH8JtaEjZdIm1df77skv/nNnHEWVhbezN+86Rtnf0UuhZWFt/Ehwsqw8RJp80KH+BTVgmLa+HczXn9OkPWdi7esw9qv2xhptfFn7zzlrUCkQ7dgwZfWjLYz/+31Pt6aS9rIYMGX1owxL3tpI4MFX1ozum68OAqSNlIZ0tk/NWXAzNNTIDYv9Nk/5/zlpAZs8bKf+7ntnAjJGhsf+O+nGWlr/cL1OGjdsf7M+22tFFb97yq+tEKAszG1fdh3/Y/KrZXCdBsfo4j+hNxuGz+H39ZKYduN931XQWwnUZ7c/9W4M2aesI1n7Vfjjtn4yV5fw+RfS1GTYjsDBjY+MPtv/J7yOr6kH8tvvO975X4bLOLdrcV8uUCsP/MKWwqr6UWetq+c9+djq42PFDbeqbDxToWNB+fBxDsUJt6hMPEOFdnEL1u2rLm5eWBg4I033qBuXV0d+2188cv8H330ERthpzrIyclh53/Q7swHsTh06BBrsNVjm3P//n122gLaKNpScUYHtjBtDttYWpg9CTRCy4jnxCrCmg96XthTQ5dvvfUWbSHrKk9zwUbokq6l6RddQouxrgnRirFTc1BbbA5NKl36Vn3uqTwIW5gWYF0xLp4Tq7DSuoKKMPEmdburN6e4JuVPf3477eiKTV9T+9yN4F/NiBwm3ngdDwe8rfL/NIQZiTtO8nuJECbeaH5zGUUcrLzK7y1smHiDXajOkGYxigjx/8/BYOKN9mry3vn0iGiHE/zXWWYDE28xZY2zXxyKOTDx1pBa2s5bxG8WowhMvKlt/j7Qr9b5zeLr8bHmmaHzcwYJ684NTPw8pO+wxx78fkOa51eK/WbRN8im/MEO/vVR9lVS73jg5THx85KmLfbg9xtIuOdK8ZtFXxCadUY0BGlhTPy8pGmLPfj9zjXq8fJWOPxmMYrAxM9DmrbYg9/vrKqWAd6KiN8sRhGY+HlI0ybikxPt0ogUyr9oVQbVb+k3tSPmN4tRROuDfn5vYXPExFffGmR/didNmwi2mGjTpTjzZMWNvprbg8Emnt0wdu9vOyzNZThxoTrDO+37q+so2HbiAyaiNG2xB79fC7LPxFNOhz4fCZGmLfbg92tB1p740mu9vBUeadpiD36/FmS9iU8ubuOtyEnTFnvw+7UgC0y8Z2r6Zs8Y74BKwp3495J3zyyQT/wWawR3+GKoMyN3dgb60BsiEdbEv7++QJ4ztUIh1Ol+5sLExy6sifd9MCRNmEqRd+nR1qou/jBhq62t5S2IlsETzx8gQtXV1bwF0VJv4jdvlkfCCTCIShPPUKO9nY+kpPBG6ACDqJfxLHSZ+Lt37/IWREvtiY80ohIfH89bEC2DJz66j+Ew8bELa+LzK3+SJky1mCunupu3QoqLi+MtiFYkb7bSnMUWK1bu5XcbRM/T57cezn690M/y5ct5C6IVZZXV34Hah7wVLen/V2IPfr/WZJmJl0TxhSdp2mIPfr/WZMmJ//nnn3lrlnf6ZclPT3gnOGnaYg9+v9ZkyYkPvVd/9d7I8MQL3lGQpi324PdrTZac+ISEBN4KQ0b5vdzzPdSQpk1E1F+xZfdvUZac+KysLN6KhDRtIpTXskb4X7GlV9XsrdXR8XAg6YtD0ldp/SO3IDv31GV+m2hZcuILCgp4KxLStMUe/H5nNbhGeCtCvh8O8pvaiGLJhkJ+X5Gw5MRHR5q22IPf71y0pznnp3Pm5TeRUYTvo9UIYeKjD36/wdFuJm+F0PbhWEuychYjDVf9J06Z+BcvAuy0z0uattiD328Y1pQF3xWYO4t1ZzKWbjjQejlNGmdBL5GN+3YcPL5FGnfKxFv3O3frK/x+i3juFEYXeKu3jBvdvgMHX8tvFqMITLwF+c3inFD+oJ6gXGA2MPEW5DeLvhhr5g3vuO9SiY3PDUy8BfnNYhSBibcgv1mMIjDxFuQ3i1EEJt56Fq09Js1ipNFycTW/r0hg4s1i8oU3cXu4J0QpOrGpriXiPz1TwsQ7FCbeoTDxDoWJdyhMvENh4h0KE+9QmHiHwsQ7FCbeoTDxDoWJdyhMvENh4h1q/olnv4e/cOFC1g1Iz9/MV+u71W+88QZvhZSTk8Maem6jDiKbeNZ+8803BwYGDh06xLp1dXV0SZYtWyYWZs8X69L46dOnfUuooafH96evsaN1++ijj6jx1ltvNTc3s82hVwNbZ9pA9spgGyJe9zROm08Nupba1GDLs+fEt4RF+FY6NLZhbMtpCtkIEanAUJctSdgrQDxZs4vP/0BhUivjaZVoslmDKF+pbFtEmy4JzSsRr3KaabEMUbYtYf51pWygTaLkEFM7u42vJ54mmLr0pLBxtgBLF9FlDVWodQZjsUps9aSJF6kvxllXjLMReidTdqlhFWquq7W2PDoi463OelMV3R/Hg8T+OQoBIeNN6uT5lvT9FSs2HV+cWZC2r3xP+aW+ocCne4yO9Sberj9PkXvqsu8b8n7fpPYPb+uHSzce5zeLFibeeInZedLUhhkXqjP4XUTOehMf0bnOLMBvOiONFtf8J3f0h507Ix08vl2axSgjcsh4I92+tEqewugicsh4I8V4visWi9f6PniOlPUm3lY/T9H24cQvScpZjDRyDmX7GpGz3sTb6kfnXs1f6+W08vLPRXfe6Lv+x+0Hcl6PRM56E5+dnc1bNiBmThF78re98+kReh001a52N66ko/buhpS6MxmJ2/Zm/mmntDCPyFlv4m3zgzQ3ukfl+Ys6IoedO2Nk/zB7WgNp/qKOyFlv4q3+n79zTnEszV/UETlkvK7k39mT5i/qiBwyXj+D41O8JUjzx0JcNXR+zgjDRqSInPWexLKyMt6yjqC/lSTNHws2/mDHjOc+7yqJxZQROetNfHp6Om9FSHnGcVUizLPLVTb385Y/af5YTPb6cp0uxYls2bh3/PV7gBSRs97ER304J01b7DHvxPeN+r23S6T5YyGmXBn0BkDjwW4SOavWyyhI0xZ7hJ741NJ23gpBmj8WZKzZN9OEslzi/5qgiJz1Jj7qz+qlaYs9Qkx8uD9LI81f1BE5ZHz0EXDiA/wGRQjS/EUdkbPexEf9//HStMUe/hPf6n7GW2GS5i/qiJz1Jj7qP2mQpi32UE58WD845U+av6gjctab+Ly8PN6KkDRtEUX3oEcaoRATv+dstH/HKc1f1BE56018dL8vSqRpY0Hjky+mpUH/cA9PSiMUNPF0ye48OkvWH5WnMPKYvJPE7y4STt+5o/F953pYW/ykLGuIW3191R1w4v+vjG/YMrGo+DaC718Ejqg4IuPZnrY0bSxofPP3ndRgvx/MBqWJpwiW8WyZGF1odnlb/aYzjHj/ixJ+F5GzecZXtbw+WYE0bbGHWhMvLNlQWHcmQ5pdKYZvJi/67OjkCy+/TbSsN/Hh/O3co6HnT0YmeecVadpiD9UnXk/Wm/jQp0KpuBH0f0SkaYs9MPG6unv3Lm/NlXnqPm8FIU1b7IGJ15X/H00mFbXyVkjStMUemHjDRPTJiTRtsQcmXlednZ1T3pcNrog/IpWmLfbAxOvnzuPxZ5NRHslI0xZ7YOL1UFTvZg21znPncBaY+LC+ygIRimDiV3yS6/6bv51ZsCDGaIj7x+prYc3lqpMdvAVqC3vi/eYvxkj8b1/yew5EvLGDRsKa+KZf/4M0beqEH9px6+id4B3QUngZL02YSrEi4/Ve8dV7I+F+QRHUYOTE0+HQv+Tf2VXzgD8K6MjIif/9p18fvvgoio9iIHYGZzy/f9AdJt6hMPEOpdLEh7OMX2DiDaTSxP/mN77LZctmbt70telyYGDOAoECE28gNSaeZppJSfF1SXs7b4hlAgUm3kBqTDxbgNKdJp7QrFMQ9joIHph4A6kx8SLETLOMny8w8QZSdeIjDEy8gTDxDoWJdygjJ/73a45H+WflEDNTZHyYX4wHFRk58bl/vsTvf1bJT0/wX/K6CWviEzcUSnOmTgQy5X0Z4u/fQC3hZfzMzNJP9svTFmPM54PCwH8jB6oId+KNcvbO06j/ggJCMPvECwdqH/JWVKQ/gok9rH4sapmJZ6L+4wpp2mIPTLwBbj0c73n6nHfCI01b7IGJN1LWd+GeP1SattgDE288/sM+IUnTFntg4s2ib3SqsWuUd/xI0xZ7YOJNR/7Bn1nStMUemHiTOnb5MW/NkqYtogh9IluLsu3EM1Pel1UtA/QeIE0bC1oglhPZsoewKJtPPGlwjWSeui9NGwu6NuoT2f4/W6rYMhZl/4lnpGljQeMxnsh2/p8bisT7X3xTU7VWOoepFA+vpSxck89vEANHT3wsId7qpZ2JKDTc7fGdetxvjueNlD3l/C4ih4mPMqQaX3glylN4FJ3YJE1nxBEVTHyU4b9zF8Xb/uJMFX6oYOIX/FBBcNK0xR7B9uoj+hqZu3GlNItRRuQw8VFGsIlnwsz+nEPZ8hRGHvnFm/ndRQITH2WEnngS1om72j5M3LZXOYuRxq6j232NyGHio4x5J56ZZ5//1fwt3XBAtMOMylPrX3cjh4mPMsKceBLgZ+MFMXOvIv2rr06ezJIGRdSdyXjn0yMB9gwi55SJN1zgnT5p/qKOyGHidSX/Uo40f1FH5DDxeiv56QlvEWn+oo7IYeKNkXfp0c2eMXn+oo7IYeKNcbFtaGtVlzx/LJjJXnmEEYPKiBwm3lDS/LHwjvsuPfdnekt9DYlYTBmRs+TEv3jxgresTpo/FoSmfOj867mnGGv2jbPXhH9EzpITH/o3B61Emj8W9CZP800zLUaUxKAyImfJiff/6TmrkuaPBVG+ybPBBzt8LwjRlSJyqPGGkuYv6ogcMt5Q0vxFHZFDxhtKmr+oI3KWnPhwfkncGqT5izoiZ8mJHxsb4y2rk+Yv6oicJSe+rKyMtyzuSNHs1yhij8hZcuKzsrJ4ywakKYw8WlyK//UJG3bujJeyI+Kv37BoPLeG30XkLDnxCQkJvGUjO0+c7W5IkaY2WLyTcYzfLFrIeDOaeD6Vf/rntH3lC9fkv512NPHL0pziGtfjp/xqNSDjHcqSE+924yeHY2XJiV++fDlvQbRQ4x0KE+9QmHiHwsQ7FCbeoTDxDoWJdyhMvENh4h0KEw/gIEh4AAdBwgM4CBIewEGQ8AAOgoQHcBAkPICDIOH10NnZyVsAhlIn4RcseH0/Cxcu5K1INDc3v/nmm9Q4ceIEGwHtsPl66623Tp8+zUailjOLd2Zmli1blpmZSY2PPvqIjYCpaJXwdXV17HVArwC6pAXu378/MDDAlqSrWING6JKhG9KgSHhq052Imxw6dIi9OlmXrnrjjTeowR6F2uyGbNBszHZaNnoO2dPIUJueT9agS+XsUIM9pfR2zJ5hGpGmkk2BQMuzZahNc8rex2n6WIMu6bb0/s5uTguwtwb2spHuHFSnztOqnB5lW8wcm2zB/1Ui0HyLuWcjrEEvI5bw7Fp6lSirk/JBYV7s6aJLmiDRFZSzI64S7+BhTiVNEN2EJou9lZCADyo9tHTnoLo5T7epSC8FS7PPiVcjpEx4MAP7JJWZ3bp1i7cADIWE10NTUxNvARgKCa8H+5xZHSwOCQ8QVGPbow1Hv1+0Nv/g19sbz63xtspnEQ4R7saV1ZXrMvYcWLrxeFFN09jE3N8RNggSXg+2OqG+fbU+6F+8Lv/htXDPGR51FJ3Yknn4e/6o+kLCg9N9vKts+GaylJO6xfbDf3IP6veTSkh4PeB0+ua0p/yilH5GRW5h4K+lqA4Jr4eKigreAjOhw2wp8QyMhWvy+WppCQmvByS8OS3JPChlnVExeSfp7bSjfLW0hITXQ3x8PG+BqbR92HJxdcW3G5S5p38sXnuILpHwABpTZN2F6oz3t+yL6D/eYgn/h0PC2wcqvEkpMlCKxnNraIc//auv6s5kSFdFGsM3k/OLNy/67NDOI9m06y5dKwIJD6Axv6wzMJDw9hEXF8dbYAJryu55pqZ9Lb+sMzCQ8PaB79KbQe75nq4BD+8wfllnYCDh7QNfvDFQze3B2tYh3pH4ZZ2BgYS3DzudzMMqXP0TeZce8U4wflkXVogbesdnxppfj7OI9m6R8AARezbpzTzlO51eWPyyLmh47r9uixsqB5VdsYBohBFIePvQ/0M7evWYP/i6qiS5uI23wueXdUGD6S31XT7Y4cttQuMiyana0zgFG2ehbM8Xqj8hASHh7ekPebdNHmq9vlNL2/lH7lHwy7rAMXR+ZrLXt/dOCU9dym02rryHYMQC8wUS3j70r/BSdpkwYnx97znb82joOe9EzS/rgoZIeEJlnL0F0Li4E9aggk/XsuWV14YRSHj7qK6u5i29SNllwoju9V3VMnD13gjvxM4v68INls8U9BZAyS/GYwgkvH1kZ2fzll6k7DJhRPT6bnU/K7zi5h0V+WWdgYGEtw/9/x9eyi4TRjiv71GPd32Fi3e04Jd1BgYSHqInZZcJI8Tr2zv9MqmolXc05Zd1BgYS3j5Q4f0j4Os7tbR9yvuSd3Tgl3UGBhIeoidlV4gQC0d0q4Dx9VW3e3hSGgwW7PX9L/l36HLnjw/6RqeoobPbl1ZJWWdgvP/FN3y1tISE14P+p6mWsitE0MKUpZSrrFtze5DdA2EjY8+91KZlrneNDo7ztKS2/8KRJrxo07E6vxd9jU1MXqiO9c/dVYnF63T68yokvB50+6mpWw/Hk4vbnk16RS7NG3QrcckaAnsXoAa7ipKc5bkYnF2Ko4WjqPBmMPF8aunneVIG6hON59bkfHOWr4cukPB60PQkloVX3Icvyn8lImVXiBAL//JonIIalLQ02NE7wcYnX0yzxuWOYQrWFrdSLrzvXE/3oIeNzxvmSXil1gf972Tkl5d/LmWmWtF3/Y8pXx7Y/vUZ/ni6Q8LrQd2/h6cCTmW8sWuU9wORssuEYc6ED8j1+OmesguJX35D7wUbcvfR20FT7WoKd+NKdqJrb6vvh6W6G1JosO5MRm5BdmJ23qK1+ZmHKqrqdfnvhrAh4a2B0juiT7Cl7DJhWCjh7QQJr4fa2lreigTtqBfVR/n1Mim7TBhIeEMg4U1k1ONNKmq983ic92MgZZcJAwlvCCS8HkJU+IttQ5mn7nunVf62iZRdJgwkvCGQ8HoYG5vz86C7ah58e72Pd7QhZZcJAwlvCCS8HlwPe2lfXT5lqpak7DJhIOENgYTXSs3twc3fd7J2Zydv6IbSyfzB1xV0hIRXU/YPXdW3Xn/bFMBskPAxeTIymXi8lS55H8Dc1E/4JakHZxYsMG0s/WQ/X9FoVbUM7PzxAe8AWIqaCZ+4sUjKLtPGu5kRfNfVO/1yfYXrYluQXy8Jj/6H8QD+VEv4ojPXpaQyeeT/5Spf9UC6BjxJRa3DEy94P2Yej34f0QMEo1rC+z509UsqM4f/p8TfXu/LPd/DO2pzuzU4ByNAhByd8LSjzv5PmG+DlqL7Oj2AulDhARwECa8T/EQ8mAESHsBBkPA6QYUHMzBZwvfMfkguDWoTOic8/h8ezMBMCU/o8ne/mzl71tdISfGNUJeCXdXePjMw4GsUFfFGDIEKDw5kmoSn2i7alMy/+Y0v4SnD2QihS+rSoHIkhtA54fU/NT2APzNVeCmUCc9CmfAxh84JD2AGJk54jUPnhNf/5+UA/CHhARwECa+T+Ph43gIwDhJeJ7r9vBxACEj4mbxLj7T7IzkhNTWVtwCMo1rCv5dZIGWUyWPppwEq/LNJ75qye8pfQVZLXFwcbwEYR7WEJ61/FycllWnj4b/793ylQ2p1P/ug8K6ep5cG0JSaCU/eT9ohpZYJIyVhE1/dCFU296+vcIX/i45KqPBgBionPFP9U2vix7sa//4/jf3q30rJZkhM/NWvmn79Dx8nfVle28JXUQ07f3wQ9Y89AhhCk4R3oMHxqaSi1gbXCO/7WUBvPbp42+/3HkwYfF1Bd0h4TVDmJxe30bsA78/M1NXV8ZbGxG85mTaQ8AZCwuuB9vz/OfsvvKMxKbtMGEh4AyHhdSI+tJvy+s5yX9ncz7qqk7LLhIGENxAS3mBdA54PCu+2up/xfsyk7DJhIOENhITXSZjfpa+5Pbim7J5napr3IydllwkDCW8gJLyp7Tnbk3fpEe+ER8ouEwYS3kBIeJ3E/tdyox7vypL22tZ5fuJOyi4TBhLeQEh4ndy9e5e3VHKzZyzxeOujoee8/4qUXSFCLBzRrQLG11fd7uFJaTBYIOENhITXSXZ2Nm9pg+VS9g9dIq/mDboVZSnlKusq/2SIjYw991KblrneNSq+U0Bt/4WR8FaBhNeJbiexlLIrRLCFxU1mb82xdwFqsKsoyVmei8HZpThaONKEv3pvpPCKZb6VPDzuKfrxWtreP/9+dX7aVwcLv9lUXbmuqXY1hbtxJYuH11Koe7XmU7oq58iuFRvzVmwqyCn6oa6li9+LOSDh7UbKrhAhFv7l0TgFNShpabCjd4KNT76YZo3LHcMUrC1upVx437me7kEPG583lBW+b3Qq89R93jGBh/0jS7MK84u3zLR9qEV0XElNzD6y88RZ/ni6Q8LrxIQV3qgIuEvvnX6ZXNxGl7yvI+/09OLMfCkz9Ylzp9ce/O4KXw9dIOHtRsouE8a8x/BU86ny846WxiYmL1RnSEloSCxZr9PvlCDhdaLbL89I2WXCCP9DOzrOp6N93tEAHXVLiWdgvP/FN3y1tISE18nY2BhvaUzKLhNG+AkvNLhGjl1+zDvqOfK1VsfqUUQUT0sUkPA6KSsr4y2NSdllwojllU27+hnl93gndm0fLvrskDLrDImWi6spkPC2Ultby1sak7LLhKHKK9s7/TKpqDXWD/lepRzt26fk7BZdfaLx3Jp3N+aKLhIeoiFllwlD9Vf2+gpXlB/yKdJPxOSdpJ1HshO37e1uSJGuijGKSja+8+mRujOBPyZEwtuKbr8PT68b8wdfV7UV1bsvdwzzTjj8si50TPyS1HB2TX7x5i37c9J27F7x+f6F6UfYFlGb3iNoMOdQduWp9a76T6TbzhvaPS1KSHiwocau0bD+ytAv6wwMJLyt6FbhQWlwfGpNWfAP+fyyzsBAwgOoRvlNvottr/7E2C/rDAwkPID6Vpa0s88OfR2/rAsrhs7zxoMdrwdFRHu3SHgAjfllXdCgJO8tnRlr5jdUokG2jOe+75K6YoSwRhiBhAfQmF/WBY3JXt5gWU2847zBxtnbAWuwEQpxbRiBhLcbfG5nOn5ZFzgIJTzlM10y1GbYAqKkxxBIeACN+WVd0GAJT9WbCjs7hmeXovKze6P6zw7s2XgkD4GEtxvdvl0L4fLLuqDBEphVeMp5cVvKcHoXCE25kx88kPB24/Hgd+ZNxi/rDAwkvN3U19fzFpiEX9YZGEh4u9HtHBgQLr+sMzCQ8AAa88s6AwMJbzeo8GbjblwpZZ2B8ftV+Xy1tISEB+fKKa6Rss6oyP9G298pEZDw+tHtTNUQkcQvS8dakqUM1C125e90PX7KV0V7SHj9DA3N8zuQYKyGuz1LPz/ad/2PUk6qHpWn1qf8SadzHEqQ8PpJT0/nLbCIupau9P3f0rvAyZNZrZfTpLwNHbTXUHcmY+P+ve9kHDv43VX3oE6nLQ4NCa+fhIQE3gIwCBIewEGQ8PqJj4/nLQCDIOEBHAQJD+AgSHgAB0HCAzgIEh7AQZDwAA6ChAdwECQ8gIMg4QEcBAkP4CBIeAAHQcIDOAgSHsBBkPAADoKEB3AQJDyAgyDhARwECQ/gIEh4AAdBwgM4CBJec/jBGTAPJLzmFizAkwxmgdei5lDhwTyQ8AAOgoTXXFxcHG8BGE2FhD80i3diOGR988036banT5/mfbswW8K/8cYbrEFP9cKFC1k7atJ0Nzc30whN5cDAAB8CM1Eh4XNm8U60CU+3qquro1cJvVb4EGiDTRA91dHNlES6E+rSPVPaK18SYB5aJTx7p1+2bBkbpAWo+9FHH7EutTMzM0WpIeyFwjuzVej+/ft0+dZbb7GREydO0DLKtwO2R0BvE9SmS2qLhU0lLy+Pt8yBnihxybDnVkwWtcXs0C4AjdMITSi71n8qWYORunQnbHl6CDbC7k3cXNojkO4cVDdneqJDk0R459WUs0s2zXQtezHRRLKdf7qWBimlqc2wgiPymdpsb5NuwqafdWkvlC1DC7A9ArpnumSvThphDVMx229I0lNHz5J48ulJY2+UNDVsmpSzQ22aRGpTg7oBp5IuBfbewaaMsJuzBl1SerMuTaK4f5o+Wgfif+egujmzFR2aJ8I7r6aWSgQ1aBbZiMBeW9TwLeqHlmdXKRdgbUp1eplSm3XZJUOPzsYZPmoaBQUFvGUO7CkSTxS9k84+bZzyKiLa4iohxFSKHQRxLY2wFwMl8+ytfa8NQuNsAcLGGXPurNlAgNmKFE2bqMzUVk4VtelNXVwr0Izylh92lViA6gC7Q1G6pQUIFQ1WNyAc7KkTs0YZKKo9o3xuRZs1wp9KNi6upRuy3TFWuuk9mlZATC7jf+egusCzFSlRewkb4Z3ZLttdZwLuBxJ2LWGvAGqI+6SbKxcg1KXFWFssz4j3BfMw4S49a9BTJ6aDEbv0s9f7iDZrhJ5KymF2FWFvwbwzi7o0yNo0TbSwcgHag/O/c1Dd69kyFZpy3rI+syW8nuw0j/aAhAcNYR7NBvOhubKyMt4CMBoSXnMJCQm8BWA0JLzmKioqeAvAaEh4AAdBwmsOu/RgHkh4zSHhbWNwdOJCs2tP+cWc4pq0feUrNhctzix4O+3oik3HU/705/T9FTR+8nzL7a5efgPzQcIDBDbxfIqyd2lWYfruA1V/WffwWspM24fhR1Pt6iNF2xetzd9w9PvGtkf8To2GhNdcbW0tb4EVtLieLN14PLdg++SdJCmHY4nbl1a9uykv99Rl7/Q0fyQjIOE1h3PaWcWGo6fzi7dKiap60J7CkvX5TR2P+aPqCwmvOVR480vaebK6cp2UmZqGt/XDxOw8/Y/2kfDgaIOjExty90jZqFuMtSQn7vgzXxVdIOE1Z7a/hwehu3fowpm1UhLqH4vW6vcKQcJrDglvWjmHd0q5Z1RsLzrL10ljSHhwqMLqBinrDIzCkq18tTSGhNdcU1MTb4GZLF5XIGWdgUEH83y1NIaE1xx26c3p7bSjUtYZGN0NKXy1NIaE1xwqvDlRwk/8ouZXa2KJFZ/v56ulMSQ8OBQl/OK1h6TEMyRaLq7uu/5HvloaQ8JrDl+8MSe2S294zldXrqOE97V1gYTXHBLenMQx/J78bVdrPmVtnYP25F8fVugCCQ8OpfzQzvdF1217dft2beCH0wUSXnNut5u3wEyUCS9i+4GclJzdkf4lbPhRXv75os8ONdXO7sNLoQskvOawS29OAROeBVXgnUeyKTPzizcPNn8sXRtp0PFC5p92Lsk82HhujXTVnNAFEl5zd+/e5S0wkxAJL0VN1doNe3csXnuI9sNzj22tO5NBJbq7IcXduJLeGmgBalDQIAXVcFqY3ixo4T3521z1nyjvKlToAgkPDhV+wusUukDCa66zs5O3wDRWlrS/nZYvp5yxoQskvOaQ8KbSNeA5fNF3hjlUeACb21rVNTg+xdpIeADbejbpzTw152fwkfAA9lTy05NbD8d55xUkPIANJRW18tZcSHgAW7nRPVpxo593/CDhAexj1ckOz1Son3yIMuG94zND53mbiHEWY82vr400dIGEB7vpG53Kqe7mneDCTXiP4qM+ymcaIeJSREDKBeYNXSDhwVb2nO3pefqcd0KKrMIT1mA5rxyRur2lvMhP9vKRMEMXSHg9jI2N8RZoZsr7cmVJO++EIYKEpxymYG3CGrRvzxr0FhAaW2ze0AUSXg86f9mOXsrmD76uKqm+NXi5Y5h3wuNbBynlgoVAbcp8QpcPdvARsQy7FLsAqPCO1d8f9LNiLdBL+Q95t00efF3VkFzc5p1+yTthiyzh2SVrUCazBrtkbwHUYMH2BRgxGE7oAglvQ85J+I7eiWOXo/wZ1nATnrC9d0KXnvt8nBKb5TYFHbQHJKp9OKELJLwe9N+ll7LLhMHXNQZZ37mGJ17wTuQiqPDKhGeXygYFJTz7oI4NsuVpnx8J70xIeP/g6xqVUY/8xfgoRJPwykN3KvUU1KU2Eh4MZO+EL6p333ksfzE+CpElvEBdwj6Zp4Rnx/OhKe8qROgCCa8Hnc9jaeOETy5u462YRZDw/iE+gSdiMMbQBRJeDzqfx9KWCd/gGqlqGeAdNcSU8FqELpDwetD5PJb2S/jU0vYpb8T/8RYaEh5swk4J/2RkclfNA95RFRIetKLzL0bbJuF3/viAEp531IaEB60g4f2Dr2sQtANPu/G8ow0kPNiE1RO+qmWgwTXCO5pBwoNWOjo6eEsXlk54Ff/jLTQkPGgFu/T+wddV4c7j8aJ6/b6wgIQHrdTV1fGWLoxKeHpoaSREsFUVMk/dH/V4eUcXSHiwifAT/nrXKAU1OnonKJRXRRH00NJIiKCFqaR3DXiGJ15kfeeaXXFdffzVCTnljIvuhhS+WhpDwushKyuLt3QRacLvO9czOD4lXRVF0ENLIyFCLExvNLNrrTf34NjEL0lS4hkVKzaqfEaQYJDwejB5wtNNxAi7B1Jze5B1J1/4zv2qvIoEW5iNhxMX24ZYQ7dP6fwtXlcgJZ4h0XJxdd+QCn8OFA4kvA0tzjkr8ip0ULb/8sj3UmPdiht9LHUp2CBdfnKiXTlCMfbcS5cBF2bdcOLsnaex/DW7WgzP+erKdS2uJ3xttIeE10NTUxNvaSynuvvb632RVniK7kEP6/I7mkUj7JKFaLuHJ+ky9MLzBruVGewpu3C15lMpD/UJ2pOfeM5/3FIfSHg9aL1LP+V9uepkR+OrDIziQzvab6cj+c3fd1KDuqJB98aWVLZZwodeeN7wrahpeKenE78spWIrJaRG4W39MDH7cPU1bb9KGBASXg8VFRW8pbYnI5MfFN6VzsQefsJf7himYG26IV3Sjjo1aKed0pi6LJOVC1CwhKeQFhYLhBO0sAlt//pMyo6DD6+lSCmqVpSXf75obX5TR5Tn4YsdEt6qbnSPJhe3PZsM8H/X4Se8gcHX1ZSo4O8sPb/os6P5xZsHmz+WkjbSoOOFzD37l2wobGx7xB/AOEh4PSQkJPCWGqpvDW6t6uKdQJDw6qpp7NiQV7E481hidl5uQXbdmYym2tXdDSnuxpW0c04pTQ0KGqSgGr4hdx+9WSR++c2esguux0/5vZgDEl4PaiX84YuPwjkrMxIegkHCW0PWdy4q7LwzHyQ8BIOE18O//uu/8laE6BCdDtRvPYzsWxlIeAgGCa+H+Ph43gpb14Dng8K7faPR/CctEh6CQcLrITs7m7fC0OAaWXWyI5ZzNiLhIRgkvIl8e71PlRM2IuEhGCS8Hubdpd9ztqfkJ9W+UI2Eh2CQ8HoIlvDe6ZeZp+5fbBvifZUg4SEYJLwxhideJBW1uvo1+VNwJDwEg4TXQ1lZGW/NzLS6n1Gqa/qXoUh4CAYJr4e4uDi6rG0doh142o1ng9pBwkMwSHg9fPjlnw9f1O8PJ5DwEAwSXls7f3xQ2dzPO3pBwkMwSHhNSGekYLv0uqGEN3/wdQV9IeFV9mRkMvF4q/QTiDonPEAwSHjV3OwZW1nS7pnyneMVwJyQ8CqY94wUOv/UFEAwSPiYHLv8OJyfQ1uwAM8zmIImL8SkrOO5/yXh9n/4j95/82/oxW5s0DrQmhz8p39O/Pw4Xz81UEmvuR3uGSnUPcUVQNRUTviahtYL//v/IaWceaLx7/9T5YUWvq5RYWekuPNYp98JAVCXmglP2T783/21lGNmi4m/+lV0Od/z9HnUX4nFp/RgEmomvJlruzKafv0P3ukIPktv7BpdU3YvljNSIOHBJFRLeDpul/LKzPHe2mN8vUNS64wUACahWsLn/pcEKanMHIW/+698vYM4UPuwrLGXd2JWX1/PWwCGUi3hb/+H/ygllZnD/Td/y9d7LnZGissdw7yvEuzSg0molvBm+B+4yGLW2Tv8h0HYGSm6Bjysq67ly5fzFoChVEt4OZ3MHzMzdx6P/yHvtqt/YmVJe8AfaQOwGecm/JT3Jfs7zXC+KhejKM5LD6AF5yb8t9f7Dl98tPPHB6G/Bq8KJDyYhKN36QGcBgmvh6amJt4CMBQSXg/YpQeTQMLrITU1lbcADIWEB3AQJLwe8PfwYBJIeD0g4cEkzJTwv/udOvcTZgA4j5kSfmBgpr3dl/bSuEaho87OTt4CMJSZEv7ZM99lTw/vkpQU3+XNm68X+O4738i+fXwkltBRVlYWbwEYyjQJv3nzzLJlvoa4K1Jf72vQWwDLcCIabJlYQkc4TTWYhHqveymdIg0lSn42wq76zW98u/rKkbNnfcWftaMOAOcxU8KLNtu3FyNU1SnDlSN0tE/vAqwddegIu/RgEuZIeMpn5WE5uzclMS6wkVhCR0h4MAlzJHzA8L9PdR8FwHmQ8HpwuzU/xwZAOEyc8FqHjvApPZgEEl4P1dXVvAVgKCQ8gIMg4fVQW1vLWwCGQsLrAQkPJoGEB3AQJLwehoaGeAvAUM5N+LLG3qzvXN7p6H8EOnzYpQeTcHqFd/VPfFB4t6N3gnU1gtNUg0molvAW/TFJhup89g9dJT894X0Am1It4Vv/Lk7OKBNHsJ+Lrr41uKbsnmdqmvdVgjPegEmolvAH/+mfpaQycxT958V8vQPpG51KPN56o3uU92OGhAeTUC3h38sskJLKzLE0I5+vd0h7zvbkXXrEOwDWp94x/PR006//QcorcwYdfdDa8vUOw+WO4eTituGJF7wfOY/Hw1sAhlIt4UnlhZaJv/qVlF1mC1rDkh+u8TWOxKjHu7Kk/WJbNP+jjl16MAk1E55UnG82c52n2h5dtisdu/x4V80D3gkP/h4eTELlhCe0t7zis/zC3/1X3yfhfilnSNCaFP3nxXTcHtGefGg3ukcTj7c+GZnkfQArUD/hHWXK+zKj/F5VywDvA5gbEl4dpdd6t1Z16fNF3RDeTjtq/uDrCkZAwqupo3eC9vNd/dp+UTcESqc/5N02efB1BSMg4dVHdX7z951U83l/1osX0f+vXviQ8BAaEl5DdGxPR/h0nE9tff5nDgkPoSHhNfdo6Dnt519oiex/8qKDhIfQkPD62VXzoPCKtv8hj4SH0JDwOhG79LWtQytL2kc9XtZVFxIeQkPC60Q6hh+eeJFc3Hb13gjvqwQJD6Eh4Q2Wd+lR7vke3okZEh5CQ8LrZGxsjLcCaezyfVG3b3SK96OFhIfQkPA6Cec8ls8mvWvK7tXcHuT9yCHhITQkvE4iOo9lyU9Pcqq7o/iiLhIeQkPCm1er+9kHhXe7BiI4eQYSHkJDwusk6lPTT3lfZn3nqrjRz/shIeEhNCS8TmL/LYrK5v71FS72Rd1gkPAQGhLeYnqe+r6oe+vhOO/PZVTC00NLIyGCrSoYAgmvE9XPcrXzxwdF9a/vkw746TL8hL/eNUpBjY7eCQrlVVEEPbQ0EiJm1xeMgYTXSUFBAW+p6uydp6ml7c8mvZRIT0YmI034fed6BsenpKuiCFoTaSREsDUHQyDhdVJdXc1bGii91sty6e20fJFXoYMlPN1WjLC7IjW3B1l38oXvFIDKq0iwhdl4OMFuCIZAwtsBZRHV+a1VXf/vzgvK1AoRlO2/PPJ9EMC6FTf6WOpSsEG6/OREu3KEYuy5b1ci4MKsG07gFIAGQsLrJCsri7e0FOkuPUX3oId1+V3MohF2yUK03cOTdBl64XmDFl5Z4jsMmb21BdS1dOUU/fDuluMrNublHNlVXbnuas2nTbWruxtS3I0rWVCXgq4q/GZT2lcHf786P23vnwurf+obCvzxqlGQ8DoxZ8JTg/bb6Uh+8/ed1KCuaNC9sSWVbZbwoReeN3wrOnv+v50/6nFSkOjkn/5p4Zr8Pce+cNV/MtP2YSxRU7X245xD720rbnEZ//PESHhbCT/hL3cMU7A23ZAuaUedGrTTTmlMXZbJygUoWMJTSAuLBcIJWljIqe7W+sf5I5L77YX3tuR1XEmVklatKDqxafG6go6Hhh3UIOF1gnPaieDr+grt29MePu8YJ23fqfLyz6X81Cgm7yS9uymPjhT4Y+sICa8Ts+3SGxh8Xeeqahkw6sM8OtKmY28pJ/WJpZ/nT77Q9bMMJLxONPp/eIl1E55h3yngHV0szSqUklDnqPj2cz2P7ZHwtmL1hCd6fpi3dONxKf0MiQvVGcPjOv2gOBJeJwkJCbylJRskPEM5r/WHeX1D42MtyVLuGRVL1uv0C1xIeJ0g4UXwdZ2P1h/mffxVqZR1BkZ3QwpfLY0h4W3FTgnPVLUMVDaHdS6ASNFzJWWdgTHxSxJfLY0h4XXS36/Jq1Ziv4RntPhmnqkSvqRUj//EIUh4nWCXXgRf1wjRIX1OdTfvqIGeqz3526TEMyo27N3BV0tjSHidpKen85aWbJzwjIof5tFz1XJx9YXqDCn39I8lmQd9DV0g4W3F9glPPFPTqnyYR88VpdnDayk5h7JF7ukcdOj+3qZc3tUFEl4n8fHxvKUlJyQ8U31rMMYP81jCs3h3Y27dGb1LfdqO3VdrPn09ogskvE6Q8CL4uqohlg/zlAnPgjLw5MksaVD1GL6ZHPj9RRdIeFtxWsITV3+UH+b5JzwLqrqL1x4qKtkojccY3Q0pH3/xp+0HcibvJElX8dAFEl4nPT2q/WJkCA5MeGZXzQN2Gs/wBUt4EX3X/3jw+JZFnx3asHeH79uvNyP7Wl7HlVTaX1jx+X7acQjro0FdIOF1EhcXx1taohex+YOvq9o8U9PJxW28EwbfmkgpN19Qla76y7qcQ9mUw4nb9lIysy1amH6EJfaW/Tn5xZsbzq7xfZHG7+bzhC6Q8Dr57W9/y1ugpepbg+H/So+ccsaGLpDwYEPhfJiHhAcN6bNLD4KrfyL7h1CnlEHCg4aQ8IbYc7Yn2Id5SHgAGwr2YR4SHjSkz0ksIZia268/zBscn6JLJDxoaMECPNXGSy1t3/njA/ZdACQ8aAjH8GbwbPZXNynKGnujTPix5tdtItosHuyY8Y7Lg2GGLpDw4CA3uke/vd6352wPlfpwE95zn9+YsGwnbLxXcZKsgMS14YQukPA6wS692URW4Qm79KdcQNmY7OWNMEMXeBXqBAlvNpElPNVz1hB77IQ1qOyHxhabN3SBVyE4VAQJL7Kdguo2K92U58oaTti1YgQV3sn0+Ws5CF8ECS+IY3j2FkDEAuLDPPLg1Qnq2EiYoQskvE7wKb3ZRPahHTWEofO8ISq/chcg6tAFEl4n+Gs5s4mgwrPjdiIuKZRJLt4CJKLshxO6QMKDQ0WZ8OwjOtYVR+mU8BRiGbY87dgj4R1Ln3PaQfgiS3iGZThd9pb6QuQ5Eh4kSHizibjCUzDUUH5ENy+25LyhCyQ8OFQECa9P6AIJrxO3281bYA5IeNAQdunNBgkPGlq+fDlvgTkg4QEcBAkPGtLn56IhfEh40BAS3myQ8AAOgoQHDfX3x/TbxqA6JDxoCLv0ZrM4s0BOOePC91t0ukDC6yQ9PZ23wByOfF8vZZ2BUVK6ia+WxpDw4Fx7jn0pJZ5RsSH/NF8njSHhdZKVlcVbYBotridh/XK7xrFk/TG+QtpDwusECW9OD/tHco7skjJQt6BD9/e2l/BV0QUSHmDm3a3FdWf0LvVpXx24erubr4FekPA6GRoa4i0wq7R9p06ezJLSUvUYvpn87qa8upZQP2WtHSS8TgoKCngLzI2q7uLMgqITm6REjTG6G1I+zjm0vahm8oWXP5IRkPA6KSsr4y2wiL6h8YPfXV209tiGfXsvVGdQZZZyOHR0XEml/YUVG4+m7fv2QrOL36nRkPAA4eruHaqqb80prknbV574ZemKTcffTjtKsXBNPrVpcEthdf7pnxvu9kw89/0itQkh4XWCXXowAyS8TpDwYAZIeAAHQcLrxOPx8BaAcZDwOqmtreUtAOMg4XVSX1/PWwDGQcIDOAgSHsBBkPAADoKEB3AQJDyAgyDhARwECQ/gIEh4AAdBwgM4CBIewEGQ8AAOgoQHcBAkPICDIOEBHAQJD+AgSHgAB0HCAzgIEh7AQZDwAAAANoQCDwAAYEMo8AAAADaEAg8AAGBDKPAAAAA2hAIPAABgQyjwAAAANoQCDwAAYEMo8AAAADaEAg8AAGBDKPAAAAA2hAIPdrBgAV7JAABz4G0R7AAFHgBAgrdFAAAAG0KBBzvYvXv30NAQ7wAAAAo82ENcXFxnZyfvgHUcOnRowYIFdMn7AKAesxT4nJwcynO65H2FhQsX6vk/rOzhJPw6MCscwYcmvYybm5vfmEUNPmSQEIlP6urq2JorBVsYACQo8HOwNxS8g4DN+ArjqyT66KOPqH3ixAnWNVaIxCc679wD2IzFCvz9+/fpsIO6Al3LrhoYGHjrrbf46Cs0QuNsAfYQymWonLOrlNgCb775ZsB3QHaVtA7KIyH2kaNEuV3s7VVJbAI9Ih9SMMkbscnFx8fX1tbyDvhhryW2/ypeb0r+L0vlqzpY7lAjYDqIpAszKwMmPjl9+jS7Ca0e5T4ffYW9M/jfv0gZWn8+pKDcfPZJBr/iFbZp8645gPlZrMCzjA2YZjRI7zW88wotLPKZPURmZibrhkZJzh5XeQ+EjfDOLFoTGvF/aIG9TbA2PTq16W2LdZXYe5lUztk6B9wRASUU+NDoVUTYy2nZsmV89BX2spQqqDLvguUODRKxH0DYTqpI5DCzUiwfEKUYLcAqMV2K9GFrKGUHSzf/vQGGbako4dSm5dlVknnXHMD8zFLg2YFvwDynNKOreGcWZS8lKkt42rVng9QOKKK3En/0XqC8k9m7lJ80ZQlnj+KPXcvekgIeBAS7IUGBhxixFxI16LXHEkdZldnL0h8tyV6rwXKHLcY7s1i+iCXZAv6izkq2qiwjlG1BWcLZyvhTXuu/18KwJf2JNQcwP7lWGYgVcsof8dZDRwPszUh8yZbeCAjbPae3HjoWoWtZurLPGJUf5dE4LSDeO6hBC4huQLQ83QO7Q0Jrwoq3uBW1Ca0kW4Yeiy0g3iaoTess1pBuyDaBXcuOb2hL2c1pAdo0dkRFN2G3FQcoNMKeAdaFEAoKCvAt+hDopUV451UVFIfy7GVJr2SRetSgl64oZsFyx3enIQt8LFlJ60CZQusmsokdBlBGsN0OVuDZMmwB9nC0Ib7bv1pAJBQtxt5kRIKzLj0b7A7pEUVqz7vmAOZnogJPKJcoo1hFJJRsLHUF5QJ0SW2WmQzLcJa0/jdn7w5iXyEgdv+sZhO6K8pn5UOwcfZGwNqU8+ItgLB1YGtIN6eHo3ugLr967gJ0STcXbzeE3ozEPdPNaUnlnUMw+Ig+NHqlEd6ZRa8rGqGXmXh50wtVvPKpoXzlB8sd/7ulFzMtqayC7AUfXVbSkrQ8W08iJQvLFEoZun+2AK22cgFCa8Iemu6ECjktTG2xH0NoJNhWh15zAPMzV4E3P5bqvAOmgSN4B2IFXqroACCgVkVm9ogFn5kDGI8O6KnAKw/HAUAJBR7sICEhgQ7ieQcAAFDgwR5Q4AEAJCjwAAAANoQCD3ZQUVHR1NTEOwAAgAIP9oCP6AEAJCjwYAc4ggcw0PC4p66lq6imKae4Jm1f+btbvv796vwVG/PTdh3OydtbWLK1unIdxdWaT5tqV7PobkhxN66U4uG1FLEALcxuRTenO6G7ojv03e2m4/QQ9ECFZ67Tg/YNjfOVAD8o8AAAEJaOhwP5p396/4tvFq7JT/ri8J5jX9RUrXXVfzLT9qGBQbsFtBq5Bdkff3mYVuy9bcW5py63uJ7wlXYwFHiwg6xZvAMAanjYP0KVcunG4+9tOZpfvLXjSqpUWU0etMJFJ7YkZuctXle4s/QC7Z3wDXMMFHiwAxR4AFU03O1J23dqxcaj5eVZk3eSpJJp6aDNqazY8O6mvJQ95XUtXXyDbQ0FHgDA0bzT04XVDYsz86sr10lF0cZx7vTapVn5ByuvTr7w8ifCdlDgwQ5+/vnn77//nncAIDyux0+XZn1ddGKrVPwcFRWnNi1ZX2jL/7NHgQc7KCgoSEhI4B0AmM/YxOTSjV9fOLNWqnaODXoqlmw4Pjzu4U+QLaDAgx00NTVVVFTwDgCE1Dc0vnhd/lhLslTkHB4TvyQtWXfU9fgpf5qsDwUeAMBZUnaXNtWulsobgqK7IeX9L77hT5P1ocCDHeAjeoDwvZ12tO/6H6XahqCgg3h6cvjTZH0o8GAHKPAA4aMaduTrLVJtQ1CUlGahwAMAgFVRDcs5lL0nf5tU3hwe+cWbN+zdgQIPYC5ut7u6upp3ACAkXw1r+7Dl4upFnx26UJ2hLHLODHoqlmQepEtqo8ADmEttbW18fDzvAEBIrMCzeHgtJXHbXjqgd+CX6id+Sdp1dPt7m3KVp9NHgQcwl7t372ZnZ/MOAISkLPAimmpXv7sxNyVnd90Zmx/TN55bk7ZjN23s1ZpPpasoUOABAMCqAhZ4ZTSc9ZXApRsOnDyZZYPv2w/fTK48tT7M3RcUeABzwUf0AOGbt8ArY/JOEh3p7jySvXjtocRte4tKNnY3pEjLmCpo9WglP/7iT+98emT7gRyq6BH9ag4KPIC5oMADhC+iAh8w6LCeCufB41vSv/pq0WeH6Fh/w94ddLh/oTqj9XIaHTFLy6sYYy3JHVdS6dHp4Tbu27Hi8/1UyNN27KaVoUd3N66Ulo80UOABAMCqYi/w88bEL0l0JN1wdk3VX9blF2/OOZS9ZX8OlWGKxG17qSpTLEw/QmsigrpsnBZgS9JN6IZ088pT6+muXPWf0N1KD6R6+J4cu0CBBzsYGxsrKyvjHQAIYtTjPVD78J/Sj0tVDSECBR7AXDo7O+Pi4ngHAOaa8r4svOJOLW3v6J2grq+G+RU2BAsUeABz6e/vT09P5x0AmOWdflnZ3J9U1NrgGuFDs1DgQwQKPAAAmNfVeyPJxW1VLQO8PxcKfIhAgQcwnQUL8GIGp7vzeDy1tL2o3j3lfcmHAkGBDxEo8ACmgwIPjvVo6Hnmqft7zvaMerx8KCQU+BCBAg8AAAYbnnix88cHWd+5+kan+FB4UOBDBAo8gOkUFBTwFoCteaamj11+vOpkB/tKfBR0LfBjzTND5+XBBzt86yEN+gdbzDsuj2sZKPAApoOP6MHevNMvK270Jxe3NXaN8qFoqVzgPff5/UqotLMFJnt93d7SOcuLrhThk26oUqDAA5hOQkICbwHYy+WO4aSi1prbg7wfM62O4OlInbDKTdU9TGI/gAWhPQDRZfcjLcP2GJQj6gUKPAAAaOvWw/GVJe0lPz2hY3c+pBKtCrx3fE5tDjhI5Z9Ih+/h7w0EpLyrmAMFHsB04uLiOjs7eQfAsnqe8q/EP5sM6yvxUdCkwFMVpwPrYIPimJvqPTscD3gUzsq/OF6nZQJ+kh/wtioFCjyA6di+wNP7zh/ybiNiDNO+fQ+OT+VUd2/+vjPSr8RHQf0C738IToWZBtn348RH91Tv2dE8K+RS8WZEdWfB7pluxb5wJ2j2zTsUeADQGwq8KmG2t2/P1HTepUdryu65+qP8SnwUVC7wVH0ZdrDOSjI7yBaoxks7AazSi6BrpRGDAgUewHTKysrGxsZ4x45Q4FUJk7x9e6dffnu9L7m47UZ3rF+Jj4LKBZ4FHVIrC7w4Oic0wtosqJAHPP5mB/rhk+5WpUCBBzAdfESPCCcMf/uubR2iuq7iV+KjoF+Bp0HqssrNFhNtWpgtL4JdRZesyz4YeLCDd8X9U7CP61Hg54MCDzaRnp7e39/PO3aEAq9KGPX2fbNnLLW0vfRar+pfiY+CVgVeiQo81WPxqTsr58r/epfKuf8ICnzMUOABrAEFXpXQ+e27a8CTUX4v97yGX4mPglYFXhRgCmoLdBWNEKkksxLOqn4s2B6DSoECD2A68fHxtbW1vGNHKPCqhD5v34PjU9k/dG2t6qIGHzITTQq8XQIFHsB0UOAR4YSmb9+eqekDtQ/XlN2jA3c+ZEoo8CECBR4A9IYCr0po8fbtnX5Z1ti7sqTdkK/ERwEFPkSgwAOYTl1dXUdHB+/YEQq8KqHu2/fZO0+Ti9vokvctAgU+RKDAA5hOQkKCvX8xFgVelVDl7ZuO1Ol4nY7azfCV+CigwIcIFHgA08nLy6ODeN6xIxR4VSKWt++uAc+asnuHLz7yTE3zIWtCgQ8RKPAAoDcUeFUiirfvwfGprVVd2T+Y9CvxUUCBDxEo8ACmg4/oEeFE+G/fzya9ued7MsrN/pX4KKDAhwgUeADTQYFHhBPzvn17p1+WXutNLW2/2WPbnzZAgQ8RKPAAoDctCvzljmG6Z7pUDrIPor++6lYO6hz06LQO7uFJaTz2UL59d/TO+QG3mtuDycVtta1DvG9fi9Ye67v+R6mwISgmfkn6/ap8/jRZHwo82MTPP//c1NTEO3akRYG/3uX7u226ZF32CyhU4MUCRoUOBX7z953UXVN2r8E1QnX92+t9Fv1KfBSOfH/1SNF2qbYhKEpKN+0pv8ifJutDgQebyJrFO3akaYH/5ET72HPv5Ivpfed6pGWo1nYP8v+EptpfcaNPeS3dhMow3YpuTgvQMTENsg8G6IbUYOO0mNiNEBHinjUt8FTIqa4rB61yghoV5RTX7Dn2hVTeHB7532zbcPQ0f4JsAQUebKJiFu/YkXYFnqovXbLarAyq+nQVBR3ZsxGqu2xELDO7ar7yzPYM2CW7W0L1mx0osxEq9uxW896zdgX+/1x1jI7XWTvxeOv6CteB2oeVzf09T5/PrrKDtLieLFp77EJ1hlTnHBgtF1cvWX+MnhD+1NgFCjyAqXUNeKgCfVB4d3HOWVGl1ApWd+mSFVQqwFR6pWsDEkfbrCtuwkLcrXKQqrVYct571vQInj0QMA/7RxJzSnIO7RhrSZbKnu1j4pekXflfvre9xPXYYuciDBMKPNiEbT6i906/vHpvJPPU/VUnO+gAd8rL/2NY04/oWZfVYPGdO/Zf8uIgO2D41izyAj/vPaPA66+p4/G7W4tTdhyoO2PzY/rGc2vSvjpAG3v1djffeJtCgQebsHSBH5548e31vsTjrTt/fHDn8TgfnUuHAk9BB9A0Ig7l2QIS/4/oRZfFvAWeIvQ9o8Abq+FuT9q+U0s/P3ryZJYNvm8/fDO58tT6dzflpfyprK6li2+kA6DAAxjD1T+xq+ZBUlFryU9PqMDz0eC0KPAODBT4SE2+8NKR7s4TZxdnFiRuP1x0YlN3Q4pUQU0VtHq0kh/nHHon49j2ohqq6LQJfGMcBgUebKKjo8Pk56L3Tr+82DaUeep+Rvk9aojP3sOEAq9KoMCrom9onArnwcqr6blliz7Lp2P9Dft20eH+heqM1stpdMQsFV0VY6wlueNKat2ZDHq4jft3r9iY905Gftq+MlqZC80u96BtT08UBRR4sImCgoKEhATeMQ06NC+91kuH6XSwLp1WJVIo8KoECrw+Jp5PdfcONdztqapvza+6mlN0ekvBd2n7ytP2/jkxp3jFpuMrNhUuTM+n6RBBXRqkq2gBWowWppvkFP1AN6+8cpfuyvX4Kd0tfwAIAwo82AQdvufl5fGOoVrdz6icJx5vLWvsDeez9zDRO6BUqxBRBD2N/AkFsDsUeIBYTXlf1rYOrTrZkXnq/tV7IxqdEA0FXpVAgQfnQIEHm6itrY2Pj+cd7Q2OTxXVu+kw/UDtQ31+bQwFXpVAgQfnQIEHm9ChwN95PJ5T3Z1c3FZxo//ZpN7fy0WBVyVQ4ME5UOABgpryvqy+NbjqZEfWd67GQH+3rScUeFUCBR6cAwUebMLtdldXV/NODJ6MTBZe8X32fvjio0dDJjo/OQq8KoECD86BAg82EctH9Dd7xrZWdSUXt9Hxuv6fvYcJBV6VQIEH50CBB5u4e/dudnY278zHMzVNtXxlSTvVdav8VCgKvCqBAg/OgQIPTvFkZDLv0qPE462FV9zU5qPWgQKvSqDAg3OgwINNdHZ2xsXF8c4rjV2jm7/vXHWyg47XIz01rNmgwKsSKPDgHCjwYBOswD+b9FY29ycXt+VUdwf7WTaLQoFXJVDgwTlQ4MHyugY8hy/6PnsvqncPjtv2VNUo8KoECjw4Bwo8WI93+mWDa2R9hWvVyY7aVv6zbGNjY2VlZWwBW0KBVyVQ4ME5UODBGkY93m+v9yUXt+2qedDqfsZHFQL+HzwAgGOhwIN5dQ14cs/3JBW1ll6b/2fZ+vv709PTeQcAwPFQ4MFEvNMvL3cMZ566T3GxbUijn2UDAHACFHgwGB2alzX20mE6HazH+LNsCxbg9QwAwFnsDfFh/0jRmevpX5x4f33B4vSjb6cheCxanfdeZgE9M0U//EzPEn++zKqjd2JXzYPk4rZvr/eNelQ7NSwKPACAYIE3xNtdvYkbi9Le39z063+gt3BEONH6d3Fp/9+mxA2FjW2P+PNoqCnvy4ttQ2vK7q2vcDW4RvDZOwCA1kxd4MvP/Pxe8u7u//F/kaoXIvzo++s33k/aceTkBf6c6mh44kXJT0+Sinw/y9bzVI+fZTtx4oTHE9OH/AAAtmHSAk9H7UtSD6K0qxVU5pd+sr/hbg9/fjVz5/E4++y9qmVA/59li4uL6+zs5B0AAGczY4Gva+lKe3+zVKIQsUfmHzKrf2rlz7JKprwvz955uupkx9aqrsYug3+WLT09vb+/n3cAAJzNdAX+Yf/Iik9ypcqEUCsSP97V8XCAP9fR6hudKqp3Jx5vPXb5sRV/lg0AwAlMV+DTvzjREPePUllCqBWNf/+fPt5UxJ/rSNx6OJ79Q1dqabuZf5YNH9EDAAimK/CL04+6/+ZvpbKEUCvGfvVv3w7vXNxUxamWU0Wnuk7VnY+aGwo8AIBgugLvKz9+ZQmhYigLfM3twT/k3e7onWDdJyOTxy4/Zj/L1jdq259lAwBwAhR4xwUr8IPjU8nFbeIntlad7Dh756lpP3sPU1NT09DQEO8AADgbCrzjgp7hwxcfidLOIuHYHf3/qk118fHxtbW1vAMA4Gwo8I4LdgRvS6mpqXQQzzsAAM6GAu+4sHGBBwAAAQXecWHjAh8fH19QUMA7AADOhgLvuECBBwBwAhR4x4WNCzwAAAgo8I4LGxf4u3fvut1u3gEAcDYUeMeFjQt8QkICPqIHAGBQ4B0XNi7w2dnZ1dXVvAMA4Gwo8EHiu+/4Ct28KV9l8bBxgQcAAAEFPlAsW+ZblX37fEGKiuQFrBw2LvBZs3gHAMDZUOADxbNnM/X1vH32rG+1qOSLa1NSfCM9Pb5LJRr5zW/4Mu3tfERCewzifgwKFHgAACdAgfeLmzd9hVk5QtV6YOB1lxV4GhQjFJs3+wbFbgEr8LSkWICC1fvf/W7OoO5h4wIPAAACCvzcKCri6+FPFO+ABZ6O3ZWDAQs8+zBAGtQ9bFzgO2fxDgCAs6HAK4KOrUnAb9Wx2sw+YGcFnlDJZ4fjNPLsmW+EjuPZ8qzA03E/uwmVf1qYSJ8NGBE2LvD4iB4AQECBVwRVZarT4v/RpWAlnBriCJ7KOZVwQlfRHoDyhqzA0wKsrhMq7UYfu7OwcYEvmMU7AADOhgIfeYgCL40rgxV4c1R0KWxc4AEAQECBjzzYH9GF/vt4upYov3tvmrD3EXxCQgLvAAA4Gwq84wIFHgDACVDgHRc2LvAAACCgwDsubFzgh4aGmpqaeAcAwNlQ4B0XNi7wtbW18fHxvAMA4Gwo8I4LGxd4OnxPTU3lHQAAZ0OBd1zYuMADAICAAu+4+L83f5936ZGrf4I/4zaCj+gBAAQUeMcFO4L3Tr+8em8k6ztXcnFbWWPv8MQL9vxbGgo8AICAAu+4CPgR/ajHW9ncn1rannnqfm3r0JT3Jb8CAACsyXQFftHqPPff/K1UkxBqxcRf/SpggZf0PH1eeMWdeLx1V82Dmz1jfNT0PB5PvTj5PwCAs5muwKdtL2mI+0epLCHUiqZf/0NS1nH+XIeNavzOHx9Qvaeq/2joOR81n87Ozri4ON4BAHA20xX4jocD76bM/sQqQoP4OOnLFtcT/lxHZcr78uydp5mn7qeWtle1DDyb9PIrTMDtdi9fvpx3AACczXQFnlRfa09PyJIqEyL22Lh8TXltC3+WVTI4PlXW2Jtc3Lb5+86r90a80/jPewAAUzBjgSeNbY+WfrK/76/fkEoUIroY/O//hxUr91693c2fX824+icO1D5MPN6ae76no1fvv8TDR/QAAIJJCzxz8MT5xP/2Jcp8LEGlPSVhU27xWf6c6oiO5htcI1urupKKWkuv9dKxPr9CMyjwAACCqQs8Q8ed72YWbPjDZ67/6X+VqhciWDz8d/9+4/I172Yc1eGoPUzPJr3VtwZXnezIKL939s5Tz9Q0vwIAADRggQKv1OJ6kvvnSx9vKlqRcfTtNMScWPrp0aSs4/T8xPg1On08GZksqvf9JV5OdfeN7lE+GrPa2lreAgBwNosVeLCrWw/H95zt+aDw7rHLj3ueRv+XeAsW4CUNAOCDd0MwnSnvy8sdw+srXCtL2iub+yM6ja7NTlX7dtrRP+TdRsQY9DTyJxTASVDgweyowFfc6E8ubqOST4XfUafRRYFXJVDgwZlQ4MFiugY8xy4//qDw7p6zPa3uZ3z0lbi4uM7OTt6xPhR4VQIFHpwJBR6s7Ub3aE51d+Lx1pKfnjwZmUSBR/gHCjw4Ewo82Idnarrm9uCasnurTnZU3xo01Wl0o4MCr0qgwIMzocCDrdTX13s8HtbuG50qvdabVNSa/UNXY9eoFU+jiwKvSqDAgzOhwIOthPiIvqN3Ive87y/xDl985OrX+zS60UGBVyVQ4MGZUODBVpYvX+52u3knODqav3pvJOs7V3JxW1ljb0R/iacnFHhVAgUenAkFHmBm1OOtahlILW3PPHW/tnXIPH+JhwKvSqDAgzOhwIOtxMfHx3622kdDzwuv+E6ju/PHBzd7xvioEVDgVQkUeHAmFHiwFVUKvIRqPFV6qvdU9an281FdoMCrEijw4Ewo8AARmPK+rG0dyjx1P7W0vaplYNSj7V/iocCrEijw4Ewo8GArTU1NQ0NDvKO94YkXZY29ycVtWd+5rt4bUf0v8VDgVQkUeHAmFHiwFS0+og+fq3/i8MVHHxTezT3f09Grwl/iocCrEijw4Ewo8GArqampdBDPO4aio/kG10j2D11JRa2l13r7Rqf4FZFAgVclUODBmVDgAfTwbNJbfWtw1cmOjPJ7NbcHPVPT/IqQtCjwlzuG6Z7pUjk4OO7b//j6qls5qHPQo9M6uIcnpfHYAwUenAkFHmwlISGhoKCAd0zsychkyU9PEo+35lR33+ge5aOzp9v7l/w74lfytCjw17t8D0eXrEt7G9SlAi8WMCpQ4AHUhQIPtmKVAi+583h8z1nfaXQTjt1hNWlN2T06yte0wH9yon3suXfyxfS+cz3SMlRruwf5Kf2p9lfc6FNeSzehMky3opvTArRTQoPsgwG6ITXYOC0mdiNEhLhnFHgAdaHAA5iCd/rlypJ2qTIt/vK8NBJ7sAJP1ZcuWW1WBlV9uoqCjuzZCNVdNiKWmV1fX3lmewbskt0tofq9+ftOMULFnt1q3ntGgQdQFwo82EpHR0dPTw/vWJ+mR/CsoFIBptIrXRuQONpmXXETFuJulYNUrcWS896zdgX+958W5VR3Z566r8qfNkCYBkcnbnf1Xmh2nTx/c8+fz+cU1+R8/X16bnnavvKU3SUrNh2nWLyugF7kFNRgI3QVLUCL0cJ0E7oh3fzcjft0V31D4/yuITwo8GArWbN4x/o0LfCsy2qw+M4d+y95cZAdMHxrFnmBn/eedTiCfzbpPXb58cqS9gbXCBuB6Ew8n2pse0Sld8PRyqVZhYs+O5q+O/fg8S1Vf1nXeG7Nw2sp3tYPZ9o0CXfjyqba1dWV6458vSXjT/vooWkFaDWKam7QKo1N+F5ywKDAg60UzOId69OhwFPQATSNiEN5toDE/yN60WUxb4GnCH3Pen5E751+Wdncn1TUWtUyoPrpieynxfUk99tLSzcef3dTXm7B9paLqyfvJEl11yRBOxa3L63KL95Kq0qFn1a7qeOxdzqsP1qxHxR4APPSosA7MPwLvNLVeyN0TF94xU3H93zI8Vof9G/Ir1q8Lj+/eAsdjktF1HJBm1B0YtOS9UczD1dSvecb6QAo8GAr+Ige4R+hC7zQ0TuRUX5v548PojsrkdV1PBxI2nkyMftIdeU67T5gNzxo0y5UZyRmH07ccfJ2Vy/feJtCgQdbQYFH+EeYBV6gAk9l3iFfyhscnfh4d9mG3D3DN5OlWmj7GGtJ3n5od+KXpe5BI38VWjso8ADmhQKvSkRa4IVnk97CK+7UUnt+Ka+7d2hpViEdzkplz4FBT8KitcdaXE/4U2MXKPBgK2NjY/39/bxjfSjwqkTUBV7wTr+sahlILm6zzZfy9pTV5hzaIdU5h0duwRfbi2r4E2QLKPBgKwUFBQkJCbxjfSjwqkTsBV6JjubpmN7SX8orrG44+PV2qbwhKApLthysvMqfJutDgQdbKSsrS09P5x3rQ4FXJdQt8EJH70Tmqfs51d2W+1Le4swCd+NKqbYhKCZ+SVqYrsmrxRAo8ADmhQKvSmhU4AX2pbyM8ntW+VIePSG3L62SahuCorshRetXi55Q4MFW8BE9wj90e8tmX8pbWdJ+9Z6pv5RHT8iSzIN0tCqVN4fH5J2kFZ/vR4EHMCkUeIR/6P+Wzb6Ul1TUWtncb8Iv5dETMtaSvHjtIXyFXkTLxdX0hPRd/yMKPADoAQVelTD2LbvB5TtT3rHLj83zpTzfE6KoahXfbhB1zoFRXbmOngR6KlgXBR7ApDwej9vtO6W5PaDAqxImect29U+sr3CZ4Ut5osCzmPglaU/+tqUbDlyt+VQ5bu9oPLdmxef7cw5lS/9VgQIPYFK1tbXx8fG8Y30o8KqE2d6yB8endtUY+aU8qcCL4Kdx3bb3/S377HfC2jC3DgUewKTq6+uXL1/OO9aHAq9KmPYt2zM1XVRvwJfyghV4KVovp20/kPPOp0dScnaXl39uuV+d6bv+R1ptWvlFnx2iDWmq5R/Chw4UeADQAwq8KmH+t2zv9MvqW4O6fSkvzAIvBR3yNp5bs/NI9pLMg1Qy07/6Kr94c92ZjMHmj6UldY7hm8lXaz6llcn8005aMVo9Wkla1eh+0xYFHsCkOjs74+LieMf6UOBVCWu9ZTd2jdIxfd6lR9p9KS+6Ah8iXPWf1FSt3ZO/bcPeHYnb9i5ee4iO+6lB3dxjW+kwmvYD6ACaRXdDirtxJQvxOTk1xCAtIBamG9LN6U7YPVP9pqAG1XJ6uKq/rKOHFquhSqDAA5iUzQo8OBn7Ul72D11PRib5kEpUL/B2ChR4AADQCftS3pqye63uZ3woNijwIQIFHsCkXrx40dPTwzsA9sK+lJdc3Ha5Y5gPRQUFPkSgwAOYFD6iBydgX8qjSl9xI5ov5aHAhwgUeACTosP33/72t7wD4AA3ukdTSyP7Uh4KfIhAgQcAAHPpGvBkfRfWl/JQ4EMECjyAeS1YgFc1ONrg+NSesz3iS3nU/aDwbu75199NQYEPESjwAOaFAg/AeKamd/74QJwPYNXJjimv7z/sUeBDBAo8AACYXeEVt6juLP4l/86joee6FnjvuG9Vhs4HGBxrnjPoH7SA/201DhR4APPq7OzkLQAIROUC77nP71eirN+iS9WahKjZ4ZNuqFKgwAOYFz6iBwhNqyN4VumlQXYUrhwhtKRyRBn+1/aW+gaV+wSTvb4R0VU1UOABzAt/Bw8QmiYFnpVhuvQffLDj9QgVb+/46y4LthMQNeneYgsUeAAAsCpNCryEVXrWoAJPqLQr6z0bETdnwf5vni3AbkWUH/WzwBF8eFDgwW7oCB7/DQ8QgvoFnqG6S212OK78v3Yap2AFmxV+Ku2E3ZaCFX7C7kEKupYd9LPyz4T4X/zYAgUewLxQ4AFCU7nAEyq3VH2VBZ4Vcgr/z+RZORcLiGD7BOHzP7JXI1DgAQDAqtQ/gqcIWOBZWyzDggQ8/mYFXlzFjvLZx/UU4v4p2IcBKPDzQYEHu3G73R6Ph3cAwI9OBZ4dqVMxZvWYVW5ahv3Xu1S/KVDg1YYCD3YTHx9fW1vLOwDgR6sCr8TKPCvPrB4TNsiwj+6pZotro+b/aX8MgQIPYF7Lly+vr6/nHQDwo0mBnzeolotDcBbsEF8cspsjUOABAMCqjCnwFgkUeADzwkf0AKGhwIcIFHgA80KBBwgNBT5EoMADAIBVocCHCBR4APNyu91jY2O8AwB+UOBDBAo8gHklJCQUFBTwDgD4QYEPESjwAOaVnp5eVlbGOwDgBwU+RKDAAwCAVaHAhwgUeADzwkf0AKGhwIcIFHgA80KBBwjt469Kb19aJRU2BEV3Q8r7X3zDnybrQ4EHAHAW9+DYkvVHJ35Jksqbw2PyTtKKjUdbH/Tzp8n6UODBboZm8Q4ABDI2Mbl4XeGF6gypyDk2Wi6upiekb2juT+ZYHAo82E3WLN4BgOBaXE8Wryuo+PZzqdo5Kqor19GTQE8Ff1JsBAUe7Gb3LN4BgPlMPJ/aU35paVb+1ZpPpeJn42g8t2bFxqM535yjzedPhO2gwAMAgI93evpCsyvxy9L3tx2m41pvq1wULR20OReqMxKzD7//RUn1tXbaWL7Z9oUCD3ZTUFCQkJDAOwAQrdYH/du/PvNORn7KjoPl5Z8/vJYilUyTR9/1P9Jqp3x5YNHafNqQpo7HfMMcAwUe7AYFHkALdMjb2PZoZ+n5JRsKqWSm796fX7yl7kzGYPPHUmXVOYZvJl+t+ZRWJnPPfloxWj1aSVrVyRdevupOhQIPAAAxcT1+WtPYsaf80oajlYlfnli8ruCdjGOJX+Rv2H8gt3AHHUbTfkBT7WoW3Q0p7saVLMT/AlBDDNICYmG6Id2c7oTuiu6Q6veitcfoITIPV9LDVdW30kPzlQA/KPBgN2NjY/399vlLVgCA6KDAg93gI3oAAIICD3ZTVlaWnp7OOwAAToUCDwAAYEMo8GA3tbW18fHxvAMA4FQo8GA3KPAAAAQFHgAAwIZQ4AEAAGwIBR4AAMCGUOABAABsCAUeAADAhlDgAQAAbAgFHgAAwIZQ4AEAAGwIBR4AAMCGUOABAABsCAUeAADAhlDgAQAAbAgFHgAAwIZQ4AEAAGwIBR4AAMCGUOABAABsCAUeAADAhlDgAQAAbGdm5v8HZ+keHMYzDIEAAAAASUVORK5CYII=)

频繁的状态切换会带来一定的开销，降低IO效率





使用NIO方式则会使用到零拷贝技术，不需要用户态参与，提高性能

![img](data:image/png;base64,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)





Java标准库（如果这样回答就要小心了，因为很少回答仅仅只是调用到某个方法就完事儿了的）

没有使用NIO机制







## Java并发包的工具



主要是JUC及其子包的类，如：

- CountDownLatch

主要的API为：await、countDown、getCount。当CountDownLatch减到零的时候await的线程就会继续运行，且CountDownLatch不可重置

- CyclicBarrier

主要的API为：await，当await的线程达到指定数量后，这一组线程同时开始运行，且CyclicBarrier会自动发生重置

- Semaphore

主要的API为：acquire、release两个方法。就对应着操作系统中的信号量



具体使用可见Java并发编程实战的笔记，里面进行了详细的说明



JUC提供的支持并发操作的集合：

![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAvAAAAFHCAYAAADHgXgqAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsMAAA7DAcdvqGQAADtLSURBVHhe7d1N6FzXefhxtXZp07qJW7vGqYNkqoQIYlJTvBCO27hgKoFVMI2LVeI0ggpsNYaYJsGiUXHoL1QRDmihFhcMfxWykEGlWpgiqCleeOGAoSpo4YURXnihhQtaZOFFF/ff79V9rEfHd+Y3L3fu3Dvz/cBl5r6d+3Zm7jNnzj1nTyWt0c//X1X95Cf/N/zf60fNNEmSJE1mAK+1MoCXJEmajwF846OfN4Fk2/BGs5Dm9kbb+WyGn/9fxG4AL0mSNB8DeK2VAbwkSdJ8DOAlSZKkETGAlyRJkkbEAF6SJEkaEQN4SZIkaUQM4CVJkqQRMYCXJEmSRsQAXpIkSRoRA3hJkiRpRAzgJUmSpBExgNfaffDBB9Xp06erw4cPVw8++GC1Z8+e+vXxxx+vdnZ2qvfff79ZUpIkSQbwWpv//d//rV5++eXqC1/4QvXiiy9Wb7zxxifBOkE94ydPnqyD+e9///vVxx9/XM+TJEnaZgbwWov33nuvOnjwYHX06NHqxo0bzdR2v/jFL6rjx49XDz/8cHXlypVmqiRJ0nYygFfvCN4pdb948WIzZTaUyLPeO++800yRJEnaPgbw6hXVZih5nzd4DwTxBw4csDqNJEnaWgbw6hV13qk2s4wXXnihrhMvSZK0jQzg1RseTKUKzG513ndDnfgvfvGL1dWrV5spkiRJ28MAXr2hqUham+kCJfmnTp1qxiRJkraHAbx6Qzvv1GHvAg+y0iqNJEnStjGAV29oz72rTpk+/PDDujqOJEnStjGAV2/oYbUrH330UfUrv/Ir1ec///nqscceq0v3jx07Vj/c+qMf/ag6f/583dLNW2+9VTdbScAvSZK0CQzg1RtK4HmQtQsE5HfffXf113/913WpPoH6pUuX6sB9Z2en7sGVgP7JJ5+sHn/88Xrb99xzT3X//fd3tg+SJEnrYACv3hBId1kHfv/+/dUDDzxQty2/G5b52te+Vp09e7aZIkmSNE4G8OpNlIx3IdJ65JFHZuoUihZrjhw50oxJkiSNlwG8ekNVF6qy0I77MuiFld5Y33333brKzBNPPNHMaUf1GrZLvXlJkqSxM4BXr3jI9Pjx483YYkiD3lhBMH/vvffWD6q2IWgneH/zzTebKZIkSeNmAK9eEXDTfvuideEpTacX1lyKnwP60h//8R9Xf/M3f9OMSZIkjZ8BvHp35cqVug33y5cvN1NmQ/DOerxmtCpDizRl1Zxz585Vv/d7v9e6jiRJ0lgZwGstaEWGeuyUnO9WJ55Se0rZKXmfFIjzgOqrr77ajN38kRBNRlJ9hmo0PPQ6S4s1kiRJQ2YAr7WJwJxA/uWXX66D+v/6r/+q59HOO+O0NjNLoE+VnIceeqh+z3Jf+cpXqtdff70eB3Xhn3rqqbrVmq56g5UkSVoHA3itHaXlNPO4d+/e6s4776x7bKX0nICcUnNam5nFvn37qrfffrt67rnnqr/6q79qpt6OUnrSfu2115opkiRJ42IAr0GgpJ1OmT7/+c9XV69ebabO55VXXqmr2Xz5y1+eWlpPizU8SPv0009XN27caKZKkiSNgwG81opA+xvf+EbdS+v169fr95cuXWrmzodqMgTllOjvhrrwVN/xAVdJkjQ2BvBaG+qiU03mO9/5zicPl1IX/vTp0/X7PviAqyRJGhsDeK0FTUhSF/1f/uVfmik3XbhwoTp27Fgz1g8fcJUkSWNiAK/e/fjHP65LvdseTqX6y8GDB5uxfvmAqyRJGgMDePWmrO/ehmXolGldfMBVkiQNnQG8etFW330SHiylWsu6+ICrJEkaMgN49eL555+v9u/fP1Ob7ocOHarbc183H3CVJElDZACvXlBlhhZmqGPOw6Lnz5+ve2JtQ6+r1EcfAh9wlSRJQ2MAr15Rkn3x4sW6Hjx13SmZ/+CDD5q5N/3jP/5jXYVlSHzAVZIkDYUBvNaCQP7ee++t/uIv/qK66667qiNHjlRvvPFGPY8mJhkfGh9wlSRJQ2AAr5W6du1atWfPrWxGPXjqt//sZz+rHnvssXoaLc+cO3eufsh13759den7gQMH6nm7od140uyLD7hKkqR1M4DXzHi4lGA8hllMCuApyb506VIz9RaCYkq477zzzol15LO+A/jgA66SJGldDOA1E4LwEydONGM3A3MC+t20BfBnz56tvvjFL04NfHl4tC2AP3PmzG37sU4+4CpJktbBAF67ImAmcF5EWwD/6KOP1kH8IoYUwAcfcJUkSX0ygNeuCMAJxKchMGe5GGL5MoDfu3dv/dAq9d6R1ysD83IeQ4wzUH2GIf8TwHheJqcZaeSqQF21N+8DrpIkqS8G8JqqDMDbEBDnQDmCaJTrf/azn61bngHr5QCacdYFwXtOM5Ql8DmAJy22lX9skE78e8B6OWiPYL4rPuAqSZL6YACvqSIAz0FxNinAJ3AmUM7zqTP+y7/8y9W//du/fTK9HAi2WY/120wL4JkewXoo5+d1p21nGT7gKkmSVskAXrsiAC4D4zBPAL+zs1NXnymnl8YewMMHXCVJ0qoYwGtXBLoE2wTDgQA8AmNey6A6AuMI1GlRhgc9qV5CemCZvB7vWR6sk4PxeM9rbBc5QI/9jDTANmK/+wzggw+4SjfR4/Lp06erw4cP1/9Q8VnllV6Z+XHvD11Jmp0BvGbGDTeGMvBlPM8PEcCfP3++vlGzXATwyOvkHwjI8yIoj/Ri+RzAg/G8Xv4RsI4AHj7gqm1GNbKXX365/vH+4osv1j0uR7BOUM841c0I5nmGZJb+HyRp2xnAqxeTOm7aFj7gqm3Ej9eDBw9WR48e3fXHKy1THT9+vP6uuHLlSjNVktTGAF4rx0OddNwkH3DV9iB45wfrxYsXmymzoUSe9d55551miiSpZACvlTty5Eh17ty5Zkw+4KpNx49TSt7nDd4DQfyBAwesTiNJExjAa6Uohbv77ru9EbfwAVdtKuq8U21mGS+88EJd7UyS9GkG8Fqp559/vq4uonY+4KpNw4OpVIFZNj9TJ56qd1evXm2mSJKCAbxW5vr163W777xqMh9w1SahqUham+kCJfmnTp1qxiRJwQBeK8PN99lnn23GtBsfcNUmoJ136rB3gQdZ+YdKknQ7A3itRHTcZHNw8/EBV40dP0K7yrsffvhh/c+UJOl2BvBaCR7MpOMmLcYHXDVWdKDWpa7Tk6RN4DejVuKhhx7a6o6buuADrhojSuB5kLULlMDzQ1aSdDsDeHXu8uXLdtzUER9w1djwz1uXdeApDJAk3c4AXp3jITY7buqWD7hqLHZ2djprOrbLtCRpkxjAq1O02WzHTavhA64aA/ImPzZpx30ZfIfQG+u7777bTJEkBQN4der48eO227xiPuCqoaPaF98FyyANemOVJH2aAbw6Q4dNlL7bcdPq+YCrhozSc/LnonXhed6D52iWLcWXpE1lAK/O0HHTsWPHmjGtmg+4asjoA4K8yUPts3r77berP//zP68eeOAB87QkTWEAr07YcdP6+ICrhopWZKjHTlWY3UrT//M//7P67d/+7Wrfvn3V2bNnm6mSpDYG8OoE9bGfeOKJZkx98wFXDRU/7vmniECef+kI6mnfHbwyTn35z372s3UJ/OnTp+vlJEmTGcCrE7TV3FXbz5oPneYQ9NB85z333FP3XEmJPO1x0wyfAb2GgH/neMCd74r77ruv+qVf+qXqN37jN+r3EbyDgP73f//3q4sXL9bVaHjeIwJ+SdJNBvBaGoE7pWvqF9VlKKmknvGLL75YX4cI1gnqGadaDcE8JaA27akhIXi/8847q3vvvfeT4B3k68997nPVt771rerJJ5+sf4iSh/lxSjW9rnp5laQxM4DX0qg6Q9OG6g+lkgcPHqyOHj26ays01D2migKtgviMgobiq1/9anXXXXfdFrwH/k0qH2L9+te/Xv30pz9txiRpuxnAaykEhJSgWbrbH4J3St2pYjAPSuRZjyoK0joRtNPkbFvwjp/85Ce31YN/5ZVX6pJ4SdJNBvBaCs1G2nFTf6heQMn7vMF7iOpO/uDSuhC0UxVmUvAOfmRGwE4hActbD16SbjGA18LsuKl/lEpSbWYZNOlHnXipb7ME7+CHKv/sUT2Mql8XLlxo5kiSYACvhVHybsdN/eHhParALNvzKnXi6eXy6tWrzRRp9aj6xQ9+/gGi1SRK2af1W0A9+D/5kz+pHn300WaKJCkYwGshVMGghMyHIvtD0ENrM12gJN+qT+obPx7pmZXWkXiI9dd//dfrQP3HP/7xpwL6v//7v69L6+nbgOo0tj4jSbcYwGshtDpjx039ItChDnsXCJaomiCtC02e/sEf/MEnAf3v/u7v1q3SkM//4R/+oTp37lz1la98pQ7qeYiVAgOmSZIM4LUg/gbnxqv+0BZ2V50y8UAg1XGkdaE0PudB/s2jrfd//dd/rQN35v3qr/7qJw9cR9OplsZLkgG8FkApML0pql/0sNqlrtOT5kWJe8YzNVSZ4d8hAneC/FytxtJ4SbrJO7jmRtWZ1157rRlTXyiB76rkkRJ46hdL68TD1Ll5SN4//fTTuz5bY2m8pG1nAK+5RJvMtiPeP4KVLuvA+y+K1o08/e677zZj87E0XtI2M4DXXPiLO/eQqP7s7OzUD/t1ocu0JOzfv39itaxr167V806cONFMuenZZ59d+keppfGStpEBvGZmx03rxQOsVKOhXvAy+PeEh5AXLfmU2hDAM5w5c6aZcguBO/PKAJ4OxbqojmdpvKRtYwCvmVFie/z48WZM60DAs+w1IA16Y5W6FMF7WQofpe/MKwN4gu4u/9GzNF7StjCA10wotaX03d4714vrQAsdi1Y7eOutt+oHB5ctxZdKBPAXLlyoDh06VL+GCNxjCBHsM7BuiOVIJ+a3lepPYmm8pG1gAK+ZcCOkgxWtHw8S00b2vO3wE7yzHq9S1yKAZ8gBOQE4pfA5gGecoJy8eOTIkduCfpZhnbfffrsejxL8GJ+VpfGSNpkBvGZCqa0dNw0HrchQj52qMLuVplNqT7UZrqHBu1YlAvh4T8BNkE5wjhzAh717936qlL1tubJUf1aWxkvaVAbw2tWlS5dscnCAIjAnkKceMUE97WjHwDitzcwa6EvLyAF8BO4RyCMH5swnaKdKHv8KMX0VAXywNF7SpjGA16646dlx03BRpebUqVP1j6z77ruvuuOOO+ou6RnnwWNbm1EfcgAf1V6YFnJgngN2emMtS+AZD/wAYJw0l2FpvKRNYgCvqey4aVzOnj1bfelLX6qbm7xx40YzVVq9HMAjB+nIAXwE5TF861vf+lQJfJ4/b/33aSyNl7QJDOA1FR2t2HHTeFBdhmcVfvCDH9QPHVPqKA0ZgXT+lygH+qtiabyksTOA10R02MTf2x999FEzRUNG4E4ADwIUAvgf/vCH9bg0VGVvrH0E8MHSeEljZQCviag//fzzzzdjGrqnnnqqrkITqEJDyzMXL15spkjDU/bG2mcAD0vjJY2RAbxaRcdNlFBp+Gh1hn9LynrvtPLBMwx2wKWhIngeQjU9S+MljYkBvFpREkUHKxqHl156qXruueeasdtRAk9JvA+1aoh+9rOfVcePH2/G1svSeEljYQCvVgR8b775ZjOmIePfEkrZaTFoEpqZ9KFWDVH0xjoklsZLGjoDeH0KHTc9/PDDzZiG7vXXX6+DjWnioVYCeWlI3n///eqRRx5pxobD0nhJQ2YAr0957LHHqvPnzzdjGjqC91l6qfShVg0RPQTTG+tQWRovaYgM4HUb2mO246bxmNbRFtOYT8C+s7NTPfPMM3VnOzzsykOv0lCQJ4fM0nhJQ2MAr9scPXq0DvY0DjTzyQOAPK9AYPHd7363OnToUN0TK60IUTWBdrZPnz5dB/K0RuOPMw0N/wyN4UelpfGShsIAXp+IpgjtuGk8vvnNb9al6k8++WQdvL/66qv1Q4F0wkVwT/UEaegIiHNvrENmabykITCA1yfoUMWOm4bv7bffrvbs2fPJwHgbHkQeS1Ck7Vb2xjoGlsZLWicDeNUoqbXjpuG7du1aHbTzGib1Wkl1qPwwcrmeNBRlb6xjYWm8pHUxgFeNLvjtuGn4aG2GOu6zoN47gVEwgNdQEQQPoTfWRVkaL6lvBvCqS5F4iIy60xq2qD4zqdpMWb3m0Ucf/aTUPgbqzEtDMqTeWBdlabykPhnAy46bRoZS+LZAnEA9T/uP//iPernAe0vgNURD7I11UZbGS+qDAbzqjpsoAdO4UPedoPzMmTP1OK+Ml0ME7fm9NCRD7Y11UZbGS1o1A/gtFx03ccPR+ET1GF4J4Cc90AoDeA3V0HtjXZSl8ZJWxQB+y9FSCQ87ahyoPhMl7og67/l9rh+fH3gt50lDMvTeWBdlabykVTCA32LRcdONGzeaKRqDqDoTQy5Vj/rxMWSxng+xaojG0hvroiyNl9QlA/gt9uKLL1YvvPBCMyZJ60Ngu+kdj1kaL6krBvBbilJ3Om7i4TFJWrcx9sa6KEvjJS3LAH5L0XHTU0891YxpkxEkWE1KQzfW3lgXZWm8pGUYwG8hbhwPPvigHTdtCZrn2/SqCRo/gtkx98a6KEvjJS3CAH4LXbx40Y6btsixY8eq8+fPN2PSMG1Cb6yLsjRe0rwM4LcQpT123LQ9CAxOnjzZjEnDtEm9sS7K0nhJszKA3zLvvPOOHTdtGR4M9HkHDd2m9ca6KEvjJc3CAH7LPP3003bctGUoyTtw4EAzJg3TpvbGuihL4yVNYwC/RbgJ0HSkLZJsn03uIEebY1N7Y13UKkvjuR9QmHP48OG6UQM6eeOVHww7Ozs2MSwNnAH8FqHjJgZJGqJN7411UV2WxvOjgNZ++LeD+wFV7CJYJ23GeWaGYJ6mPT/++ON6nqRhMYDfEpS6U4pjqYqkoSJAtcnTdl2UxscPgaNHj+76TyxVmmgViBbLrly50kyVNBQG8FvCjpskDd029ca6qEVL41mPUneaEZ4H14P1aABB0nAYwG8BSm74O/Ttt99upkjS8Gxbb6yLmrc0nuUJ+ucN3gNBPA/CW51GGg4D+C3Al7bNs0kaOoLSbeyNdVGzlsZzTqk2s4wXXnih/oElaRgM4LcAwfuFCxeaMW0rWvigXqs0VNvcG+uidiuNJ7CnCsyyrY/x3cFDxlevXm2mSFonA/gNR7UZvrz5ktd2e+ihh3xAUINmb6yLm1QaT1ORXbU+Rkn+qVOnmjFJ62QAv+F4cJXSGYlOvM6fP9+MScNjb6zLaSuNp533rh4M5kFWWqWRtH4G8BuMm6EdNylQemYdVg2ZvbF2I5fGcz67aj6YNvq9PtIwGMBvMDtuUsbDzFZP0NDZG2s3ojSeHla71HV6khbjJ3FDUepO6bsdNynw8Bn14KUhszfWbtGE8DztxU/Ddbn//vubMUnrZAC/oSh5seMmSWNjb6zd4nx2WQfeQgBpGAzgNxB/nVJP0Y6bJI2NvbF2a2dnpzp58mQztpwu05K0HAP4DUSb7zzAJEljY2+s3aIaJdVolu0Dgl5Y6Y3Vf0ekYTCA30A0w7Zol9mStE5U/7M31m7xo2jZDrJIg95YJQ2DAfyGodoMpS123CRpjOyNtXuUntN++6JVk+hgi4eL7clZGg4D+A3Dg6tnz55txqTb8cOuqxYppFWwN9bVuHLlSv1s1OXLl5sps+F6sB6vkobDAH6DUNeRHvjsuEnT0M62JWkaKr7H7I11NWhFhnrsVIXZ7TuAUnuqzVDybvAuDY8B/Aax4ybNguDIB9E0VPbGuloRmBPI86wBQT3/yvHvHO28M05rM7MG+pLWwwB+xPjSPX36dHX48OG63js95HHjo91fvoDtxEltjh07VtczlobK3lhXjyo1p06dqtt1p3OmO++8s7rvvvuqBx54oPrLv/zL6uc//3mzpKQhMoAfIUpKKDkhWKfEnQeTIlgnqGectnoJ6ilpocRFCrTyYVvOGjJ7Y+3f1772tTqgf+mll+pmiOnJm8IhCokolbdhBGlYDOBH5r333qu/XI8ePbprXXf++qQ1B1ofoLRFAj/w7KVXQ2ZvrP0jUP+7v/u7Zuzm/YMHXnNAf+jQIQN6aSAM4EeE4J1S93nbeCdgYz2+dCX+peEHoDRU9sbaP+4P/HCaJAL6+HeXaje2aCWtjwH8SFDaQSnIoh00cTPkoSSr00gaOqr+2Rtrv7jH0IrZbvcI+hoheOdV0voYwI8Edd6XLTWlRQFujJI0ZPbGuh5UkZnWZKTBuzQcBvAjwN+UVIFZtn13/gLl4bCrV682UyRpeOyNdT2o3/6jH/2oGbudwbs0LAbwI8CXalftu1OqRUsDkjRU9sa6HtSD//rXv96M3WLwLg2PAfwI0JRXVw908QVNqzSSNFT2xroebfXgc/B+/fr1ZqqkdTOAHwGe+O+qUybaVqY6jrYbN+g333yzGZOGhep+fk+tR64HXwbvlsJLw2EAPwL0sNqlrtPbNpw/h2EN2jz2xroeUQ8+B++hbZqk9fDONwKUwHfV3i4l8HwBa3EGjMPi9dhM9sa6HlSz5NxPCtQN4qVh8M43AnSu0WUd+IceeqgZ0yIMGIfF67GZ7I31dvv3768uXLjQjK0O9eC/8Y1vVP/0T/808bNF8P6bv/mb1Z/92Z81UyT1zTvfCOzs7NS933Why7S2lQHjsHg9NtMYe2MlwCY/njhxoply07Vr1+rpvC6qrwA+EKRP+2x95zvfqb797W83Y5PFOTlz5kwz5Xacq2XPjbSNvPONAA+wUo2GB7uWwYOL9MZqqdZypt3U1D+vx2YaY2+sBKs8BEqezMF2FwF8V/ghMEv1l90CeALy8odKG84D22xLi/MR8wzgpfl45xsJbmbLdmxCGvTGquVMu6mpf16PzTTG3lgjWI1S57DtATw/ahjKUnjWZ9pQzo00Jt75RoLSc9pvX/QvZZoF48GkZUvxZcA4NF6PzTTG3lgjgAcBawS4bQE84zGwXkyL98hBdBl4R0k/QwTBIbYXQ6yXpzFg0rKx7fgxwhDHhjKAj32IIcQ5Ib28PlgutrPbuYn9zPvDIG0rc/+IXLlypW4b+fLly82U2RC8s1607avlrOOmwY3yb//2b5sxZd7EN9MYe2ONYBU54Iz3EaTmwDcHtgTBBOYhSqgRQTCYXi6XPwc5fbafl83pYNKyEVjn+cyL8RzAsx7v6eWb9XI6vI/j45VxxPqznptYrjzuPC5tE+98I0MrMtRjpyrMbqXplNpTbYaSd4P37uQbZZe4Qf3Wb/3WJzfsrLzpDsn//M//1DdS9p1z07b/q7Sq66H1GmNvrDlYBZ+FCE7jNbAs02IIsVysE/J3QCwTymUR22bI+9T2XdK2LMuUaebAnHUi2GZarF+mk89JXp9l2Ebsez6etnPTthzr52OTtsntn06NQgTmBPLUESWojy9k2k1mnNZmZg30NZ+4oXSNGxtDGQD/93//dx0cz4t0/vmf/7kZ6wY3zxxUEbyzb9xwY/yZZ56p3++G5cpAYhGruh5arzH2xpqD1cA4gW4OPnkfwW8EpoHpfHZzgAzSic9LTgs5jXgf3yNlkJvTmbYs7/N+IQfgef+YFt8BFy9erO6+++5PtlGeE9JkvUgn9iGOJ+aj7bhiOZTHJm0T73wjRpUa/rL88pe/XN1xxx31lxsdbNDOO01F2trMasQNpUsE2gS03BTLALi8kc+KwJrgv0uxn4FqPWXVHoL43Sz6o6TNKq6HhmFsvbG2BfARCEfwmYNS8PnO4xGUMuRglfEIigl+83cC7yONMqhlXh7nfQTb05aN/Y5lwbwI9vP3UnkMFCB95jOfqdMoz0nsa6SbA/Np5ybmxfZRngdpm9z6pGi0/uiP/qj6nd/5nfpLU6uXbzBdIJjlBkfgy80pSqYCJd4EyUxn2ywbgTmvBNRMZ/j3f//3+sYY4wysy/RIh2lsixti3EwZ8o2aG29ON27CMc4Qpe3l/gbmk35UrYnl8jYZ2LdlkIY209h6Yy2D1RB5ns8c4rPMEPMy0ig/V0zjcxlifYYygM6fVdLP+xTLxvKTlo0APn9eeR9IJ4/n5dh3SuIPHjz4qXNSBukxvtu5ieXK7Ujb6vZvDY3OT3/60+oP//AP69J2/m6mFz2tVtxQukJgHVVduCnmKir5pkVAHEFzlIITHOebeiBQL2+uUSJPGmCb8UMgfiCAGy77UKbLeuxL3GjB+qRbLs+ypBfBOePcxGMZls8/GJbR9fXQcNgb62z4XOUgeSi6vB+Vgb607bzzjRhVaKgyEyVUBEWUemi1ugwYy4C9LEkjyC1LmXLJF8Ez88tgmDRzyTY397Kkm3HS4scA8wniCbRJs+0myfJ5XwPLRqlY7EccRzmwbPwQ4LULpKXNNMbeWPvA5yv/YObzm3+wbyIDeOl23vlGKtqFz4Hb+fPnqyeeeKIZ06p0FTBSek1abUMguOZmneWSbIJgStIJuqMUP250IUrJsyjFJ50oveO17QdDIEAo9yXLpfh5H0vTtrGIfKzaLGPsjbUvfMbi+6LLz9NQGcBLt/PON1Lc2KIaRSCov/fee6v33nuvmaJV6CpgpDSboDcj6CX9qNrC+7jO3Lh4HyVtBNME8Ay5FJ5APkrKmcdyuXQufjjEuuxDBPikwfvYPuNxw2Q6pfCsA7YR81g+l/rnEkGWiX0FxxCl/V3o6npoeMbYG6umo3lQScvzzjdC0THTjRs3mim3ENjTdKRWp4uAkYC2rToKQS3pE8gTDBMIsyzT4n0gQG6bznpMJ+AmeGY7EdyDaVF6R+BPMJ0D/EiX9aNUH6TD9PjRwSvjkU4E74gfAjEvfhAg1ms7/kWQljbTGHtj1WQE71T7nPTvnKTZeecbGYL2ab2qfvDBB3UbvLb9vjoGjMPi9dhcY+yNVdMRvBvES8vzzjcyR48erb73ve81Y+244b366qvNmLpmwDgsXo/NNcbeWLU7g3hped75RoRqCTy4Sl33aWi1gc6ctBoGjMPi9dhcY+yNVbOJIJ6ewyXNzzvfiBw7dqzu2OTb3/52/XAXgTpVZtrs27fP0o0VMWAcFq/HZuE77fTp09Xhw4erBx98sL6+vNImPJ3V+RDk5iB4v379ejMmaR7e+UaEknfafqck/uTJk9VTTz1VHThwoO5unL+Zc2D/4osvfqqVGnXDgHFYvB6bgU5/aHGGEne+v/gei2CdoJ5xvvcI5nlYf7d/IiVpk3nnGzlaDPnqV7/aGtjfeeed1UcffdQsqa4YMA6L12P8aPqWbvd5xqetda2MajW0TEN1Qr73JGkbeecbCZoJbOus40//9E+rvXv3NmO3o4TKUqruGTAOi9dj3AjeKXWftxdpSuRZzzrUkraRd76RmBTA81cz1WamoWS+bV0txoBxWLwe40W1GUre5w3eA0E8/zZaULE5qEbl81vS7rzzjcSkAP7pp5+uA/RpDOC7ZcA4LF6P8SJYo9rMMui4jjrx2gw2MSnNxjvfSEwK4Kk+Q533QDATA4E76+VpjGs5nEcNh9djnHgwlSowu9V53w114mmd6+rVq80UjZ1BvLQ773wjMSmA/9znPlc9++yz9XseaG0L0C2B75YB47B4PcaJpiKpAtgFSvJPnTrVjGkTGMRL03nnG4lJATwPsRK4I0rbYzwYwHfLgHFYvB7jRDvv1GHvAg+y0iqNNgvBO+3/S/o073wjMSmAZ3oZsEcgH3XjDeC7tSkB44cffti8GzcD+HGiPfeuOmUiL9tjq6Rt4p1vJGYJ4HMgn6cTwO/fv79+r+URMG7CcMcdd7ROH+Og8en6upkPJG0Tv/FGIkrV80BgngN1/m7M87OYxvLS2bNnqy996Ut1KeiyDxFKiyDv8SBrFyiBp760JG0LA3hpC9F29uXLl6sf/OAHdV1k2uOW+kTd5i7rwD/00EPNmDZdV1WvpDEzgJe2DIE7ATwI3Angf/jDH9bjUl92dnaqkydPNmPL6TItDRvBu63TSAbwo/bqq6/6Jaa50W8AVWgCVWhoR3vR3jClRRCIUY2GdtyXQS+s/CB99913mynadNz3DOK17QzgR4y/oN96661mTNoddYXvuuuuT9V7pxMcboh2hqM+0YPq8ePHm7HFkAa9sWq7GMRr2xnAj1iXD4FpO7z00kvVc88914zdjhJ4SuJ9qFV9ofSc9tsXrQtPAQZ5dtlSfI1TBPE8AyFtGwP4EaNVGR8+1KwIlrjZXblypZnyafRm6UOt6hP5kTbceTZjHgTvrOe/kNuN4P369evNmLQ9DOBHymbTNK/XX3+9OnjwYDPWLh5qtVt69YkgjHrsVIXZrTSdH6JUm6Hk3eBd0rYygB8pbni7BWNSRn6J3nmn8aFWrUME5gTyL7/8cv0dR0FFDIzT2sysgb4kbTID+JHihjbvX87aXlRT4B8bgqQS05hPwE6A9Mwzz9Q99/KwK/lM6hN5kX+AaNf9vvvuq3sMvueee+pxmoq0tRlJMoCXtsLzzz9ft/bx5ptvVufOnau++93vVocOHar27t1bfeYzn6keeeSR6tlnn61Onz5dB/K0RtMW7Et9ih6D9+3b58PVmhnfdbZOo01nAC9tgW9+85t1qfqTTz5ZB+/0IUD9YR7+uvvuu31oVYNEdRn+afze977nw9WaGYUQfK8ZxGuTGcBLW47Sd9t/19DYY7CWYRCvTWcAL225Y8eO+cCqBsceg7Usg3htMgN4acu98sordasf0lDw8LQ9BqsLBPFHjhxpxqTNYQA/Qjxc+PTTTzdj0nLoBdP8pCGxx2BJms4AfoTef//96sEHH2zGpOV88MEHdRN90hBQQGGPwZI0nQH8CNF6yOOPP96MScuzJRoNhT0GS9LuDOBH6Pz583Wb3VJXbIlGQ2GPwVo1fgD6faexM4AfIR44tORJXbIlGg2BPQarD5HPbJ1GY2YAP0L0MkdHPFJXbIlGQzCpx2Ce+aGalz0GqysE7wbxGjMD+BHipkUPmlJXbIlGQzCtx2CqDs5StUaalUG8xswAXpIt0WyZa9euVXv27PlkOHHiRDNnuF577bVOnv2hRJ9j5hy0YR7LaDtEEP/uu+82U6RxMICXVLMlmu1AKTZBai51PHPmzOBLt6m3vMiPTEr087ESnDOt7UcL54F5BvDbheD9F7/4RTMmjYMBvKSaLdFsh2mlz0PGj8tf+7VfmzvQagvgCdTbzgPTmGcAL2noDOAl1WyJZvNRyr5bcEqwSyAbQ14+gltKr2N+Lrkvq+Ywj/QIojPmRVCdA2reRxoxLdLP28zpxfpMi/kRmMd4DIjlSY/XEOeGabwGppdpIJbL+9VWqi9Jq2AAL6lmSzSbLwLXSSJ4LkusYx3WZ34E1YxHMB3rxrwwSwDPeIh08n6yHYYXXnihOnv27G3HEetH0J73F2y7PB7WZ1rebizHPJYJOS3eMx+8sn4+3nJcklbFAH5kLl++XDehJnXNlmg2XxmclpifA1bkALxcP4JttK2LvH5gnQiqI6AOkWYE5GB9puUh9qNcnwA67+OkAD7eszzDpGME47HdOMa25XKAr3Hjx2LON9LQGMCPDMH797///WZM6o4t0Wy+tuA4I/gcagA/KZgq158ngI/AnWm8Rz5GpuV9zceYlwsG8JuDa24TkxoyA/iRib+QpVWwJZrNR5AZQXcg6CRYjeB5UsBbBq05gGedvC7z2tIsg+KcPmJ5XgP7nH8EkAYDyvWZnveR9WJZlMuzLYaQj5H3+UcJaeUAPu9neZwaP66lQbyGygB+ZI4cOVJdunSpGZO6ZUs02yGC6BhyQEuwkuflADYHt4igNeR1c8AdwW6kx2sERWVA3RbAg+VyGqFcvwzg87ZRLs/7SccY+5K3G8vGchxnzM8/FLQZDOI1VAbwI/Pwww/b4YRWxpZopNnkQF+bjeDd54M0NAbwI3PvvffW3YpLq2BLNNJsDOAlrZMB/MjQG6G0KrZEo6F77733BvGwtQG8pHUygJf0CVui0dAt2iOrJG0SA3hJt7ElGg0dD1v7UKHWie9IH/jXOhnAS7qNLdFo6GhO99y5c82Y1D++I22dRutkAC/pNrZEo6F77bXXquPHjzdj0nrYxKTWyQBe0m1siUZDx8P8jz32WDMmrY9BvNbFAH5Ejh49Wr3zzjvNmLQatkQjSbOLIN4+WtQnA/gROXDgQN2EmrRKtkQjSfMheLdlJPXJAH5EbDpNfbElGkmShssAfiQ++uijOqiS+mBLNJIkDZcB/Ejw0NbDDz/cjEmrZUs0kiQNlwH8SFy6dKk6cuRIMyatli3RaAzef//96uOPP27GpGHhnm3rNFoVA/iRoO471WikPtgSjcaApiRt+UNDxb+YVH01iNcqGMBL+hRbotEY2COrhs4gXqtiAC+plS3RaOjskVVjYBCvVTCAl9TKlmg0dD7cr7EgiPfHprpkAC+plS3RaOj4h4j+MXyQVdK2MYCX1MqWaDQGlGp++OGHzZgkbQcD+BG4ceNG9YUvfKEZk/phSzSSJA2TAfwIUA/5wIEDzZjUD1uikaTVoXlom0HVogzgR+Dy5cvVE0880YxJ/bElGklaDYL3+++/39ZptBAD+BGgqTQeKJT6Zks0krQ6BO8G8VqEAfwInDp1yocJtRa2RCNJq2UQr0UYwI8AQdT58+ebMak/tkSjMaAqgsGPxiyCeP/x1KwM4EeANo552EXqmy3RaAyoZvjss882Y9I4Ebz7zJFmZQAvaSJbotEY2COrpG3TSwC/Z88eB4e5hi61pe/gMG3YBG3H5bCdwzq07YeDQxeDbuotgJdm1XV+Mf9pHpuSX8z3wrrygflPq2C+uqWXM+EJ1zy6zi/mP81jU/KL+V5YVz4w/2kVzFe39HImPOGaR9f5xfyneWxKfjHfC+vKB+Y/rYL56pZezoQnXPPoOr+Y/zSPTckv5nthXfnA/KdVMF/d0suZ8IRrHl3nF/Of5rEp+cV8L6wrH5j/tArmq1t6OROecM2j6/xi/tM8NiW/mO+FdeUD859WwXx1Sy9nwhPe7sKFC9X+/fubsXG7du1afZ15XVbX+cX8Nxn5j3yoWzYlv5jv+3Xo0KHqzJkzzdhwrCsfmP+WMyk/0WPrsue2izTWxXx1Sy9nYpYTzjIxkHH7cOLEiYW3y/Jl4MP6ZRp8ACel20UAT/rsS18mBepjD+DjPMbQV7fseZsMswbTLMfyWVyDMo1pQXoXATzb7DtwYb/jnPE57lJ5Xsdqt+Mov/+6+OyOwaTvKj7zy3wf8z0/6XMQAdOkvBrfP6v43iHddZhnuywbwzz34U0w6btsUn6aJ/ietOw8aWSxr23ic9X193Fpkf3eVL2ciWknPDJSDiJ4v+qAgA9H+UVBxpv1C5xly4zKcZTHOu1LvcS25/0CZx22sWwQNqtJN79ZzbJueQ6XtVt6nL/yus+aDxYV57G8bkybNb+wbM4vpMW0nC/nuV6kNe9xs82287dK5WeK97McH+dllnPL+doE044jvjeyPq9hnyKPhkmfiUXyf1bmyyzSnvRZZDrz5/3+n8W68vMs2+V4WS5/D/J+1u/AsSGP5GOd9l02LT/NKs7vIuJzkpFHGdr2K+KnMi7q2rry8xD1ciamnfBVfWlNwweI7bYpP2CTlGlwDKxbHg/H3vaF3Wbec8GyrDPteLoWH+pZj6k0y7pdf0Cnpce547r1bdKXc9uX5iRlGnxxxpdomOf4Ij/Ng7Qj//Hah0XznwH8TZyDVd9kh6T8DEz6Dlsk/2eTPtOItOMzmsX1YP483/+zWld+nmW7qzrmoYrvy9CWD8O0/DQrzu2i17/tXsT1Yp/K6bFsH98tix7PJurlTEw64bN8YUbGiCEvH4FDZCiG+KImE+UvbcQHgnmTPhg5A/Ia6UT6+cuG8fjwsV4eUB4fyzMvXvONJdKPIbB+TIv9Cvk4mJ/3jemkHfuOfK4irRhniH1lWt5vxLmI6xHHHWJ6YNlIl32Ydh1LzO/StPTYt90Cz3wsDHl5joPxPD+uQ7lsPnfx2oY0yzR4ZcjnjemRf8A80oxXsO+RR1ieeXE8sSzbKo+BZcC8PD3nsTgORH7LSDvnt0grpuVjjCHOV36P2Fbsc+xfG+ZHerFcfA5iyMdRYv4mmHQccc2nmXa+WL/ML1lc3zyP6xD5EJEXEdc21mMe79mH2I+8XAyRP2J63qfIi+W+MJ7zUsYxxj6B8bxuXj5Pj/1gm6Sf81+sE2nFtrO4HkzP5znSYJh2rAzT7DZ/VXbbbnm+28TxxpCX5xwwnq8x77O8br72GWnE+WWZMt/FuvGKGI8hxPrk95gXabOdvA77wrRJ32WRnxDXm/HyvMU2yrQR+apUppH3l+3Genka4lwxHscF9os0Ygj5PJXbYyCdmB/HuhuW1U29nIlJJzwywjSsmzNKXHREps0XnkwS48yLjBwfFpSZL8v7xLZIg8wc43l/czq8ZzmGWIZ5rBNIK6+ftwX2L7aFSDMwnvc77xvz8rbig5PXJ/38IeKc5HOX0y/3hbTAOryP8xpiOlgvbydrW7cU6XRlWnrlcZY4P+U5I71Yh3k5fc5fjJfXl7QY8rlqk68Dy+XtMx7zcjq8j+XIB7EMacW+xr7FPOTjL69bThPlfnMscXysyzyWCayb149lcp7LeTZvn31sO3eBdBhi30M+XuRzybZyGpPkYxyzScfB9HydSpynfO7La8s1ymkzHueV17btlueeaxLXOvJVzguRTr6WZbqMs26sH/sc47FumZdiftuQ82u5PzFeHktgG6RB+mC5WCfOIVgu1s/7xvzY57xtppXnKh8Py+bxUmy3b7ttt7wubUgjPr/Ix8p05udzmbeZ54U4fxnnNrbB8swv813eT5bN14fxmB/r5/RynmK5mBdYvtwmWJb1WZ75ka9yfgDz8njeJssyv5TTKNML084VQ14n9o/zEueGcfYl5GNnGdaJY45txfg0bcezrXo5E5NOeJkJSszPHxzkTNW2PhkmMhCvkYEmvS+V68d7lBk9L5uPMd4zLzIsmJ4zaHl8pF1m6HKI/S6PnXHmB5Yrzx3L5/0JTC/Tz8eW38d+8Zrl65L3vdS2bqltvWVMS2/SOQn5mgTORZyntvXzMc7yvpS3yXJ5+3nbiGXZh7hG+T3rB6azfJa3xWuez3ZYvxxiv1k2Hzvj5b7l+aTP+iWWyemHvK38PkR6sc/MjzTyEPtUnrtJWGcTTDoOpsc1b8P88lznm295XTmnkd+Y15Z2ee5Zv7xueZssm7+/yjwSA9tqWz/vL685rbblQVqxT4H9jm1FGuwb43HMgfnlMcY6kVfjfWwnn0vm53NXHjPa9r1tv7NYt2+7bZfjm7bf5XVDHD/a1o9z07Yu8vohX4My34Hl83VhPtPyEPtRrl9uj3mxrSzyRz4elo1tZeX1Zn7ev7zNSLeU04jly+V2O1e8J518zHwmys8Fy0X68floW27SuSmV+7TNejkT004483Lmy9o+hDlTMT9nZJBBImPkTMo6rIu29ULORGUmy+mB9Bhnel6ONGLZ2CbKYy2PL9JC24cnYz3ml0Pse9sXEenHfLAM68Q+cgzxAcvbb9uvfFxo29+Ylvejbd1Smc6ypqXXdp6yfOwhn6fynCIfYyxbXmveRxpZeR55n7eftw3eMzA9X6PY77xN9oHpWT4+XvP8SLcNy7Jv5ZDX530+N7FOxnhsozz2SeeuFOeyXL9UnrtJpqUxJpOOg/Mw6bqC9SL/Bs5xXMvyuuZ8kvNTVp77nBfjuuVtsmyZdyflgbb18/6W67Ytj5z/I69OSgPsY16G+eUxxjpl3mc7nJPYHpgf5473cU5jf/P7vO95v9vk7fZplu2yTBxzqe2c53PB/PK449y0rYu8fiCNuIZlvkO5j8yP5Uvl+uX2pq0L5kce4j37Vm6/vN7l/LxNpufthzINxHqx/7OeK6bF9smzkW+Zz/qkg/wdkJcLu52b0HY826qXMzHthJcXGVzEuNDMyxc1X3imMz9n3nJ5MkVbZiHTRUYNjOdp5XptmZ5xlsnb5D3TyvTLfWW5vEz+cCDSDrznPMUHK6cF5sf+xYcrK9Nn+TjPIM08znyGnE5sO18vxHSwjdi3PB1t+13Ky3dht/SYz3Fm+TzGe5THz7x8ftqWj2XycfOedPL1iLTzNMbzeuU1Yx5p522CaSyb0+J9uRzjkX7sU4jxvP041jJfoNx/0s7bL9OP5QPHVc5vO3dtx5C3ma8l70kn3ud5k+R9GLNJxxHnPecjxHktr2153fL5BunEeeU1X5/YBq95OunHeOxPXCewfN6HWCZvN+a3rc+8WJbXvO225cFxxnKsk7fP+xjPeSgfO/PjeJHTKM8hyzGel2ec5WL/QiyLmJfXYxt5n0o5rT7Nst04tnwtOG9xfMyL6wiOM461vK6ItOI85XXb0ozrEuMsE9csMJ/lQuxzYF6kXa5fXssyj5T7z3jsSyxbHgvby+sxL19/3sc+xPGVchqkG8eX9zfe8xry/sX8vC/5+vCaj5VlY5x5eb9iP/O2JsnrbbtezsRuJzwuXgyRARCZJIb84SAjkXmYFvNzhgHLMD0yaBaZKIa8XTCep+VMHyKNnPFin8t9YVreD/YtHw/Lx76EGGeIDw7bzOuF2C7LkVa5TP7woe28532O+Xmd8nowsJ2YHthWzM/rx/kqz2OW0+nCLOnlPMSQr2fscwz5GnIc5fwSaU863rweQ04b5bTyGoFlymsdeSkfB9eh3A/Gy+NhPbYD1mE8BsS1LvcVrBfb4LUtv2X5vMd5zNrOXaQTQ9v5iKFt+wxt+x6Yvwl2O4641jFk0z4P5XXl/Ed+QVxHhnxt8vZyPon8lLfBemWeLq97LN+2PuvmfYx1SLdteZB+zmuxDgP7G/tT7kdgfj5etl+ukzGe94HxyJesF+nnz0Xsez7HsY1JYt2+zbrd8nxybCGON4Z8rJzffL3AMnFOy3UD68U00sv5uS3fsVxclzDp/Jfrxz6EvG3mlcee8w/pxHikwzTWKfNp3p88r0yfgWXLNHgf8+NcgO0xLY4pnyuQVt5nxhlQbjsvG8vl+Sw/C5bVTb2ciVWdcDJSzoRtZllG7eID2Leut7nKYyi/0NrkL2LNZx3nblNuEJtyHLolAjleZ7WufGD+6wfnedbgdyhyoD8v89UtvZyJVZ3wWYLzWQIstVtX4Nl1flnlB363/LWuH0GbYFN+QK6L+W7zGMCrxHk2gN9OvZyJVZ3waQE8gSfbXTSTbDPOK+cu/jbrW9f5ZZUf+GkBPPPY9ti+XIdgnedulfmlT5tyHLrFAF4lzrMB/Hbq5Ux4wjWPrvOL+U/z2JT8Yr4X1pUPzH9aBfPVLb2cCU+45tF1fjH/aR6bkl/M98K68oH5T6tgvrqllzPhCdc8us4v5j/NY1Pyi/leWFc+MP9pFcxXt/RyJjzhmkfX+cX8p3lsSn4x3wvrygfmP62C+eqWXs6EJ1zz6Dq/mP80j03JL+Z7YV35wPynVTBf3dLLmfCEax5d5xfzn+axKfnFfC+sKx+Y/7QK5qtbejkTnnDNo+v8Yv7TPDYlv5jvhXXlA/OfVsF8dUsvZ8ITrnl0nV/Mf5rHpuQX872wrnxg/tMqmK9u6eVMeMI1j67zi/lP89iU/GK+F9aVD8x/WgXz1S29nAlPuObRdX4x/2kem5JfzPfCuvKB+U+rYL66pZczwQl3cJhn6FJb+g4O04ZN0HZcDts5rEPbfjg4dDHoJs+EJEmSNCIG8JIkSdKIGMBLkiRJI2IAL0mSJI2IAbwkSZI0IgbwkiRJ0ogYwEuSJEkjYgAvSZIkjYgBvCRJkjQiBvCSJEnSiBjAS5IkSSNiAC9JkiSNiAG8JEmSNCIG8JIkSdKIGMBLkiRJI2IAL0mSJI2IAbwkSZI0IgbwkiRJ0ogYwEuSJEkjYgAvSZIkjYgBvCRJkjQiBvCSJEnSiBjAS5IkSSNiAC9JkiSNiAG8JEmSNCIG8JIkSdKIGMBLkiRJI2IAL0mSJI2IAbwkSZI0IgbwkiRJ0ogYwEuSJEkjYgAvSZIkjYgBvCRJkjQiBvCSJEnSiBjAS5IkSSNiAC9JkiSNiAG8JEmSNCIG8JIkSdKIGMBLkiRJI2IAL0mSJI2IAbwkSZI0IgbwkiRJ0ogYwEuSJEkjYgAvSZIkjYgBvCRJkjQiBvCSJEnSiBjAS5IkSSNiAC9JkiSNiAG8JEmSNCIG8JIkSdKIGMBLkiRJI2IAL0mSJI2IAbwkSZI0GlX1/wHpUtMFCpTGyQAAAABJRU5ErkJggg==)



CopyOnWriteArraySet的底层就是使用到了CopyOnWriteArrayList，因此只需要学习一种即可

看下CopyOnWrite的具体表现：

```java
public boolean add(E e) {
    synchronized (lock) {
        Object[] es = getArray();
        int len = es.length;
        es = Arrays.copyOf(es, len + 1);
        es[len] = e;
        setArray(es);
        return true;
    }
}
```

应该使用到读多写少的场景里面。



至于Map，如果不追求有序推荐使用ConcurrentHashMap，如果需要对大量数据进行非常频繁的修改，建议使用ConcurrentSkipListMap







## Java提供的线程池



其实在并发编程实战笔记中有，这里在敲一遍增加印象



- CachedThreadPool

构造方法：

```java
public static ExecutorService newCachedThreadPool();
public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory);
```

特点：缓存是对于线程来说的，会试图缓存线程并重用，当线程闲置60s之后就会被回收，内部使用SynchronousQueue作为工作队列。适用于处理大量短时间工作任务的线程池，由于没有指定初始化大小，线程个数无上限



- FixedThreadPool

构造函数：

```java
public static ExecutorService newFixedThreadPool(int nThreads);
public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory);
```

特点：构造函数都需要指定线程个数，因此线程个数是固定的，其中的Fixed指的是工作队列，即无界的性质，如果工作任务积压过多或许会造成OutOfMemoryException异常



- SingleThreadExecutor

构造函数：

```java
public static ExecutorService newSingleThreadExecutor();
public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory);
```

特点：工作线程数目被限制为1，依旧是操作一个无界的工作对列



- SingleThreadScheduledExecutor和ScheduledThreadPool

构造函数：

```java
public static ScheduledExecutorService newSingleThreadScheduledExecutor();
public static ScheduledExecutorService newSingleThreadScheduledExecutor(ThreadFactory threadFactory);
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize);
public static ScheduledExecutorService newScheduledThreadPool(
            int corePoolSize, ThreadFactory threadFactory);
```

特点：都含有Scheduled（周期性调度）这个特性，区别是单一工作线程还是可以指定多个工作线程



- WorkStealingPool

构造函数：

```java
public static ExecutorService newWorkStealingPool();
public static ExecutorService newWorkStealingPool(int parallelism);
```





大部分时候使用Executors这几个静态方法即可，但是有时候仍然需要我们指定其中的细枝末节，这就需要我们对ThreadPoolExecutor构造方法足够了解

Executor框架的结构组成：

![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAvUAAAHICAYAAADOThVjAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsMAAA7DAcdvqGQAAEEOSURBVHhe7d1PyBznYT9wl7hJDi441CFq6jYqNkT8KrDb6KBSp1WpigU11FBBBHGJoYLatdq4RRDRGuSig2p8sEEHGVTigAou6CCDU3zQwQcdVOqDCj7oYIwOOviggg865ODD/Pqd7GM/Hu++7+777r67M/P5wPDu/J+dnXfmu88+88x9DQAA0GtCPQAA9JxQDwAAPSfUAwBAzwn1AADQc0I9AAD0nFAPAAA9J9QDAEDPCfUAANBzQj0AAPScUA8AAD0n1AMAQM8J9QAA0HNCPQAA9JxQDwAAPSfUAwBAzwn1AADQc0I9AAD0nFAPAAA9J9QDAEDPCfUAANBzQj0AAPScUA8AAD0n1AMAQM8J9QAA0HNCPQAA9JxQDwAAPSfUAwBAzwn1AADQc0I9AAD0nFAPAAA9J9QDAEDPCfUAANBzQj0AAPScUA8AAD0n1AMAQM8J9QAA0HNCPQAA9JxQDwAAPSfUAwzc7du3m/PnzzfHjh1r9u/f39x3333t3yNHjjTnzp1rPvzww8mUAPSVUA8wUJ9++mlz9uzZ5uGHH25efPHF5p133vkswCfop//MmTNtwD99+nTzi1/8oh0HQP8I9QADdOvWrebw4cPNiRMnmk8++WQydLp79+41J0+ebB5//PHm5s2bk6EA9IlQDzAwCfQpnb9y5cpkyHxScp/5bty4MRkCQF8I9QADkio3KaFfNNAXCfYHDhxQFQegZ4R6gAFJHfpUudmNU6dOtXXsAegPoR5gIHLza6rPbFeHfjupY//oo482H3zwwWQIAJtOqAcYiDRbmVZuliEl/i+99NKkD4BNJ9QDDETaoU+d+GXIzbJpDQeAfhDqAQYi7c0v60FSd+7caavyANAPQj3AQORJscu07OUBsDrO2AADkZL63Cy7DCmp37dv36QPgE0n1AMMxJEjR5Zap/7gwYOTPgA2nVAPMBDnzp1rzpw5M+nbnWUuC4DVE+oBBiI3yaYKTtqZ3408TTZPlX3//fcnQwDYdEI9wIDkSbAnT56c9O1MlpGnygLQH0I9wICklD3ty++0bv17773XPk12t6X9AOwtoR5gYG7evNm2Mf/uu+9OhswngT7z5S8A/SLUAwxQWq9JvfhUo9mu1D2l+6lykxJ6gR6gn4R6gIEqYT3h/uzZs23QT/vzpUt/WrmZN/wDsLmEeoCBS3Wcl156qW13/pvf/Gbzla98pf2b/jRbqZUbgP4T6gFG5Pjx480jjzzSPPPMM5MhAAyBUA8wEm+//XZb1ebu3bvNgw8+2JbgAzAMQj3ACHzyySdtyzapRx+vv/56c+TIkfY1AP0n1AOMQG6EfeGFFyZ9v7yJNq3dLNrsJQCbSagHGLiUzqeUPqX1tcuXLzePPfZY8+mnn06GANBXQj3AgCWwf/e7322uXr06GfJFefrsm2++OekDoK+EeoABe/nll9sWb2YpT5FNdRwA+kuoBxioW7duNfv27WsfNLWVY8eONefPn5/0AdBHQj3AQD3xxBPNxYsXJ32zpWnLhx566Et17gHoD6EeYIDeeOONNtTP69lnn21efPHFSR8AfSPUAwxQQvp99923UPe1r32t+fDDDydLAKBPhHqAkUmAB2BYnNkBNszzzz//pVL0jz76aDJ294R6gOFxZgfYMAn16VZlnlD/yiuvrHQbAFguoR5gwwj1ACxKqAfYMLNC/SOPPNI8+eSTk75f9pfprl+//llVnXTpL7rj0r311ltfWFaq92R4/maZ3WkjQb8env4iyyrj6+UCsDeEeoAN0w3VCe9FXidk1yXpCeL1NCWgRwn0dchP/1ahProl9Zm+LLMoy4ksqzsegL3jDAywYRKm60BdKyG9DvHdEvTSlVL3ukQ9Mm7RUJ9pS4Av6mkyvrseAPaOUA+wYXYS6mdNn+FCPcDwCfUAGyZBeVZIT/BOsK+nmVbFpgT2BPE6rNfTdkN8t78O/WU5tbKcEOoB1kuoB9gwCesJzHVXQnUd9tNfgnQZX7paPS7hu4yv11Nel1Cfv2Vc5o8S/LvDQ6gHWC+hHmBkEsgBGBZndoCREeoBhseZHWBkhHqA4XFmBxgZoR5geJzZAUZGqAcYHmd2gJER6gGGx5kdYGSEeoDhcWYHGBmhHmB4nNkBRkaoBxgeZ3aAkRHqAYbHmR1gZIR6gOFxZgcYGaEeYHic2QFGRqgHGB5ndoCREeoBhseZHWBkhHqA4XFmBxgZoR5geJzZAUZGqAcYHmd2gJER6gGGx5kdYGSEeoDhcWYHGBmhHmB4nNkBRkaoBxgeZ3aAkRHqAYbHmR1gZIR6gOFxZgcYGaEeYHic2QFGRqgHGB5ndoCREeoBhseZHWBkhHqA4XFmBxgZoR5geJzZAUZGqAcYHmd2gJER6gGGx5kdYGSEeoDhcWYHGBmhHmB4nNkBRkaoBxgeZ3aAkRHqAYbHmR1gZIR6gOFxZgcYGaEeYHic2QFGRqgHGB5ndoCREeoBhseZHWBkhHqA4XFmBxgZoR5geJzZAUZGqAcYHmd2gJER6gGGx5kdYGSEeoDhcWYHGBmhHmB4nNkBRkaoBxgeZ3aAkRHqAYbHmR1gZIR6gOFxZgcYGaEeYHic2QEG7vbt28358+ebY8eONfv3729Dff4eOXKkOXfuXPPhhx9OpgSgr4R6gIH69NNPm7NnzzYPP/xw8+KLLzbvvPPOZwE+QT/9Z86caQP+6dOnm1/84hftOAD6R6gHGKBbt241hw8fbk6cONF88sknk6HT3bt3rzl58mTz+OOPNzdv3pwMBaBPhHqAgUmgT+n8lStXJkPmk5L7zHfjxo3JEAD6QqgHGJBUuUkJ/aKBvkiwP3DggKo4AD0j1ANsoIsXLzbPPvts8/7770+GzCd16FPlZjdOnTrV1rEHoD+EeoAN9PHHH7cBfd++fc2hQ4eaN998c9vS89z8muoz29Wh307q2D/66KPNBx98MBkCwKYT6gE2WKrTpCpNmp986KGH2hL0hPdp0mxlWrlZhnyheOmllyZ9AGw6oR6gJ1JynqoxDzzwQPPUU0+19d9raYe+O2yncrNsWsMBoB+EeoCeSfWYCxcuNAcPHmzbmH/11Vebu3fvtq+X9SCpO3futFV5yuu0qPPee++1vxqkKtDLL7/c/mqQev/5MvHEE0+068+vCR5mBbD3hHqAHkld+4Tr3EibUJ0w/Z3vfKd58MEH2yfFLku+JPzqr/5qu9wsP9V//vzP/7wN8XlgVZ5Em3B/9erVdnsS5J955pl2PAB7T6gH2DCzbnRNiE9JeJqsTIBOHfq0dJMS9bSSk5LyWfXtF5XS+W9/+9vtuuaRLxupFqSUHmA9hHqANUkALiXuTz/9dFud5v7772+r02wnVXCOHz/elqAnUEdeL7NO/e/+7u+2of6nP/3pZOhsuUFXKT3A+gj1ACuSsJ1wPOsJram6cvLkybbEPXXVb968OddDn/JlIF8ActNsWscpUiUmVWOWoSwr25RmNVMdZxal9ADrJ9QDLMn169fbEve0GpOQm6oyaWM+N7Uuy7vvvtuG7NRn70qoThWclOLvRr5Y5Kmy5cFXP/nJT5of/vCH7etplNIDrJ9QD7CN1HFPafvly5fbtttTsj5NSqxT4p4wvNsHQE2T0vOE9q2eMpuqPCn9340sI78CFAn5eRhVvlB05T1//etfn/lrBAB7Q6gHmCHtwqe0PSXjaWUmN6cmWE8Lt6s0rf78LAng+aVgp3Xr05JNAny3tD/Dp/0K8OMf/7j53ve+17aSky88q/gyA8D2hHpgNFL/vC5xT8sxCcCpnz5Npt9tVZbdmlV/fiupB58WcRb98pHgnvnyd5of/ehHbSl+UdelT5cvPfkS9Nprr829rQAsh1APDEpCeELtNBmXOu4J8wn1CferqiqzLGkJZ1r9+e3ky0vqxefLwHZfTFK6n7CeEvpZgT6yn/KrRan+M60uffZ9flHIst56663JUABWTagHei3hPCXEaXoxpcSlHXc+D+sJ92fPnm2DftqfL136U51o3vAfCeqPPfZYO/9WLd7kV4L8CpIvUVt9UQBgOYR6YCOlCcWUCCdEJnjOKk1PSXYfStzXKaXn+fKTajzf/OY3m6985Svt3/Sn2cqtbryd5qmnnmpL4rdr8SZVcPIrQ6r0ZJ7cowDAagj1wEbJDaG56TJdSnrTn+ApsC9H9ucjjzzS/rqxU3lq7Q9+8IOZpfRd+cUgLQblV5R8EdjuZl8AFifUAytV6rinqccEu4S6VI+Z1QTirVu3tnzQETv39ttvt1Vtsn/zpWnWvQerki9mqYevpRyA5RPqgV1LOJtVHzs3paaaRx7KlPrdFy9ebB/SlNJb9k4+o1SDKV+mXn/99faG1nXQUg7A8gn1wEISyOsS9wSz3DCZkng2V26EfeGFFyZ9Wz9Qaq9oKQdgeYR64DMJermZ8erVqzOrZiR81SXu6kdvvpTOp5S+W90lNxinJZt1l5RrKQdg94R6GLk8eTRPS0374/fff39b5zotlSTY038J7N/97ndnfp4J0ztpB3/ZtJQDsDtCPQxQAlJuOE1gT53l5557rm36cZrUb7527Vrb7jjD8/LLL7ct3sxSniK7Kfc4aCkHYGeEehiYVGVIHff65tQLFy7seUsnrF++2OUXmO2+sB07dqwN0ptESzkAixHqYcMlzNQl7kePHm1LVtfVcgn9kWpVufdhO/nCl5LxTQzOWsoBmM9oQ/19992n27BuzBJcZrXbnnEpSU3rJSlxT0l8hsFW3njjjTbUzytVXVIyvqm0lAOwtVGHejbHmD6PlIYmoCeoJ6B8/etfb6vKnDx5cjIF7F5Cev2leZ7ua1/72sZ/YdRSDsB0Qj0bYQifR31zakrUtyr1VOLOGD3yyCNLKWXXUg7Alwn1bIS+fx4pcc97yN+UwG/V2gzDk8++7p588snJmM109uzZyasvev755z97Dwngy7asUF9oKQfgc0I9G2HTPo+0FpJmHkuJe0oD0377rFZENAc5bjl+8yCudUpgnncbpv2/vfLKK1/4MpJl9aXuupZyAIR6NsRefx6lqsystrlT2p6bDEuJex7ck5/4tbzBNEMI9Qn0fb8BNdXZtJQDjJVQz0ZY9edx5cqVtiSv3Jya9eWvurgsw7RQ/9FHH7XDUwJe95fgvFVVl2njuqG7Llkv05auyLz18GxDlP4sI3+z3Kyzux21rKvMV5fo53VZTllfvS+y7LLc/K3HbbXMacPnpaUcYIyEejbCbj6P27dvt1Vl0h73rBtPL1269IUS9015eibDUAJo6bpBPn8TTkvAzPgyTeR1QnXMCtf1/JF56sA7LTCXZUbmzbZE2c56fGSeevuLTFevO/1lmjJPkXH1cuvtrrcxw9N1bbWuRWkpBxgToX4FcgEqF83SdUvx+KJFP49z5861ddzvv//+z25OTTORnprKOmz1P17OB3XQTbgt54bSlYCb16VEvVaH48hy61BcB+byZaKrTFPWOW090R1f+uuuvJ9sQx2663VnXVlnUdY/a/uiXkfp6n23KC3lAGOxumS74XKhWJXuxXYduhfTTZfPIy1XZLtTqn769Onm6aef/kKIqaV0Xok7myLHb47daWaF+lnTZ9ppYXuZof7v//7vZ66nqJe/1bTdUB9lW7ul7POG+q22a6e0lAMMnVC/At2L7Trkwtm3UP/tb3+7OXr0aPsQplx8Uw8+4R02XY7faSE9w8q5Jv+PJZQn7Nb/nxlexuXcUZ8/SijOPPXwzN/tr0N/xtVfJDKurLOE6jo8Z/q6P/OW+TOuXle2qbzfDO+G+qwrw7vn2ay/zJfX9faVZWy1rmXQUg4wVEL9CuQiVF+UilzA6vWmv1xkI+NKV18kywW4dLlglmG1clFPV09fLpyzhkfZ5vpCXLY33bT3s0xZh3bd6avyf1J3JdDn/y5Kfx2Gy7T1/2LU48r8UYaVeer/y/wPl3FFzgn1PMW0UB/1tN3/+XpZ9fkp09X9Rabrvq8sow7o9TJnDZ+27GXQUg4wNF9MhSOSi8Wq1BfX0hW5yKXLBSwXriKv6wtsufiVi299YY+tQn10l5/+TN9dR7lglm0uF9bu/KuWdR88eHDSB7A3tJQDDIVQvwIJyN1SrlrWXQfsEri7XS4wWVa3tCsWDfVZRgnwRaYt29nd5rL87jpWJevJja83btyYDAHYO1rKAfpOqF+BnYT6WaXi6wr1RVnPVu9nGbKOVL/JDWwA66ClHKDPhPoV2CrUZ3jGJ1DXoTvbU4fuMn8pxc/fSMguwT3D6xA/rb8o/eWLRNRfArrbnOH1OutlrUKWn9YoHnjggebevXuTocAqnD17dvKKabSUA/SRUL8CCchZft2V0FwH+fSXIF2Cc+lKoI4SyNPV8yd4l+Fl2SWkR/ozrpT019Onq79EdEN9lPnT1ctdhawjtHYDq1f+39ialnKAPhHq2Qg+D9g7/t8Wo6UcoA+EejaCzwP2jv+3ndFSDrDJhHo2gs8D9o7/t93RUg6wiYR6NkI+j9yY5mdtWD3nv93TUg6waYR6NkI+j6effrptq/7atWvNnTt3XCRhRbR+szxaygE2hVDPRiifR37WTn3VP/zDP2y+//3vt8MANp2WcoB1E+rZCPXnkZKvlCR+9atfbUvsAfpCSznAugj1bIRpn8epU6dUEwB6SUs5wF4T6tkI0z6PXBT3798/6QPoHy3lAHtFqGcjzPo8cjHMjbNFnjh77ty5tooOQB9oKQfYC0I9G2HW53H9+vUv1KvP6+PHj7cl+FevXp0MBRahWtt6aCkHWCWhno2w6OeR0vs0f3ns2LHm1q1bk6HAPJz/1ktLOcAqCPVshJ18HvlJ+9VXX2327dvnoggLcP7bDFrKAZZp1KFet1ndTt27d2/yCpjHbv7fWD4t5QDL4MwOMDJC/WbSUg6wG87s9MaiD6K6cuWKVnJgCqF+c2kpB9gpZ3Z6Ia1EpN7pvCE9F8bUVc3P2VrJgS/S+s3m01IOsCihnt5IqVVKsBaRJjFLKzm5KQ2gT7SUA8xLqKc33nnnnebw4cOTvvml1D4lXrkoZhkAfaOlHGA7Qj29kYtYmq/cabv0qZOvpRygz7SUA8wi1NMrZ86caU6fPj3pAxgnLeUAXUI9vXL79u3m4sWLk77lSMm/eqpA32gpB6gJ9YxeWgJJtZ7Lly9PhsCwaf1mWLSUA4RQD//nxo0b7c/YuRH3/fffnwyl78rTinW6ebs+01IOjJtQDxP5KTtVe1LapSRzGPoe0thbQzletJQD4+SKBx13795VWj8QQj2LGNrxoqUcGBdXPHorpU9KoNiKUM8ihnq8aCkHxsEVj956+umn9/RhUrkZLaX49IdQzyKGfLxoKQeGzxWP3kr99+PHj0/6Vu/atWttHdVLly5NhrDphHoWMYbjRUs5MFyuePRWWnZIKw97WXqe0q0nnnhCKzk9IdSziDEdL1rKgeFxxaPXTpw40bbusNfSpn1+xs5FUb3+zbWTkPb88883//RP/zTpo4/+53/+p/nGN74x6ZvfGL8EaikHhkOop9dSJSY3gK1DSraW/XRblmtWSPvoo4/a0PfKK69MhnzukUceaa5fvz7p2yzZ3rynukuA7Zts8w9+8IN2+/M5/Od//udkzHK88cYbzZNPPjnpm1+2Z6y0lAP9J9TTe7lh9t69e5M++NyskJbAl64b6ndawpuAuuwvAvnikdZKaulfdgBeVPZZQvNOZT/VX5yyzzfll5Exh/pCSznQX85gsAJ+wt4M00JaAmlCeMJp/tYyLNVvFrHTLwLbKdtZJOTn/fzv//7vZMh65L3u5teBaV9M1v2eCqH+l7SUA/3kDAZLll8N9u/f31bNEe7XqxvSEkZTSpwQmQDfraKRwJlS4wzPvJm2BNj8LVVG0iWY5gtA6S/D0pXlZFjWlUBeT1tXb0iJdb3cjMt6S3+6LCPDu19CItuVabKOyJeB8sWku831LxPT1lv2T62E+Iwv06Yrpevl/XaHR15nW7I/M01kefV21OplZbr0R95/huW9ZXiWmb+1rCPjs51lXTHtfca0deU1n9NSDvSLMxisQGklJ6Hhxo0bk6HstW5Iy+eR4BcJlnX4KyXhCYwJkekSBkuQTvArVUZqWUYJilECZ4JwlhFZZ/qjfGmIEkC7yy0htgT1qINpurJdUd5XllcH+qynrDd/M1/MWm8J4UXmqUN+d3x3OWUdpT/zZnzZn5F5yvbX769se/3lpN5PmSd/oxvAM23ZH1lG+Yy721fMWle9TD6npRzoB2cwWKG0krNv377m5MmTHly1BnVI64b49NfjE/RKiCwyTQmxCeoZX4JllPBdAmskyJYS5iL9WVaCZ8YnHGeeLLMOtkUpRa5lPdOmjYTSLLcO3CWk1l2m2Wq9WWe97d0QX4+ftZysIyE6w7POet8UGZ/tq7/85HV3e8u68zfbUpRlR/niUdbTXWZ3+7ZaV1km02kpBzabMxisWEq1Tp8+3V4Q2VslpJUS5GldkcCd4F0rATUSBhOeEwjrkuD6i0DWk/G1LDddlpOuLHPal4giAbPelmkhv5bps97yhaMOvV2z1jttnmxrCfHd8RneXU6mKe8/+6iUns+S95T3udW+iCwz+7aWbcmwLKN8RmX/xqxlbrWuWfuML9JSDmwmZzAG48qVK83Zs2cnffB5SEvwq0t6IwGwBMPI6xJCE07zupQUJ3gm1KdLICxBJtOUUvfIdGWeKF8myryZtoTeLKMOq+nPeiPDE5rLcrshv8j4bEPmTYguwb8E8LKdWUeZf9Z6y/7I6yw360x/US8/47N99XIyf8aXLwH1foqMzzLLe8p0CeBZX3ebMq7MWwf1WqbPOur9kv1bPudZ73OrddXvl+1pKQc2izMYg3Hr1q22qoufhCkS0hL6ShitJVxmfEJjCZiZNsPK66IE3O7wBMgML8vP3xIQIyEy82SaBNBMX4f+styEzFL6H1lOhpeAWpZRukyf7a/Xl3VlXAmrZdvS1WE6pq034+ttzTR1SXv2UZkn64p6HZk2+zLKvi3TRYZluWUZmb5sa3dcdx/X+6zI9Olq2f5sZzHrfc5aV4axGC3lwOZwBmNQcnNqSuz7IvVT07qELyKrIaSxCMfLzmkpB9bPGYxBSYlRSov6Ir8uHDt2rDlw4MBnpZwsj5DGIhwvu6elHFgfZzAGJaVFuZjcuXNnMqQfrl692rZtn5J7JVzLI6SxCMfL8mgpB/aeMxiDc+rUqbYpyb7JF5Jz5861P2GzHEIai3C8LJ+WcmDvOIMBgyWksQjHy+poKQdWzxkMGCwhjUU4XlZLSzmwWs5g0AMp5VIvdXFCGotwvOwNLeXAajiDQQ+klZyjR4+2reRcu3ZtMpTtCGkswvGyt7SUA8vlDAY9UlrJOXHiRHP79u3JUGYR0liE42U9tJQDy+EMxmDdu3evLQUamvx0ffbs2famM7YmpLEIx8t6aSkHdscZjEFLdZUbN25M+oZFadb2hLTlyhfKIXO8bAYt5cDOOIMxaK+++mrz3HPPTfoYGyFtcQnuKTG9cuVK+9yEVPVKuOrjQ90W5XjZHFrKgcU5gzFoaVUh9TSHXsJY5H3mi8xY3u92hLTZ8r+Rm64vXLjQ/PjHP26efPLJ9n6NhPeE+NRxTgslCfd5oNtf/MVfTOYcLsfL5sm5TEs5MB9nMAbv+PHjvXzC7E7kgpeS1dRJzU/YY5eQNuTuG9/4xtThW3W/8iu/0tx///3Nr/3arzXf//73m7/9279tLl682FZzmBWYfvM3f3PqsobYsZm0lAPbcwZj8N55553m2LFjk75xSAls7id4+umntZIzYK+//nrb1Om88oXvhz/8YfPTn/60+e3f/u25glG+HOZYgk2gpRyYTahn8HLSH2OpTt53+dk6F0KGJ5/x4cOHm5/97GeTIbPlGEgpZymN/+d//uf2y+52oSjVbvLlATaJlnLgy4R6GLi7d+9OXjFECTf79u3b9nPOrzYvv/zypO+XXwgS6lOVYZbcGPvAAw+o6sDG0lIOfE6oB+i5n/zkJ221mlmuX7/eBv88u6GWsJ6SztwMO83f/M3fNN/73vea999/fzIENk++oGopB4R6GK3Uu9dKzjDkc/zOd74z8+boJ554or0ZdpoEoAT+bhDKMr/1rW81//AP/9DWqU9JaJbR/WIAm0JLOYydUA8jlWYK04Th1atXJ0Poo1SRyY2DCeYpqeyG7rfffrstjd+q7nxK6jNNXc3mP/7jP9r6+kWqNuRG2wSmPPtB6T2bKsexlnIYI6GeUUl4UXrzuVTLOHjwYNuCyq1btyZD6YOUSubhUAnZZ8+ebft/9KMfNadPn55M8ctqCQnr83xxS/ipb5xNoJ92A2Lq7udZCErv2XRaymFshHpG5eTJk+3Ps3wuF7pc8FKqlXqpbL58Oc2vLCk5r5/ymhLJlNiXUvQE7lS9mUeOgzyAKuG+3HybLwpbUXpPH2gph7EQ6hmVGzdutCd2vizhsA6IbJ7Ue084SQl5fmWZJqHlscce+yzgz5pumsyT/4/Mf+bMmcnQ7Sm9pw+0lMPQCfWMTqqbOKHTJwnbKQlPnflLly5tW40gLYDkOE8b84vKF4cf/OAHO/6Cp/SeTZb/HS3lMFRCPaOTEsXUs2Q+uQfBjWbrkQBy4cKFNiCnrvy8n0OeIpzqVOt86JjSezaZlnIYIqGe0UnYSOARVOeTUJlqHJcvX54MYS+kydGUtufm1Z3cxLxJNz4rvWdT5TqgpRyGQqhnlFLPeLubAPlc7kVIaWtuuvRz9WqldD1Pf03d9lntzveV0ns2lZZyGAKhHphLLnIJYbnopUSL5Uq4zc2p+VUkwXfooULpPZtISzn0mVAPLCSlrUMrQV63VG3KjXsnT54cXd1epfdsIi3l0EdCPcCapFpTHvKUak1KqpXes1m0lEPfCPXA0ihhnU9K49PiRsKCG5C/TOk9m0RLOfSFUM+oJShsUishfZb9KKRurQ4HuS/BzdrbU3rPptBSDptOqGfU3nnnnbb6A8uRn6dTlUQrOV929erV9ua7BNS0I89ilN6zKbSUw6YS6hm1nIzT2ogAulx56mn2a0q1xh688gvG0aNH25vu3HC3HErv2QR71VJOCgHyC1+eWbF///7mvvvua/9m3efOnVvrQ+bYLEI9o5dmBPO0TpYrP02PuYpJ3v+pU6faLzcpVVaat3xK79kEq2opJ+eMs2fPttUaU0CSX5ZLgE/QT3+uXwn4uYapzodQz+jlJJkSPydEliEX4oTLhPmEevVu94bSe9Yp//fLbCknv/ClamiO6e3OIfkie/LkyfaLRX49YLyEevg/+RnzypUrkz7YmQTLgwcPttVt3IC9HkrvWaf6ZvidtpRTGh1Y9JqUkvvMl6ZyGSehHv5PwpgT4d5IiVbC1pBuMMtP4cePH2/r1ubCymZQes+6pHR9Jy3l5JyYEvqdFjLl/JMvtH55HiehHthzCVdpISel2tevX58M7Z9cOHPBTlWblM65kG4mpfesy6It5aQOfb6I7kaq/blPbJyEemBt0qZ9AnEuen17oEu2PT91exhNvyi9Zx3maSknv/jlnLLb+3DyhTXr0arb+Aj1wFrlApYWHK5duzYZstnKrwz5iVyVrf5Ses86bNVSTn7tS5WdZUiJf35FZFyEeqDXEsb2op3mlManhYn8suCpucOi9J69NKulnLRDv6x7clLgkC8PjItQDx1K6/olpfypvrMquQCnBC1hPiVfjo/hUnrPXuq2lPNbv/VbSyuguHPnTvulgXER6qGSqiBprcCFfDPkaYl5Ou1WN5eVz2wV7TPnp/LUTU3LNnvxawCbQ+k9eyXnsFS7yZNil2nZy2Pz+cShIwEuJXSsX9prTpvvKTXdqv56WpXITWjLkvXmp/C0ztOXuv6shtJ79kqeDJubZZchJfX5dZFxEeqhI3Uac/Fmc6S1iK1amsnP2ClRT8n6bqTELE3BpXT2woUL2zY/x7govWeVUjCxzDr1KZRgXIR66EiQS4DUHNhmSenoVvXnc/NqbgzbSRDPPKnmk5KthLWEe5hF6T2rkOqGOcctwzKXRX8I9TBFbojMAzzol4T6tCqxiDz8KvOlmo8vcixK6T3Lkvt2UgVnt18Q88tlvnA6FsdHqIcpUq9RKUf/JGDlV5Z5nuyaOqcJY6m2s9NHskOh9J5lSPW/kydPTvp2JstQKDVOQj3Qe7mxNU3DpRpNbnDN61kS+PNglpSs5ifqeb4AwCKU3rNTOR/ll8Od1q3PsZeCCl8ox0moB3ovv6wkzKeU9I033mjD1LR68bnhNj9vp15+SuphlZTesxNpnje/OC5643/5pTJ/GSehHhiMq1evtqH9d37nd5q//uu/ngz95UXyiSeeaIPVVk1jwqoovacrVTxzTspxkV8NUw0w56qU1uc8lS+DqUaz3RfBTJ8qNymhF+jHTagHNl6qy+RBKot0999/f/Pf//3fbf3UlF4tegMtrILSe4o00ZsuhRGpMvijH/2oOXz4cPswvRRO/Nmf/VnzB3/wB823vvWt5q/+6q+an//855/9wpi/Cf75MjBv+Gf4hHoYgGmhVqfbqhuCae9Lt96O+eXXmhQ4TGuGN1/+0jJXvvT94z/+Y/Onf/qnza//+q+3hRXZz2l+N+3Qp7Tfrz4U/gNhG3l896a3juJiyiKGcrw47jeLz2Nx+bVmnutLSuEfeeSR9unZMIv/QNhGqm089dRTk77N5GLKIoR6VsHnsbhcX/KMjO385V/+pWYq2Zb/QNhGbkJKHcdNbi3FxZRFCPWsgs9jcbm+5ObpNMs7S+rNHzlyZEdPy2Zc/AfCHNJaRU6sm8rFlEUI9ayCz2NxCfV/93d/N7MUPs1a5qbZjz/+eDIEZvMfCHPIjUg5sW4qF1MWIdSzCj6P+eWX35deeqm94fVf/uVf2l+Du63XfPjhh+14N8IyL/+BMKf8/JmT7CZyMWURQj2r4PPYXtqRP378eFvlJo0wlGtK7ttKSzdFAn5at/nZz342GQLb8x8IA7Dsi2laWcjTV/da1ptm3Fbpo48+Gn34EOqXI/8jOWaX4ZVXXmmefPLJSd965H9vN+9HqN9aqtAkqF+4cOFLpfLvvPNOO67IjbEvvPDCpA/m4z8QBmDRi2kCRObpduWCvgmh/vnnn5+6bbvVDfUJUvV66nFDNZT3uNP3kWOp/rx3emxtF+ozft6gvkio7x6zy/oyINSv13e+8532M3BjLDvlPxAGYDcX01zEu6XjGbYJoT5dkdfLCC/TQn0C1Trlve3lNgwlfO30fSzr+F5mqF9E95hNf/2/slNC/e4liKfd+TztdVF50nBK6xPu3RjLTvgPhAEYQ6hfVrUZoV6oX9bxvSmhfrdhvBDqdy4hPCXsubH18OHDzbVr1yZj5penyOaGWTfGslPj/Q+EAdnNxbQO0kUJPVlu6YoSZBJEMzwhOeppu4GjHlemj0xXj0s3b6jvztsNxdnGenxZ77yhPvPXgay85+jum/o9TRuX5dfvpeyTyDrq6ev3Xw/PcotsS3n/9XLnlfmGYKfvI/uv3p+1WZ9HpL/s9/wt/wtFhtf9GV8fQ93/hXpcfYyU6epjqf6cM1+mLzJ9vd6tjp2ox6UrustZVL2ssUgrNidOnGgeeOCB5uTJk7sO5JvaGAP9ML7/QNilnLQ37VHdu7mY5iJeB5fIsHqZdYgoQaMOCpm+Drb1MuswUgeXspyiBJl6vm6QmTVv1NtUTxv19NNCffpLl/4i7yPzdgNXPU36M12U9dT7Iur5o7zXIuPqkJbXZZlRtrnsm4yrxy+qXnef7fR9ZN9l3tKVzyZ/u59txpfPM6/r8fm8y+eQcfVnGBlfpu9+hpFxZd3dY6zeru52ZL56XdmG0j/PsVPPW0+faep5F5X17KXy3sp+2Y3sh7K/t5N9VPZnbnhNtZlPPvmk7d9KfTwwv+zv7Du2N4wzO+yh/Myan1jz0JBNsZuLaX2BKron0fqCVweZKIGj29XzZ94yvFzU8rd7oq63pZ4nXX3BnTZv2cZyoe8qy+6O7wakWnlv9fvtblfpYtp2RTcwlOUWGVdvQ70finqajJ+2nnnV6+6znb6PWfsvy+sGxPozzfj6c8nwLGvW8jIs80f3GIgsK/NGPb57fESmK+vOMjO+dPMeO/X6allG3ves8fPqbvOiyvsuXfd9dJX/5e5nthPTPp9Zpu3jedTHQ9TvNV09rk/q43EV7yH7e9r/F182jDM77LG0KZyboTZFTqY7Ne0C1T2J1he8DK8v/FsFgXKRLsuqL2r52z1R19uS9c26yE6bt2xjN7QXZdnd8VlWHYpqZfvr95d1zJp+2nZFNzCU5RbdZdb7oainyfhp65nXtP3TRzt9H7P2X5bXDYj1Z5rx9eeS4RmW5dWfb5HxmT+6x0BkWeXYqsd3j4+oj4mtjtl6uqIcO/X6auV9zxo/r+42L6L8X9b7f9o+rU2bZ6emfT5Fmps8duxYc/ny5bZ/2j6eR308RLZ9J8tZpp2+lyL7rN5veY/rfk9jNowzO+yxq1evtif5TbGbi+m0k3qGlSAT9QUvw7sX/qy/DhnlwtW9iOV16c/09XIybX2R614samXaWj1v1tG90JR1lSBQZNp622tlmfW2lHWXIJG/ZVyWUy8702Z8d3szfbe/LCO6+6Zsc1ln9/NZVL3uPtvp+5i1//IZ1MdrPvt6HXld/69kGeVzqo/tIv1lPeUzrOevj738LcdAd72R9ZR56/m65jl26nnzumx3ll/Pu6juNi8i+6m7/7bTfW+7Ue//SHWaVLN89NFHm0OHDjVvvvnmZ7/O1p/FIrrvsXs8rMNO30ux2/lZrmGc2WGPpdmyPBEwN0ltgt1cTKedlDOsDj31Ba8OMkW5uJauXl49vBua8roeV29L+uuLbFe2qV52vb2RZdXji7KtRb0Npcs25G+9/vSXMFSvu7sv6nH1/N33mr9FWV+6+v2XYfXw6H4+i8ryhmCn72Or/dc9HurAmP76c+j+L5R5y7GX/lr9Oaerj4/6f6xMV8syy7qz3HIsTrPVsRP1uHr7M13dv6gsb6fKe+5ua9E9x2Tfl2F5XYZ393n5LNLV+7u7vHr8v/7rvzZf/epXm2effba9h6pMW46F+rOI+pjprr8MnzY+/d33W/ZDOT67+6X+bLuf1bRxWV99rOe4KdtQpi1dUe+zdOV9l/1QznFZbpbVfc+1jCvLqafLOspyyjT1vsiyy3vI33rcVsssw+vPekyGcWaHNTh9+nR7g9QmyEkM5jWU48Vxv1l2+3mUcF7CXFHCZB1Oowyvg136SwDM8G4YLMvoLi/rLEHw3//935s/+ZM/aV9HWU8Jt3XIzDz1ctJfvnBluvrLV9bf3da6K9Nm2emPel0ZXy8vr8s25293v0X9niPz1NtQLz8yriwzMm/ZlrIf6vGRZWR4vZ7IdFvtm3p7M65ebr3d9TZmeL39RYZ130d3e8bAGRF2aCg3yjI+QzleHPebZVmfR8JdllUCYB1ea92wHSXMlXHdLsvK+EyXRg/KebxeRxlfdNdTh8zu8tNlORlfh9boLjfT1kG01t0HkeXV60lXlpfX9X4oyv4osrx6G+r3Ut5nV5mmux9qZVw9f+mvu7KPs8x6u+p1d/ddd/1d9brrrt53Y7Gc/0BgrXICg3kN5Xhx3G+WZX4eJajlbx24a/U0RQmxZdw0L7/8cvMbv/Ebbdvy5SFRuwn19fqLbjCNZYT6WdPP2o69CvVFtrvsx62mzTLr7YqyrZl/2vuetX2zho+RvQAD4ITGIoZyvDjuN8tuPo+EuTrIJcSV5ZXXdfjM9CXM1cGxDrEJgyVgpkT+j/7oj5r/9//+X/Pwww+38/385z9vx0X6y7RlfWW52a66vw7CWV8dkjNtGZd56uCa+eppM75MW8s8mTbqabJ9ZXhkurL8adsRmace3t2G9NfbmHFlP0S9LdP2d709kfnLuvO6Xle9b7rrjfRn+qyjlmnr+ertK+vqDs/rejvHwhkRBqB7EoStDOV4cdxvlt1+HgliWUbp6lCWUFeGlyA5LWQmFNZhscyT7vd///fb5ikj09TjEg67obCMK6/LeuqQGemvl1PU21zG1SG3Hle6sl1l+aW/rLuE3nT19kY9btY+yDz1NmSbyriifj/18Gn7uwwrXXebZu2bDK+3sZi1jFn7ux5ehqWbtuwxcEaEAchJDOY1lOPFcb9ZfB6wXv4DYZfSvOW6W8FxMWURQj2rsO7PI23L51x8+/btyRAYF2dEWIKDBw8277333qRv7wk3LEKoZxXW9Xl88MEHzXPPPdc8+OCDzTPPPCPUM1rOiLAEefJgLibrItywCKGeVdjrzyMPhjp69Gizf//+5vz5883du3cnY2CcnBFhCXIxSSlRfv5dB+GGRQj1rMJefx4pkc+Nr6kCCQj1sDQnTpxoLly4MOnbW8INixDqWQWfB6yX/0BYktSpP3To0KRvb7mYsgihnlVY9ueRtuXffPPN9ryaqjbA1pwRYYk+/PDDyau9JdxMd+fOnckrakI9q7CszyPVak6fPt089NBDzbFjx1SxgTk5I8IA5GKq+3L3la98Zepw3XBCvW6zut1KyXzuT0qoX1chCfSVUA8M0vHjx9snD66zVSJgMWlsINVugMUJ9cDgvP32282BAwc+a5Xo5s2bkzHAJvA/Ccsn1AODkpK+hx9+uLlx40bb//rrrzdHjhxpXwPrk3rxV65caZ544on2f9RDomC5hHpYgTzh0E2a63Hq1KnmhRdemPT9sgWNRx99tHn33XcnQ4C9lF/M8nCoBPnc+Hr16lU3vsIKCPWwAmfOnGlv9GJvpXQ+waH7ELDLly83jz32mCABa5CbX1988cXm1q1bkyHAKgj1sAK5eO3bt0+I3EPZ19/97nfbUsBpHn/88TZcAMAQCfWwIocPH27bV2ZvvPzyy22LN7Pk4WApxdeyBixf6sefPXvW/xeskVAPK3Lp0qUtQybLU34Z2e4+htTnTd1eYDlyr8pTTz3VPigqVWy6Vd+AvSPUw4rcu3evvdDlJjFWK61pXLx4cdI3W5rRy2cieMDupBWb3ICeam3531NCD+sn1MMKpRRLgFytN954ow3183r22WfbEkVg53JTemk2FtgMQj3Qawnp0x5Xv1X3ta99zSPoARgUoR4YrAR4YHG5PyVN8+7fv1/VGugJVzxgsIR6WEyqDD799NPtvSd5kJu25aE/XPGAwRLqYX651+TgwYPtja+50R/oF1c82CMff/zx5BV7RaiH+almA/3migd74Nq1a82hQ4cmfewVoR6+7P3335+8AobEFQ/2wKeffto+zfSDDz6YDGEvCPXwS2la97XXXmsOHDjQHDlypD0nAcPiigd7JI9Qz41n7B2hnrFLqfxzzz3XPPjgg83JkyfbB7ABw+SKB3skTcSlRQn1VveOUM/YvfTSS8358+c92RpGwBUP9tCxY8eat956a9LHqgn1AIyFKx7soffee6+5evXqpI9VE+oZg9yInwdFAePmigcMllDPUOXG1wsXLrQ3vqZt+bwGxs0VDxgsoZ4hSj353Ph6/Pjx9tc/gHDFAwZLqGeIEuQ9zA7ocsUDBkuoB2AsXPGAwRLq6aM8pC5tyx8+fHgyBGB7rniwBnmaY25wS9v1rI5QT1/knJDmbp944olm//79zauvvtreDAswL1c8WJOUxOWhMKyOUE9fHDlypDl69Gjb5G0CPsCiXPFgTfL49pTWszpCPX1x7969ySuAnXHFgzU6dOhQc+PGjUkfyybUs0kS3P2/A6viigdrlAfGpBoOqyHUswly4+upU6fatuXzF2AVXPFgjXIj3IkTJyZ9LJtQzzpduXKlrSu/b9++9oFRbowHVskVDxgsoZ51On36dNuijRtfgb3gigcMllAPwFi44gGDJdSzSimBTxWb1157bTIEYH1c8YDBEupZhY8//rg5e/Zs8/DDDzdPPfVUc+3atckYgPVxxQMGS6hnmVIynxvbc+PrmTNnmtu3b0/GAKyfKx5siPyEn6dJsjxCPct2/fr15he/+MWkD2BzuOLBhkgrGceOHZv0sQxCPTsluAN944oHGyIhIg+n8ZP+8gj1LKLc+Hr06NHm2WefnQwF6AdXPNggedpkHlLDcgj1zCM3vp47d6698fXw4cPtr2ZK6oG+ccWDDZLHySdYeFjNcgj1bCfhPTe+Pvfcc83NmzcnQwH6xxUPNkweKy9cLIdQzzx8iQaGwBUPGCyhnuLDDz9s3n///UkfwPC44gGDJdTz7rvvtg+Ieuihh5oLFy5MhgIMjyseMFhC/TilOk0C/IEDB5pDhw41ly5dcuMrMHiueMBgCfXjlFCflqRu3LgxGQIwfK54wGAJ9QCMhSsebKjc2Hfx4sVJHzsh1A9X2pY/f/58c/ny5ckQgHFzxYMNdffu3fYJs5988slkCIsS6ocnVWpOnDjR/m/kqa95tgMAQj1stIQXLXbsnFA/HPmS+/jjjzf79+9vS+jTD8DnXPFgg6U5voMHD076WJRQPyzvvfeeB0UBzOCKBxsuJZMemrMzQn0/aX4SYHGueLDhUtUgzfOxOKG+P3LvyGuvvdY8+uij7V8AFuOKBxsuYUfJ5c4I9Zvv1q1b7ZfWPPE195BoWx5gZ1zxgMES6jdbmm19+OGHm3PnzrVNVAKwc654wGAJ9QCMhSseMFhC/WZIlZqUygOwOq54wGAJ9euT+0AuXbrUHDp0qG3B6dq1a5MxAKyCKx70yJUrVyavmIdQv/fu3LnTnD59un3i67Fjx5p33nlH2/IAe8AVD3okD6LKA6mYj1C/93LD64svvti2agPA3nHFgx65cOFC2+wf8xHqARgLVzzokbRZn2oNd+/enQxhK0L9auQJxydPnlRPHmCDuOJBzzzzzDPNq6++OuljK0L98uTG18uXLzeHDx9u25bPk461LQ+wOVzxoGfee++95sCBA5M+tiLUL8f169fbJ74ePXq0vVnbja8Am8cVD3oowZ7tCfXLkVL6Dz74YNIHwCZyxQMGS6hfTMK7UniAfnLFAwZLqJ/P7du327bl9+3b11a1AaB/XPGAwRLqt5YHQz311FNtffkzZ860D44CoJ9c8YDBEupnS6B//PHHm0uXLrXVbgDoN1c86LG0W6/N+tmEegDGwhUPeixVJk6dOjXpo2vsoT4l8G+99Vb75Q+AYRPqocdyg2OeMKv6xHRjDfWpG58vfLnxNW3Lf/jhh5MxAAyVUA89d+zYsbY0li8bW6i/efNm8/TTT7df9PILjrblAcZDqIeeyxM+jxw5MumjNrZQnxB/4cIF1W0ARkioh55L1Zs0SXjr1q3JEAo3ygIwFq54MACpfpP69XzREEP9u+++27Ytr015AGpCPTBYQwn1qU7z6quvNo8++mhz+PDh9kvcp59+OhkLAEI9MGBDCPVvvvlm88ADDzQnTpxo3n///clQAPgioR4YrCGE+lSz+fjjjyd9ADCdUA8MVp9CveAOwG4I9TAwHkT1uT6E+mvXrjXHjx9v25YX7AHYKaEeBiRPDs2NlPzSpob63Pia9uQPHDjQdnl97969yVgAWJxQDwNz8OBBN1RObGqoP3fuXPvk15TSA8AyCPUwMGn68Lnnnpv0jdsQbpQFgHm44sHApF526mfPqluf0D8W6wz1qWKT5igBYC8I9TBAqdoxLVAm8OcBRmOxjlD/wQcftL+UPPTQQ+1fdeUB2AtCPQxQ6mpfvHhx0ve5DH/qqacmfcO3l6H+vffea44cOdJ+aXrttdfaknoA2CtCPYxIWll58cUXJ33Dt5ehPl+Y3n333UkfAOwtoR5GJIF+Wgn+ULlRFoCxcMWDnkvp+5UrVyZ9Wzt27FhbTWQslhnqUzc++/ro0aPNp59+OhkKAJtBqIeeu3nzZrN///7mpZde2jZsZroxPbV0GaH+1q1bzalTp9obX7UtD8CmEuphAO7evduWIKckftYNmmniMk1djsluQ/3Zs2fbMH/mzJnmzp07k6EAsHmEehiIlNKfPn26bX0lzSoWae0mgTQl+ocOHZoMHYfdhvr8qjGrvX8A2CRCPQzM5cuXm3379n1Wzz5VR1LinP5nnnmmHTYW84b627dvT14BQD8J9TBAdT37//qv/2pfnzt3rjl//vxkinHYKtSnBD5fgPLrxeOPPz4ZCgD9JNTDQNX17H/v936v+eM//uO5W8kZimmhPlVq8mUnv2akapK25QEYAqEeBqzUs//mN7/ZPPDAA1+oaz8G00J92urPPlHlBoAhEephBP7t3/6t+epXvzq6mz53e6MsAPSFKx6sQcKmTjfGDoDVcIaFNRBuGCPHPcDqOMPCGgg3jJHjHmB1nGFhDYQbxshxD7A6zrCwBsINY+S4B1gdZ1hYA+GGMXLcA6yOMyysgXDDGDnuAVbHGRbWYBXh5sknn2xeeeWVSd/uZPs++uijSd9sb731VrvenXjkkUea69evT/pmy3RZz16bd/t2I/t4TEFXqAdYHWdYWIN5wk3CcqZLN09w7muozzaX91l3GV+mW3eof/7556du2251Q339mZduSIb2fgA2iTMsrMF24SYhMl2RULtdqXFfQ31t1rBNCPX155HXO33ftWmhflmf4U7lva1qG4R6gNVxhoU12C7cTAu32xHql2urUN8N4zsl1AOwLM6wsAbbhZuEu63CcsZlGenKdCUQJoyWcXUwz7gyPF0tQa4el67MW7+Osp7ohvr0T1tGdNefbpFQX89XZJlZf7p6eD1t2daYtZzYavu2C/Xdeet1Rv2ZpCv7pbucet/WMk29/vSX6bba59PGZb56WXmPGRdlP5aufv/18Cy3yHsr779e7jSZBoDVcIaFNZgn3JQgWAeoKCG2qwSyEuoSsErIyjLqwJX+sowEsqyrKEGwLKd+HZmvDpRlOQmA9Xalvyy3LLMoQbKExiLTTxtWz1uvv4TJep5MX29vvcx6H+R1/T622r56X0a2ofR35430Z3jU00Y9fbaznjfTpr909f5Mf7an3u7019Okv7vP630RmbfenvJei3r5kddlmVG2ueybjKvHb6VeDwDL5QwLazBvuCkBqkzfDYG1hLs6jCXUlcDXDYvpShDL3xLQiowvYbB+HfV66nUkDNbLL11kmhJyi2nrnTWsnrcOpXld1h8loHa7ev56X5TlbLd93fdW5otp85ZtnPV5lWV3x2dZ9WdYyzoybf1+u9tVupi2XVHvvyj7rMi4ehumfSb1NN3PZyv1egBYLmdYWINFw01CVLrdhPpZwWtaaMs6dhLq6/XXpq1/2npnDavnzTqyrvK6rD8yb6afJsvIeynLr5ez3faV/T/NtHnLsvci1M+aftp2Rf2+I9tRb0N3mdM+k3qa7uezlWn7AoDlcIaFNdgu3HSDaR32Mq4OZWV4NxAmaJUAmOH1OhPSyrRZVh0Uy7QlyGd9ZdoSQkt/vY4SOst8+Vu2M9PX76lM2w2LmWbasDo0Zln1cuttj3r7ot4H9X6r9+N225fp6nlrZdpaPW/WX8+b6cu65g31Zbr8racp6561z+tlZ9qM725vpu/219vb3Tf1tkT389lKvR4AlssZFtZgu3BTglPp6pAVCVJlXB0e60CYoFUCbZTwlq4eHvXyShgsoS3LL+MyXTdU1ssq85Zpa5mujMu2ZPy0AD9tWB0as46yP/K6+166+64sb9o+rffrVtvXnbarft/puiE3y6rHF2WbinobSpdl5W/Z52Weso6t9nk9bqv3mr9F/XnX778Mq4dH9/PZSuYFYDWcYWENhBvGyHEPsDrOsLAGwg1j5LgHWB1nWFgD4YYxctwDrI4zLKyBcMMYOe4BVscZFtZAuGGMHPcAq+MMC2sg3DBGjnuA1XGGhTUQbhgjxz3A6jjDwhoIN4yR4x5gdZxhYQ2EG8bIcQ+wOs6wsAbCDWPkuAdYHWdYWAPhhjFy3AOsjjMsrIFwwxg57gFWxxkW1kC4YYwc9wCr4wwLa5Bwo9ONsQNgNZxhAQCg54R6AADoOaEeAAB6TqgHAICeE+oBAKDnhHoAAOg5oR4AAHpOqAcAgJ4T6gEAoOeEegAA6DmhHgAAek6oBwCAnhPqAQCg54R6AADoOaEeAAB6TqgHAICeE+oBAKDnhHoAAOg5oR4AAHpOqAcAgJ4T6gEAoOeEegAA6DmhHgAAek6oBwCAnhPqAQCg54R6AADotab5/wjSxV4/Bw0yAAAAAElFTkSuQmCC)

线程池的足证部分都体现在线程池的构造函数中

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler);
```

- corePoolSize：核心线程数，也可以理解成常驻线程数，如newFixedThreadPool这个就为设置的nThread，对newCachedThreadPool则是0（因为会在无工作的时候缩容）
- maximumPoolSize：最大线程数，对newFixedThreadPool就是指定的nThread（固定大小了），对于newCachedThreadPool则是Integer.MAX_VALUE
- keepAliveTime和unit：共同决定额外的线程能够闲置多久，超过了就会回收
- workQueue：阻塞任务队列





线程池的问题：

- 避免任务堆积，如newFixedThreadPool就会造成OOM错误
- 避免过度扩展线程：有时候面对大量短期任务直接使用CachedThreadPool，因为很难明确一个线程数目
- 可能发生线程泄露（线程数不断增长）：可能是因为任务逻辑有问题，很多线程卡在了同一个位置迟迟不得释放
- 尽量避免在使用线程池时候使用ThreadLocal（因为工作的生命周期往往会超过线程的生命周期）



线程池大小的确定：

- CPU密集型任务：推荐设置线程数为N+1
- 等待较多的任务：线程数 = CPU 核数 × 目标 CPU 利用率 ×（1 + 平均等待时间 / 平均工作时间）









## 原子类的底层原理



原子类如AtomicInteger支持对其封装的数据的原子性的访问和更新操作，底层是基于CAS操作

例如AtomicInteger的getAndIncrease方法的底层实现可以导到Unsafe类的这段代码中来

```java
public final int getAndAddInt(Object o, long offset, int delta) {
    int v;
    do {
        v = getIntVolatile(o, offset);
    } while (!weakCompareAndSetInt(o, offset, v, v + delta));
    return v;
}
```

可以明显的看到do while这个CAS重试机制的具体使用



CAS的具体底层实现是基于CPU提供的特定指令的，在大多数情况下CAS是个非常轻量级的操作，这也是他的优势所在



问题：如何在实际场景中使用到CAS操作，毕竟直接调用Unsafe往往不是一个好的选择

如果想要使用，可以使用AtomicLongFieldUpdater，核心APi为：

```java
public abstract boolean compareAndSet(T obj, long expect, long update);
```

但是有个局限性，只能支持Long数据类型的操作，至于其他的数据操作就需要换了



在Java9以后可以使用VarHandle类中的方法：

```java
public final native
    @MethodHandle.PolymorphicSignature
    @HotSpotIntrinsicCandidate
    boolean compareAndSet(Object... args);
```

传参顺序依旧是obj，expect，update



不能过度依赖CAS，CAS的问题：

- 通常大部分问题只要重试一次即可成功，但是如果压力过大，重试次数过多，往往会造饥饿等问题
- ABA问题，加版本号即可解决，Java的实现类为：AtomicStampedReference 



往往不会直接与CAS打交道，而是通过与Doug Lea的JUC包打交道间接使用到了CAS



JUC包的基础：AbstractQueuedSynchronizer（AQS）

Doug Lea选择将基础的同步操作抽象在了AbstractQueuedSynchronizer当中去



对AQS内部结构和方法的简单拆分：

- 一个volatile的整型成员表征状态，提供get/set方法

```java
private volatile int state;
```

- 一个FIFO队列，实现线程之间的等待和竞争
- 基础基于CAS方法，如acquire/release方法，实现对资源的获取与释放



ReentrantLock的内部实现的简单抽象：

```java
public class ReentrantLock implements Lock, java.io.Serializable {
    // Sync是一个继承了AQS的内部类 
    private final Sync sync;   
    
    public void lock() {
        sync.acquire(1);
    }
    
    public void unlock() {
        sync.release(1);
    }
}
```



CountDownLatch中对AQS的利用页大差不差







## Java类加载过程



大概可以分为三个步骤：加载、链接、初始化



加载过程（Loading）：JVM将字节码数据从不同的数据源（jar文件，class文件，甚至是直接来源于网络的数据源等等）读入的过程，最终都会在JVM内部构建出Class对象（并没有直接读入到内存当中去，仅仅只是存储了Class的结构而已），如果不满足构成Class对象的规范，那么会抛出ClassFormatError异常。我们可以定义自己的类加载器来完成对加载过程的参与。



链接过程（Linking）：核心步骤，将原始的类定义信息平滑的转入JVM中，可以进一步分为三步：

- 验证（Verification）：虚拟机安全的保证，检测输入的类定义信息是否有害于JVM（主要还是因为class文件的结构开源导致的，为了让JVM支持不同的语言），如果有害就会抛出VerifyError异常
- 准备（Preparation）：创建类或接口中的静态变量并赋初始值
- 解析（Resolution）：将常量池中的符号引用替换为直接引用（因为内存地址已经定了）



初始化阶段（Initialization）：执行静态变量的赋值，类中静态代码块的运行



双亲委派模型：当前类加载器去试图加载某个类型的时候，除非父加载器找不到相应的类型，否则尽量将这个任务代理给父加载器去做，这样做的主要目的是为了避免重复加载Java类型



>  直接使用javap仅仅只会出现反编译之后的Java文件，不会出现字节码信息，如果要查看详细信息可以加上-c参数，查看全部信息使用-v参数







## 有哪些方法可以在运行时动态生成一个Java类



实际上就是动态代理的底层分析



我们使用到的动态代理（框架底层频繁使用，Java语言层面提供支持，我们只是简单使用）本质上就是等待特定的时机，去修改已有类型实现，或者创建新的类型。



那么，如果生成一个java文件了，如何将其编译成一个class文件以供JVM读取呢？

可以使用java.compiler

或者如果我们可以直接书写一个Class文件吗，难度太大



Proxy内部实现逻辑——内部类ProxyBuilder中的代码段：

```java
/*
 * Generate the specified proxy class.
 */
byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
    proxyName, interfaces.toArray(EMPTY_CLASS_ARRAY), accessFlags);
try {
    Class<?> pc = UNSAFE.defineClass(proxyName, proxyClassFile,
                                     0, proxyClassFile.length,
                                     loader, null);
    reverseProxyCache.sub(pc).putIfAbsent(loader, Boolean.TRUE);
    return pc;
} catch (ClassFormatError e) {
    /*
     * A ClassFormatError here means that (barring bugs in the
     * proxy class generation code) there was some other
     * invalid aspect of the arguments supplied to the proxy
     * class creation (such as virtual machine limitations
     * exceeded).
     */
    throw new IllegalArgumentException(e.toString());
}
```

这里就是ProxyBuilder内部读取Java文件并且通过ProxyGenerator编译成class字节流并交给Unsafe类进行读取成Class的过程



Java提供的动态代理，实现过程可以简化为：

- 提供一个普通接口作为共同接口，作为被调用类型和代理类之间的统一接口
- 实现InvocationHandler接口，实现其中的invoke方法，该方法为我们使用代理对象真正调用的方法
- 通过Proxy.newProxyInstance静态方法生成的代理对象，我们即可直接操作代理对象了





字节码操作技术除了动态代理还用在什么地方：

- Mock框架（测试框架）
- ORM框架
- IOC容器
- 部分Profiler工具
- 生成形式化代码的工具





## JVM内存划分



可以大致划分为几个方面：

- 程序计数器，每个线程都有且唯一，其中存储当前正在执行的Java方法的JVM指令地址
- Java虚拟机栈，简称栈，每个线程都有且唯一，内部保存栈帧，对应着一次次的方法调用。JVM对栈帧的操作只有入栈和出栈操作

栈帧中存储有：局部变量表、操作数栈、动态链接、方法正常退出和异常退出的定义等

- 堆：放置Java对象实例，被所有线程共享，可以通过Xmx等参数指定堆指标

堆内的空间会被不同的垃圾收集器细分

- 方法区：所有线程共享，存储元数据（类结构信息、运行时常量池、字段、方法代码等）

运行时常量池可以存放版本号、字段、方法、超类、接口等信息，还有就是存储常量池（存放各种常量信息，无论是编码的字面量还是运行时候决定的符号引用等）

- 本地方法栈：每个线程都会创建一个，类似于Java虚拟机栈。HotSpot虚拟机直接将本地方法栈与虚拟机栈使用同一块区域，JVM标准并未强制限定



![img](data:image/png;base64,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)

我这里简要介绍两点区别：

- 直接内存（Direct Memory）区域，它就是我在[专栏第 12 讲](http://time.geekbang.org/column/article/8393)中谈到的 Direct Buffer 所直接分配的内存，也是个容易出现问题的地方。尽管，在 JVM 工程师的眼中，并不认为它是 JVM 内部内存的一部分，也并未体现 JVM 内存模型中。
- JVM 本身是个本地程序，还需要其他的内存去完成各种基本任务，比如，JIT Compiler 在运行时对热点方法进行编译，就会将编译后的方法储存在 Code Cache 里面；GC 等功能需要运行在本地线程之中，类似部分都需要占用内存空间。这些是实现 JVM JIT 等功能的需要，但规范中并不涉及。







## 常见垃圾收集器



- Serial GC：最古老的单线程收集器，会出现Stop The World的状态

**对老年代采用了标记-整理算法，新生代使用复制算法**

开启参数为：`-XX:+UseSerialGC`



- ParNew GC：新生代的GC实现，实际是Serial GC的多线程版本，一般配合老年代的CMS GC使用

开启参数为：`-XX:+UseConcMarkSweepGC -XX:+UseParNewGC`



- CMS（Concurrent Mark Sweep） GC：老年代GC，基于标记-清除算法，目的是尽量减少停顿时间



- Parallel GC：JDK8早期版本中的默认使用GC，特点是新生代和老年代GC都是并行的

开启参数为：`-XX:+UseParallelGC`



- G1 GC：兼容吞吐量和停顿时间的GC实现，JDK9之后默认的GC选项

G1 GC仍然存在着年代的划分，但是将内存直接划分成一个个Region，推荐使用G1 GC



可以使用参数：`java -XX:+PrintCommandLineFlags -version`来查看Java使用的GC



其中GC使用的算法在《深入理解Java虚拟机》中有





## Java应用程序在Docker中的运行部署



Docker看起来类似于虚拟机，拥有自己的shell、能够独立安装软件包、运行时候与其他容器互不干扰，但是后来你会发现Docker不是一种虚拟化技术而是一种隔离技术



Docker仅仅在类Linux内核上实现了有限的隔离和虚拟化，而不是像传统虚拟软件那样，独立运行出一个新的操作系统。运行在Docker之上的多个程序只需要像调用操作系统API那样来操作Docker就可以获取资源，基本不存在兼容性改变问题



Java程序运行在Docker环境的问题：

- JVM会在启动时候检测内存大小，设置堆的内存起始大小为1/64，最大堆内存大小为1/4。
- JVM会检测CPU核心数目，会直接影响到Parallel GC，JIT Compiler甚至是ForkJoinPool的执行



如何解决这些问题呢？升级到最新的JDK即可解决问题。

从JDK10开始就开始完善了，完全可以自动化实现内存和CPU盒数的检测了



如果无法更新JDK版本，可以限制堆，元数据区的大小，明确制定可用CPU核数









## Java应用开发的安全



注入类攻击是源于程序允许攻击者将不可信的动态内容注入到程序中并将其执行，这就可能改变最初预计的执行过程并产生恶意效果

场景：



- SQL注入攻击

如果只是简单的生成SQL语句通过JDBC让MySQL去执行，如：

`Select * from use_info where username = “input_usr_name” and password = “input_pwd”`

传入参数：`“ or “”=”`

于是拼接出来的SQL为：`Select * from use_info where username = “input_usr_name” and password = “” or “” = “”`

这种情况下MySQL一定会返回True

期望输入数值，但是用户实际输入了SQL语句片段，这就导致了问题的产生，甚至可能加上Delete语句



- XML注入攻击

Java提供了工具操作XML文件，如果使用不当可能导致恶意访问





Java自身提供的安全检查：

- 运行时安全机制

主要是类加载过程中的验证，利用SecurityManager机制，限制代码的运行时行为能力

- JDK提供的安全工具

keytool：管理密钥、证书等

jarsigner：对jar包的签名和认证等



达到攻击需求也未必需要绕过各种权限设置，直接使用哈希碰撞也可以达到攻击的目的，模拟大量相同HASH值的数据，通过JSON方式发送到服务器当中去，会使得算法的复杂度上升一个数量级，从而导致严重的性能退化





## MySQL事务隔离级别和锁实现原理



隔离级别：在数据库事务中，为保证并发数据读写的正确性而提出的定义

各家的关系型数据库都提供了自己的事务隔离级别，按照隔离级别由低到高，MySQL事务隔离级别可以分为：

- 读未提交：一个事务可以看到其他事务未提交的修改，非常低的隔离级别，可能出现脏读。
- 读已提交：事务能够看到的数据都是其他事务以及提交的修改，虽然不会出现脏读，但是这仍然是比较低的隔离级别，并不保证在读取两次可以获取到相同的数据，也就是允许其他事务并发修改数据，可能有不可重复读和幻读的出现。
- 可重复读：保证同一个事务中多次读取到的数据是一致的，也就是保证在事务执行过程中其他事务无法进行数据修改，也是InnoDB的默认隔离级别，可以简单认为不会出现幻读。
- 串行化：最高的隔离级别，在读取过程中需要获取共享读锁，在更新数据的时候需要获取排他写锁，如果存在WHERE字句，还会获取区间锁如：行锁等。



至于乐观锁和悲观锁，不是MySQL或者是数据库独有的概念，而是并发编程的基本概念。区别在于：

- 悲观锁认为数据出现冲突的可能性较大
- 乐观锁认为数据出现冲突的可能性较小

悲观锁一般是由SELECT......FOR UPDATE 语句导致出现的锁，防止数据被意外修改。乐观锁则是使用的CAS机制，不会直接对数据进行加锁，而是对比数据的时间戳或者是版本号来保证并发程序的正确执行。



MVCC本质可以看做乐观所的一种实现，而读写锁，双阶段锁则可以认为是悲观锁的实现



主流ORM框架的对比：

- Hibernate

一个JPA Provider，以对象为中心，屏蔽底层SQL语句，使用HQL语言来进一步屏蔽对底层数据库之间的差异，降低维护数据库的成本，内部大量使用Lazy-Load来提升性能，同时提供了强大的持久化功能。但是缺点也相当明显，HQL需要额外的学习成本，数据库管理人员无法方便的对SQL进行优化，隔绝了与数据库层的交互。

- MyBatis

以SQL为中心，开发者更加侧重于SQL和底层数据库，仅仅提供了半自动如数据封装等功能，支持较高的自定义化。

- Spring JDBC Template

更加倾向于SQL层面，是对原生JDBC的简单封装