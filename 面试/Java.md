# 面试常见题目



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

在符合JVM规范的虚拟机中，如果boolean单独使用就占4个字节，如果以boolean数组使用就占用一个字节，但JVM规范也只是建议，实现仍然是看具体虚拟机。



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
|          |      有      |     0      |      127      |         2          |
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

|                   成员变量                    |    局部变量     |
| :-------------------------------------------: | :-------------: |
|              从属于类或者是对象               |   从属于方法    |
|  可以被public，protected、private、final修饰  | 只能被final修饰 |
| 存储于堆、元空间（HotSpot是使用堆来实现的）中 |   存储于栈中    |
|             非final类型存在默认值             |    需要赋值     |



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



