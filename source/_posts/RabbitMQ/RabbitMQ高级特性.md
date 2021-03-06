---
title: RabbitMQ高级特性
date: 2019-01-06 00:09:08
tags: RabbitMQ
categories: 技术
---

# Rabbit高级特性



## 生产端-消息如何保障100%投递成功？

### 什么是生产端可靠性投递？

- 保障消息成功发送
- 保障MQ节点成功接收
- 发送端收到MQ节点（Broker）确认应答
- 完善的消息进行补偿机制

**互联网大厂解决复方案：**

> 方式一：消息落库，对消息状态进行打标（最大努力尝试）

![](https://ws4.sinaimg.cn/large/006tNc79ly1fyw5pht4uxj30qj0d7djb.jpg)

==在高并发的场景下，第一种方式是否合适？==



> 方式二： 消息的延迟投递，做二次确认，回调检查

![](https://ws4.sinaimg.cn/large/006tNc79ly1fywqd7j8h9j30q40d9acm.jpg)

上述的step4不同于以往的消息ack， 而是发送一条新的消息，callbackService会监听这条消息

延迟投递的消息step2与step发送到的不是同一队列，延迟投递的消息也需要callbackService来监听；如果check消息确认成功了，不做任何事情，如果失败了，会去执行resend，进行补偿。

这样做可以**少落一次库**，提升并发性能；同时业务与消息**解耦**；

注意两点：

1.一定要先业务数据落库，再发消息。

2.互联网大厂不加任何事务，事务会成为严重的性能瓶颈。



## 消费端-幂等性如何保障？

> 消费端实现幂等，就意味着我们的消息永远不会被消费多次，即使收到多条一样的消息。

**业界主流的幂等操作：**

1. **唯一ID+指纹码 机制**

    - 唯一ID+指纹码 机制， 利用数据库主键去重

    - SELECT COUNT(1) FROM T_ORDER WHERE ID=唯一ID+指纹码
    - 优点：实现简单
    - 缺点：高并发下有数据库写入性能瓶颈
    - 解决方案：根据ID进行分库分表进行算法路由

2. **利用Redis院子特性实现**
    - 使用Redis进行幂等需要考虑的问题
    - 第一： 我们是否进行数据落库， 如果落库的话，关键解决的问题是数据库和缓存如何做到原子性？
    - 第二： 如果不进行落库， 那么都存储到缓存中，如何设置定时同步策略？

## confirm 确认消息

