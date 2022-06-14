#Proxy #Spring #AOP 

# 内部方法调用AOP不生效的问题
- Spring AOP中*内部方法调用，切面逻辑是不生效*。

```java
//事务注解
@Transactional(rollbackFor = Exception.class)
@Override  
public Result create(Api api) {  
    saveApi(api);    
    return Result.OK();  
}

// public 方法
public void saveApi(Api api){
    //数据库操作
	sqlSave(api);
}
```

- `@Transactional` 原理是 [[动态代理]]，实际调用是*通过代理类调用*； 
- 在当前方法中，调用同类中定义的 saveApi() *实际代码为 `this.saveApi(api)`*，**不是通过代理类调用**，无法被代理。

# 解决方案
## 移动到另一个类
- 业务逻辑梳理，是否可新增一层如：*事务层*；
- 将*调用方法移动到另外的类*，然后在本类中注入该服务，然后调用；


使*同类调用时切面依然生效*，有以下三个方法：

## AopContext 获取当前类
 - 使用 `AopContext#currentProxy()`，获取当前类的代理对象，然后调用方法：
```java
@Transactional(rollbackFor = Exception.class)
@Override  
public Result create(Api api) {
	// 获取当前类的代理对象
	ApiServiceImpl service = (ApiService)AopContext.currentProxy();
	service.saveApi(api);    
	return Result.OK();  
}
```
- 需要在*启动类加上 @EnableAspectJAutoProxy(exposeProxy = true)*
- *exposeProxy = true* 用于公开AOP框架代理，公开后才可以通过AopContext获取到当前代理类。
- 默认情况下不会公开代理，因为会降低性能；
- **不能保证这种方式一定有效**，使用 *@Async* 时本方式可能失效。


## @Autowired 注入自身

在当前类中获取自身的 Bean 实例，再用 Bean 进行内部调用：
```java
@Service
public class A{

  // 或者使用 ApplicationContext.getBean() 获取本class bean实例
  //拿到代理类
  @Autowired
  private A self;

  public void a(){
    //通过代理类来调用方法
    self.b();
    self.c();
  }
}
```
- 一般不会有循环依赖的问题，spring 通过**三级缓存**基本解决了循环依赖问题。
- 但是 [[@Async 导致循环依赖问题|@Async 仍然有可能导致循环依赖]]。

## 通过 spring 上下文获取到当前代理类
*推荐方法*

```java
// Spring 上下文获取当前代理类
@Component
public class SpringBeanUtil implements ApplicationContextAware {

    private static ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }

    /**
     * 通过class获取Bean
     * @param clazz class
     * @param <T> 泛型
     * @return bean
     */
    public static <T> T getBean(Class<T> clazz) {
        return applicationContext.getBean(clazz);
    }
}



@Service
public class StudentServiceImpl implements StudentService {

    @Autowired
    private StudentMapper studentMapper;
    @Override
    public void insertStudent(){
        StudentService bean = SpringBeanUtil.getBean(StudentService.class);
        if (null != bean) {
		    // 调用同类中有事务注解的方法
            bean.insert();
        }
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public void insert() {
        StudentDO studentDO = new StudentDO();
        studentDO.setName("小民");
        studentDO.setAge(22);
        studentMapper.insert(studentDO);

        if (studentDO.getAge() > 18) {
            throw new RuntimeException("年龄不能大于18岁");
        }
    }
}
```


# 参考
1. [[动态代理]]
2. [spring同类调用事务不生效-原因及三种解决方式 ](https://www.jianshu.com/p/083605986c8f)