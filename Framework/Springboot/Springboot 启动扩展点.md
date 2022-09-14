#Springboot 

## 1.背景

Spring的核心思想就是容器，当容器refresh的时候，外部看上去风平浪静，其实内部则是一片惊涛骇浪，汪洋一片。Springboot更是封装了Spring，遵循约定大于配置，加上自动装配的机制。很多时候我们只要引用了一个依赖，几乎是零配置就能完成一个功能的装配。

我非常喜欢这种自动装配的机制，所以在自己开发中间件和公共依赖工具的时候也会用到这个特性。让使用者以最小的代价接入。想要把自动装配玩的转，就必须要了解spring对于bean的构造生命周期以及各个扩展接口。当然了解了bean的各个生命周期也能促进我们加深对spring的理解

## 2.可扩展的接口启动调用顺序图

![[SpringBoot可扩展点整理.png]]

## 3.ApplicationContextInitializer

> org.springframework.context.ApplicationContextInitializer

这是整个spring容器在刷新之前初始化`ConfigurableApplicationContext`的回调接口，简单来说，就是在容器刷新之前调用此类的`initialize`方法。这个点允许被用户自己扩展。用户可以在整个spring容器还没被初始化之前做一些事情。

可以想到的场景可能为，在最开始激活一些配置，或者利用这时候class还没被类加载器加载的时机，进行动态字节码注入等操作。

扩展方式为：

`public class TestApplicationContextInitializer implements ApplicationContextInitializer {       @Override       public void initialize(ConfigurableApplicationContext applicationContext) {           System.out.println("[ApplicationContextInitializer]");       }   }   `

因为这时候spring容器还没被初始化，所以想要自己的扩展的生效，有以下三种方式：

-   在启动类中用`springApplication.addInitializers(new TestApplicationContextInitializer())`语句加入
    
-   配置文件配置`context.initializer.classes=com.example.demo.TestApplicationContextInitializer`
    
-   Spring SPI扩展，在spring.factories中加入`org.springframework.context.ApplicationContextInitializer=com.example.demo.TestApplicationContextInitializer`
    

## **4.BeanDefinitionRegistryPostProcessor**

> org.springframework.beans.factory.support.BeanDefinitionRegistryPostProcessor

这个接口在读取项目中的`beanDefinition`之后执行，提供一个补充的扩展点

使用场景：你可以在这里动态注册自己的`beanDefinition`，可以加载classpath之外的bean

扩展方式为:

`public class TestBeanDefinitionRegistryPostProcessor implements BeanDefinitionRegistryPostProcessor {       @Override       public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {           System.out.println("[BeanDefinitionRegistryPostProcessor] postProcessBeanDefinitionRegistry");       }          @Override       public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {           System.out.println("[BeanDefinitionRegistryPostProcessor] postProcessBeanFactory");       }   }   `

## **5.BeanFactoryPostProcessor**

> org.springframework.beans.factory.config.BeanFactoryPostProcessor

这个接口是`beanFactory`的扩展接口，调用时机在spring在读取`beanDefinition`信息之后，实例化bean之前。

在这个时机，用户可以通过实现这个扩展接口来自行处理一些东西，比如修改已经注册的`beanDefinition`的元信息。

扩展方式为：

`public class TestBeanFactoryPostProcessor implements BeanFactoryPostProcessor {       @Override       public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {           System.out.println("[BeanFactoryPostProcessor]");       }   }   `

## **6.InstantiationAwareBeanPostProcessor**

> org.springframework.beans.factory.config.InstantiationAwareBeanPostProcessor

该接口继承了`BeanPostProcess`接口，区别如下：

**`BeanPostProcess`接口只在bean的初始化阶段进行扩展（注入spring上下文前后），而`InstantiationAwareBeanPostProcessor`接口在此基础上增加了3个方法，把可扩展的范围增加了实例化阶段和属性注入阶段。**

该类主要的扩展点有以下5个方法，主要在bean生命周期的两大阶段：**实例化阶段** 和**初始化阶段** ，下面一起进行说明，按调用顺序为：

-   `postProcessBeforeInstantiation`：实例化bean之前，相当于new这个bean之前
    
-   `postProcessAfterInstantiation`：实例化bean之后，相当于new这个bean之后
    
-   `postProcessPropertyValues`：bean已经实例化完成，在属性注入时阶段触发，`@Autowired`,`@Resource`等注解原理基于此方法实现
    
-   `postProcessBeforeInitialization`：初始化bean之前，相当于把bean注入spring上下文之前
    
-   `postProcessAfterInitialization`：初始化bean之后，相当于把bean注入spring上下文之后
    

使用场景：这个扩展点非常有用 ，无论是写中间件和业务中，都能利用这个特性。比如对实现了某一类接口的bean在各个生命期间进行收集，或者对某个类型的bean进行统一的设值等等。

扩展方式为：

`public class TestInstantiationAwareBeanPostProcessor implements InstantiationAwareBeanPostProcessor {          @Override       public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {           System.out.println("[TestInstantiationAwareBeanPostProcessor] before initialization " + beanName);           return bean;       }          @Override       public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {           System.out.println("[TestInstantiationAwareBeanPostProcessor] after initialization " + beanName);           return bean;       }          @Override       public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {           System.out.println("[TestInstantiationAwareBeanPostProcessor] before instantiation " + beanName);           return null;       }          @Override       public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {           System.out.println("[TestInstantiationAwareBeanPostProcessor] after instantiation " + beanName);           return true;       }          @Override       public PropertyValues postProcessPropertyValues(PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) throws BeansException {           System.out.println("[TestInstantiationAwareBeanPostProcessor] postProcessPropertyValues " + beanName);           return pvs;       }   `

## **7.SmartInstantiationAwareBeanPostProcessor**

> org.springframework.beans.factory.config.SmartInstantiationAwareBeanPostProcessor

该扩展接口有3个触发点方法：

-   `predictBeanType`：该触发点发生在`postProcessBeforeInstantiation`之前(在图上并没有标明，因为一般不太需要扩展这个点)，这个方法用于预测Bean的类型，返回第一个预测成功的Class类型，如果不能预测返回null；当你调用`BeanFactory.getType(name)`时当通过bean的名字无法得到bean类型信息时就调用该回调方法来决定类型信息。
    
-   `determineCandidateConstructors`：该触发点发生在`postProcessBeforeInstantiation`之后，用于确定该bean的构造函数之用，返回的是该bean的所有构造函数列表。用户可以扩展这个点，来自定义选择相应的构造器来实例化这个bean。
    
-   `getEarlyBeanReference`：该触发点发生在`postProcessAfterInstantiation`之后，当有循环依赖的场景，当bean实例化好之后，为了防止有循环依赖，会提前暴露回调方法，用于bean实例化的后置处理。这个方法就是在提前暴露的回调方法中触发。
    

扩展方式为：

`public class TestSmartInstantiationAwareBeanPostProcessor implements SmartInstantiationAwareBeanPostProcessor {          @Override       public Class<?> predictBeanType(Class<?> beanClass, String beanName) throws BeansException {           System.out.println("[TestSmartInstantiationAwareBeanPostProcessor] predictBeanType " + beanName);           return beanClass;       }          @Override       public Constructor<?>[] determineCandidateConstructors(Class<?> beanClass, String beanName) throws BeansException {           System.out.println("[TestSmartInstantiationAwareBeanPostProcessor] determineCandidateConstructors " + beanName);           return null;       }          @Override       public Object getEarlyBeanReference(Object bean, String beanName) throws BeansException {           System.out.println("[TestSmartInstantiationAwareBeanPostProcessor] getEarlyBeanReference " + beanName);           return bean;       }   }   `

## **8.BeanFactoryAware**

> org.springframework.beans.factory.BeanFactoryAware

这个类只有一个触发点，发生在bean的实例化之后，注入属性之前，也就是Setter之前。这个类的扩展点方法为`setBeanFactory`，可以拿到`BeanFactory`这个属性。

使用场景为，你可以在bean实例化之后，但还未初始化之前，拿到 `BeanFactory`，在这个时候，可以对每个bean作特殊化的定制。也或者可以把`BeanFactory`拿到进行缓存，日后使用。

扩展方式为：

`public class TestBeanFactoryAware implements BeanFactoryAware {       @Override       public void setBeanFactory(BeanFactory beanFactory) throws BeansException {           System.out.println("[TestBeanFactoryAware] " + beanFactory.getBean(TestBeanFactoryAware.class).getClass().getSimpleName());       }   }   `

## **9.ApplicationContextAwareProcessor**

> org.springframework.context.support.ApplicationContextAwareProcessor

该类本身并没有扩展点，但是该类内部却有6个扩展点可供实现 ，这些类触发的时机在bean实例化之后，初始化之前

# 参考
1. [掌握这些 Spring Boot 启动扩展点，已经超过 90% 的人了！ (qq.com)](https://mp.weixin.qq.com/s?__biz=MzAxODcyNjEzNQ==&mid=2247569739&idx=1&sn=893aff64b2cace9d1d98b0e2f4d1d52d&chksm=9bd27cd3aca5f5c5579dbca06e94fbc13f68150bbd9061ec3a3a39af8f0b3eeb3c5c25e57a08&scene=21#wechat_redirect)