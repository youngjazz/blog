---
title: Redis一站式学习
date: 2018-11-25 10:45:08
tags: Redis
categories: 技术
---

## Redis特征

- 速度快 

- 持久化

- 多种数据结构

- 支持多种编程语言

- 功能丰富

- 简单

- 主从复制

- 高可用、分布式

    <!--More-->

### 速度快

官方给出的数据是 10W OPS

采用C语言编写；纯内存；数据结构简单，对数据的操作也简单；使用IO多路复用，非阻塞IO；单线程避免线程切换和竞态消耗

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g09p9qx0dej31jy0rsanz.jpg)



![](https://ws1.sinaimg.cn/large/006tNbRwly1fxk4bx1b65j31480tmk3h.jpg)

| 类型     | 每秒读写次数 | 随机读写延迟 | 访问带宽  |
| -------- | ------------ | ------------ | --------- |
| 内存     | 千万级       | 80ns         | 5GB       |
| SSD      | 35000        | 0.1-0.2ms    | 100-300MB |
| 机械硬盘 | 100左右      | 10ms         | 100MB左右 |

### 持久化

Redis所有数据保存在内存中，对数据的更新将一步地保存到磁盘中。

### 多种数据结构

主要数据结构有5种：

![](https://ws2.sinaimg.cn/large/006tNbRwly1fxk4j8d2ikj310m0g2grs.jpg)

在此之上延伸出：

- BitMap 位图
- HyperLogLog 超小内存唯一值计数
- GEO 地理信息定位

### 多客户端支持

由于Redis基于TCP的通讯协议使得Redis的客户端非常之多 Java、PHP、Python、 Ruby、Lua、NodeJs等等，这也是Redis如此受欢迎的一个原因。

### 功能丰富

- 发布订阅
- Lua脚本，可以实现自定义的一些功能
- 事务
- pipeline， 提升客户端的并发效率 

### 简单

单机版Redis核心代码只有2万多行，查阅源码会相对简单；同时不依赖外部代码；单线程模型， 服务端或客户端的开发会相对容易

### 主从复制

![](https://ws2.sinaimg.cn/large/006tNbRwly1fxk4uh57b7j30nm0j40y5.jpg)

### 高可用、分布式

对于一个单点或者主从复制模型来实现高可用本身是很难的。从Redis2.8版本开始，通过引入Redis-Sentinel来支持高可用。从Redis3.0版本，通过Redis-Cluster来支持分布式



### 典型使用场景

- 缓存系统
- 计数器
- 消息队列系统
- 排行榜功能
- 社交网络
- 实时系统

### 安装

![image-20181125145435592](https://ws4.sinaimg.cn/large/006tKfTcly1g09p2ilb5cj31oo0iqk03.jpg)

redis-cli 查看配置 config get *



### 通用

- keys
- dbsize
- exists key
- del key [key...]
- expire key seconds
- type key

### 数据结构与内部编码

![](https://ws1.sinaimg.cn/large/006tNbRwly1fxkcjjv6itj30u00uigzu.jpg)

## 慢查询

![](https://ws1.sinaimg.cn/large/006tNbRwly1fxmnbi5acdj318c0r0wn6.jpg)

- 整个生命周期，慢查询发生在第三阶段
- 而客户端超时的原因不一定是因为慢查询，所有阶段都有可能

### 两个配置- slowlog-max-len（默认128）

![](https://ws4.sinaimg.cn/large/006tNbRwly1fxmnfonrq3j31420igmzy.jpg)

1.先进先出队列

2.固定长度

3.保存在内存中

### 两个配置- slowlog-log-slower-than（默认10000）

1.慢查询阈值（单位：微秒），默认10ms， 通常设置为1ms

2.slowlog-log-slower-than=0，记录所有命令

3.slowlog-log-slower-than<0，不记录

#### 慢查询命令

1.slowlog get [n] ：获取慢查询队列

2.slowlog len ：获取慢查询队列长度

3.slowlog reset：清空慢查询队列



## Pipeline

一次普通请求时间=一次网络时间+一次命令时间

一次pipeline时间=一次网络时间+n次命令时间

![](https://ws2.sinaimg.cn/large/006tNbRwly1fxmnzahik1j31da0majue.jpg)

redis命令是微秒级，往往瓶颈是在网络传输上。

```java
Jdeis jedis = new Jedis("127.0.0.1",6379);
for(int i=0;i<100;i++){
    Pipeline pipeline = jedis.pipelined();
    for(int j = i*100;j<(i+1)*100;j++){
        pipeline.hset("hashkey"+j,"field"+j, "value"+j)
    }
    pipeline.syncAndReturnAll();
}
```

pipeline与原生M操作的区别：

M操作是一个原子操作，而pipeline到了redis服务端还是会拆分pipeline子命令与其他操作，然后按顺序返回，并不是一个原子操作。

- 注意每次pipeline携带的数据量

- pipeline每次只能作用在一个Redis节点上



## 发布订阅

![](https://ws4.sinaimg.cn/large/006tNbRwly1fxmodd6r3sj31ao0k20w3.jpg)

## 	消息队列

![](https://ws3.sinaimg.cn/large/006tNbRwly1fxmohtliowj31aw0ig41o.jpg)

二者场景不同



## BitMap 位图

![](https://ws4.sinaimg.cn/large/006tNbRwly1fxmopwbl04j30w209uwfc.jpg)

redis是可以直接操作位的

![image-20181127161855538](https://ws3.sinaimg.cn/large/006tKfTcly1g09p3bwv0aj319c0o0aj7.jpg)

![](https://ws3.sinaimg.cn/large/006tNbRwly1fxmp0fdoggj319q0ba75t.jpg)

#### HyperLogLog

- 基于HyperLogLog算法：极小空间独立数量统计
- 本质还是字符串，并不是新的数据结构

- pfadd pfcount pfmerge
- 官方给出的容错率0.81%

### GEO

type geoKey = zset可以看出geo的实现是zset



## 持久化

谈到持久化，市场上已经有很多产品有自己的持久化方式，大致两个方式

1.快照，如MySQL Dump, Redis RDB

2.写日志， 如MySQL Binlog, Hbase Hlog, Redis AOF

### RDB(快照，保存某个时间点全量数据)

![](https://ws4.sinaimg.cn/large/006tNbRwly1fxmpw74zfuj30vw0i4di0.jpg)

**RDB触发机制**：

- save（同步）
- bgsave（异步）
- 自动，通过配置规则，创建的过程是使用了bgsave

```bash
127.0.0.1:6379> save
OK
```

client发送save命令即可生成RDB二进制文件，但是这个阻塞的命令；如果老的RDB文件存在，新的替换老的。O(N)的复杂的。

![](https://ws2.sinaimg.cn/large/006tNbRwly1fxmq4jvmt7j317g0j0ju3.jpg)

子进程fork()在极少情况下也会阻塞redis，也是O(N)的复杂的

通过配置

```bash
#一般关闭自动save
#save 900 1
#save 300 10
#save 60 10000
dbfilename dump-${port}.rdb
dir /bigdistpath
stop-writes-on-bgsave-error yes
rdbcompression yes
```

**触发机制-不可忽略的方式**

1.全量复制

2.debug reload

3.shutdown

**RDB有两个问题**：

1.全量同步，数据量大时I/O耗时、耗性能

2.redis宕机会丢失上一次备份时间到当前期间的数据

### 持久化的第二种方式AOF（Append-Only-File保存写状态）



![](https://ws4.sinaimg.cn/large/006tNbRwly1fxowzv4rnoj30qk0hk44f.jpg)

AOF：redis会将每一个收到的写命令都通过write函数追加到文件中(默认是 appendonly.aof)

==AOF备份的是接受到的指令==

**AOF三种策略：always，everysec，no**

redis的写命令实际上是先到缓冲区，从缓冲区到硬盘的过程就有了区别：

![](https://ws1.sinaimg.cn/large/006tNbRwgy1fxq0opqwj3j31xk0iadip.jpg)

![](https://ws3.sinaimg.cn/large/006tNbRwgy1fxq0p3bddoj31xc0j2776.jpg)

![](https://ws2.sinaimg.cn/large/006tNbRwgy1fxq0qge4xhj31xo0iutby.jpg)

| 命令 | always                               | everysec      | no     |
| ---- | ------------------------------------ | ------------- | ------ |
| 优点 | 不丢失数据                           | 每秒一次fsync | 不用管 |
| 缺点 | IO开销较大，一般的stata盘只有几百TPS | 丢一秒的数据  | 不可控 |

随着AOF文件的逐步变大，就会暴露出很多的问题。比如使用AOF恢复数据的时候就会很慢，同时也会带来比加大的存储压力。

#### AOF重写

为了解决AOF文件膨胀的问题，Redis服务器可以创建一个新的AOF文件，新旧两个文件所保存的数据库状态是相同的，但是新的AOF文件不会包含任何浪费空间的冗余命令，通常新的AOF文件会小很多。

```bash
rpush list "A" "B"
rpush list "C"
rpush list "D" "E"
lpop list
lpop list
rpush list "F" "G"
rrange list 0 -1
```

执行这几条命令，后数据库中list的值为`["C","D","E","F","G"]` ，最简单的方式不是去读取和分析现在的AOF文件内容，而是直接读取list数据库中的内容，然后用一条`RPUSH list "C" "D" "E" "F" "G"`代替上面的命令。



**AOF重写的两种方式：**

- bgrewriteaof

![](https://ws3.sinaimg.cn/large/006tNbRwgy1fxq208c554j31do0lcgny.jpg)

- AOF重写配置

    | 配置名                     | 含义                  |
    | -------------------------- | --------------------- |
    | auto-aof-rewrite-min-size  | AOF文件重写需要的尺寸 |
    | auto-aof-rewite-percentage | AOF文件增长率         |

    ![](https://ws3.sinaimg.cn/large/006tNbRwgy1fxq26p0csvj30l40me0u5.jpg)


**AOF重写的作用**

1.减少磁盘占用量

2.加速磁盘恢复速度

### RDB和AOF的抉择

| 命令       | RDB    | AOF          |
| ---------- | ------ | ------------ |
| 启动优先级 | 低     | 高           |
| 体积       | 小     | 大           |
| 恢复速度   | 快     | 慢           |
| 数据安全性 | 丢数据 | 根据策略决定 |
| 轻重       | 重     | 轻           |

**RDB最佳策略：关；集中处理；主从时，从开**

**AOF最佳策略：“开”：缓存和存储；AOF重写集中管理；everysec**

**通用：小分片；缓存或者存储；监控（硬盘、内存、负载、网络）；足够的内存**

### 子进程开销与优化

1.CPU

- 开销：RDB与AOF文件生成，属于CPU密集型
- 优化：不做CPU绑定，不和CPU密集型部署

2.内存

- 开销： fork内存开销，copy-on-write
- 优化： echo never > /sys/kernel/mm/transparent_hugepage/enable

3.硬盘

- 开销：AOF和RDB文件写入，可以结合iostat，iotop分析

- 优化：

    - 不要和高硬盘负载的服务部署在一起：存储服务、消息队列等
    - no-appendfsync-on-rewrite = yes
    - 根据写入量决定磁盘类型：例如ssd
    - 单机多实例持久化文件目录可以考虑分盘

    ### AOF阻塞

    ![](https://ws4.sinaimg.cn/large/006tNbRwgy1fy1xr50aztj30u00vlwna.jpg)

    如何定位AOF阻塞？

    1.Redis日志

    2.info Persistence

    3.top命令，看io资源是否紧张



### RDB-AOF混合持久化方式

- bgsave做全量备份，aof做增量备份

## Redis数据恢复

RDB与AOF文件共存的情况下，会加载AOF，否则加载RDB



## Redis同步机制

### 主从复制

单机有什么问题？机器故障、容量瓶颈、QPS瓶颈

主从复制作用：数据副本、扩展读性能

#### 全同步过程：

- slava发送sync命令到master
- master启动一个后台进程，将redis中的数据快照保存到文件中
- master将保存数据快照期间接受到的写命令缓存起来
- master完成写文件操作后，将该文件发送给slave
- 使用心得AOF文件替换旧的AOF文件
- master将这期间收集的增量写命令发送给slave端

#### 增量同步过程：

- master接受到用户操作指令，判断是否需要传播至slave（增删改需要）
- 将操作记录追加到AOF文件
- 将操作传播到其他slave：1.对齐主从库；2.往缓存写入指令
- 将混存中的数据发送给slave

**实现主从：**

1.slaveOf命令 `redis-6380> slaveof 127.0.0.1 6379`

取消`redis-6380> slaveof no one`

无需重启，但不便于管理

2.配置：slaveof ip port

slave-read-only yes

统一配置、需要重启

#### 主从模式的弊端

不具备高可用性，当master挂掉之后redis将不能对外提供写入操作，于是需要借助其他手段实现高可用。如Redis Sentinel

#### Redis Sentinel

**解决主从同步master宕机后主从切换问题：**

- 监控：检查主从服务器是否运行正常
- 提醒：通过API向管理员或者其他应用程序发送故障通知
- 自动故障迁移：主从切换（当检测到master宕机，sentinel会将从服务之一升级为主服务器，并让其他slave改为复制新的master，当客户端试图连接失效的master，sentinel集群也会向客户端返回新master地址，sentinel集群进程间使用流言协议来接收主服务器是否下线的信息，使用投票协议决定是否进行自动故障迁移，以及决定哪个从服务器作为新的主服务器）

#### 流言协议Gossip

反熵：**在杂乱无章中寻求一致**

- 每个节点都是随机与对方通信，最终所有节点的状态达成一致
- 种子节点定期随机向其他节点发送节点列表以及需要传播的消息
- 不保证信息一定传递到所有节点，但最终会趋于一致



## Redis集群原理

### 如何从海量数据中找到所需？

- 分片：按照某种规则去划分数据，分散存储在多个节点上
- 常规的按照哈希划对分（对服务器数量取模）无法实现节点的动态增减

为此，redis引入了**一致性哈希算法：**对2^32取模，将哈希值空间组织成虚拟的圆环

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0arphqmdvj30eq0fk41p.jpg)

将数据key使用相同的Hash函数计算出哈希值。假设使用ip哈希之后再环空间上的位置如下;

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0artbqvssj30f20fkn11.jpg)

当要存储新的数据时，也先计算出hash值，对应换上的位置，存在顺时针方向离他最近的节点上。

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g0aruzk2iij30fg0fq0yb.jpg)

这样我们就可以将A，B，C，D这个四个数据分散储存到四台不同的服务器上了。

现在假设Node C宕机了

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g0arz3xex8j30f60fqq8v.jpg)

此时A，B，D并不会受影响，C会重新定位到Node D里面去。由此可知，如果一台服务器发生故障，受影响的是宕机节点按逆时针方向行走遇到的第一台服务器之间的数据。最小化有损服务。

如果增加一台节点X

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0as3wletjj30f00fqwkl.jpg)

影响的是新节点X环空间到节点B环空间之间的数据

**Hash环数据倾斜问题：**在服务器节点很少的情况下，容易引起数据分布不均，被缓存的对象，大部分会缓存在某一台或某几台服务器上。

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0as7ueyv5j30em0fkdls.jpg)

**如何解决数据倾斜的问题呢？**
一致性哈希算法引入了虚拟节点的机制，即对每一台服务器计算多个hash，计算结果都放置到环中，称之为虚拟节点，可以在服务器ip或者主机名后面添加编号实现

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g0asbbik09j30eq0f4gqf.jpg)

在实际应用中，将虚拟节点设置为32或者更大，即使很少的节点也能实现相对均匀的数据分布，结合redis集群技术，我们还可以在期间引入redis主从同步，sentinel哨兵机制来进一步确保集群的高可用性，这也是主流的做法。



## 应用

### 如何利用redis实现分布式锁

#### 分布式锁需要解决的问题

- 互斥性， 任意时刻只能有一个客户端获取某个锁。
- 安全性， 锁只能被持有该锁的客户端删除，不能被别的客户端删除，
- 死锁， 占有锁的客户端由于其他原因如宕机未能及时释放锁，其他客户端就不能获取到锁而导致死锁，需要有机制来避免。
- 容错，部分节点宕机，客户端仍然能够获取锁和释放锁。

#### SETNX

> SETNX key value: (即set if not exists) 如果key不存在，创建并赋值

- 时间复杂度O（1），并且是原子操作，故而人们利用这一特性早期用在实现分布式锁
- 然后利用EXPIRE key seconds， 当key过期后会被自动删除

伪代码如下；

![](https://ws3.sinaimg.cn/large/006tKfTcly1g09rg6vax8j31420a2k01.jpg)

但是这样是有问题的。。。因为是两部操作，就破坏了原子性。

#### SET

![](https://ws2.sinaimg.cn/large/006tKfTcly1g09rjk42poj31mk0sa1ar.jpg)

`set locktarget 123 ex 10 nx`

![](https://ws2.sinaimg.cn/large/006tKfTcly1g09rkoehl6j314a05a43g.jpg)





### 大量key同时过期问题

![](https://ws1.sinaimg.cn/large/006tKfTcly1g09tf1wyyxj31d20c4h18.jpg)

在key的过期时间上加上一个随机值即可解决



### 从海量数据中获取指定前缀的键，生产可用

`scan 0 match key1* count 10`（返回下一个游标，和具体数据，不一定准确可能会重复，需要客户端自己去重；