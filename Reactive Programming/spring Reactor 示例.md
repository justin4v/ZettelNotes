#Reactive-streams #Reactor #Todo 

# 示例
```java
@Test
public void range() {
    // [1]
    Flux flux = Flux.range(1, 10);
    // [2]
    Subscriber subscriber = new BaseSubscriber<Integer>() {
        protected void hookOnNext(Integer value) {
            System.out.println(Thread.currentThread().getName() + " -> " + value);
            request(1);
        }
    };
    // [3]
    flux.subscribe(subscriber);
}

```

Reactor中，发布者Publisher负责生产数据，有两种发布者，Flux可以生产N个数据，Mono可以生产0~1个数据。  
订阅者Subscriber负责处理，消费数据。  
- `1` 构建一个发布者 Flux 。注意，这时发布者还没开始生产数据。  
- `2` 构建一个订阅者 Subscriber  
- `3` 创建 publisher-subscriber 订阅关系，生产者开始生产数据，并传递给订阅者

## 调用链分析
### Flux.range
Flux.range，Flux.fromArray 等静态方法都会返回一个Flux子类，如 FluxRange，FluxArray。

### Flux.subscribe
```java
public final void subscribe(Subscriber<? super T> actual) {
    CorePublisher publisher = Operators.onLastAssembly(this);
    CoreSubscriber subscriber = Operators.toCoreSubscriber(actual);

    try {
        ...

        publisher.subscribe(subscriber);
    }
    catch (Throwable e) {
        Operators.reportThrowInSubscribe(subscriber, e);
        return;
    }
}
```

- 获取内部的CorePublisher
- 程序中订阅者，会转化为一个 CoreSubscriber。
- CorePublisher 也有一个内部的 subscribe 方法，由Flux子类实现。

### FluxRange.subscribe
```java
public void subscribe(CoreSubscriber<? super Integer> actual) {
    ...
    actual.onSubscribe(new RangeSubscription(actual, st, en));
}
```

- Subscription 代表了发布者与订阅者之间的一个订阅关系，由 Publisher 端实现。  
- Flux 子类 subscribe 方法中通常会使用CoreSubscriber 创建为 Subscription，并调用订阅者的onSubscribe方法，这时订阅关系已完成。

### onSubscribe

BaseSubscriber#onSubscribe -> hookOnSubscribe
```java
protected void hookOnSubscribe(Subscription subscription) {
    subscription.request(9223372036854775807L);
}
```

- Subscription#request 由 Publisher 端实现，是核心方法;
- 订阅者通过该方法向发布者拉取特定数量的数据。此时发布者才开始生产数据。

RangeSubscription#request -> RangeSubscription#slowPath -> Subscriber#onNext

```java
void slowPath(long n) {
    Subscriber<? super Integer> a = this.actual;
    long f = this.end;
    long e = 0L;
    long i = this.index;

    while(!this.cancelled) {
        // [1]
        while(e != n && i != f) {
            a.onNext((int)i);
            if (this.cancelled) {
                return;
            }

            ++e;
            ++i;
        }

        ...
    }
}
```

`1` RangeSubscription负责生产指定范围内的整数，并调用Subscriber#onNext将数据推送到订阅者。

可以看到，  
Publisher#subscribe完成订阅操作，生成Subscription订阅关系，并触发订阅者钩子方法onSubscribe。  
订阅者的onSubscribe方法中，订阅者开始调用Subscription#request请求数据，这时发布者才开始生产数据，并将数据推给订阅者。

# 参考
1. [Reactive Spring实战 -- 理解Reactor的设计与实现 ](https://www.cnblogs.com/binecy/p/14458911.html)