#Spring #SourceCode #Todo 

# 结构
*Spring 中默认的 IoC 完整功能实现*，继承关系如下：
![[DefaultListableBeanFactory继承关系.png|700]]


# 意图/功能简介
-   *AliasRegistry*：定义对 Bean *Alias* 的**注册、删除、获取操作规范**（*规约层面要求对 Alias 具有注册、删除、查询*操作）。
-   *SimpleAliasRegistry*：
	- 对接口 AliasRegistry 的简单*实现*，使用 *aliasMap（map<name, alias>）* 作为缓存。
	- 对 Alias 的*注册、删除、获取*操作都是针对 aliasMap 的操作。
-   *BeanDefinitionRegistry*：定义对 BeanDefinition 的**注册、删除、获取操作规范**。
	- Spring beanFactory 中*唯一定义了 BeanDefinition 注册操作规范*的接口。
	- 标准的 BeanFactory 接口一般只包括对*完全配置完成的（fully configured）* BeanFactory 实例的访问。
	- `DefaultListableBeanFactory` 实现了该接口：BeanDefinition 存储（同时操作处于）在 *Map<String, BeanDefinition> beanDefinitionMap*。

-   *SingletonBeanRegistry*：定义对单例（singleton）的**注册、获取操作规范**。
-   *DefaultSingletonBeanRegistry*：接口 SingletonBeanRegistry 的*默认实现*。持有三种缓存：
	- 三种缓存：`Map<String, Object> singletonObjects`


-   *HierarchicalBeanFactory*：
	- 继承自 `BeanFactory`，用于处理具有继承层级( Hierarchy )结构的 BeanFactory。
	- 增加了对 `parentFactory` 的支持（`getParentBeanFactory()`）。
	- `parentFactory` 的设置在 `ConfigurableBeanFactory#setParentBeanFactory`。

-   FactoryBeanRegistrySupport：在 DefaultSingletonBeanRegistry 基础上增加了对FactoryBean的特殊处理功能。
-   ConfigurableBeanFactory：提供配置Factory的各种方法。
-   ListableBeanFactory：根据各种条件获取bean的配置清单。
-   AbstractBeanFactory：综合FactoryBeanRegistrySupport和ConfigurableBeanFactory的功能。
-   AutowireCapableBeanFactory：提供创建bean、自动注入、初始化以及应用bean的后处理器。
-   AbstractAutowireCapableBeanFactory：综合AbstractBeanFactory并对接口Autowire Capable BeanFactory进行实现。
- ConfigurableListableBeanFactory：BeanFactory配置清单，指定忽略类型及接口等。
-  DefaultListableBeanFactory：综合上面所有功能，主要是对bean注册后的处理。