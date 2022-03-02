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
简单容器系列（概念层次），包含了容器的基本功能

![[Pasted image 20220302144707.png]]

## ApplicationContext
应用上下文，高级容器系列，增加了面向框架特性

## BeanDefinition
- 表达容器中 Bean 的概念