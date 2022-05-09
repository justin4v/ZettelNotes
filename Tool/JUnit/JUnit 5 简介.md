#UT #JUnit

# 简介
- JUnit 是 Java 单元测试框架 ，与TestNG 占据了 Java 领域单元测试框架的主要市场；
- JUnit 最初是由 Kent Beck 和 Erich Gamma 1997年完成的，旨在提供好用的 Java 测试框架；
- JUnit 5 与以前版本不同，拆分成三个不同子模块组成。
	- JUnit 5 = JUnit Platform + JUnit Jupiter + JUnit Vintage
	- **JUnit Platform**： 用于JVM上启动测试框架的基础服务，提供命令行，IDE和构建工具等方式执行测试的支持。
	- **JUnit Jupiter**：包含 JUnit 5 新的编程模型和扩展模型，主要就是用于编写测试代码和扩展代码。
	- **JUnit Vintage**：兼容运行 JUnit3.x 和 JUnit4.x 的测试用例

JUnit 到目前的结构如下：
![[JUnit 架构.png]]


# JUnit 5特性
-   提供全新的断言和测试注解，支持测试类内嵌
    
-   更丰富的测试方式：支持*动态测试，重复测试，参数化测试*等
    
-   实现了模块化，让测试执行和测试发现等不同模块解耦，减少依赖
    
-   提供对 Java 8 的支持，如 Lambda 表达式，Sream API等。


# 参考
1. [Java单元测试之JUnit 5快速上手](https://www.cnblogs.com/one12138/p/11536492.html)