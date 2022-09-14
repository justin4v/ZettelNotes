#Todo #Springboot #SpringSecurity

# 框架
![[SpringSecurity主要流程.svg]]

# 启动

导入spring-boot-starter-security启动器后，Spring Security已经生效，默认拦截全部请求，如果用户没有登录，跳转到内置登录页面。

在浏览器输入：`http://localhost:8080/` 进入Spring Security内置登录页面

用户名：user。

密码：项目启动，打印在控制台中。

# 参考
1. [Spring Security 学习笔记，看了必懂！ (qq.com)](https://mp.weixin.qq.com/s?src=11&timestamp=1663117899&ver=4043&signature=UsN5purUNjQ2rb9dFTvgMTP9EMiZ6jKVFpsSdDWuu6mu0MHUIxLbMjKb1qmRktvaDqTRuG5uNFL3uOh0s7aUQYlkWFR8BDr5jOSTprIpuLrjRZKVyk9f2eUO6uUTFXyz&new=1)