# 定义
## 内存模型
- 给定一个程序和该程序的执行轨迹，**内存模型描述了该执行轨迹是否是该程序的一次合法执行**。A memory model describes, given a program and an execution trace of that program, whether the execution trace is a legal execution of the program. 
- 对于 Java，内存模型检查执行轨迹中的每次读操作，然后根据特定规则，检验该读操作观察到的写是否合法。Java’s memory model works by examining each read in an execution trace and checking that the write observed by that read is valid.
- 内存模型描述了某个程序的可能行为。JVM 的实现可以自由地生成想要的代码，只要该程序最终执行的结果都能通过内存模型进行预测。
- 内存模型的一个更高级、非正式的描述是：JMM 是一组规定了一个线程的写操作何时对另一个线程可见的规则。

## Wiki
- The Java Memory Model describes **how threads in the Java programming Language interact through memory** JMM描述了 java 程序中**线程之间如何通过内存交互**；
- The Java Memory Model(JMM) defines the allowable behavior of multithreaded programs, and therefore describes when such reorderings are possible. It **places execution-time constraints on the relationship between threads and main memory** ，in order to achieve consistent and reliable Java applications，JMM 定义了多线程程序允许的行为，进一步，描述了何时可能进行重排序。JMM 约束了**运行时线程和内存间关系**，以获得持续可靠的Java 应用程序. 
- By doing this , it makes it possible to **reason about  code exection in a multithreaded enviroment** , even in the face of **optimizations performed by the dynamic complier** ,the **processor** and the **caches**，通过约束线程和内存的关系， JMM 使得**多线程环境下对代码执行的推演成为可能**，即使面临动态编译器优化、处理器和缓存等因素。

## 其他
JMM决定一个**线程对共享变量的写入何时对另一个线程可见**。

总之，**JMM 是一套Java 规范（提出了一系列规则）**，不同平台上的 Java 实现（JVM等）需要**遵守 JMM 规范**，因此正确应用这套规范的（并发）程序能够**表现出正确的行为**。
所以说 JMM 对程序而言屏蔽了不同硬件处理器架构等底层细节，只需要遵循 JMM 规范即可。


# 参考

1. [[代码重排与乱序执行]]
2. [[Java 同步语义]]
3. [[正确的同步与安全的多线程]]
4. [JSR 133 (Java Memory Model) FAQ](https://blog.csdn.net/lemon89/article/details/73695204)
5. [JSR 133 (Java Memory Model) FAQ-上](https://blog.csdn.net/u012005313/article/details/81226956)
6.  闲话Java内存模型-微信文章