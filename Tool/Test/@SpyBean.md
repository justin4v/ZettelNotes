#Test #Spring #Mock 

# 问题
- 期望测试 `TestService` 的 `test` 方法;
- 但是 `doSomething` 执行很复杂。 
- 希望 `test` 方法真实执行，而为 `doSomething` 方法打桩。

```java
package com.example.demo.service;

import com.example.demo.repositroy.TestRepository;
import org.springframework.stereotype.Component;

@Component
public class TestService {
    private final TestRepository testRepository;

    public TestService(TestRepository testRepository) {
        this.testRepository = testRepository;
    }

    public String doSomething(){
        //假装有复杂的无法执行的业务逻辑
        testRepository.doSomething();
        throw new RuntimeException();
    }

    public String test() {
        doSomething();
        //其他逻辑
        return "id";
    }
}
```

# 解决
## MockBean

- `MockBean` 会替换目标对象，其中所有方法全部 mock， `test` 不能真实地被执行。
- 其中的  `when...thenCallRealMethod` 可达到部分 mock 的效果，仅`test`方法真实执行。

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class TestServiceTest {
    @MockBean
    TestService testService;

    @Test
    public void test(){
        Mockito.when(testService.test()).thenCallRealMethod();
        assertThat(testService.test(), equalTo("id"));
    }
}
```

# SpyBean
- `@SpyBean`修饰的`testService`是一个真实对象，仅当`Mockito.doReturn("").when(testService).doSomething()`时，`doSomething`方法被打桩，其他的方法仍被真实调用。

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class TestServiceTest {
    @SpyBean
    TestService testService;

    @Test
    public void test(){
        doReturn("").when(testService).doSomething();
        assertThat(testService.test(), equalTo("id"));
    }
}
```

## 小陷阱

- 与使用`@MockBean`不同，调用`Mockito.doReturn("").when(testService).doSomething()` 时`doSomething`方法被打桩。
- 而 `Mockito.when(testService.doSomething()).thenReturn("")` 则不会。
- 因为使用 `@SpyBean` 修饰的 `testService` 是一个真实对象，所以 `testService.doSomething()` 会真实调用 `doSomething()` 方法。

-   spy修饰的变量，Mockito会重新创建一个实例的copy，并不直接作用于真实实例
-   spy对final方法无效

  # 比较
- `@MockBean` 只能 mock 本地的代码——或者说是自己写的代码，对于三方库中又以 Bean 的形式装配的类无能为力；
- `@SpyBean` 不会生成一个 Bean 的替代品装配到类中；
- 而是会*监听一个真正的 Bean 中某些特定的方法*，并在调用这些方法时给出指定的反馈。
- *不会影响 Bean 其它的功能*
- `@SpyBean` 与 `@Spy` 的关系类似于 `@MockBean` 与 `@Mock` 的关系；


# 参考
1. [使用 @MockBean 和 @SpyBean 解决 SpringBoot 单元测试中 Mock 类装配的问题 ](https://sspai.com/post/48245)
2. [Springboot单元测试:SpyBean vs MockBean](https://juejin.cn/post/6881981078735699976)