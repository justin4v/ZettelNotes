#Workflow #Activiti

# 简介
- process definition
- process deploy

# 基本概念
一般工作流的完整生命周期包括：*流程定义、部署、启动、任务查询、任务处理、流程结束*
1. **集成 activiti**
	Activiti 是一个工作流引擎（其实是一堆 jar 包 API），业务系统使用 activiti 来对系统的业务流程进行自动化管理，为了方便业务系统访问(操作)activiti 的接口或功能，通常将 activiti 环境与业务系统的环境集成在一起。
2. **流程定义**
	使用 activiti 流程建模工具(activity-designer)*定义业务流程*(.bpmn 文件) 。 .bpmn 文件就是业务流程定义件，通过 xml 定义业务流程。 一般都提供了可视化的建模工具(Process Designer)生成流程定义文件，一般都支持图形化拖拽方式、多窗口的用户界面等功能。
3. **流程定义部署**
	向 activiti 部署业务流程定义（.bpmn 文件）。 使用 activiti 提供的 api 向 activiti 中部署.bpmn 文件（一般情况还需要部署业务流程的图片.png）
4. **启动一个流程实例（ProcessInstance）**
	*流程实例表示一次业务流程的运行*，比如员工请假流程部署完成，如果张三要请假就可以启动一个流程实例，如果李四要请假也启动一个流程实例，两个流程的执行互相不影响。
5. **用户查询待办任务(Task)**
	通过 activiti 可以查询当前流程执行位置，当前用户待办任务，不需要在 sql语句中查询。
6. **用户办理任务**
	用户查询待办任务后，可以办理某个任务，并进行状态流转。
7. **流程结束**
	当任务办理完成没有下一个任务/结点了，流程实例就完成了

# 数据库设计
- 数据库表名默认*以“ACT_”开头*，如 ACT_GE_BYTEARRAY；
- 第二部分用两个字母说明用途：
	- “GE”代表“General”；
	- “HI”代表“History”，*历史数据*；
	- “ID”代表“Identity”，*用户信息*；
	- “RE”代表“Repository”，*存储静态信息*，如流程定义信息；
	- “RU”代表“Runtime”，*运行时数据*，如流程实例、用户任务、变量，**流程结束后被删除**。
- *数据库表名基本和下面的服务接口对应*。

# 服务接口
- **RepositoryService**：*管理流程部署和流程定义的API*，实现流程定义的部署。处理与流程定义相关的静态数据。
- **RuntimeService**：在流程运行时*对流程实例进行管理与控制*。管理 ProcessInstances（当前正在运行的流程）以及流程变量
- **TaskService**：*对流程任务进行管理*，例如任务提醒、任务完成和创建任务等。会跟踪 UserTasks，需要由用户手动执行的任务是Activiti API的核心。我们可以使用此服务创建任务，声明并完成任务，分配任务的受让人等。
- **FormService**：表单服务。是一项*可选服务*，它用于定义中开始表单和任务表单。
- **IdentityService**：*对流程角色数据进行管理*，这些角色数据包括用户组、用户及它们之间的关系。管理用户和组。
- **HistoryService**：*对流程的历史数据进行操作*，包括查询、删除这些历史数据。可以设置不同的历史级别。
- **ManagementService**：*对流程引擎进行管理和维护*。**与元数据相关**，在创建应用程序时通常不需要。
- **DynamicBpmnService**：在*不重新部署的情况下更改流程中的内容*。


# 参考
1. [ SpringBoot + Activiti 工作流引擎（一、基本概念与环境搭建）](https://blog.csdn.net/u014553029/article/details/111147223)
2. [SpringBoot + Activiti 工作流引擎（二、流程&任务操作）](https://blog.csdn.net/u014553029/article/details/112438038)
3. [Introduction - Activiti & Activiti Cloud Developers Guide](https://activiti.gitbook.io/activiti-7-developers-guide/)
4. [Activiti User Guide](https://www.activiti.org/userguide/#_introduction)