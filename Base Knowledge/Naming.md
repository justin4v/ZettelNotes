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

LDAP名称 `cn=Rosanna Lee, o=Sun, c=US`命名了一个LDAP条目 `cn=Rosanna Lee`，改条目相对于条目 `o=Sun`，而`o=Sun`条目又相对于 `c=US`。

# Bindings

**名称与对象相关联称为绑定( _binding_)** 。
如：
1. 文件名绑定到文件。
2. DNS包含将机器名称到 IP 地址的绑定。
3. LDAP名称绑定到LDAP条目。

# References and Addresses
命名服务中，有些对象不能直接由命名服务存储;
也就是说，对象的副本不能放在命名服务中。
相反，它们必须通过引用来存储;也就是说，对象的_pointer_或_reference_被放置在命名服务中。

引用表示关于如何访问对象的信息。
通常，它是一种紧凑的表示，可用于与对象通信，而对象本身可能包含更多的状态信息。
使用引用，您可以联系对象并获得关于对象的更多信息。

如：
1. 飞机：
	- 飞机物体可能包含飞机乘客和机组人员、飞行计划、燃料和仪器状态、航班号和起飞时间的列表；
	- 飞机对象引用可能只包含它的航班号和起飞时间。
2. 使用_file引用来访问文件对象。
3. 打印机
	- 打印机对象可能包含打印机的状态，比如它的当前队列和纸盒中的纸量。
	- 打印机对象引用可能只包含如何访问打印机的信息，比如使用打印服务器名称和打印协议。

尽管通常引用可以包含任何信息，但将其内容引用为 _addresses_（如何访问对象） 是很有用的。

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