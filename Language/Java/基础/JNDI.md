# 概念
The Java Naming and Directory Interface™ (JNDI) **provides consistent use of naming and/or directory services as a Java API.** This interface can be used for binding objects, looking up or querying objects, as well as detecting changes on the same objects.
The Java Naming and Directory Interface™ (JNDI)  提供一些列*命名（naming）和目录（directory）服务* 的 Java *API*，可用于绑定对象，查找对象或者检测对象上的改动。

# 架构
The JNDI architecture consists of an API and a service provider interface (SPI). 
Java applications use the JNDI API to access a variety of naming and directory services.
The SPI enables a variety of naming and directory services to be plugged in transparently, thereby allowing the Java application using the JNDI API to access their services. 

![[JNDI 结构.png]]

# 使用
a database connection service calls for specific properties and exception handling.However, JNDI’s abstraction decouples the connection configuration from the application.
Let's explore _Name_ and _Context_, which contain the core functionality of JNDI.

### _Name_ Interface

```java
Name objectName = new CompositeName("java:comp/env/jdbc");
```

The _Name_ interface provides the ability to manage the component names and syntax for JNDI names. 
The first token of the string represents the *global context*, after that each string added represents the next sub-context:

```java
Enumeration<String> elements = objectName.getAll();
while(elements.hasMoreElements()) {
  System.out.println(elements.nextElement());
}
```

Our output looks like:

```plaintext
java:comp
env
jdbc
```

As we can see, `/` is the delimiter for _Name_ sub-contexts. Now, let's add a sub-context:

```java
objectName.add("example");
```

Then we test our addition:

```java
assertEquals("example", objectName.get(objectName.size() - 1));
```

###  _Context_ Interface

_Context_ contains the properties for the naming and directory service_._ Here, let's use some helper code from Spring for convenience to build a _Context_:

```java
SimpleNamingContextBuilder builder = new SimpleNamingContextBuilder(); 
builder.activate();
```

Spring's _SimpleNamingContextBuilder_ creates a JNDI provider and then activates the builder with the [_NamingManager_](https://docs.oracle.com/en/java/javase/11/docs/api/java.naming/javax/naming/spi/NamingManager.html):

```java
JndiTemplate jndiTemplate = new JndiTemplate();
ctx = (InitialContext) jndiTemplate.getContext();
```

Finally, _JndiTemplate_ helps us access the _InitialContext_.

##  JNDI Object Binding and Lookup

Now that we've seen how to use _Name_ and _Context_, let’s use JNDI to store a JDBC _DataSource_:

```java
ds = new DriverManagerDataSource("jdbc:h2:mem:mydb");
```

### Binding JNDI Objects

As we have a context, let's bind the object to it:

```java
ctx.bind("java:comp/env/jdbc/datasource", ds);
```

In general, services should store an object reference, serialized data, or attributes in a directory context. It all depends on the needs of the application.

Note that using JNDI this way is less common. Typically, JNDI interfaces with data that is managed outside the application runtime.

However, if the application can already create or find its _DataSource_, it might be easier to wire that using Spring. In contrast, if something outside of our application bound objects in JNDI, then the application could consume them.

### Looking Up JNDI Objects

Let's look up our _DataSource_:

```java
DataSource ds = (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
```

And then let's test to ensure that _DataSource_ is as expected:

```java
assertNotNull(ds.getConnection());
```


## 参考文献
1. [Naming Concepts ](https://docs.oracle.com/javase/tutorial/jndi/concepts/index.html)
2. [Naming Package](https://docs.oracle.com/javase/tutorial/jndi/overview/naming.html)
3. [Directory Concepts](https://docs.oracle.com/javase/tutorial/jndi/concepts/directory.html)
4. [Directory and LDAP Packages](https://docs.oracle.com/javase/tutorial/jndi/overview/dir.html)
5. [[API vs. SPI]]
6. [JNDI - Baeldung](https://www.baeldung.com/jndi)