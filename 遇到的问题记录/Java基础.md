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



