# Redis基本数据类型

## String

- `set key value` 

- `setNX key value` key不存在时才设置
- `get key`
- `mset k1 v1 k2 v2`  设置一个或多个值
- `mget k1 k2`
- `strlen key`  返回所存储字符串值的长度
- `incr key` 将key中存储的==数字值==加一
- `decr key`

- `exists key` 判断是否存在
- `del key`
- `expire key seconds` 设置过期时间(默认不过期)
  - `ttl key` 查看过期时间

### 应用场景

- 需要存储常规数据的场景
  - 缓存session、token、图片地址、序列化后的对象(==相比较于Hash更节省内存==）
- 需要计数的场景
  - 用户单位时间的请求数(简单限流)、页面单位时间的访问数
- 分布式锁
  - setNX key value 实现简易的分布式锁



## List

- `rpush k v1 v2 v3` 在指定列表的尾部添加若干元素
- `lpush k v1 v2 v3`  在指定列表的头部添加若干元素
- `lset k index v`      将index位置的值设置为v
- `rpop key`                   移除并获取最有元素
- `rpop key`
- `llen key`                    获取元素数量
- `lrange key start end` 获取start-end间的元素(0 -1)

### 应用场景

信息流展示

- 最新文章、动态



## Hash

|             命令              |              介绍              |
| :---------------------------: | :----------------------------: |
|    `hset key filed vlaue`     |    设置哈希表中指定字段的值    |
|   `hsetnx key filed value`    |          不存在时设定          |
|    `hmset key f1 v1 f2 v2`    |         设置若干键值对         |
|       `hget key field`        |        获取指定字段的值        |
|       `hmget key f1 f2`       |            获取多个            |
|         `hgetall key`         |            获取所有            |
|      `hexists key field`      |     查看对应field是否存在      |
|       `Hdel key f1 f2`        |          删除若干字段          |
|          `hlen key`           |          查看字段数量          |
| `hincrby key field increment` | 对哈希表中的指定字段做运算操作 |
|           `del key`           |         删除指定hash表         |

### 应用场景

- 用户信息、商品信息、文章信息、购物车信息等



## Set

- 无序集合，类似java中的hashSet
- 提供了`判断一个元素是否在Set集合内的重要接口`，是List没有的
- 实现求交集、并集、差集的操作(查找共同关注、粉丝等)

|              命令               | 介绍                                        |
| :-----------------------------: | :------------------------------------------ |
|   `sadd key member1 member 2`   | 添加若干元素                                |
|         `smembers key`          | 获取指定集合的所有元素                      |
|           `scard key`           | 获取指定集合的元素数量                      |
|     `sismember key member`      | 判断指定元素是否在集合中                    |
|         `sinter k1 k2`          | 获取给定若干集合的交集                      |
| `sinterstore destination k1 k2` | 获取给定若干集合的交集并存储在destination中 |
|         `sunion k1 k2`          | 获取给定若干集合的并集                      |
| `sunionstore destination k1 k2` | 获取给定若干集合的并集并存储在destination中 |
|          `sdiff k1 k2`          | 获取给定若干集合的差集(显示前一个的不同项)  |
| `sdiffstore destination k1 k2`  | 获取给定若干集合的差集并存储在destination中 |
|        `spop key count`         | 随机移除并获取指定集合中的若干元素          |
|     `srandmember key count`     | 随机获取指定集合中的指定数量的元素          |
|         `srem k value`          | 删除指定元素                                |

### 应用场景

需要存放的`数据不能重复`的场景

- 网站uv统计(数据量巨大的场景`HyperLogLog`更合适)、文章点赞、动态点赞等

![截屏2023-03-03 下午3.27.31](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-03-03 下午3.27.31.png)

- 需要或者多个数据源`交集、并集、差集`的场景
  - ==共同好友、粉丝、关注(交集)==
  - ==好友推荐、音乐推荐(差集)==
  - ==订阅号推荐(差集+交集)==

![截屏2023-03-03 下午3.29.31](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-03-03 下午3.29.31.png)

- 需要随机获取数据源中的元素的场景
  - 抽奖系统、随机
  - `spop`适用于不可重复中奖
  - `srandmember`适用于可重复中奖

## Sorted Set

相对于`Set`，增加了一个权重参数`score`，使得集合中的元素能够按`score`进行有序排列，还可以通过`score`的范围来获取元素的列表，类似于HashMap与TreeSet的结合体。

|                   命令                   | 介绍                                                        |
| :--------------------------------------: | :---------------------------------------------------------- |
| `zadd key score1 member1 score2 member2` | 向指定有序集合中添加若干元素                                |
|               `zcard key`                | 获取指定有序集合的元素数量                                  |
|           `zscore key member`            | 获取指定有序集合中指定元素的socre值                         |
|        `zinter numkeys k1 k2 ...`        | 将给定所有有序集合的sum值进行sum聚合操作，numkeys为集合数量 |
|                 `zunion`                 | 同set                                                       |
|                 `zstore`                 | 同set                                                       |
|          `zrange key start end`          | 获取指定有序集合start-end之间的元素(升序)                   |
|        `zrevrange key start end`         | 获取指定有序集合start-end之间的元素(降序)                   |
|          `zrevrank key member`           | 获取指定有序集合中指定元素的排名                            |

### 应用场景

- 需要随机获取数据源中的元素根据某个权重进行排序的场景
  - 排行榜(直播间送礼、微信步数、王者段位)

- 需要存储的数据有优先级或者重要程度的场景
  - 优先级任务队列





# Redis特殊数据结构

## 1.Bitmap

Bitmap 存储的是连续的二进制数字（0 和 1），通过 Bitmap, 只需要一个 bit 位来表示某个元素对应的值或者状态，key 就是对应元素本身 。我们知道 8 个 bit 可以组成一个 byte，所以 Bitmap 本身会极大的节省储存空间。

![截屏2023-03-04 下午4.23.24](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-03-04 下午4.23.24.png)

### 应用场景

需要保存状态信息(0/1)的场景

- 用户签到情况
- 活跃用户情况
- 用户行为统计(是否点赞过某个视频)



## 2.HyberLogLog

HyperLogLog 是一种有名的基数计数概率算法 ，基于 LogLog Counting(LLC)优化改进得来，并不是 Redis 特有的，Redis 只是实现了这个算法并提供了一些开箱即用的 API。

Redis 提供的 HyperLogLog 占用空间非常非常小，只需要 12k 的空间就能存储接近`2^64`个不同元素。这是真的厉害，这就是数学的魅力么！并且，Redis 对 HyperLogLog 的存储结构做了优化，采用两种方式计数：

- **稀疏矩阵** ：计数较少的时候，占用空间很小。
- **稠密矩阵** ：计数达到某个阈值的时候，占用 12k 的空间。

基数计数概率算法为了节省内存并不会直接存储元数据，而是通过一定的概率统计方法预估基数值（集合中包含元素的个数）。因此， HyperLogLog 的计数结果并不是一个精确值，存在一定的误差（标准误差为 `0.81%` ）。

### 应用场景

数量量巨大（百万、千万级别以上）的计数场景

- 热门网站每日/周/月访问ip数统计
- 热门帖子uv统计



## 3.Geospatial index(地理空间)

Geospatial index（地理空间索引，简称 GEO） 主要用于存储地理位置信息，基于 Sorted Set 实现。

通过 GEO 我们可以轻松实现两个位置距离的计算、获取指定位置附近的元素等功能。

### 应用场景

- 附近的人





# 缓存读写策略

## 1.Cache Aside Pattern(旁路缓存模式)

### 适用场景

- **读请求多**



### 读写

- 读

  - ==从cache中读取数据，读取到就直接返回==

  - ==cache中读不到数据，从数据库中读取返回==

  - ==将数据存入cache==

![截屏2023-03-04 下午3.31.47](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-03-04 下午3.31.47.png)

- 写
  - ==更新db==
  - ==直接删除cache==

![截屏2023-03-04 下午3.26.48](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-03-04 下午3.26.48.png)



1. Q: 在写数据的过程中，可以先删除cache，后更新db吗？

​	A: 请求1删除cache中的数据A，请求2从db中读取数据，请求1更新db中的数据，导致了数据的不一致。

  2.Q: 在写数据的过程中，先更新db，后删除cache就没有问题了吗？



### 缺陷

#### 1.首次请求的数据一定不在cache中

解决方法:**将热点数据提前放入`cache`中**



#### 2.写操作频繁会导致cache中的数据频繁被删除，影响缓存命中率

解决方法：

1、数据库和缓存数据强一致的场景:更新db时同样更新`cache`，使用分布式锁来保证`cache`的线程安全

2、短暂允许数据库和缓存数据不一致:更新db时同样更新`cache`，但是给`cache`加一个比较短的过期时间





## 2. Read/Write Through Pattern(读写穿透)

将`cache`视为主要数据存储，从中读取数据并将数据写入其中。**`cache`服务负责将此数据读取和写入db**，减轻应用程序的职责。

除了性能的影响，Redis中没有提供cache将数据写入db的功能，因而使用较少。

### 读写

- 读
  - ==从cache中读取数据，读取到就直接返回==
  - ==读取不到先从db加载到cache中==
  - ==cache返回==

![截屏2023-03-04 下午3.52.25](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-03-04 下午3.52.25.png)

- 写
  - ==先查cache，不存在直接更新db==
    - ==cache中存在，先更新cache，然后cache服务自己更新db(同步更新)==

![截屏2023-03-04 下午3.52.14](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-03-04 下午3.52.14.png)



### 缺陷

#### 1.首次请求数据一定不在cache

解决方法:**将热点数据提前放入`cache`中**





## 3.Write Behind Pattern(异步缓存写入)

Write Behind Pattern 和 Read/Write Through Pattern相似

- 相同点
  - 均是由`cache`服务来负责cache和db的读写。

- 不同点
  - 读写穿透==同步更新==cache和db
  - 异步缓存则==只更新缓存，不直接更新db，而改为异步批量方式更新db==

### 适用场景

- db写性能非常高，适用于==数据经常变化但对一致性要求不高的场景==，比如`浏览量与点赞量`、消息队列中的消息异步写入磁盘、Mysql中的innodb Buffer Pool机制





# 内存管理

## 过期数据的删除策略

1. 惰性删除：只会在取出key的时候才对数据进行检查。这样对cpu最友好，但是可能会造成太多过期的key没有被删除
2. 定期删除：每隔一段时间抽取一批key执行删除过期key操作。Redis底层会通过限制删除操作执行的市场和频率来减少删除操作对cpu时间的影响

定期删除对内存更加友好，惰性删除对CPU更加友好。通过这两种方式仍然会有漏掉的过期key，需要redis`内存淘汰机制`



## 内存淘汰机制

1. volatile-lru：从`已设置过期时间的数据集`（server.db[i].expires）中挑选最近最少使用的数据淘汰
2. volatile-ttl：从`已设置过期时间的数据集`（server.db[i].expires）中挑选将要过期的数据淘汰
3. volatile-random：从`已设置过期的数据集`中任意选择数据淘汰
4. allkeys-lru：当内存不足以容纳写入新数据时，在`键空间`中，移除最近最少使用的key(==最常用==)
5. allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰
6. no-eviction：禁止驱逐数据

4.0 版本后增加以下两种：

1. volatile-lfu：从`已设置过期时间的数据集`（server.db[i].expires）中挑选最不经常使用的数据淘汰
2. allkeys-lfu： 当内存不足以容纳写入新数据时，在`键空间`中，移除最不经常使用的key(==最常用==)



# Redis持久化机制

## 如何保证Redis挂掉之后再重启数据可以恢复？

一般来说持久化数据就是将内存中的数据写入到硬盘里，为了之后重用数据(重启机器、故障后恢复数据)，或者是为了防止系统故障而将数据备份到一个远程位置。

Redis支持两种持久化操作，`快照(snapshotting,RDB)`和`只追加文件(append-only file,AOF)`。

### 1.RDB持久化

Redis可以通过创建快照来获得存储在内存里面的数据在某个时间点上的副本。创建快照后，可以对快照进行备份，可以将快照复制到其他服务器从而创建具有相同数据的服务器副本(Redis主从结构提升性能)。还可以将快照留在原地以便重启服务器的时候使用。

- 是redis默认的持久化方式
- 适用于
  - ==快速备份和恢复数据==
  - ==数据库的数据量较大==
  - ==需要在特定时间点保存数据==
- 由于不是实时的，可能会丢失一定量的数据。

```bash
# redis.conf
save 900 1 #在900秒后，如果至少有1个key变化，自动触发bgsave命令创建快照

# save 300 10 #在300秒后，如果至少有10个key发生变化，自动触发bgsave命令创建快照

# save 60 10000 #在60秒后，如果至少有10000个key发生变化，自动触发bgsave命令创建快照
```

#### RDB持久化会阻塞主线程吗？

- save:主线程执行，会阻塞主线程
- bgsave:子线程执行，不会阻塞主线程(默认)



### 2.AOF持久化

AOF 是一种追加方式，将 Redis 所有的写操作追加到一个文件中。通过将写操作记录到 AOF 文件中，可以保证 Redis 数据库的完整性。可以通过配置定期执行 fsync() 将 AOF 文件同步到磁盘上，或者通过 BGREWRITEAOF 命令将 AOF 文件重写为更紧凑的格式来降低磁盘空间占用。

- AOF持久化的实时性更好
- 适用于
  - ==需要高可靠性的生产环境==
  - ==对数据完整性要求高==
  - ==可以容忍更高的磁盘空间和IO开销==

```bash
appendonly yes
```

开启AOF持久话后每执行一条会更改Redis中数据的命令，Redis就会将该命令写入到内存缓存server.aof_buf中，再根据appendfsync配置来决定合适将其同步到硬盘中的AOF文件。

在Redis中存在三种不同的AOF持久化方式

- appendfsync always      # 一有数据修改就写入AOF文件，严重降低Redis速度
- appendfsync everysec  # 每秒钟同步一次，显示将多个写命令同步到硬盘(`兼顾数据和写入性能`)
- appnedfsyc no               # 让操作系统决定何时同步



### 同时使用

根据实际需求，可以将二者同时使用。

```bash
save 900 1
appendonly yes
```



# Redis事务

- MULTI
- EXEC
- DISCARD
- WATCH

```bash
> MULTI
OK
> SET PROJECT "JavaGuide"
QUEUED
> GET PROJECT
QUEUED
> EXEC
1) OK
2) "JavaGuide"
```

`MULTI`命令后可以输入多个命令，Redis 不会立即执行这些命令，而是将它们放到队列，当调用了`EXEC`命令后，再执行所有的命令。

过程：

1. 开始事务（`MULTI`）；
2. 命令入队(批量操作 Redis 的命令，先进先出（FIFO）的顺序执行)；
3. 执行事务(`EXEC`)。

`discard`命令取消事务，清空事务队列中保存的所有命令

```bash
> MULTI
OK
> SET PROJECT "JavaGuide"
QUEUED
> GET PROJECT
QUEUED
> DISCARD
OK
```

`watch`命令监听指定的key，当调用`exec`命令执行事务时，如果一个被`watch`命令见识的key被`其他`客户端修改的话，整个事务都不会执行。

```bash
# 客户端 1
> SET PROJECT "RustGuide"
OK
> WATCH PROJECT
OK
> MULTI
OK
> SET PROJECT "JavaGuide"
QUEUED

# 客户端 2
# 在客户端 1 执行 EXEC 命令提交事务之前修改 PROJECT 的值
> SET PROJECT "GoGuide"

# 客户端 1
# 修改失败，因为 PROJECT 的值被客户端2修改了
> EXEC
(nil)
> GET PROJECT
"GoGuide"
```

如果`watch`与事务在同一session里，并且被`watch`监视的key被修改的操作发生在事务内部，则可以成功。

```bash
> SET PROJECT "JavaGuide"
OK
> WATCH PROJECT
OK
> MULTI
OK
> SET PROJECT "JavaGuide1"
QUEUED
> SET PROJECT "JavaGuide2"
QUEUED
> SET PROJECT "JavaGuide3"
QUEUED
> EXEC
1) OK
2) OK
3) OK
127.0.0.1:6379> GET PROJECT
"JavaGuide3"
```

事务外部修改`watch`监视的key

```java
> SET PROJECT "JavaGuide"
OK
> WATCH PROJECT
OK
> SET PROJECT "JavaGuide2"
OK
> MULTI
OK
> GET USER
QUEUED
> EXEC
(nil)
```



# Redis性能优化

## Redis bigkey

### 1.什么是bigkey

bigkey指的是一个key对应的value所占用的内存过大。大约 string 类型的 value 超过 10 kb，复合类型的 value 包含的元素超过 5000 个（对于复合类型的 value 来说，不一定包含的元素越多，占用的内存就越多）。



### 2.bigkey的危害

消耗更多的内存空间，bigkey 对性能也会有比较大的影响。



### 3.如何发现bigkey

#### 1) 使用redis自带的--bigkeys参数来寻找

```bash
redis-cli --bigkeys

# Scanning the entire keyspace to find biggest keys as well as
# average sizes per key type.  You can use -i 0.1 to sleep 0.1 sec
# per 100 SCAN commands (not usually needed).

[00.00%] Biggest string found so far '"ADMIN_hxy_f6b17d33a95c4405aa69566eef69e31a"' with 155 bytes
[00.00%] Biggest hash   found so far '"hs"' with 2 fields
[00.00%] Biggest list   found so far '"mylist"' with 4 items

-------- summary -------

Sampled 6 keys in the keyspace!
Total key length in bytes is 146 (avg len 24.33)

Biggest   list found '"mylist"' has 4 items
Biggest   hash found '"hs"' has 2 fields
Biggest string found '"ADMIN_hxy_f6b17d33a95c4405aa69566eef69e31a"' has 155 bytes

1 lists with 4 items (16.67% of keys, avg size 4.00)
1 hashs with 2 fields (16.67% of keys, avg size 2.00)
4 strings with 350 bytes (66.67% of keys, avg size 87.50)
0 streams with 0 entries (00.00% of keys, avg size 0.00)
0 sets with 0 members (00.00% of keys, avg size 0.00)
0 zsets with 0 members (00.00% of keys, avg size 0.00)
```

#### 2) 分析RDB文件



## 大量key集中过期

定期删除执行过程中，如果突然遇到大量过期 key 的话，客户端请求必须等待定期清理过期 key 任务线程执行完成，因为这个这个定期任务线程是在 Redis 主线程中执行的。这就导致客户端请求没办法被及时处理，响应速度会比较慢。

1. 给 key 设置随机过期时间。
2. 开启 lazy-free（惰性删除/延迟释放） 。



# Redis生产问题

## 缓存穿透

简单来说即是`大量请求的key是不合理的，根本不存在于缓存中，也不存在于数据库中`。导致这些请求直接到了数据库上，根本没有经过缓存这一层，对数据库造成了巨大的压力导致宕机。

![截屏2023-03-07 下午3.05.45](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-03-07 下午3.05.45.png)

eg：某个黑客故意制造一些非法的 key 发起大量请求，导致大量请求落到数据库，结果数据库上也没有查到对应的数据。也就是说这些请求最终都落到了数据库上，对数据库造成了巨大的压力。

### 解决方法

- 规范化参数校验(基本)
  - 查询的数据库 id 不能小于 0、传入的邮箱格式不对的时候直接返回错误消息给客户端等等。



- 缓存无效key
  - 如果缓存和数据库都查不到某个 key 的数据就写一个到 Redis 中去并设置过期时间，具体命令如下： `SET key value EX 10086` 。这种方式可以解决`请求的 key 变化不频繁`的情况，如果黑客恶意攻击，每次构建不同的请求 key，会导致 Redis 中缓存大量无效的 key 。很明显，这种方案并不能从根本上解决此问题。如果非要用这种方式来解决穿透问题的话，尽量将无效的 key 的过期时间设置短一点比如 1 分钟。



- 布隆过滤器

  - 一种来检索元素是否在给定大集合中的数据结构，这种数据结构是高效且性能很好的，但缺点是具有一定的错误识别率和删除难度。`布隆过滤器说某个元素存在，小概率会误判。布隆过滤器说某个元素不在，那么这个元素一定不在`。

    

布隆过滤器的使用场景:

1. 判断给定数据是否存在：比如判断一个数字是否存在于包含大量数字的数字集中（数字集很大，5 亿以上！）、 防止缓存穿透（`判断请求的数据是否有效避免直接绕过缓存请求数据库`）等等、邮箱的垃圾邮件过滤、黑名单功能等等。
2. 去重：比如爬给定网址的时候对已经爬取过的 URL 去重。



## 缓存击穿

请求的key对应的是 `热点数据`，该数据存在于数据库中，但不存在于缓存中(通常因为缓存中的那份数据已经过期)。瞬时大量的请求打到了数据库上，导致宕机。

![截屏2023-03-07 下午3.23.30](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-03-07 下午3.23.30.png)

eg ：秒杀进行过程中，缓存中的某个秒杀商品的数据突然过期，这就导致瞬时大量对该商品的请求直接落到数据库上，对数据库造成了巨大的压力。

### 解决方法

- 设置热点数据永不过期或者过期时间较长
- 针对热点数据提前预热，将其存入缓存中并设置合理的过期时间比如秒杀场景下的数据在秒杀结束之前不过期
- 请求数据库写数据到缓存之前，先获取【互斥锁】，保证只有一个请求会落到数据库上，减少数据库压力



## 缓存雪崩

`缓存在同一时间大面基失效`或`服务器宕机`，导致大量的请求都直接落到了数据库上，对数据库造成了巨大压力。

![截屏2023-03-07 下午3.34.04](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-03-07 下午3.34.04.png)

eg ：数据库中的大量数据在同一时间过期，这个时候突然有大量的请求需要访问这些过期的数据。这就导致大量的请求直接落到数据库上，对数据库造成了巨大的压力。

### 解决方法

- 针对redis服务不可用的情况

  - 采用redis集群，避免单机出现问题导致整个缓存服务都无法使用

  - 限流，避免同时处理大量请求

- 针对热点缓存失效的情况
  - 设置不同的失效时间比如随机
  - 缓存永不失效
  - 二级缓存

### 缓存击穿与雪崩的区别

- ==雪崩的原因是缓存中的大量或者所有数据失效==

- ==击穿的原因主要是某个热点数据不存在于缓存中(通常是过期)==



