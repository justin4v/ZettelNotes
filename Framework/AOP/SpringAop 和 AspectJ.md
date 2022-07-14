#Spring  #AspectJ #Todo 

# 四、Spring AOP和Aspectj的区别

-   Spring AOP
    
    -   `Spring AOP是在运行期间通过代理生成目标类，属于动态代理。默认如果使用接口的，用JDK动态代理实现，如果没有接口则使用CGLIB实现`
    -   Spring AOP致力于解决企业级开发中最普遍的AOP（方法织入），仅仅是`方法织入`
    -   Spring AOP需要`依赖IOC容器`来管理，并且只能作用于Spring容器，使用纯Java代码实现
    -   在性能上，由于Spring AOP是基于动态代理来实现的，在容器启动时需要生成代理实例，在方法调用上也会增加栈的深度，使得Spring AOP的`性能较`AspectJ的`差`
    -   Spring AOP`支持注解`，在使用@Aspect注解创建和配置切面时将更加方便。
-   AspectJ
    
    -   AspectJ是在编译期间将切面代码编译到目标代码的，属于静态代理，通过修改代码来实现，有如下几个织入的时机：
        -   编译期织入（Compile-time weaving）： 如类 A 使用 AspectJ 添加了一个属性，类 B 引用了它，这个场景就需要编译期的时候就进行织入，否则没法编译类 B。
        -   编译后织入（Post-compile weaving）： 也就是已经生成了 .class 文件，或已经打成 jar 包了，这种情况我们需要增强处理的话，就要用到编译后织入。
        -   类加载后织入（Load-time weaving）： 指的是在加载类的时候进行织入，要实现这个时期的织入，有几种常见的方法。
            -   1、自定义类加载器来干这个，这个应该是最容易想到的办法，在被织入类加载到 JVM 前去对它进行加载，这样就可以在加载的时候定义行为了。
            -   2、在 JVM 启动的时候指定 AspectJ 提供的 agent：`-javaagent:xxx/xxx/aspectjweaver.jar`。
    -   AspectJ可以做Spring AOP干不了的事情，它是AOP编程的完全解决方案，AspectJ支持`所有切入点`，不仅仅是方法织入。
    -   因为AspectJ在实际运行之前就完成了织入，所以说它生成的类是`没有额外运行时开销`的
    -   需要通过.aj文件来创建切面，并且需要使用ajc（Aspect编译器）来编译代码。


| Spring AOP                 | AspectJ                                |
| -------------------------- | -------------------------------------- |
| 在纯 Java 中实现                | 使用 Java 编程语言的扩展实现                      |
| 不需要单独的编译过程                 | 除非设置 LTW，否则需要 AspectJ 编译器 (ajc)        |
| 只能使用运行时织入                  | 运行时织入不可用。支持编译时、编译后和加载时织入               |
| 功能不强-仅支持方法级编织              | 更强大 - 可以编织字段、方法、构造函数、静态初始值设定项、最终类/方法等。 |
| 只能在由 Spring 容器管理的 bean 上实现 | 可以在所有域对象上实现                            |
| 仅支持方法执行切入点                 | 支持所有切入点                                |
| 代理是由目标对象创建的, 并且切面应用在这些代理上  | 在执行应用程序之前 (在运行时) 前, 各方面直接在代码中进行织入      |
| 比 AspectJ 慢多了              | 更好的性能                                  |
| 易于学习和应用                    | 相对于 Spring AOP 来说更复杂                   |


## 误区

-   `曾经以为`AspectJ是Spring AOP一部分，是因为Spring AOP使用了AspectJ的Annotation。
    -   使用了Aspect来定义切面,使用Pointcut来定义切入点，使用Advice来定义增强处理。
-   `Spring AOP虽然使用了Aspect的Annotation（@Aspect，@Pointcut，@Before等），但是并没有使用它的编译器和织入器。其实现原理是JDK动态代理和CGLIB，在运行时生成代理类。`
-   为了启用 Spring 对 @AspectJ 方面配置的支持，并保证 Spring 容器中的目标 Bean 被一个或多个方面自动增强，必须在 Spring 配置文件中添加如下配置：<aop:aspectj-autoproxy/>，在高版本的springboot中已默认开启支持，并不需要配置。

  
# jdk动态代理，cglib，Spring AOP和Aspectj关系

-   Spring AOP和Aspectj是两种实现aop的框架
-   Spring AOP采用的是动态代理
    -   动态代理有两种底层技术实现：
        -   jdk动态代理（默认有接口的目标类使用jdk动态代理）
        -   cglib（没有接口或有接口的目标类使用）
    -   Spring AOP采用了Aspectj包提供的注解，但是底层编译器和织入器并不是Aspectj
-   Aspectj采用的是静态代理


# 参考
1. [🚀一文搞懂：jdk动态代理,cglib,Spring AOP和Aspectj](https://juejin.cn/post/7042484603365359646#heading-15)