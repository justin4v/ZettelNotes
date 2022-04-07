#Cache #CPU-cache
# 简介
- CPU缓存： **解决 CPU 运算速度与内存读写速度不匹配的矛盾**（也是[[0.程序性能的限制与解决|程序性能限制的根源]]）
- 是**CPU与内存之间的临时数据交换器**，*容量小的多、但是交换速度却快得多*。
- CPU Cache 一般*直接和 CPU芯片集成或位于主板总线互连的独立芯片*上。

## Cache Line
- 为了简化与内存之间的通信，Cache 控制器是**针对数据块，而不是字节**进行操作。
- Cache 是一组称为**缓存行(Cache Line) 的固定大小的数据块组成**，典型的一行是 `64` 字节

# 局部性原理
- 在 CPU 访问存储设备时，*存取数据或者存取指令，都趋于聚集在一片连续的区域中*，这就被称为**局部性原理**。

> **时间局部性（Temporal Locality）**：如果一个信息项正在被访问，那么*近期很可能还会被再次访问*。

比如*循环、递归、方法的反复调用*等。

> **空间局部性（Spatial Locality）**：如果一个存储器的位置被引用，那么*将来附近的位置也可能会被引用*。

比如*顺序执行的代码、连续创建的两个对象、数组*等。

# 三级缓存
- CPU Cache 通常分成了三个级别：`L1(Level 1)`，`L2(Level 2)`，`L3(Level 3)`。
- 级别越小越接近 CPU，速度越快。
- CPU 读取数据时，*查找顺序是： L1 Cache => L2 Cache => L3 Cache / Memory*。
- 一般来说，每级缓存的命中率大概都在 80%左右，全部数据量的 80% 都可以在 L1 Cache 中找到，只剩下 20% 的总数据量需要 L2 、L3 或 Memory 中读取；
- 可见 L1 Cache 是 CPU 缓存架构中最重要的部分。

# L1 Cache
- 一级缓存（Level 1 Cache），位于 CPU 内核的旁边，与 CPU 结合最为紧密，也是历史上最早出现的CPU缓存。
- 一般来说，一级缓存可以分为**一级数据缓存（Data Cache，D-Cache）** 和**一级指令缓存（Instruction Cache，I-Cache）**。
- **L1 D-Cache 用于存放数据**；
- **L1 I-Cache 用于对操作数据的指令进行即时解码**；
- **两者可以同时被 CPU 访问**，减少了争用 Cache 所造成的冲突，提高了处理器效能。
- 大多数CPU *L1 D-Cache  和 L1 I-Cache 具有相同的容量*。


# L2/L3 Cache
- *L2 Cache 是 L1 Cache 的缓冲器*：L1 Cache 制造成本很高、容量有限，L2 Cache 存储那 CPU 处理时需要用到、L1 Cache 又无法存储的数据。
- *L3 Cache 和 Memory 是 L2 Cache 的缓冲器*；
- *L2/L3 Cache 或 Memory 都不能存储处理器操作的原始指令*，原始指令**只能存储在 CPU 的 L1 I-Cache 中**，而 L2/L3 Cache 或 Memory 仅用于存储 CPU 所需数据。


# Cache 工作原理
- 当 CPU 要读取数据时，*首先从 Cache 中查找*，如果找到就读取并送给 CPU 处理；
- 如果没有找到，就*从 Memory 中读取*；
- 同时*把数据所在的数据块存入 Cache*，以后对整块数据的读取都从 Cache 进行，不必再调用 Momery。 
- 该读取机制使 CPU 读取缓存的*命中率非常高（可达90%左右）*，也就是说CPU下一次要读取的数据90%都在缓存中，只有大约 10% 需要从内存读取。

## 多核CPU多级缓存结构
![[多核 CPU 多级缓存结构示意.png]]

- 多核 CPU 的情况下有**多个 L1 Cache**，如何保证 Cache 内部数据的一致性？ **缓存一致性协议 [[MESI 协议]]**


# 参考
1. [缓存一致性（Cache Coherency）入门](https://www.infoq.cn/article/cache-coherency-primer)