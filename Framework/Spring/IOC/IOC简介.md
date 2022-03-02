#IoC #Spring
# 控制反转
- IoC：**Inversion of Control** 控制反转，是一种设计原则。
- 在2004年，Martin Fowler 提出“*哪些方面的控制被反转了？*"的问题。
- 结论是：**依赖对象的*获得*被反转了**。
- 他为控制反转创造了一个更好的名字：*依赖注入(DI, Dependency Injection)*，是 *IoC 原则的一种实现模式*。
	- 许多应用都是由*两个或多个类通过彼此的合作来实现业逻辑*，使得每个对象都需要合作对象（所依赖的对象）的引用。
	- 如果获取过程要靠自身实现，那么将导致代码*高度耦合且难以测试*。
- Spring IoC是Spring-Framework的核心。

# Spring IoC 容器系列简介

![[IoC 容器接口设计图.png]]

## BeanFactory
简单容器工厂（概念层次），包含了容器的基本功能
- BeanFactory 的 Specification (规约)如下

![[BeanFactory结构.png]]
- 从 *BeanFactory => HierarchicalBeanFactory => ConfigurableBeanFactory*，是一条主要的 BeanFactory 设计路径。
- *BeanFactory* 体现了**容器工厂的概念**，定义了基本的 *IoC 容器规范*。包含 loC 容器的基本方法（ 如 `getBean()`，从容器中取得Bean）。
- *HierarchicalBeanFactory* 继承了 BeanFactory，增加了 `getParentBeanFactory()` 功能，容器工厂具备了*双亲 IoC 容器管理功能*。
- *ConfigurableBeanFactory* 主要定义了对 *BeanFactory 的配置功能*：
	- `setParentBeanFactory()` 设置双亲 IoC 容器;
	- `addBeanPostProcessor()` 配置 Bean 后置处理器。
- 这些接口设计的叠加，定义了 BeanFactory 作为简单 loc容器的基本功能。

## ApplicationContext
应用上下文，高级容器系列，增加了面向框架特性

- 第二条设计主线：以 *ApplicationContext*(应用上下文) 为核心的接口设计，*BeanFactory => ListableBeanFactory =>App]icationContext => WebApplicationContext 或者ConfigurableApplicationContext*。
- 常用的应用上下文是 *ConfigurableApplicationContext* 或者 *WebApplicationContext* 的实现。
- *ListableBeanFactory* 指*可列出所有 bean* 而不是当客户端请求时一个一个查找的 Bean-Factorty。 如一些预先加载所有 bean  定义的工厂（XML-based factories ）。
- *ApplicationContext* 通过继承 *MessageSource（解析Message）*、*ResourceLoader（加载文件资源）*、*ApplicationEventPublisher（注册 Listener）* ，在BeanFactory 的基础上添加了*高级容器的特性*。

## BeanDefinition
 表达容器中 Bean 的概念