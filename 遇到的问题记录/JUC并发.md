# 问题汇总



## 1、

问题描述：学Java虚拟机的内存模型的时候学到long和double分别存储于两个变量槽中，已知单个变量槽的修改操作是原子性的，long和double占据两个变量槽，想要模拟两个线程，一个修改高32位，一个修改低32位，出long和double不断修改后出现未知数值的情况

Demo

```java
public class Test {

    private static volatile int a = 65537;

    public static void main(String[] args) {

        new Thread(() -> {
            while (true) {
                if (a == 65538) {
                    a = 65537;
                    System.out.println("修改");
                } else if (a == 65537) {
                    continue;
                } else {
                    try {
                        throw new Exception("线程1出问题了"+a);
                    } catch (Exception e) {
                        e.printStackTrace();
                        break;
                    }
                }
            }
        }).start();

        new Thread(()->{
            while (true) {
                if (a == 65537) {
                    a = 65538;
                    System.out.println("修改");
                } else if (a == 65538) {
                    continue;
                } else {
                    try {
                        throw new Exception("线程2出问题了"+a);
                    } catch (Exception e) {
                        e.printStackTrace();
                        break;
                    }
                }
            }
        }).start();
    }

}
```

实验结果：

如果运行上述代码，出现以下两种情况：

```
修改
java.lang.Exception: 线程1出问题了65538
	at cn.luckycurve.Test.lambda$main$0(Test.java:20)
	at java.base/java.lang.Thread.run(Thread.java:834)
```

```
修改
修改
修改
修改
java.lang.Exception: 线程2出问题了65537
	at cn.luckycurve.Test.lambda$main$1(Test.java:38)
	at java.base/java.lang.Thread.run(Thread.java:834)
```



问题出现原因：满足了先检查后执行的竞态条件（还有就是：读取修改写入），导致爆出了异常

这里使用volatile是为了保证变量的可见性（基于happen before规则），不过后来在网上查询得知：由于Java内存模型保证声明为volatile的long和double变量的get和set操作是原子性的，实验失败。

解决方法：待续。。。