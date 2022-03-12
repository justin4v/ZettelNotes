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
- 一般只能*顺序读写*；
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

## 组合流
- 将*流整合*以便实现更高级的输入和输出操作。
- 比如，一次读取一个字节是很慢的，所以可以从磁盘中一次读取块数据到缓冲中，然后从缓冲中获取字节。
- 为了实现缓冲，把 *InputStream 包装到 BufferedInputStream 中*。
```java
InputStream input = new BufferedInputStream(new FileInputStream("c:\data\input-file.txt"));
```

整合Reader与InputStream

一个Reader可以和一个InputStream相结合。如果你有一个InputStream输入流，并且想从其中读取字符，可以把这个InputStream包装到InputStreamReader中。把InputStream传递到InputStreamReader的构造函数中：

```
Reader reader = new InputStreamReader(inputStream);
```

在构造函数中可以指定解码方式。
# 参考
1. [Java IO 模型之 BIO，NIO，AIO](https://cloud.tencent.com/developer/article/1825524)
2. [[IO模型]]