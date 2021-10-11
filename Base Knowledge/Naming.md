# 1.Naming Concepts
**在任何计算系统中，一个基本的设施就是命名服务（_naming service_）**

naming service:
- **名字和对象关联;**
- **根据名称找到对象. **

当使用几乎任何计算机程序或系统时，你总是在命名一个对象或另一个对象。
例如：
- 使用电子邮件的时候，需要填写收件人. 
- 要访问计算机中的文件，必须提供它的名称. 

![[Naming System.png]]

命名服务的**主要功能是将对人类友好的名称映射到对象**，如
- 地址；
- 标识符；
- 计算机程序通常使用的对象。

 **DNS**
the [Internet Domain Name System (DNS)](http://www.ietf.org/rfc/rfc1034.txt) 将机器名 映射到 IP Addresses :

www.example.com ==> 192.0.2.5

**文件系统**
文件系统将**文件名映射到一个文件引用**，程序可以使用该文件引用来访问文件的内容。

c:\bin\autoexec.bat ==> File Reference

# 2 Names
要在命名系统中查找一个对象，您需要向它提供对象的 _name_。

**naming system （命名系统）定义了 name 必须遵循的语法**。这种语法被称为命名系统 的** _命名约定_（_naming convention_）**

名称是由组件和组件分割符组成的。

![[命名服务示例.png]]

## 2.1 Unix 文件系统
UNIX文件系统的命名约定是：
- 根据文件相对于文件系统根路径来命名文件；
- 路径中的每个组件使用正斜杠字符("/")从左到右分开。

例如，UNIX _pathname_， /usr/hello 将文件目录usr 中的一个文件命名为 hello，该文件目录 user 位于文件系统的根目录中。

## 2.2 DNS 
DNS命名约定要求：
- DNS名称中的组件按从右到左的顺序排列，并由点字符(".")分隔。


因此，DNS 目 `sales.Wiz.COM`名称为 sales，属于条目中`Wiz.COM`的条目。而DNS条目`Wiz.COM`则使用名称Wiz来命名 `COM`中的一个条目。

## 2.3 LDAP
[轻量级目录访问协议(LDAP)](http://www.ietf.org/rfc/rfc2251.txt)命名约定：
- 按从右到左的顺序排列组件，由逗号("，")分隔；
- 名称的每个组件必须是一个名称/值对，名称和值由一个等号字符("=")分隔。

LDAP名称 `cn=Rosanna Lee, o=Sun, c=US`命名了一个LDAP条目 `cn=Rosanna Lee`，该条目相对于条目 `o=Sun`，而`o=Sun`条目又相对于 `c=US`。

# 3 Bindings

**名称与对象相关联称为绑定( _binding_)** 。
如：
1. 文件名绑定到文件。
2. DNS包含将机器名称到 IP 地址的绑定。
3. LDAP名称绑定到LDAP条目。

# 4 References and Addresses
## 4.1 背景
 - 命名服务中，有些对象**不能直接由命名服务存储**，也就是说，对象的副本不能放在命名服务中。
 - 它们**必须通过引用来存储**，也就是说，对象的 _pointer_ 或 _reference_ 被放置在命名服务中。

## 4.2 概念
**引用（References）表示如何访问对象的信息。**
通常，它是一种**紧凑**的表示，可用于与对象通信，而对象本身可能包含更多的状态信息。

如：
1. 飞机：
	- 飞机物体可能包含飞机乘客和机组人员、飞行计划、燃料和仪器状态、航班号和起飞时间的列表；
	- 飞机对象引用可能只包含它的航班号和起飞时间。
2. 使用_file引用来访问文件对象。
3. 打印机
	- 打印机对象可能包含打印机的状态，比如它的当前队列和纸盒中的纸量。
	- 打印机对象引用可能只包含如何访问打印机的信息，比如使用打印服务器名称和打印协议。

尽管通常引用可以包含任何信息，但将其内容引用为 _addresses_（如何访问对象） 是很有用的。

# 5 Context
## 5.1 概念
_context_ 是一个 **name-to-object 绑定的集合**。每个 _context_ 都有一个相关的**命名约定**。

## 5.2 特点
1. context **总是提供了一个查找( _resolution_ 解析)操作，返回对象**。通常还提供 *绑定名称、解绑、列出所有已绑定名称* 等操作。
2. 一个 context 中的名称**可以绑定到另一个具有相同命名约定**（naming convention）的 context (称为_subcontext_)。

![[context实例.png]]


## 5.3 示例
### 5.3.1 Unix 文件系统
UNIX文件系统中的文件目录(例如/usr)表示一个 context。相对于该文件目录的文件目录为 sub-context (UNIX称为 _subdirectory_)。
在文件目录/usr/bin中：目录 bin 是 usr 的 sub-context。

### 5.3.2 DNS
一个 DNS 域，例如 COM，代表一个 context。
相对于该域命名的 DNS 域是一个 sub-context 。对于DNS域 Sun.COM：Sun 是 COM 的子上下文。

### 5.3.3 LDAP
一个LDAP条目(例如c=us)表示一个 context。相对于一个LDAP条目的LDAP条目表示该条目的 sub-context 。、
对于LDAP条目`o=sun,c=us`：条目 `o=sun` 是 `c=us` 的 sub-context。

# 6 Naming Systems and Namespaces

## 6.1 Naming Systems
### 概念
命名系统是**同一类型（相同的命名约定） context 的相互关联的集合**，并提供了一组**公共操作**。
例如：
- 实现 DNS 的系统称为命名系统；
- 实现 LDAP 进行通信的系统称为命名系统。

### 特点
*命名系统为客户提供命名服务，用于执行命名相关的操作*。
例如：
- DNS 提供了将机器名称映射到IP地址的命名服务。
- LDAP 提供了将 LDAP 名称映射到LDAP条目的命名服务。
- 文件系统提供了将文件名映射到文件和目录的命名服务。

## 6.2 Namespaces
### 概念
_namespace_ 是命名系统中所有可能的名称的集合。

例如：
1. UNIX文件系统有一个名称空间，由该文件系统中所有文件和目录的名称组成。
2. DNS命名空间包含DNS域和表项的名称。
3. LDAP命名空间包含LDAP条目的名称。