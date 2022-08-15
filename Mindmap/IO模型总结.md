---

mindmap-plugin: basic

---

# IO模型

## blocking IO

### CPU处于阻塞状态
- 阻塞等待，浪费CPU资源

## non-blocking IO

### 立即返回错误/成功
- 1.CPU立即返回成功或失败，不阻塞
- 2.不断轮询确认

## IO multiplexing

### 监听事件
- select
   - 1.最大1024
   - 2.轮询 fd 集
- poll
   - 1.无最大1024数量限制
   - 2.轮询 fd 集
- epoll
   - 1.无最大fd限制
   - 2.事件驱动

## event driven IO

### 事件驱动，使用很少

## asynchronize IO

### 异步 IO