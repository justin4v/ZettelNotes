#Test #Spring #Mock 

# 比较
- `@MockBean` 只能 mock 本地的代码——或者说是自己写的代码，对于三方库中又以 Bean 的形式装配的类无能为力；
- `@SpyBean` 不会生成一个 Bean 的替代品装配到类中；
- 而是会*监听一个真正的 Bean 中某些特定的方法*，并在调用这些方法时给出指定的反馈。
- *不会影响 Bean 其它的功能*
- `@SpyBean` 与 `@Spy` 的关系类似于 `@MockBean` 与 `@Mock` 的关系；

# SpyBean


# 参考
1. [使用 @MockBean 和 @SpyBean 解决 SpringBoot 单元测试中 Mock 类装配的问题 ](https://sspai.com/post/48245)
2. [Springboot单元测试:SpyBean vs MockBean](https://juejin.cn/post/6881981078735699976)