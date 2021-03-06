#JMS #Message-queue  #Publish-subscribe 
# 概念
JMS 即 **Java消息服务（Java Message Service）**，是Java平台中面向消息中间件（MOM）的 *API*

用于在两个应用程序之间，或分布式系统中发送消息，进行**异步通信**。

# 优点
1.  **Asynchronous**（异步）
    JMS 默认是一个异步的消息服务，客户端不需要主动请求消息，消息会自动发送给可用的客户端
    
2.   **Reliable**（可靠）
    JMS保证消息只会递送一次
	
# JMS编程模型　

1.  管理对象（Administered objects）-连接工厂（Connection Factories）和目的地（Destination）
2.  连接对象（Connections）
3.  会话（Sessions）
4.  消息生产者（Message Producers）
5.  消息消费者（Message Consumers）
6.  消息监听者（Message Listeners）

![[JMS 编程模型.png]]

## JMS管理对象
管理对象（Administered objects）是**预先配置的JMS对象**，由系统管理员为了使用 JMS 的客户端创建，主要有两个被管理的对象：

-   连接工厂（ConnectionFactory）
-   目的地（Destination）

由JMS系统管理员通过使用 Application Server 管理控制台创建，存储在应用程序服务器的 *JNDI namespace 或 JNDI 注册表*。

## Connection Factories

1. `QueueConnectionFactory` 适用于[[#点对点消息模型]]；
2. `TopicConnectionFactory` 适用于 [[#发布 订阅消息模型]];
3. **通过 [[JNDI]] 查找 ConnectionFactory 对象。**

客户端创建一个连接对象连接到 JMS 服务提供者。
JMS 客户端（如发送者或接受者）在 [[JNDI]] namespace 中查找并获取该连接。

使用该连接，客户端能够与目的地通讯，往队列或话题发送/接收消息。

```java
QueueConnectionFactory queueConnFactory = (QueueConnectionFactory) initialCtx.lookup ("primaryQCF");
Queue purchaseQueue = (Queue) initialCtx.lookup ("Purchase_Queue");
Queue returnQueue = (Queue) initialCtx.lookup ("Return_Queue");
```

## Destination

Destination 指明*消息被发送的目的地以及客户端接收消息的来源*。
JMS 使用两种目的地：*queue和 topic*。如下代码指定了一个队列：

**创建一个队列Session**
```java
QueueSession ses = con.createQueueSession (false, Session.AUTO_ACKNOWLEDGE);  //get the Queue object 
Queue t = (Queue) ctx.lookup ("myQueue");  //create QueueReceiver 
QueueReceiver receiver = ses.createReceiver(t); 
```

**创建一个话题Session**
```java
TopicSession ses = con.createTopicSession (false, Session.AUTO_ACKNOWLEDGE); // get the Topic object 
Topic t = (Topic) ctx.lookup ("myTopic");  //create TopicSubscriber 
TopicSubscriber receiver = ses.createSubscriber(t);
```

## Connection

*Connection 表示在客户端和 JMS 系统之间建立的链接（对TCP/IP socket的包装）*。
Connection可以产生一个或多个Session。跟ConnectionFactory 一样，Connection也有两种类型：QueueConnection 和 TopicConnection。

连接对象封装了与JMS提供者之间的虚拟连接，如果我们有一个ConnectionFactory对象，可以使用它来创建一个连接。
```java
Connection connection = connectionFactory.createConnection();
```

创建完连接后，需要在程序使用结束后关闭它：
```java
connection.close();
```

## Session

*Session是一个单线程 Context，Session 是我们对消息进行操作的接口*，可以通过session *创建生产者、消费者、消息等*。

Session 提供了事务的功能，如果需要使用session发送/接收多个消息时，可以将这些发送/接收动作放到一个事务中。

我们可以在连接创建完成之后创建session：
```java
Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
```

这里面提供了参数两个参数，第一个参数是是否支持事务，第二个是事务的类型

## Producter

*消息生产者由Session创建，用于往目的地发送消息。*
生产者实现MessageProducer接口，我们可以为目的地、队列或话题创建生产者；

```java
MessageProducer producer = session.createProducer(dest);
MessageProducer producer = session.createProducer(queue);
MessageProducer producer = session.createProducer(topic);
```

## Consumer

*消息消费者由Session创建，用于接收被发送到Destination的消息。*
```java
MessageConsumer consumer = session.createConsumer(dest);
MessageConsumer consumer = session.createConsumer(queue);
MessageConsumer consumer = session.createConsumer(topic);
```

## MessageListener

*如果注册了消息监听器，一旦消息到达，将自动调用监听器的 onMessage 方法*。

```java
Listener myListener = new Listener();
consumer.setMessageListener(myListener);
```

EJB中的MDB（Message-Driven Bean）就是一种MessageListener。


# JMS消息结构

JMS客户端使用JMS消息与系统通讯，JMS消息虽然格式简单但是非常灵活， JMS消息由三部分组成：

## 消息头

JMS消息头预定义了若干字段用于**客户端与JMS提供者之间识别和发送消息**，如下：

- JMSDestination 
- JMSDeliveryMode  
- JMSMessageID  
- JMSTimestamp  
- JMSCorrelationID  
- JMSReplyTo  
- JMSRedelivered  
- JMSType  
- JMSExpiration  
- JMSPriority

## 消息属性

可以给消息*设置自定义属性*，可用于实现*消息过滤功能*，消息属性非常有用，JMS API定义了一些*标准属性*，JMS服务提供者可以选择性的提供部分标准属性。

## 消息体

在消息体中，JMS API定义了五种类型的消息格式：

**Text message** : javax.jms.TextMessage，表示一个文本对象。  
**Object message** : javax.jms.ObjectMessage，表示一个JAVA对象。  
**Bytes message** : javax.jms.BytesMessage，表示字节数据。  
**Stream message** :javax.jms.StreamMessage，表示java原始值数据流。  
**Map message** : javax.jms.MapMessage，表示键值对。


## 应用
常见的开源JMS服务的提供者：
-   JBoss 社区所研发的 HornetQ
-   Joram
-   Coridan的MantaRay
-   The OpenJMS Group的OpenJMS


# 参考
1. [[消息模型]]