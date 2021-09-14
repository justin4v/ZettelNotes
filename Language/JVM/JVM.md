## JVM
**Definition**
在《 The Java® Virtual Machine Specification》 Java SE 8 Edition 中对 JVM 有如下定义：
The Java Virtual Machine is the **cornerstone** of the Java platform. It is the component of the technology **responsible for its hardware- and operating system-independence, the small size of its compiled code, and its ability to protect users** from malicious programs.

The Java Virtual Machine is an**abstract computing machine**. Like a real computing machine, it has an instruction set and manipulates various memory areas at run time. It is reasonably common to implement a programming language using a virtual machine; the best-known virtual machine may be the P-Code machine of UCSD Pascal.

**Conception**
1. JVM是Java平台的基石，使得 Java **独立于硬件和操作系统**；
2. JVM是一台**抽象的计算机**，具有**指令集**，运行时管理一些**内存区域**，如寄存器、栈、垃圾回收、堆等。

## Hotspot JVM
Hotspot JVM 是 JVM 的官方实现
4. Hotspot JVM 中的 Java 线程与原生操作系统线程有直接的映射关系。当线程本地存储、缓冲区分配、同步对象、栈、程序计数器等准备好以后，就会创建一个操作系统原生线程。
5. Java 线程结束，原生线程随之被回收。操作系统负责调度所有线程，并把它们分配到任何可用的 CPU 上。当原生线程初始化完毕，就会调用 Java 线程的 run() 方法。当线程结束时，会释放原生线程和 Java 线程的所有资源。

