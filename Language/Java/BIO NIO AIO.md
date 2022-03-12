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
- 可以从流中读取数据，也可以往流中写数据
- Java IO 中流既可以是*字节流*(以字节为单位进行读写)，也可以是*字符流*(以字符为单位进行读写)

## Stream/reader 和程序关系
![[IO中stream、reader和程序关系.png]]
# 参考
1. [Java IO 模型之 BIO，NIO，AIO](https://cloud.tencent.com/developer/article/1825524)
2. [[IO模型]]