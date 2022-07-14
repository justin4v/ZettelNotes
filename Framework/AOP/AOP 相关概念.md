#AOP #AspectJ 

# 切面(Aspect)

-   切面是一个`横切关注点的模块化`，一个切面能够包含同一个类型的不同增强方法，比如说事务处理和日志处理可以理解为两个切面。
-   切面由`切入点`和`通知`组成，它既包含了横切逻辑的定义，也包括了切入点的定义。
-   Spring AOP就是负责实施切面的框架，它将切面所定义的横切逻辑织入到切面所指定的连接点中。

```java
@Component
@Aspect
public class LogAspect {
}
```

# 目标对象(Target)

-   目标对象指将要`被增强的对象`，即包含主业务逻辑的类对象。或者说是被一个或者多个切面所通知的对象

# 连接点（JoinPoint）

-   程序执行过程中明确的点，如`方法的调用`或特定的异常被抛出。连接点由两个信息确定：
    -   方法(表示程序执行点，即在哪个目标方法)
    -   相对点(表示方位，即目标方法的什么位置，比如调用前，后等)
-   简单来说，`连接点就是被拦截到的程序执行点`，因为Spring aop`只支持方法类型的连接点`，所以在Spring中连接点就是被拦截到的方法。

```java
@Before("pointcut()")
public void log(JoinPoint joinPoint) { //这个JoinPoint参数就是连接点
}
```

# 切入点(PointCut)

-   `切入点是对连接点进行拦截的条件定义`。切入点表达式如何和连接点匹配是AOP的核心，Spring缺省使用AspectJ切入点语法。
-   一般认为，所有的方法都可以认为是连接点，但是我们并不希望在所有的方法上都添加通知，而`切入点的作用就是提供一组规则(使用 AspectJ pointcut expression language 来描述) 来匹配连接点`，给满足规则的连接点添加通知。

1.  切入点的匹配规则是 `com.ljw.test.aop.service`包下的所有类的所有函数。

```java
@Pointcut("execution(* com.ljw.test.aop.service..*.*(..))")
public void Pointcut() {}
```

整个表达式可以分为四个部分:

-   第一个*号：表示返回类型， *号表示所有的类型
-   包名: 表示需要拦截的包名，后面的..表示当前包和当前包的所有子包,com.ljw.test.aop.service包、子孙包下所有类的方法。
-   第二个*号: 表示类名,*号表示所有的类。
-   *(..):最后这个星号表示方法名, *号表示所有的方法,后面括弧里面表示方法的参数，两个句点表示任何参数

2.  com.ljw.controller包中所有的类的所有方法切面

```java
 @Pointcut("execution(public * com.ljw.controller.*.*(..))")
```

3.  只针对 UserController 类切面

```java
@Pointcut("execution(public * com.ljw.controller.UserController.*(..))")
```

4.  统一切点,对com.ljw及其子包中所有的类的所有方法切面

```java
@Pointcut("execution(* com.ljw.controller.*.*(..))")
```

# 通知(Advice)

-   通知是指拦截到连接点之后`要执行的代码`，包括了around、before和after等不同类型的通知。
-   Spring AOP框架以拦截器来实现通知模型，并维护一个以连接点为中心的拦截器链。

```java
// @Before说明这是一个前置通知，log函数中是要前置执行的代码，JoinPoint是连接点，
@Before("pointcut()")
public void log(JoinPoint joinPoint) { 
}
```

#  织入(Weaving)

-   织入是将切面和业务逻辑对象连接起来, 并创建通知代理的过程。
-   织入可以在编译时，类加载时和运行时完成。
-   在编译时进行织入就是静态代理，而在运行时进行织入则是动态代理。

# 增强器(Advisor)

-   Advisor是切面的另外一种实现，能够将通知以更为复杂的方式织入到目标对象中，是将通知包装为更复杂切面的装配器。
-   Advisor由切入点和Advice组成。
-   Advisor这个概念来自于Spring对AOP的支撑，在AspectJ中是没有等价的概念的。Advisor就像是一个小的自包含的切面，这个切面只有一个通知。切面自身通过一个Bean表示，并且必须实现一个默认接口。

```java
// AbstractPointcutAdvisor是默认接口
public class LogAdvisor extends AbstractPointcutAdvisor {
 private Advice advice; // Advice
 private Pointcut pointcut; // 切入点

 @PostConstruct
 public void init() {
 // AnnotationMatchingPointcut是依据修饰类和方法的注解进行拦截的切入点。
 this.pointcut = new AnnotationMatchingPointcut((Class) null, Log.class);
 // 通知
 this.advice = new LogMethodInterceptor();
 }
}
```

# AOP常用注解

-   @Aspect ：作用是把当前类标识为一个切面供容器读取
-   @Pointcut ： 切入点是对连接点进行拦截的条件定义，在程序中主要体现为书写切入点表达式
-   @Before ：标识一个前置增强方法，相当于BeforeAdvice的功能
-   @AfterReturning ：后置增强，相当于AfterReturningAdvice，方法退出时执行
-   @AfterThrowing ：异常抛出增强，相当于ThrowsAdvice
-   @After ：final增强，不管是抛出异常或者正常退出都会执行
-   @Around ：环绕增强，相当于MethodInterceptor


# AOP执行顺序

执行成功:

![[AOP执行顺序-正常情况]]



执行失败（`注意没有Around后`）:

  



# 参考
1. [【线上排查实战】AOP切面执行顺序你真的了解吗](https://segmentfault.com/a/1190000037558963)
2. 