# 概念
- *The Java Naming and Directory Interface™ (JNDI)*  提供一些列**命名（[[Naming service]]）和目录（[[Directory service]]）服务的 Java API**；
- 可用于*绑定（binding）、查找（look up）或者检测对* 象上的改动。

# 架构
- 包含 JNDI API 和 SPI；
-  SPI （**参考**：[[API vs. SPI]]）允许透明地插入（类似 plugins）各种 [[Naming service]] 和 [[Directory service]] 服务；
-  Java application 通过 API 使用 naming 和 directory 服务（plugins）；

![[JNDI 结构.png]]

# 使用
- JNDI abstraction 解耦了 database connection 配置和使用。
- 核心功能：
	1. *Name*
	2. *Context*

## Name Interface

```java
Name objectName = new CompositeName("java:comp/env/jdbc");
```

- Name 接口提供了*管理组件名称和 JNDI 名称语法*的能力。
- 字符串 `java:comp/env/jdbc` 的第一个标记 (`java:comp`) 表示 **全局上下文（global context）**;
- 之后的字符串(`env` 和 `jdbc`)表示下一个**子上下文（sub-context）**:

```java
Enumeration<String> elements = objectName.getAll();
while(elements.hasMoreElements()) {
  System.out.println(elements.nextElement());
}
```
输出:
```plaintext
java:comp  // gobal context
env        // sub-context
jdbc       // sub-context
```

 - `/`是 _Name_ 中 sub-contexts 的分隔符. 
 - 增加一个 sub-context:
```java
objectName.add("example");
```

## Context Interface

_Context_ 包含 naming 和 directory 服务的属性。
使用 spring 的工具类构建 _Context_:

```java
SimpleNamingContextBuilder builder = new SimpleNamingContextBuilder(); 
builder.activate();
```

Spring 的 _SimpleNamingContextBuilder_ 创建一个 JNDI provider 且用[_NamingManager_](https://docs.oracle.com/en/java/javase/11/docs/api/java.naming/javax/naming/spi/NamingManager.html)激活 builder。

最后，通过 _JndiTemplate_ 得到  _InitialContext_.

```java
JndiTemplate jndiTemplate = new JndiTemplate();
ctx = (InitialContext) jndiTemplate.getContext();
```


##  JNDI 对象绑定与查找

 _Name_ and _Context_ 的使用：
使用 JNDI 存储一个 JDBC _DataSource_:

```java
ds = new DriverManagerDataSource("jdbc:h2:mem:mydb");
```

### 绑定 JNDI 对象

获得 context 之后, 将对象绑定到context:

```java
ctx.bind("java:comp/env/jdbc/datasource", ds);
```

In general, services should store an object reference, serialized data, or attributes in a directory context. It all depends on the needs of the application.
通常，服务应该在 directory context 中存储对象引用、序列化数据或属性。不过这取决于应用程序的需要。

### 查找 JNDI 对象

查找 _DataSource_:

```java
DataSource ds = (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
```

## Role of JNDI in Modern Application Architecture

虽然JNDI在**轻量级、容器化**的Java应用程序(如Spring Boot)中扮演的角色较小，但还有其他用途。
如：
1. [JDBC](https://www.baeldung.com/java-jdbc)；
2. [EJB](https://www.baeldung.com/ejb-intro)；
3. [JMS](https://www.baeldung.com/spring-jms). 


## 参考文献
1. [Naming Concepts ](https://docs.oracle.com/javase/tutorial/jndi/concepts/index.html)
2. [Naming Package](https://docs.oracle.com/javase/tutorial/jndi/overview/naming.html)
3. [Directory Concepts](https://docs.oracle.com/javase/tutorial/jndi/concepts/directory.html)
4. [Directory and LDAP Packages](https://docs.oracle.com/javase/tutorial/jndi/overview/dir.html)
5. [[API vs. SPI]]
6. [JNDI - Baeldung](https://www.baeldung.com/jndi)
7. [[JMS]]