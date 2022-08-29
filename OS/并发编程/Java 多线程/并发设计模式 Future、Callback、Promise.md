#Java基础 #Concurrent #Todo 

## Java中的线程

经常有面试题问Java中创建线程有哪些方式，大多数回答是`Thread，Runnable，Callable`，然而这是错误的回答。Java代码提供给用户创建线程的方式只有`Thread`，而`Runnable`只是提交给线程执行的任务，线程`Thread`也只会执行其对应的run方法，最后`Callable`是一个辅助接口，线程并不会执行其对应的call方法，所以`Callable`通常情况下需要与`Runnable`一起使用，然后在对应的run()方法中调用call()的方法。 理解这一点对接下来的内容很有帮助。

## Future模式

**Future模式**是并发当中经常用到的一种设计模式，它的核心思想是**异步执行，同步返回**。把耗时操作放到异步线程中执行，然后再获取结果时判断是否执行完，执行完则直接返回结果，没执行完则阻塞等到返回，这是future模式的一般做法，目的是**充分利用等待时间**

### JDK Future模式的使用

以JDK中的`Future`接口为例，可以发现`Future`是一个read-only 的结构，一旦任务被提交除了取消任务外就不可以改变任务，这一点也是Future模式与其他模式的重要区别。

清单1：JDK中的Future接口
```java
public interface Future<V> {

  boolean cancel(boolean mayInterruptIfRunning);

  boolean isCancelled();

  boolean isDone();
  /**
   * 获取结果
   */
  V get() throws InterruptedException, ExecutionException;

  V get(long timeout， TimeUnit unit)
      throws InterruptedException， ExecutionException， TimeoutException;
}
```


Future模式在JDK中一般如下方式使用，主要步骤是创建对应的异步任务，然后需要结果时主动调用get方法

```java
private static ExecutorService executorService = Executors.newCachedThreadPool();
public static void main(String[] args) throws InterruptedException, TimeoutException, ExecutionException {
  // 把耗时任务放到异步线程中执行
  Future<String> task = new FutureTask<>(() -> {
    Thread.sleep(1000L);
    return "quding";
  });
  // 提交任务
  executorService.submit((Runnable) task);
  // 模拟耗时操作
  Thread.sleep(1000);
  // 获取结果
  String result = task.get(1000,TimeUnit.MILLISECONDS);
}
```

不考虑其他因素，上面代码主线程中逻辑需要1000毫秒，同时任务执行也需要1000毫秒，两者利用`FutureTask`并行执行，因此总耗时也是1000毫秒，这是`FutureTask`带来的最直观的效果：**增加系统的并发性，减少不必要的等待**。

### JDK Future模式的原理

从代码角度上来分析，一个`FutureTask`会经历以下几个过程:

**1. 接收任务** 从代码角度上看，`FutureTask`首先会接收一个`Callable`任务的任务，并将自身状态设置为`NEW`(关于状态在该类中有详细注释描述)

```java
public FutureTask(Callable<V> callable) {
  if (callable == null)
      throw new NullPointerException();
  this.callable = callable;
  this.state = NEW;       // ensure visibility of callable
}
// 即使提交的是runnable也会适配成对应的callable
public FutureTask(Runnable runnable， V result) {
  this.callable = Executors.callable(runnable， result);
  this.state = NEW;       // ensure visibility of callable
}
```

**2. 执行任务** 当线程启动时会调用其`run`方法，该方法会调用`callable`任务，然后把返回结果调用`set`进行更新。


```java
public void run() {
  // CAS更新把当前对象与线程绑定起来
  if (state != NEW ||
      !UNSAFE.compareAndSwapObject(this， runnerOffset，
                                   null， Thread.currentThread()))
      return;
  try {
      Callable<V> c = callable;
      if (c != null && state == NEW) {
          V result;
          boolean ran;
          try {
            // 执行call方法
              result = c.call();
              ran = true;
          } catch (Throwable ex) {
              result = null;
              ran = false;
            // 对任务执行异常的封装
              setException(ex);
          }
          if (ran)
              // 更新对应的结果值。
              set(result);
      }
  }
  。。。
}
```

**3. 设置结果** `set`方法中会把对应的结果赋值给属性变量`Object outcome`，那么`FutureTask`原理就就是利用了**属性变量内存共享来实现的返回值获取**。

```java
protected void set(V v) {
  // CAS更新当前状态为已完成
  if (UNSAFE.compareAndSwapInt(this， stateOffset， NEW， COMPLETING)) {
      outcome = v;
      // 状态调回，该方法会可以防止重排序带来的影响
      UNSAFE.putOrderedInt(this， stateOffset， NORMAL); // final state
      // 完成通知，唤醒等待结果的线程
      finishCompletion();
  }
}
```

**4. 获取结果** 结果是共享的，因此获取时根据当前task所处于的状态，如果是未完成的话则直接进入等待线程队列中，当结果被设置时会主动唤醒这些等待线程。

```java
public V get() throws InterruptedException， ExecutionException {
  int s = state;
  if (s <= COMPLETING)
      // 未完成则进入等待线程队列中
      s = awaitDone(false， 0L);
  // 获取返回值
  return report(s);
}
```

## Callback（回调式Future）

对于Future模式，最麻烦的地方是需要手动获取对应的值，这个过程并且是阻塞操作，有时候的业务并不怎么关心值什么时候被计算出来，只关心计算出来后后续要做哪些操作，因此一种改进策略就是利用回调方式(Callback)实现后续逻辑。 以guava中`FutureCallback`为例，其接口定义如下：

```java
public interface FutureCallback<V> {

  void onSuccess(@NullableDecl V result);

  void onFailure(Throwable t);
}
```

该接口本身是一个业务处理，可以使用`Futures#addCallback`添加到Future当中。Callback的实现原理可以的想到在对应的`Future`中维护一个`Callback`链表，当任务执行完成后依次执行对应的回调，类似于[观察者模式](https://mrdear.cn/2018/04/20/experience/design_patterns--observer/)的`Subject`依次调用`Observer`。 `Callback`很好的解决了`Future`手动调用get所带来的阻塞与不便。因为在值算出来时自动调用后续处理因此不存在阻塞操作。但是在业务后续操作很多时，其存在一个嵌套的问题，俗称回调地狱，[这一点在JS中经常遇到](https://zhuanlan.zhihu.com/p/29783901)：

```java
api.getItem(1)
  .then(item => {
    item.amount++;
    api.updateItem(1, item)
      .then(update => {
        api.deleteItem(1)
          .then(deletion => {
            console.log('done!');
          })
      })
  })
```

回调嵌套过多在服务端倒不是很常见，但是嵌套会使得逻辑变得很难梳理，因此诞生了Promise模式，也是目前使用最多的一种模式。


## Promise（可变的Future）

`Promise`结合了Future与Callback两种形式，对于Future，Promise在其基础上提供结果写入的接口，也就是可以主动完成这个Future，对于Callback，Promise所作的是把调用形式由嵌套打平，避免了循环嵌套，其使用方式大概如下JS代码所示：

**清单8：Promise使用形式**

```javascript
api.getItem(1)
  .then(item => {
    item.amount++;
    return api.updateItem(1, item);
  })
  .then(update => {
    return api.deleteItem(1);
  })
  .then(deletion => {
    console.log('done!');
  })
```


`Promise`对于Future的改进原理是提供主动完成的方法入口，并且完成任务时会主动触发所有的Callback，

在JDK中提供了`CompletableFuture`类，用于实现Promise模式编程，**清单9**展示了其可以主动完成任务的能力，即使异步任务会导致异步线程无限休眠，但是仍然可以通过主动设置值的方式完成该任务。这一特性可以很好的在两个线程中交换数据使用，举个例子在一些RPC框架中客户端在对应的Handler中发出来`RPCRequest`后创建一个Promise放入到全局Map中，然后阻塞获取响应结果，在`RPCResponse`异步返回的线程中从Map中取出Promise，然后主动把结果设置进去，那么对于使用方来说就像是同步完成了一次调用。

**清单9：Promise主动完成任务能力**

```java
CompletableFuture<String> promise = CompletableFuture.supplyAsync(() -> {
     // 无限休眠
     sleep(Long.MAX_VALUE);
     return null;
   });

   // 调用主动完成
   promise.complete("quding2");

   System.out.println(promise.get()); // 输出quding2
```

`Promise`对于Callback的改进是把每一个Callback封装成一个`Stage`阶段，所有的`Stage`之间使用单链表关联，因此当完成时启动整个链表链路即可。这是一种常用的打散嵌套调用的一种做法，比如Java8的Stream也是类似的做法，详细的可以参考之前写的Stream分析文章。[Java8知识点](https://mrdear.cn/tags/java8/)

**清单10：Promise对于Callback的优化**

```javascript
CountDownLatch latch = new CountDownLatch(1);

   CompletableFuture<String> promise = CompletableFuture.supplyAsync(() -> {
     sleep(2000L);
     return "quding";
   });
   // 将一系列的Callback打平
   promise.thenApply(String::toUpperCase)
       .thenApply(t -> t + ": 屈定")
       .whenComplete((t, ex) -> {
         if (ex != null) {
           ex.printStackTrace();
         } else {
           System.out.println("result: " + t); // 输出 result: QUDING: 屈定
         }
         latch.countDown();
       });

   latch.await();
```


## 总结

由于篇幅过长，因此原本打算加入的Netty相关内容被去掉了，后续会分析Netty相关的实现机制。另外三种模式中，`Future`是核心也是最基础的一种模式，`Callback`，`Promise`都是一种优化手段，一般业务上使用`Promise`就可以了，其包含了前两者的全部功能。