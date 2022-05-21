#Spring #Springboot #Todo #Bean


# 1. 概述

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
  
就从 `getBean` 方法作为入口，去理解 Spring 加载的流程是怎样的，以及内部对创建信息、作用域、依赖关系等等的处理细节。

# 2. 总体流程
![[Bean加载过程.png]]

上面是跟踪了 getBean 的调用链创建的流程图，为了能够很好地理解 Bean 加载流程，省略一些异常、日志和分支处理和一些特殊条件的判断。

从上面的流程图中，可以看到一个 Bean 加载会经历这么几个阶段（用绿色标记）：

-   **获取 BeanName**，对传入的 name 进行解析，转化为可以从 Map 中获取到 BeanDefinition 的 bean name。
-   **合并 Bean 定义**，对父类的定义进行合并和覆盖，如果父类还有父类，会进行递归合并，以获取完整的 Bean 定义信息。
-   **实例化**，使用构造或者工厂方法创建 Bean 实例。
-   **属性填充**，寻找并且注入依赖，依赖的 Bean 还会递归调用 `getBean` 方法获取。
-   **初始化**，调用自定义的初始化方法。
-   **获取最终的 Bean**，如果是 FactoryBean 需要调用 getObject 方法，如果需要类型转换调用 TypeConverter 进行转化。

整个流程最为复杂的是对循环依赖的解决方案，后续会进行重点分析。


# 3. 细节分析

## 3.1. 转化 BeanName

而在我们解析完配置后创建的 Map，使用的是 beanName 作为 key。见 `DefaultListableBeanFactory`：

```java
/** Map of bean definition objects, keyed by bean name */
private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<String, BeanDefinition>(256);
```
`BeanFactory.getBean` 中传入的 name，有可能是这几种情况：

-   **bean name**，可以直接获取到定义 BeanDefinition。
-   **alias name**，别名，需要转化。
-   **factorybean name**, **带 `&` 前缀**，通过它获取 BeanDefinition 的时候需要去除 & 前缀。

为了能够获取到正确的 BeanDefinition，需要先对 name 做一个转换，得到 beanName。

![[name转beanName.png]]
见 `AbstractBeanFactory.doGetBean`：

```java
protected <T> T doGetBean ... {
    ...
    
    // 转化工作 
    final String beanName = transformedBeanName(name);
    ...
}
```
如果是 **alias name**，在解析阶段，alias name 和 bean name 的映射关系被注册到 SimpleAliasRegistry 中。从该注册器中取到 beanName。见 `SimpleAliasRegistry.canonicalName`：
```java
public String canonicalName(String name) {
    ...
    resolvedName = this.aliasMap.get(canonicalName);
    ...
}
```

如果是 **factorybean name**，表示这是个工厂 bean，有携带前缀修饰符 `&` 的，直接把前缀去掉。见 `BeanFactoryUtils.transformedBeanName` :
```java
public static String transformedBeanName(String name) {
    Assert.notNull(name, "'name' must not be null");
    String beanName = name;
    while (beanName.startsWith(BeanFactory.FACTORY_BEAN_PREFIX)) {
        beanName = beanName.substring(BeanFactory.FACTORY_BEAN_PREFIX.length());
    }
    return beanName;
}
```

## 3.2. 合并 RootBeanDefinition

我们从配置文件读取到的 BeanDefinition 是 **GenericBeanDefinition**。它的记录了一些当前类声明的属性或构造参数，但是对于父类只用了一个 `parentName` 来记录。

```java
public class GenericBeanDefinition extends AbstractBeanDefinition {
    ...
    private String parentName;
    ...
}
```

接下来会发现一个问题，在后续实例化 Bean 的时候，使用的 BeanDefinition 是 **RootBeanDefinition** 类型而非 **GenericBeanDefinition**。这是为什么？

答案很明显，GenericBeanDefinition 在有继承关系的情况下，定义的信息不足：

-   如果不存在继承关系，GenericBeanDefinition 存储的信息是完整的，可以直接转化为 RootBeanDefinition。
-   如果存在继承关系，GenericBeanDefinition 存储的是 **增量信息** 而不是 **全量信息**。

**为了能够正确初始化对象，需要完整的信息才行**。需要递归 **合并父类的定义**：
![[合并beanDefinition.png]]

见 `AbstractBeanFactory.doGetBean` ：

```java
protected <T> T doGetBean ... {
    ...
    
    // 合并父类定义
    final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
        
    ...
        
    // 使用合并后的定义进行实例化
    bean = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
        
    ...
}
```

在判断 `parentName` 存在的情况下，说明存在父类定义，启动合并。如果父类还有父类怎么办？递归调用，继续合并。

见`AbstractBeanFactory.getMergedBeanDefinition` 方法：

```java
    protected RootBeanDefinition getMergedBeanDefinition(
            String beanName, BeanDefinition bd, BeanDefinition containingBd)
            throws BeanDefinitionStoreException {

        ...
        
        String parentBeanName = transformedBeanName(bd.getParentName());

        ...
        
        // 递归调用，继续合并父类定义
        pbd = getMergedBeanDefinition(parentBeanName);
        
        ...

        // 使用合并后的完整定义，创建 RootBeanDefinition
        mbd = new RootBeanDefinition(pbd);
        
        // 使用当前定义，对 RootBeanDefinition 进行覆盖
        mbd.overrideFrom(bd);

        ...
        return mbd;
    
    }
```

每次合并完父类定义后，都会调用 `RootBeanDefinition.overrideFrom` 对父类的定义进行覆盖，获取到当前类能够正确实例化的 **全量信息**。

### 3.3. 处理循环依赖

什么是循环依赖？

举个例子，这里有三个类 A、B、C，然后 A 关联 B，B 关联 C，C 又关联 A，这就形成了一个循环依赖。如果是方法调用是不算循环依赖的，循环依赖必须要持有引用。

  
  
作者：RunAlgorithm  
链接：file://isi-sh/InternetDownload/Downloads/%E5%9B%BE%E6%96%87%E5%B9%B6%E8%8C%82%EF%BC%8C%E6%8F%AD%E7%A7%98%20Spring%20%E7%9A%84%20Bean%20%E7%9A%84%E5%8A%A0%E8%BD%BD%E8%BF%87%E7%A8%8B%20-%20%E7%AE%80%E4%B9%A6.html  
来源：简书  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。


# 参考
1. 图文并茂，揭秘 Spring 的 Bean 的加载过程（简书）
2. [spring bean的加载过程](https://calmtime.github.io/2020/05/02/spring-bean/)
