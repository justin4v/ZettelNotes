#Spring #SourceCode #Todo 

# 结构
*Spring 中默认的 IoC 完整功能实现*，继承关系如下：
![[DefaultListableBeanFactory继承关系.png|700]]


# 意图/功能简介
## Registry
 *AliasRegistry*：定义对 Bean *Alias* 的**注册、删除、获取操作规范**（*规约层面要求对 Alias 具有注册、删除、查询*操作）。
-   *SimpleAliasRegistry*：
	- 对接口 AliasRegistry 的简单*实现*，使用 *aliasMap（map<name, alias>）* 作为缓存。
	- 对 Alias 的*注册、删除、获取*操作都是针对 aliasMap 的操作。
-   *BeanDefinitionRegistry*：定义对 BeanDefinition 的**注册、删除、获取操作规范**。
	- Spring beanFactory 中*唯一定义了 BeanDefinition 注册操作规范*的接口。
	- 标准的 BeanFactory 接口一般只包括对*完全配置完成的（fully configured）* BeanFactory 实例的访问。
	- `DefaultListableBeanFactory` 实现了该接口：BeanDefinition 存储（同时操作处于）在 *Map<String, BeanDefinition> beanDefinitionMap*。

## BeanFactory
-   *HierarchicalBeanFactory*：
	- 继承自 `BeanFactory`，用于处理具有继承层级( Hierarchy )结构的 BeanFactory。
	- 增加了对 `parentFactory` 的支持（`getParentBeanFactory()`）。
	- `parentFactory` 的设置在 `ConfigurableBeanFactory#setParentBeanFactory`。
-   *ListableBeanFactory*：定义了**根据各种条件获取 bean 和 beanDefinition 的操作规范**。
-   *ConfigurableBeanFactory*：定义了**配置 BeanFactory 的各种操作规范**。
	- 应用代码不应该直接使用该接口，应使用 `BeanFactory` 和 `ListableBeanFactory`。
	- 这个扩展接口是为了支持*框架内部的即插即用*和*对 beanFactory 配置方法的特殊访问*。
-   *AutowireCapableBeanFactory*：定义了**创建 bean、自动注入、初始化以及应用bean的后处理器等操作规范**。
	- 应用代码不应该直接使用该接口，应使用 `BeanFactory` 和 `ListableBeanFactory`。
	- 其他框架的集成代码可利用该接口*填充 Spring 无法控制的 bean 实例*。
-   *AbstractAutowireCapableBeanFactory*：综合 AbstractBeanFactory 并对接口 AutowireCapableBeanFactory 进行实现。
	- 提供了 *bean creation (with constructor resolution), property population, wiring (including autowiring), and initialization*. Handles runtime bean references, resolves managed collections, calls initialization methods, etc. Supports autowiring constructors, properties by name, and properties by type。
	- 需要子类实现的主要模板方法是 `resolveDependency(DependencyDescriptor, String, Set, TypeConverter)` 用于*按类型自动装配*。
- *ConfigurableListableBeanFactory*：BeanFactory 配置清单，指定忽略类型及接口等。
	- 提供了*分析和修改 beanDefinition 以及预实例化单例的方法*。

-   *SingletonBeanRegistry*：定义对单例（singleton）的**注册、获取操作规范**。
-   *DefaultSingletonBeanRegistry*：接口 SingletonBeanRegistry 的*默认实现*。持有三种缓存（目的是为了**解决 setter 和 field 注入的循环依赖**）：
	- `Map<String, Object> singletonObjects`：singleton objects 缓存 *bean name -> bean instance*;
	- `Map<String, ObjectFactory<?>> singletonFactories`: singleton factories 缓存 *bean name -> ObjectFactory*;
	- `Map<String, Object> earlySingletonObjects`: early singleton objects 缓存 *bean name -> bean instance*，提前暴露的 Singleton bean 引用（刚注册到 beanFactory *还没有注入属性等*）。
	-   *FactoryBeanRegistrySupport*：在 `DefaultSingletonBeanRegistry` 基础上增加了对 FactoryBean 其他功能。
		- 如`getObjectFromFactoryBean()` 功能；
		- 持有 `Map<String, Object> factoryBeanObjectCache` singleton objects created by FactoryBeans 缓存： *FactoryBean name -> object*。
		- 作为 `AbstractBeanFactory` 基类。
-   *AbstractBeanFactory*：综合 FactoryBeanRegistrySupport 和 ConfigurableBeanFactory 的功能。
	- 主要 *模板方法* 是 `getBeanDefinition` 和 `createBean`，分别*查询 beanDefinition 和 创建 bean 实例*，由子类实现。
	- BeanFactory implementation 的*抽象基类*，提供了 *ConfigurableBeanFactory SPI 的全部功能*；
	- 默认实现为  `DefaultListableBeanFactory`.


-  DefaultListableBeanFactory：综合上面所有功能，主要是对bean注册后的处理。