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



