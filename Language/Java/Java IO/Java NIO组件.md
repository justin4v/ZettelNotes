#Java基础 #NIO #Channel #Selector


# 和传统IO对比
传统IO（简称IO）基于字节流和字符流进行操作，而NIO基于Channel和Buffer(缓冲区)进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector(选择区)用于监听多个通道的事件（比如：连接打开，数据到达）。因此，单个线程可以监听多个数据通道。

- **IO 是面向流的，NIO 是面向缓冲区的**；
- 面向流：
	1. Stream 是单向、只读的；
	2. 不能前后移动流中的数据；
	3. Socket 连接传输速度，取决于短板：如果客户端处理慢，在服务端也只能等待；服务端发送慢，客户端同样只能等待。
- NIO 的缓冲导向方法略有不同。数据读取到一个它稍后处理的缓冲区，需要时可在缓冲区中前后移动。这就增加了处理过程中的灵活性。但是，还需要检查是否该缓冲区中包含所有您需要处理的数据。而且，需确保当更多的数据读入缓冲区时，不要覆盖缓冲区里尚未处理的数据


# Channel
Channel和IO中的Stream(流)是差不多一个等级的。只不过Stream是单向的，譬如：InputStream, OutputStream.而Channel是双向的，既可以用来进行读操作，又可以用来进行写操作。  
NIO中的Channel的主要实现有：

-   FileChannel
-   DatagramChannel
-   SocketChannel
-   ServerSocketChannel

分别可以对应文件IO、UDP和TCP（Server和Client）


# Buffer

NIO中的关键Buffer实现有：ByteBuffer, CharBuffer, DoubleBuffer, FloatBuffer, IntBuffer, LongBuffer, ShortBuffer，分别对应基本数据类型: byte, char, double, float, int, long, short。当然NIO中还有MappedByteBuffer, HeapByteBuffer, DirectByteBuffer等


# Selector

Selector运行单线程处理多个Channel，如果你的应用打开了多个通道，但每个连接的流量都很低，使用Selector就会很方便。例如在一个聊天服务器中。要使用Selector, 得向Selector注册Channel，然后调用它的select()方法。这个方法会一直阻塞到某个注册的通道有事件就绪。一旦这个方法返回，线程就可以处理这些事件，事件的例子有如新的连接进来、数据接收等。

selector中会有一些SelectionKey,SelectionKey中有一些表示操作状态的OP Status,根据这个OP Status的不同，selectionKey可以有四种状态，分别是isReadable,isWritable,isConnectable和isAcceptable。

当SelectionKey处于isAcceptable状态的时候，表示ServerSocketChannel可以接受连接了；
需要调用 register 方法将 *serverSocketChannel accept 生成的 socketChannel 注册到selector中*，以监听它的OP READ状态，后续可以从中读取数据。




# 参考
1. [Java NIO？看这一篇就够了！](https://blog.csdn.net/u011381576/article/details/79876754)