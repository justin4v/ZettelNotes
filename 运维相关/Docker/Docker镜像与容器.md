# Image
- Docker image 由**多层文件系统叠加而成**，类似于**栈结构**；
- 类似于一个精简的 OS，对外表现为独立的一个系统（参考[[#Union Mount|联合加载]]）；
- 最底层是引导文件系统（bootfs）；
- 第二层是 root 文件系统（rootfs）；
- 传统 Linux 引导过程中，rootfs 先以只读的方式加载，引导结束后，rootfs切换为读写模式；
- Docker 中 rootfs 等都是只读。
- Docker 利用联合加载（union mount，一次加载多个文件系统）一次同时加载所需要的文件系统，都是只读的，永远不会变化。

1. Docker 称这样的文件系统为 Image 镜像；
2. 从Image启动容器时，在 Image 的最顶层加载一个读写文件系统；
3. Docker 中运行的程序就是在顶层的读写层中执行的。

![[Docker文件系统分层示意.png]]

![[Docker文件分层示意2.png]]
# 特点
## Image-Layering Framework
- Docker 从镜像创建容器时，Docker 会构建出一个镜像栈（image stack）
- 在栈的最顶层添加一个读写层


## Copy-on-Write
- Docker 启动容器时，读写层是空的。文件系统发生的变化都会应用到这一层。
- 如需要修改一个文件：
	- 从下方的只读层复制该文件到读写层；
	- 对文件进行修改；
	- 下方的只读版本被读写层的修改版本覆盖。

## Union Mount
- 联合挂载技术可以在一个挂载点同时挂载多个文件系统，将挂载点的原目录与被挂载内容进行整合；
- 联合挂载文件系统最终*对外表现为一层*。


# Container
- Docker容器(Container) 是独立运行的一个或一组应用。
- Docker容器(Container) 是从[[#Image|镜像]] 创建的运行实例，它可以被启动、开始、停止、 删除。
- 每个 Docker容器(Container) 都是相互隔离的、保证安全的平台。
- Image 类似于 Container的模板。