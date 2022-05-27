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

## 解释
### F
# 参考
1. [Reactive Spring实战 -- 理解Reactor的设计与实现 ](https://www.cnblogs.com/binecy/p/14458911.html)