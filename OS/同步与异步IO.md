# 对比
POSIX 定义如下：
- 同步 I/O 操作（synchronous I/O operation）：导致请求进程阻塞，直到 I/O 操作完成；
- 异步 I/O 操作（asynchronous I/O operation）：不会导致请求进程阻塞。

## Blocking
POSIX 定义：
A property of an open file description that causes function calls associated with it to wait for the requested action to be performed before returning.