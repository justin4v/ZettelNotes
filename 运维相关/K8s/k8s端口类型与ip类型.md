#k8s 

# nodePort
- nodePort 提供了**集群外部客户端访问 Service 的一种方式**；
- 通过 `nodeIP:nodePort` 提供了外部访问k8s集群中service的入口。
- 如外部用户要访问k8s集群中的一个Web应用，可以配置对应 service 的`type=NodePort`，`nodePort=30001`。其他用户就可以通过浏览器`http://node:30001`访问到该web服务。
- 而数据库等服务可能不需要被外界访问，只需被内部服务访问即可，那么我们就不必设置service的NodePort。

# port
- port 是暴露在 cluster ip上的端口，提供了**集群内部客户端访问service的入口**，即 `clusterIP:port`。
- mysql 容器暴露了 3306 端口（参考[DockerFile](https://github.com/docker-library/mysql/)），集群内其他容器通过33306端口访问mysql服务；
- 外部流量不能访问mysql服务，因为mysql服务没有配置NodePort。
- 对应的service.yaml如下：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  ports:
  - port: 33306
    targetPort: 3306
  selector:
    name: mysql-pod
```

# targetPort
- targetPort 是 **pod 上的端口**；
- 从 port/nodePort 上来的数据，**经过 kube-proxy 流入到后端 pod 的targetPort上**，最后进入容器。
- 与制作容器时暴露的端口一致（使用DockerFile中的EXPOSE），例如官方的 nginx（参考[DockerFile](https://github.com/nginxinc/docker-nginx)）暴露80端口。
- 对应的service.yaml如下：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort            // 配置NodePort，外部流量可访问k8s中的服务
  ports:
  - port: 30080             // 服务访问端口，集群内部访问的端口
    targetPort: 80          // pod控制器中定义的端口（应用访问的端口）
    nodePort: 30001         // NodePort，外部客户端访问的端口
  selector:
    name: nginx-pod
```

# containerPort
- containerPort是在 pod **控制器中定义的、其中容器需要暴露的端口**。
- 例如，mysql 服务需要暴露 3306 端口，redis 暴露 6379 端口

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: redis-master
  labels: 
    name: redis-master
spec:
  replicas: 1
  selector:
    name: redis-master
  template:
    metadata:
      labels:
        name: redis-master
    spec:
      containers:
      - name: master
        image: kubeguide/redis-master
        ports:
        - containerPort: 6379	# 此处定义暴露的端口
```


# IP类型
## Node IP
1. Node节点的IP地址，即物理网卡的IP地址。  
2. 每个Service都会在Node节点上开通一个端口，外部可以通过NodeIP:NodePort即可访问Service里的Pod

查看 nodeIp
```shell
[root@k8s-master ~]# kubectl get nodes -o wide
NAME         STATUS   ROLES                  AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                                 KERNEL-VERSION         CONTAINER-RUNTIME
k8s-master   Ready    control-plane,master   21h   v1.23.3   172.20.10.88   <none>        Alibaba Cloud Linux 3 (Soaring Falcon)   5.10.60-9.al8.x86_64   docker://20.10.12

[root@k8s-master ~]# kubectl describe node k8s-master | grep Addresses -C 5
  NetworkUnavailable   False   Wed, 16 Feb 2022 17:57:07 +0800   Wed, 16 Feb 2022 17:57:07 +0800   CalicoIsUp                   Calico is running on this node
  MemoryPressure       False   Thu, 17 Feb 2022 14:59:18 +0800   Wed, 16 Feb 2022 17:56:36 +0800   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure         False   Thu, 17 Feb 2022 14:59:18 +0800   Wed, 16 Feb 2022 17:56:36 +0800   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure          False   Thu, 17 Feb 2022 14:59:18 +0800   Wed, 16 Feb 2022 17:56:36 +0800   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready                True    Thu, 17 Feb 2022 14:59:18 +0800   Wed, 16 Feb 2022 17:57:12 +0800   KubeletReady                 kubelet is posting ready status
Addresses:
  InternalIP:  172.20.10.88
  Hostname:    k8s-master
Capacity:
  cpu:                4
  ephemeral-storage:  103080204Ki
```


## Pod IP
- Pod的IP地址，即docker容器的IP地址，此为虚拟IP地址。  
- Pod IP是每个Pod的IP地址，他是Docker Engine根据docker网桥的IP地址段进行分配的，通常是一个虚拟的二层网络
	1.   同Service下的pod可以直接根据PodIP相互通信
	2.   不同Service下的pod在集群间pod通信要借助于 cluster ip
	3.   pod和集群外通信，要借助于node ip

查询 pod Ip
```shell
[root@k8s-master ~]# kubectl get pods -n kube-system -o wide
NAME                                       READY   STATUS    RESTARTS   AGE   IP               NODE         NOMINATED NODE   READINESS GATES
calico-kube-controllers-566dc76669-tp7fz   1/1     Running   0          21h   10.244.235.194   k8s-master   <none>           <none>
calico-node-z2v7l                          1/1     Running   0          21h   172.20.10.88     k8s-master   <none>           <none>
coredns-6d8c4cb4d-q9w46                    1/1     Running   0          21h   10.244.235.195   k8s-master   <none>           <none>
etcd-k8s-master                            1/1     Running   0          21h   172.20.10.88     k8s-master   <none>           <none>
kube-apiserver-k8s-master                  1/1     Running   0          21h   172.20.10.88     k8s-master   <none>           <none>
kube-controller-manager-k8s-master         1/1     Running   0          21h   172.20.10.88     k8s-master   <none>           <none>
kube-proxy-kh7z9                           1/1     Running   0          21h   172.20.10.88     k8s-master   <none>           <none>
kube-scheduler-k8s-master                  1/1     Running   0          21h   172.20.10.88     k8s-master   <none>           <none>

[root@k8s-master ~]# kubectl describe pod -n kube-system coredns-6d8c4cb4d-q9w46 | grep IP
                      cni.projectcalico.org/podIP: 10.244.235.195/32
                      cni.projectcalico.org/podIPs: 10.244.235.195/32
IP:                   10.244.235.195
IPs:
  IP:           10.244.235.195
```

## Cluster IP
Service的IP地址，此为虚拟IP地址。外部网络无法ping通，只有kubernetes集群内部访问使用。

查看 clusterIp
```shell
[root@k8s-master ~]# kubectl get svc -A
NAMESPACE     NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  21h
kube-system   kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   21h
```

Cluster IP是一个虚拟的IP，但更像是一个伪造的IP网络，原因有以下几点:
1.  Cluster IP 仅仅作用于 Kubernetes Service这个对象，并由Kubernetes管理和分配IP地址
2.  Cluster IP 无法被ping，他没有一个“实体网络对象”来响应
3.  Cluster IP 只能结合Service Port组成一个具体的通信端口，单独的Cluster IP不具备通信的基础，并且他们属于Kubernetes集群这样一个封闭的空间。
4.  在不同 Service 下的 pod 节点在集群间相互访问可以通过Cluster IP


# 参考
1. [k8s 辨析 port、NodePort、targetPort、containerPort 区别](https://www.cnblogs.com/veeupup/p/13545361.html)
2. [k8s-集群里的三种IP（NodeIP、PodIP、ClusterIP）](https://www.cnblogs.com/xjzyy/p/15904750.html)