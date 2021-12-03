#运维 #Containerization 

Truth be told, certain software applications in the wild may need to be controlled or limited—at least for the sake of stability and, to some degree, security. A bug or just bad code can disrupt an entire machine and potentially cripple an entire ecosystem. 
Fortunately, a way exists to keep those same applications in check. Control groups (cgroups) is a kernel feature that limits, accounts for and isolates the CPU, memory, disk I/O and network's usage of one or more processes.

## History
- Originally developed by Google；
- First land in Linux kernel in version 2.6.24 (January 2008)；
- Redesign would be merged into both the 3.15 and 3.16 kernels.

The primary design **goal**:
- provide a unified interface to **manage processes-level or whole operating-system-level virtualization**, including Linux Containers(LXC)

## Feature
The cgroups framework provides the following features:
-   **Resource limiting:** a group can be configured not to exceed a specified memory limit or use more than the desired amount of processors or be limited to specific peripheral devices.
-   **Prioritization:** one or more groups may be configured to utilize fewer or more CPUs or disk I/O throughput.
-   **Accounting:** a group's resource usage is monitored and measured.
-   **Control:** groups of processes can be frozen or stopped and restarted.

## Other Feature
1. A cgroup can consist of one or more processes that are all bound to the same set of limits. 
2. Groups also can be hierarchical, which means that a subgroup inherits the limits administered to its parent group.

The Linux kernel provides access to a series of controllers or subsystems for the cgroup technology. T
- The `memory` controller is what limits memory usage；
- The `cpuacct` controller monitors CPU usage.





# 参考
1. [Everything You Need to Know about Linux Containers, Part I: Linux Control Groups and Process Isolation](https://www.linuxjournal.com/content/everything-you-need-know-about-linux-containers-part-i-linux-control-groups-and-process)