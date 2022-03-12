#IO-model #Java基础 
# BIO（Blocking IO）
- BIO 是同步阻塞模型，一个客户端连接对应一个处理线程。
- 缺点：
	1. BIO 代码里的 accept() 和 read() 方法是阻塞方法，如果没有客户端连接或者连接不做数据读写操作会导致线程阻塞，浪费资源。
	2. 如果线程很多，会导致服务器线程太多，压力太大，比如 C10K 问题。
- 应用场景：
	- 适用于*连接数比较小且固定的架构*;
	- 对*服务器资源要求比较高*，但程序简单易理解。
```java
/**
 * 同步阻塞
 * BIO 同步阻塞模型，一个客户端连接对应一个处理线程
 * @author 程治玮
 * @since 2021/3/19 9:34 下午
 */
public class SocketServer {
    public static void main(String[] args) throws IOException {

        //服务器监听9000端口
        ServerSocket serverSocket = new ServerSocket(9000);

        while (true) {
            System.out.println("等待连接...");
            //接受客户端请求，阻塞方法，没有客户端连接时就会阻塞
            Socket clientSocket = serverSocket.accept();
            System.out.println("有客户端连接了...");

            // 单线程一次只能接收一个客户端的连接请求
            handler(clientSocket); 
            
            //启动多线程，这样可以接收多个客户端的请求
//            new Thread(new Runnable() {
//                @Override
//                public void run() {
//                    try {
//                        handler(clientSocket);
//                    } catch (IOException e) {
//                        e.printStackTrace();
//                    }
//                }
//            });
        }

    }

    private static void handler(Socket clientSocket) throws IOException {
        byte[] bytes = new byte[1024];
        System.out.println("准备读取数据...");
        //接收客户端的数据，阻塞方法，没有数据可读时就阻塞
        int read = clientSocket.getInputStream().read(bytes);
        System.out.println("读取数据完毕...");
        if (read != -1) {
            System.out.println("接收到客户端的数据：" + new String(bytes, 0, read));
        }

        //服务器向客户端发送数据
        clientSocket.getOutputStream().write("HelloClient".getBytes());
        clientSocket.getOutputStream().flush();
    }
}
```

# NIO（Non Blocking IO）
- 核心组件：*Channel(通道)*， *Buffer(缓冲区)*，*Selector(多路复用器)*
-  **Selector**：Selector 允许*单线程处理多个 Channel*。
	- 应用需要向 Selector 注册 Channel；
	- 然后调用 select() 方法，*select() 会阻塞到某个注册的 channel 有事件*就绪。
	- 一旦这个方法返回，线程就可以处理这些事件。
-  **Channel**：Channel 类似于 Stream，数据可以从 Channel 读到 Buffer，也可以从Buffer 写到 Channel。
-  **Buffer**：本质上是一个*可以读写数据的内存块*，能够跟踪和记录缓冲区的状态变换情况，Channel 提供从文件，网络读取数据的渠道，但是*读取写入的数据都经由 Buffer* 。
- 应用场景：
	- 适用于连接数目多且连接比较短（轻操作）的架构；
	- 比如聊天服务器，弹幕系统，服务器间通讯。

## NIO 不使用 Selector
- NIO 同步非阻塞，服务器*一个线程可以监听/处理多个连接*;
- 通过*轮询*的方式遍历所有的连接。
- 但是如果连接数太多的话，可能有*大量的无效遍历*。假如有 10000 个连接，其中只有 1000 个连接有写数据，90% 遍历都是无效的。

```java
/**
 * 同步非阻塞
 * NIO 服务端，每次遍历轮询所有的客户端连接去读取数据
 * @author 程治玮
 * @since 2021/3/21 12:36 下午
 */
public class NioSever {

    // 保存客户端连接
    static List<SocketChannel> channelList = new ArrayList<>();

    public static void main(String[] args) throws IOException, InterruptedException {

        //创建NIO ServerSocketChannel,与 BIO 的 serverSocket 类似
        //创建一个在本地端口进行监听的服务 Socket 通道，并设置为非阻塞方式
        ServerSocketChannel serverSocket = ServerSocketChannel.open();
        serverSocket.socket().bind(new InetSocketAddress(9000));  //监听 9000 端口
        //设置 ServerSocketChannel 为非阻塞
        serverSocket.configureBlocking(false); //false 非阻塞，配置成 true ，就成 BIO 了
        System.out.println("服务启动成功");

        while (true) {
            // 非阻塞模式 accept() 方法不会阻塞
            // NIO 的非阻塞是由操作系统内部实现的，底层调用了 linux 内核的 accept 函数
            SocketChannel socketChannel = serverSocket.accept();
            if (socketChannel != null) { // 如果有客户端进行连接
                System.out.println("连接成功");
                // 设置 SocketChannel 为非阻塞
                socketChannel.configureBlocking(false); //false 非阻塞，配置成 true ，就成 BIO 了
                // 保存客户端连接在 List 中
                channelList.add(socketChannel);
            }
            // 遍历客户端连接 SocketChannel 进行数据读取
            Iterator<SocketChannel> iterator = channelList.iterator();
            while (iterator.hasNext()) {
                SocketChannel sc = iterator.next();
                ByteBuffer byteBuffer = ByteBuffer.allocate(128);
                // 非阻塞模式 read() 方法不会阻塞
                int len = sc.read(byteBuffer);
                // 如果有数据，把数据打印出来
                if (len > 0) {
                    System.out.println("接收到消息：" + new String(byteBuffer.array()));
                } else if (len == -1) { // 如果客户端断开，把 socket 从集合中去掉
                    iterator.remove();
                    System.out.println("客户端断开连接");
                }
            }
        }
    }
}
```

## NIO 使用 Selector
- NIO 底层在 JDK1.4 版本是用 linux 的内核函数 *select()* 或 *poll()* 来实现;
- Selector 每次都会轮询所有的 SockChannel ;
- JDK1.5开始引入了 *epoll* 基于事件响应机制来优化 NIO。
```java
/**
 * 同步非阻塞
 * NIO 引入多路复用器 Selector，只对有事件的 serverSocket （本例是客户端连接或者客户端发送数据）进行处理
 * @author 程治玮
 * @since 2021/3/21 12:49 下午
 */
public class NioSelectorServer {

    public static void main(String[] args) throws IOException, InterruptedException {

        //创建NIO ServerSocketChannel
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        serverSocketChannel.socket().bind(new InetSocketAddress(9000)); //监听 9000 端口
        //设置 ServerSocketChannel 为非阻塞
        //必须配置为非阻塞才能往 Selector 上注册，否则会报错，Selector 本身就是非阻塞模式
        serverSocketChannel.configureBlocking(false);  //false 非阻塞
        // 打开 Selector 处理 Channel ，即创建 epoll
        Selector selector = Selector.open();
        // 把 ServerSocketChannel 注册到 Selector 上，并且 Selector 对客户端 accept 连接操作感兴趣
        SelectionKey selectionKey = serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
        System.out.println("服务启动成功");

        while (true) {
            //阻塞等待需要处理的事件发生
            //轮询监听 channel 里的 key，select()是阻塞的，当有客户端连接事件发生时 OP_ACCEPT
            //或者是读取客户端传的数据 OP_READ，才会停止阻塞
            selector.select();

            // 获取 selector 中注册的全部事件的 SelectionKey 实例
            Set<SelectionKey> selectionKeys = selector.selectedKeys();
            Iterator<SelectionKey> iterator = selectionKeys.iterator();

            // 遍历 SelectionKey 对事件进行处理
            while (iterator.hasNext()) {
                SelectionKey key = iterator.next();
                // 如果是OP_ACCEPT事件，则进行连接获取和事件注册
                if (key.isAcceptable()) {
                    //通过 selector 注册事件的 Key 获取到对应的客户端连接的 ServerSocket
                    ServerSocketChannel serverSocket = (ServerSocketChannel) key.channel();
                    SocketChannel socketChannel = serverSocket.accept(); //连接客户端
                    socketChannel.configureBlocking(false);  //false 非阻塞

                    // 把客户端连接的 socketChannel 注册到 Selector 上，对读操作感兴趣
                    // 这里只注册了读事件，如果需要给客户端发送数据可以注册写事件
                    socketChannel.register(selector, SelectionKey.OP_READ);
                    System.out.println("客户端连接成功");
                } else if (key.isReadable()) {  // 如果是OP_READ事件，则进行读取和打印
                    SocketChannel socketChannel = (SocketChannel) key.channel();
                    ByteBuffer byteBuffer = ByteBuffer.allocate(128);
                    int len = socketChannel.read(byteBuffer);
                    // 如果有数据，把数据打印出来
                    if (len > 0) {
                        System.out.println("接收到消息：" + new String(byteBuffer.array()));
                    } else if (len == -1) { // 如果客户端断开连接，关闭Socket
                        System.out.println("客户端断开连接");
                        socketChannel.close();
                    }
                }
                //从事件集合里删除本次处理的key，防止下次select重复处理
                iterator.remove();
            }
        }
    }
}
```
- 把需要探知的 SocketChannel 注册到 Selector，调用 select() 方法阻塞；
- 当有事件发生时，Selector 传回一组 SelectionKey（linux 内核中的 rdlist 就绪事件列表）；
- 根据 Key 获得注册过的 SocketChannel，并从 Channel 中读取并处理这些数据。
- Selector 实际对所注册的 Channel（SocketChannel）不断地轮询访问，一旦轮询到所注册的事情发生，返回 channel 的 可以。

![[Java NIO示意.png]]

### Epoll函数详解
Hotspot 与 Linux 内核函数级别理解如下方法：
```java
Selector.open()  //创建多路复用器
socketChannel.register(selector, SelectionKey.OP_READ)  //将channel注册到多路复用器上
selector.select()  //阻塞等待需要处理的事件发生
```

-  Selector.open() 会*调用 Linux 内核函数 epoll_create* 创建 epoll 实例（Selector 对象）。
-  *socketChannel.register() 将 SocketChannel 添加到一个内部集合*中（pollWrapper.add(fd)）。
-  执行到 selector.select()；
	1. 先调用 updateRegistrations 方法从而调用 Linux 内核函数 epoll_ctl 将 SocketChannel 进行注册绑定事件；
	2. 当 Socket 收到数据后（网卡接收到数据包），调用 Linux 内核中的中断处理程序调用回调函数往 epoll 实例的事件就绪列表 rdlist（SelectionKey） 里添加该 Socket 的引用。
	3. 然后会调用 Linux 内核函数 epoll_wait，如果 rdlist 有 Socket 的引用，那么 epoll_wait 直接返回，程序继续完成后面的处理；
	4. 如果 rdlist 为空，则阻塞进程。


# AIO(NIO 2.0)
- 异步非阻塞， 由操作系统完成后回调通知服务端程序启动线程去处理，；
- 一般适用于*连接数较多且连接时间较长*的应用；
- 应用场景：
- AIO方式适用于连接数目多且连接比较长(重操作)的架构，JDK7 开始支持。

```java
/**
 * AIO 服务端程序，异步非阻塞
 * @author 程治玮
 * @since 2021/3/21 2:20 下午
 */
public class AioServer {

    public static void main(String[] args) throws Exception {
        final AsynchronousServerSocketChannel serverChannel =
                AsynchronousServerSocketChannel.open().bind(new InetSocketAddress(9000));

        serverChannel.accept(null, new CompletionHandler<AsynchronousSocketChannel, Object>() {
            @Override
            public void completed(AsynchronousSocketChannel socketChannel, Object attachment) {
                try {
                    System.out.println("2--" + Thread.currentThread().getName());
                    // 再此接收客户端连接，如果不写这行代码后面的客户端连接连不上服务端
                    serverChannel.accept(attachment, this);
                    System.out.println(socketChannel.getRemoteAddress());
                    ByteBuffer buffer = ByteBuffer.allocate(1024);
                    socketChannel.read(buffer, buffer, new CompletionHandler<Integer, ByteBuffer>() {
                        @Override
                        public void completed(Integer result, ByteBuffer buffer) {
                            System.out.println("3--" + Thread.currentThread().getName());
                            buffer.flip();
                            System.out.println(new String(buffer.array(), 0, result));
                            socketChannel.write(ByteBuffer.wrap("HelloClient".getBytes()));
                        }

                        @Override
                        public void failed(Throwable exc, ByteBuffer buffer) {
                            exc.printStackTrace();
                        }
                    });
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }

            @Override
            public void failed(Throwable exc, Object attachment) {
                exc.printStackTrace();
            }
        });

        System.out.println("1--" + Thread.currentThread().getName());
        Thread.sleep(Integer.MAX_VALUE);
    }
}
```

## 为什么Netty使用 NIO 而不是 AIO ？

- 在 Linux 系统上，AIO 的底层实现仍使用 Epoll，因此在性能上没有明显的优势;
- AIO被 JDK 封装了一层不容易深度优化，Linux 上 AIO 还不够成熟。Netty 是异步非阻塞框架，Netty 在 NIO 上做了很多异步的封装。
# 参考
1. [Java IO 模型之 BIO，NIO，AIO](https://cloud.tencent.com/developer/article/1825524)
2. [[IO模型]]