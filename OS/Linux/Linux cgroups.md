#运维 #Containerization #Linux #Linux-cgroup 

Certain software applications in the wild may* need to be controlled or limited*—at least for the sake of stability and, to some degree, security.  
- **Control groups (cgroups)** is a *kernel feature* that **limits**, **accounts** for and **isolates** the *CPU, memory, disk I/O and network's usage* of one or more processes.
- 用于**资源限制**；


## History
- *Originally developed by Google*；
- First land in Linux kernel in version 2.6.24 (January 2008)；
- Redesign would be merged into both the 3.15 and 3.16 kernels.

The primary design **goal**:
- provide a unified interface to **manage processes-level or whole operating-system-level virtualization**, including *Linux Containers(LXC)*。

## Feature
The cgroups framework provides the following features:
-   **Resource limiting:** a group can be configured *not to exceed a specified memory limit* or use more than the desired amount of processors or be limited to specific peripheral devices.
-   **Prioritization:** one or more groups may be configured to utilize fewer or more CPUs or disk I/O throughput.
-   **Accounting:** a group's resource *usage is monitored and measured*.
-   **Control:** groups of processes *can be frozen or stopped and restarted*.

## Other Feature
1. A cgroup can consist of one or more processes that are all bound to the same set of limits. 
2. Groups also can be hierarchical, which means that a subgroup inherits the limits administered to its parent group.

The Linux kernel provides access to a series of controllers or subsystems for the cgroup technology. T
- The `memory` controller is what limits memory usage；
- The `cpuacct` controller monitors CPU usage.

# 术语

-   **任务（Tasks）**：就是系统的一个**进程**。
-   **控制组（Control Group）**：按照**某种标准划分的一组进程**。
	-   比如官方文档中的 Professor 和Student，或是WWW和System之类的，其表示了某进程组。
	-   cgroups 中的*资源控制单位是控制组*。
	-   一个进程可以加入到某个控制组。
-   **层级（Hierarchy）**：控制组可以组织成 **hierarchical 的形式**，一棵控制组的树（目录结构）。
	-   控制组树上的子节点继承父结点的属性。
	-   ![[cgroup层级示意图.png]]
-   **子系统（Subsystem）**：一个子系统就是一个**资源控制器**。
	-   比如 CPU 子系统就是控制 CPU 时间分配的一个控制器。
	-   子系统必须附加到一个层级上才能起作用；
	-   一个子系统附加到某个层级以后，层级上的所有控制族群都受到这个子系统的控制。

# 子系统
-   `cpuset` - assigns individual processor(s) and memory nodes to task(s) in a group;
-   `cpu` - uses the scheduler to provide cgroup tasks access to the processor resources;
-   `cpuacct` - generates reports about processor usage by a group;
-   `io` - sets limit to read/write from/to [block devices](https://en.wikipedia.org/wiki/Device_file);
-   `memory` - sets limit on memory usage by a task(s) from a group;
-   `devices` - allows access to devices by a task(s) from a group;
-   `freezer` - allows to suspend/resume for a task(s) from a group;
-   `net_cls` - allows to mark network packets from task(s) from a group;
-   `net_prio` - provides a way to dynamically set the priority of network traffic per network interface for a group;
-   `perf_event` - provides access to [perf events](https://en.wikipedia.org/wiki/Perf_/(Linux/)) to a group;
-   `hugetlb` - activates support for [huge pages](https://www.kernel.org/doc/Documentation/vm/hugetlbpage.txt) for a group;
-   `pid` - sets limit to number of processes in a group.

Linux 系统中，一切皆文件。*Linux 将 cgroups 实现成了文件系统*。
```shell
[root@node7 ~]# mount -t cgroup  # 挂在到 linux 文件树上
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,xattr,release_agent=/usr/lib/systemd/systemd-cgroups-agent,name=systemd)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,hugetlb)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,memory)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,net_prio,net_cls)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,freezer)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,cpuacct,cpu)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,perf_event)
cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,pids)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,cpuset)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,blkio)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,devices)

[root@node7 ~]# lssubsys -m
cpuset /sys/fs/cgroup/cpuset   # cgroup 的父级目录，可在在其中新建目录（子group），会自动生成配置文件
cpu,cpuacct /sys/fs/cgroup/cpu,cpuacct
memory /sys/fs/cgroup/memory
devices /sys/fs/cgroup/devices
freezer /sys/fs/cgroup/freezer
net_cls,net_prio /sys/fs/cgroup/net_cls,net_prio
blkio /sys/fs/cgroup/blkio
perf_event /sys/fs/cgroup/perf_event
hugetlb /sys/fs/cgroup/hugetlb
pids /sys/fs/cgroup/pids

[root@node7 ~]# ll /sys/fs/cgroup/cpu,cpuacct
total 0
-rw-r--r--. 1 root root 0 Nov  6 14:05 cgroup.clone_children
--w--w--w-. 1 root root 0 Nov  6 14:05 cgroup.event_control
-rw-r--r--. 1 root root 0 Nov  6 14:05 cgroup.procs
-r--r--r--. 1 root root 0 Nov  6 14:05 cgroup.sane_behavior
-r--r--r--. 1 root root 0 Nov  6 14:05 cpuacct.stat
-rw-r--r--. 1 root root 0 Nov  6 14:05 cpuacct.usage
-r--r--r--. 1 root root 0 Nov  6 14:05 cpuacct.usage_percpu
-rw-r--r--. 1 root root 0 Nov  6 14:05 cpu.cfs_period_us
-rw-r--r--. 1 root root 0 Nov  6 14:05 cpu.cfs_quota_us
-rw-r--r--. 1 root root 0 Nov  6 14:05 cpu.rt_period_us
-rw-r--r--. 1 root root 0 Nov  6 14:05 cpu.rt_runtime_us
-rw-r--r--. 1 root root 0 Nov  6 14:05 cpu.shares
-r--r--r--. 1 root root 0 Nov  6 14:05 cpu.stat
-rw-r--r--. 1 root root 0 Nov  6 14:05 notify_on_release
-rw-r--r--. 1 root root 0 Nov  6 14:05 release_agent
-rw-r--r--. 1 root root 0 Nov  6 14:05 tasks  # 将进程PID写入task文件，即可使得进程受控
```


# 进程和cgroup的关系
- 限制一个进程的内存和CPU，会绑定进程到 CPU cgroup 和 Memory cgroup 的节点上；
- Cpu cgroup 节点和 Memory cgroup节点属于两个不同的Hierarchy 层级。
- 进程和 cgroup 节点是*多对多的关系*，因为一个进程涉及多个子系统，每个子系统可能属于不同的层次结构(Hierarchy)。

示意图：
![[进程和cgroup关系示意图.png]]

- P 代表进程；
- 多个进程可能共享相同的资源，所以会抽象出一个 `CSS_SET`, 每个进程会属于一个CSS_SET 链表
- 同一个 `CSS_SET` 下的进程都被其管理；
- 一个 `CSS_SET` 关联多个 cgroup节点，`CSS_SET`和 `Cgroup`节点就是多对多的关系(MxN 的关系)。

## CSS_SET 
```C
#ifdef CONFIG_CGROUPS  
	 /* Control Group info protected by css_set_lock */  
	 struct css_set __rcu *cgroups; 关联的cgroup 节点  
	 /* cg_list protected by css_set_lXock and tsk->alloc_lock */  
	 struct list_head cg_list; // 关联所有的进程  
#endif
```

# 参考
1. [Everything You Need to Know about Linux Containers, Part I: Linux Control Groups and Process Isolation](https://www.linuxjournal.com/content/everything-you-need-know-about-linux-containers-part-i-linux-control-groups-and-process)
2. [Linux Control Groups V1 和 V2 原理和区别](https://mikechengwei.github.io/2020/06/03/cgroup%E5%8E%9F%E7%90%86/)
3. [理解Docker（4）：Docker 容器使用 cgroups 限制资源使用](https://www.cnblogs.com/sammyliu/p/5886833.html)
4. [Docker基础技术：Linux CGroup ](https://coolshell.cn/articles/17049.html)