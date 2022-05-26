#Linux #Operating-system #fork

# 含义
- fork 在英文中是"分叉"的意思。
- 一个进程在运行中，如果使用了fork，就产生了另一个进程，于是进程就"分叉"了；


fork的基本框架：
```c
```
\#include <unistd.h>  
\#include <stdio.h>  
 
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


```