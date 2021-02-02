# 面试常见题目

> **wait和notify的实现是依赖于Monitor的，因此这些方法在调用时候需要持有这个对象的monitor，否则会报错。**
>
> ```java
> synchronized (obj) {
>     // 等待其他线程获取到obj并调用notify或者是notifyAll
>     obj.wait();
> }
> ```
>
> ![img](https://img-blog.csdn.net/20161019204824590?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
>
> 

## 一、并发基础



进程和线程

进程是程序的一次执行过程，是系统资源调度的基本单位，在Java中，当我们启动main函数就启动了一个JVM进程，main函数所在的线程从属于JVM，被称为主线程

线程是轻量级进程，是一个更小的执行单位，且同一个进程下的线程可以完成对堆和方法区的数据共享，但也独享如：程序计数器，本地方法栈，虚拟机栈等内存空间

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-3/JVM%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA%E5%9F%9F.png)

Java就是天生的多线程，一般每个进程都包含主线程和垃圾回收线程。

其实线程共有的还有一部分：就是直接内存（NIO直接操作的部分）

各部分简单介绍及可能抛出的异常：

程序计数器：存储当前线程执行字节码的行号，用于控制线程的顺序执行，循环、分支选择等

是唯一一个不会抛出OutOfMemoryError异常的地方

Java虚拟机栈：内部存放的是一个个栈帧，每个栈帧代表着一次方法调用，栈帧中包括：局部变量表，操作数栈、方法出口等信息。可能抛出StackOverflowError（当虚拟机栈不支持动态扩展）或者是OutOfMemoryError（虚拟机栈支持动态扩展）

本地方法栈：调用本地方法时候产生一个个栈帧存放的地方，和虚拟机栈类似，异常和虚拟机栈一模一样

堆：用于存放对象实例，按照GC操作可以分为老年代和新生代，新生代又可以分为Eden和两个Survivor，总体比例为8：2，每次只能使用一个Survivor，每次GC时候Eden清空，Survivor替换，当Survivor内部对象存在多次就到了老年区里面去了

堆只会报OutOfMemoryError错误

方法区：存储已加载的类信息，常量，静态变量，即时编译代码数据，HotSpot的实现是直接在堆内开辟一块空间来作为方法区。

方法区只会报OutOfMemoryError错误

（运行时常量池在1.7从方法区中移出到堆中）



关于这里可能出现的问题：

程序计数器为什么私有：每个线程都需要记录自己运行到哪里，以便在线程在分配到CPU时间片的时候知道从哪里继续执行（如果执行native方法，程序计数器为undefined ）

虚拟机栈和本地方法栈为什么是私有：这两个问题合二为一的原因是HotSpot将虚拟机栈作为了本地方法栈的实现了，实现了两者的合并，主要是为了保证局部变量表的私有，不被其他线程访问到



并发与并行的区别：

并发：一段时间内，多个任务共同执行

并行：一个时间点上，多个任务共同执行



为什么要使用多线程

线程较之于进程，线程间的切换是远小于进程的，另外，多核CPU意味着可以同时执行多个线程，多线程可以更高效得利用硬件资源

多线程是高并发系统的基础，用于解决百万级请求。

当然，多线程带来了诸多问题如：内存泄露、死锁、线程安全性等问题



线程状态

NEW RUNNING WAITINT BLOCKING TIME_WAITING TERMINATED

之间的转换关系：

NEW ——Thread#start——> RUNNING

RUNNING ——Object#wait/Thread#join——> WAITING/TIME_WAITING

WAITING/TIME_WAITING ——Object#notify/notifyAll——> RUNNING

RUNNING ——try get synchronized——> BLOCKING

BLOCKING ——get synchronized——> RUNNING

RUNNING ——FINISH——> TERMINATED

> 几个注意的点：调用Thread.join后，调用线程会等待被调用线程执行完毕之后再去执行，即调用线程会被阻塞，内部使用的是Object#wait方法
>
> RUNNING是包含运行中和就绪两种状态的，JVM将其合二为一，运行中想要转换为就绪可以调用Thread.yield方法建议JVM挂起该线程，就绪到运行中为该线程获取到CPU时间片



上下文切换

随着科技的发展，CPU可以同时处理多个线程，采取的策略是时间片轮转的方式，得到时间片的线程拥有执行权，当时间片用完之后，当前线程会由运行中转换成就绪状态，这个过程叫上下文切换

上下文切换可能是操作系统中消耗时间最大的操作



什么是死锁，死锁如何避免

死锁就是已经获取到对方需要的资源，而在同时等地啊对方释放资源的一种系统状态

死锁的四个条件：

1、互斥条件，该资源同一时刻只能由一个线程占用

2、部分分配：先申请一次资源，对已持有资源不放再去申请另一份资源

3、不可剥夺条件：没有执行完就不会释放当前持有资源

4、循环等待条件：形成头尾相接的循环等待资源条件

避免死锁：避免其中一个就可以了

可以破坏的为2（一次性申请所有资源）3（如果申请不到资源就主动释放）4（按照顺序申请资源，释放资源则反序释放）



sleep和wait方法的区别：

- wait方法会释放锁，sleep方法不会释放锁
- wait是用于进程间交互的，需要其他线程调用同一个对象上的nitify方法来让该线程重新回到RUNNING状态（超时的就不算了），sleep方法则是必须传入超时时间，只能通过时间的方式自省

相同点：都可以暂停当前线程的执行，让线程进入WAITING或者是TIME_WAITING



Thread#run和start的区别

调用run方法不会以多线程的方式执行，仅仅是在main线程上执行Runnable的内容，如果是start方法则会执行相应的线程准备工作，让线程从NEW到RUNNING状态



Synchronized关键字

Synchronized关键字可以保证被他修饰的方法或者代码块在同一时间只有一个线程可以执行

Java在早期版本中的锁属于重量级锁，是直接由操作系统的monitor来实现的，每次试图去持有锁的时候都会发生用户态——内核态的切换，非常耗时

Java在1.6就JVM层面对synchronized进行了优化如：自旋锁、适应性自旋锁、锁消除、锁粗话、偏向锁、轻量级锁等技术来减小了锁的开销

synchronized锁定的内容不同，如果标注在静态方法上锁定的就是当前类，如果是标注在非静态方法上锁定的就是当前对象，如果标注在代码块上则需要指定锁定的是什么，可以是类也可以是对象，尽量不要将String作为锁，因为String在JVM中具有缓存功能

手写单例模式：

```java
public class Singleton {
    private volatile static Singleton singleton;
    
    public static Singleton getInstance() {
        if (singleton == null) {
            synchronized(singleton) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
                return singleton;
            }
        }
    }
}
```

使用volatile主要是防止以下三个步骤进行了重排：

1. 分配内存空间
2. 执行初始化操作（清零）和构造函数
3. 返回引用



构造方法不能用synchronized关键字来修饰，Java自身具有初始化锁，无需再加入synchronized来保证线程安全。



synchronized关键字的底层原理是基于JVM层面的，如果是同步代码块则是使用MonitorEnter和MonitorExit分别指向同步代码块的开始位置和结束位置，实际上就是针对对象监视器（monitor）的持有权，锁中持有计数器，只有当计数器为0的时候别的线程才可以试图去持有，每获取一次这个锁该计数器就+1，释放锁则-1。

如果锁定的是一个对象，本质上是对对象头中的MarkWord进行CAS操作

**wait和notify的实现是依赖于Monitor的，因此这些方法需要在（持有对象锁的）同步代码块和非静态方法中，否则会报错。**

如果是方法上声明了synchronized，经过反编译可以看到方法上加上了ACC_SYNCHRONIZED的标识，从而实现方法的同步调用

本质都是对象监视器monitor的获取



JDK1.6对synchronized的优化

存在非常多种的状态如：偏向锁、轻量级锁、自旋锁、适应性自旋锁、锁消除、锁粗化等技术来减少锁操作的开销，状态标识都处于对象头的MarkWord当中

主要存在四种状态，随着竞争的激烈依次升级：无锁状态——偏向锁状态——轻量级锁状态——重量级锁状态，锁只能升级不能降级，为了提高获取锁和释放锁的效率（主要是锁降级效率比较低，频繁升降会影响性能）

无锁应该就是没有线程来尝试申请该资源，所以不需要锁定

偏向锁是针对一个线程而言的，之所以设计偏向锁，是因为大多数条件下都是一个线程对锁的访问，使用偏向锁可以大量减少锁获取和释放的时间，但如果有两个或者是多个线程来竞争锁，锁就会升级成轻量级锁。

轻量级锁是基于CAS来实现的，CAS操作的目标就是对象头的MarkWord，如果失败咋进行自旋操作，如果自旋次数过多，则会进行锁升级（实质上就是怼MarkWord里面字段的修改）

重量级锁则是直接由monitorenter和moniterexit接管，进入到重量级锁时期

适用场景对比：

偏向锁：适用于只存在一个线程访问该资源的场景

轻量级锁：适用于锁竞争不激烈的时候，追求响应速度

重量级锁：适用于锁竞争激烈的时候，追求高吞吐量



synchronized和ReentrantLock的联系与区别

都是可重入的

synchronized由JVM层面实现，而ReentrantLock由语言层面实现

ReentrantLock增加了一些高级功能：

- ReentrantLock可中断等待，如lockInterruptibly和带时间的tryLock
- 可以实现公平性：通过构造函数的boolean，会返回一个ReentrantLock的静态内部类
- 锁可以绑定多个条件（有点忘记有啥用了）

```java
//采用和synchronized一样的锁获取方式，无法中断
void lock();
//可中断的锁获取操作，比synchronized定制性高
void lockInterruptibly() throws InterruptedException;
//仅当锁空闲的时候获取锁
boolean tryLock();
//在指定时间尝试获取锁，可以响应中断
boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
//释放锁，常用在finall块
void unlock();
//下一章单独介绍Condition
Condition newCondition();
```

如果使用synchronized只能配合wait和notify实现线程的等待与唤醒，但是使用ReentrantLock可以创建多个condition，每个Condition都可以完成对线程的等待与唤醒操作，实现更加细粒度的线程通知管理



volatile关键字

volatile告诉JVM这个变量是不稳定的，每次使用他都必须到主存中进行读取，而不是从线程私有的本地内存中读取，所以volatile不仅能防止指令重排还能保证变量的可见性

![JMM(Java内存模型)](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/2020-8/0ac7e663-7db8-4b95-8d8e-7d2b179f67e8.png)



并发编程的三大特性

原子性（synchronized）、可见性（volatile和synchronized）、有序性（volatile）

synchronized和volatile之间的区别

volitable只能修饰变量，synchronized可以用于方法以及代码块

volatile解决的是多个线程间数据的可见性，synchronized也可以解决这个问题，但是是通过让线程之间串行执行来保证后执行的线程可以看到先执行的线程的修改



ThreadLocal

保证每个线程都存在一份自己的专属变量，存放于线程的本地内存当中，不共享，从而实现了线程安全。

创建ThreadLocal的时候需要给定默认值，此后才是get方法set方法获取默认值或者是将其值更改为本线程所存的值

原理：每个Thread内部都持有一份ThreadLocalMap，实际数据存储于此，我们使用ThreadLocal时候只是将其作为代理，存取数据都是直接映射到了本地线程的ThreadLocalMap上，ThreadLocalMap的结构为`<ThreadLocal, Value>`，Value就是我们通过ThreadLocal#set进来的值

ThreadLocalMap的内存泄露问题：ThreadLocalMap内部持有的ThreadLocal是软引用，而Value是强引用，如果GC了Key，存在一个key为null的Entry，那么该Value永远不会被GC掉。当然ThreadLocalMap已经考虑到了，在增删查的时候会手动清理到key为null的记录

> 强引用：只要强引用存在，指向的对象一定不会被回收
>
> 软引用：只有在内存要溢出的时候才会进行回收
>
> 弱引用：只要发生GC，这个对象就会被回收
>
> 虚引用：最弱的引用，唯一的作用是用队列接收对象即将死亡的通知

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

[ThreadLocalMap更加细致的讲解](https://gitee.com/SnailClimb/JavaGuide/blob/master/docs/java/multi-thread/%E4%B8%87%E5%AD%97%E8%AF%A6%E8%A7%A3ThreadLocal%E5%85%B3%E9%94%AE%E5%AD%97.md)



线程池技术

线程池就是一种标准的池化技术的实现，类似于数据库连接池，Http连接池等，实现了对资源的统一管理，使用线程池之后，可以减少每次新建线程和回收线程的开销，提高了资源利用率

好处（也是池化资源的好处）：

降低资源消耗、提高响应速度、提高线程的可管理性



Runnable和Callable之间的区别：都标注了@FunctionalInterface，支持Java8的函数式编程，Runnable是没有返回值的，也不能抛出异常，Callable是有返回值的，并且可以抛出异常



线程池的execute和submit方法的区别

execute用于提交不需要返回值的任务，因此无法判断是否执行成功

submit用于提交需要返回值的任务，任务会返回一个Future类型的对象，可以通过Future对象判断是否完成，调用get方法获取，如果没有执行完则会阻塞，可以使用带时间的get方法，超时自动停止等待



创建线程池

阿里巴巴开发规范中指明了不允许使用Executors去创建，应该通过使用ThreadPoolExecutor的方式去创建，因为可能导致如下问题：

FixedThreadPool（固定线程数量的线程池）和SingleThreadPool（只有一个线程的线程池）：允许请求队列长度到达Integer.MAX_VALUE

CachedThreadPool（根据实际情况调整线程数量的线程池）和ScheduledThreadPool：允许创建的线程数量达到Integer.MAX_VALUE

都可能导致OOM，因此需要自己实现

```java
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler);
```

几个参数：

corePoolSize：核心线程数量

maximumPoolSize：最大线程数量

workQueue：请求队列

<hr>

keepAliveTime：非核心线程的存活时间，没事情干之后

unit：keepAliveTime的单位

threadFactory：创建新线程的时候使用到

handler：饱和策略

- `ThreadPoolExecutor.AbortPolicy`：给新任务抛出RejectedExecutionException异常并拒绝处理（默认）
- `ThreadPoolExecutor.CallerRunsPolicy`：让调用线程自己去执行这个任务
- `ThreadPoolExecutor.DiscardPolicy`：直接丢弃新任务
- `ThreadPoolExecutor.DiscardOldestPolicy`：丢弃最早的未处理的任务请求（应该是队列的最前面）

![图解线程池实现原理](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-7/%E5%9B%BE%E8%A7%A3%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86.png)



Atomic原子类

原子类简而言之就是具有原子/原子操作特性的类

原子类主要包含四大类：

基本类型如：AtomicInteger、AtomicLong、AtomicBoolean

数组类型如：AtomicIntegerArray、AtomicLongArray、AtomicReferenceArray

引用类型如：AtomicReference、AtomicStampedReference（带有版本号的引用类型，解决ABA问题）、AtomicMarkableReference（带有标记位的引用类型）

对象的属性修改类型：AtomicIntegerFieldUpdater（更新原子整型字段的更新器）、AtomicLongFieldUpdater（原子更新长整型字段的更新器）、AtomicReferenceFieldUpdater



主要是AtomicInteger的使用和原理

主要是如下方法：

```java
public final int getAndSet(int newValue)//获取当前的值，并设置新的值
public final int getAndIncrement()//获取当前的值，并自增
public final int getAndDecrement() //获取当前的值，并自减
public final int getAndAdd(int delta) //获取当前的值，并加上预期的值
public final boolean compareAndSet(int expect, int update) //如果输入的数值等于预期值，则以原子方式将该值设置为输入值（update）[主要]
```

主要使用的是volatile关键字保证修改时候线程的可见性还有对native方法的调用（native方法实现了CAS）









## 二、乐观所与悲观锁



悲观锁也就是每次拿数据的时候都会认为别人会修改，所以需要锁定。大部分都是这个实现，如synchronized，reentrantlock，行锁，表锁，读写锁

乐观锁则认为别人不会修改数据，但是在更新数据的时候会去判断在此期间是否有人修改（通过版本号机制和CAS来实现），用于多读的场景，可以明显提高吞吐量，具体实现有java util下的原子类，使用到了CAS技术（自旋锁，顾名思义，遇到问题就不断的重试）

使用场景比较：

乐观锁适合于读多写少的场景，这样可以免去很多获取锁和释放锁带来的开销（主要是RUNNING与BLOCKING状态转换的开销），悲观锁适合于写多的场景，因为频繁的写入，乐观锁就会需要不断的去重试，导致线程进入长时间的循环等待，消耗CPU。



CAS算法，Compare And Swap：

个人认为是处理器厂商底层提供的一种技术，可以让比较并交换这两个操作直接具备原子性，一次性完成，中途不会被打断。只有当值相等的时候才会进行更新操作，否则重试，存在ABA问题。解决方法就是增加一个版本号或者是时间戳用于比较



synchronized在1.6之前被称为重量级锁，在1.6之后对效率进行了改良，减少性能消耗引入了偏向锁和轻量级锁的各种优化，牺牲了公平性，但是获取了效率上的提升。





## 三、AQS

AQS，全名AbstractQueuedSynchronizer，是一个用来构建锁和同步器的框架，例如ReentrantLock和Semaphore都是基于此实现的

**核心思想：如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态，如果请求资源被占用，就会使用FIFO队列来阻塞线程，将获取不到锁的线程加入到该队列当中**

![AQS原理图](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/AQS%E5%8E%9F%E7%90%86%E5%9B%BE.png)

通过维护一个int成员变量来表示同步状态，每次视图改变变量状态（获取锁和释放锁）就会使用CAS操作来保证操作的正确性。



AQS定义资源的两种共享方式：

独占方式：只能有一个线程可以占有资源

独占方式又分为公平的和非公平的，如果是公平的独占方式，AQS就会按照队列中的排队顺序，先到者先拿到锁，如果是非公平的方式，无视顺序直接去抢占锁，通过CAS改变同步状态，谁抢到就是谁的

共享方式：多个线程可以同时执行，如CountDownLatch、Semaphore、ReadWriteLock等

共享方式的实现各有不同，只需要考虑同步状态怎么改变就好了，至于队列的维护则是交个AQS实现



AQS的底层是同步器，类似于JdbcTemplate，其中大量使用到了模板方法模式

当我们继承AQS的时候，只需要书写对同步变量的处理即可

由于AQS模板方法模式的加持，会在模板方法内部调用我们书写的对同步变量的处理逻辑，从而设计出一个类用于管理并发线程

```java
isHeldExclusively()//该线程是否正在独占资源。只有用到condition才需要去实现它。
tryAcquire(int)//独占方式。尝试获取资源，成功则返回true，失败则返回false。
tryRelease(int)//独占方式。尝试释放资源，成功则返回true，失败则返回false。
tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。
```



AQS实现的几个组件：

Semaphore：两个核心方法：release和acquire，用于凭证的增加和减少

CountDownLatch：核心方法为：countDown和await，用于减少计数和等待计数器为0

CyclicBarrier：可循环屏障，让一组线程到达同步点，同步点为await方法所在位置。可不断循环



CountDownLatch

当CountDownLatch内部持有的计数器大于零的时候，线程调用await方法就会被阻塞，当其他线程调用CountDown方法，计数器减一，直到减少零的时候，所有被阻塞的方法都会直接释放，继续执行

可能用到的场景：

假设用户执行一个批量操作，如同时上传5个文件，就可以使用CountDownLatch让这五个线程全部执行完之后一起返回