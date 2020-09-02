# 第一部分、入门







## 第一章、初识Redis





Redis提供了5种不同类型的数据结构：字符串（STRING），列表（LIST），集合（SET），散列表（HASH），有序集合（ZSET）

结构特征：

| 结构类型 |                         结构存储的值                         |
| :------: | :----------------------------------------------------------: |
|  STRING  |                   字符串，整数或者是浮点数                   |
|   LIST   |          一个链表，链表上的每个节点都存储一个STRING          |
|   SET    |            包含STRING的无序收集器，且是独一无二的            |
|   HASH   |                    包含键值对的无序散列表                    |
|   ZSET   | 字符串成员（num）与浮点数分值（score）之间的有序映射，排列顺序由分值大小决定 |



>  使用redis-cli来连接redis命令行界面
>
> `redis-cli -h localhost -p 6379`连接到Redis命令行中去
>
> 如果设置了密码需要使用`auth <密码>`的方式来实现登陆，不然会报AUTH错误



STRING使用示例

```redis
127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> get hello
"world"
127.0.0.1:6379> get Hello
(nil)
127.0.0.1:6379> del hello
(integer) 1
127.0.0.1:6379> get hello
(nil)
```





LIST使用示例：

```redis
127.0.0.1:6379> rpush list-key item
(integer) 1
127.0.0.1:6379> rpush list-key item2
(integer) 2
127.0.0.1:6379> rpush list-key item
(integer) 3
127.0.0.1:6379> 
// -1为范围内的结束索引，即直接查看全部
127.0.0.1:6379> lrange list-key 0 -1
1) "item"
2) "item2"
3) "item"
127.0.0.1:6379> lindex list-key 1
"item2"
127.0.0.1:6379> lpop list-key
"item"
127.0.0.1:6379> lrange list-key 0 -1
1) "item2"
2) "item"
```

> redis中的LIST操作远不止此





SET使用示例：

```redis
127.0.0.1:6379> sadd set-key item
(integer) 1
127.0.0.1:6379> sadd set-key item2
(integer) 1
127.0.0.1:6379> sadd set-key item3
(integer) 1
127.0.0.1:6379> smembers set-key
1) "item"
2) "item2"
3) "item3"
127.0.0.1:6379> sismember set-key item4
(integer) 0
127.0.0.1:6379> sismember set-key item1
(integer) 0
127.0.0.1:6379> sismember set-key item
(integer) 1
127.0.0.1:6379> srem set-key item2
(integer) 1
127.0.0.1:6379> sismember set-key item2
(integer) 0
127.0.0.1:6379> smembers set-key
1) "item"
2) "item3"
```





HASH使用示例：

```redis
127.0.0.1:6379> hset hash-key sub-key1 value1
(integer) 1
127.0.0.1:6379> hset hash-key sub-key2 value2
(integer) 1
127.0.0.1:6379> hset hash-key sub-key3 value3
(integer) 1
127.0.0.1:6379> hgetall hash-key
1) "sub-key1"
2) "value1"
3) "sub-key2"
4) "value2"
5) "sub-key3"
6) "value3"
127.0.0.1:6379> hset hash-key sub-key3 value1
(integer) 0
127.0.0.1:6379> hgetall hash-key
1) "sub-key1"
2) "value1"
3) "sub-key2"
4) "value2"
5) "sub-key3"
6) "value1"
127.0.0.1:6379> hget hash-key sub-key1
"value1"
127.0.0.1:6379> hdel hash-key sub-key1
(integer) 1
127.0.0.1:6379> hgetall hash-key
1) "sub-key2"
2) "value2"
3) "sub-key3"
4) "value1"
```





ZSET使用示例：

```redis
127.0.0.1:6379> zadd zset-key 200 member1
(integer) 1
127.0.0.1:6379> zadd zset-key 400 member2
(integer) 1
127.0.0.1:6379> zadd zset-key 400 member3
(integer) 1
127.0.0.1:6379> zadd zset-key 100 member2
(integer) 0
127.0.0.1:6379> zrange zset-key 0 -1 withscores
1) "member2"
2) "100"
3) "member1"
4) "200"
5) "member3"
6) "400"
127.0.0.1:6379> zrangebyscore zset-key 0 200 withscores
1) "member2"
2) "100"
3) "member1"
4) "200"
127.0.0.1:6379> zrem zset-key member1
(integer) 1
127.0.0.1:6379> zrange zset-key 0 -1
1) "member2"
2) "member3"
127.0.0.1:6379> zrange zset-key 0 -1 withscores
1) "member2"
2) "100"
3) "member3"
4) "400"
```

> 有序集合的键被称为成员（number），每个成员的值则被称为分值（score），需要根据分值来进行排

