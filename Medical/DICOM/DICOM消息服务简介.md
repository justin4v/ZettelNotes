#DICOM #DIMSE #C-MOVE #C-STORE #C-FIND #C-GET

# 引言

  DICOM(Digital Imaging and Communications in Medicine)医学数字成像与通信，是医疗影像领域一个非常重要的标准，本文主要简单介绍 DICOM 标准中的消息交换服务。

# 名词简介

1. **DIMSE**：DICOM Message Service Element（DICOM 消息服务元素）  
2. **DIMSE-C**：DICOM Message Service Element - Composite（复合 DICOM 消息服务元素）  
3. **DIMSE-N**：DICOM Message Service Element - Normalized（标准化的 DICOM 消息服务元素）  
4. **DIMSE-service-user**：that part of an application entity that makes use of the DICOM Message Service Element.（使用 DICOM 消息服务元素的应用实体部分）

# DIMSE-C
DIMSE-C 服务是支持在有同等 DIMSE-service-user 复合信息对象定义的复合 SOP 实例上操作的 DIMSE 服务的子集，复合 SOP 实例大致可以理解为不会被改变的文档类的实体，例如 dicom 影像文件。DIMSE-C 服务包含以下5个服务：

1.  **C-STORE**：用于一个 DIMSE-service-user 在同等的 DIMSE-service-user 上存储一个复合 SOP 实例；其实就是存储服务，可以用来归档影像，也可以用来获取影像；  
    有关详细介绍请参考 [Dicom 学习笔记-DICOM C-Store 消息服务](https://www.jianshu.com/p/bab6a85d3486)
2.  **C-FIND**：查询服务，用于一个 DIMSE-service-user 在同等的DIMSE-service-user 上查询复合 SOP 实例的属性满足查询条件给出的一组属性的复合 SOP 实例；我们可以通过此服务查询某一 PatientID 为xx的患者的所有检查影像；  
    有关详细介绍请参考 [Dicom 学习笔记-DICOM C-Find 消息服务](https://www.jianshu.com/p/035dfa708077)
3.  **C-GET**：获取服务，用于一个 DIMSE-service-user 在同等的DIMSE-service-user 上查询复合 SOP 实例的属性满足查询条件给出的一组属性的复合 SOP 实例，并取回这些符合条件的复合 SOP 实例，同时在这个过程中将触发一个或多个 C-STORE 子操作过程，所有的操作（包含 C-STORE 子操作）均在同一个 Association 连接中；  
    有关详细介绍请参考 [Dicom 学习笔记-DICOM C-Get 消息服务](https://www.jianshu.com/p/c7f5b9fa597c)
4.  **C-MOVE**：也是获取服务，但是获取的发起方和接收方可以是同一个实体也可以是两个不同的实体。标准中是这么定义的：用于一个 DIMSE-service-user 在同等的 DIMSE-service-user 上查询复合 SOP 实例的属性满足查询条件给出的一组属性的复合 SOP 实例，并取回这些符合条件的复合 SOP 实例，同时在这个过程中将触发一个或多个 C-STORE 子操作过程，所有的 C-STORE 子操作触发在另外一个单独的 TCP 连接中；和 C-GET 最大的区别是这个是两个 Association 连接，而 C-GET 服务是一个；  
    有关详细介绍请参考 [Dicom 学习笔记-DICOM C-Move 消息服务](https://www.jianshu.com/p/7e753628a865)
5.  **C-ECHO**：验证两个同等的 DIMSE-service-user 之间端到端的通信是否成功；  
    有关详细介绍请参考 [Dicom 学习笔记-DICOM C-Echo 消息服务](https://www.jianshu.com/p/ef577f069f4b)

# DIMSE-N
DIMSE-N 服务是支持在有对等 DIMSE-service-user 规格化信息对象定义的规格化 SOP 实例上操作和通知的 DIMSE 服务的子集。这类服务会在打印（具体可以参考 DICOM 标准第4部分的附录H和第17部分的附录BB）中使用到。DIMSE-N 服务包含以下6个服务：

1.  N-EVENT-REPORT：用来由一个 DIMSE-service-user 给对等的另一个 DIMSE-service-user 报告一个事件；唯一一个通知类型的服务；
2.  N-GET：用于一个 DIMSE-service-user 从对等的另一个 DIMSE-service-user 取回属性值；
3.  N-SET：用于一个 DIMSE-service-user 向对等的另一个 DIMSE-service-user 请求属性值修改；
4.  N-ACTION：用于一个 DIMSE-service-user 向对等的另一个 DIMSE-service-user 请求一个操作；
5.  N-CREATE：用于一个 DIMSE-service-user 向对等的另一个 DIMSE-service-user 请求创建新的托管 SOP 实例，完成其标识和相关属性的值，同时注册其标识。
6.  N-DELETE：用于一个 DIMSE-service-user 向对等的另一个 DIMSE-service-user 请求删除一个托管 SOP 实例，同时注销其标识。


# 参考
1. [Dicom 学习笔记-Dicom 消息服务（DIMSE-C/DIMSE-N）](https://www.jianshu.com/p/2812b0b6e548)