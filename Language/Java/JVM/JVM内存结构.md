# 内存结构
[[JVM简介]] 结构如下：

![[JVM架构图.png]]

《java 虚拟机规范》中正式叫法是** Runtime Data Area** （运行时数据区），内存结构是为了方便的称呼。

## 分类
- **Method Area** 和 **Heap Area** 是线程共享的。
- **Stack Area** 、**PC Registers** 和 **Native Method Area**是每个线程独有的。


# Heap
- Heap 由所有 JVM 进程中的**线程共享**；
- **类实例（对象）** 和**数组**空间从 heap 分配；
- JVM **启动时创建 Heap**。
- new 出来的对象空间都在 Heap 上分配；
- Heap 空间由 **GC（garbage collector）** 自动回收。


根据[[分代收集理论#两个假说|分代收集理论]]，可将 Heap 空间细分为 :**Young Generation 和 Old Generation**（存活的时长不同）
具体如下图：
![[Heap的划分.png]]

*Eden*: 指《圣经》中亚当和夏娃最初居住的地方，这里引申为对象最初存放的位置。参考[[对象分配规则]]

不同区域的关系如下：
1. *Java Heap* = 老年代 + 新生代；
2. *Young Generation* = Eden + Survivor 0 + Survivor 1；
3. 默认比例=> Eden：from ：to = 8:1:1。


Heap 各个区域的大小可以通过 JVM 参数控制，控制参数如下：
1. `-Xms`： 堆**初始（最小）** 容量（Heap 大小*默认为新生代和老年代容量之和*）。 例如：-Xms20M ;
2. `-Xmx`： 堆最大容量。 例如：-Xmx30M ;
3. `-Xmn`： 新生代**初始（最大）**容量。例如：-Xmn10M  ;
4. `-XX:SurvivorRatio`: **Eden/Survivor** 的比例。Eden:form:to的比例默认是8：1：1。例如：-XX： SurvivorRatio=8 代表比例8：1：1。
5. `-XX:NewRatio`: **old/new** 的比例。默认是2。
**注意：建议将 -Xms 和 -Xmx 设为相同值，避免每次垃圾回收完成后JVM重新分配内存！**  

Heap 初始容量分配根据系统的配置而定（参考 [Ergonomics](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/ergonomics.html#sthref5)），一般系统（CPU 2 核以上、内存 2 GB 以上）Heap size **默认设置**：
1. Heap size **初始容量为内存的 1/64，最大 1GB**；
2. Heap size **最大容量为内存的 1/4，最大 1GB**。

当 Heap **没有足够的空间分配给对象且到达最大容量**，无法扩展时，会抛出常见的 *OOM(OutOfMemoryError)* 异常

具体 JVM 参数选项参考：
1. [JDK 8](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/java.html)
2. [JDK7 HotSpot VM Options](https://www.oracle.com/java/technologies/javase/vmoptions-jsp.html)

# Method Area(Metaspace)
**Conception**
Method Area stores **per-class structures（类结构信息）** such as the *run-time constant pool, field and method data, and the code for methods and constructors, including the special methods* ([§2.9](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.9 "2.9. Special Methods")) used in class and instance initialization and interface initialization.

Method Area ：
1. *线程共享的*；
2. *JVM 启动时创建*；
3. **存储类的结构信息（metadata）**。


JDK1.8之前称为永久代，**PermGen space**。
1. JDK1.8开始 **Method Area 被称作 Metaspace(元空间，存放元数据)**；
2. **存于本地内存**中，最大默认无限制，**最大为系统内存**，不会出现内存溢出错误。
3. 大小通过`–XX:MetaspaceSize`设置，默认21M。


# PC Registers
当前线程所执行的字节码的行号指示器，用于记录正在执行的虚拟机字节指令地址，线程私有。

> 注意，程序计数器是唯一 一个在Java虚拟机规范中**没有规定任何 `OutOfMemoryError` 情况**的区域。

# Native Method Stacks
和虚拟栈相似，只不过它服务于Native方法，**线程私有**。当 Java 虚拟机使用其他语言（例如 C 语言）来实现指令集解释器时，也会使用到本地方法栈。
如果 Java 虚拟机不支持 natvie 方法，并且自己也不依赖传统栈的话，可以无需支持本地方法栈。

本地方法栈区域会抛出 `StackOverflowError` 和 `OutOfMemoryError` 异常。

**HotSpot虚拟机直接就把本地方法栈和虚拟机栈合二为一**

# JVM Stacks
**所执行方法的存储结构**。

## Stack Frame 特点
-  **Java 方法执行的内存模型**：**每个方法在执行时都会创建一个Stack Frame**，用于存储局部变量表、操作数栈、动态链接、方法出口等信息。每一个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中入栈到出栈的过程。
-   存储**数据和部分过程结果**，也用来处理**动态链接(Dynamic Linking)、 方法返回值和异常分派（ Dispatch Exception）**。栈帧随着方法调用而创建，随着方法结束（正常或异常）而销毁。
-   **栈上分配**：对于小对象（一般几十个bytes），在没有**逃逸**的情况下，可以直接分配在栈上（直接分配在栈上，可以自动回收，减轻GC压力）；大对象或者逃逸对象无法栈上分配

## Stack Frame 结构

![[JVM Stack Frame结构.png]]

其中，*指向运行时常量池的引用* 是**动态绑定**

**局部变量表**
-   存储方法中的局部变量值或地址（包括方法中的**非静态变量以及函数形参**）；
-   编译时确定大小，**程序执行期间大小不变**。

**操作数栈**
-   程序中的所有计算过程都在操作数栈上完成；
-   最典型的一个应用就是对表达式求值。

**指向运行时常量池的引用**
指向在方法执行的过程中使用的常量。  
  
**方法返回地址**
方法执行完后，返回之前调用它的位置，因此在栈帧中保存一个方法返回地址。
