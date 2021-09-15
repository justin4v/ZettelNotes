## JVM架构
JVM架构图如下：

![[JVM架构图.png]]

## ClassLoader SubSystem
**KEYS**
- Loading：加载 .class JMV（结构信息=>Metaspace、class对象=>Heap）; 
- Linking：准备 class对象（校验class，变量赋予初始值，符号引用解析=>静态链接）；
- Initialization：初始化（初始化变量，动态链接）。



### 解析
**conception**
- **符号引用替换成直接引用**
符号引用是指用一组符号指明需要引用的对象，如静态方法、类变量等。 
在类加载过程中进行的解析称为静态解析。



