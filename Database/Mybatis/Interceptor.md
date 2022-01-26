#Mybatis 
# 拦截器
- `org.apache.ibatis.session.addInterceptor(Interceptor)` 增加拦截器;
-  `org.apache.ibatis.plugin.InterceptorChain` 采用 [[Chain of Responsibility]] 设计模式
	- 使用 `List<org.apache.ibatis.plugin.Interceptor> interceptors` 保存 chain；
	- `org.apache.ibatis.plugin.Interceptor` 
	```java
	package org.apache.ibatis.plugin;  
	import java.util.Properties;  
	  
	/**  
	 * @author Clinton Begin  
	 */
	public interface Interceptor {  
	     // 拦截入口
		 Object intercept(Invocation invocation) throws Throwable;  

		  // 责任链 execute 方法
		  default Object plugin(Object target) {  
			 return Plugin.wrap(target, this);  
		  }  
		  
		 default void setProperties(Properties properties) {  
			 // NOP  
		 }  
	}
	```
