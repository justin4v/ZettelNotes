#运维
# 概述
- Kubectl 是命令行工具，用于管理 Kubernetes 集群；
- `kubectl` 在 `$HOME/.kube` 目录中查找一个名为 `config` 的配置文件。 通过设置 KUBECONFIG 环境变量或设置 `--kubeconfig`参数来指定其它 kubeconfig 文件。

# 语法
```shell
kubectl [command] [TYPE] [NAME] [flags]
```
-   `command`：*对一个或多个资源的操作*，例如 `create`、`get`、`describe`、`delete`。
-   `TYPE`：*[资源类型](https://kubernetes.io/zh/docs/reference/kubectl/overview/#%E8%B5%84%E6%BA%90%E7%B1%BB%E5%9E%8B)，通过`kubectl api-resources`命令获取*。资源类型不区分大小写， 可以指定单数、复数或缩写形式。
	例如，以下命令输出相同的结果:
    ```shell
    kubectl get pod pod1
    kubectl get pods pod1
    kubectl get po pod1
    ```
-   `NAME`：指定资源的名称。名称区分大小写。 如果省略名称，则显示所有资源的详细信息 `kubectl get pods`。
    对多个资源执行操作时，可以按类型和名称指定每个资源，或指定一个或多个文件：
    -   要按类型和名称指定资源：
        -   要对所有类型相同的资源进行分组，请执行以下操作：`TYPE1 name1 name2 name?`。
            例子：`kubectl get pod example-pod1 example-pod2`
        -   多个`类型/资源`：`TYPE1/name1 TYPE1/name2 TYPE?/name?`。
            例子：`kubectl get pod/example-pod1 replicationcontroller/example-rc1`
    -   指定文件资源：`-f file1 -f file2 -f file<#>`
        -   *用 YAML 而不是 JSON*，因为 YAML 更容易使用，特别是用于配置文件时。 例子：`kubectl get -f ./pod.yaml`
-   `flags`: *可选的参数*。例如，可以使用 `-s` 或 `-server` 参数指定 Kubernetes API 服务器的地址和端口。

**注意：**

- 从命令行指定的参数会覆盖默认值和任何相应的环境变量。
- `kubectl --help`获取帮助


# 常用命令
1. 列出群集中的 Namespace 列表
```bash
kubectl get namespaces
```
默认的两个 namespace
-   `default`
-   `kube-system`：由 Kubernetes 系统创建的对象的Namespace

2. 列出所有命名空间下的全部的 Pods
```bash
kubectl get pods --all-namespaces 
```

3. describe 命令的详细输出
```bash
kubectl describe nodes my-node
kubectl describe pods my-pod
```

4. 列出所支持的全部资源类型和它们的简称

```bash
kubectl api-resources
```

5. 日志
```bash
kubectl logs my-pod                                 # 获取 pod 日志（标准输出）
kubectl logs -l name=myLabel                        # 获取含 name=myLabel 标签的 Pods 的日志（标准输出）
kubectl logs my-pod --previous                      # 获取上个容器实例的 pod 日志（标准输出）
kubectl logs my-pod -c my-container                 # 获取 Pod 容器的日志（标准输出, 多容器场景）
kubectl logs -l name=myLabel -c my-container        # 获取含 name=myLabel 标签的 Pod 容器日志（标准输出, 多容器场景）
kubectl logs my-pod -c my-container --previous      # 获取 Pod 中某容器的上个实例的日志（标准输出, 多容器场景）
kubectl logs -f my-pod                              # 流式输出 Pod 的日志（标准输出）
kubectl logs -f my-pod -c my-container              # 流式输出 Pod 容器的日志（标准输出, 多容器场景）
```

6. 进入 pod(运行中) 进行交互
```bash
kubectl exec my-pod -- ls /                         # 在已有的 Pod 中运行命令（单容器场景）
kubectl exec --stdin --tty my-pod -- /bin/sh        # 使用交互 shell 访问正在运行的 Pod (一个容器场景)
kubectl exec my-pod -c my-container -- ls /         # 在已有的 Pod 中运行命令（多容器场景）
kubectl top pod POD_NAME --containers               # 显示给定 Pod 和其中容器的监控数据
kubectl top pod POD_NAME --sort-by=cpu              # 显示给定 Pod 的指标并且按照 'cpu' 或者 'memory' 排序
```



# 参考
1. [kubectl 备忘单](https://kubernetes.io/zh/docs/reference/kubectl/cheatsheet/)