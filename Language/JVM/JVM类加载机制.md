## JVM架构
JVM架构图如下：

![[JVM架构图.png]]

## ClassLoader SubSystem
**KEYS**
- Loading：加载 .class JMV（结构信息=>Metaspace、class对象=>Heap）; 
- Linking：准备 class对象（校验class，变量赋予初始值，符号引用解析=>静态链接）；
- Initialization：初始化（初始化变量，动态链接）。

类加载过程：
![[类加载过程.png]]

### Loading
1. 通过类的全限定名获取类的二进制字节流；
2.  加载类的静态存储结构 到 Metaspace；
3. 在 Heap 中创建**java.lang.class 对象** 并指向 Metaspace 中的 class 结构。

### 解析
**conception**
- **符号引用替换成直接引用**
符号引用是指**一组指明需要引用对象的符号**，是一个唯一标识，但不是实际内存地址，如静态方法、类变量等（类加载过程中只能确定属于类的属性）。 
直接引用是指**能确定数据在内存中的位置**的引用，如直接指针、相对偏移量、句柄

在类加载过程中完成的解析称为**静态链接**。
在运行期间完成的解析称为**动态链接**。



