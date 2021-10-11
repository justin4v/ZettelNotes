# 概念
The Java Naming and Directory Interface™ (JNDI) is an *application programming interface (API)* that provides [naming](https://docs.oracle.com/javase/tutorial/jndi/overview/naming.html) and [directory](https://docs.oracle.com/javase/tutorial/jndi/overview/dir.html) functionality to applications written using the Java™ programming language.



# 架构
The JNDI architecture consists of an API and a service provider interface (SPI). 
Java applications use the JNDI API to access a variety of naming and directory services.
The SPI enables a variety of naming and directory services to be plugged in transparently, thereby allowing the Java application using the JNDI API to access their services. 

![[JNDI 结构.png]]


## 参考
[[API vs. SPI]]