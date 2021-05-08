## Spring Testing Documentation
[Testing (spring.io)](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#integration-testing)
### Goals of Spring Integration Testing framework
- Spring's integration test support has the following goals
	- manage Spring IoC container caching between tests
	- provide DI of test fixture instances
	- provide transaction management appropriate to integration testing
	- supply Spring-specific base classes that assist developers in writing integration tests

### Context Management & Caching
- **Loading-Caching** : Spring provides consistent loading of `ApplicationContext` & `WebApplicationContext` and caching of those contexts
- **Why Caching** : objects instantiated by Spring container  (all Hibernate mappings, etc) can take time if not cached
- **Context reuse** : By default, once loaded, the `ApplicationContext` is reused for each test in the test suite. "test suite" means all tests run in the same JVM (e.g. all tests run from Maven)
- **Context corruption** : If the context is corrupted by a test then the context will be rebuild & reloaded
### Transaction Management
- By default, the framework creates and ***rolls back*** a transaction for each test. 
	- if you want a transaction to commit, use the `@Commit` annotation
- Transactional support is provided to a test by using a `PlatformTransactionManager` bean defined in the test’s application context.

### Spring Testing annotations

Annotation | Description
|--------|-----|
`BootstrapWith`        | class level annotation to configure how the Spring `TestContext` framework is bootstrapped. Can specify a custom `TestContextBootstrapper`
| `ContextConfiguration` | defines class-level metadata to determine how to load and configure an `ApplicationContext` for integration tests. Declares the application context resource `locations` or the component `classes` used to load the context. You can also declare `ApplicationContextInitialize` classes|
| `@WebAppConfiguration` | class-level annotation to declare that the `ApplicationContext` which loads for an integration test should be a `WebApplicationContext`  |
| `ContextHierarchy`| class-level annotation to define a hierarchy of `ApplicationContext` instances for tests|
| `@ActiveProfile`       | class-level annotation to declare which bean definition profile(s) should be active when loading an `ApplicationContext` for an integration test |
| `@TestPropertySource`  | class-level annotation to configure the locations of properties files and inline properties to be added to the set of `PropertySources` in the `Environment` for an `ApplicationContext` loaded for the test |
|`@DynamicPropertySource`|method-level annotation to register dynamic properties to be added to the set of `PropertySources` in the `Environment`Dynamic properties are useful when you do not know the value of the properties upfront – for example, if the properties are managed by an external resource such as for a container managed by the "Testcontainers" project.|
| `@DirtiesContext`      | indicates that the underlying `ApplicationContext` has been dirtied during the execution of a test and should be closed. Can be used on method-level or class-level |
|`@TestExecutionListeners`| Class-level annotation for configuring `TestExecutionListeners` that should be registered with the `TestContextManager`|
| miscellaneous          | `@Commit`, `@Rollback`,, etc.   |

### JUnit 4 Testing annotations

Annotation|Description
|----|----|
|`@IfProfileValue`|indicates that the annotated test is enabled for a specific testing environment. If the configured `ProfileValueSource` returns a matching `value` for the provided `name`, the test is enabled. Otherwise, the test is disabled and, effectively, ignored.|
|`@Repeat`|method-level annotation to indicate the test is repeated|

### JUnit 5 Testing Annotations

Annotation|Description
|----|----|
|`@SpringJUnitConfig`|composed annotation that combines `@ExtendWith(SpringExtension.class)` from JUnit Jupiter with `@ContextConfiguration` from the Spring TestContext Framework.|
|`@SpringJUnitWebConfig`|`@SpringJUnitWebConfig` is a composed annotation that combines `@ExtendWith(SpringExtension.class)` from JUnit Jupiter with `@ContextConfiguration` and `@WebAppConfiguration` from the Spring TestContext Framework.|
|`@EnabledIf`|is used to signal that the annotated JUnit Jupiter test class or test method is enabled and should be run if the supplied `expression` evaluates to `true`. Expressions can be SpEL or property value or text literal|
|`@DisabledIf`|is used to signal that the annotated JUnit Jupiter test class or test method is disabled and should not be run if the supplied `expression` evaluates to `true`|

### Meta-annotation support
- JUnit Jupiter supports `@Test`, `@RepeatedTest` & `@ParameterizedTest` as meta-annotations -- so these can be used to create composed annotations
```java
@Target(ElementType.METHOD) 
@Retention(RetentionPolicy.RUNTIME) 
@Transactional 
@Tag("integration-test") // org.junit.jupiter.api.Tag 
@Test // org.junit.jupiter.api.Test 
public @interface TransactionalIntegrationTest { }
```


## Spring TestContext Framework
### Key Abstractions
- `TestContextManager` is created for ***each class***
- `TestContextManager` 
	- manages a `TestContext` that holds the context of the current test and udpates the state of `TestContext` as the test progresses
	- and delegates to `TestExecutionListener` implementations which instrument the actual test execution by providing dependency injection, managing transactions, and so on
- A `SmartContextLoader` is responsible for loading an `ApplicationContext` for a given class
- Main : `TestContextManager` class & `TestContext`, `TestExecutionListener`, `SmartContextLoader` interfaces
- Details : [TestContext Abstractions (spring.io)](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-key-abstractions)

### Bootstrapping TestContext Framework
- [Bootstrapping TestContext (spring.io)](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-bootstrapping)

### TestExecutionListener Configuration
- Spring provides the following `TestExecutionListener` implementations that are registered by default, exactly in the following order:

|`TestExecutionListener`|Details|
|----|----|
|`ServletTestExecutionListener`|configures servlet API mocks for a `WebApplicationContext`|
|`DirtiesContextBeforeModesTestExecutionListener`|Handles the `@DirtiesContext` annotation for “before” modes.|
|`ApplicationEventsTestExecutionListener`|Provides support for `ApplicationEvent`s|
|`DependencyInjectionTestExecutionListener`|Provides dependency injection for the test instance.|
|`DirtiesContextTestExecutionListener`|Handles the `@DirtiesContext` annotation for “after” modes.|
|`TransactionalTestExecutionListener`|Provides transactional test execution with default rollback semantics.|
|`SqlScriptsTestExecutionListener`|Runs SQL scripts configured by using the `@Sql` annotation.|
|`EventPublishingTestExecutionListener`|Publishes test execution events to the test’s `ApplicationContex`|

#### Registering `TestExecutionListener`
- You can register `TestExecutionListener` implementations for a test class and its subclasses by using the `@TestExecutionListeners` annotation.
- **Discovery** : 
	- default `TestExecutionListener` implementations are discovered through the `SpringFactoriesLoader` mechanism
	- `spring-test` declares in `META-INF/spring.factories` file. Third-party developers can declare in their own `META-INF/spring.factories`
- **Ordering** : through `Order` annotations
- **Merging impls** : 
	- If a custom `TestExecutionListener` is registered via `@TestExecutionListeners`, the default listeners are not registered
	- `mergeMode` attribute of `@TestExecutionListeners` with value `MergeMode.MERGE_WITH_DEFAULTS` indicates that locally declared listeners should be merged with the default listeners.
- 
