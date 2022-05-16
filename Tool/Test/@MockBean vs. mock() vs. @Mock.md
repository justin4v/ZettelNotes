#Mock #Test #Spring 


# Mockito.mock()
- mock() 方法可以创建类或接口的模拟对象。
- 可用 mock 来指定方法的行为，并验证它们是否被调用。

```java
@Test
public void givenCountMethodMocked_WhenCountInvoked_ThenMockedValueReturned() {
	// mock user service
    UserRepository localMockRepository = Mockito.mock(UserRepository.class);
    Mockito.when(localMockRepository.count()).thenReturn(111L);

    long userCount = localMockRepository.count();

    Assert.assertEquals(111L, userCount);
    // verify called or not
    Mockito.verify(localMockRepository).count();
}
```
- 可在类中 Mock字段或者在函数中 Mock 局部变量

# @Mock
- *Mock 注解实际上是 mockito.mock() 简化*；
- 与 mock() 方法不同，使用前需要手动启用（@Runwith _MockitoJUnitRunner_ 或调用 _MockitoAnnotations.initMocks()_）

```java
@RunWith(MockitoJUnitRunner.class)
public class MockAnnotationUnitTest {
    
    @Mock
    UserRepository mockRepository;
    
    @Test
    public void givenCountMethodMocked_WhenCountInvoked_ThenMockValueReturned() {
        Mockito.when(mockRepository.count()).thenReturn(123L);

        long userCount = mockRepository.count();

        Assert.assertEquals(123L, userCount);
        Mockito.verify(mockRepository).count();
    }
}
```

- 相较 mock()，@Mock 注解使代码更易读，在故障时更容易找到问题。


# @MockBean
- @MockBean 注解将 Mock对 象*添加到 Spring 上下文*中。


# 参考
1. [Mockito.mock() vs @Mock vs @MockBean](https://www.baeldung.com/java-spring-mockito-mock-mockbean)