#Spring #Springboot #Best-practice 

# 用途
**多属性条件注入**
```java
@Component  
@ConditionalOnExpression("${prefetch.config.hl7.enabled:false} && !${prefetch.config.hl7.standard:true}")  
public class HL7NonStandardReceiver implements DisposableBean, ApplicationRunner {
......
}
```
- `${prefetch.config.hl7.enabled:false}` 和 `!${prefetch.config.hl7.standard:true}` 为两个条件。
- 条件中使用 `&&` 连接。

## ApplicationRunner
- java docs
```java
1. Interface used to indicate that a bean should run when it is contained within a SpringApplication.
2. Multiple ApplicationRunner beans can be defined within the same application context and can be ordered using the Ordered interface or @Order annotation
```
- 用于 Bean 初始化注入后自动执行代码，作为 *hook*；

## DisposableBean
- java docs
```java
1. Interface to be implemented by beans that want to release resources on destruction. A BeanFactory will invoke the destroy method on individual destruction of a scoped bean. 
2. An org.springframework.context.ApplicationContext is supposed to dispose all of its singletons on shutdown, driven by the application lifecycle.
3. A Spring-managed bean may also implement Java's AutoCloseable interface for the same purpose. An alternative to implementing an interface is specifying a custom destroy method, for example in an XML bean definition. For a list of all bean lifecycle methods, see the BeanFactory javadocs.
```
- bean 销毁时释放资源
## 注意
- 最好使用 `${exp:default-value}` 的形式设置默认值，否则可能会由于没有配置属性出现异常
```java
org.springframework.expression.spel.SpelParseException: EL1041E: After parsing a valid expression, there is still more data in the expression: 'lcurly({)'
```

# 参考
1. [Spring Boot SpEL ConditionalOnExpression check multiple properties - Stack Overflow](https://stackoverflow.com/questions/40477251/spring-boot-spel-conditionalonexpression-check-multiple-properties/40497419#40497419)
2. [spring - SpelParseException: After parsing a valid expression, there is still more data in the expression: 'lcurly({)'](https://stackoverflow.com/questions/49714249/spelparseexception-after-parsing-a-valid-expression-there-is-still-more-data-i)