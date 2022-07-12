#Linux #Linux-capabilities
- `Capabilities` 是在 Linux 内核 `2.2` 之后引入的；
- 原理是将之前与超级用户 root（UID=0）关联的特权细分为不同的功能组；
- Capabilites 作为线程（**Linux 并不真正区分进程和线程**）的属性存在，每个功能组都可以独立启用和禁用。
- 本质上就是将内核调用分门别类，具有相似功能的内核调用被分到同一组中。
- Capabilities 可以在进程执行时赋予，也可以直接从父进程继承。

# 种类
带有 Capabilities 的权限检查的过程：
1. 执行特权操作时，如果线程的有效身份不是 root；
2. 检查其是否具有该特权操作所对应的 capabilities，并以此为依据，决定是否可以执行特权操作。

种类：
| capability  名称     | 描述                                                            |
| -------------------- | --------------------------------------------------------------- |
| CAP_AUDIT_CONTROL    | 启用和禁用内核审计；改变审计过滤规则；检索审计状态和过滤规则    |
| CAP_AUDIT_READ       | 允许通过 multicast netlink 套接字读取审计日志                   |
| CAP_AUDIT_WRITE      | 将记录写入内核审计日志                                          |
| CAP_BLOCK_SUSPEND    | 使用可以阻止系统挂起的特性                                      |
| CAP_CHOWN            | 修改文件所有者的权限                                            |
| CAP_DAC_OVERRIDE     | 忽略文件的 DAC 访问限制                                         |
| CAP_DAC_READ_SEARCH  | 忽略文件读及目录搜索的 DAC 访问限制                             |
| CAP_FOWNER           | 忽略文件属主 ID 必须和进程用户 ID 相匹配的限制                  |
| CAP_FSETID           | 允许设置文件的 setuid 位                                        |
| CAP_IPC_LOCK         | 允许锁定共享内存片段                                            |
| CAP_IPC_OWNER        | 忽略 IPC 所有权检查                                             |
| CAP_KILL             | 允许对不属于自己的进程发送信号                                  |
| CAP_LEASE            | 允许修改文件锁的 FL_LEASE 标志                                  |
| CAP_LINUX_IMMUTABLE  | 允许修改文件的 IMMUTABLE 和 APPEND 属性标志                     |
| CAP_MAC_ADMIN        | 允许 MAC 配置或状态更改                                         |
| CAP_MAC_OVERRIDE     | 忽略文件的 DAC 访问限制                                         |
| CAP_MKNOD            | 允许使用 mknod() 系统调用                                       |
| CAP_NET_ADMIN        | 允许执行网络管理任务                                            |
| CAP_NET_BIND_SERVICE | 允许绑定到小于 1024 的端口                                      |
| CAP_NET_BROADCAST    | 允许网络广播和多播访问                                          |
| CAP_NET_RAW          | 允许使用原始套接字                                              |
| CAP_SETGID           | 允许改变进程的 GID                                              |
| CAP_SETFCAP          | 允许为文件设置任意的 capabilities                               |
| CAP_SETPCAP          | 参考 capabilities man page                                      |
| CAP_SETUID           | 允许改变进程的 UID                                              |
| CAP_SYS_ADMIN        | 允许执行系统管理任务，如加载或卸载文件系统、设置磁盘配额等      |
| CAP_SYS_BOOT         | 允许重新启动系统                                                |
| CAP_SYS_CHROOT       | 允许使用 chroot() 系统调用                                      |
| CAP_SYS_MODULE       | 允许插入和删除内核模块                                          |
| CAP_SYS_NICE         | 允许提升优先级及设置其他进程的优先级                            |
| CAP_SYS_PACCT        | 允许执行进程的 BSD 式审计                                       |
| CAP_SYS_PTRACE       | 允许跟踪任何进程                                                |
| CAP_SYS_RAWIO        | 允许直接访问 /devport、/dev/mem、/dev/kmem 及原始块设备         |
| CAP_SYS_RESOURCE     | 忽略资源限制                                                    |
| CAP_SYS_TIME         | 允许改变系统时钟                                                |
| CAP_SYS_TTY_CONFIG   | 允许配置 TTY 设备                                               |
| CAP_SYSLOG           | 允许使用 syslog() 系统调用                                      |
| CAP_WAKE_ALARM       | 允许触发一些能唤醒系统的东西(比如 CLOCK_BOOTTIME_ALARM  计时器) |

