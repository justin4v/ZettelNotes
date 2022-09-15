#Redis #Stream #Message-queue

# 简介
- Redis Stream 是 Redis 5.0 版本*新增加的数据结构*。
- Redis Stream 主要用于*消息队列（MQ，Message Queue）*；
- Redis 本身有一个 Redis 发布订阅 (pub/sub) 来实现消息队列的功能：
	- 但消息无法持久化，如果出现网络断开、Redis 宕机等，消息就会被丢弃。
	- 发布订阅 (pub/sub) 可以分发消息，但无法记录历史消息。
- Redis Stream 提供了消息的*持久化和主备复制* 功能，可以让任何客户端访问任何时刻的数据，并且能记住每一个客户端的访问位置，还能保证消息不丢失。


# Stream 结构

![[Redis Stream结构示意.png]]


Redis Stream 的结构如上所示：
- 有一个*消息链表*，将所有加入的消息都串起来；
- 每个消息都有一个唯一的 ID 和对应的内容；
- 每个 *Stream 实例都有唯一的名称，即 Redis 的 key*；

## 相关概念
1.  `消费者组：` Consumer Group，使用 `XGROUP CREATE` 命令创建的，一个消费者组中可以存在多个消费者，这些消费者之间是`竞争`关系。
    1.  同一条消息，只能被这个消费者组中的某个消费者获取。
    2.  多个消费者之间是相互独立的，互不干扰。
2.  `消费者：` Consumer 消费消息。
3.  `last_delivered_id：` 这个id保证了在同一个消费者组中，一个消息只能被一个消费者获取。每当消费者组的某个消费者读取到了这个消息后，这个last_delivered_id的值会往后移动一位，保证消费者不会读取到重复的消息。
4.  `pending_ids`：记录了消费者读取到但还没有处理的消息 id 列表；
5.  `消息内容：`是一个`键值对`的格式。
6.  `Stream 中 消息的 ID：` 默认情况下，ID使用 `*` ，redis可以自动生成一个，格式为 `时间戳-序列号`，也可以自己指定，一般使用默认生成的即可，且后生成的id号要比之前生成的大。




# 参考
1. [Redis入门 - 数据类型：Stream详解 ](https://www.cnblogs.com/pengdai/p/14664214.html)