#Session #JSessionId

# jsessionId sessionId
- sessionid是一个会话的 key，浏览器第一次访问服务器会在服务器端生成一个session，有一个sessionid和它对应。
- tomcat生成的sessionid叫做jsessionid。
- session在访问tomcat**服务器**HttpServletRequest的getSession(true)的时候**创建**，tomcat的ManagerBase类提供创建 sessionid 的方法：**随机数+时间+jvmid**；
- sessionId 存储在服务器的内存中，tomcat 的 StandardManager 类将session**存储在内存中**，也可以持久化到file，数据库，memcache，redis等。
- **客户端只保存sessionid到cookie**中，而不会保存session，session销毁只能通过invalidate或超时，关掉浏览器并不会关闭session。

# 参考
1. [sessionid如何产生？由谁产生？保存在哪里？](https://www.cnblogs.com/woshimrf/p/5317776.html)