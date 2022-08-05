#Socket #IO-model #ServerSocket
# 交互

![[Socket服务端与客户端交互示意.png]]


## ServerSocket
- 创建一个 *ServerSocket 对象*，会在运行该语句的计算机*指定端口处建立一个监听服务*，如：
```java
ServerSocket server=new ServerSocket(600)；
```
- 计算机可*同时提供多个服务(多个端口)*，通过端口号来区别。
- 为了监听可能的 Client 请求，执行如下的语句：
```java
Socket client=server.accept()；
```
- accept() 方法将使 Server 端处于**阻塞状态**，直到*接收到一个来自 Client 的连接请求*；
- 并返回一个用于与该 Client 通信的 Socket 对象 client。
- 此后 Server 程序*只要向 Socket 对象读写数据*，就可实现向远端的 Client 读写数据。
- 结束监听时，关闭 ServerSocket 对象：
```java
server.close()；
```
## Socket
- 当 Client 需要从 Server 端获取信息及其他服务时，创建一个 Socket 对象：
```java
 Socket MySocket=new Socket(“ServerComputerName”，600)；
```
- 有两个参数，Server 的 IP 和 Server 的端口号。
- ServerSocket 和 socket 连接建立成功之后，可通过这个连接在两个端点之间传递数据。
- Socket 的 getOutputStream() 和 getInputStream() 分别获得向Socket读写数据的输入／输出流。
- 当通信结束时，可以调用 close() 方法关闭 Socket，拆除连接。

## 注意
- ServerSocket 一般仅用于设置端口号和监听，真正进行通信的是服务器端的Socket与客户端的Socket；
- 在 ServerSocket 进行 accept 之后，就将主动权转让了。