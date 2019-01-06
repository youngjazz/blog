---
title: JVM-类文件结构
date: 2018-10-15 21:59:07
tags: JVM
categories: 技术

---

- Class文件是一组以8位字节为基础单位的二进制流，各个数据项严格的按照顺序排列在Class文件中，中间没有任何的分隔符，整个Class文件中存储的内容几乎全部是程序运行的必要数据，必有空隙存在。
- 当遇到8位以上的空间数据项时，则会按照高位在前的方式分割成若干8位字节进行存储。
- Class文件中有两种数据类型，分别是无符号数和表。

包括：
- 魔数
- Class文件版本
- 常量池
- 访问标志
- 类索引、父类索引，接口索引集合
- 字段表集合
- 方法表集合
- 属性表集合

## Class文件设计理念以及意义
运行在JVM之上的语言： Clojure、Groovy、Jruby、Jpython、Scala...

魔性配图
![-w781](http://p7b5cwgjy.bkt.clouddn.com/15405253424245.jpg)



![-w281](http://p7b5cwgjy.bkt.clouddn.com/15405334048549.jpg)

其中依次代表：

![](http://p7b5cwgjy.bkt.clouddn.com/15405214476706.jpg)

看起来很费力，可以借助工具来阅读`javap -verbose HelloWorld.class`



### 魔数
java的魔数是CAFEBABE




### 常量池
可以使用工具辅助我们读取