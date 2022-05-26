#Linux #Operating-system #fork

# 含义
- fork 在英文中是"分叉"的意思。
- 一个进程在运行中，如果使用了fork，就产生了另一个进程，于是进程就"分叉"了；
- fork 会有两次返回，将子进程的PID返回给父进程，0返回给子进程；
- fork is an operation whereby a process creates a copy of itself.
- fork是类Unix操作系统上创建进程的主要方法。
- fork用于创建子进程(等同于当前进程的副本)，fork() 完后，父进程和子进程有着同样的*数据段和代码段，栈，PCB*也大部分相同，两个进程就会干着同样的事情。

## fork 的基本使用
```c
int main ()   
{   
    pid_t fpid; //fpid表示fork函数返回的值  
    int count=0;
    
    // 调用fork，创建出子进程  
    fpid=fork();

    // 所以下面的代码有两个进程执行！
    if (fpid < 0)   
        printf("创建进程失败!/n");   
    else if (fpid == 0) {  
        printf("我是子进程，由父进程fork出来/n");   
        count++;  
    }  
    else {  
        printf("我是父进程/n");   
        count++;  
    }  
    printf("统计结果是: %d/n",count);  
    return 0;  
}  
```

结果
```
我是子进程，由父进程fork出来

统计结果是: 1

我是父进程

统计结果是: 1
```

# 实际过程
- fork 底层是调用了内核的函数来实现 fork 的功能的：
	- 先create()先创建进程，此时进程内容为空；
	- 然后clone()复制父进程的内容到子进程中，此时子进程就诞生了；
	- 接着父进程就return返回了。

- 子进程诞生后，父进程已经运行到了 fork() 的 return 语句，PC 指针已经指向了 return。
- 子进程 PC 指针也是复制于父进程，所以会直接运行 return 语句返回的，并接着执行后面的程序；
- 子进程是不会执行前面父进程已经执行过的程序是从 fork() 后面的程序继续执行。
- fork() 中的 return 会返回子进程的 pid：
	- 在父进程得到的是子进程的 pid ；
	- 子进程没有子进程了，所以返回 0；

![[fork() 实际过程示意.png]]



