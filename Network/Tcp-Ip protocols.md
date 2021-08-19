## Tcp/Ip protocol layers
Tcp/Ip是一个网络协议簇，包含了从**底层链路层（physical layer+data link layer）、网络路由层（network layer）、数据传输层（transport layer）到上层应用（application layer）** 的一系列协议。
标准的TCP协议模型分为如上的四层，不同层次可能的协议如下：

![[网络协议分层.png]]

1. 物理层 + 数据链路层 data link：internet 基础硬件，传输网络比特流信号。链路层：PPP、Wifi、Ethernet；
2. 网络层 network：IP、ARP；
3. 传输层 transport：ICMP、TCP、UDP；
4. 应用层 application：[[HTTP Protocol]]、FTP、SMTP、PING、DNS
