#Nodeport #k8s #ClusterIp

# ClusterIP（默认）

- 在群集中的内部IP上公布服务， Service（服务）**只在集群内部可以访问到**

```bash
[root@master ~]# kubectl get service -n test -o wide
NAME           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE   SELECTOR
cloud-config   ClusterIP   10.96.249.98   <none>        8888/TCP   21d   k8s.kuboard.cn/layer=cloud,k8s.kuboard.cn/name=cloud-config
cloud-eureka   ClusterIP   10.96.181.80   <none>        8761/TCP   27d   k8s.kuboard.cn/layer=cloud,k8s.kuboard.cn/name=cloud-eureka
```

# NodePort

- 使用 NAT 在集群中**每个 pod 的同一端口上公布服务**。
- 可以通过访问集群中任意节点+端口号的方式访问服务 `<NodeIP>:<NodePort>`。
- ClusterIP 的访问方式仍然可用。

```bash
[root@master ~]# kubectl get service -n test -o wide
NAME           TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE   SELECTOR
cloud-config   NodePort   10.96.249.98   <none>        8888:32155/TCP   21d   k8s.kuboard.cn/layer=cloud,k8s.kuboard.cn/name=cloud-config
cloud-eureka   NodePort   10.96.181.80   <none>        8761:31739/TCP   27d   k8s.kuboard.cn/layer=cloud,k8s.kuboard.cn/name=cloud-eureka
```

# service yaml文件示例

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service	#Service 的名称
  labels:     	#Service 自己的标签
    app: nginx	#为该 Service 设置 key 为 app，value 为 nginx 的标签
spec:	    #这是关于该 Service 的定义，描述了 Service 如何选择 Pod，如何被访问
  selector:	    #标签选择器
    app: nginx	#选择包含标签 app:nginx 的 Pod
  ports:
  - name: nginx-port	#端口的名字
    protocol: TCP	    #协议类型 TCP/UDP
    port: 80	        #集群内的其他容器组可通过 80 端口访问 Service
    nodePort: 32600   #通过任意节点的 32600 端口访问 Service
    targetPort: 80	#将请求转发到匹配 Pod 的 80 端口
  type: NodePort	#Serive的类型，ClusterIP/NodePort/LoaderBalancer
```

# 参考
1. [Service中spec.type 字段的值：ClusterIP和NodePort理解 ](https://www.cnblogs.com/hahaha111122222/p/13403771.html)