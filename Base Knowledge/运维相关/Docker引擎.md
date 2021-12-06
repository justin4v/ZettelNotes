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
对 LXC 的依赖自始至终都是个问题。  
  
1. LXC 是基于 Linux 的。这对于一个立志于跨平台的项目来说是个问题。  
  
其次，如此核心的组件依赖于外部工具，这会给项目带来巨大风险，甚至影响其发展。  
  
因此，Docker 公司开发了名为 Libcontainer 的自研工具，用于替代 LXC。  
  
Libcontainer 的目标是成为与平台无关的工具，可基于不同内核为 Docker 上层提供必要的容器交互功能。  
  
在 Docker 0.9 版本中，Libcontainer 取代 LXC 成为默认的执行驱动。