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





