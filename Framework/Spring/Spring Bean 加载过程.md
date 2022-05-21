#Spring #Springboot #Todo #Bean


## 1. 概述

Spring 作为 Ioc 框架，实现了依赖注入，由一个中心化的 Bean 工厂来负责各个 Bean 的实例化和依赖管理。各个 Bean 可以不需要关心各自的复杂的创建过程，达到了很好的解耦效果。

我们对 Spring 的工作流进行一个粗略的概括，主要为两大环节：

-   **解析**，读 xml 配置，扫描类文件，从配置或者注解中获取 Bean 的定义信息，注册一些扩展功能。
-   **加载**，通过解析完的定义信息获取 Bean 实例。
  
  ![[spring 总体流程.png]]
假设所有的配置和扩展类都已经装载到了 ApplicationContext 中，然后具体的分析一下 Bean 的加载流程。

思考一个问题，抛开 Spring 框架的实现，假设我们手头上已经有一套完整的 Bean Definition Map，然后指定一个 beanName 要进行实例化，需要关心什么？即使我们没有 Spring 框架，也需要了解这两方面的知识：

-   **作用域**。单例作用域或者原型作用域，单例的话需要全局实例化一次，原型每次创建都需要重新实例化。
-   **依赖关系**。一个 Bean 如果有依赖，我们需要初始化依赖，然后进行关联。如果多个 Bean 之间存在着循环依赖，A 依赖 B，B 依赖 C，C 又依赖 A，需要解这种循环依赖问题。

Spring 进行了抽象和封装，使得作用域和依赖关系的配置对开发者透明，我们只需要知道当初在配置里已经明确指定了它的生命周期和依赖了谁，至于是怎么实现的，依赖如何注入，托付给了 Spring 工厂来管理。

Spring 只暴露了很简单的接口给调用者，比如 `getBean` ：
```java
ApplicationContext context = new ClassPathXmlApplicationContext("hello.xml");
HelloBean helloBean = (HelloBean) context.getBean("hello");
helloBean.sayHello();
```
  
  




# 参考
1. 图文并茂，揭秘 Spring 的 Bean 的加载过程（简书）
2. [spring bean的加载过程](https://calmtime.github.io/2020/05/02/spring-bean/)
