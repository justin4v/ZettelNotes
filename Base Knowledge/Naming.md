# Naming Concepts

A **fundamental facility** in any computing system is the **_naming service_**

naming service:
- **名字和对象关联;**
- **根据名称找到对象. **


When using almost any computer program or system, you are always naming one object or another. 
例如：
- 使用电子邮件的时候，需要填写收件人. 
- 要访问计算机中的文件，必须提供它的名称. 

![[Naming System.png]]

命名服务的**主要功能是将对人类友好的名称映射到对象**，如
- 地址；
- 标识符；
- 计算机程序通常使用的对象。

## DNS
the [Internet Domain Name System (DNS)](http://www.ietf.org/rfc/rfc1034.txt) 将机器名 映射到 IP Addresses :

www.example.com ==> 192.0.2.5

## 文件系统
文件系统将**文件名映射到一个文件引用**，程序可以使用该文件引用来访问文件的内容。

c:\bin\autoexec.bat ==> File Reference

# Names

To look up an object in a naming system, you supply it the _name_ of the object.
要在命名系统中查找一个对象，您需要向它提供对象的 _name_。

**naming system （命名系统）定义了 name 必须遵循的语法**。这种语法被称为命名系统 的** _命名约定_（_naming convention_）**

名称是由组件和组件分割符组成的。

![[命名服务示例.png]]

## Unix 文件系统
UNIX文件系统的命名约定是：
- 根据文件相对于文件系统根路径来命名文件；
- 路径中的每个组件使用正斜杠字符("/")从左到右分开。

The UNIX _pathname_, /usr/hello, for example, names a file hello in the file directory usr, which is located in the root of the file system.

## DNS 
DNS命名约定要求：
- DNS名称中的组件按从右到左的顺序排列，并由点字符(".")分隔。

Thus the DNS name sales.Wiz.COM names a DNS entry with the name sales, relative to the DNS entry Wiz.COM. The DNS entry Wiz.COM, in turn, names an entry with the name Wiz in the COM entry.



## LDAP
[轻量级目录访问协议(LDAP)](http://www.ietf.org/rfc/rfc2251.txt)命名约定：
- 按从右到左的顺序排列组件，由逗号("，")分隔；
- 名称的每个组件必须是一个名称/值对，名称和值由一个等号字符("=")分隔。

LDAP名称 `cn=Rosanna Lee, o=Sun, c=US`命名了一个LDAP条目cn=Rosanna Lee，相对于条目o=Sun，而该条目又相对于c=US。

# Bindings

The association of a name with an object is called a _binding_. A file name is _bound_ to a file.

The DNS contains bindings that map machine names to IP addresses. An LDAP name is bound to an LDAP entry.

# References and Addresses

Depending on the naming service, some objects cannot be stored directly by the naming service; that is, a copy of the object cannot be placed inside the naming service. Instead, they must be stored by reference; that is, a _pointer_ or _reference_ to the object is placed inside the naming service.

A reference represents information about how to access an object. Typically, it is a compact representation that can be used to communicate with the object, while the object itself might contain more state information. Using the reference, you can contact the object and obtain more information about the object.

For example, an airplane object might contain a list of the airplane's passengers and crew, its flight plan, and fuel and instrument status, and its flight number and departure time. 

By contrast, an airplane object reference might contain only its flight number and departure time. The reference is a much more compact representation of information about the airplane object and can be used to obtain additional information. 

A file object, for example, is accessed using a _file reference_. A printer object, for example, might contain the state of the printer, such as its current queue and the amount of paper in the paper tray. A printer object reference, on the other hand, might contain only information on how to reach the printer, such as its print server name and printing protocol.

Although in general a reference can contain any arbitrary information, it is useful to refer to its contents as _addresses_ (or communication end points): specific information about how to access the object.

For simplicity, this tutorial uses "object" to refer to both objects and object references when a distinction between the two is not required.

# Context

A _context_ is **a set of name-to-object bindings**. Every context has an associated naming convention. 
_context_ 是一个 **name-to-object 绑定的集合**。每个 _context_ 都有一个相关的命名约定。

A context always provides a lookup (_resolution_ 解析) operation that returns the object, it typically also provides operations such as those for binding names, unbinding names, and listing bound names. A name in one context object can be bound to another context object (called a _subcontext_) that has the same naming convention.

![[context实例.png]]

A file directory, such as /usr, in the UNIX file system represents a context. A file directory named relative to another file directory represents a subcontext (UNIX users refer to this as a _subdirectory_). 

That is, in a file directory /usr/bin, the directory bin is a subcontext of usr. A DNS domain, such as COM, represents a context. A DNS domain named relative to another DNS domain represents a subcontext. For the DNS domain Sun.COM, the DNS domain Sun is a subcontext of COM.

Finally, an LDAP entry, such as c=us, represents a context. An LDAP entry named relative to another LDAP entry represents a subcontext. For the LDAP entry o=sun,c=us, the entry o=sun is a subcontext of c=us.

# Naming Systems and Namespaces

A _naming system_ is a connected set of contexts of the same type (they have the same naming convention) and provides a common set of operations.

A system that implements the DNS is a naming system. A system that communicates using the LDAP is a naming system.

A naming system provides a _naming service_ to its customers for performing naming-related operations. A naming service is accessed through its own interface. The DNS offers a naming service that maps machine names to IP addresses. LDAP offers a naming service that maps LDAP names to LDAP entries. A file system offers a naming service that maps filenames to files and directories.

A _namespace_ is the set of all possible names in a naming system. The UNIX file system has a namespace consisting of all of the names of files and directories in that file system. The DNS namespace contains names of DNS domains and entries. The LDAP namespace contains names of LDAP entries.