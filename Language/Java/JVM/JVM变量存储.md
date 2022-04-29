#JVM #Memory-structure 

# 简介
1. 寄存器：最快的存储区, 由编译器根据需求进行分配,程序中无法控制.  
2. 栈 stack：方法执行时创建方法栈帧：
	1. 存放*基本类型变量数据和对象的引用*；
	2. 对象本身不存放在栈中，而是存放在堆（new 出来的对象）或者常量池中（字符串常量对象存放在常量池中。）  
3. 堆 *Heap*：所有 *new 对象*。  
4. 静态域（*Method Area*）：存放*静态成员（static）*  
5. 常量池（*Method Area*）：存放*字符串常量和基本类型常量（public static final）*。  
6. 非RAM存储：硬盘等永久存储空间


# 参考
1. [JVM 变量存储位置](https://www.cnblogs.com/sw008/p/11054352.html)