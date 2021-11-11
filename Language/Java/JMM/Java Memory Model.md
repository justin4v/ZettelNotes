
# JSR 133
在 Java Specification Request 133（JSR 133，2004）中，定义如下：
- The Java Memory Model describes what behaviors are legal in multithreaded code, and how threads may interact through memory，Java 内存模型描述了**在多线程代码中，哪些行为是合法的，以及线程之间如何通过内存交互**；
- A high level, informal overview of the memory model shows it to **be a set of rules for when writes by one thread are visible to another thread**，从一个更高、非正式的视角看，内存模型是**一系列描述一个线程的写入何时对另一个线程可见的规则**；
- The memory semantics **determine what values can be read** at every point in the program，内存语义决定了*程序中每一时刻哪些值可以被读取*。

The actions of each thread in isolation must behave as governed by the semantics of that thread, with the exception that the values seen by each read are *determined by the memory model*。We say that the program obeys *intra-thread semantics*.

# Wiki
- The Java Memory Model describes **how threads in the Java programming Language interact through memory** JMM描述了 java 程序中线程之间如何通过内存交互；
- The Java Memory Model(JMM) defines the allowable behavior of multithreaded programs, and therefore describes when such reorderings are possible. It **places execution-time constraints on the relationship between threads and main memory** order to achieve consistent and reliable Java applications. 
- By doing this , it makes it possible to **reason about  code exection in a multithreaded enviroment** , even in the face of **optimizations performed by the dynamic complier** ,the **processor** and the **caches**.

# 其他
JMM决定一个**线程对共享变量的写入何时对另一个线程可见**。

总之，**JMM 是一套Java 规范（提出了一系列规则）**，不同平台上的 Java 实现（JVM等）需要**遵守 JMM 规范**，因此正确应用这套规范的（并发）程序能够**表现出正确的行为**。
所以说 JMM 对程序而言屏蔽了不同硬件处理器架构等底层细节，只需要遵循 JMM 规范即可。

# 为什么需要JMM
## Reordering
重排序

## Happens-before


# 参考
1. [JSR 133 (Java Memory Model) FAQ](https://blog.csdn.net/lemon89/article/details/73695204)
2. [JSR 133 (Java Memory Model) FAQ-上](https://blog.csdn.net/u012005313/article/details/81226956)