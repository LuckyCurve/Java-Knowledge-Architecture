# 问题汇总



## 1、

不算是问题，只是看着好玩，加进来

关于Java重载方法的选择问题（结论：如果有多个方法满足需求，选择最合适的重载方法即可）。

Demo：

```java
/**
 * @author LuckyCurve
 * @date 2020/5/10 21:41
 * 对Java重载的测试案例
 * 往往只能确定一个相对适合的方法
 */
public class OverLoadPro {

    public static void sayHello(Object arg) {
        System.out.println("hello Object");
    }

    public static void sayHello(int arg) {
        System.out.println("hello int");
    }

    public static void sayHello(long arg) {
        System.out.println("hello long");
    }

    public static void sayHello(Character arg) {
        System.out.println("hello Character");
    }

    public static void sayHello(char arg) {
        System.out.println("hello char");
    }

    public static void sayHello(char... arg) {
        System.out.println("hello char ...");
    }

    public static void sayHello(Serializable arg) {
        System.out.println("hello Serializable");
    }

    public static void main(String[] args) {
        sayHello('a');
    }
}

```

使用IDEA即可看出来调用的是哪个方法，然后把调用的方法注释掉，看下一个调用的又是哪个方法：

结论：char>int>long>Character>Serializable>Object>char...

《深入理解Java虚拟机：第三版》的418页有详细分析，图一乐就好。



 

## 2、

`String flag = new String("hello world");`到底会创建多少个对象

网上答案（有可信度）：1个或者2个（如果hello world字符串之前出现过，常量池中就已经存在了，就只创建了1个对象）

查看class文件字节码，如下：

![image-20200511154449868](images/image-20200511154449868.png)

（常量池中对象的创建发生在编译过程中，这里不会涉及到，这里是编译好的方法）

发生了如下几件事儿：

1. 为String对象在堆中分配内存，完成置零操作，并使其引用进栈
2. 复制栈顶元素（1中创建的String对象的引用），并重新压入栈
3. 把常量池中的"hello world"压入栈
4. 调用String的init函数，对String对象进行赋值（猜测：两次出栈操作，第一次出栈获取String的值，第二次出栈获取String的引用）
5. 将栈顶数值取出存入局部变量表下标为1的位置
6. 当前方法返回void，还有ireturn返回int等等。

没有问题。

==都只是理论模型，《Java虚拟机规范》中并没有规定虚拟机的实现，只要求预期的结果一致即可，但仍能让我们深刻的了解Java虚拟机的运行思想和逻辑==

> 上面的栈多指的是操作数栈，一般讨论的入栈出栈都指的是操作数栈，如果进出的对象是栈帧的话那就是虚拟机栈了

参考资料：[Class伪指令集网站](https://blog.csdn.net/weixin_40234548/article/details/81533673)





## 3、

学到Lambda表达式的时候就感觉到了，会默认创建一个类（有点匿名类的意思了），并把Lambda的方法体注入到类中去，并返回类的实例化对象。

既然Java支持动态类型检查，可以查看下该对象的实际类型：

选用我们最常见的Runnable函数式接口，分别使用Lambda和匿名类的方式来实例化该接口，并输出对象的实际类型

Demo：

```java
public static void main(String[] args) {
    Object o = "hello";
    System.out.println(o.getClass().getName());

    Runnable runnable1 = new Runnable() {
        @Override
        public void run() {
            System.out.println("hello world");
        }
    };
    System.out.println(runnable1.getClass().getName());

    Runnable runnable2 = () -> System.out.println("hello world");
    System.out.println(runnable2.getClass().getName());
}
```

输出：

```
java.lang.String
cn.luckycurve.demo.character3.InternalFunctionInter$1
cn.luckycurve.demo.character3.InternalFunctionInter$$Lambda$14/0x0000000100066840
```

获取两个Runnable的类加载器的名称，发现都是`app` ，而Object o的类加载器无法获取，使用时候会报错。





## 4、

实现将对象注入到容器当中，看上去不起眼的操作卡了半天，最后还是在StackOverFlow上提问才解决的。

常规解决方法是在方法上标注@Bean注解，然后将该对象作为方法的返回值，这样就实现了IoC容器的注入，但是如果返回值被占用了，这种方法就不起效了，也是我遇到的这种情况。

解决方法其实也找的差不多了，只是没有注意到重载的几个方法，整体思路就是操作ApplicationContext接口，将其转换为实现类GenericWebApplicationContext，调用registerBean方法即可。

刚开始一直想使用：

```java
public <T> void registerBean(Class<T> beanClass, Object... constructorArgs);
```

但是不想单独为了注入一个对象而为该类增加一个构造函数，是非常麻烦的。

后来的方法：

```java
@Autowired
ApplicationContext context;

@Test
void contextLoads() {
    Book book = Book.builder().id("222").build();
    GenericWebApplicationContext genericWebApplicationContext = (GenericWebApplicationContext) context;
    genericWebApplicationContext.registerBean(Book.class, () -> book);

    // 验证结果
    Book bean = context.getBean(Book.class);
    System.out.println(bean);
}
```

这里调用的方法是：

```java
public final <T> void registerBean(
			Class<T> beanClass, Supplier<T> supplier, BeanDefinitionCustomizer... customizers)
```

刚开始没反应过来，因为Supplier接口是Java8里提供的，用的不是很多，除非是在使用设计模式的时候我会想起来，用于函数的传递，返回一个beanClass对象即可。第三个参数BeanDefinitionCustomizer用于BeanDefinition的定制，如加上lazy-init或者是primary的特性。



