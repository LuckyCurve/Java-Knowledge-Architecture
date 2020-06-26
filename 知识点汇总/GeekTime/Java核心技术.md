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



|  参数  |              意义               |
| :----: | :-----------------------------: |
| -Xint  |      告诉JVM只进行解释执行      |
| -Xcomp | 告诉JVM关闭解释器，启动会非常慢 |
|        |                                 |
|        |                                 |
|        |                                 |
|        |                                 |
|        |                                 |
|        |                                 |
|        |                                 |
|        |                                 |
|        |                                 |
|        |                                 |
|        |                                 |
|        |                                 |
|        |                                 |
|        |                                 |
|        |                                 |
|        |                                 |
|        |                                 |





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