# ClassLoader
`ClassLoader` 即类加载器，负责将类加载到 JVM。在 Java 虚拟机外部实现，以便让应用程序自己决定如何去获取所需要的类。

JVM 加载 `class` 文件到内存有两种方式：

-   隐式加载 - JVM 自动加载需要的类到内存中。
-   显示加载 - 通过使用 `ClassLoader` 来加载一个类到内存中

## 分类
![[ClassLoader分类.png]]