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
点对点消息模型

![[p2p 消息模型.png]]

### 涉及到的概念
1. 应用程序由消息队列，发送方，接收方组成。
2. 每个消息都被发送到一个特定的队列，接收者从队列中获取消息。
3. 队列保留着消息，直到他们被消费或超时。

### 特点

-    每个消息只要一个消费者
-    发送者和接收者在时间上是没有时间的约束，也就是说发送者在发送完消息之后，不管接收者有没有接受消息，都不会影响发送方发送消息到消息队列中。
-    发送方不管是否在发送消息，接收方都可以从消息队列中取到消息
-    接收方在接收完消息之后，需要向消息队列应答成功

## Publish/Subscribe Messaging Domain
发布/订阅消息模型

![[发布-订阅模型.png]]

### 涉及到的概念：

在发布/订阅消息模型中，发布者发布一个消息，该消息通过topic传递给所有的客户端。该模式下，发布者与订阅者都是匿名的，即发布者与订阅者都不知道对方是谁。并且可以动态的发布与订阅Topic。Topic主要用于保存和传递消息，且会一直保存消息直到消息被传递给客户端。

### 特点：

-   一个消息可以传递个多个订阅者（即：一个消息可以有多个接受方）
-   发布者与订阅者具有时间约束，针对某个主题（Topic）的订阅者，它必须创建一个订阅者之后，才能消费发布者的消息，而且为了消费消息，订阅者必须保持运行的状态。
-    为了缓和这样严格的时间相关性，JMS允许订阅者创建一个可持久化的订阅。这样，即使订阅者没有被激活（运行），它也能接收到发布者的消息。


# JMS编程模型　

1.  管理对象（Administered objects）-连接工厂（Connection Factories）和目的地（Destination）
2.  连接对象（Connections）
3.  会话（Sessions）
4.  消息生产者（Message Producers）
5.  消息消费者（Message Consumers）
6.  消息监听者（Message Listeners）



## Connection Factories

创建Connection对象的工厂，针对两种不同的jms消息模型，分别有QueueConnectionFactory和TopicConnectionFactory两种。可以通过JNDI来查找ConnectionFactory对象。客户端使用一个连接工厂对象连接到JMS服务提供者，它创建了JMS服务提供者和客户端之间的连接。JMS客户端（如发送者或接受者）会在JNDI名字空间中搜索并获取该连接。使用该连接，客户端能够与目的地通讯，往队列或话题发送/接收消息。

QueueConnectionFactory queueConnFactory = (QueueConnectionFactory) initialCtx.lookup ("primaryQCF");
Queue purchaseQueue = (Queue) initialCtx.lookup ("Purchase_Queue");
Queue returnQueue = (Queue) initialCtx.lookup ("Return_Queue");

## Destination

目的地指明消息被发送的目的地以及客户端接收消息的来源。JMS使用两种目的地，队列和话题。如下代码指定了一个队列和话题：

　　　　创建一个队列Session：

QueueSession ses = con.createQueueSession (false, Session.AUTO_ACKNOWLEDGE);  //get the Queue object 
Queue t = (Queue) ctx.lookup ("myQueue");  //create QueueReceiver 
QueueReceiver receiver = ses.createReceiver(t); 

　　　　（3）、Connection

　　　　　　Connection表示在客户端和JMS系统之间建立的链接（对TCP/IP socket的包装）。Connection可以产生一个或多个Session。跟ConnectionFactory一样，Connection也有两种类型：QueueConnection和TopicConnection。

　　　　　　连接对象封装了与JMS提供者之间的虚拟连接，如果我们有一个ConnectionFactory对象，可以使用它来创建一个连接。

Connection connection = connectionFactory.createConnection();

　　　　（4）、Session

　　　　　　Session 是我们对消息进行操作的接口，可以通过session创建生产者、消费者、消息等。Session 提供了事务的功能，如果需要使用session发送/接收多个消息时，可以将这些发送/接收动作放到一个事务中。

　　　　　　我们可以在连接创建完成之后创建session：

Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);

　　　　　　这里面提供了参数两个参数，第一个参数是是否支持事务，第二个是事务的类型

　　　　（5）、Producter

　　　　　　消息生产者由Session创建，用于往目的地发送消息。生产者实现MessageProducer接口，我们可以为目的地、队列或话题创建生产者；

MessageProducer producer = session.createProducer(dest);
MessageProducer producer = session.createProducer(queue);
MessageProducer producer = session.createProducer(topic);

　　　　（6）、Consumer

　　　　　　消息消费者由Session创建，用于接收被发送到Destination的消息。

MessageConsumer consumer = session.createConsumer(dest);
MessageConsumer consumer = session.createConsumer(queue);
MessageConsumer consumer = session.createConsumer(topic);

　　　　（7）、MessageListener

　　　　　　消息监听器。如果注册了消息监听器，一旦消息到达，将自动调用监听器的onMessage方法。EJB中的MDB（Message-Driven Bean）就是一种MessageListener。