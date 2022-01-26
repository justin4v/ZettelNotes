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

		  // 责任链 execute 方法；实际上是包装为 代理类
		  default Object plugin(Object target) {  
			 return Plugin.wrap(target, this);  
		  }  
		  
		 default void setProperties(Properties properties) {  
			 // NOP  
		 }  
	}

	public static Object wrap(Object target, Interceptor interceptor) {  
	 Map<Class<?>, Set<Method>> signatureMap = getSignatureMap(interceptor);  
	  Class<?> type = target.getClass();  
	  Class<?>[] interfaces = getAllInterfaces(type, signatureMap);  
	  if (interfaces.length > 0) {  
		// 代理类
		// Plugin 实现了 InvocationHandler
		 return Proxy.newProxyInstance(  
		 type.getClassLoader(),  
		        interfaces,  
		        new Plugin(target, interceptor, signatureMap));  
	  }  
	 return target;  
	}

	```
