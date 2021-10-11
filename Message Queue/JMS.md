# 概念
JMS 即 **Java消息服务（Java Message Service）**，是Java平台中面向消息中间件（MOM）的 *API*

用于在两个应用程序之间，或分布式系统中发送消息，进行**异步通信**。

# 优点
1.  Asynchronous（异步）
    JMS 原本就是一个异步的消息服务，客户端获取消息的时候，不需要主动发送请求，消息会自动发送给可用的客户端
    
2.   Reliable（可靠）
    JMS保证消息只会递送一次。大家都遇到过重复创建消息问题，而JMS能帮你避免该问题。
	
	
# 消息模型
两种通信模式：
1. Point-to-Point Messaging Domain （点对点）
2. Publish/Subscribe Messaging Domain （发布/订阅模式）

JMS 定义了这两种消息发送模型的规范，它们相互独立。

## Point-to-Point Messaging Domain

![[p2p 消息模型.png]]


### 涉及到的概念
在点对点通信模式中，应用程序由消息队列，发送方，接收方组成。每个消息都被发送到一个特定的队列，接收者从队列中获取消息。
队列保留着消息，直到他们被消费或超时。

### 特点

-    每个消息只要一个消费者
-    发送者和接收者在时间上是没有时间的约束，也就是说发送者在发送完消息之后，不管接收者有没有接受消息，都不会影响发送方发送消息到消息队列中。
-    发送方不管是否在发送消息，接收方都可以从消息队列中去到消息（The receiver can fetch message whether it is running or not when the sender sends the message）
-    接收方在接收完消息之后，需要向消息队列应答成功