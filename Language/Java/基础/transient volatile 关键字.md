#Java基础 #Volatile #Transient #Todo 


# volatile

一旦一个共享变量（类的成员变量、类的静态成员变量）被volatile修饰之后，那么就具备了两层语义：

1）保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。

2）禁止进行指令重排序。

# 参考
1. [Java并发编程：volatile关键字解析](https://www.cnblogs.com/dolphin0520/p/3920373.html)
2. [Java transient关键字使用小记 ](https://www.cnblogs.com/lanxuezaipiao/p/3369962.html)