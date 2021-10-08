# 简介
Thread Dump 是一个 Java 工具。
Thread Dump 常用于诊断 java 问题。

thread dump 是一个文本文件，打开后可以看到每一个线程的执行栈，以stacktrace 的方式显示。
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
# 线程名称：Timer-0；线程类型：daemon；优先级: 10，默认是5；
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