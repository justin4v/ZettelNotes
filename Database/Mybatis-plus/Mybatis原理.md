# Mybatis 原理

## 示例源代码

```
MtConfiguration configuration = new MtConfiguration("mybatis-config.properties");
MtSqlSessionFactoryBuilder sqlSessionFactoryBuilder = new MtSqlSessionFactoryBuilder(configuration);
MtSqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build();
MtSqlSession sqlSession = sqlSessionFactory.openSession();
PersonDao personDao = sqlSession.getMapper(PersonDao.class);
Person person = personDao.queryPersonById(1l);
```

## 解释

### 第1步 MtConfiguration

1. 从文件中读取属性
2. 加载xml文件
3. 加载DataSource
4. 加载Mapper文件 注册Mapper
5. 解析加载 Plugin Chain
6. 加载TypeHandler

```
 //记载配置文件，这里使用properties代替xml解析
 loadConfigProperties();
 //初始化数据源信息
 initDataSource();
 //解析并加载mapper文件
 loadMapperRegistory();
 //解析加载plugin
 initPluginChain();
 //解析加载typeHandler
 initTypeHandler();
```



其中 解析加载plugin 是 interceptorChain.addInterceptor(class) 。 class从配置文件中读取，通过反射获得对象。本代码示例中为 ExecutorLogPlugin（ implements MtInterceptor）



### 第2步 MtSqlSessionFactoryBuilder

通过configuration 获得 SqlSessionFactoryBuilder



### 第3步 sqlSessionFactoryBuilder.build

建造者模式 构建 MtSqlSessionFactory



### 第4步 sqlSessionFactory.openSession

获得 sqlsession

调用 openSessionFromDataSource -> configuration.newExecutor 获得executor -> new MtSqlSession(configuration, executor) 得到sqlSession 返回。



newExecutor 过程：

```
public MtExecutor newExecutor(ExecutorType type){
        MtExecutor executor = null;

        if(ExecutorType.SIMPLE==type){
            executor = new MtSimpleExecutor(this);
        }

        return (MtExecutor)this.interceptorChain.pluginAll(executor);
    }
```

其中 pluginAll 是在 Executor （MtSimpleExecutor）上进行增强代理（interceptorChain中所有 interceptor） 。本例中将interceptorChain 中MtInterceptor 增加到 executor中。**返回 Executor**



MtInterceptorChain 的 pluginAll

```
public Object pluginAll(Object target){
    for(MtInterceptor var1 : interceptors){
    target = var1.plugin(target);
    }
    return target;
}
```



ExecutorLogPlugin 中 plugin

```
@Override
public Object plugin(Object var1) {
	return MtPlugin.wrap(var1,this);
}
```

调用了 MtPlugin的 wrap静态方法   

```
public static Object wrap(Object target,MtInterceptor interceptor){
        return Proxy.newProxyInstance(target.getClass().getClassLoader(),new Class[]{MtExecutor.class},new MtPlugin(target,interceptor));
    }
```

该方法生成一个 **由 MtPlugin 作为handler（调用 代理接口方法时 实际调用 MtPlugin 中 invoke方法），实现 MtExecutor 接口的 代理对象 [ new MtPlugin(target,interceptor) ]**  

**new MtPlugin(target,interceptor)** 创建 MtPlugin 对象，**interceptor参数为当前对象 ExecutorLogPlugin** **target为目标 （executor）**

返回



### 第5步 sqlSession.getMapper(PersonDao.class)

首先 getMapper  (MtSqlSession)

```
public <T> T getMapper(Class<T> clazz){
        return (T) Proxy.newProxyInstance(this.getClass().getClassLoader(),new Class[]{clazz},new MtMapperProxy(this,clazz));
    }
```

生成一个 **由 MtMapperProxy 作为handler（调用代理接口的方法时 实际调用 MtMapperProxy 中 invoke方法），实现 clazz的（本例中为 PersonDao ） 接口的 代理对象 [ new MtMapperProxy(this,clazz) ]**   

**new MtMapperProxy(this,clazz)** 创建 MtMapperProxy 对象 ,**this为当前对象 MtSqlSession** **clazz为Mapper接口**

 返回



### 第6步 执行 personDao.queryPersonById(1)

流程如下：

1. 执行 Mapper 接口 PersonDao中方法 ；

2. 由于此时已经是第五步生成的代理类，将会执行 handler--**MtMapperProxy** **对象**  中invoke方法。本例中实际执行代理对象的  **return this.mtSqlSession.selectOne(mapperData,args);** 

3. 跳转执行 mtSqlSession 对象的 selectOne：  return executor.query(mapperData, parameter); 该Executor在第4步经过plugin 增强 代理后的 SimpleExecutor，代理类为 MtPlugin。

4. 实际执行 MtPlugin 对象 中  invoke：  return this.interceptor.intercept(new MtInvocation(target,method,args));  其中 MtPlugin 对象在第4步代理时新建。Interceptor从第4步知 为ExecutorLogPlugin **target为目标 （executor）**  method为 query

5. 跳转  ExecutorLogPlugin  intercept（MtInvocation invocation）   代码中调用  invocation.proceed(); 实际调用 MtInvocation 的 proceed（）  -- return this.method.invoke(this.target, this.args); 

6. 实际调用  MtSimpleExecutor 的query 方法

   ```
   @Override
       public <T> T query(MapperData mapperData, Object parameter) throws Exception {
           StatementHandler statementHandler = new StatementHandler(configuation);
           return statementHandler.query(mapperData,parameter);
       }
   ```

   最后执行 SQL语句





## 其他补充



Mapper执行的过程是通过Executor、StatementHandler、ParameterHandler和ResultHandler来完成数据库操作和结果返回的，理解他们是编写插件的关键：

- **Executor**：执行器，由它统一调度其他三个对象来执行对应的SQL；
- **StatementHandler**：使用数据库的Statement执行操作；
- **ParameterHandler**：用于SQL对参数的处理；
- **ResultHandler**：进行最后数据集的封装返回处理；



### 插件

在MyBatis中使用插件，需要实现Interceptor接口，定义如下：

```
public interface Interceptor {
  Object intercept(Invocation invocation) throws Throwable;
  
  Object plugin(Object target);
  
  void setProperties(Properties properties);
}
```

详细说说这3个方法：

- intercept：它将直接覆盖你所拦截的对象。有个参数Invocation对象，通过该对象，可以反射调度原来对象的方法；
- plugin：target是被拦截的对象，它的作用是给被拦截对象生成一个代理对象；
- setProperties：允许在plugin元素中配置所需参数，该方法在插件初始化的时候会被调用一次；




Plugin提供了静态方法wrap方法，它会根据插件的签名配置，使用JDK动态代理的方法，生成一个代理类，当四大对象执行方法时，会调用Plugin的invoke方法，如果方法包含在声明的签名里，就会调用自定义插件的intercept方法，传入Invocation对象。

另外，Invocation对象包含一个proceed方法，这个方法就是调用被代理对象的真实方法（调用 method.invoke ）。如果只有一个个插件，第一个传递的参数是四大对象本身，然后调用一次wrap方法产生第一个代理对象。如果有第二个插件，这里的反射调用的是第一个代理对象的 invoke 方法。

所以，在多个插件的情况下，调度proceed方法，MyBatis总是从**最后一个代理对象方法运行到第一个代理对象，最后是真实被拦截的对象方法被执行**。



