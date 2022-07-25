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
- Node IP：Node节点的IP地址，即物理网卡的IP地址。  
- Pod IP：Pod的IP地址，即docker容器的IP地址，此为虚拟IP地址。  
- Cluster IP：Service的IP地址，此为虚拟IP地址。

# 参考
1. [k8s 辨析 port、NodePort、targetPort、containerPort 区别](https://www.cnblogs.com/veeupup/p/13545361.html)
2. 