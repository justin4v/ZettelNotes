#Java基础 #Thread #Main-thread

# main
- Java 程序总是从 main() 方法开始运行，此时会有一个线程立即开始运行，通常称为主线程。
- 从 JDK6 开始，**main() 方法在 Java 程序中是强制使用的**。

## 特点
1. **由 JVM 自动创建**（同样会为 main 线程分配堆栈空间）；
2. *子线程都从该线程中被孵化*（见下图）；
3. 通常主线程是*最后一个结束的线程*，因为它会执行各种的关闭操作。
4. 通过 `Thread.currentThread()` 方法**获得线程引用**，以便于操作主线程（因为由 JVM 创建，无法创建时直接得到引用）。

![[JVM主线程和其他线程关系.png]]


## 主线程中死锁（单个线程）

主线程创建一个死锁：
```java
public class Test {
    public static void main(String[] args) {
        try {
            System.out.println("Entering into Deadlock");
            Thread.currentThread().join();
            // the following statement will never execute 
            System.out.println("This statement will never execute");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
} 
```

看下输出：

```csharp
Entering into Deadlock
```

语句`Thread.currentThread().join()`会告诉主线程一直等待这个线程（也就是等它自己）运行结束，所以我们就有了死锁。

# main 和子线程

```java
class ExampleOfThreadCreation extends Thread
{  
   public static void main(String args[])
   {  
       firstMethod();
       SecondMethod();
       ExampleOfThreadCreation t1 = new ExampleOfThreadCreation();
       t1.start();
   }  
   public void run()
   {
       System.out.println("Thread is running...");
       for(int i = 0; i < 5 ; i++)
           methodOfThread(i);
      
   }
   
   public static void firstMethod()
   {
       System.out.println("First method");
   }
   
   public static void SecondMethod()
   {
       System.out.println("Second method");
   }
   
   public static void methodOfThread(int i)
   {
       System.out.println("Counting in thread : " + i);
   }
}  
```

可能的输出：
```
First method  
Second method  
Thread is running…  
Counting in thread : 0  
Counting in thread : 1  
Counting in thread : 2  
Counting in thread : 3  
Counting in thread : 4
```

main和子线程的*堆栈关系如下*：

![[主线程和子线程的堆栈关系示意.png]]