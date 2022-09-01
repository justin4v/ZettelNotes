#Helm #k8s #Chart #Todo 

# 简介
- Helm是Kubernetes的包管理工具，如果把 k8s 比作操作系统，那么Helm 就好比yum，apt-get。
- 使用Helm template可以方便我们部署和管理自己的应用。
- Helm 使用 gotemplate 模板语言来编写 Kubernetes 资源（deployment,service, etc...）的模板文件，并**可让用户配置模板变量**。
- 在部署时Helm通过模板引擎将模板渲染成真正的Kubernetes资源文件，并将它们部署到节点上。

# Chart
- 创建模板：
```shell
$ helm create mychart
```

- 在当前目录下创建如下结构的文件夹：
```shell
mychart
├── Chart.yaml
├── charts
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── service.yaml
│   ├── serviceaccount.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml
```

-   `templates/` 目录下是模板文件：
	- **核心，是模板化的 K8S manifests 文件**；
	- 当 Helm 需要生成 chart 的时，会**渲染该目录下的模板文件**，将**渲染结果发送给 kubernetes**。
-   `values.yaml` 文件保存 **template 的默认配置**：
	- 用户可修改；
	- 或在 `helm install` 或者 `helm upgrade`可以指定新的 **values.yaml** 来覆盖默认值。
-   `Chart.yaml`文件保存chart的**基本描述信息**，这些描述信息也可以在模板中被引用。
-   `_helper.tpl` 用于保存一些可以在该chart中复用的函数。



# template
-   模板渲染**本质就是一种字符串替换**,一种高级的字符串替换.



# 参考
1. [Helm template快速入门 ](https://juejin.cn/post/6844904199818313735#heading-0)
2. [Helm Chart语法概要 ](https://www.cnblogs.com/ssgeek/p/15511387.html)