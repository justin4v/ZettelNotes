# Docker 引擎
- Docker 引擎是用来运行和管理容器的核心软件。
- 通常会简单地将其代指为 Docker 或 [[Docker基础#docker 平台构成|Docker 平台]]。

# 原始 Docker 引擎
Docker 引擎由如下主要的组件构成：
1. Docker 客户端（Docker Client）；
2. Docker 守护进程（Docker daemon）；
3. containerd 
4. runc

![[原始docker引擎逻辑示意.png]]


Docker 首次发布时，Docker 引擎由两个核心组件构成：**LXC 和 Docker daemon**。  
- *Docker daemon* 是单一的二进制文件，包含诸如 *Docker 客户端、Docker API、容器运行时、镜像构建*等。  
- *LXC* 提供了对诸如*命名空间（Namespace）和控制组（CGroup）* 等基础工具的操作能力，是基于 Linux 内核的容器虚拟化技术。

Docker 初始版本中，Docker daemon、LXC 和操作系统之间的交互关系：
![[原始docker引擎中LXC和docker daemon关系.png]]

## 参考
1. [[Linux Capabilities]]
2. [[Linux Namespace]]
3. [[Linux cgroups]]

# 演化
## 摆脱 LXC
Docker 引擎对 LXC 的依赖是个问题：  
1. *LXC 基于 Linux*。对于跨平台的项目来说是个问题。
2. 核心的组件依赖于外部工具，会带来风险，甚至影响其发展。  

Docker 公司开发了 Libcontainer 用于替代 LXC：
1. Libcontainer 的目标是成为与平台无关的工具，可基于不同内核为 Docker 上层提供必要的容器交互功能。  
2. Docker 0.9 中，Libcontainer 取代 LXC 成为默认的执行驱动。

## 摒弃大而全的 Docker daemon
- 随着时间的推移，Docker daemon 变得难于变更、运行越来越慢
- 需要拆解 Docker daemon 进程，并将其模块化。  
- 目标是尽可能拆解出其中的功能特性，并用*小而专的工具来实现它*。
- 目前所有*容器执行和容器运行时*的代码已经完全从 daemon 中移除，并重构为小而专的工具

![[Docker引擎新的架构示意.png]]


## runc
- runc 是 OCI 容器运行时规范的参考实现；
- runc 实质上是一个轻量级的、针对 Libcontainer 进行了包装的命令行交互工具；
- runc 实质是一个独立的容器运行时工具。

## containerd
- Docker daemon 的功能进行拆解后，容器执行逻辑被重构到名为 containerd的工具中；
- 主要任务：
	- 容器管理——start | stop | pause | rm 等
	- 镜像管理


# 容器的创建过程
常用的启动容器的方法是使用 Docker 命令行工具。
下面的`docker container run`命令基于 `alpine:latest` [[Docker镜像与容器#Image|镜像]]启动一个新容器。  
```shell
$ docker container run --name ctr1 -it alpine:latest sh
```

1. 命令行工具执行命令时，Docker client 将其转换为合适的 API 格式，并发送到 Docker Host 中的 daemon；
2. daemon 接收到创建新容器的命令，向 containerd 发出调用；
	daemon 使用一种 CRUD 风格的 API，通过 gRPC 与 containerd 进行通信
3. containerd 将 Docker 镜像转换为 OCI bundle，并调用 runc 创建一个新的容器；
4. runc 与操作系统内核接口进行通信，基于所有必要的工具（Namespace、CGroup等）来创建容器。
	容器进程作为 runc 的子进程启动，启动完毕后，runc 将会退出。


![[Pasted image 20211206140149.png]]

