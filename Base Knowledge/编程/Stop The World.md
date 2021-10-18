# Stop The World
1. Stop-The-World，简称STW，指某个 JVM 事件，如 GC（参考[[GC 算法]]）JIT编译（参考[[JIT 编译器]]）发生过程中，整个 JVM 所有线程暂停的现象。
2、所有的 GC 都有这个事件。
3、G1也不能完全避免stop-the-world情况发生，尽可能地缩短了暂停时间。
4、STW是JVM在后台自动发起和自动完成的。在用户不可见的情况下，把用户正常的工作线程全部停掉。开发中不要用System.gc() ;会导致stop-the-world的发生。

# 为什么需要STW(stop the world)
垃圾回收是根据可达性分析算法，搜索GC Root根的引用链，将不在引用链上的对象当做垃圾回收，如果回收过程中程序重新引用了待回收对象，将会造成错误。

Key reason why compaction leads to STW pause is as follows：
**JVM needs to move object and update references to it**. 
1. if you move object before updating the references and application that is running access it from old reference , there will be trouble.
2. if you update reference first and than try to move object the updated reference is wrong till object is moved and any access while object has not moved will cause issue.

For both CMS and Parallel collecter the young generation collection algorithm is similar and it is stop the world ie application is stopped when collection is happening Stuff JVM is doing is, marking all objects reachable from root set, moving the objects from Eden to survivor space and moving objects that have survived collections beyond tenuring threshold to the old generation. Of course JVM has to update all the references to the objects that have moved.

For old generation parallel collector is doing all marking, compaction and reference updates in a single stop the world(STW) phase, this leads to pauses in seconds for heaps in GBs. This was painful for the applications that have strict response time requirements. Till date Paralle collector is still the best collectors(among Oracle Java) for throughput or batch processing. In fact we have seen for same scenario even if time spent in pauses is more in parallel collector than CMS still we get a higher throughput, this I think has to do with better spatial locality due to compaction.

CMS solved the problem of high pauses in major collection by doing the Marking concurrently. There are 2 STW parts, Initial marking (getting the references from root set) and Remark Pause (a small STW pause at the end of marking to deal with changes in the object graph while marking and application was working concurrently). Both these pauses are in range of 100 -200 milliseconds for few GB of heap sizes and reasonable number of application threads(remember more active threads more roots)

G1GC is planned to be a replacement of CMS and accept goals for pauses. takes care of fragmentation by incrementally compacting the heap.Though the work is incremental so you can get smaller pauses but that may come at the cost of more frequent pauses

None of the above can compact heap(CMS does not compact at all) while application is running. AZUL GPGC garbage collection can even compact without stopping the application and also handle reference update. So if you want to go deep into how GCs work it will be worth reading the algorithm for GPGC. AZUL markets it as a pause-less collector.

## 参考
[Does Java Garbage Collect always has to "Stop-the-World"?](https://stackoverflow.com/questions/40182392/does-java-garbage-collect-always-has-to-stop-the-world)





