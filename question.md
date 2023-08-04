# mysql

执行一条mysql期间，发生了什么?

- 客户端通过连接器连接，连接器会判断用户身份
- 解析器
  - 词法分析：对关键字解析
  - 语法分析：检查语句是否符合sql语法
- 预处理器
  - 判断表和字段是否存在
- 优化器
  - 确定执行计划
- 执行器
  - 执行sql



## 1.索引

### 1.1 为什么InnDB存储引擎选择使用B+tree索引结构(索引原理)？

数据库中的数据存储在磁盘中，而磁盘IO的效率很低，因此磁盘IO次数越少，性能的提升就越大。

- 相对于二叉树，B树是一颗多路平衡查找树，高度会比二叉树矮很多。
- 相对于B-树，B-树无论是否是叶子结点都会保存数据，这样会导致一页内存储的数据变少，磁盘IO次数也就更多了。
- B+树只有叶子节点会存储数据，磁盘IO更加稳定。
- 叶子节点之间通过双向链表链接，方便搜索与排序。



【补充】**二叉搜索树和二叉平衡树有什么关系?**

- 二叉搜索树的特性是所有节点都比当前节点小，右边的都比当前节点大

- 平衡二叉树是二叉搜索树的升级，左右子树高度差的绝对值为1

  【补充】**强平衡二叉树和弱平衡二叉树有什么区别？**

  - 强平衡二叉树是AVL树
  - 弱平衡二叉树是红黑树
    - AVL树比红黑树对于平衡的要求更加严格，相同情况下，AVL树高度低于红黑树
    - 红黑树节点增加了一个颜色概念
    - AVL的旋转操作比红黑树的旋转操作更耗时



### 1.2 Innodb中的b+树是怎么产生的?

Innodb中的page就是B+树叶子节点所指向的数据页。

如图所示，用户数据被存储在页中，页之间通过指针链接

![截屏2023-05-23 下午8.28.01](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-23 下午8.28.01.png)

 

### 1.3 索引分类

- 按照【数据结构】分类：
  - B+树索引
  - Hash索引(Hash索引在范围查询时性能很差)
  - Full-text索引
- 按照【物理存储】分类：
  - 聚簇索引
  - 二级索引
- 按照【字段特性】分类：
  - 主键索引
  - 唯一索引
  - 普通索引
  - 前缀索引
- 按照【字段个数】分类：
  - 单列索引
  - 联合索引

![截屏2023-05-23 下午8.36.10](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-23 下午8.36.10.png)

1）**聚簇索引(主键索引)**

Innodb中，会默认对主键创建聚簇索引(没有主键会通过隐藏字段)，聚簇索引根据主键构建B+树，叶子节点存储页地址，页地址中存储着数据。

![截屏2023-05-23 下午8.31.45](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-23 下午8.31.45.png)

2）二级索引

- 唯一索引、普通索引、全文索引
- 叶子阶段存储的是数据行(页)地址
- 对于二级索引来说，查询数据最终也会访问聚簇索引。

![截屏2023-05-23 下午8.35.06](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-23 下午8.35.06.png)

```mysql
# bcd是一个联合索引

# 在最左前缀法则中，看的是联合索引的顺序，不是where的顺序
# 即 c = 1 and b = 1有效
select * from t where c = 1 and b = 1;

# c = 1 and d = 1无效
select * from t where c = 1 and d = 1;

# 但如果只设计一个也是有效的
select * from t where d = 1;
```



### 1.4 使用索引有哪些注意事项？

#### 1）索引特性

- 覆盖索引【回表】
  - 覆盖索引是指【索引包含了查询所需的所有的列】
  - 在索引检索过程中，由于<font color=red>索引无法提供查询所需的所有数据，需要查询基表来获取所需的额外数据</font> 的操作，叫做【回表】
- 最左前缀法则(联合索引)
  - 查询从索引的最左列开始，不跳过索引中的列，即与索引创建时的顺序有关，与where等条件的顺序无关
- 索引下推(Using index condition)
  - 对于Innodb来说，适合于二级索引。利用索引条件先进行一些过滤，防止回表过多。

#### 2）索引不合适的场景

- 数据量少
- 更新比较频繁
- 区分度低(性别)

#### 3）索引失效

使用索引会提升搜索效率，但是错误使用会导致索引失效。

- 在索引列上进行运算操作
- 字符串类型字段不加引号(默认会将字符串转换为0再判断)
- 头部模糊匹配
- or两侧不是都有索引

#### 4）设计原则

- 针对数据量较大，且查询比较频繁的表建立索引。
- 针对于常作为查询条件(where)、排序(order by)、分组(group by)操作的字段建立索引。
- 尽量选择【区分度高】的列作为索引，尽量建立唯一索引，区分度越高，使用索引的效率越高。
- 如果是字符串类型的字段，字段的长度较长，可以针对于字段的特点，建立前缀索引。
- 尽量使用联合索引，减少单列索引。查询时，联合索引很多时候可以覆盖索引，节省存储空间，避免回表，提高查询效率。
- 要控制索引的数量，索引并不是多多益善，索引越多，维护索引结构的代价也就越大，会影响增删改查的效率。
- 如果索隐列不能存储NULL值，在创建表时用not null约束它，当优化器知道每列是否包含NULL时，它可以更好的确定那个索引最有效的用于查询。



## 2.事务

### 2.1 事务有哪些特性，如何保证这些特性？

- 原子性(Atomicity)：一个事务的所有操作，要么全部成功，要么全部失败。

  - 执行失败时，通过undo log【记录事务操作的逆操作】进行回滚

  

- 一致性(Consistency)：

  - 事务操作前和操作后，数据库应保持一致。

  

- 隔离性(Isolation)：

  - 事务的操作对其他并发的事务是隔离的，互不影响。

  

- 持久性(Durability)：事务一旦提交，对数据库的影响是永久的。

  - redo log，记录事务提交时数据页的物理修改。
  - 由【重做缓冲日志(redo log buffer)(内存)】和【重做日志文件(redo log file)(磁盘)】组成。
  - 事务提交后会把所有修改信息都存到该日志中，用于在刷新脏页到磁盘发生错误时进行数据恢复。

  - 当Buffer Pool中的数据还没写入到ibd文件时，称为【脏页】，如果此时发生故障，那么就可以用redo log进行恢复，日志文件是追加读写(顺序磁盘I/O)，比不使用redo log的随机磁盘I/O效率高。
  - 脏页的数据顺利刷新到磁盘后，redo log中的日志会在一段时间后被清理。

![截屏2023-05-26 下午4.23.21](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-26 下午4.23.21.png)

![截屏2023-05-26 下午2.45.57](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-26 下午2.45.57.png)



- redo log 记录了此次事务「**完成后**」的数据状态，记录的是更新**之后**的值；
- undo log 记录了此次事务「**开始前**」的数据状态，记录的是更新**之前**的值；

### 2.2 事务隔离级别以及会产生的问题？

- Read UnCommitted
  - 一个事务没提交时，它的变更就能被其他事务看到
  - 脏读、不可重复读、幻读
- Read Committed
  - 一个事务提交后，它的变更才能被其他事务看到
  - 幻读、不可重复读
- Repeatable Read
  - 一个事务执行过程中看到的数据，与事务开始时的一致
  - 幻读
- Serilizable
  - 类似单线程



### 2.3 什么是MVCC？

- 当前读（悲观锁）: 读取最新版本的记录
  - select ... lock in share mode;(共享锁)
  - select ... for update(排他锁)
  - update,insert,delete(排他锁)
- 快照读（乐观锁）: 简单的select就是快照读，读取的是记录数据的可见版本，有可能是历史数据，不加锁，是非阻塞读。
  - Read committed: 每次select，都生成一个快照读
  - Repeatable Read: <font color=red>开启事务后第一个select语句才是快照读的地方</font>
  - Serializable: 快照读会退化为当前读



**数据库并发场景有三种，分别为：**

- `读-读`：不存在任何问题，也不需要并发控制
- `读-写`：有线程安全问题，可能会造成事务隔离性问题，可能遇到脏读，幻读，不可重复读
- `写-写`：有线程安全问题，可能会存在更新丢失问题。

MVCC是一种用来解决<font color=red>读-写</font>冲突的**无锁并发控制**

#### 实现原理

具体实现依赖于数据库记录中的`三个隐式字段、undolog、readView`

##### 1.隐藏字段

| 隐藏字段    | 含义                                                         |
| ----------- | ------------------------------------------------------------ |
| DB_TRX_JD   | 最近修改事务ID，记录插入这条记录或最后一次修改该记录的事务ID |
| DB_ROLL_PTR | 回滚指针，指向这条记录的上一个版本，配合undo log             |
| DB_ROW_ID   | 隐藏主键，如果表结构没有指定逐渐，将会生成该隐藏字段         |

![截屏2023-05-26 下午3.01.43](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-26 下午3.01.43.png)

记录中的 trx_id 划分这三种情况：

![截屏2023-05-26 下午3.08.25](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-26 下午3.08.25.png)



##### 2.undo log

记录数据库事务操作的逆操作

回滚日志，在insert、update、delete的时候产生的便于数据回滚的日志。

insert时，产生的undo log日志只在回滚时需要，在事务提交后，可立即被删除。

而update、delete时，产生的undo log日志不仅在回滚时需要，在快照读里也需要，不会被立即删除。

##### 3.readView(读视图)

是快照读SQL执行时MVCC提取数据的依据，记录并维护系统当前活跃的事务。

![截屏2023-05-26 下午3.04.05](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-26 下午3.04.05.png)

- 如果记录的 trx_id 值小于 Read View 中的 `min_trx_id` 值，表示这个版本的记录是在创建 Read View **前**已经提交的事务生成的，所以该版本的记录对当前事务**可见**。
- 如果记录的 trx_id 值大于等于 Read View 中的 `max_trx_id` 值，表示这个版本的记录是在创建 Read View **后**才启动的事务生成的，所以该版本的记录对当前事务**不可见**。
- 如果记录的 trx_id 值在 Read View 的`min_trx_id`和`max_trx_id`之间，需要判断 trx_id 是否在 m_ids 列表中：
  - 如果记录的 trx_id **在** `m_ids` 列表中，表示生成该版本记录的活跃事务依然活跃着（还没提交事务），所以该版本的记录对当前事务**不可见**。
  - 如果记录的 trx_id **不在** `m_ids`列表中，表示生成该版本记录的活跃事务已经被提交，所以该版本的记录对当前事务**可见**。



**可重复读隔离级别是启动事务时生成一个 Read View，然后整个事务期间都在用这个 Read View**。

**读提交隔离级别是在每次读取数据时，都会生成一个新的 Read View**。

## 3.锁

按照【锁力度】分类：

- 行锁：锁某行数据，锁力度最小，并发度高
- 表锁：锁整张表，锁力度最大，并发度级
- 间隙锁：锁一个区间



还可以分为：

- 共享锁：读锁，A事务加了读锁，其他事务【可读不可写】
- 排他锁：写锁，A事务加了写锁，其他事务【不可读不可写】



还可以分为：

- 乐观锁
  - 并不是真正加锁，通过版本号机制实现
  - **<font color=red>适用于多读场景</font>，避免频繁加锁影响性能，大大提升了系统的吞吐量。**
- 悲观锁
  - 行锁、表锁都是悲观锁
  - **<font color=red>适用于多写场景</font>，避免频繁失败和重试影响性能。**



### 字节面试题

在【可重复读】级别下，以下操作会发生什么？

![截屏2023-05-26 下午3.14.40](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-26 下午3.14.40.png)

|                          A                           |                           B                           |
| :--------------------------------------------------: | :---------------------------------------------------: |
|   update t_student set score = 1000 where id = 26;   |                                                       |
|                                                      |   update t_student set score = 1000 where id = 25;    |
| insert into t_student values(26, 26, 'ace', 28, 90); |                                                       |
|                                                      | insert into t_student values(25, 25, 'sony', 28, 90); |

死锁，因为25,26不存在，事务使用的是next-key lock，锁住了(20,30]，此时执行插入操作会被阻塞，互相等待造成死锁。



## 4.日志

- **undo log（回滚日志）**：是 Innodb 存储引擎层生成的日志，实现了事务中的**原子性**，主要**用于事务回滚和 MVCC**。
- **redo log（重做日志）**：是 Innodb 存储引擎层生成的日志，实现了事务中的**持久性**，主要**用于掉电等故障恢复**；
- **binlog （归档日志）**：是 Server 层生成的日志，主要**用于数据备份和主从复制**；



## 3.SQL优化

### 3.1 优化思路

1. 硬件和操作系统层面的优化
   - 包括CPU、内存、磁盘速度，网络带宽等，一般有DBA完成
2. 架构设计的优化
   - 主从集群、读写分离、分库分表
   - 针对热点数据，引入缓存
3. SQL语句优化
   - 通过慢查询日志分析sql语句
   - 使用explain分析



### 3.2 分库分表设计

#### 1) 分库分表方案

- 水平分库：
  - 以字段为依据，按照策略(hash、range)等，将一个【库】中的数据拆分到多个【库】中
- 水平分表：
  - 以字段为依据，按照策略(hash、range)等，将一个【表】中的数据拆分到多个【表】中
- 垂直分库：
  - 以表为依据，按照业务归属不同，将不同的【表】拆分到不同的【库】中
- 垂直分表：
  - 以字段为依据，按照字段的活跃性，将表中字段拆到不同的表（主表和扩展表）



#### 2) **常用的分库分表中间件**

- sharding-jdbc(当当)
- Mycat
- TDDL(淘宝)
- Oceanus(58同城)
- vitess(谷歌开发的数据库中间件)
- Atlas(Qihoo 360)



#### 3) **可能遇到的问题**

- 事务问题：需要分布式事务

- 跨节点 join 问题：两次查询

- 跨节点的 count,order by, group by，聚合函数的问题：分别在各个节点上得到结果后再应用程序端合并

- 数据迁移，容量规划，扩容等

- ID问题

  - |                            | 优点                           | 缺点                                             |
    | -------------------------- | ------------------------------ | ------------------------------------------------ |
    | UUID                       | 代码简单，没有网络开销，性能好 | 占用空间大、无序、不符合主键id的简介             |
    | 数据库自增ID               | 成本小(奇偶数)                 | 扩展性差                                         |
    | Redis INCR                 | 性能优于数据库、ID有序         | 解决单点问题带来的数据一致性等问题使得复杂度提高 |
    | 雪花算法                   | 性能高，根据bit位分配灵活      | 强依赖机器时钟                                   |
    | 号段模式                   | 数据库压力小                   | 单点故障id不连续                                 |
    | Leaf、Uidgenerator、TinyID | 高性能、高可用、接入简单       | 依赖第三方组件如Zookeeper                        |

- 跨分片的排序分页问题



#### 4) 应用场景

- 只分库不分表
  - <font color=red>数据库的读写访问量过高，可能会出现数据库连接不够</font>。这时需要考虑分库，通过增加数据库实例来获取更多连接，提升并发性能。
- 只分表不分库
  - <font color=red>单表存储的数据量非常大，并发量不高、数据库连接数够用，但读写性能出现了瓶颈</font>。这时需要考虑分表，将数据拆分到多张表中来减少单表存储的数据量，提升性能。

- 分库分表
  - <font color=red>结合上面的两种情况，数据库连接不够用，单表数据量也很大</font>。



### 3.3 设计数据库表时，字段应如何选择

- 字段优先级

  - 整型 > date，time > enum char > varchar > blob text

  - 选用字段长度最小、优先使用定长型、数值型字段避免使用“ZEROFILL”

  - time:：定长运算快，节省时间，考虑时区，写sql不方便

  - enum：能约束值的目的，内部用整型存储，但与char联查时，内部要经历串与值的转化

  - char：定长，考虑字符集和校对集

  - varchar：不定长，要考虑字符集的转换与排序时的校对集，速度慢

  - text，blob：无法使用内存临时表(排序只能在磁盘上进行)

- 可以选整型就不选字符串
- 够用就行
  - 大的字段影响内存影响速度，以年龄为例，tinyint unsigned not null;可以存储0~255，用int浪费了3个字节。
  - varchar(10)，varchar(300)储存的内容相同，但在表中查询时，varchar(300)要花更多内存
- 避免使用Null
  - Null不利于索引，也不利于查询，=null或!=null都查询不到值，只有使用is null或者is not null才可以。因此创建字段时使用not null default ""的形式
- char 与 varchar的选择
  - char 长度固定，处理速度要比 varchar 快很多，但是相对较费存储空间
  - 对存储空间要求不大，但在速度上有要求的可以使用 char
  - 反之使用 varchar



## 4.MyIsam与Innodb的区别

| 特点     | Innodb | MyISAM |
| -------- | ------ | ------ |
| 存储限制 | 64TB   | 无限   |
| 事务     | 支持   | 不支持 |
| 锁       | 行锁   | 表锁   |
| 外键     | 支持   | 不支持 |



## 5.日常查询问题

1.`on`和`where`有什么区别

- on用于join操作中，指定连接表的条件，因此出现在连接表之前

- where用于在查询中过滤，在连接表之后

  

# redis

redis的性能瓶颈不在cpu，而在于内存和网络io。引入多线程虽然可以提高cpu利用率，但是对于多线程的上下文切换、死锁预防等的处理复杂，会带来很大开销。虽然redis的单线程的，但是它是一个reactor模型。使用了IO多路复用技术，通过select,poll,epoll等系统调用同时监听多个FD，可以使得单线程高效处理多个任务。redis6.0时引入了多线程，但是是对网络IO的多线程，分担网络符合，redis命令的执行还是单线程的



# java基础

## 1.集合

### HashMap

#### 1.1 hashmap如何解决hash冲突？

hash冲突指的是不同的数据经过计算得到了相同的hashcode。

结算hash冲突的算法有

- 再hash法
  - 如果某个hash函数产生了冲突，再用另一个hash进行计算，比如布隆过滤器
- 开放寻址法
  - 直接从冲突的数组位置往后寻找一个空下标存储，在ThreadLocal中有应用
- 公共溢出区
  - 把冲突的key统一放在一个公共溢出区中
- 链式寻址法
  - hashmap解决hash冲突的方法，在jdk1.8中使用的是尾插法。为了避免链表过长影响查询效率，当数组长度>64，链表长度>=8时，链表会转化成红黑树



#### 1.2 HashMap中的hash方法为什么要>>>16？

使得散列度更高，减少hash冲突



#### 1.3 HashMap与HashTable的区别?

- 线程安全性
  - HashMap线程不安全
  - HashTable线程安全（绝大部分方法加了synchronized，效率低）

- 存储结构
  - HashMap采用数组+链表+红黑树
  - HashTable采用数组+链表
- 初始容量
  - HashMap初始容量为16
  - HashTable初始容量为11
- null作为key
  - HashMap允许null作为key(null的key值会转化为0)
  - HashTable不允许null为key

- 散列算法
  - HashMap在散列算法中，会额外>>>16位，减少hash冲突
  - HashTable直接使用key的hashcode对数组长度做取模



#### 1.4 HashMap扩容？

根据默认的负载因子0.75，HashMap在长度达到容量3/4的时候会扩容，创建一个大小为原来的2倍的新表。

假如事先知道要存储元素数量的大致范围，可以先指定大小，避免多次扩容印象性能。

负载因子的大小是空间利用率与冲突概率的平衡设计，通过破松分布确定默认值



#### 1.5 为什么HashMap中不直接使用红黑树直接存储？

基于性能和查找效率的综合考量，虽然红黑树的查找速度更快，但是红黑树本身的一些旋转操作也会影响性能。



### ConcurrentHashMap





## 2.线程

### 2.1 什么时候线程无法终止？

- 死锁
- 无限循环
- 异常处理不当



## 3.JUC

### 什么是CAS？

Compare And Swap，是一种乐观锁思想，在无锁情况下，底层调用Unsafe类，保证线程操作共享数据的原子性。JUC包下的AQS，Atomic类都用到了CAS操作。



### 什么是AQS？

- AbstractQueuedSynchronizer是阻塞式锁和相关的同步器工具的框架。

- 通过继承AQS抽象类，实现其中的

  - tryAcquire()

  - tryRelease()

  - tryAcquireShared()

  - tryReleaseShared()

  - isHeldExclusively()

    这五个模版方法，可以实现高性能的同步器和并发工具。例如`ReentrantLock、CountdownLatch、Semaphore、CyclicBarrier`都是基于AQS实现的。

- AQS采用了`volatile int state=0`来表示资源的状态(分为共享锁和排他锁)，子类需要定义如何维护这个状态，来控制锁的获取与释放

  - getState() - 获取state状态
  - setState() - 设置state状态
  - compareAndSetState --- cas设置状态

- 提供了基于FIFO的CLH队列，类似于synchronized锁中，Monitor对象的EntryList

- 提供了条件变量来实现等待、唤醒机制，支持多个条件变量，类似Monitor对象的WaitSet



### 什么是JUC？

JUC是java语言中的一个包，用于编写并发程序的工具和类。

- 并发集合(ConcurrentHashMap)
- 提供了AQS抽象类，AQS全称（Abstract Queued Synchronizer)，抽象队列同步器。通过继承其中的模版方法，可以实现可以实现高性能的同步器和并发工具
  - 比如说基于独占的ReentrantLock
  - 基于共享的CountDownLatch、Semaphore、CyclicBarrier
- 原子类
  - 原子整数、原子引用、原子数组
  - 字段更新器
  - 原子累加器
- 线程池



### ThreadPoolExecutors？

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```

```java
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

为什么不建议使用？

- newFixedThreadPool、SInglePool
  - 队列无界，可能OOM
- newCachedThreadPool
  - 最大线程数量为MAX_VALUE，可能导致OOM



### 多线程使用场景？

- 大量数据的导入
  - mysql导入es、从文件中读取大量数据等
- 异步调用
  - 比如搜索时，异步线程记录搜索历史



## 为什么JDK动态代理只能代理接口？

Proxy的底层实现，在程序运行期间动态生成一个代理类$Proxy并继承Proxy类，而java不支持多继承，因此只能代理接口。但如果把Proxy中的InvocationHandler接口抽出来，也是可以直接代理类的。

之所以这么设计我认为是基于java的设计原则考虑的：

1.遵循面向接口编程，避免单继承的限制从而方便扩展。

2.代理类需要动态生成，接口不包含具体的实现代码，相对于类降低了开销



# 设计模式

基础的设计模式分为三类
第一类创建型模式，关注对象的创建，包括单例，原型，建造者模式等
第二类结构型模式，关注类与对象的组合关系，包括享元、装饰器模式、代理模式等。
第三类行为型模式，关注对象之间的消息传递，包括策略模式、模版方法、观察者、迭代器模式等
此外，还了解过两阶段终止、保护性暂停、生产消费者模式多线程设计的常用模式。





# Springboot



## 核心功能

- 可以独立运行Spring项目
  - 可以以 jar 包形式独立运行
- 内嵌 Servlet 容器
  - 执行DemoApplication快速运行
- 提供 `starter` 简化 `Maven` 依赖
  - 提供pom文件简化依赖
- 自动配置 Spring
- 无XML配置



## 构造函数推断

在存在多个构造器的情况下，默认会选择无参构造器。

如果没有无参构造器，需通过@Autowired指定，否则spring因不知道使用哪个而报错



## 什么是IOC

`Inversion of Control`将对象的创建和管理通过注解交给容器来完成，使得程序员只需要关心业务逻辑的实现而不是对象之间的依赖关系。



## @Autowired和@Resouce区别

都是Spring中实现Bean注入的注解

- @Autowired 根据==类型==匹配

```java
@Service
public class UserService{
  // 默认true,强制要求实现注入
  // 不希望注入，false 
  @Autowired(required = true)
  private HelloService helloService;
  
  @Autowired
  @Qualifier()
}
```

- @Resouce 根据==bean的名字==注入

```java
@Service
public class UserService{
			// 根据bean的名字注入
  		@Resource(name = "userService01")
  		private UserService01 userService;
  		
  		// 根据类型注入
  		@Resource(type = UserService02.class)
      private UserService02 userService;
  }
}
```



## @Component和@Confuguration区别

- @Component 用于标识`普通的 Spring 组件类`,该类需要被Spring管理和实例化。
- @Configuration 用于标识`配置类`，即定义 Bean 的工厂类。该类可以包含一个或多个@Bean方法，这些方法将返回一个被Spring管理的对象实例



## @Component和@Bean区别

- `@Component`注解作用于类，`@Bean`作用于方法

- @Component通常通过类路径扫描来自动侦测以及自动装配到Spring容器中(`@ComponentScan`注解定义要扫描的路径从中找出标了需要装配的类自动装配到Spring的bean容器中)。@Bean注解通常是我们在标有该注解的方法中定义产生这个bean，`@Bean`告诉了Spring这是某个类的实例。

- `@Bean`注解比`@Component`注解的自定义性更强

  

## 拦截器和过滤器的区别

|          | 过滤器filter                                                 | 拦截器interceptor                          |
| :------: | :----------------------------------------------------------- | :----------------------------------------- |
| 生命周期 | 在Servlet容器中执行                                          | 在Spring容器中执行                         |
| 作用对象 | 针对请求URL或者请求的servlet进行拦截处理，解析post、resutful风格的表单数据等 | 针对方法进行拦截处理，权限校验、登录校验等 |
| 作用范围 | 可以拦截静态资源(图片、css文件)                              | 拦截请求到Controller层的请求               |
| 拦截时机 | 请求进入Servlet容器之前和Servlet容器之后                     | 请求进入Spring MVC控制器之前和进入视图之前 |
|          |                                                              |                                            |







## 自动装配机制

@SpringApplication注解中带有@AutowiredConfiguration，这个注解中引入了AutoConfiguration类，其中定义了SpringFactoryl.loadFactoryNames方法，通过这个方法会扫描META-INFO下的spring.factories，spring根据定义去配置与扩展，比如Spring-data-redis等依赖。而像fastjson这种独立的JSON处理库就没有spring.factories，因为它不需要spring框架的自动配置机制。

## 循环依赖

三级缓存 确保循环依赖情况下的单例性

1. singletonObjects：单例对象缓存，存储已经完全初始化好的单例Bean实例对象，其中Key为Bean的名称，Value为Bean实例对象。
2. earlySingletonObjects：早期单例对象缓存，存储正在创建和初始化的单例Bean实例对象，其中Key为Bean的名称，Value为Bean实例对象。
3. singletonFactories：单例对象工厂缓存，存储用于创建和初始化单例Bean实例对象的ObjectFactory，其中Key为Bean的名称，Value为ObjectFactory对象。

当Spring容器创建Bean时，会先尝试从singletonObjects缓存中获取Bean实例，如果缓存中不存在，则会从earlySingletonObjects缓存中获取Bean实例，如果仍然不存在，则会从singletonFactories缓存中获取ObjectFactory，通过ObjectFactory创建Bean实例并加入到earlySingletonObjects缓存中，然后再初始化Bean，初始化完成后将Bean实例从earlySingletonObjects缓存中移动到singletonObjects缓存中。

三级缓存的作用是提高Spring容器创建和初始化Bean的效率，避免重复创建和初始化相同的Bean实例。同时，它也保证了Bean的单例性，即相同名称的Bean在容器中只有一个实例。



如通过构造函数注入、@Lazy注解、@Autowired注解的required属性、@DependsOn注解等方式来控制Bean的创建顺序，避免循环依赖的发生。



## bean的生命周期

1.Spring Boot根据配置文件、注解等调用PostProcessorBefroe/After Instantiation创建Bean的实例。

2.创建完成后，调用PostProcessPropertires方法对@Autowired,@Resource,@Value等注解做处理。

3.然后调用PostProcessBefore/After Initialization方法，对其中有@PostConstruct、@ConfigurationProperties注解的作初始化处理，到这里Bean就可以被使用了

4.在Bean不再需要时,如果调用postProcessBeforeDestruction等方法，作对应的销毁前处理，然后销毁bean。



创建前准备

创建实例

依赖注入

容器缓存

销毁



# JVM

![截屏2023-06-10 下午6.54.07](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-06-10 下午6.54.07.png)

## 1.JVM组成

### 什么是程序计数器？

- 线程私有
- 每个线程一份
- 记录正在执行的字节码的地址



### 什么是堆？

- 线程共享

- 主要用来保存对象实例、数组等，当没有空间利用时，OOM
- 组成：年轻代+老年代
  - 年轻代划分为Eden,from,to
  - 老年代保存一些老的对象
- jdk1.7和1.8的区别
  - 1.7，堆中有一个永久代，存储类信息、静态变量、常量等
  - 1.8移除了永久代，把数据存储到了本地内存的元空间中，防止内存溢出

- <font color=red>堆中的java对象只有【数据部分】，对象中数据的含义，要结合【方法区】中的class才能明白</font>

### 什么是虚拟机栈？

- 线程私有

- 每个线程运行时所需要的内存，称为虚拟机栈
- 每个栈由多个栈帧组成，对应每次方法调用所占内存
- 每个线程只能有一个活动栈帧，对应正在执行的方法

#### 垃圾回收是否涉及栈内存？

不涉及，栈帧弹出时内存会释放

#### 栈内存分配越大越好吗？

默认栈内存1024k，M = 活动线程个数 * 栈内存值

#### 方法内的局部变量是否线程安全？

- 方法内局部变量没有逃离方法作用范围，安全

  - 

- 局部变量引用了对象，并逃离方法作用范围，线程不安全

  - ```java
    public static void test1(StringBuilder s1){
        s1.append(3);
        s1.append(4);
        System.out.println(s1);
    }
    
    public static StringBuilder test2(){
        StringBuilder s = new StringBuilder();
        s.append(3);
        s.append(4);
        return s;
    }
    ```

    

### 什么是方法区？

- 线程共享
- 主要存储类的信息、运行时常量池
- 虚拟机启动时创建，关闭时释放
- 如果内存无法满足了，抛出OOM:Metaspace

![截屏2023-06-10 下午7.15.39](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-06-10 下午7.15.39.png)



#### 常量池

- 可以看做一张表，虚拟机根据表寻找要执行的类名、方法名、参数类型、字面量等信息

#### 运行时常量池

- 常量池是*.class文件中的，当类被加载时，常量池信息放入【运行时常量池】



### 什么是直接内存？

- 直接内存不属于JVM中的内存结构，不由JVM管理
- 是虚拟机的系统内存，常见于NIO操作，用于数据缓冲
- 分配回收成本高，读写性能好



## 2.类加载器

### 什么是类加载器？

JVM只会运行二进制文件，类加载器将【字节码文件加载到 JVM 中】

- Bootstrap ClassLoader
  - 启动类加载器，加载【JAVA_HOME/jre/lib目录下的库】
- Extension ClassLoader
  - 扩展类加载器，加载【JAVA_HOME/jre/lib/ext目录中的类】
- Applicaiton ClassLoader
  - 应用类加载器，加载【classpath 开发者自己编写的Java类】
- 自定义类加载器
  - 自定义 ClassLoader



#### 类加载机制

- 加载
  - JVM根据类名找到对应的字节码文件并加载到内存
- 链接
  - 校验字节码文件是否符合规范
  - 为静态变量分配内存空间
  - 将符号引用(字符串引用)转换为直接引用
- 初始化
  - 初始化静态变量、静态代码块



### 什么是双亲委派模型？

- 加载某个类，先委托上一级加载器加载。如果该类委托上级没有被加载，子加载器尝试加载该类

```java
protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException{
    synchronized (getClassLoadingLock(name)) {
        //首先，检查该类是否已经加载过
        Class c = findLoadedClass(name);
        if (c == null) {
            //如果 c 为 null，则说明该类没有被加载过
            long t0 = System.nanoTime();
            try {
                if (parent != null) {
                    //当父类的加载器不为空，则通过父类的loadClass来加载该类
                    c = parent.loadClass(name, false);
                } else {
                    //当父类的加载器为空，则调用启动类加载器来加载该类
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                //非空父类的类加载器无法找到相应的类，则抛出异常
            }
            ......
```



#### 为什么双亲委派？

- 双亲委派模型保证了 Java 程序的稳定运行，可以避免类的重复加载（JVM 区分不同类的方式不仅仅根据类名，相同的类文件被不同的类加载器加载产生的是两个不同的类），也保证了 Java 的核心 API 不被篡改。

例子：

编写一个称为 `java.lang.Object` 类的话，那么程序运行的时候，系统就会出现两个不同的 `Object` 类。双亲委派模型可以保证加载的是 JRE 里的那个 `Object` 类，而不是你写的 `Object` 类。这是因为 `AppClassLoader` 在加载你的 `Object` 类时，会委托给 `ExtClassLoader` 去加载，而 `ExtClassLoader` 又会委托给 `BootstrapClassLoader`，`BootstrapClassLoader` 发现自己已经加载过了 `Object` 类，会直接返回，不会去加载你写的 `Object` 类。



#### java中如何判断两个类是否相同

- 类名是否相同
- 加载此类的类加载器是否相同



#### 如何打破双亲委派？

- <font color=red>自定义类加载器</font>：自定义类加载器。继承`ClassLoader`。
  - 如果不想打破双亲委派模型，就重写 `ClassLoader` 类中的 `findClass()` 方法即可。
  - 如果想打破双亲委派模型则需要重写 `loadClass()` 方法。
- <font color=red>线程上下文类加载器</font>：Java提供了一个线程上下文类加载器，可以在程序运行时设置线程上下文类加载器来加载类。可以通过线程上下文类加载器来实现打破双亲委派机制，由线程上下文类加载器来加载类，而不是通过双亲委派机制。



### 创建对象过程

- JVM读取到new时，检查运行时常量池是否存在当前引用指向的类的信息，没有就通过类原信息进行加载。
- 加载完成后，通过常量池中的这个信息确定对象的类型，需要申请的内存大小。
- 然后对对象内存空间赋初值，保障对象没有初始化也可以直接调用。
- 接着设置对象头，包括所属的类、类元信息、hashcode、GC年龄等。
- 最后调用初始化方法初始化，执行JVM的init方法，创建完成。



#### 分配内存的方式

- 指针碰撞
  - 适用场景：<font color=red>堆内存规整(没有内存碎片)的情况</font>
  - 原理：用过的内存全部整合到一边，没用过的另一边。中间有一个分界指针，只需要向着没用过的内存方向向该指针移动对象内存大小位置即可。
  - 应用：Serial，ParNew
- 空闲列表
  - 适用场景：<font color=red>堆内存不规整的情况</font>
  - 原理：虚拟机维护一个列表，记录哪些内存块是可用的。在分配的时候，找一块足够大的内存块划分给对象实例，最后更新列表。
  - 应用：CMS

- 分区管理
  - 原理：将内存划分为多个Region
  - 应用：G1

## 3.垃圾回收

### 什么时候对象可以被回收？

- 引用计数法

- 可达性分析法



### 有什么垃圾回收算法？

- 标记清除
  - 标记和清除速率快
  - 留下内存碎片
- 标记整理
  - 没有内存碎片
  - 效率有一定的影响
- 复制
  - 垃圾多的情况下效率高，无内存碎片
  - 2块内存空间



### 什么是分代垃圾回收？

- 堆被分为了两部分，
  - 新生代 : 老年代 = 1 : 2
  - 新生代中，Eden : from : to = 8 : 1 : 1

- 回收策略
  - 新创建的对象分配到Eden区
  - Eden内存不足时，触发minor gc(STW)，Eden和from存活的对象使用copy复制到to中，存活的对象年龄加一，交换from区和to区
  - 当对象寿命超过阈值时，会晋升至老年代，最大寿命是15(4bit)
  - 当老年代空间不足时，先尝试minor gc，如果之后空间仍然不足，那么触发full gc，STW时间更长



### MinorGC、Mixed GC、FullGC？

- MinorGC
  - 【yong GC】发生在新生代垃圾回收，STW
- MixedGC 
  - 新生代+老年代部分区域的垃圾回收，G1特有
- FullGC
  - 新生代+老年代完成垃圾回收，STW时间长 



### 有哪些垃圾回收器？

- 串行垃圾回收器

- 并行垃圾回收器

- CMS【==获取最短回收停顿时间为目标==】

  - 标记清除

    初始标记 并发标记 重新标记 并发清除

  - 缺点

    标记清除会产生大量的内存碎片无法利用，过早触发full gc

    并发标记时，GC线程会影响一定的效率

    并发标记时用户线程也在运行，需要通过重新标记弥补用户线程可能发生的改变，并发清除阶段也存在这个问题，且这一部分需要下次GC才能清楚

- G1【==高概率满足 GC 停顿时间要求的同时,具备高吞吐量性能特征==】

  - 总体上是标记-整理，区域之间是复制
  
    初始标记 并发标记 最终标记 筛选回收
  
- **==可预测的停顿==**：相对于 CMS 优势。除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为 M 毫秒的时间片段内。

三色标记法

白 还没被垃圾回收器扫描过的对象

黑 被垃圾回收器扫描过，且对象及其引用都是存活的对象

灰 被垃圾回收器扫描过，但其引用的对象还没扫描过




## 4.JVM实践

### 调优参数在哪里设置?

- war包
  - tomcat中设置
  - 【TOMCAT_HOME/bin/catalina.sh   JAVA_OPTS=”-Xms512m -Xmx1024m“】
- jar包
  - 启动参数设置
  - nohup java -Xms512m -Xmx1024m -jar test.jar > test.log &

### 调优参数?

- 堆内存大小
  - 初始大小为物理内存的1/64，最大大小为物理内存的1/4
  - -Xms=128m -Xmx=2048m

- 虚拟机栈
  - -Xss=128k

- Eden与Survivor区
  - -XXSurvivorRatio=8【survior:Eden=2:8】

- 晋升阈值
  - -XX:MAXTenuringThreshold=15
- 设置垃圾回收器
  - -XX:+UseG1GC

### 调优工具?

- 命令工具
  - jps
  - jstack
- 可视化工具
  - jconsole
  - VisualVM



### 内存泄露排查思路?

- 虚拟机栈：StackOverFlow
- 堆：OOM:java heap space
- 方法区：OOM:Metaspace

内存泄露通常指堆内存中，一些大对象不被回收

- jmap或设置VM参数获取堆内存dump快照
- 通过VisualVM分析dump文件
- 查看堆内存信息情况，定位代码



### CPU飙高的排查思路？

1. top查看CPU占用情况，找到占用高的进程
2. ps H -eo,pid,tid,%cpu | grep 3169【H表示显示线程，-eo表示要显示的列】
3. 根据线程id进一步定位代码【十六进制转十进制 printf "%x\n" 30021】



# 计算机网络

## 三次握手与四次挥手

### 1.三次握手

效果表示了为什么不是2次握手

|      | 过程                                                        | 效果                                                       |
| ---- | ----------------------------------------------------------- | ---------------------------------------------------------- |
| 1    | **客户端**向服务端发送SYN(Seq=X)标志的数据包==>**服务端**   | **服务端**确认自己接收正常，**客户端**发送正常             |
| 2    | **服务端**发送带有SYN(Seq=y),ACK = 1,ack = x+1==>**客户端** | **客户端**确认自己发送、接收正常，**服务端**发送、接收正常 |
| 3    | **客户端**发送ACK=1，ack=y+1==>**服务端**接收到后完成       | **服务端**确认自己发送、接收正常，**服务端**发送、接收正常 |

SYN 同步序列编号(Synchronize Sequence Numbers) 是 TCP/IP 建立连接时使用的握手信号。

###  2.四次挥手

1. 客户端发送一个 FIN（SEQ=X） 标志的数据包==>服务端，用来关闭客户端到服务器的数据传送。
2. 服务器收到这个 FIN（SEQ=X） 标志的数据包，它发送一个 ACK （SEQ=X+1）标志的数据包==>客户端 。
3. 服务端关闭与客户端的连接，并发送一个 FIN （SEQ=y）标志的数据包==>客户端，请求客户端关闭连接。
4. 客户端发送 ACK (SEQ=y+1)标志的数据包==>服务端，服务端收到。客户端等待 **2MSL** 后依然没有收到回复，就证明服务端已正常关闭，随后，客户端也可以关闭连接了。
   **只要四次挥手没有结束，客户端和服务端就可以继续传输数据！**

#### 为什么要四次挥手？

TCP是全双工通信，可以双向传输数据。任何一方都可以在数据传送结束后发出连接释放的通知，待对方确认后进入半关闭状态。当另一方也没有数据再发送的时候，则发出连接释放通知，对方确认后就完全关闭了 TCP 连接。

#### 为什么不能把服务器发送的 ACK 和 FIN 合并起来，变成三次挥手？

因为==服务器收到客户端断开连接的请求时，可能还有一些数据没有发完==，这时先回复 ACK，表示接收到了断开连接的请求。等到数据发完之后再发 FIN，断开服务器到客户端的数据传送。

#### 如果第二次挥手时服务器的 ACK 没有送达客户端，会怎样？

客户端没有收到 ACK 确认，会重新发送 FIN 请求。

#### 为什么第四次挥手客户端需要等待 ==2*MSL（报文段最长寿命）==时间后才进入 CLOSED 状态？

第四次挥手时，客户端发送给服务器的 ACK 有可能丢失，如果服务端因为某些原因而没有收到 ACK 的话，服务端就会重发 FIN，如果客户端在 2*MSL 的时间内收到了 FIN，就会重新发送 ACK 并再次等待 2MSL，防止 Server 没有收到 ACK 而不断重发 FIN。

> **MSL(Maximum Segment Lifetime)** : 一个片段在网络中最大的存活时间，2MSL 就是一个发送和一个回复所需的最大时间。如果直到 2MSL，Client 都没有再次收到 FIN，那么 Client 推断 ACK 已经被成功接收，则结束 TCP 连接。





## TCP与UDP

|      | TCP                      | UDP                    |
| ---- | ------------------------ | ---------------------- |
|      | 有状态、有链接、可靠     | 无状态、无连接、不可靠 |
| 速率 | 较慢                     | 较快                   |
| 传输 | 字节流？                 | 数据报？               |
| 适用 | 文件传输（精准、无差错） | 语音、视频、音频       |



## cookie与session

### 区别

|          | cookie         | session        |
| -------- | -------------- | -------------- |
| 有效期   | 长             | 短             |
| 作用范围 | 客户端(浏览器) | 服务端         |
| 存取方式 | ASCII          | 任意(如userId) |
| 大小     | <4k            | 任意           |



### 禁用cookie会怎么样？

1、拼接session_id

2、token令牌



### 分布式系统中的session_id如何处理？

1. 请求精准定位（nginx、IP_hash)
2. session复制共享（tomcat）
3. 基于缓存共享（redis）



## 同源缓存策略与跨域请求

### 什么是同源？

- 协议相同
- 域名相同
- 端口相同



### 目的是什么？

保证用户信息的安全性，防止恶意篡改数据



### 如何解决跨域请求？

1. 使用CORS(cross-origin-resource-sharing),设置响应头字段(Access-control-{ })
2. nginx(使用了CORS)



## get和post的区别

- 性质
  - get请求一般用于获取数据，不会改变服务器状态
  - post请求一般用于提交数据，会改变服务器的一些状态

- 数据传输的位置
  - get请求，数据会附加在url的末尾
  - post请求将数据作为正文，不会暴露在url中
  - 因此post相对于get请求更加安全
- 数据缓存
  - 浏览器一般会主动缓存get请求，下次相同的操作提升性能
  - post请求不会
- 数据长度
  - get请求参数长度有限制
  - post请求参数的长度没有限制





# 操作系统

## 1.进程和线程的区别

- 进程是操作系统资源分配的基本单位，而线程是任务调度和和执行的最小单元
- 每个进程有了独立的程序上下文，进程中的线程共享所在进程的资源
- 进程之间的切换开销大，线程之间切换的开销小



## 2.操作系统间的进程如何通信？

### 1）管道

![截屏2023-03-23 下午3.09.19](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-03-23 下午3.09.19.png)

- 管道的本质
  - 磁盘中开辟一段大小固定的缓冲区。管道是文件，读写需要调用I/O

- 访问方式
  - 各进程<font color=red>互斥访问</font>。半双工通信，只能单向传输，如果需要实现双向就需要多设置一个管道。
- 读、写表现
  - 如果正在写入，则需要写满才能被读。如果正在读，也需要读完才能写。信息一但被读出，就马上从管道中抛弃。当管道写满时，写进程继续写会<font color=red>阻塞</font>。管道为空，读进程会<font color=red>阻塞</font>。
- 分类
  - 无名管道(pipe)：只能父子或兄弟进程使用
  - 有名管道(FIFO)：不相关的进程也能通过有名管道交换数据

- 优点：简单
- 缺点：效率低

### 2）共享内存

- 每个进程都有属于它自己的PCB(进程控制块)和地址空间(逻辑地址)，并且都有一个与之对应的页表，页表把进程的逻辑地址映射到内存的物理地址中。两个进程不同的虚拟地址映射到物理空间的同一片地址，那么这片地址就是这两个进程的共享内存。

- 共享内存会维护一个计数器，当进程脱离这片共享内存时，计算器会减一，反之，如果新增一个进程挂接到这片共享内存的话，计数器会加一。当计算器为0时，这片共享内存才能被删除。
- <font color=red>进程间的通信需要通过共享空间，不能直接互相通信</font>

![截屏2023-03-23 下午3.47.39](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-03-23 下午3.47.39.png)

使用<font color=red>信号量</font>来进行同步与互斥

- 优点：<font color=red>速度最快的通信方法</font>
- 缺点：缺乏同步机制

### 3）信号量

本质就是一个计数器，用来<font color=red>实现进程之间的互斥与同步</font>。

信号量定义了两种操作，p操作和v操作，p操作为申请资源，会将数值减去M，表示这部分被他使用了，其他进程暂时不能用。v操作是归还资源操作，告知归还了资源可以用这部分。

### 4）消息队列

消息是存储在内核中的消息**链表**，遵循队列的**先进先出**原则。

![截屏2023-03-23 下午4.28.20](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-03-23 下午4.28.20.png)

- 优点：不同于管道，更加灵活的读写方式

- 缺点：消息队列存在于内核中，进程的读写要在用户态于内核态频繁切换，系统开销大。不适合传输较大的数据，因为每个消息块有大小限制

  

### 5）信号

一旦出现系统资源紧张等问题就会告知开发或运维人员，对应到操作系统中，这就是信号。在操作系统中，不同信号用不同的值表示，每个信号设置相应的函数，一旦进程发送某一个信号给另一个进程，另一进程将执行相应的函数进行处理。也就是说先把可能出现的异常等问题准备好，一旦信号产生就执行相应的逻辑即可。

### 6）socket

套接字是一种全双工的通信方式，可以在不同主机之间进行通信，是网络编程中常用的通信方式之一。



## 3.操作系统如何预防死锁

- 产生死锁的四个条件：

  - 互斥条件：该资源任意一个时刻只能有一个线程占有
  - 请求与保持条件：一个线程因请求资源而堵塞，不释放已有的资源

  - 不可剥夺条件：一个线程已获得的资源在使用完之前不可被其他线程剥夺，只有使用完了才释放

  - 循环等待条件：若干线程之间形成一种头尾相连的循环等待关系

- 预防死锁：
  1. 预防死锁：在程序设计中，避免不必要的资源请求和保持、避免循环等待等。
  2. 避免死锁：通过资源分配策略来避免死锁的发生，如银行家算法。
  3. 检测死锁：周期性的检测系统状态，判断死锁是否发生并及时处理。
  4. 解除死锁：一但检测到死锁，采取抢占资源、回滚进程、终止进程等方式解决死锁。



## 4.$\textcolor{orange}{对I/0多路复用的理解}$

- IO
  - 操作系统中，`数据在内核态和用户态之间的读、写操作`。

- 多路
  - `多个TCP连接(多个Socket或者多个Channel)`

- 复用
  - `一个或多个线程资源`

- IO多路复用
  - <font color=red>一个或多个线程处理多个TCP连接</font>，无需创建和维护过多的进程/线程

### select

方式:轮询+遍历

在客户端操作服务器时，会创建3种文件描述符(file descriptor)【写描述符、读描述符、异常描述符】。select会阻塞监视这三种描述符，等到有数据可读、可写、异常时返回，遍历fdset找到就绪的fd，触发相应的IO操作。

- 优点
  - 跨平台支持性好
- 缺点
  - FD增多导致性能下降，且操作系统对单个进程打开的FD数量是有限制的，默认1024

### poll

与select类似，也采用了轮询+遍历。

区别在于select使用数组来存储fd，而poll使用了链表，因此没有最大数量的限制。但是仍然会因为FD的增多而导致性能下降。

### epoll(基于事件驱动)

方法:采用时间通知机制

没有FD个数的限制，从用户态拷贝到内核态只需要一次

<font color=red>将轮询改成了回调</font>，提高了CPU执行效率，<font color=red>也不会随FD数量增加而导致效率下降</font>，但是只能在`linux`下工作

|            |       select       |       poll       |                       epoll                       |
| :--------: | :----------------: | :--------------: | :-----------------------------------------------: |
|  数据结构  |        数组        |       链表       |                       B+树                        |
| 最大连接数 |        1024        |      无上限      |                      无上限                       |
|   FD拷贝   | 每次调用select拷贝 | 每次调用poll拷贝 | FD首次调用epoll_ctl拷贝，每次调用epoll_wait不拷贝 |
|  工作效率  |     轮询:o(n)      |    轮询:o(n)     |                     回调:o(1)                     |
