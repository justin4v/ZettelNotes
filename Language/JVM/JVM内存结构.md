## 内存结构
### JVM机构图
![[JVM架构图.png]]

《java 虚拟机规范》中正式叫法是** Runtime Data Area** （运行时数据区），内存结构是为了方便的称呼。

### 分类
- **Method Area** 和 **Heap Area** 是线程共享的。
- **Stack Area** 、**PC Registers** 和 **Native Method Area**是每个线程独有的。
