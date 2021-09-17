# JVM架构
JVM架构图如下：

![[JVM架构图.png]]

# ClassLoader SubSystem
**conception**
- **Loading**：加载 .class JMV（结构信息=>Metaspace、class对象=>Heap）; 
- **Linking**：准备 class对象（校验class，变量赋予初始值，符号引用解析=>静态链接）；
- **Initialization**：初始化（初始化变量，动态链接）。

类加载过程：
![[类加载过程.png]]

## Loading
1. 通过类的**全限定名**获取类的二进制字节流；
2.  加载类的**静态结构到元空间**(Metaspace)；
3. 在 Heap 中创建**java.lang.class 对象** 并指向 Metaspace 中的 class 结构。


## Linking
### Verification
1.**Target**
 - **保证 Class 字节流符合虚拟机规范**

参考：[[Class 文件结构]]

2.**DES**
1. **文件格式验证**：验证字节流是否符合Class文件格式的规范。需以**魔数`0xCAFEBABE`开头（身份标识）**、**主次版本号**是否在当前虚拟机的处理范围之内、常量池中的常量是否有不被支持的类型。
2. **元数据验证**：对字节码描述的信息进行语义分析（注意：对比javac编译阶段的语义分析），以保证其描述的信息符合Java语言规范的要求；例如：这个类是否有父类，除了 `java.lang.Object`之外。
3. **字节码验证**：通过数据流和控制流分析，确定程序语义是合法的、符合逻辑的。
4. **符号引用验证**：确保解析动作能正确执行。

### Preparation
1.**Target**
 - 在 **Metaspace 中为类变量默认初始化**

2.**Attention**
-   Preparation进行内存分配的**仅包括类变量（static），分配在 Metaspace**，而不包括实例变量。实例变量会在对象实例化时随着对象一块分配在 Java 堆中；
-   默认初始值是指**初始化为数据类型默认的零值**（如 `0`、`0L`、`null`、`false` 等），而不是在 Java 代码中被显式地赋予的值；
-   类字段**同时被 final 和 static 修饰（不允许修改，类似于常量，必须初始化时赋予值）**，称为`ConstantValue`属性，Preparation 阶段变量就会被初始化为指定的值。

### Resolution
1.**Target**
-  **符号引用替换成直接引用**

2.**DES**
符号引用是指**一组指明需要引用对象的符号**，是一个唯一标识，但不是实际内存地址，如静态方法、类变量等（类加载过程中只能确定属于类的属性）。 
直接引用是指**能确定数据在内存中的位置**的引用，如直接指针、相对偏移量、句柄

在类加载过程中完成的解析称为**静态链接**。
在运行期间完成的解析称为**动态链接**。

## Initialization
1.**Target**
 - **为类变量赋予正确的值**；
 - 真正开始执行类定义的（如static代码块）Java代码。

2.**DES**



