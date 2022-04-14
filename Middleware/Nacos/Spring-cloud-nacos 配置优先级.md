#Spring #Spring-cloud #Nacos

spring-boot项目，有bootstrap、application两个配置文件，结合profile，和支持的文件格式properties、yaml，已经有6个配置文件了。然后使用了spring-cloud-starter-alibaba-nacos-config 后，又提供了三级配置。

# bootstrap.yaml
- bootstrap.yml 用于应用程序上下文的引导阶段；
- 由父 Spring ApplicationContext 加载，其工作的阶段为：*父 ApplicationContext 被加载*到*使用 application.yml 之前*。
- bootstrap 加载优先于 applicaton.yaml。
- 用于*从额外的资源来加载配置信息*，bootstrap 里面的属性会优先加载，默认不能被本地相同配置覆盖。

## 应用场景
-  bootstrap 中*添加外部配置中心的连接属性*，以加载外部配置；
-  *系统级别的配置*，不会变动的、不能被覆盖的属性；
-   *加密 / 解密*的场景；

# application.yaml
- *应用级别的 spring-boot 配置文件*；
- 用于 Spring Boot 项目的自动化配置，其加载优先级低于bootstrap.yaml。
- spring-boot 默认有4个方式确定 application 配置：
> -   假设当前目录为 `/`，查找优先级：`/config` > `/` > `classpath:/config` > `classpath:/`
> -   不同配置文件中的同名属性，*按优先级高低覆盖*。
> -   不同配置文件中的非同名属性，*合并生效*。
> -   同名但不同后缀：后缀 `properties` > `yaml`

-   指定 profile 的配置优先级高于无 profile 的配置。
-   在启动命令中通过 `--spring.config.location` 指定配置文件路径。该配置优先级最高。

# spring-cloud-nacos 配置文件
- nacos 作为外部配置服务器，通过 bootstrap.yaml 引入。
- nacos本身提供了*三级配置体系*：
	1. 主配置：只有一个，按照不同后缀名，加载配置；
	2. 扩展配置；
	3. 共享配置。
- 优先级：主配置 > 扩展配置 > 共享配置

## 主配置
- nacos 提供的配置路径 `spring.cloud.nacos.config` 下，有一系列的属性用于定位主配置：
	- *prefix*（默认为 `${spring.application.name}` 的值）；
	- *namespace*；
	- *group*（默认为字符串 `DEFAULT_GROUP`）；
	- *file-extension*（默认为字符串 `.properties`）；
- 按命名规则  `${prefix}-${spring.profiles.active}.${file-extension}` 加载配置。

## 示例
```yaml
spring:
  application:
    name: ddd-demo-service
  cloud:
    nacos:
      config:
        server-addr: nacos-2.nacos-headless.public.svc.cluster.local:8848
        namespace: test2
```

> spring-boot 和 nacos  将按照如下规则执行：
> 
> 1.  按照规则 `${prefix}-${spring.profiles.active}.${file-extension}` 得到 dataId：ddd-demo-service.properties。 
> > -   `${prefix}`：取默认值 `${spring.application.name}`： `ddd-demo-service`。
> > -   `spring.profiles.active`：取默认值空。
> > -   `${file-extension}`：取默认值字符串：`.properties`
> 2.  spring-boot 将在 nacos 的 test2 *namespace*，在 *DEFAULT_GROUP* group 分组下，加载一个 *dataId* 为 ddd-demo-service.properties 的配置。
> 3.  如果dataId不存在，则继续尝试加载dataId 为 ddd-demo-service 的配置，去掉`file-extension`后缀名。
> 4.  如果两步都没有找到，就不再尝试去找主配置。

- 在 nacos 中，主配置（存在的情况下）具有最高的优先级；
- 同名配置不能被同名属性所覆盖。

## 共享配置和扩展配置
-  `spring.cloud.nacos.config.extension-configs` 下，允许指定一个或多个额外配置。
-  `spring.cloud.nacos.config.shared-configs` 下，允许指定一个或多个共享配置。
- 上述两类配置都支持三个属性：`data-id`、`group`（默认为字符串 `DEFAULT_GROUP`）、`refresh`（默认为`true`）。

### 共享配置和扩展配置的区别

-  `extension-configs` 和 `shared-configs` 优先级不同;
-   nacos对配置的默认理念

> nacos 的默认理念：
> -   namespace区分环境：开发环境、测试环境、预发布环境、生产环境
> -   group 区分不同应用：同一个环境内，不同应用的配置，通过group来区分。

-   主配置是应用专有的配置。
> 在 dataId 上要区分；
> 最好还要有 group 的区分；

-   要在各应用之间共享一个配置，使用 shared-configs。
> shared-configs 指定的配置；
> 本来应该是不指定 group的，应当归入 DEFAULT_GROUP 公共分组。

-   如果要在特定范围内（比如某个应用上）覆盖某个共享 dataId上的特定属性，使用 extension-configs
> - 比如，其他应用的数据库url，都是一个固定的url，使用 shared-configs.dataId = mysql 的共享配置。  
> - 但应用 ddd-demo 是特例，需要配置扩展属性来覆盖。所以定义如下扩展配置：
> 
> ```
> spring:
>  application:
>    name: ddd-demo-service
>  cloud:
>    nacos:
>      config:
>        server-addr: nacos-2.nacos-headless.public.svc.cluster.local:8848
>        namespace: test2
>        group: ddd-demo
>        ......
>        shared-configs[3]:
>          data-id: mysql.yaml
>          refresh: true
>        ......
>        extension-configs[3]:
>          data-id: mysql.yaml
>          group: ddd-demo
>          refresh: true
> ```
> 
> -   step1： 在 group 为 DEFAULT_GROP 的分组内，创建各应用的共享配置 dataId 为 mysql
> -   step2：在 group 为 ddd-demo 的分组内，创建 ddd-demo 应用专有的配置 mysql，其中定义的属性，会覆盖上述共享配置中定义的同名属性。

### 优先级
-   对同种配置，数组元素对应的下标越大，优先级越高。
- 排在后面的相同配置，将覆盖排在前面的同名配置。

> 比如：
> -   同为扩展配置，存在如下优先级关系：`extension-configs[3] > extension-configs[2] > extension-configs[1] > extension-configs[0]`
> -   同为共享配置，存在如下优先级关系：`shared-configs[3] > shared-configs[2] > shared-configs[1] > shared-configs[0]`


### 在nacos中配置的注意事项

-   主配置
> 主配置有 ${file-extension} 来指定文件后缀，dataId可以带文件后缀，也可以不要带文件后缀。  
> spring-boot 会先用带后缀的文件读取，再尝试不带后缀。

-   共享配置/扩展配置
> 共享配置和扩展配置没有 filez-extension 属性；
> spring-boot 会认为 dataId 本身已经包含了 文件后缀。如果当发现按照配置中指定的dataId去nacos取回来的文件没有后缀名时，spring-boot将不会识别读取回来的文件。


# 参考
1. [spring-cloud-nacos配置优先级](https://www.jianshu.com/p/69a76cdecdb9)