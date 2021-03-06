#Spring #IoC #SourceCode 
# Spring IoC 容器设计

![[IoC 容器接口设计图.png]]


# Bean Factory主线
## BeanFactory
- The *root interface for accessing a Spring bean container*. 
	> - This is the *basic client view* of a bean container; 
	> - further interfaces such as *ListableBeanFactory* and *ConfigurableBeanFactory* are available f*or specific purposes*.


简单容器工厂（概念层次），包含了容器的基本功能
- BeanFactory 的 Specification (规约)如下

![[BeanFactory结构.png]]
- 从 *BeanFactory => HierarchicalBeanFactory => ConfigurableBeanFactory*，是一条主**要的 BeanFactory 设计路径**。
- *BeanFactory* 体现了**容器工厂的概念**，定义了基本的 *IoC 容器规范*。包含 loC 容器的基本方法（ 如 `getBean()`，从容器中取得Bean）。
- *HierarchicalBeanFactory* 继承了 BeanFactory，增加了 `getParentBeanFactory()` 功能，容器工厂具备了*双亲 IoC 容器管理功能*。
- *ConfigurableBeanFactory* 主要定义了对 *BeanFactory 的配置功能*：
	- `setParentBeanFactory()` 设置双亲 IoC 容器;
	- `addBeanPostProcessor()` 配置 Bean 后置处理器。
- 这些接口设计的叠加，定义了 BeanFactory 作为简单 loc容器的基本功能。


# Bean Factory副线-高级特性
## ListableBeanFactory
- Extension of the BeanFactory interface to be implemented by bean factories that can *enumerate all their bean instances*, rather than attempting bean lookup by name one by one as requested by clients. BeanFactory implementations that *preload all their bean definitions* (such as XML-based factories) may implement this interface.
- If this is a HierarchicalBeanFactory, the return values will not take any BeanFactory hierarchy into account, but will relate only to the beans defined in the current factory. Use the BeanFactoryUtils helper class to consider beans in ancestor factories too.
- The methods in this interface *will just respect bean definitions of this factory*. They will ignore any singleton beans that have been registered by other means like ConfigurableBeanFactory's registerSingleton method, with the exception of getBeanNamesOfType and getBeansOfType which will check such manually registered singletons too. 
- Of course, BeanFactory's getBean does allow transparent access to such special beans as well. However, in typical scenarios, all beans will be defined by external bean definitions anyway, so most applications don't need to worry about this differentiation.
- NOTE: With the exception of getBeanDefinitionCount and containsBeanDefinition, the methods in this interface are not designed for frequent invocation. Implementations may be slow.

## ApplicationContext
- Central interface to *provide configuration for an application*. This is read-only while the application is running, but may be reloaded if the implementation supports this.
- An ApplicationContext provides:
	- *Bean factory methods for accessing application components*. Inherited from ListableBeanFactory.
	- The ability to *load file resources in a generic fashion*. Inherited from the org.springframework.core.io.ResourceLoader interface.
	- The ability to *publish events to registered listeners*. Inherited from the ApplicationEventPublisher interface.
	- The ability to *resolve messages, supporting internationalization*. Inherited from the MessageSource interface.
	- *Inheritance from a parent context*. Definitions in a descendant context will always take priority. This means, for example, that a single parent context can be used by an entire web application, while each servlet has its own child context that is independent of that of any other servlet.
- In addition to standard org.springframework.beans.factory.BeanFactory lifecycle capabilities, ApplicationContext implementations detect and invoke ApplicationContextAware beans as well as ResourceLoaderAware, ApplicationEventPublisherAware and MessageSourceAware beans.

应用上下文，高级容器系列，增加了面向框架特性
- 第二条设计主线：以 *ApplicationContext*(应用上下文) 为核心的接口设计，*BeanFactory => ListableBeanFactory =>App]icationContext => WebApplicationContext 或ConfigurableApplicationContext*。
- 常用的应用上下文是 *ConfigurableApplicationContext* 或者 *WebApplicationContext* 的实现。
- *ListableBeanFactory* 指*可列出所有 bean* 而不是当客户端请求时一个一个查找的 Bean-Factorty。 实现类一般都有**预先加载 bean-的definition 的功能**（XML-based factories ）。
	- 容器 Bean 返回值不考虑层级(HierarchicalBeanFactory 接口)的 Bean，*只读取当前容器定义的信息*。
	- 可以通过 BeanFactoryUtils 工具类来获取在父 BeanFactory 中的 Bean。
- *ApplicationContext* 通过继承 *MessageSource（解析Message）*、*ResourceLoader（加载文件资源）*、*ApplicationEventPublisher（注册 Listener）* ，在BeanFactory 的基础上添加了*高级容器的特性*。
	- *支持不同的信息源*。扩展了 MessageSource ，这些信息源的扩展功能可以支持国际化的实现，为开发多语言版本的应用提供服务。
	- *访问资源*。体现在对ResourceLoader和Resource的支持上，可以从不同地方得到 Bean 定义资源。一般来说，具体 AppIicationContext 都继承了 DefaultResourceLoader 的子类。
	- *支持应用事件*。继承了 ApplicationEventPublisher，引人了事件机制。和 Bean 生命周期的结合为Bean的管理提供了便利。
	- 在 ApplicationContext 中提供了其他附加服务，ApplicationContext 与简单的 BeanFactory 相比，使用上是一种*面向框架*的风格。

## 其他
- 这里主要涉及接口关系，***具体的 IoC 容器都是在该接口体系下实现***的。
	- 如 `DefaultListableBeanFactory` ，就是实现了 `ConfigurableBeanFactory` 接口，从而成为一个简单 IoC 容器的实现。
	- 其他 IoC 容器，如 `XmlBeanFactory`，都是*在 DefaultListableBeanFactory 的基础上做扩展*，`ApplicationContext` 也是。
- 这个接口系统是以 `BeanFactory` 和 `ApplicationContext` 上下文为核心的。
	- *BeanFactory 是IoC容器的最基本接口*；
	- `ApplicationContext` 的设计中：
		- 一方面继承了 BeanFactory 接口体系中的 `ListableBeanFactory`、`AutowireCapableBeanFactory`、`HierarchicalBeanFactory` 等接口，*具备了 IoC 容器的基本功能*；
		- 另一方面继承了 MessageSource、ResourceLoader、ApplicationEventPublisher 等接口，*为 ApplicationContext 赋予了更高级特性*。
