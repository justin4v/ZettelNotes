#Cache #Redis

# 背景
- **[Redis](https://redis.io/)** 全称为 **Remote Dictionary Server**（远程词典服务）;
- Redis 作者是一名意大利程序员 Sanfilippo（网名：antirez），早年任职于 LLOOGG.com;
- LLOOGG.com 当时使用 **MySQL** 数据库， MySQL 每次推入和弹出都要进行*硬盘写入和读取*，程序的**性能严重受制于磁盘 I/O**；
- antirez 决定写一个具有*列表结构的内存数据库原型*：
	- 原型支持**复杂度为 O(1) 的推入和弹出**操作；
	- 将数据储存在内存而不是硬盘，程序的**性能不会受到硬盘 I/O 限制**。
	- antirez 后用 **C 语言** 重写了内存数据库，并加上了**持久化功能**。

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
## SDS 简单动态字符串
![[Redis SDS.png]]


1.  SDS 中 len 保存这字符串的长度，O(1) 时间复杂度查询字符串长度信息。
2.  空间预分配：SDS 被修改后，程序不仅会为 SDS 分配所需要的必须空间，还会分配额外的未使用空间。
3.  惰性空间释放：当对 SDS 进行缩短操作时，程序并不会回收多余的内存空间，而是使用 free 字段将这些字节数量记录下来不释放，后面如果需要 append 操作，则直接使用 free 中未使用的空间，减少了内存的分配。

## zipList 压缩列表
![[Redis Ziplist.png]]

当一个列表只有少量数据的时候，并且每个列表项要么就是小整数值，要么就是长度比较短的字符串，那么 Redis 就会使用压缩列表来做列表键的底层实现




# 参考
1. [How fast is Redis? ](https://redis.io/topics/benchmarks)
2. [Redis 核心原理与实战总结系列](https://xie.infoq.cn/article/d8b459da4820c5862b626388e)
3. [Redis 核心知识点归纳总结](https://xie.infoq.cn/article/4f528d0db782b01e663cf6d56)