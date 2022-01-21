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