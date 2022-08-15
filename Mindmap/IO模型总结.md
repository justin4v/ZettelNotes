---

mindmap-plugin: basic

---

# IO模型

## blocking IO

### CPU处于阻塞状态 ^d51263ec-4781-5bb9
- 阻塞等待，浪费CPU资源

## non-blocking IO

### 立即返回错误/成功 ^edd93526-48d4-9643
- 1.CPU立即返回成功或失败，不阻塞
- 2.不断轮询确认

## IO multiplexing

### 监听事件
- select
   - 1.最大1024
   - 新节点
- poll
- epoll

## event driven IO

## asynchronize IO