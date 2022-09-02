#Reactive-streams #Publish-subscribe
# Reactive Streams 标准

## 含义
reactive stream 是一个标准，它的目标是定义带有非阻塞背压的异步数据流（可供线程间交换），它定义了实现非阻塞的back pressure的最小区间的接口，方法和协议。

reactive stream 可以有各种实现，如 JDK 9的响应流式API，Project-Reactor（Pivotal 公司 ）等。

其中，Reactive Streams规范诞生于 **Reactor** 之后，**Reactor** 是在2.0.0.RC1版本时，支持了Reactive Streams规范，点击查看[版本BLOG](https://spring.io/blog/2015/02/18/reactor-2-0-0-rc1-with-native-reactive-streams-support-now-available)。


## 设计意图

The main goal of Reactive Streams is to govern the exchange of stream data across an asynchronous boundary – think passing elements on to another thread or thread-pool — while ensuring that the receiving side is not forced to buffer arbitrary amounts of data.

**处理数据流在异步边界（比如异步线程之间）的交换（传递），同时保证接收方不会被迫缓存任意数量的数据**。

In other words, back pressure is an integral part of this model in order to allow the queues which mediate between threads to be bounded.

back pressure 是 Reactive Streams 模型中使得线程之间交换队列有界的关键。



## Stream （流）
**Stream流是由生产者生产并由一个或多个消费者消费的元素（item）的序列**。

 **生产者——消费者模型** 也被称为 **source/sink（源-库） 模型**或 **publisher-subscriber（发布者-订阅者）模型**。 


## 优点

I/O阻塞浪费了系统性能，只有纯异步处理才能发挥系统的全部性能；而JDK的异步API比较难用，成为异步编程的瓶颈，这是**Reactor**等其它反应式框架诞生的原因。

JDK的异步API使用的是传统的**命令式编程**，命令式编程是以控制流（逻辑）为核心，通过顺序、分支和循环三种控制结构来完成不同的行为。

而**Reactor**使用**反应式编程**，应用程序从以逻辑为中心转换为了**以数据为中心**，是命令式到声明式的转换。

**Reactor**反应库旨在解决JVM上 “经典” 异步方法的缺点，同时还拥有如下**特点**：

1. 可组合性和可读性，规避了**Callback Hell**
2. 以流的形式进行数据处理时，为流中每个节点提供了丰富的操作符
3. **在Subscribe之前，不会有任何事情发生**
4. 支持背压，消费者可以向生产者发出信号表明排放率过高
5. 支持两种反应序列：hot和cold



### 流处理机制

有几种**流处理机制**，其中**pull模型**和**push模型**是最常见的。

在 push 模型中，**发布者将元素推送给订阅者**。

在 pull 模式中，**订阅者从发布者拉取元素**。

例如：设计模式中的 **Iterator 迭代器就是 pull 模式**。



### 速率同步问题

 发布者和订阅者都以同样的速率工作，这是一个理想的情况，这些模式非常有效。 我们会考虑一些情况，如果**发布者和订阅者不按同样的速率工作**，这种情况下涉及的问题以及对应的解决办法。

当发布者比订阅者快的时候，有两种处理方法：

1. 后者必须有一个**无边界缓冲区**来保存快速传入的元素，或者它必须**丢弃**它无法处理的元素。 
2. 另一个解决方案是使用一种称为**背压（back-pressure ）**的策略，其中**订阅者告诉发布者减慢速率并保持元素**，直到订阅者准备好处理更多的元素。 

**背压**可确保更快的**发布者不会淹没较慢的订阅者**， 发布者可以实现有界缓冲区来**保存有限数量的元素**，如果缓冲区已满，可以**选择放弃**它们。

订阅者在请求发布者的元素并且元素不可用时，该做什么？ 在同步请求中**订阅者户必须阻塞等待数据可用**。 如果发布者同步地向订阅者发送元素，并且订阅者同步处理它们，则**发布者必须阻塞直到数据处理完成**。



## 响应式流

响应式流从2013年开始，作为提供非阻塞背压的异步流处理标准的倡议。 它旨在解决处理元素流的问题——如何将元素流从发布者传递到订阅者，而不需要发布者阻塞，或订阅者有无限制的缓冲区或丢弃。

响应式流模型非常简单——**订阅者向发布者发送多个元素的异步请求。 发布者向订阅者异步发送多个或稍少的元素。**

响应式流在**pull模型和push模型流处理机制之间动态切换**。：

- 当订阅者较慢时，它使用pull模型 – 根据自身情况，**主动通知发布者推送数据；**
- 当订阅者更快时使用push模型 – 定义一个更大的背压上限，**由发布者自由推送数据**。

## API

reactive streams 只包含四个接口：

```java
//发布者
public  interface  Publisher < T > {
    public  void  subscribe（Subscriber <？ super  T >  s）;
}
//订阅者
public  interface  Subscriber < T > {
    public  void  onSubscribe（Subscription  s）;
    public  void  onNext（T  t）; 
    public  void  onError（Throwable  t）;
    public  void  onComplete（）;
}
//表示Subscriber消费Publisher发布的一个消息的生命周期
public interface Subscription {
    public void request(long n);
    public void cancel();
}
//处理器，表示一个处理阶段，它既是订阅者也是发布者，并且遵守两者的契约
public interface Processor<T, R> extends Subscriber<T>, Publisher<R> {
}
```

- 发布者（publisher）是潜在无限数量的有序元素的生产者。 
	- 根据收到的要求向当前订阅者发送元素。
- 订阅者（subscriber）从  publisher 订阅(Publisher.subscribe() )并接收元素。
	-  *监听指定的事件*，例如OnNext,OnComplete,OnError等
- *订阅（subscription）* ：
	- Publisher 和 Subscriber *一对一的协调对象*；
	- Subscriber 可通过 subscription 向 Publisher 取消数据发送或者 request 更多数据。
- 处理者（processor）：
	-  `Processor`接口继承了`Publisher`和`Subscriber`接口。 
	- **`Processor<T,R>`订阅类型 T 的数据元素，接收并转换为类型 R 的数据，并发布变换后的数据 R 。**


# 参考
1. [Reactive Streams-01 ](https://hcqbuqingzhen.github.io/2022/02/11/001-reactive-streams/)
2. [聊聊Spring Reactor反应式编程 ](https://juejin.cn/post/6844903631133622285)