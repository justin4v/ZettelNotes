# 概念
The Java Naming and Directory Interface™ (JNDI) **provides consistent use of naming and/or directory services as a Java API.** This interface can be used for binding objects, looking up or querying objects, as well as detecting changes on the same objects.
The Java Naming and Directory Interface™ (JNDI)  提供一些列*命名（naming）和目录（directory）服务* 的 Java *API*，可用于绑定对象，查找对象或者检测对象上的改动。


# 架构
The JNDI architecture consists of an API and a service provider interface (SPI). 
Java applications use the JNDI API to access a variety of naming and directory services.
The SPI enables a variety of naming and directory services to be plugged in transparently, thereby allowing the Java application using the JNDI API to access their services. 

![[JNDI 结构.png]]

# 使用



## 参考
[[API vs. SPI]]
[JNDI - Baeldung](https://www.baeldung.com/jndi)