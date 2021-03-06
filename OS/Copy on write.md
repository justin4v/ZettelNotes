#Operating-system #Copy-on-write 

# cow 思想
 - **写入时复制（CopyOnWrite，简称COW）** 思想是计算机程序设计领域中的一种*通用优化策略*。
 - 核心思想是：
	 1. 如果有多个调用者（Callers）同时访问相同的资源（如内存或者是磁盘上的数据存储），会*获取同一份资源*；
	 2. 直到某个调用者*修改资源*内容时，才会复制一份*专用副本（private copy）给该调用者*；
	 3. 而其他调用者所见到的最初的资源仍然保持不变，是*透明的（transparently）*。
- 主要优点是：
	- *如果没有修改资源，不会有副本（private copy）被创建*；
	- 多个调用者*读取时共享同一份资源*。
	- 可用于**并发场景**。
- 通俗易懂的讲，cow 是不同进程在访问同一资源的时候，都是访问同一个资源；
- 只有更新操作，才会去复制一份新的数据并更新替换。

# linux 中的 cow
## fork
> fork()会产生一个和父进程完全相同的子进程(除了pid)

如果按**传统**的做法，会**直接**将父进程的数据拷贝到子进程中，拷贝完之后，父进程和子进程之间的数据段和堆栈是**相互独立的**。

但是，以我们的使用经验来说：往往子进程都会执行`exec()`来做自己想要实现的功能。

-   所以，如果按照上面的做法的话，创建子进程时复制过去的数据是没用的(因为子进程执行`exec()`，原有的数据会被清空)

既然很多时候复制给子进程的数据是无效的，于是就有了Copy On Write这项技术了，原理也很简单：

-   fork创建出的子进程，**与父进程共享内存空间**。也就是说，如果子进程**不对内存空间进行写入操作的话，内存空间中的数据并不会复制给子进程**，这样创建子进程的速度就很快了！(不用复制，直接引用父进程的物理空间)。
-   并且如果在fork函数返回之后，子进程**第一时间**exec一个新的可执行映像，那么也不会浪费时间和内存空间了。

另外的表达方式：

- 在fork之后exec之前两个进程**用的是相同的物理空间**（内存区），子进程的代码段、数据段、堆栈都是指向父进程的物理空间，也就是说，两者的虚拟空间不同，但其对应的**物理空间是同一个**。
-  当父子进程中**有更改相应段的行为发生时**，再**为子进程相应的段分配物理空间**。
- 如果不是因为exec，内核会给子进程的数据段、堆栈段分配相应的物理空间（至此两者有各自的进程空间，互不影响），而代码段继续共享父进程的物理空间（两者的代码完全相同）。
- 而如果是因为exec，由于两者执行的代码不同，子进程的代码段也会分配单独的物理空间。

## 实现原理
Copy On Write技术**实现原理：**
- fork()之后，kernel把父进程中所有的内存页的权限都设为read-only，然后子进程的地址空间指向父进程。
- 当父子进程都只读内存时，相安无事。当其中某个进程写内存时，CPU硬件检测到内存页是read-only的，于是触发页异常中断（page-fault），陷入kernel的一个中断例程。
- 中断例程中，kernel就会**把触发的异常的页复制一份**，于是父子进程各自持有独立的一份。

## 优点
-   COW技术可**减少**分配和复制大量资源时带来的**瞬间延时**。
-   COW技术可减少**不必要的资源分配**。比如fork进程时，并不是所有的页面都需要复制，父进程的**代码段和只读数据段都不被允许修改，所以无需复制**。

## 缺点
-   如果在fork()之后，父子进程都还需要继续进行写操作，**那么会产生大量的分页错误(页异常中断page-fault)**，这样就得不偿失。


# Java中的 cow
1. java 中 copyOnWriteArrayList 和 copyOnWriteArraySet；

## 原理
- CopyOnWriteArrayList/CopyOnWriteArraySet 容器在*查询时不需要加锁*；
- 在更新的时候，会从原来的数据复制一个副本，然后*修改这个副本*；
- 最后用*修改后的副本替换原数据*；
- 修改的同时，*读操作不会被阻塞*，而是*读取旧的数据*。和读写锁区分开。

## 源码
1. CopyOnWriteArrayList 的 add()
 
```java
    public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
```

2. CopyOnWriteArrayList 的 get(int index)，**无锁**
```java
    public E get(int index) {
        return get(getArray(), index);
    }
    
    @SuppressWarnings("unchecked")
    private E get(Object[] a, int index) {
        return (E) a[index];
    }    

```

## 优点
- 适用于**读多写少**场景，是一种*无锁的实现*，可以实现*更高的并发*；
	- 例如**配置、黑名单、物流地址**等变化非常少的数据。
- CopyOnWriteArrayList 并发安全且性能比 Vector 好。
	- Vector 是增删改查方法都加了 synchronized 来保证同步；
	- CopyOnWriteArrayList 只是在增删改上加锁，读不加锁，读性能就好于 Vector。

## 缺点
1. 只保证**数据的最终一致性**。
	- 在添加到拷贝数据而还没进行替换的时候，读到的仍然是旧数据。
2. **内存占用**。
	1. 果对象比较大，频繁地进行替换会**消耗内存**，从而引发 Java 的 GC 问题’
	2. 此时应该考虑*其他并发容器*，例如 *ConcurrentHashMap*。

# 参考
1. [理解Copy on write机制](https://www.jianshu.com/p/2d30dce24bdb)
2. [写入时复制（CopyOnWrite）](https://www.cnblogs.com/jmcui/p/12377081.html)