#Socket #IO-model 
# 交互

![[Socket服务端与客户端交互示意.png]]


## ServerSocket
- 创建一个 *ServerSocket 对象*，会在运行该语句的计算机*指定端口处建立一个监听服务*，如：
```java
ServerSocket server=new ServerSocket(600)；
```
- 一台计算机可以*同时提供多个服务(多个端口)*，通过端口号来区别。为了监听可能的 Client 请求，执行如下的语句：
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
- Socket类的构造函数有两个参数，第一个参数是欲连接到的Server计算机的主机地址，第二个参数是该Server机上提供服务的端口号。
Socket对象建立成功之后，就可以在Client和Server之间建立一个连接，并通过这个连接在两个端点之间传递数据。利用Socket类的方法getOutputStream()和getInputStream()分别获得向Socket读写数据的输入／输出流，最后将从Server端读取的数据重新返还到Server端。
当Server和Client端的通信结束时，可以调用Socket类的close()方法关闭Socket，拆除连接。

ServerSocket 一般仅用于设置端口号和监听，真正进行通信的是服务器端的Socket与客户端的Socket，在ServerSocket 进行accept之后，就将主动权转让了。