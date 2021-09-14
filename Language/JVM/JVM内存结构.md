## 内存结构
[[JVM]] 结构如下：

![[JVM架构图.png]]

《java 虚拟机规范》中正式叫法是** Runtime Data Area** （运行时数据区），内存结构是为了方便的称呼。

### 分类
- **Method Area** 和 **Heap Area** 是线程共享的。
- **Stack Area** 、**PC Registers** 和 **Native Method Area**是每个线程独有的。


### Heap
- Heap 由所有 JVM 进程中的**线程共享**；
- **类实例（对象）** 和**数组**空间从 heap 分配；
- JVM **启动时创建 Heap**。

new 出来的对象空间都在 Heap 上分配，Heap 空间由 **GC（garbage collector）** 自动回收。
**根据 GC 回收的规则**，可将 Heap 空间细分为 Young Generation 和 Old Generation（存活的时长不同），具体如下图：
![[Heap的划分.png]]

不同区域的关系如下：
1. Java堆 = 老年代 + 新生代；
2. 新生代 = Eden + S0 + S1；
3. 默认：Eden：from ：to = 8:1:1。

Heap 各个区域的大小可以通过 JVM 参数控制，控制参数如下：
1. `-Xms`： 堆**初始（最小）**容量（堆包括新生代和老年代）。 例如：-Xms20M ;
2. `-Xmx`： 堆最大容量。 例如：-Xmx30M ;
3. `-Xmn`： 新生代**初始（最大）**容量。例如：-Xmn10M  ;
4. `-XX:SurvivorRatio`: **Eden/Survivor** 的比例。Eden:form:to的比例默认是8：1：1。例如：-XX： SurvivorRatio=8 代表比例8：1：1。
5. `-XX:NewRatio`: **old/new** 的比例。默认是2。
**注意：建议将 -Xms 和 -Xmx 设为相同值，避免每次垃圾回收完成后JVM重新分配内存！**  

当 Heap **没有足够的空间分配给对象且到达最大容量**，无法扩展时，会抛出常见的 *OOM(OutOfMemoryError)* 异常

具体 JVM 参数选项参考：
1. [JDK 8](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/java.html)
2. [JDK7 HotSpot VM Options](https://www.oracle.com/java/technologies/javase/vmoptions-jsp.html)

### Method Area
**Conception**
Method Area stores **per-class structures** such as the *run-time constant pool, field and method data, and the code for methods and constructors, including the special methods* ([§2.9](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.9 "2.9. Special Methods")) used in class and instance initialization and interface initialization.

Method Area 是JVM中所有线程共享的，JVM启动时创建，**主要存储类的结构信息（metadata）**。
JDK1.8之前称为永久代，PermGen space。
1. JDK1.8开始 Method Area 被称作 Metaspace(元空间)；
2. 存于本地内存中，最大为系统内存，不会出现内存溢出错误。
3. 大小通过`–XX:MetaspaceSize`设置，默认21M。


### PC Registers
当前线程所执行的字节码的行号指示器，用于记录正在执行的虚拟机字节指令地址，线程私有。

> 注意，程序计数器是唯一 一个在Java虚拟机规范中**没有规定任何 `OutOfMemoryError` 情况**的区域。

### Native Method Stacks
和虚拟栈相似，只不过它服务于Native方法，**线程私有**。当 Java 虚拟机使用其他语言（例如 C 语言）来实现指令集解释器时，也会使用到本地方法栈。如果 Java 虚拟机不支持 natvie 方法，并且自己也不依赖传统栈的话，可以无需支持本地方法栈。

本地方法栈区域也会抛出 `StackOverflowError` 和 `OutOfMemoryError` 异常。

**HotSpot虚拟机直接就把本地方法栈和虚拟机栈合二为一。**

### JVM Stacks
1、 Java 虚拟机的每一条线程都有自己私有的 Java 虚拟机栈，这个 Java 虚拟机栈跟线程同时创建，所以它跟线程有相同的生命周期。

2、Java 虚拟机栈描述的是 Java 方法执行的内存模型：每一个方法在执行的同时都会创建一个栈帧，用于存储局部变量表、操作数栈、动态链接、方法出口等信息，每一个方法从调用直至执行完成的过程，就对应着一个栈帧在 Java 虚拟机栈中的入栈到出栈的过程。

3、**局部变量表**存放了编译期可知的各种基本数据类型、对象引用和 returnAddress 类型。

> 1、基本类型：八种基本类型  
> 2、对象引用：reference 类型，它不等同于对象本身，根据不同的虚拟机实现，它可能是一个指向对象起始地址的引用指针，也可能指向一个代表对象的句柄或者其他与此对象相关的位置。  
> 3、 returnAddress 类型：指向了一条字节码指令的地址。

其中 64 位长度的 long 和 double 类型的数据会占用 2 个局部变量空间（Slot），其余的数据类型只占用 1 个。局部变量表所需的内存空间在编译期间完成分配，当进入一个方法时，这个方法需要在帧中分配多大的局部变量空间是完全确定的，在方法运行期间不会改变局部变量表的大小。

4、Java 虚拟机栈既允许被实现成固定的大小，也允许根据计算动态来扩展和收缩，如果采用固定大小的话，每一个线程的 Java 虚拟机栈容量可以在线程创建的时候独立选定。在 Java 虚拟机栈中会发生两种异常，这个在虚拟机规范中有指出：

-   如果线程请求分配的栈容量超过 Java 虚拟机栈允许的最大容量，Java 虚拟机将会抛出 `StackOverflowError` 异常；也就是栈溢出错误！**方法递归**调用产生`StackOverflowError` 异常这种结果。
-   如果 Java 虚拟机栈可以动态扩展，并且在尝试扩展的时候无法申请到足够的内存或者在创建新的线程时没有足够的内存去创建对应的 Java 虚拟机栈，那么虚拟机将会抛出 `OutOfMemoryError` 异常。也就是OOM内存溢出错误！(线程启动过多)

当然，可以通过参数 `-Xss` 去调整JVM栈的大小！
