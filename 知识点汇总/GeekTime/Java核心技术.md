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

![img](data:image/png;base64,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)

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