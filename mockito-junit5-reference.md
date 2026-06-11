# JUnit 5 & Mockito Cheatsheet

---

## Table of Contents
1. [JUnit 5 Annotations](#junit-5-annotations)
2. [JUnit 5 Assertions](#junit-5-assertions)
3. [JUnit 5 Assumptions](#junit-5-assumptions)
4. [JUnit 5 Test Lifecycle](#junit-5-test-lifecycle)
5. [JUnit 5 Parameterized Tests](#junit-5-parameterized-tests)
6. [JUnit 5 Dynamic Tests](#junit-5-dynamic-tests)
7. [Mockito Setup & Annotations](#mockito-setup--annotations)
8. [Mockito Core Methods](#mockito-core-methods)
9. [Argument Matchers](#argument-matchers)
10. [Verification](#verification)
11. [Stubbing](#stubbing)
12. [Spy](#spy)
13. [Captor](#captor)
14. [Exception Handling](#exception-handling)
15. [Mockito Answer](#mockito-answer)
16. [InOrder Verification](#inorder-verification)
17. [Common Patterns](#common-patterns)

---

## JUnit 5 Annotations

| Annotation | Description |
|---|---|
| `@Test` | Marks a method as a test method |
| `@DisplayName("name")` | Custom display name for a test |
| `@Disabled("reason")` | Disables a test or class |
| `@BeforeEach` | Runs before **each** test method |
| `@AfterEach` | Runs after **each** test method |
| `@BeforeAll` | Runs **once** before all tests (must be `static`) |
| `@AfterAll` | Runs **once** after all tests (must be `static`) |
| `@Nested` | Denotes a nested, non-static test class |
| `@Tag("tag")` | Tags a test for filtering/grouping |
| `@Timeout(5)` | Fails test if it exceeds N seconds |
| `@RepeatedTest(n)` | Repeats a test N times |
| `@ParameterizedTest` | Marks a parameterized test |
| `@TestMethodOrder(...)` | Defines test execution order |
| `@TestInstance(Lifecycle.PER_CLASS)` | One instance per class (allows non-static `@BeforeAll`) |
| `@ExtendWith(...)` | Registers extensions (e.g., `MockitoExtension.class`) |

```java
@DisplayName("UserService Tests")
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @BeforeAll
    static void globalSetup() { /* runs once */ }

    @BeforeEach
    void setUp() { /* runs before each test */ }

    @Test
    @DisplayName("Should return user by ID")
    void shouldReturnUserById() { }

    @Test
    @Disabled("Fix pending - see JIRA-123")
    void skippedTest() { }

    @AfterEach
    void tearDown() { /* runs after each test */ }

    @AfterAll
    static void globalTeardown() { /* runs once */ }
}
```

---

## JUnit 5 Assertions

```java
import static org.junit.jupiter.api.Assertions.*;

// Equality
assertEquals(expected, actual);
assertEquals(expected, actual, "failure message");
assertNotEquals(unexpected, actual);

// Null checks
assertNull(object);
assertNotNull(object);

// Boolean
assertTrue(condition);
assertFalse(condition);

// Array / Iterable equality
assertArrayEquals(expectedArray, actualArray);
assertIterableEquals(expectedList, actualList);

// Same instance
assertSame(expected, actual);
assertNotSame(unexpected, actual);

// Exception assertion
assertThrows(IllegalArgumentException.class, () -> service.doSomething(null));

// Exception with message check
Exception ex = assertThrows(RuntimeException.class, () -> service.fail());
assertEquals("Expected message", ex.getMessage());

// No exception thrown
assertDoesNotThrow(() -> service.safeMethod());

// Grouped assertions (all run even if one fails)
assertAll("user fields",
    () -> assertEquals("John", user.getName()),
    () -> assertEquals("john@example.com", user.getEmail()),
    () -> assertNotNull(user.getId())
);

// Timeout
assertTimeout(Duration.ofSeconds(2), () -> service.process());

// Fails immediately on timeout
assertTimeoutPreemptively(Duration.ofSeconds(2), () -> service.process());

// Explicit fail
fail("This test is not yet implemented");
```

---

## JUnit 5 Assumptions

Assumptions abort (skip) a test gracefully when conditions aren't met.

```java
import static org.junit.jupiter.api.Assumptions.*;

assumeTrue(condition);                          // Skip if false
assumeFalse(condition);                         // Skip if true
assumeTrue("CI".equals(System.getenv("ENV"))); // Skip if not CI env

assumingThat(condition, () -> {
    // Only runs if condition is true; test continues regardless
    assertEquals(42, result);
});
```

---

## JUnit 5 Test Lifecycle

```
@BeforeAll (static)
    └── @BeforeEach
        └── @Test
    └── @AfterEach
    └── @BeforeEach
        └── @Test
    └── @AfterEach
@AfterAll (static)
```

Use `@TestInstance(Lifecycle.PER_CLASS)` to share state across tests (no need for `static` on `@BeforeAll`/`@AfterAll`).

---

## JUnit 5 Parameterized Tests

```java
// Requires: junit-jupiter-params dependency

@ParameterizedTest
@ValueSource(ints = {1, 2, 3, 4, 5})
void testWithInts(int value) {
    assertTrue(value > 0);
}

@ParameterizedTest
@ValueSource(strings = {"Alice", "Bob", "Charlie"})
void testWithStrings(String name) {
    assertNotNull(name);
}

@ParameterizedTest
@NullAndEmptySource
@ValueSource(strings = {" ", "  "})
void testBlankStrings(String input) {
    assertTrue(input == null || input.isBlank());
}

@ParameterizedTest
@CsvSource({
    "Alice, 30",
    "Bob,   25",
    "Charlie, 35"
})
void testWithCsv(String name, int age) {
    assertNotNull(name);
    assertTrue(age > 0);
}

@ParameterizedTest
@CsvFileSource(resources = "/test-data.csv", numLinesToSkip = 1)
void testWithCsvFile(String name, int age) { }

@ParameterizedTest
@MethodSource("provideArguments")
void testWithMethodSource(String name, int age) { }

static Stream<Arguments> provideArguments() {
    return Stream.of(
        Arguments.of("Alice", 30),
        Arguments.of("Bob", 25)
    );
}

@ParameterizedTest
@EnumSource(value = DayOfWeek.class, names = {"MONDAY", "FRIDAY"})
void testWithEnum(DayOfWeek day) {
    assertNotNull(day);
}
```

---

## JUnit 5 Dynamic Tests

```java
@TestFactory
Collection<DynamicTest> dynamicTests() {
    return List.of(
        DynamicTest.dynamicTest("Test 1", () -> assertEquals(2, 1 + 1)),
        DynamicTest.dynamicTest("Test 2", () -> assertTrue(true))
    );
}

@TestFactory
Stream<DynamicTest> streamDynamicTests() {
    return Stream.of("Alice", "Bob", "Charlie")
        .map(name -> DynamicTest.dynamicTest(
            "Test for " + name,
            () -> assertNotNull(name)
        ));
}
```

---

## Mockito Setup & Annotations

### Maven Dependency

```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-junit-jupiter</artifactId>
    <version>5.x.x</version>
    <scope>test</scope>
</dependency>
```

### Annotations

| Annotation | Description |
|---|---|
| `@Mock` | Creates a mock instance |
| `@InjectMocks` | Creates instance and injects `@Mock`/`@Spy` fields |
| `@Spy` | Wraps a real object; stubs selected methods |
| `@Captor` | Creates an `ArgumentCaptor` |
| `@ExtendWith(MockitoExtension.class)` | Enables Mockito annotations in JUnit 5 |

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    private OrderRepository orderRepository;

    @Mock
    private EmailService emailService;

    @InjectMocks
    private OrderService orderService; // Mocks injected automatically

    @Captor
    private ArgumentCaptor<Order> orderCaptor;
}
```

### Manual Initialization (without extension)

```java
@BeforeEach
void setUp() {
    MockitoAnnotations.openMocks(this);
    // or
    orderRepository = Mockito.mock(OrderRepository.class);
    orderService = new OrderService(orderRepository);
}
```

---

## Mockito Core Methods

```java
import static org.mockito.Mockito.*;

// Create mocks
UserRepository repo = mock(UserRepository.class);

// Stubbing (when...thenReturn)
when(repo.findById(1L)).thenReturn(Optional.of(new User()));
when(repo.findAll()).thenReturn(List.of(user1, user2));
when(repo.save(any())).thenAnswer(inv -> inv.getArgument(0));

// Void methods
doNothing().when(repo).delete(any());
doThrow(new RuntimeException()).when(repo).delete(null);

// Verify interactions
verify(repo).findById(1L);
verify(repo, times(2)).save(any());
verify(repo, never()).deleteAll();

// Reset mock state
reset(repo);

// Clear invocations (keep stubbing)
clearInvocations(repo);
```

---

## Argument Matchers

```java
import static org.mockito.ArgumentMatchers.*;

// Any value
when(service.process(any())).thenReturn(result);
when(service.process(any(User.class))).thenReturn(result);

// Primitives
when(repo.findById(anyLong())).thenReturn(Optional.of(user));
when(service.repeat(anyInt())).thenReturn("ok");
when(service.greet(anyString())).thenReturn("Hello");
when(service.check(anyBoolean())).thenReturn(true);

// Collections
when(service.saveAll(anyList())).thenReturn(savedList);
when(service.saveAll(anySet())).thenReturn(savedSet);
when(service.merge(anyMap())).thenReturn(merged);

// Specific value matching
when(service.find(eq("john"))).thenReturn(user);

// String matchers
when(service.search(startsWith("A"))).thenReturn(results);
when(service.search(endsWith("z"))).thenReturn(results);
when(service.search(contains("abc"))).thenReturn(results);
when(service.search(matches("\\d+"))).thenReturn(results);

// Custom matcher
when(service.find(argThat(u -> u.getAge() > 18))).thenReturn(adult);

// Null / Not null
when(service.process(isNull())).thenReturn(defaultVal);
when(service.process(notNull())).thenReturn(validResult);

// ⚠️ Rule: If you use ANY matcher, ALL arguments must use matchers
verify(service).update(eq(1L), any(User.class)); // ✅ Correct
verify(service).update(1L, any(User.class));      // ❌ Throws error
```

---

## Verification

```java
// Called exactly once (default)
verify(repo).save(user);

// Call count
verify(repo, times(3)).save(any());
verify(repo, atLeast(1)).save(any());
verify(repo, atLeastOnce()).save(any());
verify(repo, atMost(5)).save(any());
verify(repo, never()).deleteAll();

// No interactions
verifyNoInteractions(emailService);

// No more interactions after verified ones
verify(repo).findById(1L);
verifyNoMoreInteractions(repo);

// Timeout verification (async)
verify(repo, timeout(1000)).save(any());
verify(repo, timeout(1000).times(2)).save(any());
```

---

## Stubbing

```java
// Return values
when(repo.findById(1L)).thenReturn(Optional.of(user));

// Return different values on consecutive calls
when(repo.count())
    .thenReturn(1L)
    .thenReturn(2L)
    .thenReturn(3L);

// Throw exception
when(repo.findById(-1L)).thenThrow(new IllegalArgumentException("Invalid ID"));
when(repo.findById(-1L)).thenThrow(IllegalArgumentException.class);

// Chained stubbing
when(repo.findById(anyLong()))
    .thenReturn(Optional.of(user))  // 1st call
    .thenThrow(new RuntimeException()) // 2nd call
    .thenReturn(Optional.empty());     // 3rd+ calls

// Void method stubs (use doXxx syntax)
doNothing().when(emailService).sendEmail(anyString());
doThrow(new MailException()).when(emailService).sendEmail(null);
doReturn(user).when(repo).findById(1L); // Alternative syntax

// Return real method (on spies)
doCallRealMethod().when(spy).calculate();

// Custom answer
when(repo.save(any())).thenAnswer(invocation -> {
    User u = invocation.getArgument(0);
    u.setId(100L);
    return u;
});
```

---

## Spy

A Spy wraps a **real object** — real methods are called unless stubbed.

```java
// Create spy
List<String> realList = new ArrayList<>();
List<String> spyList = spy(realList);

// Real method is called
spyList.add("one");
assertEquals(1, spyList.size()); // ✅ real size

// Stub specific method
doReturn(100).when(spyList).size(); // ⚠️ Use doReturn, not when().thenReturn() on spies
assertEquals(100, spyList.size());  // stubbed

// Via annotation
@Spy
private List<String> names = new ArrayList<>();

// Verify spy interactions
verify(spyList).add("one");
```

---

## Captor

Capture arguments passed to a mock for detailed assertions.

```java
// Via annotation
@Captor
private ArgumentCaptor<User> userCaptor;

// Via static method
ArgumentCaptor<User> captor = ArgumentCaptor.forClass(User.class);

// Capture single call
verify(repo).save(userCaptor.capture());
User savedUser = userCaptor.getValue();
assertEquals("Alice", savedUser.getName());

// Capture multiple calls
verify(repo, times(3)).save(userCaptor.capture());
List<User> allSavedUsers = userCaptor.getAllValues();
assertEquals(3, allSavedUsers.size());
```

---

## Exception Handling

```java
// Assert exception type
assertThrows(UserNotFoundException.class, () ->
    userService.findById(999L)
);

// Assert exception message
UserNotFoundException ex = assertThrows(UserNotFoundException.class, () ->
    userService.findById(999L)
);
assertEquals("User not found: 999", ex.getMessage());

// Stub to throw
when(repo.findById(999L)).thenThrow(new UserNotFoundException("User not found: 999"));

// Throw from void method
doThrow(new EmailException("SMTP error")).when(emailService).send(any());
```

---

## Mockito Answer

Provides dynamic responses to stubbed calls.

```java
// Return first argument
when(repo.save(any(User.class))).thenAnswer(ReturnsFirstArg.returnsFirstArg());

// Custom logic
when(userService.create(any())).thenAnswer(invocation -> {
    User user = invocation.getArgument(0);  // Get 1st arg
    // int count = invocation.getArguments().length; // arg count
    user.setId(UUID.randomUUID());
    return user;
});

// Simulate delay
when(slowService.fetch()).thenAnswer(inv -> {
    Thread.sleep(100);
    return "result";
});
```

---

## InOrder Verification

Verify that mocks were called in a specific sequence.

```java
InOrder inOrder = inOrder(repo, emailService);

inOrder.verify(repo).save(user);
inOrder.verify(emailService).sendWelcomeEmail(user.getEmail());
inOrder.verify(repo).updateStatus(user.getId(), "ACTIVE");

// With argument matchers
InOrder inOrder2 = inOrder(paymentService, auditService);
inOrder2.verify(paymentService).charge(any());
inOrder2.verify(auditService).log(anyString());
```

---

## Common Patterns

### Full Service Test Example

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @Mock
    private EmailService emailService;

    @InjectMocks
    private UserService userService;

    @Captor
    private ArgumentCaptor<User> userCaptor;

    @Test
    @DisplayName("Should create user and send welcome email")
    void shouldCreateUserAndSendWelcomeEmail() {
        // Arrange
        CreateUserRequest request = new CreateUserRequest("Alice", "alice@example.com");
        User savedUser = new User(1L, "Alice", "alice@example.com");
        when(userRepository.save(any(User.class))).thenReturn(savedUser);
        doNothing().when(emailService).sendWelcomeEmail(anyString());

        // Act
        User result = userService.createUser(request);

        // Assert
        assertNotNull(result);
        assertEquals(1L, result.getId());
        assertEquals("Alice", result.getName());

        // Verify repo was called with correct data
        verify(userRepository).save(userCaptor.capture());
        User capturedUser = userCaptor.getValue();
        assertEquals("alice@example.com", capturedUser.getEmail());

        // Verify email was sent
        verify(emailService).sendWelcomeEmail("alice@example.com");
        verifyNoMoreInteractions(emailService);
    }

    @Test
    @DisplayName("Should throw exception when user not found")
    void shouldThrowWhenUserNotFound() {
        when(userRepository.findById(anyLong())).thenReturn(Optional.empty());

        assertThrows(UserNotFoundException.class, () ->
            userService.findById(99L)
        );

        verify(userRepository).findById(99L);
        verifyNoInteractions(emailService);
    }
}
```

### Testing Void Methods

```java
@Test
void shouldDeleteUser() {
    doNothing().when(userRepository).deleteById(anyLong());

    assertDoesNotThrow(() -> userService.deleteUser(1L));

    verify(userRepository).deleteById(1L);
}
```

### Nested Tests

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock OrderRepository orderRepository;
    @InjectMocks OrderService orderService;

    @Nested
    @DisplayName("When creating an order")
    class CreateOrder {

        @Test
        @DisplayName("Should persist order to database")
        void shouldPersistOrder() { }

        @Test
        @DisplayName("Should fail with null items")
        void shouldFailWithNullItems() { }
    }

    @Nested
    @DisplayName("When cancelling an order")
    class CancelOrder {

        @Test
        @DisplayName("Should update order status")
        void shouldUpdateStatus() { }
    }
}
```

---

## Quick Reference Card

| Task | Syntax |
|---|---|
| Create mock | `mock(Class.class)` / `@Mock` |
| Stub return | `when(x.m()).thenReturn(v)` |
| Stub void | `doNothing().when(x).m()` |
| Stub exception | `when(x.m()).thenThrow(Ex.class)` |
| Verify called | `verify(x).m()` |
| Verify N times | `verify(x, times(N)).m()` |
| Verify never | `verify(x, never()).m()` |
| Capture arg | `verify(x).m(captor.capture())` |
| Any matcher | `any()`, `anyString()`, `anyLong()` |
| Eq matcher | `eq(value)` |
| Spy real obj | `spy(realObject)` / `@Spy` |
| Assert throws | `assertThrows(Ex.class, () -> ...)` |
| Assert all | `assertAll(() -> ..., () -> ...)` |
| Ordered verify | `inOrder(a,b).verify(a).m()` |
| No interactions | `verifyNoInteractions(x)` |
| Reset mock | `reset(x)` |
