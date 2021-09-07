## 命令模式
**KEYS**
- 命令封装为 Command 对象；
- 调用者 Invoker设置调用命令 setCommand；
- Command 关联 Receiver；
- Receiver实际执行命令； 
- 支持 Undo；


**Encapsulate a request as an object**, thereby letting you parameterize clients with different requests, queue or log requests, and support undoable operations.

**DES**
关键是封装 request 为 command对象， receiver 实际执行业务逻辑。invoker 在顶层作为调用入口。
