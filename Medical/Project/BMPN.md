#Workflow #BPMN
# BPMN
- *业务流程建模符号（Business Process Modeling Notation, BPMN）*，OMG 组织在2011年推出 BPMN2.0标准
- provides a *graphical notation for specifying business processes* in a Business Process Diagram. 
- Its goal is to support Business Process Modeling by *providing a standard notation that is comprehensible to business users yet* represents complex process semantics for technical users.
- 原有的**业务流程模型转换为执行模型**，对于业务流程拥有者和IT实施人员来说，都是容易出错且很艰难的过程；
- 为了减少*业务建模与技术实现*的断层，开发 BPMN 的重要目标就是要创建一座桥梁，连接*业务流程建模标记到 IT实现语言*

# 工作流与BPMN

## 什么是工作流引擎

- 用于**管理和调度流程的应用程序** ；
- 可集成并作为*框架*使用，包括：
	- *流程定义的存储*；
	- *流程的节点与流程条件判断和调度*；
	- *流向管理*；
	- *流程实例管理*等。

## 工作流引擎与BPMN

- 通过BPMN 来进行BPM（业务流程建模）得到的结果就是业务流程的定义；
- 流程定义规定了业务的流转过程由谁参与等等。
- *协调并执行流程*，*记录流程的执行过程和结果*就是工作流引擎的职责范围。

# BPMN2.0的基本元素

BPMN2.0 规范规定的基础元素
## 顺序流（sequence flow）
- 流程中*两个元素间的连接器*。
- 在流程执行过程中，*一个元素被访问后，会沿着其所有出口顺序流继续*执行。
- BPMN 2.0默认是*并行执行*的：两个出口顺序流就会创建两个独立的、并行的执行路径

![[BPMN sequence flow.png]]

- 可以*定义条件（conditional sequence flow）*，默认计算其每个出口顺序流上的条件。当条件计算为true时，选择该出口顺序流。
- 条件顺序流的XML为含有 **conditionExpression** 子元素的普通顺序流。

```xml
<sequenceFlow id="flow" sourceRef="theStart" argetRef="theTask">  
 <conditionExpression xsi:type="tFormalExpression">  
 <![CDATA[${order.price > 100 && order.price < 250}]]>  
 </conditionExpression>  
</sequenceFlow>
```

## 活动任务

### 用户任务
- 用户任务（user task）用于*对需要人工执行的任务进行建模*。
- 当流程执行到达用户任务时，会*为指派至该任务的用户或组创建一个新任务*。

![[BPMN user task.png]]

- 可以直接*指派（assign）给用户*；
- 设置任务到期时间，设置任务的候选用户/候选用户组，并能设置任务的监听器与自定义指派。

```xml
<userTask id="task1" name="My task" >  
 <extensionElements>  
 <flowable:taskListener event="create" class="org.flowable.MyAssignmentHandler" />  
 </extensionElements>  
</userTask>
```

## 服务任务

- 服务任务（service task）用于*调用Java类*。
- 流程执行到服务任务时，会自动运行Java程序中的代码流程。
![[BPMN service task.png]]

```xml
<serviceTask id="javaService"  
 name="My Java Service Task"  
 flowable:class="com.inossem.MyJavaDelegate" />
```

## 网关
### 排他网关
- 排他网关（exclusive gateway）（异或网关 XOR gateway，或者基于数据的排他网关 exclusive data-based gateway），用于*对流程中的**决策**建模*；
- 当执行到达这个网关时，会对所有出口按顺序进行计算。
- 选择第一个条件计算为 true 的顺序流（当没有设置条件时，默认为true）继续流程

![[BPMN 网关.png]]

### 并行网关

 - 在流程模型中引入并行的最简单的网关，就是**并行网关（parallel gateway）**。
 - 可以将执行*分支（fork）* 为多条路径，也可以*合并（join）* 多条入口路径的执行。

并行网关的功能取决于其入口与出口顺序流：

-   **分支：所有的出口顺序流都并行执行，为每一条顺序流创建一个并行执行**。
-   **合并：所有到达并行网关的并行执行都会在网关处等待，直到每一条入口顺序流都到达了有个执行。然后流程经过该合并网关继续**。

**与其他网关类型有一个重要区别：并行网关不计算条件。如果连接到并行网关的顺序流上定义了条件，会直接忽略该条件**

![[BPMN 并行网关.png]]

## 事件

### 启动事件
启动事件（start event）是流程的起点。启动事件的类型（流程在消息到达时启动，在指定的时间间隔后启动，等等），定义了流程如何启动，并显示为启动事件中的小图标。在XML中，类型由子元素声明来定义

空启动事件、定时器启动事件、消息启动事件、信号启动事件、错误启动事件等

![[空启动时间.png]]
![[定时器启动时间.png]]

![[消息启动时间.png]]


### 结束事件
结束事件（end event）标志着流程或子流程中一个分支的结束。结束事件总是抛出型事件。这意味着当流程执行到达结束事件时，会抛出一个结果。结果的类型由事件内部的黑色图标表示

![[空结束事件.png]]

![[错误结束事件.png]]

![[取消结束事件.png]]

## 工作流引擎与工程结合的解决方案

### 方式一

将系统中对于*审批/经办等内容交给工作流引擎处理*，*其他复杂的传统业务由代码处理*。

此种方式可以将相对比较固定的审批流转、委托代办、会签或签等功能分离给工作流引擎处理，而相对复杂的各种业务仍然采用代码定制的方式来拟合业务

这样做的优点是，对于各种复杂的权限判断，业务上的复杂处理，牢牢掌握在自己手中，对于业务流转、条件判断等直接使用代码解决，理解起来简单，操作上直接。

缺点方面，工作流引擎形同虚设，没有发挥出更大作用。

形象的例子是：你到餐馆去吃饭，想要吃一份儿水煮鱼，你认识会做水煮鱼的厨师，直接来到后厨，找到厨师并且提供给厨师全套厨具以及待烹饪的生鱼让他给你做饭。简单直接，没有中间服务员点菜和传菜的步骤，做菜的指令直接传递给厨师，菜做好了也是直接交给你的。

### 方式二

作为**业务调度中心**

业务中的每一个步骤的具体执行由代码完成，而步骤之间的所有衔接全部交由工作流引擎处理。业务发起时，调用方只负责将对应的发起人和要发起的业务代码传递给工作流引擎，其余的事情交给引擎自己去调度。

以仓储业务的入库单的提交为例：

从单据提交，发起审批，保存批次信息，保存单据信息，到根据配置生成仓库作业请求，修改库存，回调修改单据状态等等，各个环节的启动与流转，完全交给工作流引擎调度。发起方只需要去关注整个业务的进展状态即可，不再关注后续执行的细节（如何时生成请求，应该使用什么作业模式等等）

这种做法的优点是，发起方与执行方进行了拆分，引入了调度协调器的角色。业务的执行依赖调度器的串联。发起方不必再关注后续的细节，只需关注业务本身的流转状态。业务流程的编排由事先指定好的BPMN模型定义。

缺点方面，引入了更多的复杂度，工作流引擎成为了系统的核心运转依赖组件，需要在数据一致性、并发、与高性能方面做更多的设计与考量。

# 参考
1. [深入浅出了解BPM、BPMN、BPMN2.0](https://www.cnblogs.com/amerkor/p/13728576.html)