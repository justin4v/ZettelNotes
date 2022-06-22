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


## 注意
- 最好使用 `${exp:default-value}` 的形式设置默认值，否则可能会由于没有配置属性出现异常
```java
org.springframework.expression.spel.SpelParseException: EL1041E: After parsing a valid expression, there is still more data in the expression: 'lcurly({)'
```

# 参考
1. [Spring Boot SpEL ConditionalOnExpression check multiple properties - Stack Overflow](https://stackoverflow.com/questions/40477251/spring-boot-spel-conditionalonexpression-check-multiple-properties/40497419#40497419)
2. [spring - SpelParseException: After parsing a valid expression, there is still more data in the expression: 'lcurly({)'](https://stackoverflow.com/questions/49714249/spelparseexception-after-parsing-a-valid-expression-there-is-still-more-data-i)