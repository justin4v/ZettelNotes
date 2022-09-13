#SpEL #Spring #Todo 

# 来源
- The Spring Expression Language（SpEL） 表达式的创建是为了向 Spring 社区提供一种受良好支持的表达式语言，该语言适用于 Spring 家族中的所有产品。
- 其语言特性由 Spring 家族中的项目需求驱动，包括基于eclipse的SpringSource套件中的代码补全工具需求。
- SpEL 并不与 Spring 直接相关，可以被独立使用。SpEL 是一种与技术无关的 API，可以集成其它表达式语言。



# 注意
- SpEL 支持动态注入与更新属性设置，如 `${external.redis.enble}`。
- nacos中增加 `@RefreshScope` 使得动态更新生效
# 参考
1. [Spring Expression Language Guide | Baeldung](https://www.baeldung.com/spring-expression-language)
2. [8. Spring 表达式语言 (SpEL) ](https://itmyhome.com/spring/expressions.html)