# 简介
Docker 是一个：
- 开源的；
- 轻量级的容器引擎；
- 主要运行于 Linux 和 Windows，用于创建、管理和编排容器。

和 VMware 虚拟机相比，Docker 使用容器（[[Docker镜像与容器#Container|Container]]）承载应用程序，而不使用操作系统，所以它的开销很少，性能很高。
但是，Docker 对应用程序的隔离不如虚拟机彻底，所以它并不能完全取代 VMware。

# Container 状态机

![[docker容器状态机.png]]

一个容器在某个时刻可能处于以下几种状态之一：

-   **created**：已经被创建 （使用 docker ps -a 命令可以列出）但是还没有被启动 （使用 docker ps 命令还无法列出）
-   **running**：运行中
-   **paused**：容器的进程被暂停了
-   **restarting**：容器的进程正在重启过程中
-   **exited**：图中的 stopped 状态，表示容器之前运行过但是现在处于停止状态（要区别于 created 状态，它是指一个新创出的尚未运行过的容器）。可以通过 start 命令使其重新进入 running 状态
-   **destroyed**：容器被删除了，再也不存在了

# 基础命令
```
镜像操作
    build     Build an image from a Dockerfile
    commit    Create a new image from a container's changes
    images    List images
    load      Load an image from a tar archive or STDIN
    pull      Pull an image or a repository from a registry
    push      Push an image or a repository to a registry
    rmi       Remove one or more images
    search    Search the Docker Hub for images
    tag       Tag an image into a repository
    save      Save one or more images to a tar archive (streamed to STDOUT by default)
    history   显示某镜像的历史    inspect   获取镜像的详细信息
    容器及其中应用的生命周期操作：
    create    Create a new container （创建一个容器）        
    kill      Kill one or more running containers
    inspect   Return low-level information on a container, image or task
    pause     Pause all processes within one or more containers
    ps        List containers
    rm        Remove one or more containers （删除一个或者多个容器）
    rename    Rename a container
    restart   Restart a container
    run       Run a command in a new container （创建并启动一个容器）
    start     Start one or more stopped containers （启动一个处于停止状态的容器）
    stats     Display a live stream of container(s) resource usage statistics （显示容器实时的资源消耗信息）
    stop      Stop one or more running containers （停止一个处于运行状态的容器）
    top       Display the running processes of a container
    unpause   Unpause all processes within one or more containers
    update    Update configuration of one or more containers
    wait      Block until a container stops, then print its exit code
    attach    Attach to a running container
    exec      Run a command in a running container
    port      List port mappings or a specific mapping for the container    logs      获取容器的日志    
    
容器文件系统操作
    cp        Copy files/folders between a container and the local filesystem
    diff      Inspect changes on a container's filesystem
    export    Export a container's filesystem as a tar archive
    import    Import the contents from a tarball to create a filesystem image
    
    Docker registry 操作：
    login     Log in to a Docker registry.
    logout    Log out from a Docker registry.
    
    Volume 操作
    volume    Manage Docker volumes    

网络操作
    network   Manage Docker networks
    
Swarm 相关操作
    swarm     Manage Docker Swarm
    service   Manage Docker services
    node      Manage Docker Swarm nodes       

系统操作：    
    version   Show the Docker version information     
	events    Get real time events from the server  (持续返回docker 事件)    
	info      Display system-wide information （显示Docker 主机系统范围内的信息）
```

# docker 平台构成

![[docker 平台构成.png]]

1.  客户端：用户使用 Docker 提供的工具（CLI 以及 API 等）来构建，上传镜像并发布命令来创建和启动容器
2.  Docker 主机：从 Docker registry 上下载镜像并启动容器
3.  Docker registry：Docker 镜像仓库，用于保存镜像，并提供镜像上传和下载