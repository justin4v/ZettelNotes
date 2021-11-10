
# JSR 133
在 Java Specification Request 133（JSR 133，2004）中，定义如下：
- Java’s memory model works by examining **each read in an execution trace and checking that the write observed by that read is valid**；
- A high level, informal overview of the memory model shows it to **be a set of rules for when writes by one thread are visible to another thread**；
- The memory semantics **determine what values can be read** at every point in the program。

The actions of each thread in isolation must behave as governed by the semantics of that thread, with the exception that the values seen by each read are *determined by the memory model*。We say that the program obeys *intra-thread semantics*.
When threads interact, reads can return values written by writes from different threads.

# Wiki
- The Java Memory Model describes **how threads in the Java programming Language interact through memory**.
- The Java Memory Model(JMM) defines the allowable behavior of multithreaded programs, and therefore describes when such reorderings are possible. It **places execution-time constraints on the relationship between threads and main memory** order to achieve consistent and reliable Java applications. 
- By doing this , it makes it possible to **reason about  code exection in a multithreaded enviroment** , even in the face of **optimizations performed by the dynamic complier** ,the **processor** and the **caches**.

# 其他
JMM决定一个线程对共享变量的写入何时对另一个线程可见。

# 为什么需要JMM
## Reordering
重排序

## Happens-before