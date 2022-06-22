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

- 真正使用的不是 ClassLoader 的这两个方法，而是 Class 的 getResource 和 getResourceAsStream 方法
- 因为 Class 对象可以直接从类得到（如 YourClass.class 或 YourClass.getClass()），更加方便；
-  ClassLoader 则需要再调用一次 YourClass.getClassLoader()。
- Class 对象的这两个方法其实是“委托”（delegate）给装载它的 ClassLoader 实现。

目录结构如下：
![[classpath 目录结构示意.png]]

```java
1.this.getClass().getResource（""） 
得到的是当前 class 文件的 URI 目录。只到父级
如：
file:/E:/00justin/04-Project/01-com.uih/Workflow/workflow-research/target/test-classes/com/uih/uplus/workflowresearch/controller/v2/

2.this.getClass().getResource（"/"） 
得到的是当前的 classpath 的 URI 根路径 。
如：file:/E:/00justin/04-Project/01-com.uih/Workflow/workflow-research/target/test-classes/

3.this.getClass() .getClassLoader().getResource（""） 
得到的也是当前ClassPath的绝对URI路径 。
如：file：/D：/workspace/jbpmtest3/bin/

4.ClassLoader.getSystemResource（""） 
得到的也是当前ClassPath的绝对URI路径 。
如：file：/D：/workspace/jbpmtest3/bin/

5.Thread.currentThread().getContextClassLoader ().getResource（""） 
得到的也是当前ClassPath的绝对URI路径 。
如：file：/D：/workspace/jbpmtest3/bin/

```