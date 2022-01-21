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
- 流程中两个元素间的连接器。
- 在流程执行过程中，一个元素被访问后，会沿着其所有出口顺序流继续执行。
- BPMN 2.0的默认是并行执行的：两个出口顺序流就会创建两个独立的、并行的执行路径

![[BPMN sequence flow.png]]

- 可以定义条件（conditional sequence flow），默认计算其每个出口顺序流上的条件。当条件计算为true时，选择该出口顺序流。
- 条件顺序流的XML表示格式为含有 **conditionExpression** 子元素的普通顺序流。

```xml
<sequenceFlow id="flow" sourceRef="theStart" argetRef="theTask">  
 <conditionExpression xsi:type="tFormalExpression">  
 <![CDATA[${order.price > 100 && order.price < 250}]]>  
 </conditionExpression>  
</sequenceFlow>
```

## 活动任务

### 用户任务
用户任务（user task）用于对需要人工执行的任务进行建模。当流程执行到达用户任务时，会为指派至该任务的用户或组的任务列表创建一个新任务。
![[BPMN user task.png]]

用户任务可以直接指派（assign）给用户，设置任务到期时间，设置任务的候选用户/候选用户组，并能设置任务的监听器与自定义指派。

```xml
<userTask id="task1" name="My task" >  
 <extensionElements>  
 <flowable:taskListener event="create" class="org.flowable.MyAssignmentHandler" />  
 </extensionElements>  
</userTask>
```

## 服务任务

服务任务（service task）用于调用Java类。流程执行到服务任务时，会自动运行Java程序中的代码流程。
![[BPMN service task.png]]

```xml
<serviceTask id="javaService"  
 name="My Java Service Task"  
 flowable:class="com.inossem.MyJavaDelegate" />
```

## 网关
### 排他网关
\排他网关（exclusive gateway）（也叫异或网关 XOR gateway，或者更专业的，基于数据的排他网关 exclusive data-based gateway），用于对流程中的**决策**建模


# 参考
1. [深入浅出了解BPM、BPMN、BPMN2.0](https://www.cnblogs.com/amerkor/p/13728576.html)