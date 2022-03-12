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

### 整合 Reader 与 InputStream

- Reader 可以和 InputStream 相结合。
- 如果你有一个 InputStream 输入流，想*从其中读取字符*，可以把 InputStream 包装到 InputStreamReader 中：

```java
Reader reader = new InputStreamReader(inputStream);
```
- 构造函数中*可以指定解码方式*。

## IO管道
- Java IO 中的管道为*运行在同一个JVM中的两个线程*提供了通信的能力。
- 管道可以作为数据源以及目标媒介。
- 通过Java IO创建管道:
	- PipedOutputStream和PipedInputStream创建管道。
	- PipedInputStream 应该和 PipedOutputStream 连接。
	- 一个线程通过PipedOutputStream写入的数据可以被另一个线程通过 PipedInputStream 读取出来。
```java
//使用管道来完成两个线程间的数据点对点传递
    @Test
    public void test2() throws IOException {
        PipedInputStream pipedInputStream = new PipedInputStream();
        PipedOutputStream pipedOutputStream = new PipedOutputStream(pipedInputStream);
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    pipedOutputStream.write("hello input".getBytes());
                    pipedOutputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }).start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    byte []arr = new byte[128];
                    while (pipedInputStream.read(arr) != -1) {
                        System.out.println(Arrays.toString(arr));
                    }
                    pipedInputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }).start();
```

- 当使用两个相关联的管道流时，*务必分配给不同的线程*。
- read()方法和write()方法调用时会*导致流阻塞*，如果尝试在一个线程中同时进行读和写，可能会*导致线程死锁*。


# Filter和Buffered
## Filter
- **FilterInputStream** ：*封装输入流(输入流基类，InputStream)*，并*提供额外的功能*。
- 常用的子类有 BufferedInputStream 和 DataInputStream。
```java
package java.io;

public class FilterInputStream extends InputStream {
    protected volatile InputStream in;
    ...
}
```

- **BufferedInputStream**：“输入流*提供缓冲功能*，以及mark()和reset()功能”。
- **DataInputStream** ：装饰其它输入流，允许应用程序以与机器无关方式从底层输入流中*读取基本 Java 数据类型*。
## Buffered
- BufferedReader 能为字符输入流*提供缓冲区，提高 IO  效率*。
- 可以*一次读取一个磁盘块(block)的数据*，而不需要每次从网络或者磁盘中一次读取一个字节。
- 特别在访问大量磁盘数据时，缓冲通常会让IO快上许多。
- 只需把 Reader 包装到 BufferedReader 中，可以*为 Reader 添加缓冲区*( 默认缓冲区大小为 8192 字节，即8KB)。代码如下：
```java
Reader input = new BufferedReader(new FileReader("c:\\data\\input-file.txt"));
```
- 通过传递构造函数的第二个参数，*指定缓冲区大小（字节单位）*，代码如下：
```java
Reader input = new BufferedReader(new FileReader("c:\\data\\input-file.txt"), 8 * 1024);
```
- 最好把缓冲区大小设置成 *1024 的整数倍*，能更高效地利用内置缓冲区的磁盘



# 参考
1. [Java IO 模型之 BIO，NIO，AIO](https://cloud.tencent.com/developer/article/1825524)
2. [[IO模型]]