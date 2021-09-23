# 同步与异步

## 同步
发出一个功能调用后，*等待功能完成并返回后，才能进行下一步操作*。
必须一件一件事做，等前一件做完了才能做下一件事


## 异步



POSIX 定义如下：
- 同步 I/O 操作（synchronous I/O operation）：An I/O operation that **causes the thread requesting the I/O to be blocked** from further use of the processor until that I/O operation completes；
- 异步 I/O 操作（asynchronous I/O operation）：An I/O operation that **does not of itself cause the thread requesting the I/O to be blocked** from further use of the processor. This implies that the process and the I/O operation may be running concurrently.。

# 阻塞与非阻塞
## Blocking


## Non-Blocking
