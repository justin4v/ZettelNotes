#Linux
# 目的
- Linux 提供的一种内核级别环境隔离的方法；
- 将特定的全局系统资源（global system resource）进行抽象，使 namespace 中的进程认为拥有独立的系统资源实例。

# 种类
| 分类               | 系统调用参数  | 相关内核版本                           |
| ------------------ | ------------- | -------------------------------------- |
| Mount namespaces   | CLONE_NEWNS   | Linux 2.4.19                           |
| UTS namespaces     | CLONE_NEWUTS  | Linux 2.6.19                           |
| IPC namespaces     | CLONE_NEWIPC  | Linux 2.6.19                           |
| PID namespaces     | CLONE_NEWPID  | Linux 2.6.24                           |
| Network namespaces | CLONE_NEWNET  | 始于 Linux 2.6.24 完成于  Linux 2.6.29 |
| User namespaces    | CLONE_NEWUSER | 始于 Linux 2.6.23 完成于  Linux 3.8    |

# 使用
- namespace 的 API 由三个系统调用和一系列 `/proc` 文件组成；
- 系统调用的 flag 中通过常量 `CLONE_NEW*`等指定，多个常量可以通过 **|**（位或）操作实现。

## API
-   **clone()** : 实现线程的系统调用，用来创建一个新的进程，并可以通过设计上述系统调用参数达到隔离的目的。
-   **unshare()** : 使某进程脱离某个 namespace。
-   **setns()** : 把某进程加入到某个 namespace。

## proc 文件
- 每个进程都有一个 `/proc/PID/ns` 目录，其下面的文件表示 namespace, 例如 user 就表示 user namespace。
- 从 3.8 版本的内核开始，namespace 的每个文件都是一个特殊的符号链接，链接指向 `$namespace:[$namespace-inode-number]`：
	- 前半部份为 namespace 的名称，
	- 后半部份的数字表示这个 namespace 的句柄号。

```shell
$ ls -l /proc/$$/ns  # $$ 表示当前所在 shell PID
total 0
lrwxrwxrwx. 1 mtk mtk 0 Jan  8 04:12 ipc -> ipc:[4026531839]
lrwxrwxrwx. 1 mtk mtk 0 Jan  8 04:12 mnt -> mnt:[4026531840]
lrwxrwxrwx. 1 mtk mtk 0 Jan  8 04:12 net -> net:[4026531956]
lrwxrwxrwx. 1 mtk mtk 0 Jan  8 04:12 pid -> pid:[4026531836]
lrwxrwxrwx. 1 mtk mtk 0 Jan  8 04:12 user -> user:[4026531837]
lrwxrwxrwx. 1 mtk mtk 0 Jan  8 04:12 uts -> uts:[4026531838]
```

符号链接的用途之一是用来**确认两个不同的进程是否处于同一 namespace 中**。
1. 如果两个进程指向的 namespace inode number 相同，就在同一个 namespace 下；
2. 否则就在不同的 namespace 下。

# API 详解
## clone
`clone()` 的原型如下：
```c
int clone(int (*child_func)(void *), void *child_stack, int flags, void *arg);
```
-   **child_func** : 传入子进程运行的程序主函数。
-   **child_stack** : 传入子进程使用的栈空间。
-   **flags** : 表示使用哪些 `CLONE_*` 标志位。
-   **args** : 用于传入用户参数。

特点
1. `clone()` 与 `fork()` 类似，都相当于把当前进程复制了一份；
2. `clone()` 可以更细粒度地控制与子进程共享的资源（其实就是通过 flags 来控制），包括虚拟内存、打开的文件描述符和信号量等等。
3. 如果指定了标志位 `CLONE_NEW*`，相对应类型的 namespace 就会被创建，新创建的进程也会成为该 namespace 中的一员。

### 注意
- 除了 User namespace 之外，创建其他的 namespace 都需要特权，确切地说，是需要相应的 `Linux Capabilities`

# 参考
1. [Linux Namespace 入门：Namespace API](https://cloud.tencent.com/developer/article/1583922)
2. [Linux Capabilities 入门：让普通进程获得 root 的洪荒之力]