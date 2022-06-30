#Thread #Java基础 #Join

# 1. join()介绍
- join() 定义在Thread.java中。  
- join() 的作用：*让“主线程”等待“子线程”结束之后，主线程才继续运行*。

```java
// 主线程
public class Father extends Thread {
    public void run() {
        Son s = new Son();
        s.start();
        s.join();
        ...
    }
}
// 子线程
public class Son extends Thread {
    public void run() {
        ...
    }
}
```


**说明**：  
- Son 是在 Father 中创建并启动的，所以，Father是主线程类，Son是子线程类。  
- 在 Father 主线程中，通过 new Son() 新建“子线程s”。接着通过 s.start() 启动“子线程s”，并且调用s.join()。
- 在调用s.join()之后，Father主线程会一直等待，直到“子线程s”运行完毕；
- 在“子线程s”运行完毕之后，Father 主线程才能接着运行。 
- 也就是：“join()的作用，是让主线程会等待子线程结束之后才能继续运行”

# 2. join()源码分析(基于JDK1.7.0_40)
```java
public final void join() throws InterruptedException {
    join(0);
}

public final synchronized void join(long millis) throws InterruptedException {
    long base = System.currentTimeMillis();
    long now = 0;

    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (millis == 0) {
        while (isAlive()) {
            wait(0);
        }
    } else {
        while (isAlive()) {
            long delay = millis - now;
            if (delay <= 0) {
                break;
            }
            wait(delay);
            now = System.currentTimeMillis() - base;
        }
    }
}
```

1. 从代码中，可以发现: 
	1. 当 `millis==0` 时，会进入 `while(isAlive())` 循环；
	2. 只要子线程是活的，主线程就不停的等待。  

## 问题
- 虽然 s.join()被调用的地方是发生在“Father主线程”中，但是s.join()是通过“子线程s”去调用的join()。
- 那么，join()方法中的 isAlive() 应该是判断 “子线程s” 是不是Alive状态；对应的wait(0)也应该是“让子线程s”等待才对。
- s.join()的作用应该是让"子线程等待才对(因为调用子线程对象s的wait方法嘛)"？  

**答案**：
- [wait()](http://www.cnblogs.com/skywang12345/p/3479224.html)让“*当前线程*”等待，而这里的“当前线程”是指当前在 CPU 上运行的线程。
- 虽然是调用子线程的 wait() 方法，但是它是通过“主线程”去调用的；所以，休眠的是主线程，而不是“子线程”！

# 3. join()示例

在理解join()的作用之后，接下来通过示例查看join()的用法。

```java
// JoinTest.java的源码
public class JoinTest{ 

    public static void main(String[] args){ 
        try {
            ThreadA t1 = new ThreadA("t1"); // 新建“线程t1”

            t1.start();                     // 启动“线程t1”
            t1.join();                        // 将“线程t1”加入到“主线程main”中，并且“主线程main()会等待它的完成”
            System.out.printf("%s finish\n", Thread.currentThread().getName()); 
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    } 

    static class ThreadA extends Thread{

        public ThreadA(String name){ 
            super(name); 
        } 
        public void run(){ 
            System.out.printf("%s start\n", this.getName()); 

            // 延时操作
            for(int i=0; i <1000000; i++)
               ;

            System.out.printf("%s finish\n", this.getName()); 
        } 
    } 
}
```

**运行结果**
```
t1 start
t1 finish
main finish
```


## 结果说明
**运行流程如图**   
![[Thread.join()运行流程示意.png]]

1. 在“主线程main”中通过 new ThreadA("t1") 新建“线程t1”。 
2. 接着，通过 t1.start() 启动“线程t1”，并执行t1.join()。  
3. 执行t1.join()之后，“主线程main”会进入“阻塞状态”等待t1运行结束。
4. “子线程t1”结束之后，会唤醒 “主线程main”，“主线程”重新获取 cpu 执行权，继续运行。

# 参考 
1. [Java多线程系列--“基础篇”08之 join()](https://www.cnblogs.com/skywang12345/p/3479275.html)