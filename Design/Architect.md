#Design #Architect #架构设计要素

*架构还没有统一的定义*

# 研究人员定义
- 软件架构就是一个系统的草图。
- 软件架构描述的对象是直接构成系统的抽象组件。
- 各个组件之间的连接则明确和相对细致地描述组件之间的通信。
- 在实现阶段，这些抽象组件被细化为实际的组件，比如，在面向对象领域中，组件就是具体某个类或者对象，而组件之间的连接通常用接口来实现。

由于学术界和工业界的联系不紧密，甚至脱节，导致它们对软件架构的认识不一致，从而使得软件架构至今很难有一个统一的定义，甚至是一个近似统一的定义

# 组成派定义
- 组成派定义的主要依据是软件架构主要反映系统由哪些部分组成；
- 以及这些部分是如何组成的；
- 强调软件系统的整体结构和配置

## David & Mary 模型
软件架构是*软件设计过程的层次之一*，该层次超越计算过程中的算法设计和数据结构设计。

**软件体系结构 = { 构件(component)，连接件(connector)，约束(constrain) }**．

1. 构件可以是一组代码，如程序的模块；也可以是一个独立的程序，如数据库服务器。
2. 连接件可以是过程调用、管道、远程过程调用(RPC)等，用于表示构件之间的相互作用。
3. 约束一般为对象连接时的规则，或指明构件连接的形式和条件。例如，上层构件可要求下层构件的服务，反之不行；两对象不得递规地发送消息；代码复制迁移的一致性约束；什么条件下此种连接无效等。

# 决策派定义
- 决策派定义的主要依据是软件架构设计是软件设计的一部分；
- 软件设计实际上是开发人员意志和决策在软件开发过程中的体现；
- 软件架构更是高层领导和架构师意志和决策的体现，强调的是设计决策，所以更加注重架构风格和模式的选择

## Booch 
1999年Booch等人认为软件架构是一系列重要决策的集合，这些决策与以下内容有关：
1. 软件系统的组织；
2. 选择组成系统的结构元素和它们之间的接口，以及当这些元素相互协作时所体现的行为；
3. 如何组合这些元素，使它们逐渐合成为更大的子系统；
4. 用于指导这个系统组织的架构风格。

# 简要定义
架构，又名软件架构，是有关*软件整体结构与组件的抽象描述*，用于指导大型软件系统各个方面的设计。

简单的说架构就是：
1. 一个*蓝图*，一种*设计方案*；
2. 将客户的不同*需求抽象成为抽象组件*，并且能够描述这些抽象组件之间的*通信和调用*。


# 架构设计方法
-   架构模式：顶层模型的设计方法
-   框架的设计模式：框架类结构的设计方法
-   架构设计目标：非功能性的约束

![[软件架构设计要素.png]]

## Architecture Pattern
### 领域驱动设计(Domain Driven Design)

Eric Evans在《领域驱动设计－软件核心复杂性应对之道》这本书中提出了传统的DDD四层架构模式。

### 六边形架构（Haxagonal Architecture）

六边形架构是Alistair Cockburn在2005年提出的，解决了传统的分层架构所带来的问题。Vaughn Vernon 在《实现领域驱动设计》一书中，作者将六边形架构应用到领域驱动设计的实现。将传统的分层架构变成了内部和外部，内部实现领域模型，外部实现适配器。

### 洋葱架构（Onion Architecture）

 Jeffrey Palermo在2008年提出了洋葱架构。它是从六边形架构发展而来，将六边形改为圆环，层层依赖。

### 干净架构（Clean Architecture）

Robert C. Martin在2012年提出了干净架构（Clean Architecture），这是六边形架构的一个变体。

### DCI架构

DCI代表Data, Context, Interaction。ames O. Coplien和Trygve Reenskaug在2009年发表了一篇论文《DCI架构：面向对象编程的新构想》

重点是关注数据的不同场景的交互行为， 核心思想是将面向对象系统的数据模型和动态的行为模型区分开来，用不同的对象复用同一段交互行为。


## 框架的设计模式
### MVC框架模式

MVC即模型(model)、视图(view)、控制器(controller)设计模式可谓是无人不知不人不晓。它的缺点是比较重，各部分没有解耦。

### MVP框架模式

Model-View-Presenter设计模式是在MVC基础上发展而来，将模型与视图完全分离，可以修改视图而不影响模型。

### MVVM框架模式

Model-View-ViewModel设计模式是MVP的进一步改进，使用双向绑定将View与ViewModel的数据传输自动化。

### VIPER框架模式

VIPER模式最初是在2013年由Jeff Gilbert 和 Conrad Stoll 提出，随后在《Architecting iOS Apps with VIPER》文中做了详细的介绍。

由View+Interactor+Presenter+Entity+Routing组成，是Clean Architecture的一种实现模式。


# 参考
1. D Garlan, M Shaw. An introduction to software architecture[M]. Advances in software engineering and knowledge engineering, 1993: 1-39
2. G Booch, J Rumbaugh, I Jacobson. The Unified Modeling Language User Guide [M]. 2nd ed. Addison Wesley, 1999.