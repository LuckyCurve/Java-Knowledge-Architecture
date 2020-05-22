# 第一部分、Spring基础





## 第一章、Spring起步



Spring的发展历程：

最常见的应用形式是基于浏览器的Web应用，后端由关系型数据库作为支撑。尽管这种形式的开发依然有它的价值，Spring也为这种应用提供了良好的支持，但是我们现在感兴趣的还包括如何开发面向云的由微服务组成的应用，这些应用会将数据保存到各种类型的数据库中。另外一个崭新的关注点是反应式编程，它致力于通过非阻塞操作提供更好的扩展性并提升性能。



Spring的核心是一个容器，也被称为应用上下文（Application Context），它们会创建和管理应用组件。这些组件也可以称为bean，会在Spring应用上下文中装配在一起，从而形成一个完整的应用程序。



配置类中：默认情况下，这些bean所对应的bean ID与定义它们的方法名称是相同的



创建第一个项目：

启动类：

```
@SpringBootApplication包含：
    @Target({ElementType.TYPE})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Inherited
    @SpringBootConfiguration
    @EnableAutoConfiguration
    @ComponentScan(
        excludeFilters = {@Filter(
        type = FilterType.CUSTOM,
        classes = {TypeExcludeFilter.class}
    ), @Filter(
        type = FilterType.CUSTOM,
        classes = {AutoConfigurationExcludeFilter.class}
    )}
    )
```

> 实际上有用的：
>
> - SpringBootConfiguration：@Configuration注解的特殊形式
> - EnableAutoConfiguration：启动SpringBoot的自动装配
> - ComponentScan：启用组件扫描。扫描@Component、@Service这种注解声明的其他类，并将他们注册到Spring应用上下文中



```
SpringApplication.run(TacoCloudApplication.class, args);
```

创建Spring应用上下文，一个是配置类，一个是启动参数（尽管传递的配置类不一定要和引导类相同，但这是最便利和最经典的做法，尽管也可以在引导类中配置一两个组件，但是还是建议单独创建一个配置类）



测试类：

主要就是@SpringBootTest注解了，提供SpringBoot的测试环境，相当于调用了引导类的SpringApplication.run（）方法

也可以在测试类上面加上@Runwith注解（Junit提供的）以前的测试类头上会带上这个注解@Runwith(SpringRunner.class)，现在好像都没有这个注解了（SpringBoot 2.3.0）

可以跑空的测试类来判断是否能启动应用上下文



编写一个控制类：

```java
@Controller
public class HomeController {
    @GetMapping("/")
    public String home() {
        return "home";
    }
}
```

@Controller注解：并没有做太多事情，仅仅只是让当前类被扫描，装载到容器中

@GetMapping注解：针对"/"发送的HTTP GET请求，那么这个方法将会处理请求

返回值会交给thymeleaf去进行视图解析，加上前缀“/templates/”和后缀".html"。拼接成"/templates/home.html"



编写一个视图页面：

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="https://www.thymeleaf.org/">
<head>
    <meta charset="UTF-8">
    <title>Taco Cloud</title>
</head>
<body>
    <h1>Welcome to</h1>
    <img th:src="@{/images/Logo.png}">
</body>
</html>
```

要手动引入th标签，要不然报错

唯一需要注意的是`th:src="@{/images/Logo.png}"`，这里的意思是读取"/static/images/Logo.png"这张图片





测试器：对HTML页面的内容断言是很困难的，SpringMVC提供了强大的测试。感觉蛮不好用的，浏览器测试更好





Spring Boot DevTools

作用：

- 代码变更后自动重启（依赖更新后无效，需要重启，配置文件修改了之后有效）
- 面向浏览器的资源（HTML，JS）发生变化时候自动刷新浏览器（需要安装LiveReload插件）
- 自动禁用模板缓存
- 如果使用H2数据库，内置了H2的控制台

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

默认指定打包之后就没用了（只在生产环境有效）

实现了两个类加载器，一个加载Java代码，属性和项目中基本所有的内容，另一个去加载依赖的库，每次发生改变的时候，只需要重启一个类加载器即可，另一个相对稳定的类加载器（加载依赖的那一个）就不会动了。

如果使用了H2数据库，会生成一个URL为http://localhost:8080/h2-console的控制台





## 第二章、开发Web应用



>  主要是MVC层的数据绑定
>
> 章节概括：
>
> - 在浏览器中展现模型数据
> - 处理和校验表单输入
> - 选择视图模板库



REST API：是指控制器直接返回Model对象，不进行页面渲染。



Demo：

实体类

```java
@RequiredArgsConstructor
@Data
public class Ingredient {
    private final Integer id;
    private final String name;
    private final Type type;
    public static enum Type{
        /**
         * 字段注释
         */
        WRAP,PROTEIN,VEGGIES,CHEESE,SAUCE
    }

}
```



Controller层：

```java
@Slf4j
@Controller
@RequestMapping("/design")
public class DesignTacoController {
    /**
     * 默认就是@GetMapping("")
     */
    @GetMapping
    public String showDesignForm(Model model) {
        List<Ingredient> list = Arrays.asList(
                new Ingredient(1, "Lettuce", Ingredient.Type.PROTEIN),
                new Ingredient(2, "Lettuces", Ingredient.Type.SAUCE),
                new Ingredient(3, "Lettuces", Ingredient.Type.CHEESE),
                new Ingredient(4, "Lettuces", Ingredient.Type.WRAP)
        );
        Ingredient.Type[] types = Ingredient.Type.values();
        for (Ingredient.Type type : types) {
            model.addAttribute(type.toString().toLowerCase(),filterByType(list,type));
        }
        return "design";

    }

    private List<Ingredient> filterByType(List<Ingredient> ingredients, Ingredient.Type type) {
        return ingredients.stream().filter(x->x.getType().equals(type)).collect(Collectors.toList());
    }

}
```

> 作者的编码太厉害了



Thymeleaf这种视图解析器很明显是和特定的Web框架解耦的，那么是如何获取数据的呢？显然无法直接访问到Model对象，是在到达控制器的时候就将数据从Model复制到了request里面，就可以访问到了。



在HTML中使用th：each标签去遍历结果集即可。一般都是`${}`访问request，如果取到的是一个对象，则直接使用对象的toString方法变成字符串展示在页面上。



数据校验

Spring支持Java的Bean校验API（Bean Validation API，JSR-303）

具体支持注解看[常见错误&基础结论](../常见错误&基础结论.md)





如果要写视图控制器，去继承WebMvcConfigurer，重写addViewControllers方法。也就是取代那些需要直接返回view的controller，感觉没啥用。



本章学习了强大的Web框架——Spring MVC

基于注解的，支持校验，支持视图解析







## 第三章、使用数据

使用Spring对JDBC的支持（Java DataBase Connectivity）消除样式代码（什么Class.forName（）这种代码），最后使用JPA（Java Presistence API）重写数据，消除更多冗杂代码



JDBCTemplate仅仅只是省去了大量的模板创建，依然需要自定义SQL，但是可以帮你把返回值按照字段名称自动注入到对象中去，占位符依旧使用？，跟原来的JDBC一模一样。



在插入的时候可以使用Spring提供的封装性较高的SimpleJdbcInsert



Spring Data有很多子项目支持不同的数据库：

- Spring Data JPA：基于关系型数据库进行JPA持久化（直接屏蔽掉了底层关系型数据库的类型，让开发人员不用和数据库打交道了）
- Spring Data MongoDB
- Spring Data Neo4j
- Spring Data Redis...



JPA底层的默认实现就是Hibernate，可以改变（在jpa里面排除hibernate）引入你需要的即可。



创建的Repository中的方法会根据方法名来自动实现（如果条件复杂可以使用@Query注解，然后自定义方法名）



Demo：



application.yml

```java
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test?serverTimezone=UTC
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: 123456
  jpa:
    hibernate:
      ddl-auto: update
    open-in-view: false
```

这里需要设置ddl，默认不会创建表的，会报错



实体类：

```java
@Entity
@Data
public class TechStack {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Integer id;

    private String techName;
}
```



repo：

```java
public interface TechStackRepo extends CrudRepository<TechStack,Integer> {

    List<TechStack> findByTechNameLike(String name);
}
```



这里的方法都会根据方法名自动实现





## 第四章、保护Spring



启动Spring Security来保护Spring

引入这个配置之后，会自动进行安全的基本配置

访问网站需要输入用户名和密码：用户名默认为user，密码为控制台输出的一个字符串，也是通过浏览器的JSESSIONID来匹配到服务器中的Session域的，如果清除了Cookie就需要重新登录。

默认安全信息：

- 所有HTTP请求都需要认证
- 不需要特定的角色和权限
- 没有登录页面（默认页面是HTTP basic认证的框）
- 默认只有一个用户user



增加用户：

1.用户数据量不大，修改次数少。直接存内存：

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        //需要加密方式
        auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder())
                .withUser("buzz").password(new BCryptPasswordEncoder().encode("222")).roles("USER");
    }

}
```

要指定密码加密方式



具体的加密方式：

•BCryptPasswordEncoder：使用bcrypt强哈希加密。

•NoOpPasswordEncoder：不进行任何转码。

•Pbkdf2PasswordEncoder：使用PBKDF2加密。

•SCryptPasswordEncoder：使用scrypt哈希加密。

•StandardPasswordEncoder：使用SHA-256哈希加密。



2.存储到数据库中，是使用JDBC来存储的，可以百度下



都是指定好的，用起来极其不方便





自定义用户认证：

依靠Spring Data JPA

对比了下Mybatis+Shiro，突然知道为什么大多数国内的厂商都倾向于Shiro了，Spring Security提供了太多的功能了（然而这些功能大多数情况下都用不上）。







## 第五章、使用配置属性





细粒度的自动配置

配置类中的@Bean方法，将返回值注入，很容易测试：

```java
@Bean
public String hello(){
    String a = "hello world";
    System.out.println(a);
    return a;
}
```

在配置类中加入这一行即可，会明显的看到hello world的输出

![image-20200521103711284](images/image-20200521103711284.png)

Spring完成的自动属性注入

例如Server.port=8080可以在任何一个阶段设置（JVM参数，命令行参数，操作系统环境变量（命名格式需要改变，Spring会自动适配））



接下来讲述了一些常规的可配置项，

数据库配置：例如spring.datasource

嵌入式服务器配置：例如：Server.port=0（随便选择一个可用端口）在不关心应用在哪个端口启动的情况非常重要，如成为一个服务注册到注册中心去

指定配置文件（在主配置文件里面指定）:

```yaml
spring:
  profiles:
    active: dev
```

配置文件命名：application-{profiles}.yaml

这是设置的最糟糕的一种方式，会直接使用指定的配置文件（无论是在开发环境还是在生产环境都是一样）。无法享受到生产环境和开发环境不同所带来的便利

可以在IDE中手动指定：

![image-20200521152955643](images/image-20200521152955643.png)

或者使用命令行的方式：

![image-20200521153010111](images/image-20200521153010111.png)





日志的级别设置：

```log
logging:
  level:
    root: debug
  file:
#    好像路径没用，都直接输出到项目的根目录下面了
    path: D://
    name: Test.log
```

必须使用到root上面去，要不然报错



配置类的写法：

```java
@Configuration
@ConfigurationProperties("greeting")
@Setter
public class HelloConfig {
    private String hello;

    @Bean
    public String bean() {
        System.out.println(hello);
        return hello;
    }
}

```

一定要setter注解，要不然属性注入不进去



只要是被扫描到的，无论是使用Copeonent系列的还是Configuration，里面被标注了bean方法都会被加载，返回值进入到IOC容器中。因为COnfiguration也就相当于是Component的变种（标注在配置类上面）





可以根据当前profiles的不同来选择配置（如果指定的是dev，但dev配置没有这个属性，就会再回去读取默认的配置）和指定是否创建bean

```java
@Configuration
@ConfigurationProperties("greeting")
@Data
@Profile("!dev")
public class HelloConfig {
    private String hello;
    
    @Bean
    @Profile("!dev")
    public String hello() {
        System.out.println(hello);
        return hello;
    }
}
```

如果是以下配置`@Profile({"!dev","!dev2"})`，是表示只要满足其中一个条件即可







# 第二部分、Spring集成



Spring应用与其他应用集成的话题





## 第六章、创建REST服务



REST API就是如今常说的前后端分离的后端API，只进行数据传输，而不去管页面的事情。

![image-20200522113509576](images/image-20200522113509576.png)

PUT和Patch的区别（语义层面）：

- PUT应该是对资源的整体覆盖，哪怕是有空字段，也是采取直接覆盖的态度
- Patch是完成资源的局部更新，更新非null字段





@RestController：

- 能够使得组件被扫描发现
- 控制器中处理方法的返回值要被直接写入响应体（Response Body）中，而不是将值放入模型（Request）中传递给视图进行渲染



@RequestMapping有个produces属性，指定产生的消息头，可以使用consumes指定接收的消息头



待验证：Spring提供`@CrossOrigin`注解来突破域限制，加在需要被消费的类上即可



直接弹出这个错误消息的方法：

![image-20200522151626134](images/image-20200522151626134.png)

```java
@ResponseBody
@GetMapping("/{id}")
public ResponseEntity<String> findById(@PathVariable("id") Integer id) {
    if (id == 2) {
        //是直接跳ERROR界面了，不是返回的数据
        return new ResponseEntity<>(HttpStatus.NOT_FOUND);
    }
    return new ResponseEntity<>("hello",HttpStatus.ACCEPTED);

}
```

使用ResponseEntity这个实体类即可。

不过不推荐在REST客户端上这样做，建议统一消息回复即可。





指定消息返回的Code（默认是200 OK）：

使用@RequestStatus注解

```java
@GetMapping("/")
@ResponseStatus(HttpStatus.ACCEPTED)
public String hello() {
    return "hello world";
}
```





映射到业务层的String对象一般不会为null（无论是从数据库来的还是从前端来的），一般都是empty。



后面就是：在请求体中动态的插入超链接，如查询所有商品的时候顺便在每个商品后面带上修改商品的URL。Spring提供了这个功能，感觉对于现在实际意义不大。



小结：

主要就是RestController，ResponseBody，RequestBody的使用。









## 第七章、消费REST服务



前面学会了创建REST API提供给客户端去调用，但Spring对另外一个API发起请求并不罕见，特别是在微服务领域的热度的增加。

主要方式：

- RestTemplate：Spring提供的简单的、同步的REST客户端
- Traversion：对上一章结尾讲述的动态生成的超链接的调用，依旧感觉意义不大
- WebClient：Spring 5提供的反应式、异步REST客户端（先推迟下，因为反应式的Web框架都没讲）





主要就关注RestTemplate即可

RestTemplate可以直接new就行了，但麻烦且资源利用率低，建议写个bean方法直接创建一个RestTemplate注入到容器中

发送一个HTTP请求会有太多的样板代码，于是Spring简化了这些代码，就像JDBCTemplate一样

![image-20200522162151429](images/image-20200522162151429.png)

表7.1中的大多数操作都以3种方法的形式进行了重载。

•使用String作为URL格式，并使用可变参数列表指明URL参数。

•使用String作为URL格式，并使用Map<String,String>指明URL参数。

•使用java.net.URI作为URL格式，不支持参数化URL。



1.Get请求：

主要有两个方法可以调用：

- getForObject
- getForEntity

两者区别：getForEntity会包含响应头，响应体的全部信息，即整个RequestEntity对象

而getForObject只会包含响应体里面的信息，在你比较关注响应体的信息的时候建议使用这个



测试如下：

> 测试环境是因为：在启动项目时候指定使用dev环境，在运行时候无法指定参数，默认使用8080端口，不会产生端口冲突，完美解决。

被调用代码：

```java
@RestController
@RequestMapping("/test")
public class TestController {
    @GetMapping("/get")
    @ResponseStatus(HttpStatus.ACCEPTED)
    public String hello() {
        return "hello world";
    }
}
```



调用代码：

```java
@Test
void contextLoads() {
    String s = restTemplate.getForObject("http://localhost:8090/test/get", String.class);
    System.out.println(s);
}
```

调用成功





2.post请求：

有三个：

postForObejct

postForEntity

postForLocation

关系：postForEntity =|= postForObejct（响应数据）+postForLocation（响应地址）+xx





被调用代码：

```java
@RestController
@RequestMapping("/test")
public class TestController {    
	@PostMapping("/post")
    public String hello2(@RequestBody Data hello) {
        System.out.println(hello);
        return hello.getName();
    }
```



调用代码：

```java
@Test
void test() {
    TestController.Data data = new TestController.Data();
    data.setId(1);
    data.setName("hello fuben");
    String s = restTemplate.postForObject("http://localhost:8090/test/post", data, String.class);
    System.out.println(s);
}
```

> 一定要将POST的参数加上RequestBody注解才能有效，如果不加的话就是null了，好像使用了POST就必然会使用RequestBody域了。





其他就非常类似了。





小结：

客户端使用RestTemplate针对REST API发送HTTP请求







## 第八章、发送异步消息



前面提到的REST等类似的同步通信并不是唯一的方式。异步消息就是另一种方式。

主要的三种方式：

- JMS（Java Message Service）
- RabbitMQ
- kafka

就相当于是增加了中间件嘛



JMS：

JMS是一个Java的标准，消息代理的通用API，类似于JDBC为数据库提供了通用的接口一样

Spring再次帮我们做了封装——JMSTemplate

默认实现：Apache ActiveMQ，但由于ActiveMQ快被淘汰了，Apache推出了ActiveMQ Artemis方案来作为下一代的ActiveMQ。（好像社区是真的不活跃，Docker Hub都没有对应的镜像，而RabbitMQ的镜像更新时间为37min前）

就了解下，通读下书

主要是对JMSTemplate的使用

在听雷丰阳老师讲SpringBoot的时候就说到了JMS是要不如AMQP的，因为平台相关性太强





使用RabbitMQ和AMQP

RabbitMQ可以说是AMQP最杰出的实现了



RabbitMQ的原理：

直接看我的这篇博客就好了：

https://blog.csdn.net/LuckyCurve/article/details/104534265

实践：引入amqp依赖



一个最简洁的Demo：

可以登录15672管理界面来查看Exchange，Binding，Queue是否创建成功以及消息发送情况

```java
@SpringBootTest
public class RabbitMQTest {
    @Autowired
    AmqpAdmin amqpAdmin;

    @Autowired
    RabbitTemplate rabbitTemplate;

    @Test
    void test() {
        //创建Exchange
        DirectExchange exchange = new DirectExchange("admin.exchange");
        amqpAdmin.declareExchange(exchange);
        //创建Queue
        Queue queue = new Queue("admin.queue");
        amqpAdmin.declareQueue(queue);
        //创建Binding
        Binding binding = new Binding("admin.queue", Binding.DestinationType.QUEUE,"admin.exchange","amqp.haha",null);
        amqpAdmin.declareBinding(binding);
    }

    @Test
    void test2() {
        String info = "hello";
        rabbitTemplate.convertAndSend("admin.exchange","amqp.haha",info);
    }

    @Test
    void test3() {
        String info = (String) rabbitTemplate.receiveAndConvert("admin.queue");
        System.out.println(info);
    }

}
```

上面是手动接收，下面是监听功能，要使用`@EnableRabbit`注解：

```java
@Component
public class MessageHandler {
    @RabbitListener(queues = "admin.queue")
    public void handler(Message message) {
        byte[] body = message.getBody();
        System.out.println(new String(body));
    }
}
```





kafka：

仅仅只支持主题形式的发送订阅模型，看似比RabbitMQ的功能要少，但是由于其天生涉及出来就支持集群部署，每个节点都负责一个主题（存在多个节点负责一个主题的情况）。每个节点可划分为多个分区。【即单个节点被划分成多个分区，这些分区又被分配到各个代理去处理各个主题的事务】

![image-20200522202911307](images/image-20200522202911307.png)

由于完整的kafka集群需要搭建Zookeeper，且KafkaTemplate的操作与大多数Template大同小异，就不做过多的赘述了。





小结：

异步消息需要中间件的支持，能解耦和增大应用的扩展性

