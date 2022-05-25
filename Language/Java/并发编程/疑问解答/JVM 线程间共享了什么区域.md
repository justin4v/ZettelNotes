#Java基础 #JVM #Multi-threads #Thread-safe #TLAB  #Problem-and-Solutions 

# 线程
- 从代码角度看，线程的本质是函数的运行，stack 是每个线程（函数）独有的。
- JVM 中线程间共享 [[JVM内存结构#Heap|Heap]] 和 [[JVM内存结构#Method Area|Method Area]]；

>1、堆是线程共享的内存区域，栈是线程独享的内存区域。
>2、堆中主要存放对象实例，栈中主要存放各种基本数据类型、对象的引用。



# 参考
1. [[JVM内存结构]]
2. [[JVM变量存储]]
3. [[2.线程]]
4. [[3.线程模型]]
5. [[1.进程]]
6. [Java堆内存是线程共享的！面试官：你确定吗？](https://www.cnblogs.com/hollischuang/p/12453988.html)