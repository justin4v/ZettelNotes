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

### 启动&结束事件
启动事件（start event）是流程的起点。启动事件的类型（流程在消息到达时启动，在指定的时间间隔后启动，等等），定义了流程如何启动，并显示为启动事件中的小图标。在XML中，类型由子元素声明来定义

空启动事件、定时器启动事件、消息启动事件、信号启动事件、错误启动事件等




# 参考
1. [深入浅出了解BPM、BPMN、BPMN2.0](https://www.cnblogs.com/amerkor/p/13728576.html)