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
	```
	- `org.apache.ibatis.plugin` 涉及到 [[Proxy]] 设计模式
	```java
	// 为 Interceptor 生成动态代理类(InvocationHandler 为 Plugin)
	public static Object wrap(Object target, Interceptor interceptor) {  
	 Map<Class<?>, Set<Method>> signatureMap = getSignatureMap(interceptor);  
	  Class<?> type = target.getClass();  
	  Class<?>[] interfaces = getAllInterfaces(type, signatureMap);  
	  if (interfaces.length > 0) {  
		 // 包装为代理类
		 return Proxy.newProxyInstance(  
		 type.getClassLoader(),  
		        interfaces,  
		        new Plugin(target, interceptor, signatureMap));  
	  }  
	 return target;  
	}
	
	// Plugin 实现了 InvocationHandler 接口，进行动态代理
	// signatureMap实际上是 Interceptor 注解上写明的需要拦截的 <类名, 方法名> 的 Map
	@Override  
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {  
	 try {  
		 Set<Method> methods = signatureMap.get(method.getDeclaringClass());  
	    if (methods != null && methods.contains(method)) { 
			// 执行实际的拦截方法
			 return interceptor.intercept(new Invocation(target, method, args));  
	    }  
		 return method.invoke(target, args);  
	  } catch (Exception e) {  
		 throw ExceptionUtil.unwrapThrowable(e);  
	  }  
	}
	```

	- `com.github.pagehelper.PageInterceptor` 分页拦截器