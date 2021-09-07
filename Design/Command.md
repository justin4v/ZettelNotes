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


### 示例

invoker
```java
public class Invoker {  
  
    private Command command;  
  
    public void setCommand(Command command) {  
        this.command = command;  
    }  
  
    public void action(){  
        command.execute();  
    }  
}
```

command
```java
public abstract class Command {  
    // shared receiver  
 protected final Receiver receiver;  
  
    public Command(Receiver receiver) {  
        this.receiver = receiver;  
    }  
  
    public abstract void execute();  
}
```

cookcommand
```java
public class CookCommand extends Command {  
    public CookCommand() {  
        super(new Chef());  
    }  
  
    public CookCommand(Receiver receiver) {  
        super(receiver);  
    }  
  
    @Override  
 public void execute() {  
        receiver.doExecute();  
    }  
}
```

receiver
```java
public interface Receiver {  
    void doExecute();  
}
```

chef
```java
public class Chef implements Receiver {  
    @Override  
 public void doExecute() {  
        System.out.println("I am your chef, I will start cooking...^_^");  
    }  
  
}
```

client
```java
public class Client {  
    public static void main(String[] args) {  
        Invoker invoker = new Invoker();  
        Command cookCommand=new CookCommand();  
        invoker.setCommand(cookCommand);  
        invoker.action();   
    }  
}
```