#Session #JSessionId

# jsessionId sessionId
- sessionid是一个会话的 key，浏览器第一次访问服务器会在服务器端生成一个session，有一个sessionid和它对应。
- tomcat生成的sessionid叫做jsessionid。
- session在访问tomcat**服务器**HttpServletRequest的getSession(true)的时候**创建**，tomcat的ManagerBase类提供创建 sessionid 的方法：**随机数+时间+jvmid**；
- session 存储在服务器的内存中，tomcat 的 StandardManager 类将 session **存储在内存中**，也可以持久化到file，数据库，memcache，redis等。
- **客户端只保存sessionid到cookie**中，而不会保存session，session销毁只能通过invalidate或超时，关掉浏览器并不会关闭session。


# 说明
**创建**：sessionid第一次产生是在直到某server端程序调用 HttpServletRequest.getSession(true)这样的语句时才被创建。

**删除**：超时；程序调用HttpSession.invalidate()；程序关闭；

**session存放在哪里**：服务器端的内存中。不过session可以通过特殊的方式做持久化管理（memcache，redis）。

**session的id是从哪里来的**，sessionID是如何使用的：当客户端第一次请求session对象时候，服务器会为客户端创建一个session，并将通过特殊算法算出一个session的ID，用来标识该session对象

**session会因为浏览器的关闭而删除吗？**

不会，session只会通过上面提到的方式去关闭。
# 参考
1. [sessionid如何产生？由谁产生？保存在哪里？](https://www.cnblogs.com/woshimrf/p/5317776.html)