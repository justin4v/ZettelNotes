#运维 #Containerization 

Certain software applications in the wild may* need to be controlled or limited*—at least for the sake of stability and, to some degree, security.  
**Control groups (cgroups)** is a *kernel feature* that **limits**, **accounts** for and **isolates** the *CPU, memory, disk I/O and network's usage* of one or more processes.

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
# 参考
1. [Everything You Need to Know about Linux Containers, Part I: Linux Control Groups and Process Isolation](https://www.linuxjournal.com/content/everything-you-need-know-about-linux-containers-part-i-linux-control-groups-and-process)