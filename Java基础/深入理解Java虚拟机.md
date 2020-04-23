# 简介



Java技术体系主要是由：Java虚拟机，Java类库，Java编程语言以及第三方Java框架组成



因为Java的一个重要优点：Java虚拟机在各个平台上建立了统一的运行平台，使得普通的开发人员只需要了解Java常用类库，基本语法，第三方框架即可完成大部分日常工作



如果开发人员不了解虚拟机的技术特征的运行原理，就无法写出最适合虚拟机运行和自优化的代码



由于OracleJDK在OpenJDK中占据绝对优势，所以本书大多以HotSpot为例子进行讲解



由于版面原因，本书中的许多示例代码都没有遵循最优的程序编写风格，如使用的流没有关闭
流、直接使用System.out输出日志等，请读者在阅读时注意这一点。



2.网站资源
·高级语言虚拟机圈子：http://hllvm.group.iteye.com/。
里面有一些关于虚拟机的讨论，并不只限于Java虚拟机，包括了所有针对高级语言虚拟机（High-
Level Language Virtual Machine）的讨论，不过该网站针对Java虚拟机的讨论还是绝对的主流。圈主
RednaxelaFX（莫枢）的博客（http://rednaxelafx.iteye.com/）是另外一个非常有价值的虚拟机及编译原
理等资料的分享园地。
·HotSpot Internals：https://wiki.openjdk.java.net/display/HotSpot/Main。
这是一个关于OpenJDK的Wiki网站，许多文章都由JDK的开发团队编写，更新很慢，但是有很大
的参考价值。
·The HotSpot Group：http://openjdk.java.net/groups/hotspot/。
HotSpot组群，里面有关于虚拟机开发、编译器、垃圾收集和运行时四个邮件组，包含了关于
HotSpot虚拟机最新的讨论。







# 第一部分、走进Java





## 第一章、走进Java



由于Java的热点代码检测和运行时编译及优化，使得Java程序能随着运行时间的增长而获取更高的性能



Java技术体系：

从广义上讲，Kotlin、Clojure、JRuby、Groovy等运行于Java虚拟机上的编程语言及其相关的程序
都属于Java技术体系中的一员。

从传统意义上讲：Java程序设计语言，Java虚拟机实现，Class文件格式，Java类库API，第三方Java类库



在不引起歧义的地方都能 使用JDK（Java Development Kit）来代替整个Java技术体系，JDK是支持Java程序开发的最小环境，包括：Java语言，Java虚拟机，Java类库

可以把Java类库中的子集——Java SE API 和Java虚拟机两部分统称为JRE（Java Runtime Environment），JRE是支持Java程序运行的标准环境

![image-20200421111659200](images/image-20200421111659200.png)



如果按照重点业务来划分的话：

- Java Card：支持Java小程序（applets）运行在小内存设备（智能卡）上
- Java ME：支持Java程序运行在移动终端上的平台
- Java SE：支持面向桌面级应用（如Windows的应用程序）的Java平台，在JDK6之前被称为J2SE
- Java EE：支持使用多层架构的企业应用的Java平台，并在Java SE的基础上做出了拓展（Javax.*包，Java SE是Java.*包，由于历史原因，一部分javax进入到了Java SE API中），在JDK6之前被称作J2SE，在JDK10被Oracle放弃，捐赠给了Eclipse基金会，改名Jakarta EE





Java历史（书籍p25）：

1995年5月23日，Oak语言改名为Java，并且在SunWorld大会上正式发布Java 1.0版本。Java语言第
一次提出了“Write Once，Run Anywhere”的口号。

1996年1月23日，JDK 1.0发布，Java语言有了第一个正式版本的运行环境。JDK 1.0提供了一个纯
解释执行的Java虚拟机实现（Sun Classic VM）。JDK 1.0版本的代表技术包括：Java虚拟机、Applet、
AWT等。



Sun公司在JDK7研发期间股票大跌，无力推动JDK7的研发，JDK7中就包含lambda表达式等特性，最后Sun公司被Oracle公司收购，并推迟了技术 



JDK 8提供了那些曾在JDK 7中规划过，但最终未能在
JDK 7中完成的功能，主要包括：
·JEP 126：对Lambda表达式的支持，这让Java语言拥有了流畅的函数式表达能力。
·JEP 104：内置Nashorn JavaScript引擎的支持。
·JEP 150：新的时间、日期API。
·JEP 122：彻底移除HotSpot的永久代。



从此以后，每六个JDK大版本中才会被划出一个长期
支持（Long Term Support，LTS）版，只有LTS版的JDK能够获得为期三年的支持和更新，普通版的
JDK就只有短短六个月的生命周期。JDK 8和JDK 11会是LTS版，再下一个就到2021年发布的JDK 17了。



Oracle收购Sun是Java发展历史上一道明显的分界线。在Sun掌舵的前十几年里，Java获得巨大成
功，同时也渐渐显露出来语言演进的缓慢与社区决策的老朽；而在Oracle主导Java后，引起竞争的同时
也带来新的活力，Java发展的速度要显著高于Sun时代。Java的未来是继续向前、再攀高峰，还是由盛
转衰、锋芒挫缩，你我拭目以待。
Java面临的危机挑战前所未有的艰巨，属于Java的未来也从未如此充满想象与可能。







Java虚拟机家族：



虚拟机始祖：Sun  Classic/Exact VM

JDK1.0中自带的虚拟机，唯一的功能就是以纯解释器的方式来运行Java代码，如果需要及时编译的功能，就必须外挂编译器，但编译器会完全接管虚拟机的执行系统，解释器将毫无用武之地。正因为编译器与解释器不能共存，使用编译器就不得不对每一个方法每一行代码都进行编译，效率很低，解释器就更不用说了。



在JDK1.2时候，sun公司提供了另一种虚拟机来改善Classic VM的低效率问题——Exact VM，允许编译器与解释器共存，还支持热点嗅探等操作。虽然性能较Classic VM优化了很多，但立马就被外部引进的HotSpot VM淘汰（并不是技术上的原因，而是公司的内部争吵决策），反而是Classic VM，在JDK1.2之前是唯一的VM，在JDK1.2，他是默认的虚拟机选择，在JDK1.3时，HotSpot成为默认选择，Classic VM成为备选，直到JDK1.4才被淘汰。



武林盟主：HotSpot虚拟机

sun公司在1997年收购了Longview Technologies公司，从而获得了
HotSpot虚拟机，HotSpot不是用的Java语言开发

在Oracle公司收购Sun后，选择将BEA的JRockit的优秀点融入到HotSpot中并在JDK8中发布，移除到了永久代，加入了Java Mission Control 监控工具等等





小家碧玉：Mobile/Embedded VM

面向移动端和嵌入式的虚拟机产品



目前位置比较尴尬，在移动端Android和IOS二分天下

而在嵌入式设备上，Java ME Embeddad  VM面对自家的Java SE Embeddad VM的竞争，人们更愿意选用 SE的产品，Oracle基本砍掉了这一块，归入了Java SE Embeddad VM中，Java SE Embeddad VM 本质上还是HotSpot，只是为嵌入式设计进行了定制，并减少了内存消耗和资源占用



反倒是很早就应该被淘汰的更低端的Java ME VM活的更好，在国外的老人机和经济欠发达的功能手机还在广泛使用





天下第二：BEA JRockit/IBM J9 VM

曾经的三足鼎立关系，JRockit和J9都曾宣称自己是世界上最快的虚拟机，总体上三者虚拟机之间的性能是交替上升的







对未来的一种期盼：

无语言倾向的Graal VM，在后面介绍到



及时编译器

HotSpot里存在两个及时编译器用于编译热点代码，分别是：

- 编译时间短但输出代码优化程度较低的客户端编译器（简称C1）
- 编译时间长但输出代码优化程度较高的服务器编译器（简称C2）

自JDK10起，加入了一种新的即时编译器Graal（到如今仍是非常年幼，没有见过足够的实战）企图在将来取代C2。C2是Cliff Click博士在博士期间的C++作品，但睡着C2的发展就连Cliff Click本人都不愿意维护。而Graal比C2晚问世20多年，并且使用了一种名为“Sea-of-Nodes”的高级中间表示，可以很轻易的借鉴C2的优势，具有更高的可扩展性，在测试中的数据逐渐赶上C2，在有些方面还超越了。

> Graal可以作为18年Oracle提出的Graal VM的基础
>
> Graal VM被官方称为“Universal VM”和“Polyglot VM”，这是一个在HotSpot虚拟机基础上增强而成
> 的跨语言全栈虚拟机，可以作为“任何语言”的运行平台使用，这里“任何语言”包括了Java、Scala、
> Groovy、Kotlin等基于Java虚拟机之上的语言，还包括了C、C++、Rust等基于LLVM的语言，同时支
> 持其他像JavaScript、Ruby、Python和R语言等。Graal VM可以无额外开销地混合使用这些编程语言，
> 支持不同语言中混用对方的接口和对象，也能够支持这些语言使用已经编写好的本地库文件。
>
> ![image-20200421154435988](images/image-20200421154435988.png)
>
> 





向native迈进：



由于Java的启动时间相对较长，需要预热才能达到最高性能等特点，使得对几年在从大型单体应用架构向小型微服务应用架构发展的技术潮流不太适应

已经陆续推出了跨进程的、可以面向用户程序的类型信息共享（Application Class Data
Sharing，AppCDS，允许把加载解析后的类型信息缓存起来，从而提升下次启动速度，原本CDS只支
持Java标准库，在JDK 10时的AppCDS开始支持用户的程序代码）



但提出的一个彻底的解决方案（目前正在实行）：提前编译

直接给JVM运行已经编译好的字节码，使得不用去在线编辑，避免了Java第一次运行事假慢的问题。

但坏处也非常明显：1.有悖于Java“一次编写，到处运行“的承诺，开始以来操作系统。2，显著降低Java的动态性，必须要求信息在静态编译的时候就是已知的，而不是运行时候才确定。

直到Substrate VM出现，才算是满足了人们心中对Java提前编译的全部期待。但相应地，原理上也决定了Substrate VM必须要求目标程序是
完全封闭的，即不能动态加载其他编译器不可知的代码和类库。

Substrate VM具有轻量级的特性，在运行时候不会占用过多的内存，并且速度客观





语言语法持续增强：

但一门语言的功能、语法又是影响语言生产力和效率的重要因素，很多语言特性和语法糖不论有没有，程序也照样能写，但即使只是可有可无的语法糖，也是直接影响语言使用者的幸福感程度的关键指标。

改进期盼值关注度高的项目：

- Loom：Java的线程都是直接调度本地方法，间接依赖于操作系统，对线程的操作太过重量级（早期Java也提供了一套轻量级的线程操作）Loom项目就准备提供一套与目前Thread类API非常接近的Fiber实现。
- Valhalla：提供值类型和基本类型的泛型支持，并提供明确的不可变类型和非引用类型的声明。不可变类型在并发编程中非常重要，Java只能通过将类中的全部字段声明为final来保证
- Panama：目的是消弭Java虚拟机与本地代码之间的界线，Panama项目的目
  标就是提供更好的方式让Java代码与本地代码进行调用和传输数据。







自己编译JDK

以OpenJDK为例子，在linux环境下编译











# 第二部分、自动内存管理



## 第二章、Java内存区域与内存溢出异常



C/C++ 程序员在内存管理领域拥有最高的权限，但也需要担负着每一个对象生命从开始到终结的维护者

Java程序员将内存管理权利交给了Java虚拟机，但一旦出现内存泄漏和洗出方面的问题，一旦不了解虚拟机就很难修复





根据《Java虚拟机规范》，Java虚拟机所管理的内存将会被划分成为以下几块：

![image-20200422131941909](images/image-20200422131941909.png)



组件的介绍：

- 程序计数器

较小的内存空间，可以被看成当前线程所执行的字节码的行号指示器，是用来选取下一条需要执行的字节码指令。

是程序控制流的指示器，分支，循环，跳转，异常处理，线程恢复等等都依赖于这个计数器

每个线程都会拥有一个程序计数器便于获取CPU时间片的时候恢复到正确的位置，并且计数器之间互不影响，独立存储。这类内存区域被称为“线程私有”的内存

如果执行的是Java方法，则存储的是下一条指令的地址，如果执行的是本地方法，则该计数器为null。是唯一不会出现OutOfMemoryError的地方



- 虚拟机栈

描述的是Java方法执行的线程内存模型，每个方法对应着一个栈帧，方法的调用到结束就对应着栈帧从虚拟机栈中入栈到出栈的全过程，栈帧中存放有：局部变量表（就是方法体之内定义的变量），操作数栈，动态链接，方法出口等等

线程私有的，生命周期与线程相同



不能笼统的将Java内存划分成为堆内存和栈内存，这种划分方式来源于C/C++，无法适用于Java。但这也间接说明了程序员们最关注的对象分配关系最密切的区域：堆和栈，**栈这里通常指的是虚拟机栈，或者更多情况下只是用来指虚拟机栈中的局部变量表部分**

局部变量表存放：基本数据类型，对象引用（reference类型，不同于对象本身，可能是指向对象起始地址的引用指针等）和returnAddress类型（指向字节码的地址）

局部变量表的存储空间由局部变量槽来表示，long和double（64位）会占用两个局部变量槽，其余的数据只占用一个。~~可以这样理解：一个局部变量槽可以存储32位的数据，且只能单独存放一个数值~~（有问题，后面有解释）。局部变量表的分配在编译期间就可以确定的。

> 局部变量表的大小指的是：
>
> ​	局部变量槽的数目，而不是局部变量槽的大小，他的大小由虚拟机自行决定的事情。



会抛出两种异常：

1. StackOverflowError异常：请求的栈深度大于虚拟机所允许的深度，就会抛出该异常
2. OutOfMemoryError异常：如果虚拟机能动态扩展，在需要扩展的时候却无法申请到足够的内存就会抛出该异常





- 本地方法栈

与虚拟机栈发挥相同的功效，只不过虚拟机栈服务于Java方法（分配内存，存储局部变量等等），而本地方法栈服务于本地方法

《Java虚拟机规范》对本地方法栈的使用方式和数据结构没有明确规定，有些虚拟机（例如HotSpot）因为他和虚拟机栈的功能尤其类似，直接合并到了虚拟机栈中去了

和虚拟机栈一样，也会抛出OutOfMemoryError和StackOverflowError异常





- Java堆

Java堆是虚拟机管理的内存中最大的一块，存在的唯一目的就是：存放对象实例。几乎所有的对象实例（包括数组）都在这里分配内存。

Java堆是垃圾回收器管理的内存区域，有时也被称为“GC堆”



分类：

1.从回收内存的角度看

对Java堆的一点补充：

> 从回收内存的角度看，由于现代垃圾收集器大部分都是基于分代收集理论设计的，所以Java堆中经常会出现“新生代”“老年代”“永久代”“Eden空间”“From Survivor空间”“To Survivor空间”等名词，这些概念在本书后续章节中还会反登场亮相，在这里笔者想先说明的是这些区域划分仅仅是一部分垃圾收集器的共同特性或者说设计风格而已，而非某个Java虚拟机具体实现的固有内存布局，更不是《Java虚拟机规范》里对Java堆的进一步细致划分。不少资料上经常写着类似于“Java虚拟机的堆内存分为新生代、老年代、永久代、Eden、Survivor……”这样的内容。在十年之前（以G1收集器的出现为分界），作为业界绝对主流的HotSpot虚拟机，它内部的垃圾收集器全部都基于“经典分代”[3]来设计，需要新生代、老年代收集器搭配才能工作，在这种背景下，上述说法还算是不会产生太大歧义。但是到了今天，垃圾收集器技术与十年前已不可同日而语，HotSpot里面也出现了不采用分代设计的新垃圾收集器，再按照上面的提法就有很多需要商榷的地方了。

摘自：《深入理解Java虚拟机（第三版）》



2.从内存分配的角度看：

会为每个线程划分其私有的分配缓冲区，以提升对象的分配效率。



==无论从什么角度，都不会改变堆的共性，存储的都是对象实例，将Java堆划分只是为了更好的进行垃圾回收，或者更快的分配内存==



Java堆可以固定大小，也可以动态分配（指定``-Xmx 和 -Xms`），如果内存满了，就会抛出OutofMemoryError异常





- 方法区

用于存放虚拟机已加载的类型信息，常量，静态变量，即时编译器编译出的代码缓存等等



方法区，永久代，元空间的关系：

永久代和元空间只是方法区的两种不同的实现方式

尤其在JDK8之前，HotSpot没有实现方法区，而是使用对堆划分一块名为永久代的内存在在逻辑层面上实现方法区的概念，这样就可以直接使用堆的内存回收机制来管理这一块内存，而永久代有内存上限的限制，会出现很多BUG，而BEA的JRockit和IBM的J9则没有永久代这一概念，当Sun收购BEA时候，想整合HotSpot和JRockit，就因为此出现诸多困难，因此选择逐步使用元空间来替代永久代

JDK7成功将永久代的字符串常量池，静态变量等移除

在JDK8时候废除永久代，采用元空间，并具有自己的垃圾回收算法



如果无法满足内存分配，就会抛出OutOfMemoryError异常





- 运行时常量池

运行时常量池从属于方法区。方法区对类加载文件（Class文件）会进行内存分配，Class文件包含类的版本，字段，方法，接口等描述，还有常量池表（里面存储的是各种字面量和符号引用），常量池表在类加载后存放到方法区的运行时常量池中

上面只是类的静态编译的处理方式，运行时常量池也支持对动态内存分配来支持动态编译所带来的常量，这种特性被利用的多的地方就是String类的intern（）方法

```java
//描述：调用intern方法时，如果池中包含一个equals返回true的对象，那么就直接返回这个池中对象，否则就将当前字符串加入池中，并返回引用
public native String intern();
```



因为运行时常量池从属于方法区，和方法区一样也可能抛出OutOfMemoryError异常





- 直接内存

不是虚拟机运行时数据区的一部分，也不是《Java虚拟机规范》的一部分，因为频繁的被使用，并可能==**导致**==OutOfMemoryError异常



Java1.4新加入了NIO（New Input/Output）类，可以使得Native函数直接分配堆外内存，并将这块地址的引用返回给DirectByteBuffer对象，显著提高性能，避免Java堆和Native堆来回复制数据



堆外内存没有限制大小，但是随着堆外内存的增大，其他的内存区域动态扩展时候很有可能会抛出OutOfMemoryError异常（可以通过给堆外内存设置-Xmx等参数来解决问题）





上面介绍完了内存区域，下面深入探讨一下HotSpot虚拟机在Java堆中对象分配、布局和访问的全过程。



对象的创建：

创建对象（例外：复制和反序列化）通常仅仅只需要一个new关键字，但在JVM层面，对象（不包括数组和Class对象）的创建详细详细：



当JVM遇到new指令

1.会先到常量池中检查能否定位到这条指令的参数，并且检查这个符号引用所代表的类是否被加载，解析和初始化过，如果没有，则进行类加载过程

2.为新生对象分配内存，内存大小在类加载过程中便已经确定。分配内存的任务又由Java堆内存是否工整决定的。如果是工整的，就会存在一个指针作为分界点的显示器，隔绝被使用的线程和空闲的线程，此时的分配仅仅只需要挪动指针向空闲内存方向即可分配空间，这种分配方式被称为“指针碰撞”。而如果是不工整的，虚拟机就会维护一个列表，上面记录着空闲内存和被占用内存，并从空闲内存中找到一块足够大的空间交给对象实例，这种分配方式被称为“空闲列表”。影响Java堆是否工整的因素是垃圾回收器，当使用Serial、ParNew等带压缩整理过程的收集器时，系统采用的分配算法是指针碰撞，既简单又高效；而当使用CMS这种基于清除（Sweep）算法的收集器时，理论上就只能采用较为复杂的空闲列表来分配内存。

> 需要考虑的问题：并发
>
> 可能分配内存后，指针还没来得及挪动就又开始了第二次分配内存
>
> 可选方案：
>
> 1. 内存分配的动作进行同步——实际上Java虚拟机就是采用CAS来保证内存分配操作的原子性
> 2. 在Java堆上为每个线程都分配一份私有的本地线程分配缓冲（Thread Local Allocation Buffer，TLAB），直接在自己的本地缓冲区分配，只有当本地缓冲区满了之后，在扩展本地缓冲区的时候进行同步即可，虚拟机是否使用TLAB，可以通过-XX：+/-UseTLAB参数来设定。



3.将分配到的内存空间（不包括对象头）都初始化为零，如果是TLAB分配的话就直接在分配本地线程区的时候顺便进行，这样能使得对象的实例字段在不赋初值的情况下能直接使用，对应的就是各自的零值（基本数据类型）或者NULL（引用）

4.对对象进行必要的设置，并将信息存储在对象头中，例如：类的元数据信息，对象的哈希码（实际上会延后到调用hashcode方法的时候再计算），对象的GC分代年龄信息等等

到此为止就Java虚拟机方面对象已经分配完成了，但是就程序员来说还没有，所有的字段还是零值

5.调用<Init>()方法，Java编译器在new关键字之后会紧接着调用new指令和init指令用于初始化，整个初始化工作完成





对象的内存布局：

在HotSpot，对象内存布局分配三部分：对象头（Header），实例数据（Instance Data）和对齐填充（Padding）

- 对象头

对象头包含两类信息，一类用于存储对象自身的运行时数据，被称为“Mark Word”，主要有哈希码（应该是懒加载的），GC分代信息，锁标记状态，线程持有的锁等等只启用一个标记状态记录下来，分配的比特位为虚拟机的比特位，空间极小，所以实现了高效的利用（几个比特位共同管理几种状态），状态码部分信息如下：

![image-20200422201828238](images/image-20200422201828238.png)



对象头的另一部分数据是类型指针，指向该对象的元数据类型（但也不是必须存在的，即Java虚拟机获取对象类型不一定要通过对象的元数据信息），此外，如果是一个数组，还需要存储记录数组长度的数据，这样Java虚拟机才能在类加载的时候确定该对象（把数组看成一个对象）需要分配多大的内存空间。



- 实例数据

对象真正的有效信息，存储我们定义的各种类型的字段内容，无论是从父类继承下来的，还是在子类中定义的字段，HotSpot的默认存储顺序为：longs/doubles、ints、shorts/chars、bytes/booleans、oops（Ordinary Object Pointers，OOPs），且同一层级的变量父类定义的会在子类定义之前，且相同长度的字段会被分配到一起存放（减少内存开销）。



- 对其填充

不是必须的，但由于HotSpot要求对象的起始地址是8字节的整数，所以对象的大小都必须是8字节的整数倍，不满8字节则用此来补充





对象的访问定位：

Java程序是操作栈上的reference数据来操作堆上的具体对象，《Java虚拟机规范》只是指定了该reference存储的是对象的引用，具体怎么访问没有说明，由虚拟机自己实现，主流访问方式有：

- 句柄访问

Java堆划分出一片内存作为句柄池，reference存储的就是句柄地址，在通过句柄值来找到对象的信息，示例图如下：

![image-20200422203841653](images/image-20200422203841653.png)

对象中的静态全局变量存储在方法区中，非静态全局变量存储在堆区，要同时读取两部分内容



- 直接指针访问

reference直接存储的是对象堆内存的地址，会少读取一次内存

![image-20200422204044568](images/image-20200422204044568.png)

都需要读取对象头的元数据类型来获取静态全局变量





各有优劣：

句柄存储能保存reference存储的是稳定的句柄地址，如果对象内存修改了，只需要改变句柄地址即可

直接指针能保证高速的访问速度（对HotSpot中主要是使用第二种方式，第一种也很常见，打开任务管理器即可看到句柄数量）







实战：OutOfMemoryError异常



1.模拟堆溢出：

```java
/**
 * @author Lixiang(LuckyCurve)
 * @date 2020/4/22 20:48
 * @Desc 模拟OutOfMemoryError异常
 */
public class Test {
    static class OOMObject {
    }

    public static void main(String[] args) {
        ArrayList<OOMObject> list = new ArrayList<>();

        for (; ; ) {
            list.add(new OOMObject());
        }
    }
}
```

虚拟机参数：`-Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError`

通过将最大内存和最小内存置为一样保证其无法扩展，通过-XX:+HeapDumpOnOutOfMemoryError保存内存堆转换快照以便后面分析

输出：

```java
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid12888.hprof ...
Heap dump file created [28167387 bytes in 0.079 secs]
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3210)
	at java.util.Arrays.copyOf(Arrays.java:3181)
	at java.util.ArrayList.grow(ArrayList.java:265)
	at java.util.ArrayList.ensureExplicitCapacity(ArrayList.java:239)
	at java.util.ArrayList.ensureCapacityInternal(ArrayList.java:231)
	at java.util.ArrayList.add(ArrayList.java:462)
	at cn.luckycurve.vmdemo.Test.main(Test.java:18)
```

前三行输出为指定记录堆内存快照独有的

> 使用测试工具Jprofiler来进行分析：
>
> 1.![image-20200422211713931](images/image-20200422211713931.png)
>
> 指定运行环境，选择最右边的按钮
>
> 2.![image-20200422212346966](images/image-20200422212346966.png)
>
> 主要将其设置为CPU recording和Save and immediately open a snapshot
>
> 启动CPU记录和并保存一个快照，保证一开始的数据就有效，以及当虚拟机被迫中断之后数据不会丢失
>
> 3.如果是一个立马关闭的程序，或许当CPU记录还没启动起来JVM已经停止了，例如上面那个分配了20M内存的程序
>
> ![image-20200422212916003](images/image-20200422212916003.png)
>
> ![image-20200422212925721](images/image-20200422212925721.png)
>
> 在第二章图里面很容易看出是哪个对象造成了内存泄漏
>
> ![image-20200422213356142](images/image-20200422213356142.png)
>
> 中间只存在0.01s，持续时间非常的短（一般不会出现这种情况，因为虚拟机有足够大的内存）
>
> 





2.模拟栈异常：

HotSpot不会单独区分虚拟机栈和本地方法栈，所以虽然-Xoss参数存在，但是设置本地方法栈大小不会有任何效果，直接使用-Xss来设定栈容量大小即可

《Java虚拟机规范》中提出栈可能出现两种异常：

- StackOverflowError：请求的栈深度大于栈的最大深度
- OutOfMemoryError：支持动态扩展的虚拟机如果请求内存扩展无效时抛出（很遗憾HotSpot不支持动态扩展，理论上是很难出现这个异常，也有可能出现：当线程申请分配栈内存的时候无法获取足够内存，也会抛出这个异常）



栈中存放的是栈帧，也叫方法帧，用于记录运行时候的方法信息，最简单的模拟栈溢出的方法就是：方法去调用自己，类似于一个递归函数但是没有递归出口，栈帧会一直记录外层方法，直到栈帧大到虚拟机栈装不下了，就报错，例子如下：

1.

```java
/**
 * @author Lixiang(LuckyCurve)
 * @date 2020/4/23 11:08
 * @Desc 模拟栈异常
 */
public class JavaVMStackSOF {
    private int stackLength = 1;

    public void stackLeak() {
        stackLength++;
        stackLeak();
    }


    public static void main(String[] args) {
        JavaVMStackSOF javaVMStackSOF = new JavaVMStackSOF();
        try {
            javaVMStackSOF.stackLeak();
        } catch (Throwable e) {
            System.out.println(javaVMStackSOF.stackLength);
            throw e;
        }
    }
}
```

虚拟机参数：`-Xss128k`，指定栈大小，快速模拟出溢出效果，并输出方法调用的次数



>  还有一种方法和上面例子达到的效果是一样的：
>
> ​		不控制栈的大小，保证每个方法的方法帧足够大（创建100个局部变量，使得方法帧足够大），目的是一样的，但第一种方式优雅些





模拟HotSpot的OutOfMemoryError异常（通过一直创建线程，并保证每个线程的栈内存大小足够大（通过-Xss参数指定分配的栈大小），就很容易出现了）

```java
/**
 * @author Lixiang(LuckyCurve)
 * @date 2020/4/23 11:40
 * @Desc 模拟HotSpot抛出OOM异常（很难见到的，不支持动态内存分配）
 */
public class JavaVMStackOOM {
    public static void main(String[] args) {
        for (; ; ) {
            new Thread(() -> {
                //    保证线程一直执行执行，占用物理内存
                for (; ; ) {

                }
            }).start();
        }
    }
}
```

虚拟机参数：`-Xss2m`

:warning:：在Windows上由于Java的线程会直接映射到操作系统的内核线程上，操作系统会有很大的压力，很可能造成假死，运行上述语句有很高风险（亲测风险很高）



通常出现StackOverflowError会有明确的信息可供你分析，容易定位到问题信息。

但如果是因为过多的线程导致的OutOfMemoryError，很有可能是线程脱离了管理，GC无法回收（类似于野指针的形式），也有可能只是单纯的线程太多，此时则需要来减少堆内存大小和栈容量（默认情况下大多数达到1000~2000是没有问题的）来换取更多的线程了。





3.方法区与运行时常量池溢出

先来测试运行时常量池的溢出（使用String.intern方法来不断添加对象到常量池里面去）。

在JDK6及以前，运行时常量池在永久代，使用`-XX：PermSize`和`-XX：MaxPermSize`限制永久代的大小，即可达到内存溢出目的。

在JDK7及以后，开始使用元空间取代了永久代，此时使用`-XX：MaxPermSize`限制永久代还是在JDK8以后使用`-XX：MaxMeta-spaceSize`限制元空间都无济于事，**因为运行时常量池移动到了堆区**，使用`-Xmx`来指定堆区的大小

```java
/**
 * @author Lixiang(LuckyCurve)
 * @date 2020/4/23 14:28
 * @Desc 运行时常量池的OOM异常
 */
public class RuntimeConstantPoolOOM {
    public static void main(String[] args) {
        //防止发生垃圾回收
        HashSet<String> set = new HashSet<>();
        int i = 0;
        for (; ; ) {
            set.add(String.valueOf(i++).intern());
        }
    }
}
```

虚拟机参数：`-Xmx6m`



> 对String的intern方法的测试：
>
> ```java
> public static void main(String[] args) {
>         String a = new StringBuilder("计算机").append("科学").toString();
>         System.out.println(a.intern() == a);
> 
>         String a2 = new StringBuilder("计算机").append("科学").toString();
>         System.out.println(a2.intern() == a2);
>     }
> ```
>
> 输出结果：`true		false`（Java8环境下）
>
> 





方法区的OOM测试：

方法区存放的是：类名，访问修饰符，常量池（不是运行时常量池），字段描述，方法描述等，产生OOM的基本思路是使用大量的类去填满方法区，可以借助反射或动态代理等等，但比较麻烦

笔者借助了CGLib直接操作字节码运行时生成了大量的动态类。

当前的很多主流框架，如Spring、Hibernate对类进行增强时，都会使用到CGLib这类字节码技术



在JDK7之前，包括JDK7使用此方法再加上限制持久代大小很容易使得方法区抛出OOM，但是在JDK8使用了元空间，上述代码很难使得虚拟机产生方法的溢出异常了（会自动进行类的卸载和装载）





4.直接内存溢出

主要是NIO操作本地内存导致的

> 由直接内存导致的内存溢出，一个明显的特征是在Heap Dump文件中不会看见有什么明显的异常情况，如果读者发现内存溢出之后产生的Dump文件很小，而程序中又直接或间接使用了DirectMemory（典型的间接使用就是NIO），那就可以考虑重点检查一下直接内存方面的原因了。





小结：

本章讲述了虚拟机内存划分以及可能导致的内存溢出异常

下一章讲解垃圾回收机制为了避免这些异常做了哪些努力







## 第三章、垃圾收集器与内存分配策略



垃圾回收（Garbage Collection，GC）并不是Java独有的，早在1960年Lisp语言就开始使用了



现在了解垃圾回收机制的必要性：排查内存溢出，内存泄漏等问题，或者当垃圾收集器称为并发性能的瓶颈时候，我们需要手动调整和监控垃圾收集器的参数



在线程私有的程序计数器，虚拟机栈和本地方法栈三部分中，与线程的生命周期相同，栈中的栈帧随着方法的进入而生成，随着方法的退出而毁灭，且每个方法需要分配的栈帧是已知的（在编译器即可知道），当方法结束时候，内存也就自然而然跟着回收了（相对来说动态性没那么强，大部分信息在编译期已经可以知道，其余的来源于编译器的优化，影响不大）



但堆和方法区则相对而言会具有显著的不确定性（动态性），任何时间任何类的任何方法都可能发出一个new指令来创建对象，无法预先确定对象生成的时间与需要销毁的时间以及对象的大小，垃圾收集器的回收重点也是这一部分







垃圾收集器需要保证堆内的对象已死（再无法被任何途径获取并使用）才进行回收，保证对象已死的方法如下：



1.引用计数算法（Java主流虚拟机都没有采取）

给每个对象添加一个引用计数器，当对象被引用就加一，引用失效了就减一，为零的时候对象就不可能再被使用了

优点：原理简单，判断效率高

缺点：占用额外的内存来计数，需要大量的额外处理才能保证可用性（例如对象之间的循环引用问题，如下）：

```java
ReferenceCountingGC objA = new ReferenceCountingGC();
ReferenceCountingGC objB = new ReferenceCountingGC();
objA.instance = objB;
objB.instance = objA;
```

假设ReferenceCountingGC有一个Object类型的instance字段，由于objA和objB相互引用，外界已经不可能访问到他了，但是计数器不为零，就是一个很好的引用计数算法不可解决的例子（需要额外处理）



