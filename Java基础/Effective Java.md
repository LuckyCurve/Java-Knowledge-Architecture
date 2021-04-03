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







# 第三章、对于所有对象都通用的方法



说白了就是Object中的非final方法进行继承时候可能产生的一些问题，还包括一个较为通用的方法Comparable.compareTo也具有类似的特性，一起讨论了。





## 第10条、覆盖equals时请遵守通用约定



> 上来作者就经典言论：使用equals非常容易犯错，最好的办法就是不要使用equals
>
> 严重怀疑作者写这本书和参与Java并发编程实战这本书的书写相差多少，记得其中也有并发编程非常容易出错，最好的办法就是不要使用并发编程。
>
> 还是外国作者写书都有这个习惯，感觉蛮可爱的



似乎有点理解为什么先说最好能别用就不用了，和我对设计模式的理解是一样的，宁缺毋滥。

直接在这一节的头部就给出了几个条件可以不用重写equals：

- 类的实例本质上都是唯一的：如Thread类，逻辑上是不可能存在两个Thread内存不等但是实际相等这种情况的。
- 类没有必要提供逻辑相等的测试功能：如对正则表达式当中的Pattern则完全没必要使用equals方法来判断这两个对象时候是逻辑相等的。
- 父类已经覆盖了equals，并且这个equals的判断逻辑对该类也是合适的：最常见的就是集合当中的equals了，如List中的equals大部分来自于父类AbstractList，Set大部分来源于AbstractSet



剩下的情况基本都是要完成对equals方法的覆盖的，但是有一些值类如枚举类型就不需要完成对equals方法的覆盖，因为对这种类来说逻辑相等和对象等同是一个概念。

需要保证几大特性：自反、对称、传递、一致，需要保证这些特性因为对象进行传递特别是往集合中进行传递的时候，需要保证这几个 特性要不然很容易出错。



不要异想天开在equals方法中尝试使用getClass方法来代替instanceof关键字来进行对象类型的比较，`o.getClass() == getClass()`，这样有可能导致o是当前类的子类从而出错。

因为里氏替换原则中说道：为该类编写的任何方法，在他的子类型上也应该同样运行地很好



构建equals方法的四个步骤：

1. 使用==操作符检查**参数是否为这个对象的引用**
2. 使用instanceof操作符检查**参数是否为正确的类型**（这一步就避免了NullPointerException）
3. 把参数转换为正确的类型
4. 检查参数中的域是否与该对象中的关键域相匹配

关键域的比较：对于基本数据类型除去float和double，可以直接调用==进行比较，对于这两个浮点数，当然可以使用包装类的equals进行比较，但是存在装箱操作，因此使用静态方法compare来完成float和double的比较。对引用的比较则使用Objects.equals方法来完成数据比较。

并且比较顺序是尽可能先的比较差别较大的数据。

可以尝试使用Google的AutoValue框架来进行代码的生成，非常类似于我们使用到的Lombok，可以直接通过注解的方式来减少我们的代码书写，但是强迫该类是抽象类





## 第11条、覆盖equals时总要覆盖hashCode



主要是hashCode和equals在Object类当中就有了约定，如果不遵守这个约定，那么很有可能基于这些约定的集合无法正常运转，这些类主要就是HashSet、HashMap

简单来说约定就是三条：

- hashCode在同一个程序运行中不会改变
- equals的对象hashCode一定相同
- 不equals的对象hashCode不一定不同



计算hashCode比较好的定式为：

遍历所有关键域，对每个关键域都取hashCode，如果是第一个就直接赋值到res，剩下的`res = 31 * res + hashCode`

例子：

```java
public class Student {
    private Integer age;
    private String name;
    private Integer score;
    
    @Override
    public int hashCode() {
        // 如果是基本数据类型：
        // hashCode = Integer.hashCode(age);
        // 如果是数组
        // hashCode = Arrays.hashCode(array);
        int res = age.hashCode();
        res = res * 31 + name.hashCode();
        res = res * 31 + score.hashCode();
    }
}
```

这样对绝大多数的应用程序而言已经够了。因为大部分的JDK类都是作者编写的，也是按照这个思路进行处理了，最能体现的就是Arrays#hashCode方法：

```java
public static int hashCode(Object a[]) {
    if (a == null)
        return 0;

    int result = 1;

    for (Object element : a)
        result = 31 * result + (element == null ? 0 : element.hashCode());

    return result;
}
```



如果我们觉得上面的hashCode写的麻烦，可以尝试使用Objects#hash方法：

```java
public static int hash(Object... values) {
    return Arrays.hashCode(values);
}
```

使用实例：

```java
public class Student {
    private Integer age;
    private String name;
    private Integer score;
    
    @Override
    public int hashCode() {
        return Objects.hash(age, name, score);
    }
}
```

可能效率较之于我们手码的效率要低下一些，因为涉及到基本数据类型都会进行装箱拆箱操作，涉及到数组的hash值计算也会直接将数组中的所有元素考虑进来

对不可变对象，因为关键域都是不可变的，没有必要进行多次计算，可以将数值缓存起来，Demo如下：

```java
public final class Student {
    private final Integer age;
    private final String name;
    private final Integer score;
    
    private int hashCode;
    
    @Override
    public int hashCode() {
        // 引入本地变量记录hashCode，避免并发问题，确实对基本数据类型也是一个避免并发的好办法
        int res = hashCode;
        if (res == 0) {
            res = Objects.hash(age, name, score);
            hashCode = res;
        }
        
        return res;
    }
}
```

不要想着通过删减关键域进行哈希运算来提高hashCode的执行速度，很可能得不偿失

现在可以使用Lombok来直接生成hashCode方法了。

> 不得不说作者从开发者的角度完全考虑到了用户可能怎么调用这些库函数，并且给出了最佳的一种调用方案和可能出现的错误







## 第12条：始终要覆盖toString方法



Object的toString方法默认输出：全限定类名+@+hashCode的十六进制表示，测试方法如下：

```java
Object o = new Object();
System.out.println("hashCode：" + Integer.toHexString(o.hashCode()));
System.out.println(o);
```

但是toString方法的初衷是输出一个简易的但信息丰富的，易于阅读的表达式，建议所有的子类都覆盖这个方法：

`It is recommended that all subclasses override this method.`出现在toString方法注释上。

> 读书难免会带着批判的眼光，但是对这本书，可以放松一点对书籍内容的质疑了，再次感叹作者对Java库和并发的理解。

toString方法不仅在我们日常使用中调用很多，在错误输出的时候，很可能直接打印对象，这时候如果有优越的字符串，那么就会让线上Debug变得非常简单。

存在一个观点：toString产生的字符串是否需要对外部指出具体的格式，如果指定了具体的格式，我们需要提供一个构造器或者是静态工厂方法来完成通过String的对象构建

> 感觉很有点序列化和反序列化的意思了。

但如果指定格式，那么需要慎重定义格式，因为在以后的发行版本中都需要维持这个格式

> 真的是体验过版本迭代的过程才能发出的肺腑之言：如果将来的发行版本中改变了这种表示法

无论时候指定格式都应该表明toString的意图



在最后也是推荐使用像是Lombok等自动生成工具来生成toString方法，只不过推荐的是Google的AutoValue工具







## 第13条：谨慎地覆盖clone



由于Object中的clone方法是protected的，我们在外部无法直接调用clone方法来完成对象的克隆操作，因此我们需要实现自己的clone并且扩大访问权限。

使用clone方法需要对象实现Cloneable接口，否则会抛出CloneNotSupportException异常，**这种模式不值得效仿，通常情况下，实现接口是为了表明类可以为它的客户做些什么，然而，Cloneable接口改变了父类中受保护的方法的行为**

> 确实是这样的，使用起来就感觉有点奇怪，果不其然，极端的实现案例，估计是为JVM的性能让步

:question:为什么clone方法被设置为protected

Answer：https://stackoverflow.com/questions/1138769/why-is-the-clone-method-protected-in-java-lang-object

不一定对，没几个人有权能对Java语言的Object类设计说三道四，都只是发表一下自己的看法而已：

The Clonable interface is just a marker saying the class can support clone. The method is protected because you shouldn't call it on object, you can (and should) override it as public

感觉仅仅只是因为这个方法不太安全或者是效率太低，如果我们需要使用，得手动Override这个方法，将访问权限改成public然后再使用。

突然有了些自己的理解，对有些类如不可变类，在本书当中说道：**不可变的类永远都不应该提供clone方法，因为他只会激发不必要的克隆。**然后通过这样的设计，可以有几层安全防护如果我们没有提供clone方法，第一层protected防护，第二层反射防护（通过抛出CloneNotSupportedException异常的方式来进行防护）



重写时候的clone方法模板：

```java
public class Data implements Cloneable {
    @Override
    public Data clone() {
        try {
            return (Data) super.clone();
        } catch (CloneNotSupportedException e) {
            // 不可能发生
            throw new AssertionError();
        }
    }
}
```

几个注意点：

1、重写的方法的访问范围可以比原来的大

2、返回值可以是限定返回值的子类

3、抛出异常可以是限定抛出异常的子类

采用以上这种写法可以避免1、异常抛出、2、类型转换



clone方法仅仅只会将内部关键阈进行值的传递，如果是基本数据类型就会进行值的传递，但是如果是引用数据类型，那么就会直接进行简单的引用传递，不会就行深克隆

为了达到正确性的目的，我们可以使用最简单的解决方法——在clone方法中递归调用

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    
    
    @Override
    public Stack clone() {
        try {
            Stack res = (Stack) super.clone();
            // 进行内部对象的克隆，这里因为Array也是按照这个方式进行书写的，可以直接返回Object数组，不用我们再进行墙砖
            res.elements = elements.clone();
            return res;
        } catch (CloneNotSupportExceptioin e) {
            // 理论上是不可能发生的，因此直接抛出Error
            throw new AssertionExecption();
        }
    }
}
```

> Error是可以不用捕获处理的，类似于RuntimeException异常

如果此时elements被final修饰了，那么就无法使用上述这个方法了，final域与Cloneable的设计师存在冲突的，无法相互兼容，因为一旦赋初值了就无法改变了。

参考ArrayList的clone方法：

```java
public Object clone() {
    try {
        ArrayList<?> v = (ArrayList<?>) super.clone();
        v.elementData = Arrays.copyOf(elementData, size);
        v.modCount = 0;
        return v;
    } catch (CloneNotSupportedException e) {
        // this shouldn't happen, since we are Cloneable
        throw new InternalError(e);
    }
}
```

但是这里没有进行转型操作。



设计的clone方法终究只是浅拷贝，可能是出于性能的原因吧，没有直接在native方法中实现深拷贝，感觉如果Java真的想要实现深拷贝，在JVM层面应该还是非常容易完成的（说白了对象只是一段内存，复制这段内存到一个新的地方，然后返回就行了）。



使用CLone架构最多也只能做到这样了，另一种思路是：

我们直接创建一个对象，然后将对象中所有的关键域按照当前这个需要克隆的对象来进行遍历赋值操作。

如果可克隆的类有了子类，但是不想要克隆这个属性，但是由于实现了Cloneable接口，那么clone方法调用是不会出错的，~~此时我们需要在子类中Override这个clone方法并且将这个方法的访问权限改成protected~~，并且在方法体当中直接抛出CloneNotSupportException异常

> 一旦子类扩展了这个方法的访问权限，那么他的子类也只能在他的基础上完成方法访问权限的扩展
>
> 即不存在protected——public——protected的转换过程



对工具类应该精良去覆盖clone方法，并且在clone方法的方法体当中抛出CloneNotSupportException异常，避免在外部通过反射的方式来对这个方法惊醒调用。

> 突然想到了JVM重写clone方法并不容易，并不只是将对象的内存数据直接复制，因为在其中仅仅只是存放着指向堆内存的一个引用，如果需要深拷贝也需要将这部分进行一个复制，这样就存在着一个递归操作了。



如果需要为线程安全的类编写clone方法，那么应该在clone方法上加上synchronized关键字，因为如果不加synchronized可能会导致在执行clone操作的时候，只有对象的部分数据发生了修改，处于一种不安全的状态。

> HashTable竟然自己实现了Cloneable接口，但是支持并发的ConcurrentHashMap并没有实现Cloneable接口，HashMap继承了Cloneable接口



不依赖于原生的CLoneable框架，更好的办法是使用拷贝构造器或者是澳贝方法的形式来完成对象的创建，使用这种方法可以减少很多麻烦，包括但不限于对象类型转换，异常处理



**复制功能最好由构造器或者工厂方法来进行提供，而不是通过实现Cloneable接口，然后重写clone方法来进行**，但这条规则的绝对例外是数组，数组最好还是使用clone方法来进行复制。





## 第14条：考虑实现Comparable接口



类实现Comparable接口表示类具有内在的排序关系，排序关系是由Comparable中的compareTo方法来决定的

Java当中所有的值类以及所有的枚举类型都实现了Comparable接口，如果你在实现一个值类，并且他具有非常明显的内在排序关系，那么就需要你考虑实现Comparable接口了。

Java当中的TreeSet、TreeMap的数据存储依赖于Comparable接口，并且工具类Collections和Arrays的搜索和排序算法依赖于Comparable接口。



一点强烈的建议：compareTo方法应该和equals方法施加等同的测试，即返回相同的结果，因为有些集合会按照equals方法来进行判断（Hash），有些会按照compareTo方法来进行判断（Tree），应该尽最大的努力去争取，当然这并不会造成多大的问题。



测试Demo：

```java
// 可以替换为HashSet，结果会输出 两个数字，这个只会输出一个数字
static Set<BigDecimal> set = new TreeSet<>();

public static void main(String[] args) {
    set.add(new BigDecimal("1.0"));
    set.add(new BigDecimal("1.00"));

    System.out.println(set);
}
```

> 使用BigDecimal类推荐使用`public BigDecimal(String info)`来完成对象的初始化 ，而不是使用参数为double的来进行初始化
>
> 之所以使用字符串来进行数据初始化还是因为double数据类型数据在存储的时候可能会导致小数点丢失的问题。



equals和compareTo最主要的区别在于方法意义，一个是比较是否相等，进行等同性的比较，另一个是进行大小的比较，除此之外，调用时候传入null值的处理方式应该是不一样的，equals因为有固定的`if (obj instanceof Class)`固定编程范式，会直接返回false，而compareTo方法不需要对值进行特殊处理如对null值进行单独处理。



对所有的基本数据类型的比较，我们可以直接使用基本数据类型的封装类的静态方法compare，直接使用就可以了。

如果一个类拥有多个关键域，那么这个类的compareTo方法的实现应该是从最关键的阈开始比较，降低关键级别直到所有的域对比完，如果某个阈产生了非零的比较结果，那么直接将这个值进行返回，如存在以下类需要进行compareTo方法实现：

```java
public class PhoneNumber implements Comparable {
    
    // 关键域
    private Short areaCode;
    private Integer prefix;
    private Integer offset;
    
    @Override
    public int compareTo(PhoneNumber number) {
        int res = Short.compare(areaCode, number.areaCode);
        if (res == 0) {
            res = Integer.compare(prefix, number.prefix);
            if (res == 0) {
                res = Integer.compare(offset, number.offset);
            }
        }
        return res;
    }
}
```



可以借助Java8的Comparator来完成比较器的构建：

```java
public class PhoneNumber implements Comparable<PhoneNumber> {
    private Short areaCode;
    private Integer prefix;
    private Integer offset;
    // 如果不是基本数据类型，但是实现了Comparable接口
    private Comparable obj;


    @Override
    public int compareTo(PhoneNumber o) {
        // 创建一个比较器，需要我们自己指定待比较的对象的数据类型
        Comparator<PhoneNumber> comparator = Comparator.comparingInt((PhoneNumber number) -> number.areaCode)
                .thenComparingInt(number -> number.prefix)
                .thenComparingInt(nummber -> nummber.offset)
                .thenComparing((o1) -> o1.obj.compareTo(this.obj));

        return comparator.compare(this, o);
    }
}
```

如果此时obj关键域没有 指定compareTo方法实现，那么可以递归使用thencomparingInt等方法来实现比较。