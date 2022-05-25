#Spring #SourceCode #Todo 

# 结构
*Spring 中默认的 IoC 完整功能实现*，继承关系如下：
![[DefaultListableBeanFactory继承关系.png|700]]


# 意图功能简介
-   *AliasRegistry*：定义对 Bean *Alias* 的**注册、删除、获取操作规范**（*规约层面要求对 Alias 具有注册、删除、查询*操作）。
-   *SimpleAliasRegistry*：
	- 对接口 AliasRegistry 的简单*实现*，使用 *aliasMap（map<name, alias>）* 作为缓存。
	- 对 Alias 的*注册、删除、获取*操作都是针对 aliasMap 的操作

-   *SingletonBeanRegistry*：定义对单例的**注册、获取操作规范**。
-   *DefaultSingletonBeanRegistry*：接口 SingletonBeanRegistry 的默认实现。

-   *BeanFactory*：定义*获取 bean 和各种属性（factory）的操作规范*。
-   *HierarchicalBeanFactory*：
	- 继承自 `BeanFactory`，用于处理具有继承层级( Hierarchy )结构的 BeanFactory。
	- 增加了对 `parentFactory` 的支持（`getParentBeanFactory()`）。
	- `parentFactory` 的设置
-   BeanDefinitionRegistry：定义对BeanDefinition的各种增删改操作。
-   FactoryBeanRegistrySupport：在DefaultSingletonBeanRegistry基础上增加了对FactoryBean的特殊处理功能。
-   ConfigurableBeanFactory：提供配置Factory的各种方法。
-   ListableBeanFactory：根据各种条件获取bean的配置清单。
-   AbstractBeanFactory：综合FactoryBeanRegistrySupport和ConfigurableBeanFactory的功能。
-   AutowireCapableBeanFactory：提供创建bean、自动注入、初始化以及应用bean的后处理器。
-   AbstractAutowireCapableBeanFactory：综合AbstractBeanFactory并对接口Autowire Capable BeanFactory进行实现。