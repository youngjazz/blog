---
title: Redis一步站式学习
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

采用C语言编写，纯内存，非阻塞IO，单线程避免线程切换和竞态消耗

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

![image-20181125145435592](/Users/leon/Library/Application Support/typora-user-images/image-20181125145435592.png)

redis-cli 查看配置 config get *



### 通用明亮

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

![image-20181127161855538](/Users/leon/Library/Application Support/typora-user-images/image-20181127161855538.png)

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

### RDB

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

### 持久化的第二种方式AOF

RDB有两个问题：

1.耗时、耗性能

2.不可控、丢失数据

![](https://ws4.sinaimg.cn/large/006tNbRwly1fxowzv4rnoj30qk0hk44f.jpg)

AOF：redis会将每一个收到的写命令都通过write函数追加到文件中(默认是 appendonly.aof)

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

## Redis复制原理

### 主从复制

单机有什么问题？机器故障、容量瓶颈、QPS瓶颈

主从复制作用：数据副本、扩展读性能

**实现主从：**

1.slaveOf命令 `redis-6380> slaveof 127.0.0.1 6379`

取消`redis-6380> slaveof no one`

无需重启，但不便于管理

2.配置：slaveof ip port

slave-read-only yes

统一配置、需要重启