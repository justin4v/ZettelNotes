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
1、Point-to-Point Messaging Domain （点对点）

　　　　　　2、Publish/Subscribe Messaging Domain （发布/订阅模式）

　　　在JMS API出现之前，大部分产品使用“点对点”和“发布/订阅”中的任一方式来进行消息通讯。JMS定义了这两种消息发送模型的规范，它们相互独立。任何JMS的提供者可以实现其中的一种或两种模型，这是它们自己的选择。JMS规范提供了通用接口保证我们基于JMS API编写的程序适用于任何一种模型。