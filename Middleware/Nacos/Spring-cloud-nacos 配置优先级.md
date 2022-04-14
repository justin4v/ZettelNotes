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
-   spring-boot 默认会从4个路径去找 application 配置：
> -   按优先级，假设当前运行目录为`/`，优先级依次为：`/config` > `/` > `classpath:/config` > `classpath:/`
> -   同时出现在上述四个配置文件中的同名属性，按优先级的覆盖低优先级的。
> -   分别出现在不同配置文件中的非同名属性，合并后生效。
> -   对同名但不同后缀名的配置文件，`properties` > `yaml`

-   对存在profile的场景，指定profile的配置优先级高于无profile的配置。
-   还可以在启动命令中指定外部配置文件路径。

> 可以在启动命令时，通过 `--spring.config.location` 命令参数指定用逗号分割的外部配置文件路径，该路径下配置高于一切。

# spring-cloud-nacos引入的三级配置文件

nacos作为外部配置服务器，通过spring-boot的bootstrap.yaml引入。但nacos本身，也提供了三级配置体系：主配置(只有一个，但会按照不同后缀名，去找到相关配置）、扩展配置、共享配置。

三级配置的优先级如下：主配置 > 扩展配置 > 共享配置

## 主配置

nacos提供的配置路径 `spring.cloud.nacos.config` 下，有一系列的属性用于定位主配置。基于 prefix（默认为 `${spring.application.name}` 的值）、namespace、group（默认为字符串 `DEFAULT_GROUP`）、file-extension（默认为字符串 `.properties`），按组装规则 `${prefix}-${spring.profiles.active}.${file-extension}`去找到一个配置。

```
spring:
  application:
    name: ddd-demo-service
  cloud:
    nacos:
      config:
        server-addr: nacos-2.nacos-headless.public.svc.cluster.local:8848
        namespace: ygjpro-test2
```

> 上述配置，意味spring-boot和nacos 将按照如下规则执行：
> 
> 1.  按照规则 `${prefix}-${spring.profiles.active}.${file-extension}`来获得dataId：ddd-demo-service.properties。因为：
> 
> > -   `${prefix}`：没有指定 `${prefix}` ，取默认值 `${spring.application.name}`，为字符串 `ddd-demo-service`。
> > -   `spring.profiles.active`：没有指定 `spring.profiles.active`，取默认值空。
> > -   `${file-extension}`：没有指定 `${file-extension}`，取默认值字符串`.properties`
> 
> 2.  spring-boot将尝试去nacos的 ygjpro-test2 表空间，在 DEFAULT_GROUP group 分组下，加载一个dataId叫做 ddd-demo-service 的配置。
> 3.  如果发现上述dataId不存在，则继续尝试加载名为 ddd-demo-service 的dataId，该dataId只是前面步骤1中获得的dataId，去掉`file-extension`后缀名。
> 4.  如果上述两个步骤都没有找到dataId，就不再尝试去找主配置了。

在nacos的所有配置中，主配置（存在的情况下）具有最高的优先级，其同名配置值不能被扩展配置或共享配置中定义的同名属性所覆盖。

## 共享配置和扩展配置

-   nacos在配置路径 `spring.cloud.nacos.config.extension-configs` 下，允许我们指定一个或多个额外配置。
-   nacos在配置路径 spring.cloud.nacos.config.shared-configs 下，允许我们指定一个或多个共享配置。

上述两类配置都支持三个属性：`data-id`、`group`（默认为字符串 `DEFAULT_GROUP`）、`refresh`（默认为`true`）。

### 共享配置和扩展配置的区别

实际上，nacos中并未对 `extension-configs` 和 `shared-configs` 的差别进行详细阐述。我们从他们的结构，看不出本质差别；除了优先级不同以外，也没有其他差别。那么，nacos项目组为什么要引入两个类似的配置呢?我们可以从当初[该功能的需求（issue）](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Falibaba%2Fspring-cloud-alibaba%2Fissues%2F141)上找到其原始目的。  
摘要其核心内容如下：

-   nacos对配置的默认理念

> 根据nacos的默认理念：
> 
> -   namespace区分环境：开发环境、测试环境、预发布环境、生产环境
> -   group区分不同应用：同一个环境内，不同应用的配置，通过group来区分。

-   主配置是应用专有的配置。

> 因此，主配置应当在dataId上要区分，同时最好还要有 group 的区分，因为group区分应用（虽然dataId上区分了，不用设置group也能按应用单独加载）。

-   要在各应用之间共享一个配置，请使用上面的 shared-configs。

> 因此按该理念，shared-configs 指定的配置，本来应该是不指定 group的，也就是应当归入 DEFAULT_GROUP 这个公共分组。

-   如果要在特定范围内（比如某个应用上）覆盖某个共享dataId上的特定属性，请使用 extension-configs

> 比如，其他应用的数据库url，都是一个固定的url，使用 shared-configs.dataId = mysql 的共享配置。  
> 但其中有一个应用 ddd-demo 是特例，需要为该应用配置扩展属性来覆盖。所以定义如下扩展配置：
> 
> ```
> spring:
>  application:
>    name: ddd-demo-service
>  cloud:
>    nacos:
>      config:
>        server-addr: nacos-2.nacos-headless.public.svc.cluster.local:8848
>        namespace: ygjpro-test2
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
> -   step1： 我们在group 为 DEFAULT_GROP 的分组内，创建各应用的共享配置dataId为mysql
> -   step2：我们在group 为 ddd-demo 的分组内，，创建 ddd-demo 应用专有的配置mysql，其中定义的属性，会覆盖上述共享配置中定义的同名属性。

### 优先级

-   上述两类配置都是数组，对同种配置，数组元素对应的下标越大，优先级越高。也就是排在后面的相同配置，将覆盖排在前面的同名配置。

> 比如：
> 
> -   同为扩展配置，存在如下优先级关系：`extension-configs[3] > extension-configs[2] > extension-configs[1] > extension-configs[0]`
> -   同为共享配置，存在如下优先级关系：`shared-configs[3] > shared-configs[2] > shared-configs[1] > shared-configs[0]`

-   不同种类配置之间，优先级按顺序如下：主配置 > 扩展配置（extension-configs）> 共享配置（shared-configs）。

### 在nacos中配置的注意事项

-   主配置

> 主配置有 ${file-extension} 来指定文件后缀，因此在根据规则生成 dataId时，spring-boot 知道如何去识别这个读取下来的文件。所以在nacos中配置时，dataId可以带文件后缀，也可以不要带文件后缀。  
> spring-boot 发现用不带文件后缀的dataId读取不到时，会尝试去掉文件后缀后的dataId 再去读取一次。

-   共享配置/扩展配置

> 共享配置和扩展配置没有 filez-extension 属性，因此 spring-boot 会认为 dataId 本身已经包含了 文件后缀了。如果当发现按照配置中指定的dataId去nacos取回来的文件没有后缀名时，spring-boot将不会识别读取回来的文件。


# 参考
1. [spring-cloud-nacos配置优先级](https://www.jianshu.com/p/69a76cdecdb9)