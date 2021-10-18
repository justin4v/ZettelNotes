# Stop The World
Stop一the一World，简称STW，指的是Gc事件发生过程中，会产生应用程序的停顿。停顿产生时整个应用程序线程都会被暂停，没有任何响应，有点像卡死的感觉，这个停顿称为STW。
举例：

可达性分析算法中枚举根节点（GC Roots）会导致所有Java执行线程停顿。

停顿的原因
分析工作必须在一个能确保一致性的快照中进行
一致性指整个分析期间整个执行系统看起来像被冻结在某个时间点上
如果出现分析过程中对象引用关系还在不断变化，则分析结果的准确性无法保证

2、STW事件和采用哪款GC无关，所有的GC都有这个事件。

3、哪怕是G1也不能完全避免stop-the-world情况发生，只能说垃圾回收器越来越优秀，回收效率越来越高，尽可能地缩短了暂停时间。

4、STW是JVM在后台自动发起和自动完成的。在用户不可见的情况下，把用户正常的工作线程全部停掉。开发中不要用System.gc() ;会导致stop-the-world的发生。

# 为什么需要STW(stop the world)
垃圾回收是根据可达性分析算法，搜索GC Root根的引用链，将不在引用链上的对象当做垃圾回收，设想我们执行某个方法的时候，此时产生了很多局部变量，刚好老年代满了需要进行Full gc，如果不停止线程，垃圾回收正在根据这些局部变量也就是GC Root根搜索引用链，此时这个方法结束了，那么这些局部变量就都会被销毁，这些引用链的GC Root根都销毁了，这些引用当然也成了垃圾对象，这样就会导致在垃圾回收的过程中还会不断的产生新的垃圾。

但是Stop-The-World的结果是比较严重的，如果用户正在浏览你的网站，应用程序突然Stop-The-World，所有线程被挂起，那么用户就会感觉你的网站卡住了，尽管gc时间是比较快的，但是如果并发量比较大，用户感知是比较明显的，会影响用户体验。
Key reason why compaction leads to STW pause is as follows, JVM needs to move object and update references to it. now if you move object before updating the references and application that is running access it from old reference than there is trouble. if you update reference first and than try to move object the updated reference is wrong till object is moved and any access while object has not moved will cause issue.

For both CMS and Parallel collecter the young generation collection algorithm is similar and it is stop the world ie application is stopped when collection is happening Stuff JVM is doing is, marking all objects reachable from root set, moving the objects from Eden to survivor space and moving objects that have survived collections beyond tenuring threshold to the old generation. Of course JVM has to update all the references to the objects that have moved.

For old generation parallel collector is doing all marking, compaction and reference updates in a single stop the world(STW) phase, this leads to pauses in seconds for heaps in GBs. This was painful for the applications that have strict response time requirements. Till date Paralle collector is still the best collectors(among Oracle Java) for throughput or batch processing. In fact we have seen for same scenario even if time spent in pauses is more in parallel collector than CMS still we get a higher throughput, this I think has to do with better spatial locality due to compaction.

CMS solved the problem of high pauses in major collection by doing the Marking concurrently. There are 2 STW parts, Initial marking (getting the references from root set) and Remark Pause (a small STW pause at the end of marking to deal with changes in the object graph while marking and application was working concurrently). Both these pauses are in range of 100 -200 milliseconds for few GB of heap sizes and reasonable number of application threads(remember more active threads more roots)

G1GC is planned to be a replacement of CMS and accept goals for pauses. takes care of fragmentation by incrementally compacting the heap.Though the work is incremental so you can get smaller pauses but that may come at the cost of more frequent pauses

None of the above can compact heap(CMS does not compact at all) while application is running. AZUL GPGC garbage collection can even compact without stopping the application and also handle reference update. So if you want to go deep into how GCs work it will be worth reading the algorithm for GPGC. AZUL markets it as a pause-less collector.

## 参考
[Does Java Garbage Collect always has to "Stop-the-World"?](https://stackoverflow.com/questions/40182392/does-java-garbage-collect-always-has-to-stop-the-world)





