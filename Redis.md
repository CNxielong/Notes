## Redis由来
[菜鸟教程](https://www.runoob.com/redis/redis-sorted-sets.html)：https://www.runoob.com/redis/redis-sorted-sets.html

### 1、定义

 - REmote DIctionary Server(Redis) 是一个由Salvatore
   Sanfilippo写的key-value存储系统。 Redis是一个开源的使用ANSI
   C语言编写、遵守BSD协议、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。
 - 它通常被称为数据结构服务器，因为值（value）可以是 字符串(String), 哈希(Hash), 列表(list), 集合(sets)  和 有序集合(sorted sets)等类型。
    ![中间件架构](https://img-blog.csdnimg.cn/20210121221549607.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpZWxvbmcwNTA5,size_16,color_FFFFFF,t_70)

####  1.1 缓存中间件 Memcache和Redis

   memcache:
  ![memcache](https://img-blog.csdnimg.cn/20210121224102831.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpZWxvbmcwNTA5,size_16,color_FFFFFF,t_70)
  Redis:
  理论值：10W+QPS
  ![Redis](https://img-blog.csdnimg.cn/20210121224438647.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpZWxvbmcwNTA5,size_16,color_FFFFFF,t_70)
  ![Redis为啥快](https://img-blog.csdnimg.cn/20210121224824546.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpZWxvbmcwNTA5,size_16,color_FFFFFF,t_70)

#### 1.2 IO

I/O多路复用：提高redis多路复用。

![image-20210121224638199](C:\Users\X-Dragon\AppData\Roaming\Typora\typora-user-images\image-20210121224638199.png)

- selector:监听可以读写的文件，并返回这些文件描述符，就可以读写文件了 。

![image-20210121225218954](C:\Users\X-Dragon\AppData\Roaming\Typora\typora-user-images\image-20210121225218954.png)

![image-20210121225426459](C:\Users\X-Dragon\AppData\Roaming\Typora\typora-user-images\image-20210121225426459.png)

- 更好的I/O

![image-20210121225500735](C:\Users\X-Dragon\AppData\Roaming\Typora\typora-user-images\image-20210121225500735.png)

### 2、数据类型

- Redis支持五种数据类型：string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)。

 1、语法 set name "value"

```java
set name "redis"
set name "memcache"
get name (返回"memcache")  
    
```

#### 2.1String

```
redis 127.0.0.1:6379> SET runoobkey redis
OK
redis 127.0.0.1:6379> GET runoobkey
"redis"
```

#### 2.2 Hash

- Redis hash 是一个 string 类型的 field（字段） 和 value（值） 的映射表，hash 特别适合用于存储对象。

```
127.0.0.1:6379>  HMSET runoobkey name "redis tutorial" description "redis basic commands for caching" likes 20 visitors 23000
OK
127.0.0.1:6379>  HGETALL runoobkey
1) "name"
2) "redis tutorial"
3) "description"
4) "redis basic commands for caching"
5) "likes"
6) "20"
7) "visitors"
8) "23000"
```

#### 2.3 List：按插入顺序

- Redis列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）

- 一个列表最多可以包含 2^32 - 1 个元素 (4294967295, 每个列表超过40亿个元素)。

  ```
  redis 127.0.0.1:6379> LPUSH runoobkey redis
  (integer) 1
  redis 127.0.0.1:6379> LPUSH runoobkey mongodb
  (integer) 2
  redis 127.0.0.1:6379> LPUSH runoobkey mysql
  (integer) 3
  redis 127.0.0.1:6379> LRANGE runoobkey 0 10
  
  1) "mysql"
  2) "mongodb"
  3) "redis
  ```

#### 2.4 Set:无序、可以取交集、并集

- Redis 的 Set 是 String 类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据。

- Redis 中集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。

- 集合中最大的成员数为 2^32 - 1 (4294967295, 每个集合可存储40多亿个成员)。

```
redis 127.0.0.1:6379> SADD runoobkey redis
(integer) 1
redis 127.0.0.1:6379> SADD runoobkey mongodb
(integer) 1
redis 127.0.0.1:6379> SADD runoobkey mysql
(integer) 1
redis 127.0.0.1:6379> SADD runoobkey mysql
(integer) 0
redis 127.0.0.1:6379> SMEMBERS runoobkey

1) "mysql"
2) "mongodb"
3) "redis"
```

#### 2.5 有序集合(sorted set) 分数越大顺序越靠后

 - Redis 有序集合和集合一样也是string类型元素的集合,且不允许重复的成员。
不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。
 - 有序集合的成员是唯一的,但分数(score)却可以重复，分数重复按照插入顺序先后排先后顺序。
 - 集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。 集合中最大的成员数为 2^32 - 1 (4294967295, 每个集合可存储40多亿个成员)。

#### 2.6 HyperLogLog和Geo、Stream

- HyperLogLog:方便计数
    - **基数：**

    - 比如数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8}, 基数(不重复元素)为5。 基数估计就是在误差可接受的范围内，快速计算基数。

- Geo:存储地理位置(geography)。

- Stream:Redis Stream 主要用于消息队列（MQ，Message Queue）, Redis Stream 提供了消息的持久化和主备复制功能。

#### 2.7 底层数据原理：待补充

![image-20210122002246979](C:\Users\X-Dragon\AppData\Roaming\Typora\typora-user-images\image-20210122002246979.png)

### 3、KEY应用 实际场景

启动redis:

```
$ redis-cli -h host -p port -a password
```

设置过期时间：

```
EXPIRE key seconds
```

[命令：]https://www.runoob.com/redis/redis-keys.html

#### 3.1 从海量数据查询具有特定前缀的Key

思路：1、确定数据量（边界）dbsize命令：返回数据量大小

​		   2、数据量小 可以用Keys指令: 语法：Keys [pattern]

​		   3、数据量大 可以用 :SCAN cursor [MATCH pattern\] [COUNT count]

 问题：Keys：会返回符合条件的所有数据、CPU，I/O压力大、耗时，卡顿。影响线上业务。

​			SCAN：根据下标和分页查询，比较快。但是数据可能有重复，返回值不一定对应count

返回K1开头的所有Redis: KEYS命令

![返回K1开头的所有Redis](C:\Users\X-Dragon\AppData\Roaming\Typora\typora-user-images\image-20210122003507459.png)

返回K1开头的所有Redis：SCAN （count 10 可能返回0-10个）

![image-20210122004332032](C:\Users\X-Dragon\AppData\Roaming\Typora\typora-user-images\image-20210122004332032.png)

JAVA实现：

 定义一个循环，不断调用SCAN指令。直到下标+分页大于 总数。分页取出来的信息，用集合SET去重。

### 4、实现分布式锁

