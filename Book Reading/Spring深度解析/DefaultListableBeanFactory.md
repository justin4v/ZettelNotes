#Spring #SourceCode #Todo 

# 结构
*Spring 中默认的 IoC 完整功能实现*，继承关系如下：
![[DefaultListableBeanFactory继承关系.png|700]]


# 意图功能简介
-   AliasRegistry：定义对 Bean *alias* 的**增删查操作规范**（规约层面要求）。
-   SimpleAliasRegistry：主要使用map作为alias的缓存，并对接口AliasRegistry进行实现。
-   SingletonBeanRegistry：定义对单例的注册及获取。
-   BeanFactory：定义获取bean及bean的各种属性。
-   DefaultSingletonBeanRegistry：对接口SingletonBeanRegistry各函数的实现。
-   HierarchicalBeanFactory：继承BeanFactory，也就是在BeanFactory定义的功能的基础上增加了对parentFactory的支持。
-   BeanDefinitionRegistry：定义对BeanDefinition的各种增删改操作。
-   FactoryBeanRegistrySupport：在DefaultSingletonBeanRegistry基础上增加了对FactoryBean的特殊处理功能。
-   ConfigurableBeanFactory：提供配置Factory的各种方法。
-   ListableBeanFactory：根据各种条件获取bean的配置清单。
-   AbstractBeanFactory：综合FactoryBeanRegistrySupport和ConfigurableBeanFactory的功能。
-   AutowireCapableBeanFactory：提供创建bean、自动注入、初始化以及应用bean的后处理器。
-   AbstractAutowireCapableBeanFactory：综合AbstractBeanFactory并对接口Autowire Capable BeanFactory进行实现。