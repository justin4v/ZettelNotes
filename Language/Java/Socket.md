#Socket #IO-model 
# 交互

![[Socket服务端与客户端交互示意.png]]


## ServerSocket
创建一个ServerSocket 对象，会在运行该语句的计算机的指定端口处建立一个监听服务，如：
```java
ServerSocket MyListener=new ServerSocket(600)；
```
这里指定提供监听服务的端口是600，一台计算机可以同时提供多个服务，这些不同的服务之间通过端口号来区别，不同的端口号上提供不同的服务。为了随时监听可能的Client请求，执行如下的语句：
```java
Socket LinkSocket=MyListener．accept()；
```
该语句调用了ServerSocket对象的accept()方法，这个方法的执行将使Server端的程序处于等待状态，程序将一直阻塞直到捕捉到一个来自Client端的请求，并返回一个用于与该Client通信的Socket对象Link-Socket。此后Server程序只要向这个Socket对象读写数据，就可以实现向远端的Client读写数据。结束监听时，关闭ServerSocket对象：
```java
Mylistener．close()；
```
## Socket
当Client程序需要从Server端获取信息及其他服务时，应创建一个Socket对象：
```java
 Socket MySocket=new Socket(“ServerComputerName”，600)；
```
Socket类的构造函数有两个参数，第一个参数是欲连接到的Server计算机的主机地址，第二个参数是该Server机上提供服务的端口号。
Socket对象建立成功之后，就可以在Client和Server之间建立一个连接，并通过这个连接在两个端点之间传递数据。利用Socket类的方法getOutputStream()和getInputStream()分别获得向Socket读写数据的输入／输出流，最后将从Server端读取的数据重新返还到Server端。
当Server和Client端的通信结束时，可以调用Socket类的close()方法关闭Socket，拆除连接。

ServerSocket 一般仅用于设置端口号和监听，真正进行通信的是服务器端的Socket与客户端的Socket，在ServerSocket 进行accept之后，就将主动权转让了。