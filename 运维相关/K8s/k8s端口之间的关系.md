#k8s #Todo 

# nodePort
- nodePort 提供了**集群外部客户端访问 Service 的一种方式**；
- 通过 `nodeIP:nodePort` 提供了外部访问k8s集群中service的入口。
- 如外部用户要访问k8s集群中的一个Web应用，可以配置对应 service 的`type=NodePort`，`nodePort=30001`。其他用户就可以通过浏览器`http://node:30001`访问到该web服务。
- 而数据库等服务可能不需要被外界访问，只需被内部服务访问即可，那么我们就不必设置service的NodePort。

# port

port是暴露在cluster ip上的端口，:port提供了集群内部客户端访问service的入口，即`clusterIP:port`。

mysql容器暴露了3306端口（参考[DockerFile](https://github.com/docker-library/mysql/)），集群内其他容器通过33306端口访问mysql服务，但是外部流量不能访问mysql服务，因为mysql服务没有配置NodePort。对应的service.yaml如下：

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

targetPort是pod上的端口，从port/nodePort上来的数据，经过kube-proxy流入到后端pod的targetPort上，最后进入容器。

与制作容器时暴露的端口一致（使用DockerFile中的EXPOSE），例如官方的nginx（参考[DockerFile](https://github.com/nginxinc/docker-nginx)）暴露80端口。 对应的service.yaml如下：

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

containerPort是在pod控制器中定义的、pod中的容器需要暴露的端口。

例如，mysql 服务需要暴露 3306 端口，redis 暴露 6379 端口

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


# 参考
1. [k8s 辨析 port、NodePort、targetPort、containerPort 区别](https://www.cnblogs.com/veeupup/p/13545361.html)
2. 