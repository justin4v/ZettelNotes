#IO-model #Java基础 

# IO概念
## 输入输出
- 输入输出是对于当前程序而言；
- 输入：从数据源的读取
- 输出：输出数据到目标媒介。
- 典型的数据源/目标媒介：
	1. 文件
	2. 管道
	3. 网络连接
	4. 内存/缓存
	5. System.in, System.out, System.error

## 流 Stream
- 一个连续的数据流。
- 可以从流中读取数据，也可以往流中写数据。
- 一般只能顺序读写；
- Java IO 中流既可以是*字节流*(以字节为单位进行读写)，也可以是*字符流*(以字符为单位进行读写)

## Stream/reader 和程序关系
![[IO中stream、reader和程序关系.png]]
- *InputStream（字节流）和Reader（字符流）* 与*数据源*相关联；
- *OutputStream（字节流）和writer（字符流）* 与*目标媒介*相关联

# Java IO
## IO类别
- 包含了许多 InputStream、OutputStream、Reader、Writer 的子类。
- 原因是让每一个类都负责不同的功能。
- 各类用途如下：
	1. 文件访问
	2. 网络访问
	3. 内存缓存访问
	4. 线程内部通信(管道)
	5. 缓冲
	6. 过滤
	7. 解析
	8. 读写文本 (Readers / Writers)
	9. 读写基本类型数据 (long, int etc.)
	10. 读写对象



# 参考
1. [Java IO 模型之 BIO，NIO，AIO](https://cloud.tencent.com/developer/article/1825524)
2. [[IO模型]]