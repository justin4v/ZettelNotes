# 同步与异步

## 同步
发出功能调用后，*等待功能完成，才从调用中返回并进行下一步操作*。
事务必须一个接着一个的完成，前一件完成了才能做下一件。


## 异步
发出功能调用后，立即从功能调用中返回（没有得到实际操作结果），并可。实际处理这个调用的部件在完成后，通过状态、通知和回调来通知调用者。


POSIX 定义如下：
- 同步 I/O 操作（synchronous I/O operation）：An I/O operation that **causes the thread requesting the I/O to be blocked** from further use of the processor until that I/O operation completes；
- 异步 I/O 操作（asynchronous I/O operation）：An I/O operation that **does not of itself cause the thread requesting the I/O to be blocked** from further use of the processor. This implies that the process and the I/O operation may be running concurrently.。

# 阻塞与非阻塞
## Blocking


## Non-Blocking
