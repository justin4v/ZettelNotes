#Java基础 #Lock 
# 概念

JDK中对Synchronized做的种种优化，其核心都是为了减少这种重量级锁的使用。

JDK1.6以后，为了减少获得锁和释放锁所带来的性能消耗，提高性能，引入了[[轻量级锁]]和[[偏向锁]]。


# 其他优化

## 适应性自旋（Adaptive Spinning)
从轻量级锁获取的流程中我们知道，当线程在获取轻量级锁的过程中执行CAS操作失败时，是要通过自旋来获取重量级锁的。问题在于，自旋是需要消耗CPU的，如果一直获取不到锁的话，那该线程就一直处在自旋状态，白白浪费CPU资源。解决这个问题最简单的办法就是指定自旋的次数，例如让其循环10次，如果还没获取到锁就进入阻塞状态。但是JDK采用了更聪明的方式——适应性自旋，简单来说就是线程如果自旋成功了，则下次自旋的次数会更多，如果自旋失败了，则自旋的次数就会减少。

## 锁粗化（Lock Coarsening）
锁粗化的概念应该比较好理解，就是将多次连接在一起的加锁、解锁操作合并为一次，将多个连续的锁扩展成一个范围更大的锁。举个例子：
```java
package com.paddx.test.string;

public class StringBufferTest {
    StringBuffer stringBuffer = new StringBuffer();

    public void append(){
        stringBuffer.append("a");
        stringBuffer.append("b");
        stringBuffer.append("c");
    }
}
```

这里每次调用stringBuffer.append方法都需要加锁和解锁，如果虚拟机检测到有一系列连串的对同一个对象加锁和解锁操作，就会将其合并成一次范围更大的加锁和解锁操作，即在第一次append方法时进行加锁，最后一次append方法结束后进行解锁。


## 锁消除（Lock Elimination）
锁消除即删除不必要的加锁操作。根据代码逃逸技术，如果判断到一段代码中，堆上的数据不会逃逸出当前线程，那么可以认为这段代码是线程安全的，不必要加锁。看下面这段程序：
```java
package com.paddx.test.concurrent;

public class SynchronizedTest02 {

    public static void main(String[] args) {
        SynchronizedTest02 test02 = new SynchronizedTest02();
        //启动预热
        for (int i = 0; i < 10000; i++) {
            i++;
        }
        long start = System.currentTimeMillis();
        for (int i = 0; i < 100000000; i++) {
            test02.append("abc", "def");
        }
        System.out.println("Time=" + (System.currentTimeMillis() - start));
    }

    public void append(String str1, String str2) {
        StringBuffer sb = new StringBuffer();
        sb.append(str1).append(str2);
    }
}
```

虽然StringBuffer的append是一个同步方法，但是这段程序中的StringBuffer属于一个局部变量，并且不会从该方法中逃逸出去，所以其实这过程是线程安全的，可以将锁消除。
为了尽量减少其他因素的影响，这里禁用了偏向锁（-XX:-UseBiasedLocking）
下面是本地执行的结果：
![[锁消除示例.png]]



参考
[Synchronize底层优化](https://www.cnblogs.com/paddix/p/5405678.html)




