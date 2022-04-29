#JVM #Memory-structure #Todo 

# 简介
1. 寄存器：最快的存储区, 由编译器根据需求进行分配,程序中无法控制.  
2. 栈 stack：方法执行时创建方法栈帧：
	1. 存放*基本类型变量数据和对象的引用*；
	2. 对象本身不存放在栈中，而是存放在堆（new 出来的对象）或者常量池中（字符串常量对象存放在常量池中。）  
3. 堆 *Heap*：所有 *new 对象*。  
4. 静态域（*Method Area*）：存放*静态成员（static）*  
5. 常量池（*Method Area*）：存放*字符串常量和基本类型常量（public static final）*。  
6. 非RAM存储：硬盘等永久存储空间

# 运行时常量池
- 运行时常量池（Runtime Constant Pool）是方法区的一部分；

## 常量池
- 编译阶段确定；
- jvm 规范对 class 文件结构有严格的规范，必须符合规范的class文件才能被jvm装载；
- 字节码文件中包含：
	- 类的版本信息、字段、方法以及接口；
	- 还包含*常量池表（Constant Pool Table）*，包含*字面量*和对类型、域和方法的*符号引用*。
### 为什么需要常量池？
-  Java 中的字节码需要数据支持，如果数据很大则不能直接存到字节码里；
- 换另一种方式，可以存到常量池，这个字节码包含了指向常量池的引用。在动态链接的时候用到的就是运行时常量池。

# 参考
1. [JVM 变量存储位置](https://www.cnblogs.com/sw008/p/11054352.html)
2. [彻底弄懂java中的常量池](https://cloud.tencent.com/developer/article/1450501)
3. [姆级教程，2万字详解JVM](https://cloud.tencent.com/developer/article/1894039)
