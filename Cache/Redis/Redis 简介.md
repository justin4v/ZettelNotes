#Cache #Redis

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


# 参考
1. [How fast is Redis? ](https://redis.io/topics/benchmarks)
2. [Redis 核心原理与实战总结系列](https://xie.infoq.cn/article/d8b459da4820c5862b626388e)
3. [Redis 核心知识点归纳总结](https://xie.infoq.cn/article/4f528d0db782b01e663cf6d56)