# 示例
```java
class Test {
	public int aaa() {
		int x = 1;

		try {
			return ++x;
		} catch (Exception e) {

		} finally {
			++x;
		}
		return x;
	}

	public static void main(String[] args) {
		Test t = new Test();
		int y = t.aaa();
		System.out.println(y);
	}
}

// 结果
2
```
- 问题： 为什么是2 而不是3

# 解释
1. 官方对finally 说明：
>The `finally` block _always_ executes when the `try` block exits. This ensures that the `finally` block is executed even if an unexpected exception occurs. But `finally` is useful for more than just exception handling — it allows the programmer to avoid having cleanup code accidentally bypassed by a `return`, `continue`, or `break`. Putting cleanup code in a `finally` block is always a good practice, even when no exceptions are anticipated.

2. 另外，在[java的语言规范](http://docs.oracle.com/javase/specs/jls/se7/html/jls-14.html#jls-14.17)中，如果在 try 语句里有 return 语句，finally 语句还是会执行。它会在把控制权转移到该方法的调用者或者构造器前执行finally语句。也就是说，使用return语句把控制权转移给其他的方法前会执行finally语句。
3. [官方的jvm规范](http://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.10.2.5)解释如下：
> If the try clause executes a return, the compiled code does the following:
> 
> 1.  Saves the return value (if any) in a local variable.
> 2.  Executes a jsr to the code for the finally clause.
> 3.  Upon return from the finally clause, returns the value saved in the local variable.

- 当执行到 return ++x;时，jvm在执行完++x后会在局部变量表里另外分配一个空间来保存当前x的值。  
- 注意，现在还没把值返回给y，而是继续执行finally语句里的语句。
- 等执行完后再把之前保存的值（是2不是x）返回给y。  
- 还有一点要注意:
1. 如果在 finally 里也用了return语句，比如return +xx。y是3。
3. 因为规范规定了，当try和finally里都有return时，会忽略try的return，而使用finally的return。


# 字节码层面解释
![[finally和return示例字节码.png]]

指令操作顺序：  
1. *iconst_1*： 把常数1进栈 ；
2. *istore_1*： 栈顶元素出栈，并把元素保存在本地变量表的第二个位置里（下标为1的位置里） 
3. *iinc 1, 1* ： 本地变量表的第二个元素自增1 
4. *iload_1*：第二个元素进栈 
5. *istore_2*：栈顶元素出栈并把元素保存在本地变量表的第2个位置里 
6. *iinc 1, 1* ： 本地变量表的第二个元素自增1 
7. *iload_2*：第三个元素进栈 （注意，此时栈顶元素为2）
8. *ireturn*：返回栈顶元素。
9. 后面的指令是要在 2-7 行出现异常时在跳到12行的，本例没出现异常。

上面流程栈和本地变量表的情况如下图：

![[finally和return字节码操作流程示意.png]]
参见 ![[Hotspot 字节码指令集#字节码指令集概述]]


# 总结
- 在 try块中即便有return，break，continue等改变执行流的语句，finally也会执行。
> - finally中的 return 会**覆盖 try 或者catch中的 return**
> - finally中的 return 会**抑制（消灭）前面try或者catch块中抛出的异常**
> - finally中的异常会**覆盖（消灭）前面try或者catch中抛出的异常**


-   **不要在fianlly中使用return**。
-   **不要在finally中抛出异常**。
-   不要在finally中做一些其它的事情，finally 块**仅仅用来释放资源**最合适。
-   将尽量将所有的 return 写在函数的最后面，而不是 try ... catch ... finally 中。

# 参考
1. [Java中的异常和处理详解](https://www.cnblogs.com/lulipro/p/7504267.html)
2. [你真的了解try{ return }finally{}中的return？](https://www.cnblogs.com/averey/p/4379646.html)
3. [Chapter 4. The class File Format (oracle.com)](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.10.2.5)