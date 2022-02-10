#Workflow #Activiti

# 简介
- process definition
- process deploy

# 基本概念

1. **部署 activiti**
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


# 参考
1. [ SpringBoot + Activiti 工作流引擎（一、基本概念与环境搭建）](https://blog.csdn.net/u014553029/article/details/111147223)
2. [SpringBoot + Activiti 工作流引擎（二、流程&任务操作）](https://blog.csdn.net/u014553029/article/details/112438038)
3. [Introduction - Activiti & Activiti Cloud Developers Guide](https://activiti.gitbook.io/activiti-7-developers-guide/)
4. [Activiti User Guide](https://www.activiti.org/userguide/#_introduction)