你是不是也经常在写单元测试时，被数据库连接、第三方接口这些折腾得头疼？明明只是想验证自己的业务逻辑，却不得不花半天时间处理各种外部依赖——这种体验就像是想喝杯咖啡却发现要自己种咖啡豆。

好在Mockito这个神器能让你的测试飞起来！它帮你模拟复杂依赖，让测试回归到代码逻辑本身。无论是验证某个方法是否被正确调用，还是模拟异常来测试程序的健壮性，Mockito 都能让测试变得专注而高效。

# 简介

Mockito是一个用于Java单元测试的mock框架，用于创建**模拟对象**(mock object)来替代真实对象，帮助开发者隔离外部依赖，从而专注于单元测试的逻辑，Mockito通常配合单元测试框架（如JUnit）使用。

- 官方网站：https://site.mockito.org/
- 官方文档：https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html

# 依赖

```xml
<!-- https://mvnrepository.com/artifact/org.mockito/mockito-core -->
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>4.11.0</version>
    <scope>test</scope>
</dependency>
```

如果使用Spring Boot Test 则不需要引入，Spring Boot Test 默认集成了 Mockito。

# 常见用法

Mockito的核心功能包括：

- **创建mock对象**：使用`mock()`创建mock对象。
- **打桩**：使用`when()`和`thenReturn()`等方法指定mock对象的特定方法被调用时的行为（如返回值或抛出异常）。
- **验证行为**：使用`verify()`检查mock对象的特定方法是否被调用，参数和调用次数是否符合预期。

下面通过示例展开介绍Mockito的用法。

## 验证行为

Mockito 的 `verify()` 用于验证**模拟对象的方法是否按预期被调用**，包括调用次数、参数匹配等。它支持精确验证（如 `times(2)`）、最少/最多次数（`atLeast()`/`atMost()`）、未调用（`never()`）及顺序验证（结合 `InOrder`）等，确保代码执行逻辑正确。

```java
public class MockTest {

    @Test
    public void testBasicVerification() {
        List<String> mockList = mock(List.class);

        // 模拟调用
        mockList.add("apple");
        mockList.add("banana");
        mockList.add("apple");
        mockList.add("orange");

        // 1. 验证方法被调用【恰好一次】（默认行为）
        verify(mockList).add("banana");

        // 2. 验证方法被调用【指定次数】
        verify(mockList, times(2)).add("apple");  // 精确2次

        // 3. 验证方法【从未调用】
        verify(mockList, never()).clear();

        // 4. 验证【调用顺序】
        InOrder inOrder = inOrder(mockList);
        inOrder.verify(mockList).add("apple");
        inOrder.verify(mockList).add("banana");
        inOrder.verify(mockList).add("apple");

        verifyNoMoreInteractions(mockList);
    }
    
}
```

`org.mockito.Mockito`类的`mock()`方法用于创建指定类或接口的mock对象。一旦创建，mock对象就会记住所有的方法调用。之后可以选择性地验证感兴趣的方法调用。

- **验证单次调用**：`verify(mockList).add("banana");`→ 检查 `add("banana")` 被调用 ​​1 次​​。

- **验证精确次数**：`verify(mockList, times(2)).add("apple");`→ 检查 `add("apple")` 被调用 ​2 次​​。

- **验证禁止调用**：`verify(mockList, never()).clear();`→ 确保 `clear()` ​从未调用​​。

- **验证调用顺序**：

  ```java
  InOrder inOrder = inOrder(mockList);  
  inOrder.verify(mockList).add("apple");  
  inOrder.verify(mockList).add("banana");  
  inOrder.verify(mockList).add("apple");  
  ```

  严格按顺序验证调用链。
  
- **未验证的调用**：`verifyNoMoreInteractions()` 用来检查mock对象没有未验证的调用。由于`mockList.add("orange")`被调用过，但没有验证，因此最后的测试将会失败。

## 打桩

**打桩**是为模拟对象（Mock）的方法调用预设返回值或行为，使得测试代码可以**隔离外部依赖**，并控制方法的输出或异常，一旦被打桩，方法将返回指定的值，无论调用多少次。通过打桩，可以模拟数据库、网络请求等复杂或不可控的操作。

```java
    @Test
    public void testStubbing() {
        // 1. 创建模拟对象
        List<String> mockList = mock(List.class);

        // 2. 基础打桩：返回固定值
        when(mockList.get(0)).thenReturn("apple");
        assertEquals("apple", mockList.get(0));

        // 3. 抛出异常
        when(mockList.get(1)).thenThrow(new RuntimeException("索引错误"));
        assertThrows(RuntimeException.class, () -> mockList.get(1));

        // 4. 多次调用不同返回值
        when(mockList.size())
                .thenReturn(1)
                .thenReturn(2);
        assertEquals(1, mockList.size());
        assertEquals(2, mockList.size());

        // 5. 参数匹配器（如 anyInt()）
        when(mockList.get(anyInt())).thenReturn("default");
        assertEquals("default", mockList.get(999));

        // 6. Void 方法打桩（如抛出异常）
        doThrow(new IllegalStateException("清空失败")).when(mockList).clear();
        assertThrows(IllegalStateException.class, mockList::clear);
    }
```

**语法优先级**：

- `when(...).thenX()` 适用于有返回值的方法。
- `doX().when(mock).method()` 适用于 void 方法。

**参数匹配器**：使用 `any()`、`eq()` 等灵活匹配参数，但需注意​参数一致性​（不能混用具体值和匹配器）。

**覆盖规则**：最后一次打桩会覆盖之前的定义（例如多次对 `mock.get(0)` 打桩，以最后一次为准）。

**默认情况下，对于所有返回值的方法，mock对象将返回适当的默认值**。例如，对于`int`或`Integer`返回0，对于`boolean`或`Boolean`返回`false`，对于集合类型返回空集合，对于其他对象类型（例如字符串）返回`null`。

## 连续打桩和回调打桩

**连续打桩（Chained Stubbing）**：为同一个方法的连续多次调用定义不同的返回值或行为，常用于模拟多次调用时的动态响应。

```java
    @Test
    public void testChainedStubbing() {
        List<String> mockList = mock(List.class);

        // 定义连续打桩：第一次调用返回 "A"，第二次返回 "B"，第三次抛出异常
        when(mockList.get(0))
                .thenReturn("A")
                .thenReturn("B")
                .thenThrow(new RuntimeException("No more elements"));

        // 验证
        assertEquals("A", mockList.get(0));  // 第一次返回 "A"
        assertEquals("B", mockList.get(0));  // 第二次返回 "B"
        assertThrows(RuntimeException.class, () -> mockList.get(0));  // 第三次抛出异常
    }
```

超出定义的调用次数后，最后一次行为会持续生效（例如第三次后继续调用会一直抛异常）。

**回调打桩（Callback Stubbing）**：`thenAnswer()` 可以实现动态返回值逻辑，根据方法参数或外部条件生成响应。

```java
    @Test
    public void testChainedStubbing() {
        List<String> mockList = mock(List.class);

        // 根据参数动态返回：参数是偶数时返回 "even"，奇数返回 "odd"
        when(mockList.get(anyInt())).thenAnswer(invocation -> {
            int index = invocation.getArgument(0);  // 获取第一个参数
            return (index % 2 == 0) ? "even" : "odd";
        });

        // 验证
        assertEquals("even", mockList.get(0));  //  0是偶数
        assertEquals("odd", mockList.get(1));    // 1是奇数
    }
```

- **灵活控制**：可在 `thenAnswer()` 中编写任意 Java 代码，甚至访问外部变量。
- **参数获取**：通过 `invocation.getArgument(n)` 获取第 `n` 个参数（从 0 开始）。

## 参数匹配器

Mockito默认使用`equals()`方法验证参数值。当需要额外的灵活性时，可以使用参数匹配器。

参数匹配器是 Mockito 提供的一种灵活的参数验证机制，允许开发者通过匹配器来匹配方法参数，而无需指定具体值。

参数匹配器广泛用于 `when()` 打桩和 `verify()` 验证中。

```java
    @Test
    public void testMatchers() {
        List<String> mockList = mock(List.class);

        // 1. 通用匹配器：anyInt(), anyString()
        when(mockList.get(anyInt())).thenReturn("default");
        assertEquals("default", mockList.get(999));

        // 2. 条件匹配器：startsWith(), endsWith()
        when(mockList.add(startsWith("app"))).thenReturn(true);
        assertTrue(mockList.add("apple"));
        assertFalse(mockList.add("banana"));

        // 3. 混合使用具体值和匹配器（必须用 eq() 包裹具体值）
        when(mockList.set(eq(0), anyString())).thenReturn("old_value");
        assertEquals("old_value", mockList.set(0, "new_value"));
    }
```

**通用匹配器**

- **作用**：匹配任意参数或特定类型参数。

- **常见方法**：
- `any()`：匹配任意对象（包括 `null`）。
  
- `anyInt()`, `anyString()`, `anyList()`：匹配特定类型参数。
  
- `isNull()`, `isNotNull()`：匹配 `null` 或非 `null` 参数。

**条件匹配器**

- **作用**：根据逻辑条件匹配参数。

- **常见方法**：

  - `eq(value)`：严格匹配具体值（等同于直接写值）。

  - `startsWith("prefix")`：匹配以指定前缀开头的字符串。

  - `endsWith("suffix")`, `contains("substr")`：匹配字符串后缀或子串。

  - `argThat(condition)`：自定义条件（如集合大小、对象属性）。

**混合使用规则**

- **强制要求**：若方法参数中至少有一个匹配器，则所有参数必须用匹配器。

  错误示例：

  ```java
  // 错误：混合具体值和匹配器
  when(mock.method("value", anyInt())).thenReturn(true);  
  ```

  修复方法：将具体值用 `eq()`包裹：

  ```java
  when(mock.method(eq("value"), anyInt())).thenReturn(true);  
  ```

 **自定义匹配器**

通过 `argThat()` 实现复杂条件：

```java
// 自定义匹配器：验证集合大小大于2
when(mockList.addAll(argThat(list -> list.size() > 2))).thenReturn(true);
assertTrue(mockList.addAll(List.of("A", "B", "C")));
```

更多的内置参数匹配器参考：

- https://javadoc.io/static/org.mockito/mockito-core/4.11.0/org/mockito/ArgumentMatchers.html

- https://javadoc.io/static/org.mockito/mockito-core/4.11.0/org/mockito/hamcrest/MockitoHamcrest.html

## 间谍（spy）

`spy()` 可以创建部分真实对象的代理（保留原有行为，可选择性地对某些方法打桩），适合需要混合真实逻辑与模拟行为的场景。

对比 `mock()`：

|     特性     |               `mock()`               |          `spy()`           |
| :----------: | :----------------------------------: | :------------------------: |
| **默认行为** | 所有方法返回默认值（如 `null`、`0`） | 调用真实方法，除非显式打桩 |
| **适用场景** |         完全隔离被测对象依赖         |  需保留部分真实逻辑的测试  |

```java
    @Test
    public void testSpyBasic() {
        // 1. 创建一个 ArrayList 的 spy 对象
        List<String> spyList = spy(new ArrayList<>());

        // 2. 调用真实方法
        spyList.add("apple");
        spyList.add("banana");

        // 3. 验证真实行为
        assertEquals(2, spyList.size());  // 实际调用了 add 和 size 方法

        // 4. 对某个方法打桩
        when(spyList.size()).thenReturn(100);
        assertEquals(100, spyList.size());  // 打桩生效

        // 5. 验证方法调用次数
        verify(spyList, times(2)).add(anyString()); // 验证 add 被调用两次
    }
```

当对 `spy` 对象的方法打桩时，若直接使用 `when(...)` 会触发真实方法调用，可能导致异常。

错误示例：

```java
List<String> spyList = spy(new ArrayList<>());
// 会被真实执行，但此时列表为空，导致 IndexOutOfBoundsException
when(spyList.get(0)).thenReturn("mock-value");
```

正确方式：使用 `doReturn().when()` 语法避免真实调用

```java
        List<String> spyList = spy(new ArrayList<>());
        // 正确：不会触发 get(0) 的真实调用
        doReturn("mock-value").when(spyList).get(0);
        assertEquals("mock-value", spyList.get(0));
```

**最佳实践**：

1. **优先使用 `mock()`**：除非需要保留部分真实行为，否则优先用 `mock()` 隔离依赖。
2. **谨慎打桩**：使用 `doReturn().when()` 替代 `when().thenReturn()`，避免意外触发真实方法。
3. **避免复杂间谍**：不要对复杂对象（如 Spring Bean）滥用 `spy()`，可能导致测试不可控。

## 参数捕获（ArgumentCaptor）

ArgumentCaptor 用于在测试中捕获方法调用时传递的参数，便于后续对参数值进行详细验证（如对象属性、集合内容等）。

完整示例：

```java
    @Test
    public void testCaptureArgument() {
        // 1. 创建 Mock 对象
        UserService mockService = mock(UserService.class);

        // 2. 调用被测试方法
        User user = new User("Alice", 30);
        mockService.processUser(user);

        // 3. 创建 ArgumentCaptor
        ArgumentCaptor<User> userCaptor = ArgumentCaptor.forClass(User.class);

        // 4. 验证方法调用并捕获参数
        verify(mockService).processUser(userCaptor.capture());

        // 5. 获取捕获的参数并验证
        User capturedUser = userCaptor.getValue();
        assertEquals("Alice", capturedUser.getName());
        assertEquals(30, capturedUser.getAge());
    }

    @Data
    static class User {
        private String name;
        private int age;

        public User(String name, int age) {
            this.name = name;
            this.age = age;
        }
    }

    static class UserService {
        public void processUser(User user) {
            // 实际业务逻辑（在测试中被 Mock）
        }
    }
```

## 静态方法Mock

`Mockito.mockStatic(Class)` 可以创建静态类的 Mock 作用域，并在其中定义行为。

```java
    @Test
    public void testMockStaticMethod() {
        // 1. 创建静态类（如 LocalDate）的 Mock 作用域
        try (MockedStatic<LocalDate> mockedLocalDate = mockStatic(LocalDate.class)) {

            // 2. 定义静态方法 now() 的行为
            LocalDate fixedDate = LocalDate.of(2023, 10, 1);
            mockedLocalDate.when(LocalDate::now).thenReturn(fixedDate);

            // 3. 验证静态方法调用
            assertEquals(fixedDate, LocalDate.now()); // 返回固定日期
            mockedLocalDate.verify(LocalDate::now);     // 验证 now() 被调用
        }

        // 4. 作用域结束后，静态方法恢复原始行为
        assertNotEquals("2023-10-01", LocalDate.now().toString());
    }
```

**作用域限制**：

- 静态 Mock 仅在 `try-with-resources` 或 `MockedStatic.close()` 前有效。
- 必须关闭：确保使用 `try-with-resources` 或手动 `close()`，避免影响其他测试。

# 注解

## @Mock

@Mock用于快速创建 Mock 对象，替代 `Mockito.mock(Class)` 方法。

**方式 1：通过 `MockitoJUnitRunner` 自动初始化**

```java
// 自动初始化 @Mock 注解
@RunWith(MockitoJUnitRunner.class)
public class MockTest {

    @Mock  // 自动创建 List 的 Mock 对象
    private List<String> mockList;

    @Test
    public void testMockAnnotation() {
        mockList.add("test");
        verify(mockList).add("test");
    }


}
```

**JUnit 5 适配**：需使用`@ExtendWith(MockitoExtension.class)`。

**方式 2：手动调用 `MockitoAnnotations.openMocks()`**

```java
public class MockTest {
    @Mock
    private List<String> mockList;

    @Before
    public void init() {
        MockitoAnnotations.openMocks(this);  // 手动初始化 @Mock 注解
    }

    @Test
    public void testMockAnnotation() {
        mockList.add("test");
        verify(mockList).add("test");
    }
}
```

## @MockBean

在Spring Boot 集成测试中，@MockBean用于向 ApplicationContext 注入一个Mock 对象，替换原有 Bean。适用于需要隔离外部依赖（如数据库、第三方服务）的集成测试。

示例场景：测试 `UserService` 时，Mock 其依赖的 `UserRepository`，避免真实数据库操作。

```java
@SpringBootTest  // 启动 Spring 上下文
public class UserServiceTest {

    @Autowired
    private UserService userService;  // 被测服务

    @MockBean  // 自动替换 Spring 容器中的 UserRepository Bean
    private UserRepository userRepository;

    @Test
    public void testGetUserById() {
        // 1. 定义 Mock 行为
        when(userRepository.findById(1L)).thenReturn(new User("Alice"));

        // 2. 调用被测方法
        User user = userService.getUserById(1L);

        // 3. 验证结果和交互
        assertEquals("Alice", user.getName());
        verify(userRepository).findById(1L);  // 确保方法被调用
    }
}
```

- **替换规则**：若 Spring 上下文中已存在同名 Bean，`@MockBean` 会覆盖它；若不存在，则新增 Mock Bean。
- **多 Bean 类型冲突**：若同一类型有多个 Bean，需结合 `@Qualifier` 指定名称。

## @InjectMock

- **核心功能**：自动将 `@Mock` 或 `@Spy` 创建的依赖对象注入到被测试类中，简化依赖管理。

- **适用场景**：单元测试中，快速构建被测试类（如 Service 层），并自动注入其依赖的 Mock 对象（如 Repository）。

示例场景：测试 `UserService`，其依赖 `UserRepository`（需要 Mock）。

```java
@ExtendWith(MockitoExtension.class)
public class MockTest {


    @Mock  // 创建 UserRepository 的 Mock 对象
    private UserRepository userRepository;

    @InjectMocks  // 自动将 userRepository 注入 UserService
    private UserService userService;

    @Test
    public void testGetUserById() {
        // 1. 定义 Mock 行为
        when(userRepository.findById(1L)).thenReturn(new User("Alice"));

        // 2. 调用被测试方法
        User user = userService.getUserById(1L);

        // 3. 验证结果和交互
        assertEquals("Alice", user.getName());
        verify(userRepository).findById(1L);  // 确保方法被调用
    }
}
```

`@InjectMocks` 按以下顺序尝试注入依赖：

1. **构造函数注入**（优先选择参数最多的构造函数）。
2. **Setter 方法注入**（按方法名匹配，如 `setUserRepository()`）。
3. **字段注入**（直接注入到 `private` 字段，需匹配名称和类型）。

# 结尾

Mockito 的魅力在于它用简单的语法解决了测试中的复杂问题。通过模拟对象、打桩预设行为、验证调用细节，开发者可以轻松隔离外部依赖，像搭积木一样构造测试场景。无论是新手还是经验丰富的工程师，Mockito 的直观设计都能让人快速上手。

下次当你面对一个难以测试的方法时，不妨试试 Mockito——让它帮你把“不确定”变成“可控”，把“复杂依赖”变成“精准验证”。毕竟，好的测试不是为了证明代码完美，而是为了让它足够可靠，而 Mockito 正是这条路上值得信赖的工具。
