#Spring 

# 目的
- Aware 意为感知捕获。
	- 单纯的 bean（未实现Aware系列接口）无法感知 Spring 容器；
	- 实现了 Aware 接口 bean 可访问 Spring 容器，自定义某些属性。
- Aware 接口增强了 bean 的功能，但也会造成对Spring框架的绑定，增大了与Spring框架的耦合度

# 共性
1.  都以“Aware”结尾
2.  都是Aware接口的子接口，即都继承了Aware接口
3.  接口内均定义了一个set方法

## 部分实现
### ApplicationContextAware

```javas
void setApplicationContext(ApplicationContext applicationContext)
```


### BeanClassLoaderAware

```java
void setBeanClassLoader(ClassLoader classLoader);
```

### BeanFactoryAware

```java
void setBeanFactory(BeanFactory beanFactory)
```

### BeanNameAware

```java
void setBeanName(String name);
```
- 每个子接口都定义了set方法。而方法中的形参是接口Aware前面的内容，也就是当前Bean需要感知的内容。所以我们需要在Bean中声明相关的成员变量来接收。


# 参考
1. [Spring之Aware接口介绍](https://cloud.tencent.com/developer/article/1409274)