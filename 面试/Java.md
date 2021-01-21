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

