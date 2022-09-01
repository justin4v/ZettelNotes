#Helm #k8s #Chart #Todo 

# 简介
- Helm是Kubernetes的包管理工具，如果把 k8s 比作操作系统，那么Helm 就好比yum，apt-get。
- Helm主要解决以下问题：
     1. 把**配置(yaml)作为一个整体管理**；
     2. 实现 yaml 的高效复用；
     3. 实现**应用级别的版本管理**。
- 使用Helm template可以方便我们部署和管理自己的应用。
- Helm 使用 gotemplate 模板语言来编写 Kubernetes 资源（deployment,service, etc...）的模板文件，并**可让用户配置模板变量**。
- 在部署时Helm通过模板引擎将模板渲染成真正的Kubernetes资源文件，并将它们部署到节点上。

# Chart
- 创建模板：
```shell
$ helm create demo
```

- 在当前目录下创建如下结构的文件夹：
```shell
demo                                # Chart 目录
├── charts                          # 这个 charts 依赖的其他 charts，始终被安装
├── Chart.yaml                      # 描述这个 Chart 的相关信息、包括名字、描述信息、版本等
├── templates                       # 模板目录
│   ├── deployment.yaml             # deployment 控制器的 Go 模板文件
│   ├── _helpers.tpl                # 以 _ 开头的文件不会部署到 k8s 上，可用于定制通用信息
│   ├── ingress.yaml                # ingress 的模板文件
│   ├── NOTES.txt                   # Chart 部署到集群后的一些信息，例如：如何使用、列出缺省值
│   ├── service.yaml                # service 的 Go 模板文件
└── values.yaml                     # 模板的值文件，这些值会在安装时应用到 GO 模板生成部署文件
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