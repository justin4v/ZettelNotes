#Classpath #Java基础 

# 含义
- classpath，顾名思义，就是存放 `*.class` 类文件的路径
- 也是 ClassLoader 加载类时寻找 `*.class `文件的路径。


## 示例
以一个 WEB 项目为例，发布后的目录结构大致如下：
![[web项目发布后目录结构.png]]


以 Tomcat 部署为例，参考 Tomcat Class Loader How-To 中的说明:

> WebappX — A class loader is created for each web application that is deployed in a single Tomcat instance. All unpacked classes and resources in the **/WEB-INF/classes** directory of your web application, plus classes and resources in JAR files under the **/WEB-INF/lib** directory of your web application, are made visible to this web application, but not to other ones.

- 因此，对于部署在Tomcat上的WEB应用，*/WEB-INF/classes* 和 */WEB-INF/lib* 目录就是 *classpath*。


# 获取 Classpath

**ClassLoader** 提供了两个方法用于从装载的类路径中取得资源：

```java
      public URL  getResource (String name);  
      public InputStream  getResourceAsStream (String name); 
```