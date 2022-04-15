#Cache #Redis

# 背景
- **[Redis](https://redis.io/)** 全称为 **Remote Dictionary Server**（远程词典服务）;
- Redis 作者是一名意大利程序员 Sanfilippo（网名：antirez），早年任职于 LLOOGG.com;
- LLOOGG.com 当时使用 **MySQL** 数据库， MySQL 每次推入和弹出都要进行*硬盘写入和读取*，程序的**性能严重受制于硬盘 I/O**；
- antirez 决定写一个具有*列表结构的内存数据库原型*：
	- 原型支持**复杂度为 O(1) 的推入和弹出**操作；
	- 将数据储存在内存而不是硬盘，程序的**性能不会受到硬盘 I/O 限制**。
	- antirez 使用 **C 语言** 重写了这个内存数据库，并给它加上了持久化功能

# Redis速度
![[Redis的QPS随连接数变化.png]]

- Redis 的 QPS（Quest Per Second，每秒请求数） 可以达到约 100000；
- 横轴为连接数，纵轴为 QPS。

# 实现
- Redis 基于内存实现，直接与 CPU 通信，速度远高于磁盘数据库；
- Redis 读写都在内存中完成。

磁盘调用过程：

![[磁盘调用过程.png]]

## 速度对比
计算机中各种系统通信的延迟时间如下：

![[计算机中各种延迟时间示意.png]]

# 数据结构
1. Redis 有 5 种**数据类型**，`String、List、Hash、Set、SortedSet`。
2. 不同的数据类型底层有**一种或者多种数据结构实现**，不同情况下使用不同数据结构实现；
3. 设计多种数据结构实现的目的是*追求更快的速度*。

Redis *数据类型和底层数据结构的关系*如下：
![[Redis数据类型和底层数据结构的关系.png]]



# 参考
1. [How fast is Redis? ](https://redis.io/topics/benchmarks)
2. [Redis 核心原理与实战总结系列](https://xie.infoq.cn/article/d8b459da4820c5862b626388e)
3. [Redis 核心知识点归纳总结](https://xie.infoq.cn/article/4f528d0db782b01e663cf6d56)