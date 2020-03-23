> mysql必知必会读书笔记



目录完全按照mysql必知必会来



# 了解SQL



## 数据库基础



在深入了解MySQL之前有必要了解数据库的基础知识



### 什么是数据库

数据库是一种以某种方式有组织的方式存储的数据集合

理解数据库：将数据库想象成一个文件柜，此文件柜是一个存储文件的物理位置，不用管数据是如何存放的以及以什么方式存放的



:key:**数据库**（database）：保存有组织的数据的容器（通常是一个或者一组文件）

> :warning:：人们通常用数据库这个术语来表示他们使用的数据库软件，这是引起混淆的根源，数据库软件应该被称为DBMS（数据库管理系统），数据库是通过DBMS来操纵的容器，数据库是什么并不重要，因为你使用的是DBMS，他替你访问数据库。





### 表

表是一种结构化的文件，可以用来存储某种特定类型的数据



:key:**表**（table）：某种特定类型数据的结构化清单



存储于表中的数据应该是一种类型的数据或者是一个清单，绝不是根据业务逻辑将多个清单混合存入一个表中，这样会使得以后的检索和访问都很困难

数据库中的每个表都有一个唯一的名字

> :warning:：表名的唯一性取决于多个因素，例如数据库名，在不同的数据库中可以存在同名的表



表具有某些特性，描述表的这些信息就是所谓的模式，模式可以用来描述数据库中特定的表以及整个数据库和其中表的关系



:key:：**模式**（schema）：关于数据库和表的布局及特性的信息

> 有时候，模式用作数据库的近义词，模式的含义在上下文中不是很清晰





### 列和数据类型

表由列构成，列中存储表的各方面信息



:key:：**列**（column）：表中的一个字段，所有的表都是由一个列或多个列组成



正确得将数据分解成多个列及其重要，如果处理不好可能造成排序和过滤非常苦难



每个列都具有相应的数据类型



:key:**数据类型**（datatype）：所容许的数据类型，每个列都具有相应的数据类型，它限制（容许）该列中存储的数据



### 行



:key:**行**（row）：表中的一个记录

> 经常提到行（row）时将其称为数据库记录（record），很大程度上这两个术语是可以相互替代的，但从技术层面上来说行更规范





### 主键



表中的每一行都应该有可以唯一标识自己的一列（或一组列）



:key:**key**（primary key）：一列**（或一组列）**，其值能唯一区分表中的每一行

主键可以中来标识特定的一行，没有主键，更新和删除表中的某一特定行就很困难，因为有可能直接删除了多行

> 设计人员都应该保证他们创建的每个表都具有一个主键，以便于以后的数据操作和管理



只要满足一下条件，该列即可作为主键：

- 任意两行都不具有相同的主键值
- 每一行都必须要具有一个主键值（即主键值不能为NULL）

> 这里的规则是MYSQL本身强制施行的



主键通常作用在表的一个列上，但也可以使用多个列作为主键，使用多列作为主键时，上述条件作用于主键的所有列，但第二点有所改变，只需要列值的组合唯一即可（不需要保证单个列的值唯一）

>除以上MySQL强制要求的几点之外，应该还有以下好习惯：
>
>- 不更新主键列中的值
>- 不重用主键列的值
>- 不将可能更改的列定义为主键：例如如果使用供应商的名字作为主键，那么当供应商合并或者更改名字时，主键也要随之改变，不合理



还有一个重要的键：外键





## 什么是SQL

SQL是结构化查询语言（Structured Query Language）的缩写，SQL是一门专门永安里和数据库通信的语言，与其他的编程语言不同，SQL只有很少的关键字，这是为了提供一种从数据库中读写数据的简单有效的方法。

SQL优点：

- SQL不是某个特定数据库提供商专有的语言，几乎所有的DBMS都支持SQL
- SQL简单易学，都是用少量简单的英文单词描述
- SQL灵活性高，可以进行复杂和高级的数据库操作



> 事实上任意两个DBMS实现的SQL都不完全相同，虽然他们绝大多数语法相同，所以SQL语句不是可以完全移植的









# MySQL简介



## 什么是MySQL



数据的所有存储，检索，管理和处理实际上是由数据库软件——DBMS来完成的，MySQL即是一种DBMS，他是一种数据库管理软件



MySQL的优点：

- 成本低：开源的，一般可以免费使用和免费修改
- 性能：MySQL执行非常快
- 可信赖：很多公司和站点都使用的是MySQL
- 简单：易于安装和使用



唯一缺点：不总是支持其他DBMS提供的功能和特性，这一步也在随着版本提升而改善





### 客户机-服务器软件

DBMS分为两类：

- 基于文件共享系统的DBMS
- 基于客户机-服务器的DBMS

前者包括微软的Access和FileMaker，一般用于桌面用途，不具有高可靠性

后者包括MySQL，Oracle以及SQL Server，将其分为两个部分，服务器部分是负责数据访问和处理的一个软件，运行在被称为数据库服务器的计算机上

客户机通过向服务器发送网络请求给服务器，进行数据处理并且返回数据（如果有返回值的话）

大多数网络连接都不具备对数据的访问权，甚至不具备对存储数据的驱动器的访问权，只是通过MySQL进行了交互

- 服务器软件为MySQL DBMS
- 客户机可以是MySQL提供的工具，脚本语言（Perl），web应用开发语言（JSP，PHP），程序设计语言（C，C++，Java）等



### MySQL版本



主要是DBMS版本引入的更改：

- 4：InnoDB引擎，事务处理，并，改进全文本搜索
- 4.1：函数库，子查询，集成帮助
- 5：存储过程，触发器，游标，视图





## MySQL工具



客户端工具推荐



### mysql命令行实用程序



使用注意点：

- 命令输入在>mysql之后（你也无法输入在前面）
- 命令用；或者\g结尾，不是Enter
- 输入help获取帮助（help select 获取 SELECT语句的帮助）
- 使用quit或者exit来退出命令行实用程序



> ​	即使你选择了使用后面所述的某个图形工具，也必须保证熟悉mysql命令行实用程序



### MySQL Administrator



MySQL官方提供的图形交互客户机，需要单独安装





### MySQL Query Broswer



MySQL官方提供的图形交互客户机，用来编写和执行MySQL命令







# 使用MySQL



## 连接



如果你使用的是本地服务器，并且试用MySQL，使用root登陆就可以了，但是在实际开发中需要用户列表，权限关联等操作来提升数据库的安全性



可以使用流行的navicat客户端来进行用户图形化操作

连接到MySQL需要以下信息：

- 主机名：本地就是localhost
- 端口，一般是3306
- 合法用户名
- 用户口令（密码）【如果需要】



以下均使用命令行工具来操作



## 连接



最初连接到MySQL，没有任何数据库打开让你使用，需要使用USE关键字来选择数据库

```mysql
USE （数据库表名）;
```

:key: 关键字（key word）：MySQL语言的保留字，不要使用mysql关键字去命名表和列





### 了解数据库和表



展示数据库和表

```mysql
SHOW DATABASES;
SHOW TABLES;
```



SHOW同样也可以用来展示列：

```mysql
SHOW COLUMNS FROM USER;
```

> SQL语句一般关键词用大写，自定义的表名或者数据库名用小写，尽管SQL是不区分大小写的，查询user表用以上语句不会报错

可以显示字段名，数据类型，是否允许为NULL，键信息，是否为auto_increment等



> 自动增量：
>
> ​	某些表需要使用唯一值来作为主键，可以代替手动分配唯一值，可以在create表格的时候b把它当作表定义的组成部分



> 上述语句可以使用DESCRIBE语句来代替，即DESCRIBE user是SHOW COLUMNS FROM user的简写



所支持的其他SHOW语句还有

- SHOW STATUS：显示服务器状态
- SHOW GRANTS：显示用户和其具有的权限
- SHOW ERRORS 和 SHOW WARNINGS：显示服务器错误或警告消息

更多SHOW语句用法可以执行HELP SHOW语句来查看



# 检索数据

如何使用SELECT 语句检索出一个或多个数据列



## SELECT语句



SELECT是非常常用的关键字

使用SELECT关键字必须给出两条信息：想选择什么，以及从什么地方选择





## 检索单个列



![image-20200322210728532](images/image-20200322210728532.png)

假设表的结构如图所示



检索username这一列

```mysql
 SELECT username FROM user;
```

即可，标准格式`SELECT (COLUMNS) FROM (TABLE)`

> 数据没有排序，输出的结果可能是数据被添加到表里面的顺序，也可能不是，只要返回相同的数目即是正常的

因为SQL语句（至少mysql）是遇到分号才算一条语句的结束，所以可以将一条语句拆分成多行（也是非常推荐的），这样便于阅读。



## 检索多个列



数据表还是如上图，假设要检索username和password这两个列

```mysql
SELECT username,password FROM user
```



SQL语句一般返回原始的，无格式的数据，至于数据的格式化问题，转换问题则是要在业务层中解决





## 检索所有列



使用通配符的方式来解决 *代表任意

```mysql
SELECT * FROM user
```

> :warning:：除非你要检索数据的每个列，最好别用*通配符，虽然省事儿，但是检索不需要的列通常会降低程序的性能



## 检索不同的行



如果检索单列，只需要MySQL返回不同的值

使用DISTINCT（不同的）关键字即可，如检索username，只要不重复的，用法如下

```mysql
SELECT DISTINCT username FROM user;
```

会返回不同结果的username



DISTINCT关键字直接作用于所有列，而不是他后面的那个列，如

```mysql
SELECT DISTINCT username,password FROM user;
```

唯有username和password都相同的列才会被省去



## 限制结果

限制SELECT的输出结果（只取前几行），使用LIMIT关键字

```MYSQL
SELECT * FROM user LIMIT 5;
```

至多返回5行数据，如果不足5行则直接返回



为了方便演示，使用Java插入了一些数据，便于查询

数据截图：

![image-20200322215536498](images/image-20200322215536498.png)

1~3的id已经被使用了，即使被数据被删除了也不能再次使用【主键的规则】

使用上述limit语句查询显示4~8的id数据



> :warning:： mysql的行索引是从零开始的

所以`SELECT * FROM user LIMIT 5`查询的是行索引为0~4的数据

使用`SELECT * FROM user LIMIT a,b`查询的是行下标从5开始后的五个数据





## 使用完全限定的表名



迄今为止使用的SQL例子都只是列名引用列，也可以使用完全限定的名字来限定列：

```mysql
SELECT user.id FROM user LIMIT 5;
```

也可以限定表名字

```mysql
SELECT user.id FROM test.user LIMIT 5;
```

通过 `表名.列名` 或者 `数据库名.表名` 来完全限定







# 排序检索数据

 

使用SELECT 的 ORDER BY 子句，根据需要排序检索出的数据



## 排序数据

使用 `SELECT * FROM user` 检索出的数据并不是纯粹的随机排列，如果不排序，数据一般都是按照它在底层表中出现的顺序显示，也就是数据插入的数据，但是如果后来对数据进行了更新或者删除的话，则此顺序会受到MySQL回收存储空间的影响，因此，如果不明确控制，不能依赖默认的排列顺序



:key:**子句**（clause）：SQL语句由子句构成，一个子句通常包括一个关键字和所提供的数据组成，有些子句是必须的，有些子句是可选的

像 `SELECT * FROM user LIMIT 5` 的SELECT语句的FROM子句就是必须的，而LIMIT子句就是可选的，此ORDER BY 子句就是可选的：

```mysql
SELECT * FROM user ORDER BY username LIMIT 10;
```

> 默认是升序排列，一般ORDER BY的字段都是检索的字段，但是ORDER BY没有检索的字段也是完全合法的





## 按多个列排序



经常会出现这种情况，如果显示雇员清单，由姓和名两列（首先按照性排列，在每个性中再按照名排列），实现此功能只需要指定列名，列名之间用逗号隔开即可



```mysql
SELECT first_name,last_name FROM user ORDER BY first_name,last_name;
```

会先按照firstname排序，再通过lastname排序firstname相同的数据

如果firstname都是唯一的，则不用对lastname进行排序







## 指定排序方向



默认是升序排序，如果要使用降序排序，则使用DESC关键字即可

按照降序排列username：

```java
SELECT * FROM user ORDER BY username DESC LIMIT 10;
```



DESC和DISTINCT关键字不同，DISTINCT关键字会作用于所有的列，让其都保持唯一，而DESC则只会保持其前面的列是降序的，其余列都是保持默认的升序



> 如果想让多个列都进行降序排列，则必须在每个列上都指定DESC关键字

默认的升序排列关键字是ASC，但是ASC用的非常的少，因为默认就是ASC的





> 区分大小写的排序设置：因为MySQL是不区分大小写的，所以A和a的排列地位是一样的，如果要进行区分，需要数据库管理员对数据库进行配置，用简单的ORDER BY是做不到的



使用ORDER BY找出id最大的值

```mysql
SELECT id FROM user ORDER BY id LIMIT 1;
```

先按照id降序排列，再取最上面一个即获取到id的最大值



> 子句会有位置的问题，例如：只有按照	FROM子句 + ORDER BY 子句 + LIMIT 子句才不会报错，否则会报错







# 过滤数据



使用SELECT语句的WHERE子句来指定搜索条件



## 使用WHERE子句



很少需要检索表中的所有行，往往只需要指定指定搜索条件，也叫过滤条件

```mysql
SELECT * FROM user WHERE id = 0;
```

WHERE子句不止仅能进行相等判断



> SQL过滤和应用过滤：
>
> 数据也可以再客户端这边过滤，即服务器端仅仅返回所有数据，然后再客户端这边去处理，这种实现很不令人满意，不仅会让客户机的效率大大降低，还会导致网络的数据传输量加大，造成带宽的浪费





>  子句的排列顺序为：FROM    WHERE   ORDER BY    LIMIT





## WHERE子句操作符



| 操作符  |        说明        |
| :-----: | :----------------: |
|    =    |        等于        |
|   <>    |       不等于       |
|   !=    |       不等于       |
|    <    |        小于        |
|   <=    |      小于等于      |
|    >    |        大于        |
|   >=    |      大于等于      |
| BETWEEN | 再指定的两个词中间 |



### 检查单个值



```mysql
SELECT * FROM user WHERE username = 'ECAC3';
```

>  使用单引号和双引号的效果相同



```mysql
SELECT * FROM user WHERE id < 10;
```



### 不匹配检查



```mysql
SELECT * FROM user WHERE id <> 10;
```

等价于

```mysql
SELECT * FROM user WHERE id != 10;
```





### 范围值检查



可以使用BETWEEN关键字，不过于其他WHERE子句的操作符稍有不同

```mysql
SELECT * FROM user WHERE id BETWEEN 5 AND 10;
```

> BETWEEN  a  AND  b
>
> 一定是查询大于等于a小于等于b的值



以上语句等价于：

```mysql
SELECT * FROM user WHERE id >= 5 AND id <= 10;
```







### 空值检查



设计表时候即可指定这个字段能否为NULL



:key: ：NULL	无值（no value），与包含字段0，空字符串不同



SELECT由一种特殊的WHERE子句来检查具有空值的NULL

```mysql
SELECT * FROM user WHERE password IS NULL;
```



> :warning:：在匹配过滤和不匹配过滤中都不会返回当前列为NULL的行







# 数据过滤

使用 WHERE 子句建立更强大的查询功能，和NOT 和 IN操作符





## 组合WHERE 子句



MySQL允许给出多个WHERE 子句，给出两种使用方式：以AND 子句或者以OR 子句的方式使用。



:key: **操作符**（operator）：用来连接或该表WHERE 子句中的子句关键字，也成为逻辑操作符



### AND 操作符

上面的那个user表不是很好用

![image-20200323131044433](images/image-20200323131044433.png)



新建一个product表，字段如下：

![image-20200323131115068](images/image-20200323131115068.png)

并在其中插入大量数据（Java操作，使用并发编程极大的提升了效率）



AND操作符：查询商品价格大于50的并且ID小于50的商品名称

```mysql
SELECT price FROM product WHERE id < 50 AND price > 50;
```

:key: **AND**：用在WHERE子句中的关键字，用来指定检索满足所有给定条件的行



> 上述条件只有两个，如果有多个，则必须都使用AND进行连接





### OR 操作符



指定MySQL检索匹配任一条件的行



如果需要检索price大于50或者ID小于50的商品名称：

```mysql
SELECT name FROM product WHERE price > 50 OR id < 50;
```



:key: OR：WHERE子句的关键字，用来检索匹配任一给定条件的行





### 计算次序



可以结合AND和OR关键字来完成复杂的查询操作



AND关键字的优先级比OR高，有可能造成操作符的错误组合

例如：查询 id在 大于两百或小于20 价格大于50的商品详细信息

```mysql
SELECT * FROM product WHERE id >200 OR id < 20 AND price >50;
```

如果不做任何其余的处理，MySQL会理解成为查询id大于200的商品或者是 id小于20且价格大于50的商品，有歧义



解决方法：使用（）来限定操作顺序



> 充分利用圆括号，不要过度的依赖于默认的计算次序，即使他确实是你想要的东西也是如此，使用圆括号没有坏处，能消除歧义





## IN 操作符



圆括号还可以结合IN 操作符，来指定条件取值

如要查询id为1，2，250的商品详细详细，可以使用IN 操作符

```mysql
SELECT * FROM product WHERE id IN (1，2，250)；
```

以上语句等价于下列语句

```mysql
SELECT * FROM product WHERE id = 1 OR id = 2 OR id = 250;
```

使用IN 操作符的优点：

- 语法清楚直观
- 操作次序易管理（减少了操作符的数量）
- 比OR操作的执行效率高
- 可以包含其他SELECT 语句



:key: **IN** ：WHERE子句的指定匹配值的关键字，功能与OR相当





### NOT操作符



:key: NOT：WHERE子句中用来否定后跟条件的关键字



例子：用来列出id除1和2以外的所有商品

```mysql
SELECT * FROM product WHERE id NOT IN (1,2);
```



> mysql中的NOT关键字只支持对IN，BETWEEN和EXISTS子句取反，与其他的DBMS有很大的差距





# 使用通配符进行过滤



## LIKE操作符



前面介绍的操作符都是对已知的数据进行过滤，这种过滤方法很有局限性，例如如何查找名字 中包含a的产品，可以构造一个**通配符搜索模式**来解决这个问题



:key: 通配符（wildcard）：用来匹配值的一部分的特殊字符

:key:搜索模式（search pattern）：由字面值，通配符构成的搜索条件



为在搜索模式中使用通配符，必须使用LIKE关键字



### 百分号（%）通配符

最常用的通配符就是%，他表示任意字符出现任意次数（可以出现0次）



检索a开头的商品的详细详细：

```mysql
SELECT * FROM product WHERE name LIKE 'a%';
```

> 以上SQL语句可以匹配到名字为a的商品



检索名字中包含a3的商品的详细详细

```mysql
SELECT * FROM product WHERE name LIKE '%a3%';
```



检索以2开头，以1结尾的商品的详细信息

```mysql
SELECT * FROM product WHERE name LIKE "2%1";
```



:warning:：即使通配符%可以匹配任何字段，但也没法匹配NULL



### 下划线（_）通配符



_只能匹配单个字符，不能多也不能少



```mysql
SELECT * FROM product WHERE id LIKE "23a31_";
```



## 通配符使用技巧

通配符很有用，但是有代价的，通配符花费的时间一般比其他搜索的时间更长

下面给出一些使用技巧：

- 尽量使用其他操作符，不要过分依赖通配符
- 除非真的有必要，否则不要把通配符置于搜索模式的开始处，效率会比较低
- 仔细确定通配符的位置





# 用正则表达式进行搜索



## 正则表达式简介

前面的通配符对于基本的过滤是够用了，但是随着过滤条件的增加，WHERE子句本身的复杂性也会增加

正则表达式（regexp）可以用来匹配文本的特殊的串，正则表达式也广泛用于程序设计，文本编辑器，操作系统等





## 使用正则表达式



MySQL可以使用正则表达式，过滤SELECT检索出的数据



> MySQL仅支持正则表达式的一个很小的子集



### 基本字符匹配



检索包含a字符的商品的详细信息

```mysql
SELECT * FROM product WHERE name REGEXP "a";
```

当然，这个例子也可以轻松使用LIKE去完成，甚至效率会更高，但是某些情况必须要用到正则表达式（例如只允许匹配到数字，通配符是不提供这个功能的）



> LIKE和REGEXP的区别：
>
> - SELECT * FROM product WHERE name LIKE "1000";
> - SELECT * FROM product WHERE name REGEX "1000";
>
> 第一条语句只会返回name为1000的数据（没有通配符）
>
> 而第二条语句会返回包含1000的name的数据，如果要第二条语句也像第一条语句一样只返回name为1000 的数据，则需要使用到^和$定位符





### 同时匹配多个字符中任意一个



为搜索两个串之一，可以使用 | 操作符，例子：

匹配name中包含d3或者23的字符

```mysql
SELECT * FROM product WHERE name REGEXP "d3|23";
```

| 为正则表达式的OR操作符，如果有多个OR条件，则使用多个| 如 WHERE name REGEXP "d3|23|33";



### 匹配几个字符之一



如果想匹配指定的几个字符之一，则使用[和]来完成

例如，想检索id 为 11，21，31的商品详细信息

```mysql
SELECT * FROM product WHERE id REGEXP "[123]1";
```

但是会检索出id为210的数据 :laughing:例子举的不好，大概这个意思把



### 匹配范围

如果要匹配数字0~9，可以使用[0123456789]去进行匹配，为简化，可以使用-，上述式子等价于[0-9]



例如：

```mysql
SELECT * FROM product WHERE id REGEXP "[1-2][5-6][7-8]"
```

即可匹配到名字中包含157，158，167，168，257，257，267，268的数据



也可以指定匹配a到z的字符，如下：[a-z]



### 匹配特殊字符

如果要匹配名字中包含.的商品，使用如下SQL语句

```mysql
SELECT * FROM product WHERE name REGEXP ".";
```

则会返回所有，因为.可以匹配任意字符

正确SQL语句为：

```mysql
SELECT * FROM product WHERE name REGEXP “\\.”
```

正则表达式里面具有特殊意义的字符都需要使用\\\来转义，有些字符转义后也会有特殊的意义，如：

![image-20200323143548247](images/image-20200323143548247.png)



> 如果是要匹配  \ 的话，则需要使用   \  \  \ 来匹配





### 匹配字符类

为简化常用的匹配规则，将其抽取出来成为字符类

![image-20200323143900584](images/image-20200323143900584.png)

:warning:：有些地方用不了，尽量少用



### 匹配多个实例

到目前为止正则表达式还只能匹配到单个的字符，如果要同时匹配99个s，你不可能写99个s到正则表达式中把，可以使用重复元字符来进行更强的控制



| 元字符 |             说明             |
| :----: | :--------------------------: |
|   *    |         0或多个字符          |
|   +    |    1或多个字符，等价{1，}    |
|   ？   |    0或1个字符，等价{0，1}    |
|  {n}   |        指定数目的匹配        |
| {n，}  |     不少于指定数目的匹配     |
| {m，n} | 匹配指定的范围（m不超过255） |



- 如果要匹配	(1 stick)  和  (5 sticks)

使用如下正则表达式

```regexp
\\([0-9] sticks?\\)
```

- 匹配连在一起的四位数字

```regexp
[0-9]{4}
```





### 定位符

指定匹配文本的位置



| 元字符  |   说明   |
| :-----: | :------: |
|    ^    | 文本开始 |
|    $    | 文本结束 |
| [[:<:]] | 词的开始 |
| [[:>:]] | 词的结束 |



如果要匹配所有以数字开头（包括小数点）的商品，SQL如下

```mysql
SELECT * FROM product WHERE name REGEXP "^[0-9\\.]";
```

> 一旦  . 放到了[] 里面去了，   .就会自动转义成  \ \ .   为了避免歧义，还是写成\ \ .  



   





# 创建计算字段





## 计算字段



存在很多情况，存储在表中的数据都不是应用程序所需要的，我们希望直接在数据库层面上检索，转换出需要的数据，而不是直接检索出所有数据，再传递给客户端去重新在应用程序中格式化。这就依赖于计算字段了



与前面所介绍的列不同，计算字段并不实际存在于数据库表中，计算字段是运行SELECT语句内创建的



:key: 字段（field）：基本和列的意思相同，习惯上将数据库列称为列，字段一般用在计算字段的连接上



在客户机的角度上：计算字段和普通的列是一样的



> 能在服务器端完成的操作一般也能在客户机完成，但一般来说在服务器端的运行速率比客户机的处理效率要高



## 字段拼接



:key: 拼接（concatenate）：将值连接到一起形成单个值



要将商品的名称和价格按照：名称（价格）的格式来给定

解决方法：使用Concat函数来拼接两个列

```mysql
SELECT Concat(name,"(",price,")") from product;
```

Concat 函数具有拼接字符串的作用，形成一个长串，列名为Concat(name,"(",price,")")

:warning:如果查询的name或者price其中一个是NULL的，那么整个列就是NULL



可以通过删除name右侧的空格来整理数据，通过RTrim（）函数来完成

上面的SQL可以优化为：

```mysql
SELECT Concat(RTrim(name),"(",RTrim(price),")") from product;
```

去掉了数据右侧的空格，减少了网络的带宽浪费

> RTrim去除右侧空格，LTrim去除左侧空格，Trim去除左右空格



**使用别名来优化查询结果的列名显示**，因为客户端无法识别列名Concat(RTrim(name),"(",RTrim(price),")")

优化成为去除左右两边空格

```mysql
SELECT Concat(Trim(name),"(",Trim(price),")") product_info from product;
```

> 标准格式应该为：SELECT Concat(Trim(name),"(",Trim(price),")") AS product_info from product;   不过AS关键字可以省略





## 执行算术计算



如果有一张订单表，里面存有三列，分别为订单号，单价和数量，现在算出每个订单号的总价

我们就不新建表了，直接使用product表，用id代表数量，price代表单价

计算总价：

```mysql
SELECT name,id,price,id*price AS total FROM product ORDER BY name
```

> 其他算术操作符： +  -   *   /



> 可以使用SELECT关键字来简单的访问和处理表达式，如：
>
> - 算术运算： SELECT 1*10;
> - 去除空格：SELECT Trim("  a  b  c  ");
> - 返回当前时间：SELECT Now();





# 使用数据处理函数



## 函数

SQL支持函数来处理数据，前一章中处理数据左右空格的Trim（）就是一个函数



> 函数的可移植性就没有SQL的可移植性强，因为每种DBMS的SQL都是大同小异的，且都支持一些操作，只要修改一些关键词即可
>
> 而每个BDMS都实现了大量特定的函数，不易在不同的DBMS中间移植





## 使用函数



大多数的SQL都实现了以下类型的函数：

- 处理文本串的文本函数（删除填充数值，大小写转换等）
- 算术运算的数值函数（返回绝对值，进行代数运算）
- 用于日期处理的函数
- 返回System的信息的函数



### 文本处理函数

去除左右空格的Trim函数我们已经见过了，再来介绍一个常用的

让字母大写的函数Upper函数



```mysql
SELECT Upper(name) AS upper_name FROM product LIMIT 5;
```

还有一些常见的文本处理函数如下：

|     函数      |       功能        |
| :-----------: | :---------------: |
|   Left（）    | 返回串左边的字符  |
|  Length（）   |   返回串的长度    |
|  Locate（）   | 找出串的一个子串  |
|   Lower（）   |  将串转换为小写   |
|   LTrim（）   |   去除左边空格    |
|   RTrim（）   |   去除右边空格    |
|   Rigth（）   | 返回串右边的字符  |
|   Upper（）   |  将串转换出大写   |
| SubString（） |  返回子串的字符   |
|  Soundex（）  | 返回串的SOUNDEX值 |



Soundex函数将串转换出为SOUNDEX（类似的发音字符和音节）



使用的时候了解下语法就行了





### 日期和时间处理函数

![image-20200323202030378](images/image-20200323202030378.png)

我们以前只是使用WHERE关键字来过滤字符和数值的操作，也可以使用WHERE关键字来过滤时间日期操作



和Java的一样，MySQL使用的日期格式也为 yyyy-mm-dd



使用datetime的数据类型来存储时间



在Easy Code的插件的MySQL的字段和Java的类之间的转换可以看出：

![image-20200323205452471](images/image-20200323205452471.png)

datatime和timestamp会被转换成为Java的Data数据类型



设计表为：

![image-20200323210245054](images/image-20200323210245054.png)

千万别把表名设置成了order，order 是 关键字，会报错

此时数据表中的所有数据为：

![image-20200323210525308](images/image-20200323210525308.png)

如果此时要查询order_date为2000-09-01的订单，使用以下SQL语句

```mysql
SELECT * FROM orders WHERE order_date = "2000-09-01";
```

只能查询出订单号为2的数据，即如果没有指定到时分秒，默认都是补零，这与我们实际当中的用法有很大的冲突

解决方法：使用库函数将order_date的日期部分拿出来作比较即可

```mysql
SELECT * FROM orders WHERE Date(order_date) = "2000-09-01";
```



> 在你仅需知道的是日期的时候，可以直接使用Date（）函数，尽管你知道相应的列只包含日期也是如此，如果只想获取时间，同样也可以使用Time（）函数
>
> 这两个函数都是在MySQL 4.1.1 之后引入的





还有一种情况就是只要获取2000年09月的订单：



- 可以使用

```mysql
 SELECT * FROM orders WHERE Date(order_date) BETWEEN "2000-09-01" AND "2000-09-30";
```

但是如果你误将9月记成了31天，那么就会一条数据也输不出来，尽管这些数据都是满足要求的，但是MySQL会自动的检测你输入的日期是否合法，不合法也只会给你一个Warning

![image-20200323211825855](images/image-20200323211825855.png)



- （不需要记住每个月有多少天的方法）

```mysql
 SELECT * FROM orders WHERE Year(order_date) = “2000” AND Month(order_date) = ”9“;
```



最好碰上时间数据都加上引号，`Year(order_date) = 2000` 也是合法的，但是`Date(order_date) = 2000-09-01`就是违法的了



> :warning:：在数据库规范中没有双引号，一般都是单引号





### 数值处理函数

![image-20200323212557592](images/image-20200323212557592.png)

用来进行数值处理，运用的往往不如日期的函数那么频繁，但是却是各个DBMS最统一的一块







# 汇总数据



## 汇集函数



我们经常只需要数据的汇总情况，而不是每一条数据的详细信息，主要的检索例子有以下几种：

- 确定行数
- 确定行组的和
- 找出列的最大值，最小值，平均值

如果将这些任务交给客户端去处理，不仅客户机要编写相应的处理函数，还会增加信息传输造成的带宽浪费（前提是你仅仅只想知道上述罗列的检索）



:key:聚集函数（aggregate function）：运行在行组上，计算和返回单个值的函数。

![image-20200323213258430](images/image-20200323213258430.png)



> MySQL还支持一些标准偏差函数，这里没有列举





### AVG（）函数



返回product的price字段的平均值

```mysql
SELECT AVG(price) FROM product;
```

或者id<50的price平均值

```mysql
 SELECT AVG(price) FROM product WHERE id < 50;
```



AVG函数只能返回单个列的平均值，如果要输出多个列的平均值，要多次使用AVG函数

> AVG函数在统计时自动忽略 NULL 值





### COUNT（）函数

返回检索出的数据的行数

COUNT的两种使用方式：

- 使用COUNT（*）直接检索行数，不去管行中是否有NULL值
- 使用COUNT（Field）会返回该字段非NULL的行数

![image-20200323214425808](images/image-20200323214425808.png)

![image-20200323214433301](images/image-20200323214433301.png)

可以通过这种方式确定列password存在一个NULL





### MAX（）函数

返回指定列的最大值，简单用法：

```mysql
SELECT MAX(price) FROM product;
```

虽然MAX函数一般是用来对数据和日期来返回最大值，但也可以返回文本列的最大值，自动返回升序的最后一行



> MAX函数自动忽略NULL的行







### MIN（）函数



返回列的最小值，用法与MAX差不多



```mysql
SELECT MIN(price) FROM product;
```





### SUM（）函数

可以直接嵌套*运算计算总金额

```mysql
SELECT SUM(id*price) FROM product;
```

一样忽略NULL





## 聚焦不同值



主要是DISTINCT的使用，DISTINCT默认直接限制后面所有的列



计算所有price的平均值（去除权重）

```mysql
 SELECT AVG(DISTINCT price) FROM product;
```

计算不同name的数量

```mysql
SELECT COUNT(DISTINCT name) FROM product;
```



name字段中重复的和为NULL的行数：

```mysql
SELECT COUNT(*)-COUNT(DISTINCT name) name_num FROM product;
```





## 组合聚集函数



就是SELECT里面包含多个函数



```mysql
SELECT COUNT(*),AVG(price),MAX(price) FROM product;
```

可以给每个字段取个别名，便于理解和复用







# 分组数据



## 数据分组



会有这种情况，统计每个厂商的产品的平均价格，最大价格，最小价格等，统计其中一个厂商我们可以轻而易举的达到，但是要一次统计多个，我们还办不到，这就需要数据分组



## 创建分组



数据库表模拟（表名最好都带上s）：

![image-20200323222743439](images/image-20200323222743439.png)



```mysql
SELECT vend_id,AVG(price) FROM products GROUP BY vend_id;
```

结果如图所示

![image-20200323223033331](images/image-20200323223033331.png)

因为使用了GROUP BY 进行了分组，统计结果就是对每个组而言的，而不是对整个集进行聚集



GROUP  BY可以进行嵌套，还有很多细微规定。



一旦使用了 GROUP BY 关键字后，除了根据分组的那个变量以外，其他变量都是去了意义，只有他们的统计量有意义

例如上面的例子中，只有每一行数据的vend_id有意义，其他的字段只有他们的SUM，MAX等才有意义



## 过滤分组



使用GROUP BY 进行分组后，如何进行操作筛选掉一些不满足条件的组呢？不能使用WHERE子句，WHERE只能操作行，要使用HAVING子句



可以使用HAVING子句代替所有的WHERE子句



> HAVING和WHERE的区别：WHERE用来过滤操作行，而HAVING用来直接过滤操作一个分组



HAVING由于是站在组的基础上的，所以他只支持一些统计量如AVG，SUM，MIN，MAX的判断，不能直接指定单个值的判断





WHERE和HAVING复用的例子：

对id大于8的行根据vend_id进行分组，并且筛选出平均价格大于70的数据，并展示平均价格

![image-20200323225653842](images/image-20200323225653842.png)



关键词使用的顺序：

SELECT-->FROM-->WHERE-->GROUP BY-->HAVING-->ORDER BY-->LIMIT 