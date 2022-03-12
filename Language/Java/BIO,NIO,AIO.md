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


# 参考
1. [Java IO 模型之 BIO，NIO，AIO](https://cloud.tencent.com/developer/article/1825524)