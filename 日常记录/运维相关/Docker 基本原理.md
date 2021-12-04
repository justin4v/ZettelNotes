#Containerization #运维 
# 对 [[Linux cgroups]] 的使用
1. 默认情况下，Docker 启动一个容器后，会在 /sys/fs/cgroup 目录下的各个资源目录下生成以容器 ID 为名字的目录（group）.
	比如：
	```
	/sys/fs/cgroup/cpu/docker/03dd196f415276375f754d51ce29b418b170bd92d88c5e420d6901c32f93dc14
	```
cpu.cfs_quota_us 的内容为 -1，表示默认情况下并没有限制容器的 CPU 使用。
2. 在容器被 stopped 后，该目录被删除。


运行命令 
`docker run -d --name web41 --cpu-quota 25000 --cpu-period 100 --cpu-shares 30 training/webapp python app.py`
启动一个新的容器，结果：
```shell
root@devstack:/sys/fs/cgroup/cpu/docker/06bd180cd340f8288c18e8f0e01ade66d066058dd053ef46161eb682ab69ec24# cat cpu.cfs_quota_us 
25000 
root@devstack:/sys/fs/cgroup/cpu/docker/06bd180cd340f8288c18e8f0e01ade66d066058dd053ef46161eb682ab69ec24# cat tasks 
3704 
root@devstack:/sys/fs/cgroup/cpu/docker/06bd180cd340f8288c18e8f0e01ade66d066058dd053ef46161eb682ab69ec24# cat cpu.cfs_period_us 
2000
```
- Docker 会将容器中的进程的 ID 加入到各个资源对应的 tasks 文件中。
- 表明 Docker 也是使用 cgroups 对容器的 CPU 使用进行限制。