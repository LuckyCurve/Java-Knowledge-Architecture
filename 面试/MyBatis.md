# MyBatis面试常见问题



突然感觉MyBatis可问的东西不多，原理是非常简单，但是实现的细节非常多，并且许多语法特性也很难记，往往是写的时候去查



占位符的区别：`${}`和`#{}`

感觉这个并不该属于MyBatis了，`${}`是Spring提供的SpEL表达式，用于将PropertySource注入到占位符当中，而`#{}`属于OGNL表达式，是MyBatis使用的参数占位符，在被编译成SQL语句的时候会被替换为问号，然后在执行过程中实现数值的注入



MyBatis的XML常见标签（除了增删改查的标签）：

mapper：定义与哪个接口之间发生映射，执行逻辑，MyBatis配置文件最外层标签

resultMap：做结果映射

where包裹住if：做条件判断



DAO接口如何映射到XML上的，DAO中的方法可以重载吗

靠XML中的最外层mapper标签，指定namespace，实现了DAO接口与XML的映射，

至于其中方法的绑定，则是通过SELECT/DELETE标签的id属性指定方法名字，resultMap指定返回结果，parameterType指定参数实现绑定，在运行时候每个SETECT等会被转换成为一个MappedStatement

DAO中的方法是不能重载的，因为MyBatis的匹配模式仅限于权限定类名+方法名，与参数无关，尽管XML中有参数，可以作为判断的依据。

在运行时候走的是JDK动态代理



MyBatis进行分页

主要是通过分页插件进行完成的，分页插件会对SELECT语句进行增强，加上LIMIT语句。



MyBatis的插件运行原理

主要是针对SqlSession下四大对象：ParameterHandler（处理SQL语句参数）、StatementHandler（使用PreparementStatement执行操作，核心，很多扩展都是基于此的）、Executor（实现整体的调度）、ResultSetHandler（返回值的封装）进行拦截和处理，使用InvocationHandler来对这几个类进行增强



MyBatis的动态SQL

主要还是根据OGNL的几个判断表达式，如if来判断条件是否成立，来实现SQL是否连接，从而实现动态SQL。



MyBatis的映射关系

第一种情况：默认将类名映射成对象的属性名，实现对象的属性注入，MyBatis区分大小写，为了提高匹配的匹配度

第二种情况：使用resultMap标签，指定列名与属性的映射关系



MyBatis是支持延时加载的

支持延迟加载，就是只有当访问被加载对象时候才会执行SQL语句，主要是通过CGLIB增强get方法，只要返回值为null，就去执行对应的SQL语句



MyBatis中是通过mapper标签的namespace属性作为全限定类名，通过SELECT/DELETE，id作为方法名来进行匹配的



MyBatis内部存在的几种Executor（主要组件还有ParameterHandler、StatementHandler、Executor、ResultSetHandler）：

SimpleExecutor（每次执行都创建一个Statement）、ReuseExecutor（相当于是一个Statement的池子，以SQL语句为key，实现Statement的重复利用）、BatchExecutor（批量执行SQL语句，一批SQL语句共用一个Statement）

如何使用：在DefaultSqlSessionFactory中调用带参数的ExecutorType方法创建SqlSession



MyBatis会将全部的配置（包含读取到的XML文件）都一起放入一个重量级对象Configuration中



MyBatis的执行过程（也就是不用starter时候我们使用MyBatis）：

1、通过流的方式读取配置文件，通过配置文件让SqlSessionFactoryBuilder生成SqlSessionFactory

2、通过SqlSessionFactory获取SqlSession

3、通过SqlSession执行我们编写的SQL语句，核心是其中的Executor，Executor的执行链路为ParameterHandler、StatementHandler然后执行SQL语句，通过ResultSetHandler封装结果集



MyBatis涉及到的设计模式：

构造者模式：SqlSessionFactoryBuilder

工厂模式：SqlSessionFactory

代理模式：Mapper

模板方法模式：Executor

