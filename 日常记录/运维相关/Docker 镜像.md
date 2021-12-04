# Image
- Docker image 由文件系统叠加而成，类似于栈结构；
- 最底层是引导文件系统（bootfs）；
- 第二层是 root 文件系统（rootfs）；
- 传统 Linux 引导过程中，rootfs 先以只读的方式加载，引导结束后，rootfs切换为读写模式；
- Docker 中 rootfs 等都是只读。
- Docker 利用联合加载（union mount，一次加载多个文件系统）一次同时加载所需要的文件系统，都是只读的。

1. Docker 称这样的文件系统为 Image 镜像；
2. 从Image启动容器时，在 Image 的最顶层加载一个读写文件系统；
3. Docker 中运行的程序就是在顶层的读写层中执行的。

