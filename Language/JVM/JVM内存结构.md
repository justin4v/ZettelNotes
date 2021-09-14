## 内存结构
[[JVM]] 结构如下：

![[JVM架构图.png]]

《java 虚拟机规范》中正式叫法是** Runtime Data Area** （运行时数据区），内存结构是为了方便的称呼。

### 分类
- **Method Area** 和 **Heap Area** 是线程共享的。
- **Stack Area** 、**PC Registers** 和 **Native Method Area**是每个线程独有的。


### Heap
- Heap 由所有 JVM 进程中的线程共享；
- **类实例（对象）** 和**数组**空间从 heap 分配；
- JVM 启动时创建 Heap.

new 出来的对象空间都在 Heap 上分配，Heap 空间由 **GC（garbage collector）** 自动回收。
**根据 GC 回收的规则**，可将 Heap 空间细分为 Young Generation 和 Old Generation（存活的时长不同），具体如下图：
![[Heap的划分.png]]




### Method Area
**Conception**
Method Area stores **per-class structures** such as the *run-time constant pool, field and method data, and the code for methods and constructors, including the special methods* ([§2.9](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.9 "2.9. Special Methods")) used in class and instance initialization and interface initialization.

Method Area 是JVM中所有线程共享的，JVM启动时创建，**主要存储类的结构信息（metadata）**。
JDK1.8之前称为永久代，PermGen space。JDK1.8开始 Method Area 被称作 Metaspace(元空间)。

