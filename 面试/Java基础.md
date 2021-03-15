# 面试常见题目

> 内部类和静态内部类的区别：编译完成后非静态内部类会保存一个引用指向他的外围类，但是静态内部类不会有。
>
> 静态成员变量存放于方法区当中，在HotSpot中方法区的实现是永久代或者是元空间（Java8为界），划分一部分堆空间出来由Java垃圾回收器共同管理，但是容易造成内存泄露问题，元空间直接使用直接内存
>
> 在Linux操作系统中，线程本质上就是一个进程，线程这个概念确实是windows平台独有的，JVM进行了Linux环境下的适配
>
> 为什么LinkedList在1.7的时候取消了循环？【双向循环链表】
>
> 1、代码可读性更好，first和last的概念更清晰
>
> 2、first节点中可以少存放一个pre的引用，last节点
>
> 3、最重要的一点，在头尾插入可以更高效（毕竟LinkedList的特点就是在频繁的插入和删除）
>
> 为什么是双端的？
>
> 主要移除最后一个元素后，last节点可以快速获取到倒数第二个元素，高效删除

## 一、Java基本概念与常识



Java语言有哪些特点？

面向对象的三大特性、平台无关性、可靠性、安全性、支持多线程（JavaScript单线程语言，前期的C/C++也是单线程的，11之后添加多线程）、支持网络编程。



什么是字节码，采用字节码的好处是什么？

JVM可以理解的代码被称为字节码，通过字节码这一形式，解决了传统解释型语言运行效率低下的问题，同时保留了其可移植的特点。一套字节码可以直接在多种操作系统的计算机上使用。

> 扩展： .java --> .class  --> 机器码，重点关注.class到机器码这一步
>
> 刚开始都是解释器进行解释执行的，但是随着Java的发展，效率要求愈发变高，便引进了JIT编译器去编译一些被HotSpot判定（惰性评估Lazy Evaluation）的热点代码成机器语言，一旦成为机器码之后便会被保留，以便下次直接使用，但是需要一定时间的预热带来的时间开销。
>
> 为了适应微服务这个大环境，而不是传统的单体服务，JDK9引入了新的编译模式AOT（Ahead Of Time Compilation），直接将字节码编译成机器码，避免了JIT预热开销
>
> AOT的编译质量肯定比不上JIT编译器的。

附带的几个概念（标准化）：

JVM虚拟机是运行Java字节码的机器，Java针对不同系统的特定实现（Windows，Linux，macOS），目的是使用相同的字节码可以达到同样的效果，字节码和不同系统的JVM实现是Java语言“一次编译，到处运行”的关键所在。

JDK是Java Development Kit缩写，它是功能齐全的Java SDK，拥有JRE所拥有的一切，同时还拥有编译器（javac）和工具（javadoc），能够创建和编译程序。

JRE是Java Runtime Environment的缩写，是运行已编译Java程序所需的所有内容的集合，包括Java虚拟机，Java类库，Java命令等一些基础组件，但是他不能用于创建新程序。

> 一般安装JRE就可以运行了，但也有特例，JSP，JSP需要编译成Servlet来运行，因此JSP的运行时依赖于JDK的。



Oracle JDK和Open JDK的对比

在Java7之前基本没有什么区别，OpenJDK在7的时候作为Java 7的参考实现被Oracle工程师维护

对比：

|                        Oracle JDK                         |             Open JDK             |
| :-------------------------------------------------------: | :------------------------------: |
|                  六个月发布一次主要版本                   |      三个月发布一次主要版本      |
|             OpenJDK的一个实现，不是完全开源的             | 作为一个参考模型并且是完全开源的 |
| 更加稳定，在OpenJDK的代码基础上加入了一些类和错误修复机制 |                                  |
|     响应性和JVM性能方面提供了一些扩展，具有更好的性能     |                                  |
|    不会为发行的版本进行长期支持，需要用户升级以便适配     |                                  |



Java和C++的区别与联系（即使只问了区别，也可以回答些联系）

- 都是面向对象的语言，支持封装继承多态
- Java不提供指针直接访问内存，安全性更高
- Java类是单继承的，C++是多继承。虽然类不能多继承，但接口可以实现多继承（extends多个接口，Spring ApplicationContext）
- Java有自动内存管理回收机制（GC），不需要程序员手动释放
- 在C/C++中字符串和字符数组都有一个字符`'\0'`来表示结束，但是Java没有



java包和javax包有什么区别？

刚开始Java API所必需的包都是java开头的，javax在当时还只是被当做扩展包来使用。然而随着时间的推移，javax逐渐扩展成为java API的组成部分，但是将javax移动到java内部可能会造成破坏性的修改，因此最终决定单独javax也作为java 标准API的一部分。



为什么说Java是编译与解释共存的语言？

主要是Java文件需要编译成class文件，最后再经过虚拟机解释执行，因此被说为是编译与解释执行共存的语言。



字符型常量和字符串常量的区别

|           字符型常量           |           字符串常量           |
| :----------------------------: | :----------------------------: |
|      单引号引起的一个字符      | 双引号引起的零个或者若干个字符 |
| 相当于一个整数值，可以参与运算 |        代表一个地址空间        |
|            两个字节            |            若干字节            |

> Char的封装类Character有public static属性SIZE大小为16，转换为字节为2字节
>
> 其他封装类也有这个属性



谈谈你对注释的认识

有三种类型：单行注释、多行注释、文档注释（带有Javadoc标签的多行注释）

在编译期间会被编译器直接删除，只存在于源文件.java中

注释能够帮助看代码的人快速地理清代码之间的逻辑关系，快速上手编写代码



标识符和关键字之间的区别

简单来说，标识符就是一个名字，通常用于指定一个对象或者是基本的数据类型，但是存在一些标识符，Java给予了特殊的含义，这些特殊的标识符就是关键字

常见关键字：

|                      |          |            |          |              |            |           |        |
| -------------------- | -------- | ---------- | -------- | ------------ | ---------- | --------- | ------ |
| 访问控制             | private  | protected  | public   |              |            |           |        |
| 类，方法和变量修饰符 | abstract | class      | extends  | final        | implements | interface | native |
|                      | new      | static     | strictfp | synchronized | transient  | volatile  |        |
| 程序控制             | break    | continue   | return   | do           | while      | if        | else   |
|                      | for      | instanceof | switch   | case         | default    |           |        |
| 错误处理             | try      | catch      | throw    | throws       | finally    |           |        |
| 包相关               | import   | package    |          |              |            |           |        |
| 基本类型             | boolean  | byte       | char     | double       | float      | int       | long   |
|                      | short    | null       | true     | false        |            |           |        |
| 变量引用             | super    | this       | void     |              |            |           |        |
| 保留字               | goto     | const      |          |              |            |           |        |



continue，break和return的区别是什么？

在循环结构中，当循环条件不满足的时候循环自动结束，但是我们在循环中满足某种条件之后需要执行一些特定操作，因此才出现了上面的几个关键字：

continue：跳过当前循环，直接进入下一次循环

break：跳出整个循环体，继续执行循环下面的语句

return：通常用于跳出方法，也就意味着方法的结束了，如果返回值为void则使用`return;`的语法



Java泛型，什么是类型擦除，介绍常用的通配符

泛型是在JDK5中引入的一个新特性，参与到编译时候的安全检查，泛型的本质是参数化类型，也就是将操作的数据类型指定为一个参数

Java的泛型是伪泛型，在编译期间会被擦除，也就是我们说的类型擦除，都是以Object类型存储的，然后在我们需要的时候JVM会自动将其转换为我们指定的数据类型。

验证方法：

```java
/**
 * @author LuckyCurve
 * @date 2021/1/20 11:36
 * 验证Java的伪泛型
 */
public class Test {
    public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        List<Integer> list = new ArrayList<>();

        Class<? extends List> listClass = list.getClass();
        Method add = listClass.getMethod("add", Object.class);
        add.invoke(list,"sss");

        System.out.println(list);
    }
}
```

突破了编译期的检查，使用反射，在运行期间才能确定下来，上面代码不会报错，说明类型确实被擦除了。

泛型的三种使用方式：泛型类、泛型接口、泛型方法

```java
public class Generic<T> {
    private T t;
}

// 使用
Generic<Integer> g = new Generic<>();
```

这里采用了Java7提供的菱形语法，简便一点。

```java
public interface Generator<T> {
    T method();
}

class GeneratorImpl<T> implements Generator<T> {
    @Override
    T method() {
        return null;
    }
}

class GeneratorImpl<T> implements Generator<String> {
    @Override
    String method() {
        return null;
    }
}
```

其实这里类指不指定泛型与接口无关了，只需要提供接口的泛型即可使用。

```java
public static <E> void printArray(E[] array) {
    for (E e:array) {
        System.out.println(e);
    }
}
```

在返回值前面指定就可以了，使用时候不用指定，因为传参就相当于带上了类型了。



==和equals的区别

==判断两个对象地址是否相等，实际上也就相当于判断是否是同一个对象

基本数据类型==比较的是值，而对象则比较的是地址

> 实质上都是比较变量的值，只是基本数据类型存放的是值，而应用类型存放的是地址

equals只能比较对象是否相等，是Object中的一个方法，默认是调用==的，可以进行重载



hashcode方法和equals方法

重写equals方法时必须重写hashcode

hashcode返回一个int整数，实际上就是该对象在哈希表中的索引位置，Object中的hashcode是native的，使用C/C++实现，通常只是将对象的内存地址转换为整数进行输出

为什么会有hashcode呢？集合框架可以将哈希值作为对象的唯一标识从而判断是否有重复对象，如HashSet，如果两者的hashcode相等，在调用equals方法判断是否是真的相同，如果相同了hashSet不会让对象成功插入。**大大减少了equals的使用次数，相应的就大大的提升了效率**

如果两个对象相等，那么hashcode一定是相同的，但是两个对象的hashcode相等不意味着它们两相等，还是要调用equals函数，为了能够顺利的调用equals函数判断是否相等，hashcode必须重写。

为什么不能通过hashcode相等判断两个对象相等？

hash值是有上限的，可能出现哈希碰撞，且算法越差，碰撞几率越大



Java的基本数据类型及占用字节数

| 基本类型 | 位数 | 字节 | 默认值  |
| -------- | ---- | ---- | ------- |
| int      | 32   | 4    | 0       |
| short    | 16   | 2    | 0       |
| long     | 64   | 8    | 0L      |
| byte     | 8    | 1    | 0       |
| char     | 16   | 2    | 'u0000' |
| float    | 32   | 4    | 0f      |
| double   | 64   | 8    | 0d      |
| boolean  | 1    |      | false   |

官方文档没有明确boolean占几个字节，就JVM不同可能有不同的处理方式。

在符合JVM规范的虚拟机中，如果boolean单独使用就占4位，如果以boolean数组使用就占用1位，但JVM规范也只是建议，实现仍然是看具体虚拟机。



自动拆箱装箱的实现

主要是通过封装类的valueOf方法和XXXValue方法来实现的。

具体可能会问

```java
Integer i1 = 100;
Integer i2 = 100;
Integer i3 = 200;
Integer i4 = 200;
```

i1== i2  i3 == i4。具体可以看valueOf的源码，因为构件Integer的时候调用了valueOf

如果其中又一边是基本数据类型，那么输出一定就是true了，因为会发生自动拆箱，全部变成了基本数据类型的比较，进行运算也会拆箱成基本数据类型。

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

IntegerCache.low写死了是-128，然而IntegerCache.high是经过了延迟初始化的，即是在static代码块中初始化的，注释给出了原因`high value may be configured by property`

 

| 数据类型 | 是否有缓冲池 | 缓存最小值 |  缓存最大值   |     占用字节数     |
| :------: | :----------: | :--------: | :-----------: | :----------------: |
|   Byte   |      有      |    -128    |      127      |         1          |
|  Short   |      有      |    -128    |      127      |         2          |
| Integer  |      有      |    -128    | 127【可指定】 |         4          |
| Boolean  |      有      |   false    |     true      | 1/4【符合JVM规范】 |
|  Float   |      无      |            |               |         4          |
|  Double  |      无      |            |               |         8          |
|   Char   |      有      |     0      |      127      |         2          |
|   Long   |      有      |    -128    |      127      |         64         |



什么是方法的返回值，返回值的作用是什么？

方法的返回值是指我们指定运行的方法代码所产生的结果，作用是接收方法的返回值，我们可以使用返回值进行一些其他的操作。



为什么Java只有值传递

按值传递表示方法接收的是调用者提供的值

按引用传递表示方法接收的是调用者提供的变量地址

方法可以修改传递引用所对应的变量值，但是不能修改传递值所对应的变量值。

Java采用值传递是为了让方法不能修改传递给他的任何参数变量的内容。

都是传递的值，无论是对基本数据类型和对象引用来说，对基本数据类型来说，传递的值就是所表示的数值，对对象引用来说值就是对象地址。



重载和重写的区别

重载是同样一个方法能够根据输入数据的不同，做出不同的处理

重写是子类继承父类的相同方法，要求方法名与入参一致

重载发生在同一个类中，方法名必须相同，参数类型不同，个数不同，顺序不同，这些是一个方法的标识符，只有这些才能区分调用哪个方法，方法返回值和访问修饰符可以不同

简而言之，重载就是同一个类中多个同名方法根据不同的传参来执行不同的逻辑处理

重写执行在运行期，子类对父类允许访问的方法的实现过程进行重新编写

1. 返回值类型，方法名，参数列表必须相同，抛出的异常范围小于等于父类，访问修饰符范围大于等于父类
2. 如果父类方法被修饰符`private/static/final`修饰则子类不能重写该方法，但是被`static`修饰的方法能够被再次声明，子类相当于对外部隐藏了父类的该static方法
3. 构造方法无法被重写

static、final（private属于final）是前期绑定，其他普通方法是后期绑定，因此前面涉及到的方法覆盖问题都不会具有多态性，只是单纯的覆盖掉了。

|   区别点   | 重写方法 |                 重载方法                 |
| :--------: | :------: | :--------------------------------------: |
|  发生范围  |   子类   |                 同一个类                 |
|  参数列表  | 必须修改 |               一定不能修改               |
|  返回类型  |  可修改  | 可以返回派生类，void和基本数据类型不能改 |
|    异常    |  可修改  |            可以抛出异常派生类            |
| 访问修饰符 |  可修改  | 可以放宽限定，即默认——protected——public  |
|  发生阶段  |  编译期  |                  运行期                  |



浅拷贝和深拷贝

浅拷贝：基本数据类型值传递，对引用数据类型进行引用传递

深拷贝：基本数据类型值传递，对引用数据类型，创建一个新的对象，并复制其内容

> 感觉不是非常准确
>
> 使用Object方法的clone方法拷贝的对象，尽管是两个相互独立的，但是内部的对象引用都相同，应该也不算真正的深拷贝了。

![68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d372f6a6176612d646565702d616e642d7368616c6c6f772d636f70792e6a7067 (400×195)](https://camo.githubusercontent.com/2ba88f5f00d8f37b58c303929a0cb97bc9925189287ad741ec74338c867ee14e/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d372f6a6176612d646565702d616e642d7368616c6c6f772d636f70792e6a7067)





## 二、Java面向对象



面向对象和面向过程的比较

面向过程语言性能高，因为不需要实例化类，在单片机、嵌入式开发这类注重性能的场合使用较多

面向对象具有易维护，易复用、易扩展的特性，因为有封装继承多态三种特性，可以轻松设计出低耦合的系统

> 当然这并不是Java性能瓶颈的原因，主要是因为Java是半编译语言，编译出来的文件无法被CPU直接执行，需要JVM解释执行



在Java中之所以往往要加上一个不做事儿且无参的构造方法的作用

因为在执行子类的构造方法之前，如果其中没有使用super调用父类的构造方法，会默认调用父类的无参构造方法，这里声明主要是为了子类的实例化考虑。



成员变量和局部变量的区别

|                    成员变量                     |    局部变量     |
| :---------------------------------------------: | :-------------: |
|               从属于类或者是对象                |   从属于方法    |
|   可以被public，protected、private、final修饰   | 只能被final修饰 |
| 存储于堆、元空间（HotSpot是直接内存来实现的）中 |   存储于栈中    |
|              非final类型存在默认值              |    需要赋值     |



对象实体和对象引用的关系

对象实体存在于堆中，对象引用存储于栈中

对象实体可以有n个对象引用，而对象引用只能有0或1个对象实体



面向对象的三种特性

封装：把一个类的状态信息（属性）隐藏在状态内部，不允许外部对象直接访问对象的内部信息，只能通过该对象预设的方法来完成对类的状态信息的访问。

继承：继承是使用已存在的类的定义作为基础新建新的类的技术，新增的类可以增加数据或者是功能，也可以直接沿用父类的功能，但不能选择性的继承（这也是为什么重写只能扩大父类方法的访问权限的一个原因），提升开发效率。**子类拥有父类一切的属性和方法，只不过父类中的私有属性和方法子类无法访问，仍然持有。**

多态：表示一个对象具有多种的状态，具体表现为父类的引用指向子类的实例。

具有的特点：

- 对象类型和引用类型之间具有继承/实现的关系
- 引用类型的方法调用只有在运行期间才能确定到底是调用哪个类的方法
- 多态无法调用子类存在但是父类不存在的方法



String、StringBuilder和StringBuffer各自定义和他们之间的关系

可变性：

在Java9之前，内部实现都是`char[] value;`，Java9之后使用的是`byte[] value;`实现，String直接在类里面了，后两者在父抽象类AbstractStringBuilder中，只是String的value被`private final`修饰，且String还被final修饰（基本数据类型和String都被final修饰），体现了String的不可变性。

线程安全性：

在《Java并发编程实战中》提到一种线程安全策略：只读共享，由于这里的String是不可变的，因此是满足只读共享特性，是线程安全的。

在共同父类AbstractStringBuilder中就已经提供了append方法，StringBuilder和StringBuffer都是直接对其中方法的调用，只是StringBuffer使用synchronized关键字修饰了整个方法，而StringBuilder没有。

性能：

StringBuilder>StringBuffer>String

使用终结：

少量数据：String

单线程操作大量数据：StringBuilder

多线程操作大量数据：StringBuffer



Object的方法：

```java
public class Object {

    // 本地函数的注册
    private static native void registerNatives();
    static {
        registerNatives();
    }

    // 构造方法
    @HotSpotIntrinsicCandidate
    public Object() {}

    // native方法，获取当前对象的Class对象，且该方法被final修饰，无法被子类重写
    @HotSpotIntrinsicCandidate
    public final native Class<?> getClass();

    // 返回对象的哈希值，默认是堆内地址，主要使用在Map中
    @HotSpotIntrinsicCandidate
    public native int hashCode();

    // 比较两个对象是否相等，默认是比较对象的内存地址
    public boolean equals(Object obj) {
        return (this == obj);
    }

    // 创建当前对象的一份拷贝，即x.clone() != x，且该类必须实现Cloneable否则报错CloneNotSupportedException
    @HotSpotIntrinsicCandidate
    protected native Object clone() throws CloneNotSupportedException;

    // 转字符串的方法，默认是类名@实例哈希吗的十六进制，建议子类重写
    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }

    // final native方法，随机唤醒一个在此对象上等待的线程
    @HotSpotIntrinsicCandidate
    public final native void notify();

    // 唤醒所有在此对象上等待的线程
    @HotSpotIntrinsicCandidate
    public final native void notifyAll();

    // final方法。调用内部
    public final void wait() throws InterruptedException {
        wait(0L);
    }

    // native final方法，加入等待时间，暂停线程的执行，并释放持有的锁
    public final native void wait(long timeoutMillis) throws InterruptedException;

    // 多了第二个参数单位为纳秒，相加共同表示超出时间
    public final void wait(long timeoutMillis, int nanos) throws InterruptedException {
        if (timeoutMillis < 0) {
            throw new IllegalArgumentException("timeoutMillis value is negative");
        }

        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }

        if (nanos > 0) {
            timeoutMillis++;
        }

        wait(timeoutMillis);
    }

    // 实例被垃圾收集器回收的时候触发的操作
    @Deprecated(since="9")
    protected void finalize() throws Throwable { }
}
```



==和equals

==对基本数据类型来说比的是值，引用数据类型比较的是所指向的内存地址

equals是判断两个对象是否相等，一般存在两种情况：

- 没有覆盖equals方法，此时使用Object的equals，默认还是==来比较
- 覆盖了equals，一般都会覆盖，通过内部逻辑来比较两个对象的内容是否相等



hashcode和equals

为什么重写equals的时候必须重写hashcode

hashcode 在Object中就已经存在了，意味着任何对象都有hashcode，通过hashcode可以快速的在哈希表上查找到对象的位置。

主要是在使用集合框架的时候，在比较是否有重复对象的时候，会先通过hashcode来看有没有重复，如果有重复的话再对这两个对象使用equals方法来检查到底是否相等。没有直接使用hashcode是为了减少equals直接调用的次数，提升性能。因此只有在散列表的集合框架中才有用。



有些字段不想序列化

可以使用transient关键字修饰不想被序列化的变量

transient只能修饰变量，不能修饰类和方法



Java键盘输入，使用Java7提供的Try With Resources语法

```java
try (Scanner scanner = new Scanner(System.in)) {
    System.out.println(scanner.nextLine());
}
```

```java
try (BufferedReader reader = new BufferedReader(new InputStreamReader(System.in))) {
    System.out.println(reader.readLine());
} catch (IOException e) {
    e.printStackTrace();
}
```





## 三、Java核心技术



反射机制

指在运行过程中，对任意一个类，都可以知道这个类的所有属性和方法，对任意一个对象，都可以调用他的任意一个方法和获取任意一个属性，这种动态获取的信息以及动态调用对象的方法的功能被称为Java语言的反射机制



静态编译和动态编译

静态编译：在编译期间确定类型，绑定对象

动态编译：在运行期间才能确认类型，绑定对象



反射机制的优缺点

优点：将对象类型的推断和类的加载（JDBC）推迟到了运行期间，具有更高的灵活性

缺点：1、性能瓶颈，反射机制相当于一系列解释操作，即通知JVM去做什么，远不如运行class文件来的效率高。2、安全问题：可以实现private属性的访问，突破了安全限制。



反射的应用场景

反射是框架设计的灵魂

在使用JDBC连接数据库的时候，就使用到了`Class.forName()`方法来动态加载数据库驱动

Spring的IOC和AOP都和反射有关



反射动态获取信息需要通过Class对象，获取Class对象的四种方式

```java
// 1 直接通过类名来获取class对象
Class c = TargetObject.class;
// 2 通过Class.forName来获取类信息，默认是需要初始化的，可以通过传参控制，默认true初始化，否则不初始化
Class c = Class.forName("...");
// 3 通过Object#getClass方法获取
Class c = targetObject.getClass();
// 4 通过类加载器去获取，不会就行初始化
Class c = ClassLoader.loadClass("...");
```

使用Demo：

```java
public class TargetObject {

    private String value;

    public TargetObject() {
        value = "LuckyCurve";
    }

    public void flag(String flag) {
        System.out.println(flag + "\t In " + new Date());
    }

    private void getValue() {
        System.out.println(value);
    }
}

public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException, NoSuchFieldException {
    Class<?> classInfo = Class.forName("cn.luckycurve.reflection.TargetObject");
    // 创建实例
    TargetObject instance = (TargetObject) classInfo.getDeclaredConstructor().newInstance();

    // 输出所有方法名
    Arrays.stream(classInfo.getDeclaredMethods()).forEach(System.out::println);

    // 获取方法名并调用
    Method method = classInfo.getDeclaredMethod("flag", String.class);
    method.invoke(instance, "实习上岸");

    // 修改private的值
    Field field = classInfo.getDeclaredField("value");
    // 取消安全检查，访问private
    field.setAccessible(true);
    field.set(instance, "LuckyCurve2");

    // 调用private方法
    Method method1 = classInfo.getDeclaredMethod("getValue");
    method1.setAccessible(true);
    method1.invoke(instance);
}
```



异常

Java异常类的层次结构

![68747470733a2f2f67756964652d626c6f672d696d616765732e6f73732d636e2d7368656e7a68656e2e616c6979756e63732e636f6d2f323032302d31322f4a617661254535254243253832254535254238254238254537254231254242254535254231253832254536254143254131254537254242253933254536253945253834254535253942254245322e706e67 (1946×952)](https://camo.githubusercontent.com/17d2b543fcd5c80208deea27d941b41d7cb868da7172d6cf1e91722b34a86f0b/68747470733a2f2f67756964652d626c6f672d696d616765732e6f73732d636e2d7368656e7a68656e2e616c6979756e63732e636f6d2f323032302d31322f4a617661254535254243253832254535254238254238254537254231254242254535254231253832254536254143254131254537254242253933254536253945253834254535253942254245322e706e67)

Throwable是直接继承Object的，实现了序列化接口，有两个子类：Error和Exception

只有Exception可以被try-catch，Error错误往往是Java程序无法处理的，因此只能避免，常见的Error如：VirtualMachineError、OutOfMemoryError、StackOverflowError

在Exception中，除了RuntimeException及其子类是不受检查异常，其余的都是受检查异常，需要进行处理，否则无法通过编译。

Throwable类的常用方法如下：

```java
// 异常的简要描述
public String getMessage();
// 异常的详细信息
public String toString();
// 返回异常对象的本地化信息，通常子类会覆盖，如果不覆盖则返回与getMessage相同
public String getLocalizedMessage();
// 最常用的，打印到控制台
public void printStackTrace();
```



try-catch-finally语句块

try块：用于捕获异常，后面跟零个或者多个catch，如果没有catch则必须有finally

catch块：处理捕获到的异常

finally块：无论是否捕获到异常，finally中的语句都会执行，如果其中存在return语句，则在return语句执行前执行完finally块中的语句

存在几种情况finally语句不会运行：

1. 运行了`System.exit(int)`，如果前面有异常，那么该语句没有被执行到，依然会执行finally，且finally中的return语句会覆盖原来的return语句
2. 程序所在的线程死亡
3. CPU关闭



使用Java7提供的try-with-resources代替try-catch-finally

适用范围：实现了`java.io.Closeable`（父类是AutoCloseable）或者是`java.lang.AutoCloseable`的类

执行顺序：任何catch/finally在资源关闭之后运行



线程、程序、进程的基本概念和他们之间的基本关系

进程是程序的一次执行过程，是系统运行程序的基本单位，因此进程是动态的。

程序是含有指令和数据的文件，被存储在磁盘或者其他的数据存储设备中，程序是静态的

线程与进程类似，被称为轻量级进程，一个进程可以包含多个线程，并且线程之间切换负担比进程间切换小的多



线程的基本状态

NEW：初始状态，被构建但还没有调用start方法R

RUNNING：运行状态，Java将操作系统中的就绪和运行统称为运行状态

BLOCKING：阻塞状态，表示线程阻塞于锁

WAITING：等待状态，进入该状态后等待其他线程做出一些特定动作（如通知或者是中断）

TIME_WAITING：超时等待状态，与上面的WAITING非常类似，只是超时会自动返回

TERMINATED：终止状态，表示当前线程执行完毕

![img](https://camo.githubusercontent.com/e08f7aa21580f92df00142e9b271e49597311df799e8aef6465955faacd85e41/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f31392d312d32392f4a6176612532302545372542412542462545372541382538422545372538412542362545362538302538312545352538462539382545382542462538312e706e67)	



Java流的分类：

按照流向分：分为输入流和输出流

按照操作单元划分：分为字符流和字节流

Java IO中的类都从以下四个抽象类中派生出来的：

InputStream/Reader

OutputStream/Writer



既然信息的最小存储单元是字节，为什么要有字符流？

Java的字符流是JVM将字节流转换得到的，如果留给开发者自己实现非常容易出现性能低下，乱码等错误，因此Java直接提供了这个接口作为一个可选项，只有在传输的文件涉及到字符的时候才会使用到字符流



BIO、NIO、AIO的区别

Blocking IO：同步阻塞IO，数据的读取写入必须阻塞在一个线程内部完成，当并发量不是很大的时候这种模式还是非常好的，可以让每一个线程都只专注于获取自己需要的资源，但当并发量上来之后BIO往往无能为力

Non-blocking IO/New IO：同步非阻塞IO，Java1.4时候引入，核心抽象为Channel、Selector、Buffer。他是面向缓冲的，基于通道的I9O操作方法，在文件传输过程中不会涉及到操作系统状态转换（管态——用户态），而是实现文件的直接传输，在高负载、高并发的环境下能起到很好的作用

Asynchronous IO：也被称为NIO2，在Java7中引入，是对NIO的改良版，AIO是基于事件和回调机制来实现的，不会出现线程等待的情况，当后台处理完成时，会通知线程（BIO则是需要线程自己去获取IO操作是否执行完成了）。AIO目前应用不是很广泛，Netty曾经尝试使用过，后来放弃了

这里更加注重的是网络编程中的IO操作



可以理解为Java语言对操作系统的各种IO模型的封装，我们直接调用即可。



同步、异步、阻塞、非阻塞比较

当你同步执行某项任务时，你需要等待其完成才能继续执行其他任务，当你异步执行某些操作时，你可以在完成另一任务之间继续进行。

阻塞：调用者会一直等待请求返回，只有当结果返回再能继续执行

非阻塞：调用者发起一个请求后可以去干别的事情



同步/异步是从行为角度描述的，而阻塞和非阻塞描述的是调用者线程的状态

BIO：同步阻塞IO，使用BIO，服务端通讯模型如下

![img](https://camo.githubusercontent.com/954def15d50a7807d3e43cd4d7b8a451bf88ac2e9fdd665ea4e10d728f0c8020/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f322e706e67)

Acceptor负责分配Client线程与Server线程对应，面对百万级别的请求，可能需要百万个线程同时处理，但是线程是重量级资源，是不现实的。

后来使用了线程池进行了优化，形成了伪异步IO

![伪异步IO模型图](https://camo.githubusercontent.com/707e56443bcc2e10f05f3af5e9cf75a9a5bc10b7d7b67f1c0637c52657853296/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f332e706e67)

尽管这样，由于socket的read和write方法仍然是阻塞的，无法从根本上解决问题，只会到头来任务量越来越多。

当服务量小于1000的时候（单机），使用BIO还是非常好的，每个线程专注于自己所做的事情，并且应该尽量使用线程池，可以起到异步削峰的作用，毕竟有一条工作对列作为缓冲



Java1.4引入的NIO，提供了Channel、Selector、Buffer等抽象

NIO是面向缓存的，基于通道的IO操作方法，提供了SocketChannel，ServerSocketChannel代替传统的BIO操作，都支持阻塞和非阻塞方式，因此在低负载、低并发场景下使用同步阻塞IO方法，在高负载，高并发的应用使用非阻塞方式

NIO与IO的区别：

1、名字，NIO 非阻塞IO模型，BIO 阻塞IO模型

2、面向对象：NIO是面向缓冲区的，BIO是面向流的

NIO是将数据直接读取到Buffer抽象中进行操作，BIO也有Buffered开头的流，却要将数据从流中读入缓冲区在进行操作，比较慢。NIO中的所有操作都是面向缓冲区的，读的时候直接从缓冲区里面读，写的时候直接写入缓冲区，最常见的缓冲区是ByteBuffer，实际上每一种数据类型除了Boolean都有缓冲区

3、读写方式不同：NIO使用管道进行读写，IO使用流进行读写

NIO使用Channel对Buffer进行读写操作，是双向的，而流是单向的，只能读或者写

4、NIO具有选择器，IO没有

选择器用于单个线程处理多个通道（这也正是能客户端到服务端为1：n的关系了），避免为每个Channel都创建一个线程，减小了创建成本和切换成本

NIO使用起来非常复杂，不吐槽了，Netty很大程度上简化了NIO操作，因此才被推崇

![一个单线程中Selector维护3个Channel的示意图](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-2/Slector.png)

AIO主要就是异步非阻塞，调用方完成调用指令执行后可以干其他事儿，等待被调用方执行完成后自动执行回调函数等操作来通知调用方







## 四、Java易错点



1、正确使用equals方法

如果进行a和b的比较，不要直接使用`a.equals(b)`，如果a为null则会报空指针异常

> 通过null来调用非静态方法会抛出异常，换言之调用静态方法不会抛出异常

推荐使用Java7提供的Objects#equals方法



2、关于BigDecimal的用法

《阿里巴巴Java开发手册》中强制规定：浮点数之间的基本判断，基本数据类型不能用==来比较，包装数据类型不能用equals来比较

```java
float a = 1.0f - 0.9f;
float b = 0.9f - 0.8f;
System.out.println(a == b);
```

因为浮点数采用位数+阶码的编码方式，会造成精度丢失

解决方法，通过将浮点数转换成为BigDecimal对象，然后调用方法substract进行减法运算等等，最后通过compareTo方法比较，绝对不是equals方法，因为只有compareTo 方法会忽略精度

BigDecimal内部使用到了BigInteger，推荐初始化的时候使用BigDecimal的static方法valueOf方法进行初始化，如果直接使用BigDecimal(double)可能会造成精度的丢失，使用valueOf方法会先将double转换成String，保留精度



Arrays#asList方法的一些坑

Arrays.asList入参必须是对象数组，因为asList是泛型方法，如果是基本数据类型数组则会直接得到数组本身。返回的数组是不可变的，增删会抛出UnsupportedOperationException异常

推荐使用的将数组转换成List的方法：

可以返回一个可变的List（推荐）：

```java
new ArrayList<>(Arrays.asList(nums));
```

使用Java8的方法（推荐）：可以处理原始数据类型，返回的List为ArrayList

```
Arrays.stream(nums).boxed().collect(Collectors.toList());
```

使用Guava（推荐），可以选择性的生成可变和不可变对象



集合转数组和集合元素的反转

```java
Integer[] ints = list.toArray(new Integer[0]);
```

这里的`new Integer[0]`是经过了JVM的优化的，只是起到了一个模板的作用，0是为了节省空间

数组的翻转得先将其转换为集合，调用Collections#reverse()方法



不要在foreach集合中使用增删元素操作

会抛出一个ConcurrentModificationException异常，这被称为fail-fast机制，java.util包下面的集合类都是fail-fast的，而java.util.concurrent包下面的类都是fail-safe的

> List移除倒数第二个元素不会报错

推荐使用Java8的list.removeIf(filter)方法

推荐使用Iterator的方式

之所以foreach会报错而Iterator不会报错，看报错的栈

```java
public E next() {
    // 报错
    checkForComodification();
    int i = cursor;
    if (i >= size)
        throw new NoSuchElementException();
    Object[] elementData = ArrayList.this.elementData;
    if (i >= elementData.length)
        throw new ConcurrentModificationException();
    cursor = i + 1;
    return (E) elementData[lastRet = i];
}
```

```java
final void checkForComodification() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
}
```

主要是由于修改次数和期望修改次数两个参数不等导致的

我们在foreach中直接add和remove只是会改变modCount的值，不会更改exceptedModCount的值

如果是Iterator（ArrayList中的内部类Itr），源码如下：

```java
public void remove() {
    if (lastRet < 0)
        throw new IllegalStateException();
    checkForComodification();

    try {
        ArrayList.this.remove(lastRet);
        cursor = lastRet;
        lastRet = -1;
        expectedModCount = modCount;
    } catch (IndexOutOfBoundsException ex) {
        throw new ConcurrentModificationException();
    }
}
```

最主要的就是第十行的信息，会修改expectedModCount的值，所以不会报错

之所以倒数第二个不会报错，是因为不会进入next方法，删除元素后当前hasNext（）方法返回了false

```java
public boolean hasNext() {
    return cursor != size;
}
```

0 1 2 3的第三个元素，本来size=4，结果remove后size=3，相等就返回false，但是如果删除到最后一个仍然是不相等，所以true。





## 五、Java枚举



可以使用在非常多的场景，比如SpringMVC的全局异常处理

enum在Java5中引入，使用enum标注的类默认是继承java.lang.Enum的

我们在使用枚举的时候大部分都是代替常量的，目的是增加代码的可读性，

Demo：

```java
public enum PizzaEnum {
    ORDERED,
    READY,
    DELIVERED;

    public static void main(String[] args) {
        System.out.println(PizzaEnum.ORDERED);
        System.out.println(PizzaEnum.ORDERED.getClass());
        System.out.println(PizzaEnum.ORDERED.name());
    }
}
```

因为enum标注的终究只是一个特殊的类（继承了Enum抽象类），可以包含一些状态（类型为PizzaStatus本身），也可以包含一些函数如主函数来进行测试。

因为枚举类型在JVM中能确保是一个常量实例，因此可以直接使用`==`进行运算，且`==`运算不会像equals可能抛出NullPointExceptioin异常。因此可以直接在switch语句中使用，本质上就是一些if == 的集合

重点：可以在枚举中定义方法和构造函数来让枚举更加强大

例如我们需要返回每个状态大概订单的等待时间，原来是需要switch语句然后返回固定时间，现在通过给枚举中添加方法的方式来返回，并且返回当前Pizza在该状态是否完成了。

```java
public enum PizzaEnum {
    /**
     * 常量
     */
    ORDERED(5) {
        @Override
        public boolean isOrdered() {
            return true;
        }
    },
    READY(2) {
        @Override
        public boolean isReady() {
            return true;
        }
    },
    DELIVERED(1) {
        @Override
        public boolean isDelivery() {
            return true;
        }
    };

    /**
     * 属性
     */
    private int timeToDelivery;

    /**
     * 方法
     */
    PizzaEnum(int timeToDelivery) {
        this.timeToDelivery = timeToDelivery;
    }

    public int getTimeToDelivery() {
        return timeToDelivery;
    }

    public boolean isOrdered() {
        return false;
    }

    public boolean isReady() {
        return false;
    }

    public boolean isDelivery() {
        return false;
    }

    /**
     * 测试方法
     */
    public static void main(String[] args) {
        PizzaEnum pizzaEnum = PizzaEnum.ORDERED;
        System.out.println(pizzaEnum.isOrdered());
        System.out.println(pizzaEnum.getTimeToDelivery());
    }
}
```

感觉是非常方便的。



跟随Java5一起引进的还有EnumSet和EnumMap两个抽象类型

初始化时候可以直接使用EnumSet.of方法进行对象的创建，至于返回哪一个子类则是看实例化时候的枚举常量的数量，如果需要获取当前Enum的所有可能状态可以使用`Enum.values();`方法，这个方法应该是动态字节码生成的，找不到values方法。

EnumSet可以返回如取子集、增删等操作，基本和HashSet差不多，都继承了AbstractSet接口

就相当于是对元素为enum的单独操作，HashSet会报错。

```java
private final EnumSet<PizzaEnum> unDeliveredPizzaStatus = EnumSet.of(ORDERED, READY);

public List<pizza> getUnDeliveredPizza(List<pizza> list) {
    return list.stream().filter(pizza -> unDeliveredPizzaStatus.contains(pizza.getPizzaEnum())).collect(Collectors.toList());
}
```



EnumMap则是HashMap对enum操作的实现。

之所以对enum进行了单独的实现，是希望更加高效的处理enum，而不是像对普通的Object。



通过枚举类型实现设计模式

1、 单例模式

使用枚举来实现会非常简洁，并且无偿提供序列化机制，安全性和稳定性由JVM直接保障。

```java
public enmu SystemConfiguration {
    INSTANCE;
    SystemConfiguration() {
        // 一些初始化操作
    }
    
    public static SystemConfiguration getInstance() {
        return INSTANCE;
    }
}
```

这里的INSTANCE就是绝对的单例模式了。

2、策略模式，策略模式意味着添加新的实现类

而枚举中的所有常量都可以对原来的方法进行覆盖，非常轻松的就可以完成任务，但是只能有一个对象，即枚举属性本身

Jackson序列化，和普通类达到的效果差不多，实例化所有的常量





## 六、关键字总结



final关键字：意思是最终的，不可变的，用来修饰类、方法和变量

private方法都隐式的指定为final。

使用final方法的原因有两个：1、锁定方法，避免被子类重写。2、早期的Java可以通过声明final使得方法转为内嵌调用，增加性能，现在没必要了。



static关键字，常见用法：

1、修饰成员变量和成员方法，使得变量和方法从属于类，静态成员变量存放于方法区中

2、静态代码块：初始化类信息，至于对象的初始化顺序，则是非静态代码块——构造方法，对于定义在静态代码块之后的静态变量，静态代码块可以赋值不能访问

3、静态内部类：static修饰类只能修饰内部类，非静态内部类创建时候在对象中会存放一个引用指向外围对象，而静态内部类没有，意味着静态内部类：1、不需要依赖外围类的 2、无法获取外围类的非static成员变量和方法，静态内部类只有被访问到时候才会进行初始化。运用于单例模式，主要是当静态内部类没有被访问的时候不会初始化，被访问之后才进行了初始化，延迟初始化特性，单例是通过JVM初始化锁来保证的

4、静态导包：5提供的新特性。`import static`导入某个类中的指定静态资源，省去了`类名.方法名/资源名`的类名。



需要注意的一个问题：

一旦在构造函数中显式的调用了super的构造函数或者是this的构造函数，必须在首行，否则会报错（默认是在构造函数执行之前调用`super()`）





## 七、Java的代理模式



代理模式非常好理解，可以通过使用代理对象来代替真实对象的访问，这样我们可以在不修改真实对象的前提下完成对功能的扩展。

代理模式有动态代理和静态代理两种实现：

- 静态代理

在静态代理中，对目标方法的增强都是手动完成的，非常不灵活（通常代理类和目标类都实现了同一份接口，如果接口一旦发生改变就都需要更改了），并且非常麻烦（每进行一次代理都要创建一个代理类），通常不是很实用，示例如下

```java
public interface Message {
    void sendMessage(String info);
}

// 原有的实现类
public class TargetMessage implements Message {
    @Override
    public void sendMessage(Stirng info) {
        // do something
    }
}

// 代理类
public class ProxyMessage implements Message {
    private final Message message;
    
    public ProxyMessage(Message message) {
        this.message = message;
    }
    
    @Override
    public void sendMessage(String info) {
        // do something before
        message.sendMessage(info);
        // do something after
    }
}
```



- 动态代理【依靠反射来实现的】

不需要为每次代理都创建一个代理类，从JVM角度看，动态代理是在运行时动态生成类字节码，并加载到JVM中的（Spring AOP和RPC框架【方法的远程调用，可以向调用本地方法一样直接调用远程的方法，不涉及到HTTP数据包的封装与拆解，效率高，较知名的RPC框架：Dubbo】使用到了），虽然在日常开发中使用较少，但是在框架中基本是必用的。

两种实现方法：JDK动态代理和CGLIB动态代理

JDK动态代理的两个核心类：` java.lang.reflect.InvocationHandler`和`java.lang.reflect.Proxy`，可以看到都是reflect包下面的类，是基于反射来实现的

Proxy的关键方法为：

public **static** Object newProxyInstance(ClassLoader loader,
                                      Class<?>[] interfaces,
                                      InvocationHandler h);

该方法生成一个代理对象，可以看到方法入参有：类加载器、被代理类实现的一些接口，InvocationHandler的实例化子类，不难猜测出内部方法的调用逻辑都在InvocationHandler对象当中，InvocationHandler接口里面只存在一个方法：public Object invoke(Object proxy, Method method, Object[] args);也正是这个方法来处理实的际调

使用实例：

```java
public interface Message {
    void send(String info);
}

public class SendMessage implements Message {
    @Override
    public void send(String info) {
        System.out.println(getClass().getName() + "\t" + info);
    }
}

public class DebugInvocationHandler implements InvocationHandler {

    /**
     * 被代理对象
     */
    private final Object target;

    public DebugInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // before
        System.out.println("before Method:" + method.getName());
        // 调用方法
        Object o = method.invoke(target, args);
        System.out.println("After Method:" + method.getName());
        return o;
    }
}

public class JdkProxyFactory {

    // 返回一个代理后的对象
    public static Object getProxy(Object target) {
        return Proxy.newProxyInstance(target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                new DebugInvocationHandler(target));
    }

    // 实际使用
    public static void main(String[] args) {
        Message message = (Message) getProxy(new SendMessage());
        message.send("LuckyCurve");
    }

}
```

可悲的是，在最后测试时候需要将其转换为接口，使用Java的动态绑定机制来完成对象的指定，因此，JDK动态代理只能代理实现了接口的类，并且只能代理哪些

因此CGLIB动态代理产生了

- CGLIB动态代理

CGLIB是基于ASM的字节码生成库，允许运行时候对字节码进行修改和动态生成

JDK动态代理是向上，那么CGLIB更像是向下，对这个类进行了扩展和继承

核心类为：MethodInterceptor接口和Enhancer类

MethodInterceptor唯一方法：

public Object intercept(Object obj, java.lang.reflect.Method method, Object[] args,
                           MethodProxy proxy);

传入被代理对象，被拦截的方法，方法入参，最后一个参数用于调用原始方法

可以Enhancer的create方法来获取一个被代理后的对象，当然提前要进行一些参数的设定，和Proxy类的定义是一样的。

```java
public interface Message {
    void send(String info);
}

public class SendMessage implements Message {
    @Override
    public void send(String info) {
        System.out.println(getClass().getName() + "\t" + info);
    }
}

public class MessageMethodInterceptor implements MethodInterceptor {
    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        // before
        System.out.println("before method:" + method.getName());

        // 调用方法
        Object o = proxy.invokeSuper(obj, args);

        // after
        System.out.println("after method:" + method.getName());

        return o;
    }
}

public class CglibProxyFactory {

    public static Object getProxy(Class<?> classInfo) {
        // 用于创建动态代理增强类
        Enhancer enhancer = new Enhancer();
        // 设置类加载器
        enhancer.setClassLoader(classInfo.getClassLoader());
        // 设置被代理对象
        enhancer.setSuperclass(classInfo);
        // 设置MethodInterceptor
        enhancer.setCallback(new MessageMethodInterceptor());
        return enhancer.create();
    }

    public static void main(String[] args) {
        SendMessage proxy = (SendMessage) getProxy(SendMessage.class);
        proxy.send("LuckyCurve");
    }
}
```

通过输出可以看到，第一个输出为

```
before Method:send
cn.luckycurve.proxy.SendMessage	LuckyCurve
After Method:send
```

CGLIB的输出为：

```
before method:send
cn.luckycurve.proxy.SendMessage$$EnhancerByCGLIB$$b65d50cd	LuckyCurve
after method:send
```

可以这样理解，JDK动态代理仍然是直接让被代理对象执行方法的，和代码逻辑吻合，而CGLIB是直接生成一个子类，并让方法在子类中执行。

JDK动态代理始终只能得到一个接口，CGLIB则可以得到一个被代理后的子类



JDK动态代理和CGLIB动态代理对比：

1、JDK动态代理只能代理接口中的方法，无法代理类中的方法，CGLIB则可以直接代理类中的方法

2、JDK的效率是要高于CGLIB的，且随着JDK版本提升优势只会越来越大

静态代理和动态代理的区别：

1、灵活性，毋庸置疑动态代理灵活性更强，不用去创建代理类

2、JVM层面：静态代理实际上就是普通的方法调用，而动态代理则是在运行时动态生成字节码并加载到JVM中





## 八、Java集合框架



List、Set、Map区别

List：存储的元素是有序的、可重复的

Set：存储的元素是无序的、不可重复

Map：使用KV存储，KV都是无序的，K不能重复，V可以重复



集合框架中实现类的数据结构

ArrayList：Object[]

Vector：Object[]

LinkedList：双向链表（1.6之前为双向循环链表、1.7取消了循环，改成了双向链表）

CopyOnWriteArrayList：线程安全的ArrayList

ArrayList和Vector的区别

ArrayList是List的主要实现类，适合频繁的随机查找工作，线程不安全，Vector是List的古老实现类，线程安全，底层数据结构都是`Object[]`，还有初始化和扩容的区别，默认大小都是10，但是ArrayList是扩容成1.5倍，Vector是两倍，初始化大小都是10

ArrayList和LinkedList的区别

ArrayList采用Object[]数据结构，LinkedList采用双向链表结构（1.6之前为循环双向链表）

插入删除的时间复杂度比较：ArrayList的插入和删除都和位置i和数组长度n有关，为O（n-i）因为要移动后面的元素，LinkedList的插入时间复杂度近似为O（1），但如果是指定下标的插入则近似O（n）

ArrayList支持随机访问，LinkedList不支持随机访问，随机访问就是根据元素的序号快速获得元素，Java中提供RandomAccess接口用来标识是否支持随机访问，不仅仅是标注给我们看的，还有应用在如`Collections#binarySearch(List list,T key)`中，判断是否支持随机访问，如果支持就使用基于index的二分查找，否则就走基于迭代器的二分查找

元素利用效率：ArrayList的空间浪费主要是在尾部的数组空余，也是为了避免频繁的resize，而LinkedList的空间浪费主要是每个节点都维护了一个指向前一个和后一个节点的引用。

ArrayList的扩容机制（JDK11）：

```java
// 通过add方法可以进入到这里来，只有当已经存满的时候就进入这里来发生扩容，传入的mincapacity=当前size+1
private int newCapacity(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    // 默认是原来的1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // 防止oldcapacity=0的情况，进行初始化
    if (newCapacity - minCapacity <= 0) {
        // DEFAULTCAPACITY_EMPTY_ELEMENTDATA 默认持有的一个空数组，使用无参构造函数的时候就让elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            // 通常返回DEFAULT_CAPACITY=10
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return minCapacity;
    }
    // 判断是否过大MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8
    return (newCapacity - MAX_ARRAY_SIZE <= 0)
        ? newCapacity
        : hugeCapacity(minCapacity);
}

// 其实太大了也很好解决，剩余的留着也没用，如果大于了MAX_ARRAY_SIZE直接赋值Integer.MAX_VALUE
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE)
        ? Integer.MAX_VALUE
        : MAX_ARRAY_SIZE;
}
```

其中还有一个非常重要的方法：

```java
// 在ArrayList内部完全没有被调用过
public void ensureCapacity(int minCapacity) {
    if (minCapacity > elementData.length
        && !(elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
             && minCapacity <= DEFAULT_CAPACITY)) {
        modCount++;
        grow(minCapacity);
    }
}
```

当预存非常多的数据进入到ArrayList当中去的时候，可以先调用ensureCapacity函数来预先分配足够大的空间，避免频繁的发生扩容

LinkedList，实现了List和Deque，说明也具有双端队列的特性

内部结构：

![LinkedList内部结构](https://github.com/Snailclimb/JavaGuide/raw/master/docs/java/collection/images/linkedlist/LinkedList%E5%86%85%E9%83%A8%E7%BB%93%E6%9E%84.png)

内部是基于双向链表来实现的，且持有一个first和last节点，所有插入删除操作就直接映射到链表的操作上来了



CopyOnWriteArrayList

对比于Collections.synchronizedList，在读写时候都会进行锁定，然而一般都是读多写少，在读过程中需要获取锁，仍然是非常浪费资源

CopyOnWriteArrayList将读取的性能发挥到了极致，在读写锁的读写冲突上更进一步，只有写写操作之间需要同步等待，连读写都不会产生冲突

实现：在写入过程中对原数据进行拷贝，写入的时候写的是拷贝后的数据，读取的时候是读的拷贝前的数据，只有当写完之后，才会将内存指针由原来的内存指向这份拷贝内存，实现读写的完全分离。





<hr>

HashSet：（无序）：基于HashMap的Key实现的，采用HashMap保存元素

LinkedHashSet：（有序）：内部是通过LinkedHashMap来实现的

TreeSet：（有序）：红黑树（自平衡的排序二叉树）

Set接口基本都是借助Map接口的Key值来实现的，利用Map中的Key值唯一性从而达到Set接口的不可重复性

主要的只有一点，LinkedHashSet维护的是插入的秩序，使用forEach语法遍历得时候是根据插入先后顺序来的

TreeSet维护的是对象的排序，为了可以让对象比大小，强制要求加入的元素必须实现Comparable接口，实现其中的compareTo方法，否则直接报错，排序按照compareTo的返回值来

```java
public class Test {

    public static void main(String[] args) {
        Set<Person> set = new TreeSet<>();
        set.add(new Person("Lucky", 22));
        set.add(new Person("Lucky2", 21));
        set.add(new Person("Lucky3", 20));
        set.forEach(System.out::println);
    }

    public static class Person implements Comparable<Person> {
        private String name;
        private Integer age;

        public Person(String name, Integer age) {
            this.name = name;
            this.age = age;
        }

        @Override
        public String toString() {
            return "Person{" +
                    "name='" + name + '\'' +
                    ", age=" + age +
                    '}';
        }

        @Override
        public int compareTo(Person o) {
            if (this.age > o.age) {
                return 1;
            } else if (this.age < o.age) {
                return -1;
            }
            return 0;
        }
    }
}
```

不是必须实现的，只要给一个比较规则就行，通过Comparator来定义一个规则：

```java
public class Test {

    public static void main(String[] args) {
        Set<Person> set = new TreeSet<>(new Comparator<Person>() {
            @Override
            public int compare(Person o1, Person o2) {
                return o1.age.compareTo(o2.age);

                // 等价于
                //                 if (o1.age > o2.age) {
                //                    return 1;
                //                } else if (o1.age < o2.age) {
                //                    return -1;
                //                } else {
                //                    return 0;
                //                }
            }
        });
        set.add(new Person("Lucky", 22));
        set.add(new Person("Lucky2", 21));
        set.add(new Person("Lucky3", 20));
        set.forEach(System.out::println);
    }

    public static class Person {
        private String name;
        private Integer age;

        public Person(String name, Integer age) {
            this.name = name;
            this.age = age;
        }

        @Override
        public String toString() {
            return "Person{" +
                    "name='" + name + '\'' +
                    ", age=" + age +
                    '}';
        }
    }
}
```



Comparable是待比较元素本身需要去实现，本身有方法`int compareTo(T t);`

Comparator与待比较元素是外部关系，本身方法有`int compare(T t1, T t2);`

可以使用在Collections.sort方法入参当中，否则按照自然排序从小到大排

<hr>

HashMap：在JDK1.8之前是数组+链表的实现，链表主要是为了解决哈希冲突而存在的（拉链法），在1.8之后有了变化，当链表长度大于阈值8时（`static final int TREEIFY_THRESHOLD`）将链表转换为红黑树。（其实在转换过程前一步有判断，如果当前数组长度是小于64，则进行数组扩容，不会进行树的转换）

LinkedHashMap：LinkedHashMap继承自HashMap，拥有HashMap一切的数据结构，此外还增加了一条双向链表，使得上面的结构具有了保持键值对的插入顺序，实现了访问顺序相关逻辑

HashTable：实现是数组+链表，不会转换成树结构，所有方法均被synchronized修饰

TreeMap：红黑树（自平衡的排序二叉树）

ConcurrentHashMap：相当于一个线程安全的HashMap

ConcurrentSkipListMap：使用跳表实现的支持并发的Map

可能比较多了

KV能否存储null值总结：

|      集合类       |     Key      |    Value     |
| :---------------: | :----------: | :----------: |
|      HashMap      |  允许为null  |  允许为null  |
|     HashTable     | 不允许为null | 不允许为null |
|   LinkedHashMap   |  允许为null  |  允许为null  |
|      TreeMap      | 不允许为null |  允许为null  |
| ConcurrentHashMap | 不允许为null | 不允许为null |

可以发现，并发Map/同步Map都是不支持null值的，主要是因为Map接口调用get方法后如果没有找到会返回null值，那么并发Map就无法知道是put进入的null还是没有找到了，但是HashMap可以通过containsKey来判断，但是并发Map调用containsKey时候，该Map的数据可能已经发生变化了，与调用get时候的map可能完全不同了。

此外TreeMap也不支持null值，因为要根据Key进行排序，null无法进行排序，所以删除了。



HashMap和HashTable对比

1、线程是否安全 	2、效率	3、kv能否为null

4、初始化容量大小和扩容

HashMap默认进行延迟初始化，初始化大小为DEFAULT_INITIAL_CAPACITY = 1 << 4，此后每次扩容都会变为原来的两倍，如果给了initCapacity进行初始化，大小为initCapacity扩充到2的n次方为止。

HashTable默认进行就地初始化，初始化大小为11，如果给了initCapacity，大小为initCapacity

扩容：HashMap容量变成原来的两倍，HashTable的大小变为2n+1



TreeMap：

较之于HashMap还多实现了个NavigableMap接口，该接口表明可以对数据结构进行由小到大的排序和查找。相比之下多出了按照键来排序的能力和对集合元素搜索的能力



HashMap的底层实现

1.8之前采用数组+链表的方式进行处理，将K值经过hashcode和Map的扰动处理，为了尽可能避免哈希冲突，将计算出来的hash&（length - 1）计算该键值对存放的位置，如果当前位置存在元素，调用equals方法判断是否相等，如果相等则直接覆盖，如果不等则使用拉链表解决哈希冲突

> 位运算符：
>
> |  &   |  与  |
> | :--: | :--: |
> |  \|  |  或  |
> |  ^   | 异或 |
> |  ~   | 取反 |
>
> 

> 这里的扰动函数指的就是Map的hash方法，主要是为了避免一些糟糕的hashcode实现，尽可能减少哈希冲突
>
> ```java
> static final int hash(Object key) {
>     int h;
>     return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
> }
> ```
>
> 还是非常简单的



JDK1.8之后当链表长度大于阈值8的时候会将链表转换成红黑树减少搜索时间，在转换前会进行判断当前数组长度是否小于64，如果小于的话会进行数组扩容来直接减少哈希冲突而不是转换成树结构。

> 都使用红黑树而不使用二叉树的原因是因为二叉查找树在某些情况下会退化成线性结构



为什么HashMap的长度为2的n次方

主要是计算哈希值之后，需要将哈希值映射到数组上，自然而然会采用取余操作，但是计算机取余操作的效率比较低，位运算效率比较高，为了将`hash % n == hash & (n - 1)`，需要n为2的n次方。（好理解，n-1 = 111111111……）



HashMap不适用于多线程的原因

在并发环境下resize操作会形成一个循环链表从而导致死循环，Java8解决了这个问题，但是仍然有可能存在数据丢失的风险



遍历HashMap的几种方法：

1、迭代EntrySet	2、迭代KeySet	3、Foreach EntrySet	4、Foreach KeySet	5、map.forEach（lambda语法）	6、转换成Stream再foreach



HashTable和ConcurrentHashMap

一个是同步容器一个是并发容器

数据结构不同：HashTable数据结构与1.8之前的HashMap一致，ConcurrentHashMap的数据结构在1.8之前是分段数组`Segment[]`+链表的形式，在1.8之后和HashMap保持一致

HashTable：

![HashTable全表锁](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/HashTable%E5%85%A8%E8%A1%A8%E9%94%81.png)

ConcurrentHashMap（1.8之前）

![JDK1.7的ConcurrentHashMap](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/ConcurrentHashMap%E5%88%86%E6%AE%B5%E9%94%81.jpg)

ConcurrentHashMap（1.8之后）

![Java8 ConcurrentHashMap 存储结构（图片来自 javadoop）](https://gitee.com/SnailClimb/JavaGuide/raw/master/docs/java/collection/images/java8_concurrenthashmap.png)

实现线程安全的方式不同：

1、HashTable是使用一个全局锁，锁住了整个数组，每当有需要put或者get的时候都需要获取这个全局锁

2、ConcurrentHashMap在1.7之前是采用的分段锁，将数组进行了分割，每一把锁Segment（默认共16个，一旦初始化就不能改变了）仅仅只是保证一块儿位置，因此提升了并发性，在1.8的时候使用了Node数组+链表或者是TreeNode数组+红黑树的方式来实现，使用synchronized锁定链表或者红黑数的首节点和CAS操作后续节点来进行并发控制



ConcurrentSkipListMap

数据实现是跳表，在数据存储较多的时候，跳表的查询效率提升会非常明显

跳表是一种典型的用空间换时间的数据结构

且能保证插入的顺序对比于HashMap。

<hr>

ConcurrentLinkedQueue：使用链表实现的并发队列，可以看做一个线程安全的LinkedList

BlockingQueue：阻塞队列，是一个接口，内部通过数组、链表等方式实现了



Java实现的线程安全队列有两种实现方式：阻塞队列，代表为BlockingQueue接口下的一系列实现类，还有就是非阻塞队列，典型实现为ConcurrentLinkedQueue，使用的是CAS进行实现

ConcurrentLinkedQueue应该是高并发环境中性能最好的队列了，源于内部复杂的设计结构而不是CAS



BlockingQueue：被广泛运用在生产者消费者问题中，队列满时无法插入，队列空时无法取出，会被阻塞住，BlockingQueue主要有三个实现类：ArrayBlockingQueue、LinkedBlockingQueue、PriorityBlockingQueue

ArrayBlockingQueue使用数组进行实现，一旦创建容量无法改变，默认非公平，支持更高的吞吐量，可以通过构造函数第二个参数来实现是否公平（都是传入true为公平的）



LinkedBlockingQueue使用单向链表实现的阻塞队列（所以才是Queue不是Deque），如果没指定大小，则为Integer.MAX_VALUE，可以指定大小作为有界队列



PriorityBlockingQueue

支持优先级的无界阻塞队列，即会对其中元素进行排序，由于是无界队列，因此添加操作put是不会被阻塞的，只有获取操作take可能被阻塞。因为是有优先级的，所以需要对象实现comparable接口或者传入一个Comparetor，一样不能寸null值



核心API：

poll 取出队首元素，peek 返回队首元素，add 从队尾插入



使用集合的优势

为了存放数据，原来使用的是数组，但是数组特点单一，且一旦声明之后长度无法改变，因此出现了集合来进行数据的存储，Java集合框架不仅可以实现对象的存储，还实现了映射关系的存储





