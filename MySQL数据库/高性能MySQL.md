# 高性能MySQL



## 第一章、MySQL架构与历史





MySQL具有很高的灵活性，可以通过调整配置让其在不同的硬件设备上良好的运行

MySQL最不同的是他的存储索引架构，这种架构可以轻易的将查询和任务存储进行分离，也就是我们常说的读写分离，可以很轻易的提升设备的存储性能和QPS



MySQL的逻辑架构图（并不是只有简单的执行SQL然后进行数据查询）：

![image-20200921105913590](images/image-20200921105913590.png)



最上层的工具往往是第三方提供的，例如Spring提供的JDBCTemplate就属于最上层，其中往往包含连接处理，授权认证等等

第二层是所有跨存储引擎的功能的实现地方，如：存储过程，触发器，视图等等

第三层包含存储引擎，负责MySQL中数据的存储与提取。存储引擎本身并不会去解析SQL，不同引擎之间也不会相互通讯，仅仅只是简单的响应上层服务的请求



MySQL的连接管理：当一个客户端连接到MySQL上时候，会单独拥有一个线程，该线程往往只会轮流的在某个CPU核心或者是CPU中运行（为了保证缓存的高效利用率），服务器也直接负责了线程的管理，相当于缓存优化与线程管理MySQL已经帮我们实现了。



MySQL的安全性检查：可以使用账户密码或者是安全套接字（SSL）的方式连接，一旦连接成功，MySQL还会对该用户拥有的权限进行检查，如是否对该表有执行SELECT语句的权限



MySQL的优化：MySQL在解析查询的时候会进行优化，实际上就是创建解析树（一种内部数据结构）并对该结构进行优化，当然也可以提示优化器进行优化（就类似于Java中提示GC进行垃圾收集一样，但是具体执不执行没有任何保障）。优化器是独立于存储引擎的，但是存储引擎是对最后的查询速率有影响



MySQL的执行：在执行SQL语句之前，仅限于SELECT语句，如果在缓存中可以查看到相应的结果就会直接返回，服务器不会在对其进行查询（当然要避免大量SQL的传递，虽然MySQL服务器这端可以很好的优化，但是网络传输可能也会对其造成影响）



MySQL的并发控制：通过锁来避免数据被损坏。虽然这种锁的方式（这里指的应该是悲观锁）在实际工作环境中工作良好，但是不支持并发处理，因为在同一时刻只会存在一个进程去修改数据。

解决这个可以使用共享锁和排他锁，也叫读锁和写锁。

读锁是共享的，保证与其他的锁是不会相互阻塞的。写锁是排他的，写锁会阻塞其他的写锁和读锁。

当然其中也有优化的空间，可以控制锁的粒度达到写锁只需要锁定一小部分数据，而不用直接锁定全局的数据。如果仅仅只是这样那么并发问题早就不存在了，主要是操作锁也会消耗相当一部分的资源，如果实现了粒度过于细的锁，操作锁的开销可能可以与存储数据的开销成为同一个量级的。

所以所谓的锁策略是指在锁的开销和数据的安全性之间寻求一个平衡，往往是通过性能来衡量这种平衡是否合理，一般的数据库对其支持也就是行锁。而MySQL则提供了多种选择，每种MySQL的存储引擎都实现了自己的锁策略和锁粒度，于是MySQL便原生的支持了各种场景

如：表锁：直接锁定整张表，是MySQL中开销最小的锁策略，虽然各个存储索引可以管理自己的锁，但是MySQL服务器层整体还是会在使用某些语句时候如`ALTER TABLE`的时候使用表锁，而不再去依靠存储引擎的限制。



行锁：行锁可以最大程度地支持并发处理，但是同样的，也是带来最大的锁开销，行锁仅仅只是在存储索引中有实现，而在MySQL服务器层没有相应的实现





MySQL的事务支持：

事务就是一组原子性的SQL查询（不一定，只要是一组SQL语句即可），或者说是一个独立的工作单元。事务内的语句，要么全部执行，要么全部执行失败，不会出现仅仅执行一部分的情况



事务其实是依赖于数据库的ACID特性的，并不是仅仅只是依赖他的原子性的

A（atomicity，原子性）：事务必须被视为是一个不可分割的最小工作单元

C（consistency，一致性）：数据总是从一个一致性的状态转换到另一个一致性的状态

I（isolation，隔离性）：一个事务所做的提交在最终提交之前，对其他事务是不可见的

D（durability，持久性）：一旦事务提交，则其所做的修改会永久的保存到数据库中，其实也分很多级别，有一些低级别的持久化策略无法100%保障



当然，这些ACID带来的安全性也是以牺牲性能换来的，一个实现了ACID的数据库往往需要更高的硬件配置才能达到同等效果。当然MySQL并不是强制支持事务的，将对事务的支持划分到了存储索引这个级别上了。





隔离级别（隔离性的具体体现）：提供了四种隔离级别，每一种隔离级别都规定了一个事务中锁进行的修改在哪些事务内核事务间是可见的，哪些是不可见的，同样的，低隔离级别的事务可以提供更高的并发，系统的开销也更低。



四种隔离级别：

- READ UNCOMMITTED（未提交读）

事务未提交的修改对其他事务来说也是可见的，读取未提交的数据，这种情况被称为【脏读】，脏读也就是对错误数据的读取

- READ COMMITED（提交读）

大多数的MySQL的默认隔离级别，但是MySQL不是。

非常满足前面对隔离性的简单定义，事务只能看见别的已经提交的事务的更改

有时候这个级别也叫【不可重复读】（NOREPEATABLE READ），因为两次相同的查询可能会得到不一样的结果

- REPEATABLE READ（可重复读）

保证同一个事务多次读取的数据是一样的，但无法解决【幻读】的问题

InnoDB的默认隔离级别

- SERIALIZABLE（可串行化）

最高的隔离级别，强制所有事务串行执行，不存在之前的脏读，不可重复读，幻读的问题，但是这种安全性是牺牲并发容量带来的，一般很少会直接采取这种隔离级别

> 脏读，不可重复读，幻读：
>
> #### 脏读
>
> 所谓脏读是指一个事务中访问到了另外一个事务未提交的数据，如下图：
> ![15352625974682.png](https://user-gold-cdn.xitu.io/2018/8/27/1657927364864917?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
> 如果会话 2 更新 age 为 10，但是在 commit 之前，会话 1 希望得到 age，那么会获得的值就是更新前的值。或者如果会话 2 更新了值但是执行了 rollback，而会话 1 拿到的仍是 10。这就是脏读。
>
> #### 幻读
>
> 一个事务读取2次，得到的记录条数不一致：
> ![15352627898696.png](https://user-gold-cdn.xitu.io/2018/8/27/1657927364899067?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
> 上图很明显的表示了这个情况，由于在会话 1 之间插入了一个新的值，所以得到的两次数据就不一样了。
>
> #### 不可重复读
>
> 一个事务读取同一条记录2次，得到的结果不一致：
>
> ![15352628823137.png](https://user-gold-cdn.xitu.io/2018/8/27/165792736493afd2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
> 由于在读取中间变更了数据，所以会话 1 事务查询期间的得到的结果就不一样了。



隔离级别对应的数据安全性问题：

![15352624354970.jpg](https://user-gold-cdn.xitu.io/2018/8/27/1657927364adccc5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)





死锁：

MySQL的事务执行过程中可能会使用到行锁，便有可能出现以下问题：事务1执行时候锁定第一行，要获取第二行，事务二执行时候锁定第二行，要获取第一行。

MySQL给出的解决办法是：提供了死锁检测和死锁超时机制。如InnoDB存储引擎，能检测死锁的循环依赖并立即返回一个错误，放碰到错误的时候将持有少量排他锁的事务进行回滚（相对简单的死锁回滚算法）。

当然死锁产生的原因并不仅仅只是数据冲突，还有可能和存储索引的实现有关。



MySQL事务日志：使用事务日志可以提高事务的效率。事务日志是在设备的内存中的，当存储引擎在修改数据表的数据时候往往只需要修改内存中的事务日志，而不用每次都直接修改磁盘上的数据。MySQL可以保证在系统崩溃的时候其中的数据不丢失。





MySQL中的事务支持：MySQL提供了两种存储引擎支持事务：InnoDB和NDB Cluster，当然，由于MySQL的开源，有很多第三方的存储引擎也支持事务。

MySQL默认采用的是默认提交机制，即每一条散着的命令都是一个事务操作

还有一些语句会强制COMMIT当前的活动事务，如ALTER TABLE和LOCK TABLES语句都会产生这种效果

可以改变当前会话的隔离级别：

`SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;`

InnoDB支持所有的隔离级别

事务是由存储引擎实现的，在同一个事务中使用多种存储引擎是不可靠的

InnoDB采用两阶段锁定模式，即显式锁定和隐式锁定，隐式锁定发生在事务当中，只有在COMMIT或者是ROLLBACK的时候才会释放锁。至于显式锁定则是InnoDB自己提供的语言规范，不属于SQL规范：

![image-20200921172014138](images/image-20200921172014138.png)



当然在MySQL的服务器层面也是支持显式锁的：`LOCK TABLES`和`UNLOCK TABLES`语句，不涉及到这里的事务控制

> 建议不要使用数据库服务器层面的锁，因为很容易与存储引擎的锁发生冲突从而降低性能
>
> **在任何时刻都不要显式地执行LOCK TABLES，不管使用的是什么存储引擎**







MySQL多版本并发控制：

多数MySQL存储引擎都没有使用行级锁，而是基于并发速率考虑使用了MVCC（多版本并发控制）来提升效率。

可以认为MVCC是一个行级锁，但是在很多情况下避免了加锁操作~~（个人理解：行级锁就相当于是一个悲观锁，而MVCC就相当于是乐观所）~~

> 错误的理解，MVCC实现有乐观并发控制和悲观并发控制两种实现类型



InnoDB的MVCC实现方式：通过在每行后面记录两个影藏的列来实现的，分别保存子行的创建时间和过期时间（删除时间），当然这里存储的不是真实的时间，而是系统版本号，每开启一个新的事务，系统版本号就会自动递增。分别称为行版本号和删除版本号



在REPEATABLE READ，也就是默认的情况下，MVCC是如何操作的：

SELECT：

- 查找行的系统版本号小于当前事务系统版本号的数据行
- 查找行的删除版本号要么是未定义，要么大于当前事务系统版本号

INSERT：

将当前事务系统版本号作为行版本号

DELETE：

将当前事务版本号作为行删除版本号

UPDATE：

将行版本号和删除版本号设置成当前事务的系统版本号



这样可以保证绝大多数操作都可以在不加锁的情况下安全的完成



MVCC只能在READ COMMITED和REPEATABLE READ这两个隔离级别下工作，其他的都不兼容。第一个是完全不能用，要保证读取的都是最新的数据，最后一个则是没必要用，读取的时候会对行加锁。



MySQL的存储引擎：

存储引擎对应到了每一张表上，可以通过查看表的详细信息来查看该表使用的是什么存储引擎

如：`show create table sys_user \G;`即可查看到sys_user表的详细定义：

```sql
*************************** 1. row ***************************
       Table: sys_user
Create Table: CREATE TABLE `sys_user` (
  `id` int NOT NULL AUTO_INCREMENT,
  `user_name` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '用户账号',
  `user_password` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '密码',
  `real_name` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '用户真实姓名',
  `dep_id` int NOT NULL COMMENT '部门id',
  `user_code` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci DEFAULT NULL COMMENT '员工工号',
  `user_type_id` int DEFAULT NULL COMMENT '用户类型id',
  `order_number` varchar(3) CHARACTER SET utf8 COLLATE utf8_general_ci DEFAULT NULL COMMENT '排序',
  `user_email` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci DEFAULT NULL COMMENT 'email地址',
  `user_tel` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '手机号',
  `current_score` int DEFAULT '0' COMMENT '积分',
  `has_approve` int DEFAULT '0' COMMENT '是否实名认证,0-未认证,1-已认证',
  `status` int NOT NULL DEFAULT '0' COMMENT '状态是否有效,0-注册,1-有效,2-离职',
  `news_scope` varchar(3) CHARACTER SET utf8 COLLATE utf8_general_ci DEFAULT NULL COMMENT '本地声音中的信息范围(0-本地，1-上级，2-下级，012-全部)',
  `create_time` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `create_oper` int DEFAULT NULL COMMENT '创建用户id',
  `update_time` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  `update_oper` int DEFAULT NULL COMMENT '修改用户id',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC COMMENT='用户表'
1 row in set (0.00 sec)
```

> 之所以加上了`\G`在最后面是因为要输出的时候不会带上大量的短横线

这是查询创建表的语句，当然也可以查看到表的存储引擎。

但是更多的是创建表的SQL语句，建议使用如下SQL语句：`show table status like 'sys_user'\G;`来展示，\G可以保证格式化输出，其中包含大量表统计信息。

> 包括SELECT语句都可以在后面加上\G用于格式化输出

得到的详细信息为：

```sql
*************************** 1. row ***************************
           Name: sys_user
         Engine: InnoDB
        Version: 10
     Row_format: Dynamic
           Rows: 1
 Avg_row_length: 16384
    Data_length: 16384
Max_data_length: 0
   Index_length: 0
      Data_free: 0
 Auto_increment: 2
    Create_time: 2020-09-06 15:39:40
    Update_time: 2020-09-06 15:52:14
     Check_time: NULL
      Collation: utf8_general_ci
       Checksum: NULL
 Create_options: row_format=DYNAMIC
        Comment: 用户表
```

更多的是显示当前Table的状态，而不是显示当前SQL的格式





InnoDB存储引擎：MySQL默认的存储引擎

适用于短期事务，即具有大部分事务都可以被正常提交，只有少量会发生事务回滚，

InnoDB提供了如新的大型值BLOB的存储方式

InnoDB采用MVCC来支持高并发，实现了默认的四个隔离级别，**默认是REPEATABLE READ，并且通过间隙锁的方式防止了幻读的出现**（这就意味着：InnoDB实现了不是SERIALIZABLE的条件下的所有错误发生可能）。

InnoDB是根据聚簇索引建立的，对主键索引的查询速率非常高，不过二级索引中必须包含主键列

InnoDB支持热备份，而这点对于其他的存储引擎是不支持的





MyISAM存储引擎：在5.1之前是默认的存储引擎，提供了全文索引等特性，但是不支持事务和行级锁，且崩溃后无法安全的恢复。不是非常建议使用，后期维护成本非常高。

由于不支持行级锁，所以在读取表的时候需要将涉及到的表加上共享锁，而在写入表的时候需要加上排他锁。但在查询表的过程中可以往表中查询新的记录（被称为并发插入）

MyISAM可以进行表的压缩，压缩后的表示不能进行修改的，想要修改，需要先将表解压缩然后才能进行修改，压缩过的表也支持索引，但索引仍然只是只读的，仍然需要先将数据解压再进行修改索引



MySQL内建的其他存储引擎：不是很建议使用，说不定在不久的将来就被不予支持，Archive引擎、Blackhole引擎，CSV引擎，Federated引擎，Memory引擎，Merge引擎等等。

市场上的第三方引擎：

OLTP类引擎，完全为了取代InnoDB引擎的市场地位，对InnoDB引擎进行了优化

面向列的存储引擎：因为MySQL默认是面向行的，在大数据处理时候往往需要面向列，这样才能有更高的数据处理速率。



MySQL对存储引擎的选择问题：“大部分时间选择InnoDB存储引擎，除非需要用到InnoDB不具备的特性，并且没有其他任何办法可以替代，否则都应该优先选择InnoDB引擎”。

例如需要用到全局索引，可以使用InnoDB+Sphinx的组合，而不是去使用MyISAM存储引擎。

除非迫不得已，非常不推荐混用多种存储引擎



选择存储引擎的一些例子：P61



转换表的引擎：`ALTER TABLE sys_user ENGINE = InnoDB;`

可能会执行很长时间，因为MySQL会将表数据复制一份到新的表上面去，然后通过指定新的表的存储引擎来达到修改引擎的目的。还有一个问题：如果从InnoDB表转换为MyISAM表，再转换回InnoDB表，那么这个表上的外键将会丢失。



还有一种方法是创建一个类似于当前表的表结构，然后将当前表的数据插入到新创建的表中

操作如下：

```sql
CREATE TABLE book2 LIKE book;
ALTER TABLE book2 ENGINE = MyISAM;
INSERT INTO book2 SELECT * FROM book;
```

可以达到快速更改存储引擎的目的，因为数据表是空的，修改引擎的速度非常快。

如果数据量大的话可以考虑分批次插入到book2当中去 



MySQL的历史：

最靠是MySQL公司独立研发，后来被sun公司收购，同时Oracle公司收购了InnoDB存储引擎。最后再5.5版本的时候Oracle完成对sun的收购并推出了该版本的MySQL

> 难怪Java对MySQL有这么好的支持，原来都是同一个公司出品的产品，自家的适配性当然要好了。



Oracle现在是为用户提供一些MySQL的服务器插件来赚取利润，MySQL本身还是遵循开源模式的





总结：

MySQL的分层架构：上层是服务器层的服务和查询执行引擎，下层是存储引擎。

随着Oracle对MySQL和InnoDB的收购，更加有利于MySQL的发展