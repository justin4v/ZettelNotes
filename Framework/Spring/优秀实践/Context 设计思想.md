#Spring #Best-practice #Design 

# 是什么

-   Context 指做一件事情的**背景/环境/上下文/所需的必要数据**；
	- 如程序运行的*外部变量、对象*；
	- 一般程序只有*上文数据*，操作系统进程/线程
-   AppContext 应用上下文，包含整个 App 运行期间必要的数据；
-   UserContext 用户上下文，包含一个 User 的上下文数据；

# 特点
-   每个Context对象只包含自身的数据
-   多个Context对象就对应不同的数据
-   很容易做到**数据隔离**，避免结构混乱

# 示例


## 进程、线程上下文

-   在一个操作系统中，包含多个进程，每个进程包含多个线程；
-   每个进程和线程都有自身运行的**上文和下文数据**，以便和其他进程和线程区分；


## 微信聊天
微信的时候都会登录自己的账号，并且和不同的好友进行聊天。
对于该场景可以这么设计数据结构：
```java
AccountContext[]
    Account
    SessionContext[]
        Session
```

-   首先，第一层是账号上下文，每个账号都有自己的数据，如名称、头像等，在切换账号时需要把区分旧的AccountContext和新的AccountContext
-   其次，第二层是会话上下文，同时和多个不同的好友聊天，每个聊天的上下文数据隔离
-   最后，第三层的会话实体，包含会话ID之类的数据

# 参考
1. [ApplicationContext (Spring Framework 5.3.20 API)](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ApplicationContext.html)