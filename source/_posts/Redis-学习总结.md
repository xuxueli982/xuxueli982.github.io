---
title: Redis 学习总结
date: 2021-01-18 13:53:14
tags: redis
categories: middleware
---

## 概述

### 什么是Redis

&emsp; Redis是**Re**mote **Di**ctionary **S**erver(远程数据服务)的缩写，由意大利人antirez(Salvatore Sanfilippo)开发的一款**内存高速缓存数据库**。该软件由C语言编写，他的数据模型为key-value，它支持丰富的数据结构（类型），比如：String、list、hash、set、sorted、set。可持久化，保证了数据安全。 

使用缓存减轻数据库的负载。

&emsp; 在开发网站的时候如果有一些数据在短时间之内不会发生变化，而它们还要被频繁访问，为了提高用户的请求**速度**和**降低网站的负载**，就把这些数据放到一个读取速度更快的介质上（或者是通过较少的计算量就可以获得该数据），该行为就称为对该数据的缓存。该**介质**可以是文件、数据库、内存，内存经常用于数据缓存。

缓存的两种形式：

- 页面缓存：经常用在CMS(content manage system)内存管理系统里面。
- 数据缓存：经常会用在页面的具体数据里面。

### Redis 和memcache比较

1. Redis不仅支持简单的k-v类型的数据，同时还提供list、set、zset、hash等数据结构的存储。
2. Redis支持master-slave（主从一致）模式应用。
3. Redis支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用。
4. Redis单个value的最大限制是1GB，memcached只能保存1MB的数据。

### 安装Redis

1. redis启动脚本的细节

	直接使用`./redis-server`方式启动使用的redis-server这个shell脚本中默认配置。

2. 如何在启动redis时指定配置文件启动

	默认在redis安装完成之后在安装目录没有任何配置文件，需要到源码目录中复制redis.conf配置文件到安装目录下

3. 修改redis默认的端口号

redis-cli：终端操作脚本。

redis-server：启动redis服务脚本文件。

redis-benchmark：压力测试文件。

redis-check-aof和redis-check-dump：检测备份文件脚本。

### redis中库的概念

**库**

&emsp;database是用来存放数据的一个基本单元，一个库可以存放key-value键值对 redis中每一个库都有一个唯一名称（编号）从0开始，默认库的个数：16个库。库的编号：0-15，默认使用的是0号库。

```shell
# Set the number of databases. The default database is DB 0, you can select
# a different one on a per-connection basis using SELECT <dbid> where
# dbid is a number between 0 and 'databases'-1
databases 16
```

切换库的命令：SELECT <dbid>(库编号)

redis中清除库的指令：

- flushDB	清空当前库
- flushAll     清空所有库

## Redis数据库相关指令

### 数据库操作指令

```shell
# 1.Redis中库说明
- 使用redis的默认配置启动redis服务后，默认会存在16个库，编号从0-15
- 可以使用select库的编号来选择一个redis的库

# 2.Redis中操作库的指令
- 清空当前的库	FLUSHDB
- 清空全部的库	FLUSHALL

# 3. redis客户端显示中文
- ./redis-cli	-p	6379	--raw
```

### 操作key相关指令

在redis中，除了`\n`和空格不能作为名字的组成内容外，其他内容都可以作为key的名字部分，名字长度不做要求。

|   指令    |                  语法                  |                             作用                             | 可用版本 |                            返回值                            | 其他                                                         |
| :-------: | :------------------------------------: | :----------------------------------------------------------: | :------: | :----------------------------------------------------------: | ------------------------------------------------------------ |
|    del    |           `del key[key ...]`           |         删除给定的一个或多个key。不存在key会被忽略。         | >=1.0.0  |                       被删除key的数量                        |                                                              |
|  exists   |              `exists key`              |                     检查给定key是否存在                      | >=1.0.0  |                若key存在，返回1，否则返回0。                 |                                                              |
|  expire   |          `expire key seconds`          | 为给定key设置生存时间，当key过期时（生存时间为0),它会被自动删除。 | >=1.0.0  |                       设置成功返回1。                        | 时间复杂度：O(1)                                             |
|   keys    |             `keys pattern`             |              查找所有符合给定模式patter的key。               | >=1.0.0  |                   符合给定模式的key列表。                    | `keys *` 匹配数据库中所有key。`keys h?llo` 匹配`hello,hallo,hxllo`等。`keys h*llo`, 匹配`hllo`和`heeeello`等。`keys h[ae]llo`, 匹配`hello`和`hallo`，但不匹配`hillo`，特殊符号用`\`隔开。 |
|   move    |             `move key db`              |          将当前数据库的key移动到给定的数据库DB中。           | >=1.0.0  |                 移动成功返回1，失败则返回0。                 |                                                              |
|  pexpire  |       `pexpire key milliseconds`       | 这个命令和`expire`命令的作用类似，但是它以毫秒为单位设置key的生存时间，而不像`expire`命令那样，以秒为单位。 | >=2.6.0  |          设置成功返回1，key不存在或设置失败返回0。           | 时间复杂度：o(1)                                             |
| pexpireat | `pexpireat key milliseconds-timestamp` | 这个命令和`expire`命令类似，但它以毫秒为单位设置key的过期unix时间戳，而不是像`expire`那样，以秒为单位。 | >=2.6.0  | 如果生存时间设置成功，返回1。当key不存在或没办法设置生存时间时，返回0。（查看`expire`命令获取更多信息） |                                                              |
|    ttl    |               `ttl key`                | 以秒为单位，返回给定key的剩余生存时间（TTL，time to live)。  | >=1.0.0  | 当key不存在时，返回-2。当key存在但没有设置剩余生存时间是，返回-1。否则，以秒为单位，返回key的剩余生存时间。 | 在Redis 2.8 以前，当key不存在，或者key没有设置剩余生存时间时，命令都返回-1。 |
|  `pttl`   |               `pttl key`               | 这个命令类似于`ttl`命令，但它以毫秒为单位返回key的剩余生存时间而不是像`ttl`命令那样，以秒为单位。 | >=2.6.0  | 当key不存在时，返回-2，当key存在但没有设置剩余生存时间返回-1，否则以毫秒为单位，返回key的剩余生存时间。 | 在Redis 2.8以前,当key不存在，或者key没有设置剩余生存时间时，命令都返回-1。 |
| randomkey |              `randomkey`               |          从当前数据库中随机返回（不删除）一个key。           | >=1.0.0  |   当数据库不为空时，返回一个key。当数据库为空时，返回nil。   |                                                              |
|  rename   |          `rename key newkey`           | 将key改名为newkey，当key和newkey相同，或者key不存在时，返回一个错误。当newkey已经存在时，rename命令将	覆盖旧值。 | >=1.0.0  |                改名成功提示OK，失败时返回错误                |                                                              |
|   type    |               `type key`               |                  返回key所存储的值的类型。                   | >= 1.0.0 | none(key不存在)	String(字符串)，List(列表)Set(集合)	ZSet(有序集)	Hash(哈希表) |                                                              |

### String 类型操作

> `String`是`redis`最基本的类型，`redis`的`String`可以包含任何数据。包括jpg或者序列化的都对象。单个value值最大上限是1GB。  

#### 内存存储模型

| KEY  | VALUE |
| :--: | :---: |
| name | 张三  |
| age  |  20   |

#### 常用操作命令

| 命令                                         | 说明                                       |
| -------------------------------------------- | ------------------------------------------ |
| set                                          | 设置一个key/value                          |
| get                                          | 根据key获得对应的value                     |
| mset                                         | 一次设置多个key /value                     |
| mget                                         | 一次获得多个key的value                     |
| getset                                       | 获得原始key的值，同时设置新值              |
| strlen                                       | 获得对应key存储value的长度                 |
| append                                       | 为对应key的value追加内容                   |
| getrange索引0开始                            | 截取value内容                              |
| setex                                        | 设置一个key存活的有效期（秒）              |
| psetex                                       | 设置一个key存活的的有效期（毫秒）          |
| setnx                                        | 存在不做任何操作，不存在添加               |
| msetnx原子操作（只能有一个存在不做任何操作） | 可以同时设置多个key,只要有一个存在都不保存 |
| decr                                         | 进行数值类型的-1操作                       |
| decrby                                       | 根据提供的数据进行减法操作                 |
| incr                                         | 进行数值类型的+1操作                       |
| incrby                                       | 根据提供的数据类型进行加法操作             |
| incrbyfloat                                  | 根据提供的数据加入浮点数                   |

###  List类型

&emsp; list列表相当于Java中list集合特点：元素有序且可以重复

#### 内存存储模型

![image-20200929230846635](image-20200929230846635.png)

#### 常用操作指令

| 命令               | 说明                                   |
| ------------------ | -------------------------------------- |
| lpush              | 将某个值加入到一个key列表头部          |
| lpushx（x：exists) | 同lpush，但是必须保证这个key存在       |
| rpush              | 将某个值加入到一个key列表末尾          |
| rpushx             | 同rpush，但是必须保证这个key存在       |
| lpop               | 返回和移除列表的第一个元素             |
| rpop               | 返回和移除列表的第一个元素             |
| lrange 0 -1        | 获取某一个下标区间内的元素             |
| llen               | 获取列表元素个数                       |
| lset               | 设置某一个指定索引的值（索引必须存在） |
| lindex             | 获取某一指定索引位置的元素             |
| lrem               | 删除重复元素                           |
| ltrim              | 保留列表中特定区间内的元素             |
| linsert            | 在某一个元素之前，之后插入新元素       |

### Set类型

&emsp; 特点：set类型	set集合 元素无序 不可以重复

#### 内存存储模型

![image-20200929234000826](image-20200929234000826.png)

#### 常用命令

| 常用命令    | 说明                                     |
| ----------- | ---------------------------------------- |
| sadd        | 为集合添加元素                           |
| smembers    | 显示集合中所有元素（无序）               |
| scard       | 返回集合中元素的个数                     |
| spop        | 随机返回一个元素并将元素在集合中删除     |
| smove       | 从一个集合中向另一个集合移动元素         |
| srem        | 从集合中删除一个元素                     |
| sismember   | 判断一个集合中是否含有这个元素           |
| srandmember | 随机返回元素                             |
| sdiff       | 去掉第一个集合中其它集合中含有的相同元素 |
| sinter      | 求交集                                   |
| sunion      | 求合集                                   |

### ZSet类型

&emsp; 特点：可排序的set集合，排序，不可重复

&emsp; ZSET官方，可排序SET，sortSet

#### 内存模型

![image-20200930000218342](image-20200930000218342.png)

#### 常用命令

| 命令                    | 说明                         |
| ----------------------- | ---------------------------- |
| zadd                    | 添加一个有序集合元素         |
| zcard                   | 返回集合的元素个数           |
| zrange升序zrevrange降序 | 返回一个范围内的元素         |
| zrangebyscore           | 按照分数查找一个范围内的元素 |
| zrank                   | 返回排名                     |
| zrevrank                | 倒序排名                     |
| zscore                  | 显示某一个元素的分数         |
| zrem                    | 移除某一个元素               |
| zincrby                 | 给某个特定元素加分           |

### Hash类型

&emsp; 特点：value是一个map结构 存在key-value	key无序的

#### 内存模型

![image-20200930001919773](image-20200930001919773.png)

#### 常用命令

| 命令         | 说明                   |
| ------------ | ---------------------- |
| hset         | 设置一个key/value对    |
| hget         | 获得一个key对应的value |
| hgetall      | 获得所有的key/value对  |
| hdel         | 删除某一个key/value对  |
| hexists      | 判断一个key是否存在    |
| hkeys        | 获得所有的key          |
| hvals        | 获得所有的value        |
| hmset        | 设置多个key/value      |
| hmget        | 获得多个key的value     |
| hsetnx       | 设置一个不存在的key值  |
| hincrby      | 为value进行加法运算    |
| hincrbyfloat | 为value加入浮点值      |

 开启redis远程连接

&emsp;注意：默认redis服务器是没有开启远程连接，也就是默认拒绝所有远程客户端连接。

- 修改配置开启远程连接

```shell
vim redis.conf #修改如下配置
bind 0.0.0.0 #允许一切客户端连接
```

- 修改配置之后重启redis服务

```shell
./redis-server ../redis.conf #注意：一定要加载配置文件启动。
```

## 持久化机制

client redis(内存)---------->内存数据------------->数据持久化----------->磁盘

Redis官方提供了两种不同的持久化方法来将数据存储到硬盘里面。

- 快照（`snapshot`）
- AOF(`Append Only File`)只追加日志文件

### 快照：Snapshot

#### 特点：

&emsp; 这种方式可以将某一时刻的所有数据都写入硬盘中，当然这也是**redis默认的开启持久化方式**，保存的文件是以`.rdb`形式结尾的文件因此这种方式也称之为`RDB`方式。

![image-20201005212101824](image-20201005212101824.png)

#### 快照的生成方式

**1.客户端方式之：BGSAVE**

&emsp;客户端可以使用`BGSAVE`指令来创建一个快照，当接收到客户端的`BGSAVE`命令时，`redis`会调用`fork`来创建一个子进程，然后子进程负责将快照写入磁盘中，而父进程则继续处理命令请求。

&emsp;名词解释`fork`：当一个进程创建子进程的时候，底层的操作系统会创建该进程的一个副本，在类unix系统中创建子进程的操作会进行优化，在刚开始的时候，父进程共享相同内容，直到父进程或子进程进行了写之后，对被写入的内存的共享才会结束服务。

![image-20201005214834920](image-20201005214834920.png)

**2. 客户端：SAVE**

&emsp;客户端还可以使用SAVE命令来创建一个快照，接收到SAVE命令的redis服务器在快照创建完毕之前将不再响应任何其他命令

![image-20201005215219398](image-20201005215219398.png)

- 注意：SAVE命令并不常用，使用SAVE命令在快照创建完毕之前，redis处于阻塞状态，无法对外提供服务。

**3. 服务器配置方式满足配置自动 触发**

&emsp;如果用户在`redis.conf`中设置了`save`配置选项，`redis`会在`save`选项条件满足之后自动触发一次`BGSAVE`命令，如果设置了多个`save`配置选项，当任意一个`save`配置选项条件满足，`redis`也会触发一次`BGSAVE`。

![image-20201011230449509](image-20201011230449509.png)

**4. 服务器接收客户端shutdown指令**

&emsp; 当`redis`通过`shutdown`指令接收到关闭服务器的请求时，会执行一个`save`命令，阻塞所有的客户端，不再执行客户端发送的任何命令，并且在`save`命令执行完毕之后关闭服务器。

#### 配置生成快照名称和位置

```shell
# 1.修改生成快照名称
dbfilename dump.rdb

# 2.修改生成位置
dir ./
```

### AOF只追加日志文件

#### 特点

&emsp; 这种方式可以将所有客户端执行的写命令记录到日志文件中，AOF持久化会将被执行的写命令写道AOF的文件末尾，以此来记录数据发生的变化，因此只要redis从头到尾执行一次AOF文件包含的所有写命令，就可以恢复AOF文件记录的数据集。

![image-20201011212058673](image-20201011212058673.png)

#### 开启AOF持久化

&emsp; 在`redis`的默认配置中`AOF`持久化机制是没有开启的，需要在配置中开启。

开启AOF持久化

1. 修改`appendonly	yes	`开启持久化
2. 修改`appendfilename    "appendonly.aof"    `指定生成文件名称。
3. 默认appendonly.aof是存储在与快照中指定的`dir ./`相同的目录下的。

![image-20201011222233735](image-20201011222233735.png)

#### 日志追加频率

**always [谨慎使用]**

- 说明：每个redis写命令都要同步写入硬盘，严重降低redis速度。
- 解释：如果用户使用always选项，那么每个redis写命令的都会被写入硬盘，从而将发生系统崩溃时出现的数据丢失减到最小；遗憾的是：因为这种同步策略需要对硬盘进行大量的写入操作，所以redis处理命令的速度会受到硬盘性能的限制；
- ‼注意：转盘式硬盘在这种频率下200左右个命令/s，固态硬盘（SSD）几百万个命令/s。
- ⚠警告：使用SSD用户请谨慎使用always选项，这种模式不断写入少量数据的做法有可能会引发严重的写入放大问题，导致将固态硬盘的寿命从原来的几年降低为几个月。

**everysec [推荐]**

- 说明：每秒执行一次同步，显示的将多个写命令同步到磁盘。
- 解释：为了兼顾数据安全和写入性能，用户可以考虑使用everysec选项，让redis每秒一次的频率对AOF文件进行同步；redis每秒同步一次AOF文件时性能和不使用任何持久化特性时的性能相差无几，而通过每秒钟同步一次AOF文件，redis可以保证，即使系统崩溃时，用户最多丢失一秒内产生的数据。

**no [不推荐]**

- 说明：由操作系统决定何时同步
- 解释：最后使用no选项，将完全由操作系统决定是什么时候同步AOF日志文件，这个选项不会对redis性能带来影响，但是系统崩溃时，会丢失不定量的数据；另外如果用户硬盘处理写入操作不够快的话，当缓冲区被等待写入硬盘数据填满时，redis会处于阻塞状态，并导致redis的处理命令请求的速度变慢。

### AOF文件的重写

#### AOF带来的问题

&emsp; AOF大方式也同时带来了另一个问题，持久化文件会变得越来越大。例如我们调用incr test命令100次，文件中就必须保存全部的100条命令，其实有99条都是多余的，因为要恢复数据库的状态其实文件中保存一条`set test 100`就够了，为了压缩aof的持久化文件Redis提供了AOF重写（ReWriter)机制。

#### AOF重写

> 用来在一定程度上减少AOF文件的体积。

#### 触发重写方式

**客户端方式触发重写**

&emsp; 执行`BGREWRITEAOF`命令，不会阻塞redis服务

**服务器配置方式自动触发**

- 配置`redis.conf`中的`auto-aof-rewrite-percentage`选项，参见下图↓↓↓

![image-20201011222116791](image-20201011222116791.png)

- 如果设置`auto-aof-rewrite-percentage`值为100和`auto-aof-rewrite-min-size 64mb`,并且启用的AOF持久化机制，那么当AOF文件体积大于64M，并且AOF文件体积比上一次重写之后体积大了至少一倍（100%)时，会自动触发，如果重写过于频繁，用户可以考虑将`auto-aof-rewrite-percentage`设置为更大。

#### 重写原理

&emsp;==注意：重写aof文件的操作，并没有读取旧的aof文件，而是将整个内存中的数据库内容用命令的方式重写一个新的aof文件，替换原有的文件这点和快照有点类似。==

**重写流程**

1. `redis`调用`fork`，现在有父子两个进程，子进程根据内存中的数据快照，往临时文件中写入重建数据库状态的命令。
2. 父进程继续处理client对象，除了把写命令写入到原来的`aof`文件中，同时把接收到的写命令缓存起来，这样就能保证如果子进程重写失败的话并不会出现问题。
3. 当子进程把快照内容以命令方式写入到临时文件中之后，子进程发信号通知父进程。然后父进程把缓存的写命令也写入到临时文件中。**思考一个问题：如果父进程开始将缓存的写命令也写入到临时文件时这时候父进程会不会阻塞，即此时不响应客户端的写入请求。**
4. 现在父进程可以使用临时文件替换老的`aof`文件，并重写命名，后面收到的写命令也开始往新的`aof`文件中追加。

****

![image-20201011224016694](image-20201011224016694.png)

 #### 持久化总结

&emsp; 两种持久化方案既可以同时使用(`aof`),也可以单独使用，在某种情况下也可以都不使用。具体使用哪种持久化方案取决于用户的数据和应用决定。

&emsp; 无论使用AOF还是使用快照机制持久化，将数据持久化到硬盘都是必要的。除了持久化外，用户还应该对持久化的文件进行备份（最好备份在多个不同的地方）。

## Java操作Redis

### 环境准备

**1. 引入依赖**

```xml
<!--引入jedis连接依赖-->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
```

**2. 创建Jedis对象**

```java
//创建Jedis对象
Jedis jedis = new Jedis("192.168.25.150","6379");
jedis.select(0);
//执行相关操作
/*
*/
//释放连接
jedis.close();
```

### 操 作key相关API



### 操作String相关API

### 操作List相关API

### 操作Set相关API

### 操作Hash相关API

## SpringBoot整合Redis

&emsp; Spring Boot Data（数据）Redis中提供了**RedisTemplate和StringRedisTemplate**,其中StringRedisTemplate是RedisTemplate的子类，两个方法基本一致。不同之处主要体现在操作的数据类型不同，**RedisTemplate中的两个泛型都是Object，意味着存储`key`和`value`都可以是一个对象，而StringRedisTemplate的两个泛型都是String，意味着StringRedisTemplate的`key`和`value`都只能是字符串。**

&emsp;**注意：使用RedsiTemplate默认是将对象序列化到Redis中，所以放入的对象必须实现对象序列化接口。**

### 环境准备

 **1.引入依赖**

```xml
<!--引入依赖 spring data redis 依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

## Redis主从复制

### 主从复制

主从复制架构仅仅用来解决数据冗余备份，从节点仅仅用来同步数据。

> 主从架构存在的问题：
>
> - 无法解决主节点宕机后的自动故障转移问题。

### 主从复制架构图

![image-20201022221516243](image-20201022221516243.png)

### 搭建主从复制

**准备3台机器并修改配置**

> - `master`
> 	- port 6379
> 	- bind 0.0.0.0
> - `slave1`
> 	- port 6380
> 	- bind 0.0.0.0
> 	- slaveof <masterip> <masterport>
> - `slave2`
> 	- port 6381
> 	- bind 0.0.0.0
> 	- slaveof <masterip> <masterport>

## Redis哨兵机制

### 哨兵Sentine机制

&emsp;Sentine(哨兵)是Redis的高可用性解决方案：由一个或多个Sentinel实例组成的Sentinel系统可以监视任意多个主服务器，以及这些主服务器下属的所有从服务器，并在被监视的主服务器进入下线状态时，自动将下线主服务器下属的某个从服务器升级为新的主服务器。简单的说哨兵就是带有自动故障转移功能的主从架构。

### 哨兵架构原理

![image-20201022224855604](image-20201022224855604.png)

### 哨兵模式的搭建

**1. 主节点创建哨兵配置**

> 再Master对应的redis.conf同目录下新建sentinel.conf文件，名字绝对不能错。

**2. 配置哨兵，再sentinel.conf文件中填入内容**

> sentinel monitor 被监视的主从架构别名（自己定义）ip port 1
>
> - 说明：数字1代表有一个及以上的sentinel服务检测到master宕机，才会执行主从切换的功能。

**3.	启动哨兵模式进行测试**

```sh
./redis-sentinel /root/sentinel/sentinel.conf
```

> 哨兵架构存在的问题
>
> - 单节点的并发压力问题
>
> - 单节点的内存和磁盘的物理上限问题

