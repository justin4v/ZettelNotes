#Java基础 #GC #JVM 

# 分代收集理论
[[分代收集理论]]

# GC 算法
## 主要关注点

-   **对象存活判断**
-   **GC算法**
-   **垃圾回收器**

## 对象存活判断

判断对象是否存活一般有两种方式：
### 引用计数
- 每个对象有一个*引用计数属性*；
- *新增引用时计数加1*；
- *引用释放时计数减1*，
- *计数为0时可以回收*。

此方法简单，无法解决对象**相互循环引用的问题**。
- 对象objA和objB都有字段 instance，令 objA.instance=objB 及 objB.instance=objA；
- 除此之外，*两个对象再无任何引用*，实际上这两个对象已经不可能再被访问；
- 但是它们因为互相引用着对方，导致引用计数都不为零，*引用计数算法无法回收它们*。
```java
/**
* testGC()方法执行后，objA和objB会不会被GC呢？
* @author zzm
*/
public class ReferenceCountingGC {
	public Object instance = null;
	private static final int _1MB = 1024 * 1024;
	
	/**
	* 这个成员属性的唯一意义就是占点内存，以便能在GC日志中看清楚是否有回收过
	*/
	private byte[] bigSize = new byte[2 * _1MB];
	public static void testGC() {
		ReferenceCountingGC objA = new ReferenceCountingGC();
		ReferenceCountingGC objB = new ReferenceCountingGC();
		objA.instance = objB;
		objB.instance = objA;
		objA = null;
		objB = null;
		// 假设在这行发生GC，objA和objB是否能被回收？
		System.gc();
	}
}
```
结果
```java
[Full GC (System) [Tenured: 0K->210K(10240K), 0.0149142 secs] 4603K->210K(19456K), [Perm : 2999K->2999K(21248K)], 0.0150007 secs] [Times: user=0.01 sys=0.00, real=0.02 secs]
Heap
	def new generation total 9216K, used 82K [0x00000000055e0000, 0x0000000005fe0000, 0x0000000005fe0000)
	Eden space 8192K, 1% used [0x00000000055e0000, 0x00000000055f4850, 0x0000000005de0000)
	from space 1024K, 0% used [0x0000000005de0000, 0x0000000005de0000, 0x0000000005ee0000)
	to space 1024K, 0% used [0x0000000005ee0000, 0x0000000005ee0000, 0x0000000005fe0000)
	tenured generation total 10240K, used 210K [0x0000000005fe0000, 0x00000000069e0000, 0x00000000069e0000)
	the space 10240K, 2% used [0x0000000005fe0000, 0x0000000006014a18, 0x0000000006014c00, 0x00000000069e0000)
	compacting perm gen total 21248K, used 3016K [0x00000000069e0000, 0x0000000007ea0000, 0x000000000bde0000)
	the space 21248K, 14% used [0x00000000069e0000, 0x0000000006cd2398, 0x0000000006cd2400, 0x0000000007ea0000)
	No shared spaces configured.
```
- 内存回收日志中*包含“4603K->210K”*，虚拟机并没有因为两个对象互相引用就放弃回收它们;
- 从侧面说明了Java虚拟机并*不是通过引用计数算法*来判断对象是否存活的。


### 可达性分析（Reachability Analysis）

- 当前主流的商用程序语言（Java、C#、Lisp）内存管理子系统，都是通过可*达性分析（Reachability Analysis）* 算法来判定对象是否存活;
- 从 [[#GC ROOT]] 开始向下搜索，搜索所走过的路径称为引用链。
- 当一个对象到 GC Roots 没有任何引用链相连时，则证明此对象是不可用的，**不可达对象（图论）**。
- 即给定一个 *GC Roots Set* 出发，通过引用关系遍历*对象图*，**能被遍历到的（可到达的）对象就被判定为存活，没有被遍历到的就自然被判定为死亡**。

![[GC root判定存活示意.png]]

在Java技术体系里面，*固定可作为 GC Roots 的对象*包括以下几种：
- *虚拟机栈（栈帧中的本地变量表）中引用的对象*。譬如各个线程被调用的方法堆栈中使用到的参数、局部变量、临时变量等。
- *方法区中类静态属性引用的对象*。譬如Java类的引用类型静态变量。
- *方法区中常量引用的对象*。譬如字符串常量池（String Table）里的引用。
- *本地方法栈中JNI（即通常所说的Native方法）引用的对象*。
- *Java 虚拟机内部的引用*。如基本数据类型对应的Class对象，一些常驻的异常对象（比如NullPointExcepiton、OutOfMemoryError）等，还有系统类加载器。
- *所有被同步锁（synchronized关键字）持有的对象*。
- 反映Java虚拟机内部情况的JMXBean、JVMTI中注册的回调、本地代码缓存等。
- 除了固定的 GC Roots 集合外，根据用户所选用的垃圾收集器以及当前回收的内存区域不同，还可以有其他对象“临时性”地加入。*某个区域的对象有可能被堆中其他区域的对象引用*，这时候就需要将这些关联区域的对象也一并加入GC Roots集合中。

## GC算法

GC最基础的算法有四种：复制算法、标记 -清除算法、标记-压缩算法，分代收集算法。

-   **标记 -清除算法（Mark-Sweep）**：算法分为 *“标记”和“清除”* 两个阶段：
	1. 首先标记出所有需要回收的对象；
	2. 在标记完成后统一回收掉所有被标记的对象。
-   **标记-复制算法（Coping）**：简称*复制算法*，将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用过的内存空间一次清理掉。
-   **标记-整理算法（Mark-Compact）**：标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存
-   **分代收集算法（Generational Collection）**：把Java堆分为新生代和老年代，这样就可以根据各个年代的特点采用最适当的收集算法。

### 复制算法
1969年Fenichel提出了一种称为“半区复制”（Semispace Copying）的垃圾收集算法，它将可用
内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着
的对象复制到另外一块上面，然后再把已使用过的内存空间一次清理掉。如果内存中多数对象都是存
活的，这种算法将会产生大量的内存间复制的开销，但对于多数对象都是可回收的情况，算法需要复
制的就是占少数的存活对象，而且每次都是针对整个半区进行内存回收，分配内存时也就不用考虑有
空间碎片的复杂情况，只要移动堆顶指针，按顺序分配即可。这样实现简单，运行高效，不过其缺陷
也显而易见，这种复制回收算法的代价是将可用内存缩小为了原来的一半。


- 特点：**适用于新生代（Young Generation）**

- JVM 把新生代分为了三部分：1个 Eden 区和2个 Survivor 区（分别叫 *from 和 to*），默认比例为8:1:1 （参考：[[JVM内存结构#Heap]]）。

- 一般情况下，**新创建的对象会被分配到 Eden 区**（一些大对象特殊处理），经过第一次 Minor GC 后，如果仍然存活，将会被移到 **Survivor to** 区。
- 对象在 Survivor 区中每熬过一次Minor GC，年龄 +1，当年龄（参考：[[对象结构#age]]）增加到一定程度时（*默认是最大值 15* ，-XX:MaxTenuringThreshold 设定），就会被移动到年老代（Old Generation）中。
- 新生代中的对象基本都是朝生夕死（被GC回收率90%以上），所以在**新生代的垃圾回收算法是复制算法，每次死亡的对象占大多数，不需要很大的 Survivor 空间**。

#### 步骤
复制算法是将原有的内存空间分成两块，每次只使用其中的一块（`from`）。
- 一开始，对象只会存在于 Eden 区（对象新建时分配空间）和 Survivor from 区（*上次 GC 后 to 变成了 from*），Survivor  to 是空的。
- 接着进行 GC，Eden 区中所有存活的对象都会被复制到 to;
- 在 from 区中，仍存活的对象根据他们的年龄值来决定去向：1. 年龄达阈值（默认15）的对象被移动到老年代中，没有达到阈值的对象会被复制到 to 区域。
- GC后，Eden 区和 from 区已经被清空。
- 之后，from 和 to 交换角色，GC前的 from -> to，to-> from。保证 Survivor  to 区域是空的。 Minor GC 一直重复这样的过程，直到 to 区被填满。
- to 区被填满之后，将所有对象移动到老年代中。

#### 优缺点
1. 优点
	1. 不会产生内存碎片，效率高。
2. 缺点
	1. 耗费内存空间；
	2. 不适用于存活对象较多的情况（需要分配较大的 Survivor 空间保存存活对象 ）。

![[复制算法.png]]

![[Copying算法.gif]]
**其中**：
- 空闲区域代表 to-Survivor；
- 激活区域代表 from-Survivor；
- 绿色代表不被回收的；
- 红色代表被回收的。

### 标记-清除算法
- 特点：基本的GC算法，它是现代GC算法的思想基础。

#### 步骤
分为**标记**和**清除**两个阶段：
1. 先把所有活动的对象标记出来；
2. 把没有被标记的对象统一清除掉。

#### 优缺点
1. 优点
	1. 不需要额外的内存空间。
2. 缺点
	1. *执行效率不稳定*，如果Java堆中包含大量对象，而且其中大部分是需要被回收的，这时必须进行大量标记和清除的动作，导致*标记和清除的执行效率都随对象数量增长而降低*；
	2. *内存空间的碎片化*，标记、清除之后会产生*大量不连续的内存碎片（存活对象没有移动）*，空间碎片太多可能使得程序运行过程中需要分配较大对象时，*无法找到足够的连续内存而不得不提前触发另一次垃圾收集动作*。

![[标记-清除算法.png]]

![[Mark-Sweep.gif]]

### 标记-整理算法
- 特点：**适用于老年代**

#### 步骤
1. 标记
2. 整理

#### 优缺点
1. 优点
	1. 没有内存碎片
2. 缺点
	1. 移动对象的成本，效率也不高（要标记所有存活对象，还要整理所有存活对象的引用地址）；
	2. *移动存活对象(老年代有大量存活对象)并更新所有引用这些对象的地方*将会是一种极为负重的操作，而且对象移动操作*必须全程暂停用户应用程序*才能进行；
	3. 这样的停顿被最初的虚拟机设计者形象地描述为“[[Stop The World]]”。


![[mark-compact.gif]]

![[标记-压缩算法.png]]

### 分代收集
- 当前商业虚拟机普遍采用；
- 根据对象存活周期的不同将*内存划分为几块*。
- 一般是把Java堆分为新生代和老年代，然后根据各个年代的特点采用最适当的垃圾收集算法。
	1. 在**新生代**中，每次垃圾收集都发现有大批对象死去，只有少量存活，选用**复制算法**；
	2. **老年代**因为对象存活率高，没有额外空间对它进行分配担保，须使用**标记清除或者标记压缩算法**来进行回收。

![[分代收集算法示意.png]]

## 垃圾回收器

-  **Serial 收集器**：串行收集器是最古老，最稳定以及效率高的收集器，可能会产生较长的停顿，只使用一个线程去回收。 ^1b5410
-   **ParNew 收集器**：ParNew收集器其实就是Serial收集器的多线程版本。 ^838a71
-   **Parallel 收集器**：Parallel Scavenge收集器类似ParNew收集器，Parallel 收集器更关注系统的吞吐量。
-   **Parallel Old 收集器**：Parallel Old是Parallel Scavenge收集器的老年代版本，使用多线程和“标记－整理”算法
-   **CMS 收集器**：**CMS（Concurrent Mark Sweep）** 收集器是一种以获取最短回收停顿时间（Stop-The-World）为目标的收集器。 ^e81d32
-   **G1收集器**：**G1 (Garbage-First)** 是一款面向服务器的垃圾收集器，主要针对配备多颗处理器及大容量内存的机器。以极高概率满足GC停顿时间要求的同时,还具备高吞吐量性能特征。 ^d3fbb8

参考：
[JVM 的 CMS](https://www.cnblogs.com/plxx/p/4527065.html)
# 算法比较
1. **效率**： 复制算法 > 标记清除算法 > 标记整理算法（对比时间复杂度）。
2. **内存整齐度**： 复制算法 = 标记整理算法 > 标记清除算法。
3. **内存利用率**： 标记整理算法 = 标记清除算法 > 复制算法（复制算法需要额外内存）。

- 复制算法效率最高，但是需要较多内存，
- 标记整理算法相对兼顾上面所提到的三个指标，但效率较低。

# GC ROOT

GC Roots 实际上是**当前 JVM 中必须存活的对象（不会被回收）**，则处于该引用链上的对象也需要存活。
在Java中，可以作为 GC Roots 的对象包括下面几种： 

-   虚拟机栈中引用的对象； 
-   方法区中类静态属性引用的对象； 
-   方法区中的常量引用的对象； 
-   本地方法栈中JNI（即一般说的Native方法）的引用的对象；


## 实际存活判断
不可达的对象也并非“非死不可”的，这时候它们暂时处于“缓刑”阶段，要真正宣告一个对象的死亡，只是要经历两次标记过程：

1. 如果对象在进行根搜索后发现没有与 GC Roots 相连接的引用链，那它会被第一次标记并且进行一次筛选，筛选的条件是此对象**是否有必要执行finalize（）方法**。当对象没有覆盖 finalize（）方法，或者 finalize（） 方法已经被虚拟机调用过，则不会再执行 finalize（）。此时对象再也没有“自救”机会。

2. 如果对象被判定为有必要执行finalize（）方法，那么这个对象将会被放在名为 F-Queue 的队列中，并在稍后一条由虚拟机自动建立的、低优先级的 [[JVM 重要线程#Finalizer Thread]] 去执行。这里所谓的执行是指虚拟机会触发这个方法，但并不承诺会等待进行结束。 
3. **finalize（）方法是对象逃脱死亡的最后一次机会**，并且只会有一次。稍后GC将对F-Queue中的对象进行第二次小规模的标记，如果对象要想在finalize（）中成功拯救自己，只要**重新与引用链上任何一个对象建立关联**，譬如把自己（this关键字）赋值给某个类变量或对象的成员变量，那在第二次标记时它将被移除出“即将回收”的集合。

## 示例
```java
public class FinalizeEscapeGC {
 
    private static FinalizeEscapeGC SAVE_HOOK = null;
 
    private void isAlive() {
        System.out.println("Yes,I am still alive!");
    }
 
    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("finalize() method executed!");
        FinalizeEscapeGC.SAVE_HOOK = this;
    }
 
    public static void main(String args[]) throws Throwable {
        SAVE_HOOK = new FinalizeEscapeGC();
        // 对象第一次成功拯救自己
        SAVE_HOOK = null;
        // 调用该方法建议系统执行垃圾清理，但也并不一定执行
        System.gc();
        // 因为Finalize线程优先级较低，暂停0.5秒以等待它
        Thread.sleep(500);
        if (SAVE_HOOK != null) {
            SAVE_HOOK.isAlive();
        } else {
            System.out.println("No, I am dead!");
        }
        // 下面这段代码与上面完全相同，但这次却自救失败了
        SAVE_HOOK = null;
        System.gc();
        Thread.sleep(500);
        if (SAVE_HOOK != null) {
            SAVE_HOOK.isAlive();
        } else {
            System.out.println("No, I am dead!");
        }
    }
}
```

运行结果：   
```plaintext
finalize() method executed!   
Yes,I am still alive!   
No, I am dead! 
```

由此可以看出，通过*finalize() 方法确实可以自救，但只会调用一次*，不会有第二次机会。
