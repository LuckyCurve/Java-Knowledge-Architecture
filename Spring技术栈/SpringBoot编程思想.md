# 第一部分、总览Spring Boot





## 第一章、初览Spring Boot



Spring Framework对传统Java EE的颠覆方式为新的概念，即两种技术IoC和DI

其实Spring Framework也是在重复造轮子，放眼JSR规范，IoC的实现方式为JNDI（Java命名和目录接口），而DI则是EJB容器注入

Spring Framework的局限性：由于自身并非容器，基本上不得不随着Java EE容器启动而装载，但SpringBoot的出现改变了这一局面

SpringBoot具备开发（Dev）和运维（Ops）双重特性







## 第二章、理解独立的Spring应用



SpringBoot采用了嵌入式容器，独立于外部容器，对应用生命周期拥有完全自主的控制，从此改变了寄人篱下的境地

> 误区：很多人都以为SpringBoot嵌入式Web容器启动时间少于传统的Servlet容器，实际上没有证据表明，只是一种更快捷的启动方式，提升了开发效率而已



口头上习惯将Spring Boot开发的应用称为SpringBoot应用，实际上是Spring应用，对应着标题。

在原来Web容器外部化的时候，Spring应用上下文是依附于Web容器，Spring应用上下文的操作执行例如初始化操作destroy操作都需要结合回掉函数来达到，但是随着SpringBoot的出现，Web容器从属于Spring应用上下文，此时驱动Spring应用上下文启动的核心组件是SpringBoot核心API SpringApplication，所以是Spring应用，也可以称为SpringBoot应用。



接下来讲述了在非图形化环境下完整开发SpringBoot的HelloWorld例子，非常厉害了

windows集成了tree命令，可以直接展示文件树

图形化界面开发SpringBoot程序



只有在创建可执行JAR的时候才需要使用到Spring-boot-maven-plugin到pom中，如果直接使用maven命令启动则无需此插件

java -jar方式运行的jar文件和mvn spring-boot:run命令基本无异，只是前者是生产环境后者是开发环境



打包后生成两个文件：

- .jar.original：Maven原始打包文件，不包含第三方依赖。
- .jar：加工后的文件，包含第三方依赖，springboot-plugin帮的忙



> JAR文件有时也被称为“Fat JAR”，采用zip压缩格式存储，可以解压zip的也可以解压JAR

其中文件的路径：

![image-20200808112711234](images/image-20200808112711234.png)



使用Java -jar命令运行JAR文件时候，就是标准的JAR执行操作了，会读取MANIFEST.MF文件作为启动引导类。

```
Manifest-Version: 1.0
Created-By: Maven Archiver 3.4.0
Build-Jdk-Spec: 11
Implementation-Title: Community Service
Implementation-Version: 0.0.1-SNAPSHOT
如果是WAR包就是WarLauncher
Main-Class: org.springframework.boot.loader.JarLauncher
Start-Class: com.applet.Application
Spring-Boot-Version: 2.2.6.RELEASE
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
```

使用`java -jar`命令启动程序是通过引导类JarLauncher来启动主类Start-Class



手动引入spring-boot-loader项目查看JarLauncher

```java
public class JarLauncher extends ExecutableArchiveLauncher {
    static final String BOOT_INF_CLASSES = "BOOT-INF/classes/";
    static final String BOOT_INF_LIB = "BOOT-INF/lib/";

    public JarLauncher() {
    }

    protected JarLauncher(Archive archive) {
        super(archive);
    }

    protected boolean isNestedArchive(Entry entry) {
        return entry.isDirectory() ? entry.getName().equals("BOOT-INF/classes/") : entry.getName().startsWith("BOOT-INF/lib/");
    }

    public static void main(String[] args) throws Exception {
        (new JarLauncher()).launch(args);
    }
}
```

调用Java -jar命令会执行JarLauncher中的main方法，实际上调用的方法：

```java
protected void launch(String[] args) throws Exception {
    // Spring Boot内部引入了依赖的JAR文件，默认是无法扫描到的，SpringBoot通过注册属性java.protocol.handler.pkgs到classpath中去，即扩展了JAR协议的默认内容
    JarFile.registerUrlProtocolHandler();
    // Create a classloader for the specified archives 创建指定的归档（archives）类加载器
    ClassLoader classLoader = createClassLoader(getClassPathArchives());
    // 实际上继续向下委托给了MainMethodRunner#run方法
    launch(args, getMainClass(), classLoader);
}
```

MainMethodRunner#run：

```java
public void run() throws Exception {
    Class<?> mainClass = Thread.currentThread().getContextClassLoader()
        .loadClass(this.mainClassName);
    Method mainMethod = mainClass.getDeclaredMethod("main", String[].class);
    mainMethod.invoke(null, new Object[] { this.args });
}
```

> 这里的mainClassName指的就是MANIFEST.MF中的Start-Class属性，然后通过反射调用main方法



WAR实现方式与此类似



**打包WAR文件是一种兼容措施，既能被WarLauncher启动，又能兼容Servlet容器环境，换言之，WarLauncher和JarLauncher并无本质差别，所以建议SpringBoot应用使用非传统方式部署时，尽可能地使用JAR归档方式







## 第三章、理解固化的Maven依赖



SpringBoot为我们提供的固化依赖如：spring-boot-starter-parent和spring-boot-dependencies都提供了如版本信息管理等的固化依赖，如果项目有自己的parent就需要使用到后者了

这里有介绍后者的使用：https://docs.spring.io/spring-boot/docs/2.3.2.RELEASE/maven-plugin/reference/html/#using-import



spring-boot-dependencies是spring-boot-starter-parent的parent，其中包含着大量的dependencyManagement指定的版本号信息，甚至还包含maven plugin的管理信息。



就dependency两者好像并没有实质性的差别，但是在plugin上，spring-boot-starter-parent好像对plugin进行了定制，尽量还是使用parent吧







## 第四章、理解嵌入式Web容器



SpringBoot应用直接嵌入Tomcat、Jetty 和Undertow作为核心特性，在2.0版本还嵌入了Netty来做异步请求处理。



Java对HTTP请求的处理仅有两种选择：Servlet和其他。

前者几乎垄断了Java Web的开发，Tomcat和Jetty作为Servlet的经典实现，后来的Undertow作为JBoss社区推出的新一代Servlet3.1+规范的容器（其实就是Servlet3.1规范推出的时候的实现产品），成为嵌入式Servlet容器的新选择，嵌入式容器不是Spring自家推出的功能，而是Tomcat和Jetty在很久以前就已经支持嵌入式容器了。

![image-20200809154456222](images/image-20200809154456222.png)

至于Reactive Web容器，默认实现为Netty Web Server，与业内其他基于Netty实现的Web Server类似，如Eclipse vert.x。在Spring Boot Webflux中也是使用前者作为默认嵌入式容器实现，也可以通过spring-boot-starter-reactive-netty引入，也支持嵌入式容器。

热门的Web容器均支持嵌入式容器，并非Spring独创



2.2.6的web-starter的Web容器默认实现是9.0.33的Tomcat



接下来是对Tomcat、Jetty和Undertow作为嵌入式Servlet Web容器的讨论



Tomcat：

点进官网选择最老的版本7，可以看到

![image-20200809155335453](images/image-20200809155335453.png)



即使是7版本也对嵌入式容器有支持

实际上Tomcat自己发行了Maven插件，可以不用部署直接打包成可运行的JAR或WAR文件，估计SpringBoot有偷哦。

但不是将Tomcat作为嵌入式容器打包进项目当中，依然是以Tomcat容器为主体，这点和SPringBoot有所不同

不过Tomcat官方只是将该插件维护到了Tomcat7





Jetty：

切换到Jetty相当简单，在Web-starter exclusion掉tomcat-starter，再引入Jetty-starter即可

undertow也是类似





嵌入式Reactive Web容器



嵌入式Reactive Web容器作为SpringBoot2.0的新特性，处于被动激活的状态，只有检测到starter-webflux之后才会激活，并且当web和webflux同时存在的情况下后者会被忽略





可以使用SpringBoot2.0引入的一种ApplicationContext实现——WebServerApplicationContext来完成获取当前WebServer类型：

```java
@Autowired
WebServerApplicationContext context;

@Override
public void run(ApplicationArguments args) throws Exception {
    System.out.println("=======================================");
    System.out.println(context.getWebServer().getClass().getName());
    System.out.println("=======================================");
}
```

上面是通过让主类来实现ApplicationRunner接口来实现的，等效于下面的代码：

```java
@Bean
public ApplicationRunner runner(WebServerApplicationContext context) {
    return args -> {
        System.out.println("=======================================");
        System.out.println(context.getWebServer().getClass().getName());
        System.out.println("=======================================");
    }
}
```

注册进IoC中的ApplicationRunner会在应用启动后回掉 





另一种查看WebServer的方法：WebServerInitializedEvent（弥补非Web应用类型的场景，感觉 没咋读懂，既然引入了WebServer就必然是Web应用类型了呀，在非web情况下注入WebServerApplicationContext会失败）

```java
@EventListener(WebServerInitializedEvent.class)
public void run(WebServerInitializedEvent event){
    System.out.println(event.getWebServer().getClass().getName());
}
```



如今的Tomcat、jetty、Undertow都支持Reactive，可以直接在webflux的starter外直接加入对应的Servlet容器依赖，而不用再去排除掉默认的实现了，估计是SpringBoot2.0推出的webflux的starter支持



一旦引入了这些依赖到classpath路径，我们就可以利用SpringBoot的自动装配特性来完成后续的配置工作。







## 第五章、理解自动装配



SpringBoot自动装配的对象是Spring Bean，比如通过XML方式和Java配置类方式组装Bean



启用自动装配的方式：在@Configuration类上标注@EnableAutoConfiguration或者是@SpringBootApplication，至于如何装配@Configuration类这里并没有说明，依赖于Spring Framework的装配规则：**XML方式、@Import注解和@ComponentScan注解都可以完成自动装配**，前者需要ClassPathXmlApplicationContext加载，后者需要AnnotationConfigApplicationContext进行注册。

由此看来，SpringBoot程序的自动装配来源于@SpringBootApplication注解的支持



SpringBoot官方给出的

@SpringBootApplication = @Configuration + @EnableAutoConfiguration + @ComponentScan注解，且他们都是用默认配置



实际上并没有这么简单，SpringBoot在1.3的文档中即是这样描述的，SpringBoot2.0的文档并没有真实的反映SpringBootApplication的全部作用

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
    // ...
}
```

上述文档中描述的ComponentScan使用默认值，不准确，这里排除了FilterType的两个实现：

- TypeExcludeFilter：在SpringBoot1.4中引入，用于查找BeanFactory中已经注册的TypeExcludeFilter Bean作为代理执行对象
- AutoConfigurationExcludeFilter：在1.5的时候引入，用于排除其他同时标注@Configuration和@EnableAutoConfiguration的类

而在SpringBootApplication注解中使用了这两个Filter

不过还有一个不是很大的区别：SpringBootApplication标注的不是@Configuration而是@SpringBootConfiguration，不过在运行上的行为没有差异，这种类似于对象之间的继承关系好似Component和Configuration，Controller，Service之间的关系一样。



官方还提到了SpringBootApplication上面使用了EnableAutoConfiguration和ComponentScan的属性别名来实现定制化

这里跟着源码走就行了，实践通过

具体是通过Spring Framework4.2，SpringBoot1.2引入的注解@AliasFor



从这里可以看到SpringBootApplication是一个聚合注解，类似于@RestController





如果SpringBootApplication标注于非引导类之上，仍然可以通过SpringApplication的run方法来指定到被SpringBootApplication标注的类上，被run方法指定的类需要具有自动装配的特性，即至少需要@EnableAutoConfiguration注解标注才可执行。

对于Configuration则不强制要求，发现即使run指定的类没有指定@Configuration注解也可以将内部声明的bean加载到容器当中去，就相当于是SpringBoot自动将EnableAutoConfiguration类注册到容器当中去了吧。





@SpringBootApplication作为@Configuration的继承注解的特性：

 

在普通的Component中声明的@Bean对象被称为“轻量模式”的Bean，而在@Configuration 中声明的Bean是“完全模式”的Bean，后者会执行CGLIB提升的操作，前者只是简单的将对象注册到IoC容器当中来。

> 可以通过查看容器中Bean的ClassName来得出结论

这里的CGLIB的提升并非是@Bean对象提供的，而是为@Configuration类准备的

实例代码：

```java
//@Configuration
@EnableAutoConfiguration
public class FirstAppByGuiApplication {

	public static void main(String[] args) {
		SpringApplication.run(FirstAppByGuiApplication.class, args);
	}

	@Bean
	public String hello() {
		return "hello";
	}

	@Bean
	public ApplicationRunner runner1(ApplicationContext context) {
		return args -> {
			System.out.println(context.getBean(String.class).getClass().getName());
			System.out.println(context.getBean(FirstAppByGuiApplication.class).getClass().getName());
		};
	}
}
```

> EnableAutoConfiguration千万不能省略掉，要不然项目启动会出现问题

没有加@Configuration的输出：

```
java.lang.String
thinking.in.spring.boot.firstappbygui.FirstAppByGuiApplication
```

加了@Configuration的输出：

```
java.lang.String
thinking.in.spring.boot.firstappbygui.FirstAppByGuiApplication$$EnhancerBySpringCGLIB$$d72d5601
```

明显的看出代理，Configuration类是被CGLIB增强了的。





自动装配机制

SpringBoot的自动装配机制是依赖于Spring Framework的Bean生命周期管理和Spring编程模型，单Spring Framework自身是不支持@Configuration的自动装配了，SpringBoot1.0便添加了约定配置化导入@Configuration类的方式

实现自动导入配置类的条件判断是依赖于注解：@ConditionalOnClass和@ConditionalOnMissingBean（标注于@Configuration之上），可以指定在特定条件下（当依赖的类找到了，但没有声明配置类）的时候起效。进而实现配置类上@Import注解的导入到更多的依赖类和其他的自动配置



那么最初始的一批配置该如何导入呢？不可能一个个去Import吧，实际上在spring-boot-autoconfigure项目的spring.factories文件中都写着了初始化容器时候加载的Configuration

每个starter都会提供一个autoconfigure包来，可以查看这个包中的spring.factories文件来实现该starter自己的自动装配





实践：创建自己的starter：

实现环境搭建：

非常简单，让引导类处于三级包下，让另一个配置类也处于三级包下，由于默认只会加载引导类的目录下的所有类，因此配置类不会被加载到，我们通过自动配置类来实现自动加载配置类的功能：

![image-20200810174041103](images/image-20200810174041103.png)

在resources目录下创建META-INF/spring.factories文件，其中应该指定EnableAutoConfiguration注解作为配置类的key，因为自动配置一定会开启（引导类必须指定@EnableAutoConfiguration）也就相当于我们指定的配置类一定会被加载到，

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
thinking.in.spring.boot.autoconfigure.WebAutoConfiguration
```

/只是换行符，如果有多个类需要加载，中间用逗号隔开，如下

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
thinking.in.spring.boot.autoconfigure.WebAutoConfiguration,\
thinking.in.spring.boot.autoconfigure.WebAutoConfiguration
```

> 需要注意的一点：类命名均要以AutoConfiguration作为后缀

从而加载到WebAutoConfiguration配置类（只是写着是AutoConfiguration而已）：

```java
/**
 * Web 自动装配类
 */
@ConditionalOnWebApplication
@Configuration
@Import(WebConfiguration.class)
public class WebAutoConfiguration {
}
```

这里又是Web环境，因此会引入WebConfiguration类，可以在WebConfiguration中做一些输出来验证是否自动配置成功。

从而可以完全实现基于注解的Bean组装。



不可否认的是，随着 越来越多starter的引入，大量的Spring Boot装配变成黑盒，并且搭配条件注解之后显得更加复杂，为了支持以配置化的方式调整应用行为，如Web服务器端口等，Spring Boot提供了Production-Ready特性







## 第六章、理解Production-Ready特性



官方的解释为：Provide production-ready features such as metrics,health check and externalized configuration

production-ready 的一般性定义：

https://github.com/mitodl/handbook/blob/master/production-ready.md



主要是通过Spring-Boot-actuator来实现的

提供HTTP和JMX两种方式来监控和管理Spring



需要手动添加依赖到pom文件，通过暴露的endpoint来实现对程序情况的访问，默认情况下仅仅暴露health和info，要想增加其他，得使用management.endpoints.web.exposure.include=*配置到配置文件或者启动参数当中，这类配置被称为外部化配置



现在只剩下Production-Ready的最后 一个特性——externalized Configuration

官方给出的解释：

> Spring Boot allows you to externalize your configuration so you can work with the same application code in different environments,You cna use properties file,YAML files,environment variables and command-line arguments to externalize configuration



外部化配置的三个用途：

- Bean的@Value注入
- Spring Environment读取
- @ConfigurationProperties绑定到结构化对象



PropertySource之间存在顺序，顺序高的配置可以覆盖掉顺序低的配置（就相当于是配置覆盖了）

https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-external-config

感觉SpringBoot的核心文档在Spring Boot Features中



外部化配置的概念：与外部化配置相对应的是内部化配置，如各种set方法就是内部化配置的典型代表，配置数据是来源于应用内部实现的，缺少相应的弹性

SpringBoot采取的外部化配置类似于使用properties文件，命令行参数，系统参数来达到不干预内部代码的情况下实现定制化。



理解约定大于配置：

结合SpringBoot官方给出的最后一个特性：

> - Absolutely no code generation and no requirement for XML configuration

前半句话说明绝无中间代码生成，从而影响SpringBoot应用运行时行为，后半句则是耳熟能详的约定大于配置了。

从技术的角度看，大多数人以为注解驱动是SpringBoot提供的，其实是Spring Framework自身提供的，因此SpringBoot依赖于Spring Framework

Spring Framework的注解在2.5版本时候便开始推广，那时候推荐的是在XML中配置Componentscan，然后在Java代码中使用一系列DI注解实现，依然依赖于XML

到了Spring3.0提供了@Configuration来代替XML，Bean标签也可以被@Bean注解代替，框架还提供了@Import注解来导入@Configuration class并将其注册为Spring Bean，尽管在Bean的装配上有所提升，但仍然需要以硬编码的方式指定范围

Spring3.1发布了@ComponentScan注解来取代了XML的ComponentScan标签，实现了XML的完全可替代性，还提供了部分激活注解@EnableXXX来将指定类装配的 方法，但是被@EnableXXX标注的类需要自己被IoC容器感知到，这就成为了一个先有鸡还是先有蛋的问题了。

> 例如你声明了一个@ComponentScan注解到一个类上，但是在当时脱离了XML后IoC是加载不到该类的，自然无法扫描到@ComponentScan注解了，仍然无法实现自动配置

Spring4.0提出的Conditional注解使得自动配置成为可能，因为官方给出的描述——**“whenever possible”**。



SpringBoot官方给出的六大特征如第三点是maven提供的，第六点是Spring Framework3.1就已经完全支持的，综上 ，Spring Boot的主要五大特征为：

- SpringApplication
- 自动装配
- 外部化配置
- Spring Boot Actuator
- 嵌入式Web容器

这五大特征构成了SpringBoot作为微服务中间件的基础，又提供了SpringCloud的基础设施



其实微服务开发完全没有技术的限制，传统的Java EE容器也可以实现微服务

只不过由于Spring社区十几年的开源策略和技术演进使得Spring Boot在微服务的世界中独占鳌头





虽然Spring Boot提供了丰富的功能特性，但是其天生缺少快速构建分布式系统的能力，为此，Spring官方在SpringBoot的基础上开发出Spring Cloud，致力于为开发人员提供一些快速的通道构建通用的分布式系统，核心特性包含：

- Distributed/versioned Configuration（分布式配置）
- Service registration and discovery（服务注册与发现）
- Routing（路由）
- Service-to-service calls（服务调用）
- Load Balancing（负载均衡）
- Circuit Breaker（熔断机制 ）
- Distributed messaging（分布式消息）

Spring Cloud想提供的这些特性被大多数互联网公司所实现了，Spring官方的最大优势在于其强大的API设计能力（统一抽象，屏蔽技术实现细节），在 云平台Java语言（及其派生语言）处于垄断地位，Spring Cloud高度抽象的接口对于开发人员而言，底层实现是透明的，当需要更换底层时候只需要更换一个starter即可，就类似于Tomcat换Jetty，无需过多的业务回归测试。



Spring Cloud的第二大优势在于Spring Cloud Stream的整合，通过Stream编程模式达到数据传输的目的，感觉是有偷Java语言层面设计的。



SpringBoot的成功使得Spring社区焕发出了第二春，主要是因为SpringBoot的自动装配机制，但自动装配底层依赖的是Spring Framework的注解支持。







# 第二部分、走向自动装配



随着微服务的发展，开发人员开始更加重视SpringBoot了

殊不知Spring Framework是SpringBoot的核心，Java规范才是他们的基石

对Java EE来说，SpringBoot这种优秀的技术架构遵从着“兼容并包，继往开来”的原则，兼容旧的技术实现，发展新的技术理念







## 第七章、走向注解驱动编程（Annotation-Driven）



在Spring Framework第一个版本时候，Java Annotation尚未发布，结合J2EE（Java EE的前身，当时称之为J2EE）的传统，通过XML文件的方式管理Bean之间的依赖关系





注解驱动发展史



2003年发布了Spring Framework1.0版本

Spring1.2.0版本开启了Spring Framework对Annotation的支持

主要也是当时注解的流行导致Java在语言层面开启了对Annotation的支持，随后Spring也做出了相应的支持。

框架层面已经支持了@ManagedResource和@Transactional注解，但是被注解标注的SpringBean对象仍然需要以XML的方式进行装配，对于Spring Framework1.0来说，XML是唯一的选择



2006年Spring2.0正式发布

完全兼容1.0框架，且添加了如数据相关的Repository和AOP相关的Aspect注解，同时支持扩展XML编写，这为XML配置的价值提升了一个阶段

重要版本还是2.5，引入了骨架式的Annotation ：

- 依赖注入Annotation：@Autowired
- 依赖查找Annotation：@Qualifier
- 组件声明Annotation：@Component，@Service
- Spring  MVC Annotation：@Controller，@RequestMapping，@ModelAttribute

>  @Autowired支持注入SpringBean集合

不建议使用Qualifier，建议使用Resource注解



@Qualifier还可以用于“逻辑类型”限定，例如以下的两个注解都被@Qualifier标注

@LoadBalanced和@ConfigurationPropertiesBinding，可以先经过@Qualifier筛选

支持JSR-250的@Resource注入，当然还支持JSR250的生命周期回掉函数@PostConstruct和@PreDestroy，可以被XML替换掉

尽管2.5版本提供的注解不少，但是仍然摆脱不了XML，主要是因为仍然需要在XML中使用如下标签：<context:annotation-config >用于注册Annotation处理器，还需要使用< context:component-scan>用于寻求需要注册成Spring Bean的类

且在2.0的时候提供了@order注解代替Ordered接口来进行对多个Spring Bean进行排序

虽然Spring2.0时期被称为注解的过渡时代，但是在这个版本Spring MVC却完成了蜕变，官方推荐使用注解的方式代替XML的 方式完成编码





2009年Spring Framework3.0正式发布

被称为注解驱动的黄金时代，Spring Annotation雨后春笋般的出现，体现了Spring官方对替换XML配置的决心

> 就我感觉这是非常伟大的，因为Spring在XML配置的PropertyEditor上是花了大心思的，并且Spring为了提供与产品的整合刚开始也都是使用的XML配置方式，现在却自己要推倒自己最强势的地方了，有远见的公司



这个阶段引入了@Configuration注解，@Component的另一个“派生”注解

遗憾的是这时候并未直接替代XML元素ComponentScan而是采用了过度注解的方式——@ImportResource和@Import，前者允许导入遗留的XML配置文件，后者允许导入类作为Spring Bean，通常这些类无需标注Spring模式注解如Service等等。

通常@Import和@ImportResource需要和@Configuration注解一起使用

但是被标注的类有谁来引导呢？提出了定制的ApplicationContext——AnnotationConfigApplicationContext注册@Configuration class，然后通过这个@Configuration class上面标注的Inport注解来实现对依赖的导入。

用起来还是感觉非常的别扭



于是在Spring 3.1提出了@ComponentScan注解实现了对XML元素的替换，且这时候就出现了初步的条件注解@profile，如以下用法：

```java
@Profile("!production")  //非生产环境
@Configuration
public class Configuration {
    // ...
}
```

实现对非生产环境下Configuration的注册



在Web方面更是开启了全面的支持，请求处理注解@RequestHeader、@CookieValue和@RequestPart。但是更重要的是开启了REST开发，提供@PathVariable，@RequestBody反序列化请求体，@ResponseBody将处理方法返回对象序列化为REST主体内容，并且@ResponseStattus补充HTTP响应状态

主要还抽象了一套全新并统一的配置属性API，包括配置属性存储接口Environment，配置源抽象PropertySources，奠定了Spring外部化配置的基础，Spring为了简化获取外部化配置 ，提供了@PropertySource简化实现。

其次还支持了缓存抽象，异步支持，检验方面的支持。

即使是这样，Spring作者仍然继续添加“@Enable模块驱动”来实现模块化的Bean装配，例如@EnableWebMVC被标注在Spring Bean上后，RequestMappingHandlerMapping、RequestMappingHandlerAdaptor，HandlerExceptionResolver等Bean就被装配上了，当然这需要手动声明在配置类上，只能算作手动配置，也离自动配置更近了一步

仍然存在缺陷如：@Profile条件注解仍然功能单一太过简单等等



2013年的Spring4.0

注解驱动完善时代

不像Spring3.0版本中注解的大爆发侵入，有的只是完善的注解体系补充

提升装配能力的条件判断，引入了@Conditional注解，以至于曾经的@Profile注解都以Conditional注解的方式实现了一遍

从条件判断注解的完善标志着SpringBoot项目的基础正式打牢

Spring在4.2提出了EventListener作为ApplicationListener的备选方案

使用注解@AliasFor使得注解属性可以使用别名





2017年的Spring5.0

已经作为SpringBoot2.0的核心框架了，还没发行完，万一后面引入了什么新的特性了呢



Spring Framework个个版本引入的核心注解可以查看7.2节





深度展开：

- 元注解
- Spring模式注解
- Spring组合注解
- Spring注解属性别名和覆盖

元注解：能声明在其他注解上的注解，如@Documented注解可以成为任何注解的元注解，

在Spring世界中@Component就是标准的元注解



Spring注解模式就是特定场景下的注解，如Service之于 Component，因为Java语言层面是不允许注解之间的继承，因此需要通过给元注解特殊化的方式实现注解之间的派生



使用Spring Version模拟自定义@Component派生类：

![image-20200812155247930](images/image-20200812155247930.png)

剩下的就是简单的验证过程了，单有一个地方非常有趣：

在引导类 中出现了如下代码：

![image-20200812155327550](images/image-20200812155327550.png)

来实现Spring对Java8的兼容，测试用例：

![image-20200812155426580](images/image-20200812155426580.png)

主要为了证明派生注解也拥有元注解的作用。



派生性原理：初始化一个Bean定义解析器

componentScan标签的Bean定义解析器为：ComponentScanBeanDefinitionParser

于是开始解析出需要装配的BeanDefinitionHolder，方法摘要如下：

```java
public BeanDefinition parse(Element element, ParserContext parserContext) {
    String basePackage = element.getAttribute(BASE_PACKAGE_ATTRIBUTE);
    basePackage = parserContext.getReaderContext().getEnvironment().resolvePlaceholders(basePackage);
    String[] basePackages = StringUtils.tokenizeToStringArray(basePackage,
                                                              ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS);

    // Actually scan for bean definitions and register them.
    ClassPathBeanDefinitionScanner scanner = configureScanner(parserContext, element);
    Set<BeanDefinitionHolder> beanDefinitions = scanner.doScan(basePackages);
    registerComponents(parserContext.getReaderContext(), beanDefinitions, element);

    return null;
}
```

于是对BeanDefinition的加载还要归属到ClassPathBeanDefinitionScanner#doScan方法上去：

```java
protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
   Assert.notEmpty(basePackages, "At least one base package must be specified");
   Set<BeanDefinitionHolder> beanDefinitions = new LinkedHashSet<>();
   for (String basePackage : basePackages) {
      Set<BeanDefinition> candidates = findCandidateComponents(basePackage);
      for (BeanDefinition candidate : candidates) {
         ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(candidate);
         candidate.setScope(scopeMetadata.getScopeName());
         String beanName = this.beanNameGenerator.generateBeanName(candidate, this.registry);
         if (candidate instanceof AbstractBeanDefinition) {
            postProcessBeanDefinition((AbstractBeanDefinition) candidate, beanName);
         }
         if (candidate instanceof AnnotatedBeanDefinition) {
            AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition) candidate);
         }
         if (checkCandidate(beanName, candidate)) {
            BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
            definitionHolder =
                  AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
            beanDefinitions.add(definitionHolder);
            registerBeanDefinition(definitionHolder, this.registry);
         }
      }
   }
   return beanDefinitions;
}
```

还有一次调用：findCandidateComponents的调用栈

然后归结到方法：PathMatchingResourcePatternResolver#getResources方法得到资源集合，这个类在Spring揭秘中见过 ，是Spring提供的资源操作类，解析出Resource[]对象

```java
public Resource[] getResources(String locationPattern)
```

根据传入的basepackage，获取到所有的Resource然后进行candidate进行筛选

至于筛选的标准，则需要看最开始的ComponentScanBeanDefinitionParser#parse的第八行

关注ClassPathBeanDefinitionScanner对象，其中是含有includeFilters和excludeFilters的

查看他父类的方法ClassPathScanningCandidateComponentProvider：

```java
protected void registerDefaultFilters() {
    this.includeFilters.add(new AnnotationTypeFilter(Component.class));
    ClassLoader cl = ClassPathScanningCandidateComponentProvider.class.getClassLoader();
    try {
        this.includeFilters.add(new AnnotationTypeFilter(
            ((Class<? extends Annotation>) ClassUtils.forName("javax.annotation.ManagedBean", cl)), false));
        logger.trace("JSR-250 'javax.annotation.ManagedBean' found and supported for component scanning");
    }
    catch (ClassNotFoundException ex) {
        // JSR-250 1.1 API (as included in Java EE 6) not available - simply skip.
    }
    try {
        this.includeFilters.add(new AnnotationTypeFilter(
            ((Class<? extends Annotation>) ClassUtils.forName("javax.inject.Named", cl)), false));
        logger.trace("JSR-330 'javax.inject.Named' annotation found and supported for component scanning");
    }
    catch (ClassNotFoundException ex) {
        // JSR-330 API not available - simply skip.
    }
}
```

只需要关注第一行即可，后面是对JSR的支持，默认通过AnnotationTypeFilter指定添加包含@Component的AnnotationTypeFilter实例，而excludeFilters字段为空。

归根结底还是由AnnotationTypeFilter的识别功能来判断是否注册此BeanDefinition到容器中

ClassPathBeanDefinitionScanner允许我们自定义过滤规则，从而实现与Spring的解耦

~~只需要注入ClassPathBeanDefinitionScanner~~，他的父类中实现了addIncludeFilter、addExcludeFilter等方法可以直接使用，达到指定注解添加到IoC的目的

> Spring并没有将ClassPathBeanDefinitionScanner注入到IoC当中去

自己尝试了下，最新版本的Spring并未使用ClassPathBeanDefinitionScanner

可以参考SpringBootApplication中的ComponentScan注解使用，如下：

```java
@EnableAutoConfiguration
@SpringBootConfiguration
@ComponentScan(excludeFilters = { @ComponentScan.Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
        @ComponentScan.Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) },
includeFilters = {@ComponentScan.Filter(type = FilterType.ANNOTATION,classes = Candidate.class)})
public class ThinkingInSpringBootSamplesApplication {

    public static void main(String[] args) {
        SpringApplication.run(ThinkingInSpringBootSamplesApplication.class, args);
    }

    @Bean
    public ApplicationRunner runner(TestBean testBean) {
        return args -> {
            testBean.sayHello();
        };
    }
}
```

因为SpringBootApplication没有提供includeFilter这一属性，我们需要手动代替掉SpringBootApplication注解，然后通过给@ComponentScan加上一个includeFilter属性即可



IDEA也会给出提示的，只要你注入到容器当中就不会报错了。



Spring对多重派生的支持：从3开始支持两层结构的派生，到4支持任意深度的派生，因为SpringBoot1.0是基于Spring4.0的，因此SpringBoot完美继承Spring的多重派生