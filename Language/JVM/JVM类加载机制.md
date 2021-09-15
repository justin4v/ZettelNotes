## JVM架构
JVM架构图如下：

![[JVM架构图.png]]

## ClassLoader SubSystem
**KEYS**
- Loading：加载 .class JMV（结构信息=>Metaspace、class对象=>Heap）; 
- Linking：准备 class对象（校验class，变量赋予初始值，符号引用解析=>静态链接）；
- Initialization：初始化（）

