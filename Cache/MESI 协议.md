#Todo #MESI #Cache #Cache-coherency

# cache

- 高速缓存（cache）是一种存取速率远比主内存大而容量远比主内存小的存储部件，每个处理器都有其高速缓存。
- 引入 cache 之后， 处理器在执行内存读、 写操作的时候并*不直接与主内存打交道*， 而是通过高速缓存进行的。
- cache 的内部结构相当于一个拉链散列表(ChainedHash Table). 它包含若干桶(Bucket, 硬件上称之为Set), 每个桶又可以包含若干缓存条目(CacheEntry) .




# 参考
1. [深入学习缓存一致性问题和缓存一致性协议MESI(一) ](https://www.jianshu.com/p/5e860ffd6912)
2. [深入学习缓存一致性问题和缓存一致性协议MESI(一)](https://www.cnblogs.com/dongc/p/12271042.html)
3. [[CPU 缓存]]