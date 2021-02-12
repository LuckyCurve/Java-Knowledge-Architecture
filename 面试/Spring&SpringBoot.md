# Spring面试常见问题



什么是Spring框架

准确的说是Spring Framework，Spring Framework是一种轻量级开发框架，旨在提高开发人员的开发效率以及系统的可维护性

一些重要的模块如：spring-core，spring-context，spring-aop，spring-orm



Spring的两大特性：IoC 和 AOP

IoC，控制翻转或者是依赖注入，是一种设计思想，旨在将对象的控制权交给Spring框架来进行处理，在需要的时候可以直接完成对象的注入，而不是我们手动new，IoC容器实际上就相当于一个Map，当我们需要对象的时候会根据特定的key来取出value的值。

在Spring3.0前期仍然推荐使用XML的方式来配置Bean到IoC容器当中，但是随着3.0注解的大量提出和4.0对注解的完善，使得Spring放弃了原本适配良好的XML格式数据转用注解方式，此时Spring也提出了使用注解代替繁琐的XML配置。

几种Bean Definition的注册方法：

- 配置文件方式：主要是XML和properties两种，其中XML使用的更多，在我们想引入一个XML文件的时候，只需要给Spring可以扫描到的类上加上@ImportResource即可完成XML的注册
- Java Configuration方式：通过Java配置类的方式来完成对象的注册，主要是通过在方法上标注@Bean注解和在方法中注入ApplicationContext然后通过ApplicationContext注册bean



ApplicationContext和BeanFactory

ApplicationContext是在BeanFactory之上构建的，基本上完整继承了BeanFactory的所有功能如依赖注入操作等等，此外还提供了高级功能如事件的发布，国际化支持，不过有一点区别：BeanFactory强调速度，因此延迟初始化，ApplicationContext强调功能，防止用着在注入时候功能有问题，因此运行时候全部注入。



在对Bean进行注入的时候，需要指定作用域，一共有五种：

singleton单例、prototype每次获取对象都直接创建对象、还有三个需要是在web环境下才能生效：request每个Request都有自己的一份实例、session每个session拥有一份对象实例、globalsession：在基于Servlet的Web中使用就相当于是Session



容器的启动加载阶段

如果是BeanFactory就采用延迟加载策略，不直接加载所有注入其中的bean，如果是ApplicationContext则直接全部进行初始化，在从XML或者是JavaConfiguration中读入内容的时候会查看是否要进行AOP操作，如果注册了BeanFactoryPostProcessor则需要对创建BeanDefinition过程前后进行前置和后置操作。



Bean的实例化过程/生命周期：

![image-20210211160512166](https://gitee.com/LuckyCurve/img/raw/master//img/image-20210211160512166.png)

- BeanFactory读入BeanDefinition，主要是从XML，JavaConfiguration中获取的信息
- 实例化Bean对象，通过反射的方式创建一个Bean对象（构造器注入）
- 调用对象的Set方法完成对象属性赋值（Setter注入）
- 检查自省接口，完成自省操作如对name的注入，对ClassLoader的注入，还有一些其他的Aware
- Spring IoC提供的一种特性，对创建的对象进行AOP操作，通过执行容器中BeanPostProcessor中的方法完成
- 判断bean是否实现InitializingBean接口，如果实现了就执行其中的方法，记得接口中只存在一个方法
- 检查在XML中是否指定了init-method如果指定了就进行调用
- BeanPostProcessor的后置函数调用

Bean的实例化过程到此结束，但是生命周期还剩下销毁时候的两步，如果实现了注销Bean的接口，会调用其中的方法，如果配置文件指定了destory-method会执行。



Spring中的单例对象其实是存在线程安全问题的

可以有多个对象同时获得Spring中bean的引用，但是还好我们常用的Bean如Controller、Service、Maper都是无状态的，所以无需担心线程安全问题

当然也有一些bean是有状态的，解决方法有：

ThreadLocal或者更改Bean作用域为prototype



常用注解：

对象的装配：@Component及其派生注解（@Configuration、@Controller、@Service、@Mapper），@Bean（标注在方法上，将返回值注入到容器当中）

对象的注入：@Autowired只能根据type注入、@Resource、@Qualifier可以根据name注入，并且推荐使用@Resource，因为是JSR标准，而@Qualifier是Spring提供的注解



AOP：面向切面编程，为业务模块调用共同的处理逻辑如事务处理，日志管理等

Spring AOP是基于动态代理的，实现方式有两种：JDK动态代理和CGLIB动态代理，JDK动态代理需要我们代理的方法是实现的接口当中的，因为经过JDK代理之后只能返回一个接口的实例化对象，CGLIB的代理策略则是返回当前对象类的一个子类实例化对象来实现代理的，没有那么多限制条件。之所以两者并存，还是因为JDK动态代理比较快，JDK动态代理的两个核心类：Proxy（创建接口的一个实例化对象）和InvocationHandler（定义处理逻辑），CGLIB的核心类为MethodInterceptor和Enhancer



Spring AOP和AspectJ AOP的区别

AspectJ算是Java生态中最完整的AOP框架了，Spring AOP集成了AspectJ，但两者之间仍有差别，其实就是JDK代理和CGLIB代理的差别，Spring AOP中包含了运行时增强（JDK方式）和编译时增强（CGLIB动态字节码操作技术），而AspectJ只涉及到编译时增强



走过了IoC和AOP两种特性，接下来就是Spring中一个比较重要的模块了：Spring MVC

Spring MVC主要是对MVC实现了解耦，让程序开发更高效，并且与Spring框架的天然结合非常有利于我们使用

主要的请求流程：

客户端发送请求经过DispatcherServlet——DispatcherServlet调用HandlerMapping，解析使用哪一个Handler——解析出Handler之后（也就是我们说的Controller），使用HandlerAdaptor适配器来对Handler实现调用——调用完成后，如果方法或者类上标注了@ResponseBody，直接将数据写入Http的Body部分，直接返回——如果没有标注，则返回一个ModelAndView对象——ViewResolver找到对应的视图，完成视图的渲染工作，并最终返回View给浏览器



Spring事务相关知识点：

主要使用声明式事务，包含：基于XML的声明式事务，基于注解的声明式事务

事务的隔离级别：MySQL中默认的四个隔离界别外加使用一个数据库当前默认的隔离级别

七种传播特性，传播特性的定义主要是为了如方法A调用了方法B，此时如果方法AB同时开启事务那么如何处理：

- 如果不存在外层事务，就主动创建事务，否则使用外层事务
- 如果不存在外层事务，就不开启事务，否则使用外层事务
- 如果不存在外层事务，就抛出异常，否则使用外层事务

<hr>

- 总是主动开启事务，如果存在外事务，则将外层事务挂起
- 总是不开启事务，如果存在外层事务，就将外层事务挂起
- 总是不开启事务，如果存在外层事务，则抛出异常

<hr>

- 如果不存在外层事务，就主动创建事务，否则创建嵌套的子事务



在Spring中使用注解@Transactional开启事务，保证事务的ACID特性，@Transactional可以标注在类上，也可以标注在方法上，默认是回滚的RuntimeException，一般需要使用rollbackfor指定Exception异常就回滚



可能出现自调用导致事务失效的问题，归根结底就是自调用导致了AOP的失效，这时候自调                                                 用需要使用Autowired注解注入一个当前类的增强后的对象，对这个对象进行方法调用。



Spring使用到的设计模式：

工厂模式：BeanFactory和ApplicationContext

代理模式：Spring AOP

单例模式：Spring中的Bean作用域为singleton，是使用ConcurrentHashMap的方式实现的

模板方法模式：jdbcTemplate

责任链模式：SpringMVC中的HandlerChain

适配器模式：SpringMVC中的HandlerAdaptor

观察者模式：Spring的事件驱动模型，





# SpringBoot面试常见问题



介绍Spring的优缺点，为什么有了Spring还需要SpringBoot

Spring是一款轻量级的开发框架，作为重量级的EJB的替代品，主要具有两个特性IoC和AOP

优点：轻量级、生态环境好、具有完整的社区生态

缺点：配置繁琐，虽然2.5引入了基于注解的配置代替了部分XML，但是仍然使得程序员的大部分精力花费在了框架环境的搭建上而不是业务逻辑的书写上。

至于SpringBoot，则是对Spring配置的一种简化，实现了开箱即用的功能，强调约定大于配置，让开发者可以将更多的精力集中在业务逻辑的编写上而不是框架本身。



SpringBoot的优缺点：

优点：自动配置、对开发生态的整合、提供嵌入式HTTP服务器如Tomcat、Jetty、Undertow（进行切换只需要移除web模块中的tomcat starter，进行额外引入即可），为Maven和Gradle等项目构造工具提供了多款插件

缺点：版本更替快，学习成本高、将传统的Spring项目迁移过来很难、破坏性的更替。



什么是SpringBoot Starter

Starter是对一系列依赖的整合，我们在进行特定环境的开发时候只需要直接引入SpringBoot整理好的提供给我们的依赖即可完成。



@SpringBootApplication注解

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
```

@SpringBootApplication相当于@Configuration（允许使用@Bean注入方法的返回值）、@EnableAutoConfiguration（开启自动配置）、@ComponentScan（扫描该包及其子包下的所有需要加载的Bean）。



**SpringBoot自动装配流程**

SpringBoot自动装配是通过@EnableAutoConfiguration来实现的，在@EnableAutoCOnfiguration上Import了一个AutoConfigurationImportSelector类到容器当中，这个类的核心方法是getAutoConfigurationEntry，具体步骤如下：

1、检查自动装配开关是否打开，在application配置文件中的`spring.boot.enableconfiguration`默认是开启的

2、获取@EnableAutoConfiguration注解指定的属性，如exclude和excludeName

3、获取需要装配的所有配置类，位于类路径下的`MATE-INF/spring.factories`和所有Spring Boot Starter下的`MATE-INF/spring.factories`（有些是需要使用@Enable注解去唤醒的，比如@EnableEurekaServer），只有扫描到的类满足了@Conditional注解的条件才会被加载进容器当中，否则会直接丢弃

4、经过进一步检查如Filter等，将剩下的`Set<String>`集合直接装载



SpringBoot导入配置文件

XML文件：@ImportResource

properties文件：@PropertySource实现对properties配置文件的读取，注入即可使用@Value注解等，但是无法读取yaml文件，如果存在覆盖问题，以application配置文件为主

当然更推荐使用@ConfigurationProperty方式完成字段的注入

SpringBoot默认配置的优先级：项目目录下的`config/application.yml`——resources目录下的`config/application.yml`——resources目录下的`application.yml`



SpringBoot中的定时任务可以使用@Scheduled注解和@EnableScheduled注解来开启定时任务。



SpringBoot的核心注解：

@SpringBootApplication：@Configuration+@ComponentScan+@EnableAutoConfiguration

@Autowired，@Qualifier，@Resource对象注入，支持根据name注入bean

@Component，@Controller，@Service，@Mapper，@Configuration

@Controller，@RestController

@Scope：声明对象的声明周期

@RequestMapping，@GetMapping，@PutMapping，@PostMapping，@DeleteMapping

@PathVariable：获取路径参数

@RequestParam：默认的，获取URL中带的参数

@RequestBody：读取Request请求的body部分，使用Convertor将请求body中的JSON数据转换成Java对象

@value和@ConfigurationProperties：读取配置信息

@ControllerAdvice：注解方式定义全局异常处理类

@ExceptionHandler：注解方式声明异常处理方法

@Transactional：事务（如果标注在测试方法上就会执行完毕之后直接回滚，避免污染数据）

JSON数据格式化处理：@JsonIgnoreProperties类上和@JsonIgnore

JSON格式化数据：JsonFormat

