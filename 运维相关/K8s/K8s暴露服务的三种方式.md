#k8s #Todo 


- pod ip是 docker 网桥的IP地址段进行分配的，是一个虚拟的二层网络，外部网络并没有办法访问;
- pod ip是随时会变的，不是固定的；
- k8s 引入了 Service 的概念，通过Service管理这些pod，Service创建后的Service IP是固定的。
- Service IP(Cluster IP) 是一个虚拟的 IP,由Kubernetes管理和分配，外部网络无法访问。

k8s有三种方式暴露Service给外部网络访问。

# NodePort
- 将服务的类型设置成NodePort-每个集群节点都会在节点上打 开 一个端口， 对于NodePort服务， 每个集群节点在节点本身（因此得名叫NodePort)上打开一个端口，并将在该端口上接收到的流量重定向到基础服务。
- 该服务仅在内部集群 IP 和端口上才可访间；
- Kubernetes在其所有节点上开放一  个端口给外部访问（所有节点上都使用相同的端口号）， 并将传入的连接转发给作为Service服务对象的  pod。

![[k8s nodeport 方式暴露服务.png]]

k8s 中 service 配置：
![[k8s nodeport service yaml配置.png]]

## 特点
- 简单；
- 服务多时，NodePort 在每个节点上开启的端口会很多，而且难以维护。

# LoadBalane
- 将服务的类型设置成LoadBalance, NodePort类型的一 种扩展，这使得服务可以通过一个专用的负载均衡器来访问， 这是由Kubernetes中正在运行的云基础设施提供的。 
- 负载均衡器将流量重定向到跨所有节点的节点端口。客户端通过负载均衡器的 IP 连接到服务。

![[loadbalance 方式暴露服务.png]]

k8s service 配置：

![[loadbalance service配置.png]]
## 特点
- 用 LoadBalancer 暴露的服务都会有它自己的 IP 地址和端口，不能做到一个ip地址就可以访问所有服务。


# Ingress
创建一 个Ingress资源， 这是一 个完全不同的机制， 通过一 个IP地址公开多
个服务,就是一个网关入口，和springcloud的网关zuul、gateway类似。


# 参考
1. [k8s-(七）暴露服务的三种方式](https://blog.csdn.net/qq_21187515/article/details/112363072)