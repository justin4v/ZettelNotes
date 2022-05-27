## Reactive Streams 标准

### 含义

reactive stream 是一个标准，它的目标是定义带有非阻塞背压的异步数据流（可供线程间交换），它定义了实现非阻塞的back pressure的最小区间的接口，方法和协议。

reactive stream 可以有各种实现，如 JDK 9的响应流式API，Project-Reactor（Pivotal 公司 ）等。

其中，Reactive Streams规范诞生于 **Reactor** 之后，**Reactor** 是在2.0.0.RC1版本时，支持了Reactive Streams规范，点击查看[版本BLOG](https://spring.io/blog/2015/02/18/reactor-2-0-0-rc1-with-native-reactive-streams-support-now-available)。



### 设计意图

The main goal of Reactive Streams is to govern the exchange of stream data across an asynchronous boundary – think passing elements on to another thread or thread-pool — while ensuring that the receiving side is not forced to buffer arbitrary amounts of data.

**处理数据流在异步边界（比如异步线程之间）的交换（传递），同时保证接收方不会被迫缓存任意数量的数据**。

In other words, back pressure is an integral part of this model in order to allow the queues which mediate between threads to be bounded.

back pressure 是 Reactive Streams 模型中使得线程之间交换队列有界的关键。



### Stream （流）

**Stream流是由生产者生产并由一个或多个消费者消费的元素（item）的序列**。

 **生产者——消费者模型** 也被称为 **source/sink（源-库） 模型**或 **publisher-subscriber（发布者-订阅者）模型**。 



### 优点

I/O阻塞浪费了系统性能，只有纯异步处理才能发挥系统的全部性能；而JDK的异步API比较难用，成为异步编程的瓶颈，这是**Reactor**等其它反应式框架诞生的原因。

JDK的异步API使用的是传统的**命令式编程**，命令式编程是以控制流（逻辑）为核心，通过顺序、分支和循环三种控制结构来完成不同的行为。

而**Reactor**使用**反应式编程**，应用程序从以逻辑为中心转换为了**以数据为中心**，是命令式到声明式的转换。

**Reactor**反应库旨在解决JVM上 “经典” 异步方法的缺点，同时还拥有如下**特点**：

1. 可组合性和可读性，规避了**Callback Hell**
2. 以流的形式进行数据处理时，为流中每个节点提供了丰富的操作符
3. **在Subscribe之前，不会有任何事情发生**
4. 支持背压，消费者可以向生产者发出信号表明排放率过高
5. 支持两种反应序列：hot和cold



#### 流处理机制

有几种**流处理机制**，其中**pull模型**和**push模型**是最常见的。

在 push 模型中，**发布者将元素推送给订阅者**。

在 pull 模式中，**订阅者从发布者拉取元素**。

例如：设计模式中的 **Iterator 迭代器就是 pull 模式**。



#### 速率同步问题

 发布者和订阅者都以同样的速率工作，这是一个理想的情况，这些模式非常有效。 我们会考虑一些情况，如果**发布者和订阅者不按同样的速率工作**，这种情况下涉及的问题以及对应的解决办法。

当发布者比订阅者快的时候，有两种处理方法：

1. 后者必须有一个**无边界缓冲区**来保存快速传入的元素，或者它必须**丢弃**它无法处理的元素。 
2. 另一个解决方案是使用一种称为**背压（back-pressure ）**的策略，其中**订阅者告诉发布者减慢速率并保持元素**，直到订阅者准备好处理更多的元素。 

**背压**可确保更快的**发布者不会淹没较慢的订阅者**， 发布者可以实现有界缓冲区来**保存有限数量的元素**，如果缓冲区已满，可以**选择放弃**它们。

订阅者在请求发布者的元素并且元素不可用时，该做什么？ 在同步请求中**订阅者户必须阻塞等待数据可用**。 如果发布者同步地向订阅者发送元素，并且订阅者同步处理它们，则**发布者必须阻塞直到数据处理完成**。



### 响应式流

响应式流从2013年开始，作为提供非阻塞背压的异步流处理标准的倡议。 它旨在解决处理元素流的问题——如何将元素流从发布者传递到订阅者，而不需要发布者阻塞，或订阅者有无限制的缓冲区或丢弃。

响应式流模型非常简单——**订阅者向发布者发送多个元素的异步请求。 发布者向订阅者异步发送多个或稍少的元素。**

响应式流在**pull模型和push模型流处理机制之间动态切换**。：

- 当订阅者较慢时，它使用pull模型 – 根据自身情况，**主动通知发布者推送数据；**
- 当订阅者更快时使用push模型 – 定义一个更大的背压上限，**由发布者自由推送数据**。

### API

Java API 的响应式流只包含四个接口：

```java
Publisher<T>
Subscriber<T>
Subscription
Processor<T,R>
```

**发布者（publisher）**是潜在无限数量的有序元素的生产者。 它根据收到的要求向当前订阅者发送元素。

**订阅者**（subscriber）从发布者那里**订阅 Publisher.subscribe() **并接收元素。 **发布者向订阅者发送订阅令牌（subscription token，也就是subscription 对象）**。 使用订阅令牌，订阅者可以从发布者那里**请求n个元素 subscription.request()**。 当元素准备就绪时，发布者向订阅者发送 **n个或更少的元素**。 

**订阅**（subscription）表示订阅者订阅的一个发布者的令牌。 当**订阅请求成功**时，发布者将其传递给订阅者。 订阅者使用订阅令牌与发布者进行交互，例如请求更多的元素或取消订阅。

**处理者**（processor）充当订阅者和发布者的处理阶段。 `Processor`接口继承了`Publisher`和`Subscriber`接口。 它用于转换 发布者–订阅者 管道中的元素。 **`Processor<T,R>`订阅类型 T 的数据元素，接收并转换为类型 R 的数据，并发布变换后的数据 R 。**



## Reactive Stream 的 Java 实现

### JDK 9  响应式流 API

JDK 9在 **java.util.concurrent** 包中提供了一个与响应式流兼容的API，它在java.base模块中。 API由**两个类**组成：

```java
Flow 
SubmissionPublisher<T>
```

**`Flow`类是final的**。 它封装了响应式流Java API和静态方法。 由响应式流Java API指定的四个接口作为嵌套静态接口包含在`Flow`类中：

```java
Flow.Processor<T,R>
Flow.Publisher<T>
Flow.Subscriber<T>
Flow.Subscription
```

这四个接口包含与上面代码所示的相同的方法。 `Flow`类包含`defaultBufferSize()`静态方法，它返回发布者和订阅者使用的**缓冲区的默认大小**。 目前，它返回256。

`SubmissionPublisher<T>`类是`Flow.Publisher<T>`接口的实现类。 该类实现了`AutoCloseable`接口，因此可以使用try-with-resources块来管理其实例。

 JDK 9不提供`Flow.Subscriber<T>`接口的实现类; 需要自己实现。 但是，`SubmissionPublisher<T>`类包含可用于处理此发布者发布的所有元素的`consume(Consumer<? super T> consumer)`方法。



### 发布者-订阅者交互

发布者可以拥有零个或多个订阅者。 这里只使用一个订阅者。

- **创建发布者和订阅者**。它们分别是`Flow.Publisher`和`Flow.Subscriber`接口的实例。
- 订阅者通过调用发布者的`subscribe()`方法来**尝试订阅发布者**。 如果**订阅成功**，发布者用**`Flow.Subscription`异步调用订阅者的`onSubscribe()`方法–将订阅令牌 Subscription 传递给订阅者**。 如果尝试**订阅失败**，则使用**调用订阅者的`onError()`方法**，并抛出`IllegalStateException`异常，并且发布者-订阅者交互结束。
- **订阅者通过调用`Subscription`的`request(N)`方法向发布者发送多个元素的请求**。 订阅者可以向发布者**发送更多元素的多个请求**，而不必等待其先前请求是否完成。
- **发布者**在所有先前的请求中**调用订阅者的`onNext(T item)`方法**，**直到达到订阅者请求的元素数量上限**——在每次调用中向订阅者发送一个元素。 如果发布者**没有更多的元素**要发送给订阅者，则发布者**调用订阅者的`onComplete()`方法**来发信号通知流，从而结束发布者——订阅者交互。 **如果订阅者请求`Long.MAX_VALUE`元素，则它实际上是无限制的请求，并且流实际上是推送流。**
- 如果发布者发布时遇到错误，它会调用订阅者的`onError()`方法。
- **订阅者可以通过调用`Flow.Subscription`的`cancel()`方法来取消订阅**。 一旦订阅被取消，发布者-订阅者交互结束。 然而，如果**在请求取消之前存在未决请求，订阅者可以在取消订阅之后接收元素。**

总结上述结束条件的步骤，一旦在订阅者上调用了`onComplete()`或`onError()`方法，订阅者就不再收到发布者的通知。

在发布者的`subscribe()`方法被调用之后，如果订阅者不取消其订阅，则保证以下订阅方法调用序列：

```java
1.onSubscribe (1次)
2.onNext*    (任意次)
3.(onError | onComplete)?  (0或1次)
```

这里，符号`*`和`?`在正则表达式中被用作关键字，一个`*`表示零个或多个出现， `?`意为零或一次。

在订阅者上的第一个方法调用是**`onSubscribe()`方法，它是成功订阅发布者的通知**。订阅者的`onNext()`方法可以被调用零次或多次，每次调用指示元素发布。`onComplete()`和`onError()`方法可以被调用为零或一次来指示终止状态; 只要订阅者不取消其订阅，就会调用这些方法。