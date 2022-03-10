#Linux #select-poll-epoll #IO-model 
# select
```C
int select(int maxfdp, fd_set *readset, fd_set *writeset, fd_set *exceptset,struct timeval *timeout);
```
- maxfdp：被监听的文件描述符的总数，linux上一般限制为 1024；
- readfds、writefds、exceptset：分别指向*可读*、*可写*和*异常事件*对应的描述符集合。
- timeout: 用于设置 *select 函数的超时时间*，即告诉内核select等待多长时间之后就放弃等待。NULL 表示等待无限长的时间
- 调用后 `select` *函数会阻塞*，直到:
	1. 有 fd 就绪（数据可读、可写、或者有 `except`）;
	2. 或者超时（`timeout` 指定等待时间，如果立即返回设为 null 即可），函数返回。
- 当 `select` 函数返回后，可以通过遍历 `fdset`，来找到就绪的描述符。
- `select` 目前几乎在所有的平台上支持，其良好跨平台。
- `select` 的缺点在于*单个进程能够监视的 fd 的数量存在限制*，在 Linux 上一般为 1024
- 可以通过修改宏定义甚至重新编译内核的方式提升这一限制，但是这样也会造成效率的降低。

# poll
```C
int poll (struct pollfd *fds, unsigned int nfds, int timeout);  
```

不同与 `select` 使用三个 set 三种 `fd` 集合，`poll` 使用一个 `pollfd` 的指针实现。
```c
struct pollfd {  
 int fd;         /* file descriptor */  
 short events;   /* events to watch */  
 short revents;  /* events returned */  
};  
```

- `pollfd` 结构包含了要监视的 event 和发生的 event，不再使用 select 那样“参数-值”传递的方式
- `pollfd` 并没有最大数量限制（但是数量过大后性能也是会下降）。 
- 和 `select` 函数一样，`poll` 返回后，需要轮询 `pollfd` 来获取就绪的描述符。

# epoll

`epoll` 操作过程需要三个接口，分别如下：
```C
int epoll_create(int size)；//创建一个epoll的句柄，size用来告诉内核这个监听的数目一共有多大  
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)；  
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);  
```

## epoll_create

创建一个 `epoll` 的句柄，`size` 用来告诉内核这个监听的数目一共有多大，这个参数不同于 `select` 中的第一个参数（给出最大监听的 fd+1 的值），参数 `size` 并不是限制了 `epoll` 所能监听的描述符最大个数，只是对内核初始分配内部数据结构的一个建议。当创建好 `epoll` 句柄后，它就会占用一个 fd 值，在 Linux 下如果查看 `/proc/进程id/fd/`，是能够看到这个 fd 的，所以在使用完 `epoll` 后，必须调用 `close()` 关闭，否则可能导致 fd 被耗尽。

## epoll_ctl

函数是对指定描述符 fd 执行 op 操作。

-   `epfd`：是 `epoll_create()` 的返回值。
-   `op`：表示 op 操作，用三个宏来表示：添加 `EPOLL_CTL_ADD`，删除 `EPOLL_CTL_DEL`，修改 `EPOLL_CTL_MOD`。分别添加、删除和修改对fd的监听事件。
-   `fd`：是需要监听的 fd（文件描述符）。
-   `epoll_event`：是告诉内核需要监听什么事，`struct epoll_event` 结构如下：

```C
struct epoll_event {  
 __uint32_t events;  /* Epoll events */  
 epoll_data_t data;  /* User data variable */  
};  
  
// events可以是以下几个宏的集合  
// EPOLLIN ：表示对应的文件描述符可以读（包括对端SOCKET正常关闭）；  
// EPOLLOUT：表示对应的文件描述符可以写；  
// EPOLLPRI：表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）；  
// EPOLLERR：表示对应的文件描述符发生错误；  
// EPOLLHUP：表示对应的文件描述符被挂断；  
// EPOLLET： 将EPOLL设为边缘触发(Edge Triggered)模式，这是相对于水平触发(Level Triggered)来说的。  
// EPOLLONESHOT：只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个socket的话，需要再次把这个socket加入到EPOLL队列里  
```

`epoll()` 的事件注册函数，它不同与 `select()` 是在监听事件时告诉内核要监听什么类型的事件，而是在这里先注册要监听的事件类型。当事件发生时，会触发回调，返回给用户进程。

## epoll_wait

等待 `epfd` 上的 IO 事件，最多返回 `maxevents` 个事件。  
参数 `events` 用来从内核得到事件的集合，`maxevents` 告之内核这个 `events` 有多大，这个 `maxevents` 的值不能大于创建 `epoll_create()` 时的 size，参数 `timeout` 是超时时间（毫秒，0 会立即返回，-1 将不确定，也有说法说是永久阻塞）。该函数返回需要处理的事件数目，如返回 0 表示已超时。

## epoll 工作模式

`epoll` 对文件描述符的操作有两种模式：LT（level trigger）和 ET（edge trigger）。LT 模式是默认模式，LT 模式与 ET 模式的区别如下：

-   LT 模式：当 `epoll_wait()` 检测到描述符事件发生并将此事件通知应用程序，**应用程序可以不立即处理该事件。下次调用 `epoll_wait()` 时，会再次响应应用程序并通知此事件**。
    
    -   LT(level triggered) 是缺省的工作方式，并且同时支持 block 和 no-block socket.在这种做法中，内核告诉你一个文件描述符是否就绪了，然后你可以对这个就绪的 fd 进行 IO 操作。如果你不作任何操作，内核还是会继续通知你的。
-   ET 模式：当 `epoll_wait()` 检测到描述符事件发生并将此事件通知应用程序，**应用程序必须立即处理该事件。如果不处理，下次调用 `epoll_wait()` 时，不会再次响应应用程序并通知此事件**。
    
    -   ET(edge-triggered) 是高速工作方式，只支持 no-block socket。在这种模式下，当描述符从未就绪变为就绪时，内核通过 `epoll` 告诉你。然后它会假设你知道文件描述符已经就绪，并且不会再为那个文件描述符发送更多的就绪通知，直到你做了某些操作导致那个文件描述符不再为就绪状态了（比如，你在发送，接收或者接收请求，或者发送接收的数据少于一定量时导致了一个 EWOULDBLOCK 错误）。但是请注意，如果一直不对这个 fd 作 IO 操作（从而导致它再次变成未就绪），内核不会发送更多的通知（only once）
    -   ET 模式在很大程度上减少了 `epoll` 事件被重复触发的次数，因此效率要比 LT 模式高。`epoll` 工作在 ET 模式的时候，必须使用非阻塞套接口，以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死。


## epoll 机制

在 `select`/`poll` 中，进程只有在调用一定的方法后，内核才对所有监视的文件描述符进行扫描，而 `epoll` 事先通过 `epoll_ctl()` 来注册一个文件描述符，**一旦基于某个文件描述符就绪时，内核会采用类似 callback 的回调机制，迅速激活这个文件描述符，当进程调用 `epoll_wait()` 时便得到通知。(此处去掉了遍历文件描述符，而是通过监听回调的的机制。这正是 `epoll` 的魅力所在)**。

`epoll` 的优点主要是一下几个方面：

-   监视的描述符数量不受限制，它所支持的 fd 上限是最大可以打开文件的数目，这个数字一般远大于 2048，举个例子，在 1GB 内存的机器上大约是 10 万左右，具体数目可以 `cat /proc/sys/fs/file-max` 察看,一般来说这个数目和系统内存关系很大。`select` 的最大缺点就是进程打开的 fd 是有数量限制的。这对于连接数量比较大的服务器来说根本不能满足。虽然也可以选择多进程的解决方案（Apache 就是这样实现的），不过虽然 Linux 上面创建进程的代价比较小，但仍旧是不可忽视的，加上进程间数据同步远比不上线程间同步的高效，所以也不是一种完美的方案。
-   IO 的效率不会随着监视 fd 的数量的增长而下降。`epoll` 不同于 `select` 和 `poll` 轮询的方式，而是通过每个 fd 定义的回调函数来实现的。只有就绪的 fd 才会执行回调函数。
-   如果没有大量的 `idle-connection` 或者 `dead-connection`，`epoll` 的效率并不会比 `select/poll` 高很多，但是当遇到大量的 `idle-connection`，就会发现 `epoll` 的效率大大高于 `select`/`poll`。



# 参考
1. [[IO模型]]