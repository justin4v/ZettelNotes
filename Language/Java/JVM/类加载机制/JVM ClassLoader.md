# ClassLoader
`ClassLoader` 即类加载器，负责将类加载到 JVM。在 Java 虚拟机外部实现，以便让应用程序自己决定如何去获取所需要的类。

JVM 加载 `class` 文件到内存有两种方式：
-   隐式加载 - JVM 自动加载需要的类到内存中。
-   显示加载 - 通过使用 `ClassLoader` 来加载一个类到内存中


## 类的唯一标识
- Java中，一个类用其全限定类名（包括包名和类名）作为标识；
- JVM中，一个类用其全限定类名和其类加载器作为其唯一标识。

在同一命名空间（namespace）中，同一个类只能被加载一次。

参见：[[NameSpace]]

## 分类
![[ClassLoader分类.png]]


# 创建实例对象
当new 一个新对象或者引用静态成员变量时:
1. JVM 中 ClassLoader SubSystem 将 java.lang.Class 对象加载到JVM中;
2. JVM **根据类型信息创建实例对象或者静态变量的引用值**。


# 参考
1. [Java类加载器 — classloader 的原理及应用](https://developer.aliyun.com/article/792033)
2. [Java系列 | 远程热部署在美团的落地实践](https://segmentfault.com/a/1190000041581466)