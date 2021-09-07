## 命令模式
**KEYS**
- 命令封装为 Command 对象；
- 调用者 Invoker设置调用命令 setCommand；
- Command 关联 Receiver；
- Receiver实际执行命令； 
- 支持 Undo；

**DES**