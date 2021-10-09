# 简介
Thread Dump 是一个 Java 工具。
Thread Dump 常用于诊断 java 问题。

thread dump 是一个文本文件，打开后可以看到每一个**线程的执行栈的快照**，以stacktrace 的方式显示。
通过对多个 thread dump 的分析可以得到应用是否“卡”在某一点上，即在某一点运行的时间太长，如数据库查询，长期得不到响应，最终导致系统崩溃。

参见：
[[专业词汇#Thread dump]]

# 特点
1.  能在各种操作系统下使用；
2.  能在各种Java应用服务器下使用；
3.  能在生产环境下使用而不影响系统的性能；
4.  能将问题直接定位到应用程序的代码行上；


# 抓取
## JVM 工具
```
1.  jps 或 ps –ef | grep java （获取PID）
2.  jstack [-l ] | tee -a jstack.log（获取ThreadDump）
```

# Thread Dump 分析

## 头信息
**时间，JVM信息**
```
2011-11-02 19:05:06  
Full thread dump Java HotSpot(TM) Server VM (16.3-b01 mixed mode): 
```

## 线程INFO信息

```java
1. "Timer-0" daemon prio=10 tid=0xac190c00 nid=0xaef in Object.wait() [0xae77d000] 
# 线程名称：Timer-0；
# 线程类型：daemon；
# 优先级: 10，默认是5；
# JVM thread-id：tid=0xac190c00，JVM内部线程的唯一标识（通过java.lang.Thread.getId()获取，通常用自增方式实现）。
# 对应系统线程id（NativeThread ID）：nid=0xaef，和top命令查看的线程pid对应，不过一个是10进制，一个是16进制。（通过命令：top -H -p pid，可以查看该进程的所有线程信息）
# 线程状态：in Object.wait()；
# 起始栈地址：[0xae77d000]，对象的内存地址，通过JVM内存查看工具，能够看出线程是在哪儿个对象上等待；
2.  java.lang.Thread.State: TIMED_WAITING (on object monitor)
3.  at java.lang.Object.wait(Native Method)
4.  -waiting on <0xb3885f60> (a java.util.TaskQueue)     # 继续wait 
5.  at java.util.TimerThread.mainLoop(Timer.java:509)
6.  -locked <0xb3885f60> (a java.util.TaskQueue)         # 已经locked
7.  at java.util.TimerThread.run(Timer.java:462)
```

2-7 行是 Java thread statck trace 的信息，由于是一种 stack 结构，所以实际代码执行顺序是从下到上(7 -> 2)。


## Java thread statck trace
4、6 行 stack trace 如下：
```java
4. - waiting on <0xb3885f60> (a java.util.ArrayList)
6. - locked <0xb3885f60> (a java.util.ArrayList)
```

代码如下：
```java
synchronized(obj) {  
   .........  
   obj.wait();  
   .........  
}
```

实际：
1. *对象先上锁*，锁住对象0xb3885f60，用 synchronized 获得对象的 **Monitor**；
2. 然后释放该对象锁，*进入waiting状态*，执行到 obj.wait()，线程即放弃了 Monitor的所有权，进入 “wait set”队列。

在第 2 行信息中，进一步标明了线程在代码级的状态：

```java
java.lang.Thread.State: TIMED_WAITING (parking)
```

## 线程状态

Java 中线程的状态在Thread.State这个枚举类型中定义：
```java
public enum State   
{  
       /** 
        * Thread state for a thread which has not yet started. 
        */  
       NEW,  
         
       /** 
        * Thread state for a runnable thread.  A thread in the runnable 
        * state is executing in the Java virtual machine but it may 
        * be waiting for other resources from the operating system 
        * such as processor. 
        */  
       RUNNABLE,  
         
       /** 
        * Thread state for a thread blocked waiting for a monitor lock. 
        * A thread in the blocked state is waiting for a monitor lock 
        * to enter a synchronized block/method or  
        * reenter a synchronized block/method after calling 
        * {@link Object#wait() Object.wait}. 
        */  
       BLOCKED,  
     
       /** 
        * Thread state for a waiting thread. 
        * A thread is in the waiting state due to calling one of the  
        * following methods: 
        * <ul> 
        *   <li>{@link Object#wait() Object.wait} with no timeout</li> 
        *   <li>{@link #join() Thread.join} with no timeout</li> 
        *   <li>{@link LockSupport#park() LockSupport.park}</li> 
        * </ul> 
        *  
        * <p>A thread in the waiting state is waiting for another thread to 
        * perform a particular action.   
        * 
        * For example, a thread that has called <tt>Object.wait()</tt> 
        * on an object is waiting for another thread to call  
        * <tt>Object.notify()</tt> or <tt>Object.notifyAll()</tt> on  
        * that object. A thread that has called <tt>Thread.join()</tt>  
        * is waiting for a specified thread to terminate. 
        */  
       WAITING,  
         
       /** 
        * Thread state for a waiting thread with a specified waiting time. 
        * A thread is in the timed waiting state due to calling one of  
        * the following methods with a specified positive waiting time: 
        * <ul> 
        *   <li>{@link #sleep Thread.sleep}</li> 
        *   <li>{@link Object#wait(long) Object.wait} with timeout</li> 
        *   <li>{@link #join(long) Thread.join} with timeout</li> 
        *   <li>{@link LockSupport#parkNanos LockSupport.parkNanos}</li>  
        *   <li>{@link LockSupport#parkUntil LockSupport.parkUntil}</li> 
        * </ul> 
        */  
       TIMED_WAITING,  
  
       /** 
        * Thread state for a terminated thread. 
        * The thread has completed execution. 
        */  
       TERMINATED;  
}
```


## 关键状态分析

1.  **Wait on condition**：_The thread is either sleeping or waiting to be notified by another thread._ 
	等待另一个条件的发生，来把自己唤醒，或者自己是调用了 sleep(n)。
    
    **此时线程状态大致为以下几种：**
    
    > 1.  java.lang.Thread.State: WAITING (parking)：一直等条件发生；
    > 2.  java.lang.Thread.State: TIMED_WAITING (parking或sleeping)：定时的，条件不到来，也将定时唤醒自己。
    
2.  **Waiting for Monitor Entry 和 in Object.wait()**：_The thread is waiting to get the lock for an object (some other thread may be holding the lock). This happens if two or more threads try to execute synchronized code. Note that the lock is always for an object and not for individual methods._
    
    在多线程的 JAVA 程序中， Monitor是Java中用以实现线程之间的**互斥与协作/同步**的主要手段，它可以看成是对象或者Class的锁。
	每一个对象都有且仅有一个 Monitor。
	下面这个图，描述了线程和 Monitor之间关系，以及线程的状态转换图：
	
    ![[monitor与线程关系图.png]]
	
	
	从图可以看出：
	1. 每个Monitor在某个时刻只能被一个线程拥有，该线程就是 "Active Thread"；
	2. 其他线程都是 "Waiting Thread"，分别在两个队列 "Entry Set"和"Wait Set"里面等待；
	3. 其中在 "Entry Set" 中等待的线程状态是 waiting for monitor entry，在 "Wait Set" 中等待的线程状态是 in Object.wait()。

### Entry Set

当线程申请**进入临界区时，就进入了 "Entry Set" 队列**中，有两种可能性：

-   *Monitor 不被其他线程拥有*。"Entry Set"里面也没有其他等待的线程。本线程即成为相应类或者对象的Monitor的Owner，执行临界区里面的代码；此时在Thread Dump中显示线程处于 "Runnable" 状态。
-   *Monitor已经被其他线程拥有*。本线程在 "Entry Set" 队列中等待。此时在Thread Dump中显示线程处于 "waiting for monity entry" 状态。

临界区的设置是为了保证其内部的代码执行的原子性和完整性，但因为临界区在任何时间只允许线程串行通过。如果在多线程程序中大量使用synchronized，或者不适当的使用，会造成大量线程在临界区的入口等待，造成系统的性能大幅下降。
如果在Thread Dump中发现这个情况，应该审视源码并对其进行改进。

### Wait Set

当线程获得了Monitor，进入了临界区之后，如果*发现线程继续运行的**条件没有满足**，它则调用对象的wait()方法，放弃Monitor，进入 "Wait Set"队列*。
只有当别的线程在该对象上调用了** notify() 或者 notifyAll()** 方法，"Wait Set"队列中的线程才得到机会去竞争，但是只有一个线程获得对象的Monitor，恢复到运行态。
"Wait Set"中的线程在Thread Dump中显示的状态为 in Object.wait()。


## 分析案例
问题场景
### CPU飙高，load高，响应很慢
一个请求过程中多次dump；
对比多次dump文件的runnable线程，如果执行的方法有比较大变化，说明比较正常。如果在执行同一个方法，就有一些问题了；

### 查找占用CPU最多的线程
使用命令：top -H -p pid（pid为被测系统的进程号），找到导致CPU高的线程ID，对应thread dump信息中线程的nid ，只不过一个是十进制，一个是十六进制； 在thread
dump中，根据top命令查找的线程id，查找对应的线程堆栈信息；
CPU使用率不高但是响应很慢
进行dump，查看是否有很多thread struck在了i/o、数据库等地方 ，定位瓶颈原因；

请求无法响应
多次dump，对比是否所有的runnable线程都一直在执行相同的方法， 如果是的，恭喜你，锁住了！
