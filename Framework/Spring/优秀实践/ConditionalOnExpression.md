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
- `${prefetch.config.hl7.enabled:false}` 和 `!${prefetch.config.hl7.standard:true}`


# 参考
1. [Spring Boot SpEL ConditionalOnExpression check multiple properties - Stack Overflow](https://stackoverflow.com/questions/40477251/spring-boot-spel-conditionalonexpression-check-multiple-properties/40497419#40497419)