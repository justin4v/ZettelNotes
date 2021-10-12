# 概念
JMS 即 **Java消息服务（Java Message Service）**，是Java平台中面向消息中间件（MOM）的 *API*

用于在两个应用程序之间，或分布式系统中发送消息，进行**异步通信**。

# 优点
1.  **Asynchronous**（异步）
    JMS 默认是一个异步的消息服务，客户端不需要主动请求消息，消息会自动发送给可用的客户端
    
2.   **Reliable**（可靠）
    JMS保证消息只会递送一次
	
	
# 消息模型
JMS 定义了**消息发送模型的规范**：
1. Point-to-Point Messaging Domain （点对点）
2. Publish/Subscribe Messaging Domain （发布/订阅模式）

## 点对点消息模型
Point-to-Point Messaging Domain

![[p2p 消息模型.png]]

### 涉及到的概念
1. 程序由消息队列，发送方，接收方组成。
2. 每个消息都被发送到一个特定的队列，接收者从队列中获取消息。
3. 队列保留消息直到他们被消费或超时。

### 特点

-    **每个消息只要一个消费者**；
-    发送者发送消息到消息队列和接收者接收消息是**独立的**；
-    接收方在接收完消息之后，要向消息队列应答。

## 发布/订阅消息模型
Publish/Subscribe Messaging Domain

![[发布-订阅模型.png]]

### 涉及到的概念：

- 发布者发布一个消息，该消息通过 topic 传递给所有的客户端；
- 发布者与订阅者相互之间是无感知的；
- 可以动态的发布与订阅 Topic。
- Topic主要用于保存和传递消息，且会保存消息直到消息被传递给客户端。

### 特点：

-  **一个消息可以传递给多个订阅者**（即：一个消息可以有多个接受方）。
-   **发布者与订阅者具有时间约束**，针对某个主题（Topic）的订阅者，它必须创建一个订阅者之后，才能消费发布者的消息，而且为了消费消息，订阅者必须保持运行的状态。
-    为了缓和这样严格的时间相关性，JMS **允许订阅者创建一个可持久化的订阅**。即使订阅者没有被激活（运行），它也能接收到发布者的消息。


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

这两个管理对象由JMS系统管理员通过使用 Application Server 管理控制台创建，存储在应用程序服务器的 JNDI namespace 或JNDI注册表。

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

*如果注册了消息监听器，一旦消息到达，将自动调用监听器的onMessage方法*。

```java
Listener myListener = new Listener();
consumer.setMessageListener(myListener);
```

EJB中的MDB（Message-Driven Bean）就是一种MessageListener。