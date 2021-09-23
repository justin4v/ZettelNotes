# 同步与异步
POSIX 定义如下：
- 同步 I/O 操作（synchronous I/O operation）：An I/O operation that **causes the thread requesting the I/O to be blocked** from further use of the processor until that I/O operation completes；
- 异步 I/O 操作（asynchronous I/O operation）：An I/O operation that does not of itself cause the thread requesting the I/O to be blocked from further use of the processor.。

# 阻塞与非阻塞
## Blocking
POSIX 定义：
A property of an open file description that causes function calls associated with it to **wait for the requested action to be performed before returning.**

## Non-Blocking
POSIX 定义：
A property of an open file description that causes function calls involving it to **return without delay** when it is detected that the requested action associated with the function call cannot be completed without unknown delay.