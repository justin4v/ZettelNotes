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
- NIO 同步非阻塞，服务器一个线程可以监听/处理多个请求（连接）;
- 通过*轮询*的方式遍历所有的连接。
- 但是如果连接数太多的话，可能有大量的无效遍历，假如有 10000 个连接，其中只有 1000 个连接有写数据，其中有十分之九十的遍历都是无效的。

# 参考
1. [Java IO 模型之 BIO，NIO，AIO](https://cloud.tencent.com/developer/article/1825524)
2. [[IO模型]]