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
- Mock注解实际上是 mockito.mock() 方法的简化；



# 参考
1. [Mockito.mock() vs @Mock vs @MockBean](https://www.baeldung.com/java-spring-mockito-mock-mockbean)