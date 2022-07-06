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
- @MockBean 注解将 Mock 对象*添加到 Spring 上下文*中。
- Spring context 中同类型的原有 Bean 会被替换，如果没有同类型的 Bean 会新建；
- 如果需要用 @Mockbean 注解，需要用:
	-  *@Runwith(SpringRunner.class);
	- Junit5 中是 *@Runwith(SpringExtention.class)*;
	- Springboot 中用 *@SpringBootTest*。

```java
@RunWith(SpringExtention.class)
public class MockBeanAnnotationIntegrationTest {
    
    @MockBean
    UserRepository mockRepository;
    
    @Autowired
    ApplicationContext context;
    
    @Test
    public void givenCountMethodMocked_WhenCountInvoked_ThenMockValueReturned() {
        Mockito.when(mockRepository.count()).thenReturn(123L);

        UserRepository userRepoFromContext = context.getBean(UserRepository.class);
        long userCount = userRepoFromContext.count();

        Assert.assertEquals(123L, userCount);
        Mockito.verify(mockRepository).count();
    }
}
```


# mock 静态方法
mock `StaticUtils.range()`
```java
@Test
void givenStaticMethodWithArgs_whenMocked_thenReturnsMockSuccessfully() {
    assertThat(StaticUtils.range(2, 6)).containsExactly(2, 3, 4, 5);

    try (MockedStatic<StaticUtils> utilities = Mockito.mockStatic(StaticUtils.class)) {
        utilities.when(() -> StaticUtils.range(2, 6))
          .thenReturn(Arrays.asList(10, 11, 12));

        assertThat(StaticUtils.range(2, 6)).containsExactly(10, 11, 12);
    }

    assertThat(StaticUtils.range(2, 6)).containsExactly(2, 3, 4, 5);
}
```
# 参考
1. [Mockito.mock() vs @Mock vs @MockBean](https://www.baeldung.com/java-spring-mockito-mock-mockbean)
2. [Mocking Static Methods With Mockito](https://www.baeldung.com/mockito-mock-static-methods#:~:text=As%20previously%20mentioned%2C%20since%20Mockito%203.4.0%2C%20we%20can,our%20type%2C%20which%20is%20a%20scoped%20mock%20object.)