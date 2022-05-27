

# Reactive Stream 的 Java 实现
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

