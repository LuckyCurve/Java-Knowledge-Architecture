> 因为学习Java也有一年半的时间了嘛，对Java并发、JVM也有了一定的认识，对Java8的语法也在项目中有所使用，读这本书是看看大师们对Java的理解和使用建议，带着自己的一点偏激和拙见，希望在读书过程中能产生思维的碰撞

  

  



# 第一章、序言



从自然语言的学习过程出发，掌握一门语言无非三点：语法、词汇、用法。在课堂上很容易学习到前两种，但是可能使用起来仍会让人感到忍俊不禁。

大多数编程语言的书籍也是着重介绍前两点，如语言的类型（是面向过程、对象、函数的）和词汇（标准库，数据结构，操作，工具类），对于用法往往只是让你知道怎么用，而不是针对如何高效的去使用。

> 可能在中国这种情况更为严重，因为国外都是直接谷歌best practice，一般都是官方给出的最佳实践，而国内往往则是技术论坛去了解一项技术，难免有失偏颇。  

  



作者是sun公司的一个骨灰级Java程序员，我一直感觉sun公司有一种神秘且美好的光环，对比于现在的Oracle来说，少了非常多的商业味道，各大组织纷纷参与到Java的开发和维护当中去，而现在呢，跟Apache基金会闹僵，Google更是直接想着另起炉灶了（Go语言），JVM也开始部分闭源了起来，但是也不能否认Oracle为Java做的贡献，Java8的历史意义还是存在的。

这作者就逆天，才发现是参加过Java5的Doug Lea领导的JUC包的源码书写过程当中去的，在我看来JUC包是更加偏向于使用算法实现CAS来保证线程安全的，这就是大佬吧，语言功底+算法，还有什么是干不了的呢？

初读这本书感觉和读设计模式的书的初衷差不多，都是为了更高效的进行程序的开发，但是设计模式都是多少年前的东西了，也无法无缝融入到Java当中来，另外Java的各种语言框架也可以自成体系，形成独立的一种开发范式。  



真的非常想看看作者对Java8的一些看法，虽然已经有好些年了，但是书籍具有一种显著的滞后性，权威的书籍更是少之又少，Java编程思想第五版现在还在试译当中呢。  



C++之父在Java刚面市时候的评价：

![image-20210317101203215](https://gitee.com/LuckyCurve/img/raw/master//img/image-20210317101203215.png)

不得不感慨即使没有从事到Java的开发工作中来，但对语言本身的敏感程度和我们确实是有着天壤之别的。  



从作者的前三版的前言中可以明显感觉到：作者对Java的热情有所降温，但是依然钟爱着Java，特别是当我也了解到Java的历史之后确实也有这种感觉了，更何况是当事人的作者呢  



最后再感慨一下吧，这本书的审校团队包含Java8的首席架构师和Doug Lea。确实是一本值得一读的好书，感谢国家没有把这本书也墙了。  



找到了一张Java发行名称和昵称的对应图了：

![image-20210317102752538](https://gitee.com/LuckyCurve/img/raw/master//img/image-20210317102752538.png)

  

  

  

# 第二章、创建和销毁对象

  

## 第１条：用静态工厂方法替代构造器



创建类的实例除了使用构造方法，我们还可以在类中提供一个静态工厂方法来进行返回，常见的像包装类当中的valueOf方法：

```java
public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}
```

静态工厂方法较之于构造器方法的优势在于

1、可以指定方法名字，实现见名知意的效果，增加可读性，如果有多个构造方法可以尝试使用静态工厂方法

2、不用每次调用方法的时候都进行对象的创建，独特的单例模式或者是使用缓存池的几个基本数据类型封装类都是不必每次都创建对象的

3、可以返回子类型的实例，特别是在Java8之后，常常使用在接口当中的默认方法中，我们通过接口中的静态工厂方法得到一个对象，然后通过该接口的API进行调用，这样就避免用户直接接触到子类，减小了使用负担，**甚至要求通过接口来引用被返回的对象，这是非常合理的**。

> 注意：不要将这些子类的实现也就直接放在接口当中了，因为接口当中默认是public的，还是老老实实创建一个包级别私有的类当中去然后使用

4、可以根据参数来返回不同的类，如传入初始化大小小小于8的参数，那么就返回一个底层数据结构为链表的实现类，如果是大于8就返回底层数据结构为红黑树的实现类，都是非常容易的

5、可以在调用时候才执行类加载，而不是像构造函数那样直接进行了类加载

   

缺点在于：

1、如果类不包含公有的或者是受保护的构造器，那么该类无法被继承

因为子类的创建是依赖于父类的构造方法的，但是直接把构造方法给私有化了，无法访问

书中说到了这也因祸得福了，在Java编程思想中也提到过尽量使用组合而不是继承

2、程序员难以发现这些静态工厂方法，因为通过Javadoc生成的文档仅仅只是将构造函数标粗，相信以后会有所改进。现在是通过命名的方式来尽量让静态工厂方法醒目，常见的如：from、of、valueOf、create等等。  

  

较之于构造方法、静态工厂方法各有优劣，不要一上来就想着直接提供构造方法了，还有别的选择。





## 第２条：遇到多个构造器参数时要考虑使用构建器



前面两种构造方法都无法很好的扩展到大量参数，如果存在大量的可选参数如食品包装外部的营养成分标签，那么通过重载构造函数或者是静态工厂方法都是非常困难的，此时我们倾向于使用重叠构造器，重叠构造器也就是我们创建多个构造函数，然后通过赋予默认值的方式来进行实现的，Demo如下

```java
public class Student {
    
    private final Integer id;
    
    private final String name;
    
    public Student(String name) {
        Student(null, name);
    }
    
    public Student(Integer id, String name) {
        // 赋值操作
    }
}
```

实际的赋值操作仅仅只是在全参构造函数中进行。



但是随着参数变多，会存在只想填写几个参数的情况，参数的值和顺序都需要严格按照构造函数中来，显然是不太好的。

第二种方法：JavaBeans模式

JavaBeans模式只需要提供一个无参函数和一系列的Setter方法，这样创建对象容易，设置属性也容易，可读性高。

但是Setter操作使得对象的不可变性不复存在，如果想使用final保证一致性也是不可能的，因为如果字段被final修饰需要在构造函数中赋初值或者是就地赋值，无法延迟到set方法调用的时候。这就可能需要程序员付出额外的努力来确保线程安全了。



所幸出现了建造者模式（Builder）来改善了这些问题，Demo如下：

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // Required parameters
        private final int servingSize;
        private final int servings;

        // Optional parameters - initialized to default values
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }

    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
                .calories(100).sodium(35).carbohydrate(27).build();
    }
}
```

可以发现外部的数据全是被final修饰的，且是基本数据类型，因此具有不可变性的特点。

如果我们需要使用Lombok来通过Builder创建不可变的对象的时候，可以参照如下链接：https://stackoverflow.com/questions/29885428/required-arguments-with-a-lombok-builder

大体上都差不多，还帮我们实现了一个xxxBuilder类，非常厉害的实现思路

> 可能会出现问题，因为有些框架是根据setter来完成属性注入的，不支持Builder也是非常可能的



不过我们现在都是使用对象进行数据存储嘛，应该都是提供getter/setter方法的，然后此时构造器更像是一种部分参数构造函数的替代品，不需要考虑安全性



Builder模式较之于传统的构造器，在保证不可变的前提下可以多次进行字段赋值

缺点也非常明显，需要新建一个Builder类，每次创建对象都会进行一个Builder对象的创建，在及其注重性能的场景是不可取的，并且一般只在参数大于等于4的时候考虑使用Builder，使用Builder还有一个隐式的好处：在参数进行增减的时候修改起来比较容易，如果使用传统的重叠构造器的方式的话那么需要进行大量的修改







## 第３条：用私有构造器或者枚举类型强化Singleton属性



第一种常见的方法（饿汉式）：

```java
public class Person {
    public static final Object INSTANCE = new Object();
    
    /**
     * 避免外部通过反射进行调用
     */
    private Person() {
        if (INSTANCe != null) {
            throw new IllegalCallerException();
        }
    }
}
```

使用时候直接使用`Person.INSTANCE`即可。

之所以这个没有使用开来，是因为在类加载的时候就会执行初始化了，没有延迟初始化的特性

但是如果这个类是需要序列化的，需要注意因为反序列化创建新的对象（将字段说明为transient即可），非常麻烦。

  

很多地方是非常不推荐使用双重检查机制的，像Java并发编程实战里面就是直接强烈谴责了，建议使用枚举（强烈推荐）或者是静态内部类的方式来进行：

```java
public enum Person {
    /**
     * 单例模式实现
     */
    INSTANCE;

    /**
     * 构造函数私有化
     */
    private Person() {

    }

    public static void main(String[] args) {
        Person person = Person.INSTANCE;
    }
}
```

枚举存在的限制就是这个类的父类必须是Enum，即不能有自己的父类





## 第４条：通过私有构造器强化不可实例化的能力

  

最主要的就是一些类中仅仅只包含static方法，因此这种类是不需要实例化的，我们可以通过类名.对象名或者是方法名来完成对方法的调用和静态对象的获取。

但是这种设计方法可能被滥用，因为这种编写出来的代码极度类似于面向过程变成，损失了面向对象的基本特性，这种设计方法可以适用于一些工具类如Math和Arrays、Collections等

工具类的设计是有讲究的，应该被声明成final类，~~但是并没有，估计是给我们就目前的工具类进行扩展的一种机会~~（并不是，因为构造函数被私有化了，子类无法调用父类的构造函数，因此子类也无法被实例化，直接会报错了，无论是否需要对象实例化）

如果不想类被实例化，可以使用私有构造器，然后在构造器中直接抛出异常，因为构造函数是可以抛出异常的，而且不用使用throws关键字在方法头中进行声明，保证在内部调用构造函数的时候依然会报错

这种设计模式也具有缺陷的，该类无法被子类化，因此工具类其实是无法被继承的





## 第５条：优先考虑依赖注入来引用资源



如果一个类需要依赖于资源，如密码破解类是依赖于字典这个资源的，那么不要使用静态工厂或者是Singleton，又或者是类似于工具类来通过大量的静态方法来完成，这些都是不现实的，应该使用依赖注入方式，实际上也就是通过构造函数来完成对资源的注入构成，然后将资源声明称final，是不可变的。这是依赖注入的一种体现。

由于依赖注入在项目中的依赖管理是极为复杂的，但是提供了一系列的依赖注入框架来帮助我们管理项目中纷繁复杂的依赖，极大的提升了类的灵活性，可重用性和可测试性。





## 第６条：避免创建不必要的对象



最好重用单个对象，因为重用既快速又流行，如果这些对象是不可变对象，则应该始终对他们进行重用。

极端反例：`String s = new String("hello world");`如果这个语句被用在了循环当中，那么将会在堆中创建多个对象，并且很可能触发GC，应该尽量使用`String s = "hello world";`来实现对字符串hello world的重用。

在使用提供了静态工厂方法和构造器的不可变类的时候，应该首先考虑静态工厂方法而不是构造其，Boolean类中两个方法如下：

```java
    @Deprecated(since="9")
    public Boolean(boolean value) {
        this.value = value;
    }

    @HotSpotIntrinsicCandidate
    public static Boolean valueOf(boolean b) {
        return (b ? TRUE : FALSE);
    }
```

可以看到静态工厂方法是重用了对象的，而构造器是无法重用的。

应该注意不要重复创建昂贵的对象，如需要进行正则表达式的判断，不应该直接使用String#matches方法，因为每次都需要将正则表达式Pattern，然后对Pattern进行操作。

此时由于Pattern是一个重量级对象，应该预先将正则表达式编译成Pattern然后再实现对字符串的判断。



另一种需要避免对象重复创建的模式就是Java提供的自动装箱拆箱机制，例如使用如下两段代码来统计执行时间：

```java
private static void calculate1() {
    long sum = 0L;
    for (int i = 1; i < Integer.MAX_VALUE; i++) {
        sum += i;
    }

    System.out.println(sum);
}

private static void calculate2() {
    Long sum = 0L;
    for (int i = 1; i < Integer.MAX_VALUE; i++) {
        sum += i;
    }

    System.out.println(sum);
}
```

第一个的执行时间为：721ms，第二个执行时间为8778ms，差距还是蛮大的。

结论：优先使用基本数据类型而不是包装类，要当心无意义的自动装箱。

> 难怪阿里的开发规范当中推荐在方法内部使用基本数据类型而在方法的出入参，以及其他的类属性使用包装类，因为使用包装类来避免默认数值的情况





## 第７条：消除过期的对象引用



由于Java语言的垃圾收集机制，我们不用像C/C++手动进行垃圾收集，但是仍然存在着隐式的问题，如在进行出栈操作时候就非常容易出现错误:

```java
public class Stack {
    private Object[] element;
    private int size = 0;
    
    // push操作存在扩容
    public void push(Object obj) {
        // Something TO DO
    }
    
    // 可能存在内存泄露
    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        // 置null操作，
        return element[--size];
    }
}
```

上述代码存在内存泄露问题，因为不断进行pop操作，处于element高位的元素不会被GC到，根据GC Roots判定仍是可达的对象，此时需要手动置null，这点在JDK的实现中也有相关注释来避免泄露。

> 这里没有使用到泛型，仅仅只是为了阐述这个内存泄露问题



另一个内存泄露的大头是缓存，如果实现缓存时候我们没有定期清理过期的key，很多value可能应该被回收但是还没有被回收，因为在Map中依然是可达的

**解决思路1：此时我们需要使用Doug Lea的一个类WeakHashMap，这个类相当于是仅仅持有key的弱引用，因此key可能被gc掉，如果key被gc了，那么此时形成key为null的一个Entry，WeakHashMap会自动删除这个Entry。**

解决思路2：给缓存设定一个过期时间，然后使用`ScheduledThreadPoolExecutor`这个类来完成实现对过期数据的一个清除，这样也可以完成，但是效率显然没有第一种弱引用的效率来的高，借助LinkedHashMap来完成元素的删除。



内存泄露的第三点是监听器及其他回调函数，主要是回调函数，感觉理解的不是很深刻，回调函数第一次接触是在AIO的时候，解决方法是持有弱引用，即使用WeakHashMap来存储注册到需要回掉的客户端。

> 不得不说作者对API的整体理解还是远胜于我们的，特别是集合框架以及提供的API，感觉都是在实战中使用到过的，非常厉害。
>
> 再次感叹下对并发编程大师Doug Lea的崇拜，以为对JDK的编码是基于5开始的，但是在WeakHashMap在JDK2的时候就进行了引入，作者正是Doug Lea





## 第８条：避免使用终结方法和清除方法



终结方法（finalize）通常是不可预测的，也是很危险的，一般不要使用它，会导致行为不稳定、性能降低、可移植性降低等问题，在Java9中已经明确废弃了，推荐使用Cleaner，但是仍有不可预测，运行缓慢等问题，一般情况下也是要避免使用的。

最主要的缺陷就是：当一个对象开始gc的时候，会执行finalize方法，但是无法保证能被及时执行，从gc到finalize的执行时间可以是任意长的，如果太依赖于终结方法，如想在终结方法中完成资源的close操作，那么也是不现实的，可能会导致大量资源没办法及时关闭。

使用终结方法还可能导致OOM问题，因为finalize没有执行对象是不会释放内存的，因finalize执行时间是不可控的，可能造成大量对象堆积。

如果是想要释放其他资源，可以实现接口AutoCloseable，然后使用Java7提供的try-with-resources语法来进行关闭，主要就是其中的close方法，实现close方法来保证持有资源的释放，而不是通过finalize或者是clean方法来进行的，注意，实现自己的close方法需要在对象的私有域中记录一个closed标志位，这样才能保证不会被重复关闭。

FileInputStream指定如下变量形式：

```java
private volatile boolean closed;
```

然后在其他的方法中都会对这个closed变量进行一个检查，来避免使用已经关闭的对象了，使用了volatile保证可见性，细节。

> 讲道理这里和阿里的开发规范明显冲突了。
>
> 可能阿里的开发规范仅仅只在效率和正确上做了一个权衡，而JDK API更倾向于效率，因此使用基本数据类型来避免拆箱装箱



当然也有适合的场景，不然也不会出现这么危险的方法给程序员了，Java语言毕竟也是以安全性著称的，谁没事儿会打自己的脸呢。



似乎有点理解为什么要使用Cleaner方法来代替finalize方法了，因为finalize是`protected`的，是无法在外部直接调用的，只能由虚拟机在GC的时候进行调用，但是这就造成了被调用时间的不确定性，因此我们需要使用一种确定的调用方式，也就是Java9提出的Cleaner类来完成对资源的一个清理。一般会让当前类实现一个AutoCloseable接口，然后在close方法内部调用`Cleaner.Cleanable#clean`方法，在使用资源的时候就可以直接使用7提出的try-with-resource语法来进行资源的自动释放。

这里使用Cleaner也是对我们资源释放起到了弥补的作用，如果我们没有调用close方法，那么在最后JVM仍然会帮我们调用clean方法。



书上是将资源和关闭资源这个动作封装成一个静态State类（通过实现Runnable接口来进行实现的），但是一定要是静态的，因为如果不是静态类，那么会持有一个外围对象的一个引用，导致外部类无法被回收，也不要使用Lambda表达式来进行资源关闭的逻辑书写，因为也是很容易捕获到了外部的this引用。

这种资源在不安全释放时候具有强烈的不确定性，如果使用如下编程范式，那么会得到执行：

```java
try(Resource resource = new Resource()) {
    // TO DO
}
```

但是如果仅仅只是使用`Resource resource = new Resource();`那么有可能不会执行。



总之终结方法和清理方法是具有强烈的不确定性和性能损失的，能不用尽量不用。





## 第９条：try-with-resources优先于try-finally



许多类在使用时候都需要我们调用close方法来手动关闭资源，但是我们有可能忘记关闭资源，好多都是使用终结方法来作为安全网（但是在Java9中直接不推荐使用终结方法，这些类里原来finalize代码都直接清空了）。



















