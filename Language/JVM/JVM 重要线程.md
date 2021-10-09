# 重要线程

JVM运行过程中产生的一些比较重要的线程罗列如下：

## Attach Listener
`Attach Listener` 线程:
- **监听/接收外部的命令**;
- 分发命令并返回结果。

`java -version、jmap、jstack`等等都是由`Attach Listener`线程接收。
如果该线程在JVM启动的时候没有初始化，会在用户第一次执行JVM命令时启动。

## Signal Dispatcher
- `Attach Listener`线程监听外部JVM命令；
- 之后由`signal dispather`线程**分发到各个模块处理**，并且返回处理结果。
`signal dispather`线程也是在第一次接收外部 JVM 命令时，进行初始化工作。

## CompilerThread0
- 调用`JITing`，实时编译装卸 class 。
通常，JVM会启动多个线程来处理这部分工作，线程名称后面的数字也会累加，例如：`CompilerThread1`。

## Concurrent Mark-Sweep GC Thread
-**并发标记清除垃圾回收器**（就是通常所说的CMS GC）线程；
- 主要针对于**老年代（old generation）**垃圾回收。

启用该垃圾回收器，需要在JVM启动参数中加上：`-XX:+UseConcMarkSweepGC`。

## DestroyJavaVM
- **卸载 JVM**；
- 执行 main() 的线程，*在 main 执行完后调用 JNI 中的 jni_DestroyJavaVM() 方法唤起DestroyJavaVM 线程*；
- DestroyJavaVM一开始处于等待状态，等待*其它线程（Java线程和Native线程）退出时通知它卸载JVM*；
- 每个线程退出时，都会判断自己当前是否是整个 JVM 中*最后一个非 deamon 线程*，如果是，则通知 DestroyJavaVM 线程卸载JVM。


## Finalizer Thread
- 垃圾收集前，**调用对象的 finalize() 方法**；
- 在main线程之后创建的，其优先级为10。

关于Finalizer线程的几点：
1) 只有当开始一轮垃圾收集时，才会开始调用 finalize() 方法；因此*并不是所有对象的finalize()方法都会被执行*；
2) 该线程*是 daemon 线程*，因此如果虚拟机中没有其他非 daemon线程，不管该线程有没有执行完 finalize()方法，JVM也会退出；
3) JVM 在垃圾收集时会将*失去引用的对象包装成 Finalizer 对象*（Reference的实现），并放入*ReferenceQueue*，由 Finalizer 线程来处理；最后将该 Finalizer 对象的引用置为null，由垃圾收集器来回收；
4) JVM 为什么要单独用一个线程来执行 finalize() 方法呢？如果 JVM 的垃圾收集线程自己来做，很有*可能由于 finalize() 方法中误操作导致GC线程停止或不可控*，这对GC线程来说是一种灾难；

## Low Memory Detector
- **对可用内存进行检测**，如果发现可用内存低，分配新的内存空间。

## Reference Handler
- 处理*引用对象本身（软引用、弱引用、虚引用）的垃圾回收问题*；
- JVM在创建 `main` 线程后就创建 `Reference Handler` 线程，其优先级最高，为10。

## VM Thread
- JVM里面的线程母体；
- 根据 hotspot 源码（`vmThread.hpp`）里面的注释，它是最原始的线程，会产生或触发所有其他的线程。