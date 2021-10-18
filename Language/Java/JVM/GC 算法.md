# GC 算法
## 主要关注点

-   对象存活判断
-   GC算法
-   垃圾回收器

## 对象存活判断

判断对象是否存活一般有两种方式：

-   **引用计数**：每个对象有一个引用计数属性，新增一个引用时计数加1，引用释放时计数减1，计数为0时可以回收。此方法简单，无法解决对象**相互循环引用的问题**。
-   **可达性分析**（Reachability Analysis）：从 [[#GC ROOT]] 开始向下搜索，搜索所走过的路径称为引用链。当一个对象到 GC Roots 没有任何引用链相连时，则证明此对象是不可用的，**不可达对象**。即给定一个集合的引用作为根出发，通过引用关系遍历*对象图*，能被遍历到的（可到达的）对象就被判定为存活，没有被遍历到的就自然被判定为死亡

## GC算法

GC最基础的算法有三种：标记 -清除算法、复制算法、标记-压缩算法，我们常用的垃圾回收器一般都采用分代收集算法。

-   **标记 -清除算法（Mark-Sweep）**：算法分为“标记”和“清除”两个阶段：首先标记出所有需要回收的对象，在标记完成后统一回收掉所有被标记的对象。
-   **复制算法（Coping）**：将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用过的内存空间一次清理掉。
-   **标记-压缩算法（Mark-Compact）**：标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存
-   **分代收集算法（Generational Collection）**：把Java堆分为新生代和老年代，这样就可以根据各个年代的特点采用最适当的收集算法。

### 复制算法
- 特点：适用于新生代（Young Generation）

- JVM 把新生代分为了三部分：1个 Eden 区和2个 Survivor 区（分别叫 *from 和 to*），默认比例为8:1:1 （参考：[[JVM内存结构#Heap]]）。

- 一般情况下，新创建的对象会被分配到 Eden 区（一些大对象特殊处理），经过第一次 Minor GC 后，如果仍然存活，将会被移到 Survivor 区。
- 对象在Survivor区中每熬过一次Minor GC，年龄 +1，当年龄（参考：[[对象结构#age]]）增加到一定程度时（*默认是最大值 15* ，-XX:MaxTenuringThreshold 设定），就会被移动到年老代（Old Generation）中。
- 新生代中的对象基本都是朝生夕死（被GC回收率90%以上），所以在**新生代的垃圾回收算法是复制算法**。

#### 步骤
复制算法是将原有的内存空间分成两块，每次只使用其中的一块（`from`）。
- 在GC开始的时候，对象只会存在于 Eden 区和名为 from 的Survivor 区，Survivor 区 to 是空的。
- 接着进行 GC，Eden 区中所有存活的对象都会被复制到 to;
- 在 from 区中，仍存活的对象会根据他们的年龄值来决定去向。年龄达到一定值（默认15）的对象会被移动到老年代中，没有达到阈值的对象会被复制到 to 区域。
- 这次GC后，Eden 区和 from 区已经被清空。
- 之后，from 和 to 交换角色，GC前的 from -> to，to-> from。保证 Survivor - to 区域是空的。 Minor GC 一直重复这样的过程，直到 to 区被填满。
- to 区被填满之后，将所有对象移动到老年代中。

#### 优缺点
1. 优点
	1. 不会产生内存碎片，效率高。
2. 缺点
	1. 耗费内存空间
	2. 不适用于存活对象较多的情况

![[复制算法.png]]

![[Copying算法.gif]]
**其中**：
- 空闲区域代表 to-Survivor；
- 激活区域代表 from-Survivor；
- 绿色代表不被回收的；
- 红色代表被回收的。

### 标记-清除算法
这是一个非常基本的GC算法，它是现代GC算法的思想基础。

#### 步骤
分为**标记**和**清除**两个阶段：
1. 先把所有活动的对象标记出来；
2. 然后把没有被标记的对象统一清除掉。

#### 优缺点
1. 优点
	1. 不需要额外的内存空间。
2. 缺点
	1. 需要暂停整个应用（Stop-The-World），会产生内存碎片；
	2. 两次扫描，耗时严重，效率不高。

![[标记-清除算法.png]]

![[Mark-Sweep.gif]]

### 标记整理算法

标记整理算法适用于存活对象较多的场合，它的标记阶段和标记-清除算法中的一样。整理阶段是将所有存活的对象压缩到内存的一端，之后清理边界外所有的空间。它的效率也不高。

![[标记-整理算法.png]]

## 垃圾回收器

-   Serial收集器，串行收集器是最古老，最稳定以及效率高的收集器，可能会产生较长的停顿，只使用一个线程去回收。
-   ParNew收集器，ParNew收集器其实就是Serial收集器的多线程版本。
-   Parallel收集器，Parallel Scavenge收集器类似ParNew收集器，Parallel收集器更关注系统的吞吐量。
-   Parallel Old 收集器，Parallel Old是Parallel Scavenge收集器的老年代版本，使用多线程和“标记－整理”算法
-   CMS收集器，CMS（Concurrent Mark Sweep）收集器是一种以获取最短回收停顿时间为目标的收集器。
-   G1收集器，G1 (Garbage-First)是一款面向服务器的垃圾收集器,主要针对配备多颗处理器及大容量内存的机器. 以极高概率满足GC停顿时间要求的同时,还具备高吞吐量性能特征


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
