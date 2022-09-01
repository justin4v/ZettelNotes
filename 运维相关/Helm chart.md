#Helm #k8s #Chart #Todo 

# 简介
- Helm是Kubernetes的包管理工具，如果把 k8s 比作操作系统，那么Helm 就好比yum，apt-get。
- 使用Helm template可以方便我们部署和管理自己的应用。
- Helm 使用 gotemplate 模板语言来编写 Kubernetes 资源（deployment,service, etc...）的模板文件，并**可让用户配置模板变量**。
- 在部署时Helm通过模板引擎将模板渲染成真正的Kubernetes资源文件，并将它们部署到节点上。



# template
-   模板渲染**本质就是一种字符串替换**,一种高级的字符串替换.



# 参考
1. [Helm template快速入门 ](https://juejin.cn/post/6844904199818313735#heading-0)
2. [Helm Chart语法概要 ](https://www.cnblogs.com/ssgeek/p/15511387.html)