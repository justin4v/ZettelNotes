#Session #Cookie #Web #Todo 


# 背景
- 最开始，Web 页面*只是文档浏览*， 服务器不需要记录某个用户一段时间的浏览记录；
- *每次请求都是一个新的 HTTP 协议*， 不用记住谁发了 HTTP 请求。
- 之后交互式 Web 应用兴起，*Web 页面软件化*，如在线购物网站，必须记住哪些人可以登录系统， 哪些人在购物车中放了什么商品， 必须把每个人区分开。
- 此时，在程序中，*会话跟踪*很重要。
- **一个用户的所有请求操作都应该属于同一个会话**，另一个用户的所有请求应该属于另一个会话，不能混淆。

## 问题
- Web 应用程序使用 HTTP 协议传输数据。
- HTTP协议是*无状态*的协议：
	- 一旦数据交换完毕，客户端与服务器端的连接就会关闭；
	- 再次交换数据需要建立新的连接。
	- 服务器*无法从连接上跟踪会话*。
- 为了解决会话追踪问题，出现了：
	- cookie 技术（客户端存储）；
	- session 技术（服务端存储）。


# cookie
## 历史
- Cookie意为“甜饼”，是**由W3C组织提出**，最早由 Netscape 社区发展出的一种机制。
- 目前Cookie已经成为 Web 标准的一部分。

## 原理
- 给每个**客户端颁发通行证**；
- 客户端访问都必须**携带通行证**；
- 服务器就能从通行证上确认客户身份。


## 实现
- 客户端请求服务器，如果*服务器需要记录该用户状态*，就*在 response 中向客户端浏览器颁发一个Cookie*。
- 客户端浏览器会把 Cookie 保存起来。
- 当浏览器再请求该网站时，浏览器把请求的网址连同 Cookie 一同提交给服务器。
- 服务器检查该Cookie，校验用户状态。

## cookie 不可跨域
- 很多网站都会使用Cookie。例如，Google会向客户端颁发Cookie，Baidu也会向客户端颁发Cookie。那浏览器访问Google会不会也携带上Baidu颁发的Cookie呢？或者Google能不能修改Baidu颁发的Cookie呢？

- 答案是否定的。**Cookie具有不可跨域名性**。根据Cookie规范，浏览器访问Google只会携带Google的Cookie，而不会携带Baidu的Cookie。Google也只能操作Google的Cookie，而不能操作Baidu的Cookie。

Cookie在客户端是由浏览器来管理的。浏览器能够保证Google只会操作Google的Cookie而不会操作Baidu的Cookie，从而保证用户的隐私安全

images.google.com与网站www.google.com同属于Google，但是域名不一样，二者同样不能互相操作彼此的Cookie。
注意：用户登录网站www.google.com之后会发现访问images.google.com时登录信息仍然有效，而普通的Cookie是做不到的。这是因为Google做了特殊处理

# session
## 原理
- Session 机制就是通过检查服务器上的 “*客户明细表*” 来确认客户身份。
- Session 相当于程序在服务器上建立的一份客户档案，客户来访的时候只需要查询客户档案表就可以了。
- 服务器创建session出来后，会把 session 的 id，以 cookie 的形式回写给客户机


# 参考
1. [JavaWeb学习总结(十二)——Session](https://www.cnblogs.com/xdp-gacl/p/3855702.html)
2. [Session、Cookie、Token](https://cloud.tencent.com/developer/article/1704064)
3. [cookie和session的详解与区别](https://www.cnblogs.com/l199616j/p/11195667.html)