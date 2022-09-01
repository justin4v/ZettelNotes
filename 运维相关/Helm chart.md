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


## 模板方法和管道

- 除了 [[#内置对象]]]  以外，可以使用 go template提供的**模板方法和管道（pipelines）** 来构建模板;
- 例如使用`quote`方法来为字符串添加双引号：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ quote .Values.favorite.drink }}
  food: {{ quote .Values.favorite.food }}
```

- gotemplate原生并没有提供太多方法，helm里面的许多方法来自这个库：[github.com/Masterminds…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FMasterminds%2Fsprig "https://github.com/Masterminds/sprig")
- Helm 方法完整列表：[helm.sh/docs/chart_…](https://link.juejin.cn?target=https%3A%2F%2Fhelm.sh%2Fdocs%2Fchart_template_guide%2Ffunction_list%2F "https://helm.sh/docs/chart_template_guide/function_list/")

### 管道
- 管道是 go template 提供的一个强大的功能；
- 借鉴UNIX中的管道操作(|)的概念。
- 管道是按顺序完成多项工作的有效方式。
使用管道重写以上示例：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | quote }}
  food: {{ .Values.favorite.food | quote }}
```

- 使用管道操作符(|)代替了原有的函数调用方式。与UNIX管道一样，模板的管道也支持链式操作：
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | quote }}
  # 大写，然后双引号包裹
  food: {{ .Values.favorite.food | upper | quote }}
复制代码
```

在链式操作中，**管道左侧的值会作为管道右侧函数中的最后一个参数**，例如`repeat`函数的函数签名为`repeat COUNT STRING`，则可以这样使用管道传参：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  # repeat 5 .Values.favorite.drink
  drink: {{ .Values.favorite.drink | repeat 5 | quote }}
  food: {{ .Values.favorite.food | upper | quote }}
复制代码
```

在编写Helm template时，建议优先使用管道来替代函数调用的方式。

## 控制流程

### IF/ELSE

if/else判断语句的语法如下：

```vbnet
{{ if PIPELINE }}
  # Do something
{{ else if OTHER PIPELINE }}
  # Do something else
{{ else }}
  # Default case
{{ end }}
复制代码
```

当`PIPELINE`值为以下内容，判定为`false`：

-   布尔值false
-   数字零
-   一个空字符串
-   nil
-   空的集合（map,数组）

下列模板将判断如果`.Values.favorite.drink == "coffee"`则新增`mug: true`。

```vbnet
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | default "tea" | quote }}
  food: {{ .Values.favorite.food | upper | quote }}
  {{ if eq .Values.favorite.drink "coffee" }}mug: true{{ end }}
复制代码
```

### With

```sql
{{ with PIPELINE }}
  # with声明的作用域
{{ end }}
复制代码
```

`with`用于更改当前作用域(`.`)。上文提到在`{{ .Release.Name }}`中，最左边的（`.`）表示当前作用域下的顶层命名空间，`.Values`告诉模板在当前作用域范围的顶层命名空间下查找`Values`对象。使用`with`可以改变模板变量的当前作用域，把（`.`）赋值给另一个对象：

```vbnet
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  {{- end }}
复制代码
```

在上面的例子中，在`with`的作用范围内（`{- with .xxxx}` 到 `{{- end}}`之间）可以直接引用`.drink`和`.food`，这是因为`{{- with .Values.favorite}}`把`Values.favorite`赋值给了当前作用域(`.`)。

### range

`range`用于循环遍历数组或是map。例如`values.yaml`文件中有如下信息：

```markdown
pizzaToppings:
  - mushrooms
  - cheese
  - peppers
  - onions
复制代码
```

使用`{{ range}}...{{ end}}`循环语句循环`pizzaToppings`数组：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  toppings: |-
    {{- range .Values.pizzaToppings }}
    - {{ . | title | quote }}
    {{- end }}
复制代码
```

上述模板的渲染结果如下：

```makefile
apiVersion: v1
kind: ConfigMap
metadata:
  name: RELEASE-NAME-configmap
data:
  toppings: |-
    - "Mushrooms"
    - "Cheese"
    - "Peppers"
    - "Onions"
复制代码
```

可以看到，`range`遍历了`.Values.pizzaToppings`数组，且将循环体内的作用域(`.`)设置成了每一次循环的值。`range`还可以同时获取迭代对象的`key`,`value`，例如有如下模板：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  toppings: |-
    {{- range $index, $topping := .Values.pizzaToppings }}
    {{ $index }}: {{ $topping }}
    {{- end }}
复制代码
```

其渲染结果如下：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: RELEASE-NAME-configmap
data:
  toppings: |-
    0: mushrooms
    1: cheese
    2: peppers
    3: onions
复制代码
```

### 空格处理

在上述的模板示例中，大家会注意到在一些模板指令里面会出现中划线`-`，例如`{{- xxx}}`或者`{{xxx -}}`。这个中划线的作用就是消除多于空格。例如有如下的模板：

```vbnet
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | default "tea" | quote }}
  food: {{ .Values.favorite.food | upper | quote }}
    {{ if eq .Values.favorite.drink "coffee" }}
  mug: true
    {{ end }}
复制代码
```

其渲染结果是：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: telling-chimp-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "PIZZA"

  mug: true
  
复制代码
```

可以看到，在`mug: true`前后各多了一个空行，这是因为在模板渲染的过程中，删除了`{{`和`}}`中的内容，但保留了其余的空白。使用`-`来消除模板语句占用的空格，为直观展示删除效果，在下列示例中，星号(*)表示被删除的空格：

```vbnet
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | default "tea" | quote }}
  food: {{ .Values.favorite.food | upper | quote }}*
**{{- if eq .Values.favorite.drink "coffee" }}
  mug: true*
**{{- end }}
复制代码
```

渲染结果：

```yaml
...
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "PIZZA"
  mug: true
...
复制代码
```

中横线在左边（`{{- xxx}}`）表示消除左边的空格，中横线在右边(`{{xxx -}}`)表示消除右边的空格。要小心不要写成了：

```lua
...
  food: {{ .Values.favorite.food | upper | quote }}
  {{- if eq .Values.favorite.drink "coffee" -}}
  mug: true
  {{- end -}}
...
复制代码
```

这样渲染出来的结果会是：`food: "PIZZA"mug:true`。 除了使用`-`消除模板的空格外，Helm还提供了`indent`函数增加空格来进行缩进：

```erlang
...
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "PIZZA"
{{ "mug: true" | indent2 }}  # 为"mug: true"增加两个空格的缩进。
...
复制代码
```

# 内置对象

Helm的顶级作用域`(.)`下可以直接引用的对象叫做内置对象，就好像上面示例中最常出现的`.Values.xxx`，这个`Values`就是Helm的内置对象之一。对象除了可以包含值以外，还可以包含方法。

> 文档参考: [helm.sh/docs/chart_…](https://link.juejin.cn?target=https%3A%2F%2Fhelm.sh%2Fdocs%2Fchart_template_guide%2Fbuiltin_objects%2F "https://helm.sh/docs/chart_template_guide/builtin_objects/")

**Release**

表示当次Release

-   `Release.Name` 当次release名称
-   `Release.Namespace` 发布到的命名空间（k8s namespace）
-   `Release.IsUpgrade` 如果当前操作是upgrade或rollback，则设为true
-   `Release.IsInstall` 如果当前操作是install，则为true
-   `Release.Revision` 当前release版本，从1开始，每次upgrade或rollback递增。
-   `Release.Service` 总是"Helm"

e.g:

```
{{ .Release.Name}}
复制代码
```

**Values**

用于渲染模板的变量，如`values.yaml` 文件中的变量，以及用户部署时指定的变量。（-set, -f）。

**Chart**

`Chart.yaml` 文件文件中的变量，例： `{{ .Chart.Name }}-{{ .Chart.Version}}`输出`mychart-0.1.0`。

**Files**

提供文件访问方法

-   `Files.Get` 获取指定文件名称的文件内容
-   `Files.GetBytes` 返回文件的二进制数组，读二进制文件时使用（例如图片）。
-   `Files.Glob` 返回符合给定shell glob pattern的文件数组，例:`{{ .Files.Glob "*.yaml" }}`
-   `Files.Lines` 按行遍历读取文件
-   `Files.AsSecrets` 返回文件内容的Base 64编码
-   `Files.AsConfig` 返回文件内容对应的YAML map

**Capabilities**

提供Kubernetes集群相关信息

-   Capabilities.APIVersions 集群支持的api version列表。
-   Capabilities.APIVersions.Has $version 指出当前集群是否支持当前API版本 (e.g., batch/v1)或资源 (e.g., apps/v1/Deployment)
-   Capabilities.KubeVersion / Capabilities.KubeVersion.Version Kubernetes版本号
-   Capabilities.KubeVersion.Major Kubernetes主版本号
-   Capabilities.KubeVersion.Minor Kubernetes子版本号

# 文件处理

结合`Files`内置变量，我们可以在模板中嵌入文件内容，这其中最常见的用法就是将应用的配置文件作为`ConfigMap`或者是`Secret`随应用一起发布。在上面的内置变量章节中，介绍了`Files`这个内置变量，聪明的你已经猜到其中`Files.AsConfig`和`Files.AsSecrets`就是用来把配置文件转换成`ConfigMap`和`Secret`的。这节我们来举例说明如何使用这两个函数。  
假设我们有`foo.yaml`和`bar.yaml`两个配置文件(存放在`values.yaml`同级目录下)：

```yaml
# foo.yaml
foo: true
fileName: foo.yaml
复制代码
```

```yaml
# bar.yaml
bar: true
fileName: bar.yaml
复制代码
```

使用如下`ConfigMap`模板:

```makefile
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-config
data:
{{(.Files.Glob "*.yaml").AsConfig | indent 2 }}
复制代码
```

渲染结果：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: RELEASE-NAME-config
data:
  bar.yaml: |-
    bar: true
    fileName: bar.yaml
  foo.yaml: |-
    foo: true
    fileName: foo.yaml

复制代码
```

在以上例子中，我们使用`.Files.Glob`函数来获取所有符合`*.yaml`pattern的文件，然后调用`AsConfig`方法输出以文件名称为key,文件内容为value的`yaml`格式，注意使用`indent`来增加合适的缩进以保证`yaml`文件的语法正确性，在本例中使用`ident 2`来缩进两个空格。 `AsSecret`的用法是类似的：

```makefile
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-config
data:
{{(.Files.Glob "*.yaml").AsSecrets | indent 2 }}
复制代码
```

渲染结果:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: RELEASE-NAME-config
data:
  bar.yaml: YmFyOiB0cnVlCmZpbGVOYW1lOiBiYXIueWFtbA==
  foo.yaml: Zm9vOiB0cnVlCmZpbGVOYW1lOiBmb28ueWFtbA==
复制代码
```

> 更多文件处理方法请参考文档：[helm.sh/docs/chart_…](https://link.juejin.cn?target=https%3A%2F%2Fhelm.sh%2Fdocs%2Fchart_template_guide%2Faccessing_files%2F "https://helm.sh/docs/chart_template_guide/accessing_files/")

# 模板引用/嵌套

模板嵌套允许我们在一个模板中去引入另一个模板，我们可以将可以复用的模板单独抽离，在使用时再引入。例如我们希望所有发布的资源都包含一组相同的label，这时候我们不需要在每种资源（例如deployment，service，configmap ....）的模板中复制粘贴label的模板代码，而可以将渲染label的模板单独定义，然后在各资源模板中引用即可。下面我们通过一个示例来说明如何引用模板。

在示例之前，我们先要了解一下Helm的文件命名约定：

-   `template/` 中的大多数文件都被视为Kubernetes资源清单(会被发往Kubernetes创建对应资源)
-   `NOTES.txt`是个例外
-   名称以下划线（_） 开头的文件不会被当做资源发往Kubernetes，但是可以被其他模板所引用。

这些下划线开头的文件用于定义局部模板。 实际上，当我们第一次创建mychart时，我们看到了一个名为_helpers.tpl的文件。 该文件是模板局部文件的默认位置。

`helm create mychart`所创建的默认模板中，已经为我们提供了一套局部模板，下面就以默认模板为例，演示如何引用模板。

## 定义模板

使用`{{ define}}...{{end}}`可以定义模板：

```arduino
{{ define "MY.NAME" }}
  # body of template here
{{ end }}
复制代码
```

例如在`_helpers.tpl`中，Helm定义了一些模板，以Kubernetes资源label为例：

```lua
...
{{- define "mychart.labels" -}}
helm.sh/chart: {{ include "mychart.chart" . }}
{{ include "mychart.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}
...
复制代码
```

使用`{{ include 模板名称}}`引用模板，在`deployment.yaml`中有：

```makefile
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "mychart.fullname" . }}
  # include引用mychart.labels
  labels:{{ include "mychart.labels" . | nindent 4 }}
复制代码
```

要注意的是，在使用`define`定义模板的时候，不要给模板加缩进，而是在使用`include`引用的时候搭配`nindent`或者是`indent`来缩进。`nindent`和`indent`方法类似，不过`nindent`会在最开始的地方增加一个空行，如果使用`indent`，则上面的模板可以写成：

```makefile
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "mychart.fullname" . | indent 2}}
  labels:
{{ include "mychart.labels" . | indent 4 }}
复制代码
```

# 附录：常用命令

### helm create

helm create用于创建模板

```ruby
# 创建一个名为mychart的template
$ helm create mychart
复制代码
```

### helm install

helm install用于部署chart

```shell
# 部署./redis目录，并设置release name为myredis
$ helm install myredis ./redis
复制代码
```

```shell
# 通过-f指定变量文件，（覆盖默认的values.yaml）
$ helm install -f myvalues.yaml myredis ./redis
复制代码
```

```shell
# -n 指定需要部署的命名空间
$ helm install myredis ./redis -n redis
复制代码
```

```shell
# 使用--set 覆盖values.yaml中的默认值
$ helm install --set name=prod myredis ./redis
复制代码
```

### helm uninstall

根据release name卸载chart

```ruby
# 卸载默认命名空间下的myredis
$ helm uninstall myredis 
复制代码
```

```ruby
# 卸载redis命名空间下的myredis
$ helm uninstall myredis -n redis
复制代码
```

### helm list

列举release

```ruby
# 列出默认命名空间下的所有release
$ helm list
复制代码
```

```ruby
# 列出redis命名空间下的release
$ helm list -n redis
复制代码
```

```ruby
# 列出所有命名空间下的release
$ helm list -A
复制代码
```

### --dry-run & --debug

使用`--dry-run`和`--debug`参数调试template

```bash
# 输出chart的渲染结果
helm install mychart --debug ./mychart
复制代码
```

```css
# 输出chart的渲染结果，并模拟部署（kubectl apply --dry-run ...)
helm install mychart --debug --dry-run./mychart
复制代码
```

# 参考

  
作者：水立方  
链接：https://juejin.cn/post/6844904199818313735  
来源：稀土掘金  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

# 参考
1. [Helm template快速入门 ](https://juejin.cn/post/6844904199818313735#heading-0)
2. [Helm Chart语法概要 ](https://www.cnblogs.com/ssgeek/p/15511387.html)