#Spring  #Todo 

# 简介
1.  BeanFactory的直接子接口，拥有*自动装配*的能力
2.  一般不会直接使用 `AutowireCapableBeanFactory`，因此在设计上 `ApplicationContext` 们并没有直接实现该接口。却提供了该问该接口的能力：通过`ApplicationContext#getAutowireCapableBeanFactory` 方法可以得到实例对象。
3.  作用在于**整合其它框架**：能让 Spring 管理的 Bean 装配和填充那些不被 Spring 托管的Bean(**wire and populate existing bean instances that Spring does not control the lifecycle of**)

# 参考
1. [AutowireCapableBeanFactory探密(1)——为第三方框架赋能 ](https://www.jianshu.com/p/f9718de489f0)