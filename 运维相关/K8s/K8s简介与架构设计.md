#k8s #Docker

# 简介
- *Kubernetes* 简称 *k8s*；
- *容器集群管理系统*，开源平台；
- 可实现*容器集群的自动化部署、自动扩缩容、维护*等功能。

# 能做什么
- 可在物理或虚拟机上的 Kubernetes 集群*运行容器化应用*；
- Kubernetes 能提供一个以“**容器为中心的基础架构**”；
- 能满足在生产环境中运行应用的一些常见需求，如：
	-   多个进程（作为容器运行）协同工作。（Pod）
	-   存储系统挂载
	-   Distributing secrets
	-   应用健康检测
	-   应用实例的复制
	-   Pod自动伸缩/扩展
	-   Naming and discovering
	-   负载均衡
	-   滚动更新
	-   资源监控
	-   日志访问
	-   调试应用程序
	-   提供认证和授权


# 设计
## 架构

![[k8s 架构设计.png]]



# 参考
1. [《Kubernetes与云原生应用》系列之Kubernetes的系统架构与设计理念](https://www.infoq.cn/article/kubernetes-and-cloud-native-applications-part01/)
2. [Kubernetes（k8s）中文文档 Kubernetes设计架构](https://www.kubernetes.org.cn/kubernetes%e8%ae%be%e8%ae%a1%e6%9e%b6%e6%9e%84)