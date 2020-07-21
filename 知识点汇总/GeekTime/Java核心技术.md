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