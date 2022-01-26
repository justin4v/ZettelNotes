#Mybatis 
# 拦截器
- `org.apache.ibatis.session.addInterceptor(Interceptor)` 增加拦截器;
-  `org.apache.ibatis.plugin.InterceptorChain` 采用 [[Chain of Responsibility]] 设计模式
	- 使用 `List<org.apache.ibatis.plugin.Interceptor> interceptors` 保存 chain；
	- `org.apache.ibatis.plugin.Interceptor` 
