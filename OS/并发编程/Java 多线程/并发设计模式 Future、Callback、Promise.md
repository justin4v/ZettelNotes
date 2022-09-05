#Java基础 #Concurrent 

# Java中的线程

- Java 中创建线程有哪些方式，大多数回答是 `Thread，Runnable，Callable`，然而这是错误的回答。
- Java 代码**提供给用户创建线程的方式只有 `Thread`**；
- `Runnable` 只是**代表提交给 `Thread` 执行的任务**， `Thread` 只会执行其对应的 `run` 方法;
- `Callable` 是一个辅助接口;
	- `Thread`  并不会执行其对应的 `call` 方法;
	- `Callable` 通常需要与 `Runnable` 一起使用，在 `Runnable` 的 `run()` 方法中调用 `Callable` 的 `call()` 的方法。 

# Future模式
- Future 是并发当中经常用到的一种设计模式；
- 核心思想是**异步执行，同步返回（主动查询执行结果）**。
	1. 把耗时操作放到异步线程中执行；
	2. 然后在获取结果时判断是否执行完，执行完则直接返回结果，**没执行完则阻塞等到返回**；
	3. 这是 future 模式的一般做法，目的是**充分利用等待时间**。

## JDK Future模式的使用

- 以JDK中的`Future`接口为例， `Future` 是一个 **read-only 的结构**：
	- 一旦任务被提交，**除了取消任务外不可以改变任务（没有改变的接口）**；
- 这也是Future模式与其他模式的重要区别。

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

### 使用示例
- Future 模式在JDK中一般如下方式使用；
- 主要步骤是创建对应的异步任务，然后需要结果时主动调用 get 方法。

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

- 不考虑其他因素，代码主线程中逻辑需要1000毫秒，同时任务执行也需要1000毫秒；
- 利用 `FutureTask` 并发执行，总耗时也是1000毫秒；
- `FutureTask` 最直观的效果：**增加系统的并发性，减少不必要的等待**。

## JDK Future模式的原理

从代码角度上来分析，一个 `FutureTask` 会经历以下几个过程:
### 接收任务
- 从代码角度上看，`FutureTask` 首先会接收一个`Callable`任务的任务；
- 并将自身状态设置为 `NEW` ( 关于状态在该类中有详细注释描述)。

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

### 执行任务
- 当线程启动时会调用其 `run()` 方法;
- 调用 `callable` 任务，然后把返回结果调用 `set()` 进行更新。
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

### 设置结果
- `set()` 会把对应的结果赋值给 **`FutureTask` 类的属性变量 `Object outcome`;**
```java
    /** The result to return or exception to throw from get() */
    private Object outcome; // non-volatile, protected by state reads/writes
```
- `FutureTask` 就是利用了**属性变量内存共享来实现的返回值获取**。

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

### 获取结果
- 结果 `FutureTask` 是共享的；
- 获取时*根据当前 task 所处于的状态*，如果是未完成的话则直接*进入等待线程队列*中，当运行结果 `outcome` **被设值时会主动唤醒这些等待线程**。

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

# Callback（回调式Future）

- Future 模式需要手动获取对应的值，且是阻塞操作;
- 有时候的业务并不怎么关心值什么时候被计算出来，*只关心计算出来后后续要做哪些操作*;
- 一种改进策略就是利用**回调方式(Callback) 实现后续逻辑**。 
- 以 guava 的 `FutureCallback` 为例，其接口定义如下：

```java
public interface FutureCallback<V> {

  void onSuccess(@NullableDecl V result);

  void onFailure(Throwable t);
}
```

- 该接口本身是一个业务处理，可以使用 guava 的 `Futures#addCallback` 添加到Future当中。
- Callback 的实现原理是：
	1. 在对应的 `Future` 中维护一个 `Callback` 链表；
	2. 当任务执行完成后依次执行对应的回调，类似于[观察者模式](https://mrdear.cn/2018/04/20/experience/design_patterns--observer/)的`Subject`依次调用`Observer`。 
- `Callback` **很好的解决了 `Future` 手动 `get()` 所带来的阻塞与不便**。
- 因为在值算出来时自动调用后续处理，不存在阻塞操作。
- 但是在业务后续操作很多时，可能存在一个 **多个callback 嵌套的问题，俗称回调地狱**，[这一点在JS中经常遇到](https://zhuanlan.zhihu.com/p/29783901)：

```javascript
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

- 回调嵌套过多在服务端倒不是很常见，但是**嵌套会使逻辑不清晰**，因此诞生了 **Promise 模式**，也是目前使用最多的一种模式。

# Promise（可变的Future）

- `Promise` 结合了 `Future` 与 `Callback` 两种形式:
	- 对于 `Future`，`Promise` 在其基础上提供**结果写入**的接口，也就是**可以主动完成 `Future`**;
	- 对于 `Callback`，`Promise` **把调用形式由嵌套打平**，避免了循环嵌套;
其使用方式大概如下JS代码所示：
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

- `Promise` 对于Future的改进原理是**提供主动完成的方法入口**，并且完成任务时会**主动触发所有的 `Callback`**;
- JDK 中提供了 `CompletableFuture` 类，用于实现 Promise 模式编程;
- 下面代码展示了其可以主动完成任务的能力，假如异步任务会导致线程无限休眠，仍然*可以通过主动设置值的方式完成该任务*。
- 这一特性可以很好的在**两个线程中交换数据使用**：
	- 举个例子在一些 RPC 框架中，客户端在对应的 `Handler` 中*向远端发出 `RPCRequest` 后*，创建一个 `Request Promise` 放入到**全局 Map（队列、缓存）** 中，然后*阻塞获取响应结果*；
	- 在 `RPCResponse` 异步返回的线程中从 Map 中取出 `Promise`，然后**主动把 response 结果设置进去**，对于使用方来说就像是*同步完成了一次调用*。

**Promise主动完成任务能力**
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

- `Promise` 对于 Callback 的改进是：
	- **把每一个 Callback 封装成一个 `Stage` 阶段**；
	- 所有的`Stage`之间使用单链表关联，因此*当任务完成准备回调时启动整个链表链路即可*。
	- 这是一种常用的打散嵌套调用的一种做法，比如 Java8 的 Stream 也是类似的做法；


**Promise对于Callback的优化**

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

- 三种模式中，`Future`是核心也是最基础的一种模式；
- `Callback`，`Promise`都是一种优化手段，
- 一般业务上使用`Promise`就可以了，其包含了前两者的全部功能。

# 参考
1. [Java8知识点](https://mrdear.cn/tags/java8/)
2. [并行设计模式--Future、Callback、Promise](https://cloud.tencent.com/developer/article/1347628)