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
- Spring provides the following `TestExecutionListener` implementations that are registered by default, ***exactly in the following order:***

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

### Application Events
- **Recording app events** : Since Spring 5.3.3, the TestContext framework supports recording application events published in the application context
- **Usage** : To use `ApplicationEvents` in the test : 
	1. test class to be annotated or meta-annotated with `@RecordApplicationEvents`
	2. `ApplicationEventsTestExecutionListener` is registered (it is registered by default)
	3. auto-wire a field of type `ApplicationEvents` 
	4. all events published are available as streams
```java
@SpringJUnitConfig(/* ... */) 
@RecordApplicationEvents      //(1)
class OrderServiceTests {
	@Autowired
	ApplicationEvents events;  //(3)
	@Test
	void submitOrder() {
		// business operation which generates events
		...
		// event verification
		int count = events.stream(OrderSubmitted.class).count();  //(4)
		assertThat(count).isEqualTo(1);
	}
}
```

### Test Execution Events
- **Alternative to test listener** : Since Spring 5.2, there's an alternative approach to implementing custom `TestExecutionListener`
	- **Approach** : components in the test's application context can listen to events published by `EventPublishingTestExecutionListener`. Each of these events corresponds to a method in the listener
	- **Events** : `BeforeTestClassEvent`, `PrepareTestInstanceEvent`, etc.
-  These events are only published if the `ApplicationContext` has already been loaded.
-  **Benefit** : 
	-  `TestExecutionListener` is not a bean in the `ApplicationContext`. But a test execution event can be consumed by beans and they can use DI and the app context
-  **Usage** : 
	1. implement `ApplicationListener`
	2. annotate listener methods with `@EventListener`
		- custom meta-annotations are available -- `@BeforeClass`, `@PrepareTestInstance`, etc.
- **Exception Handling** : TBD [Exception Handling (spring.io)](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-test-execution-events-exception-handling)
- **Async Listeners** : TBD [Asynchronous Listeners (spring.io)](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-test-execution-events-async)

### Context Management
- **What** : Each `TestContext` provides context management and caching support for the test instance
- **App context access** : 
	1. implement `ApplicationContextAware` to get access to app context. Spring-JUnit and Spring-TestNG implement that & hence have access to app context
	2. can auto-wire `ApplicationContext` or `WebApplicationContext` in the test

#### Context with Profiles
- You can use the `@Profile` annotation to allocate beans to profiles. No profile means beans are available in all profiles. The `default` profile means ONLY used when no profile is active
- Use `@ActiveProfiles` annotation to suggest that the specific profile(s) are active
- Resolve profiles dynamically :(implement `ActiveProfilesResolver`)
```java
@ActiveProfiles( 
	resolver = OperatingSystemActiveProfilesResolver.class, 
	inheritProfiles = false)
class MyTest {
}
```

### Test Property Source
#### @TestPropertySource
- use `@TestPropertySource` on test class to declare property sources. These property sources are ***ADDED*** to the set of property sources in the `Environment`
- Declaring
```java
// file source
@ContextConfiguration 
@TestPropertySource("/test.properties")  
class MyIntegrationTests {}

//key value 
@ContextConfiguration 
@TestPropertySource(properties = {"timezone = GMT", "port: 4242"})  
class MyIntegrationTests {}

```
- Precedence
	- test property sources have precedence over application property sources
	- inlined test properties have precedence over file test properites (e.g. `timezone=GMT` above has higher precedence over the value declared in the file `test.properties`)
	- `@DynamicPropertySource` (explained below) has higher precedence than `@TestPropertySource`
- `@TestPropertySource` has attributes `inheritLocations` & `inheritProperties` which are both set to `true` by default. Also precedence rules apply. Read : [Inheriting Test Property Sources (spring.io)](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#inheriting-and-overriding-test-property-sources)

#### Dynamic Test Property Sources
- Since 5.2.5, dynamic properties support via `@DynamicPropertySource`
- was originally designed to allow "Test Containers"
- **usage** : `@TestPropertySource` is on class-level, however `@DynamicPropertySource` is on a `static` method to add key-value pairs. Values are dynamic & provided via `Supplier`
```java
@SpringJUnitConfig(/* ... */) 
@Testcontainers 
class ExampleIntegrationTests { 
	@Container 
	static RedisContainer redis = new RedisContainer();
	
	@DynamicPropertySource 
	static void redisProperties(DynamicPropertyRegistry registry) {
		registry.add("redis.host", redis::getContainerIpAddress);
		registry.add("redis.port", redis::getMappedPort); 
	}
}
```

### Web application configuration
[Loading a `WebApplicationContext` (spring.io)](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-ctx-management-web)

### Context Caching
- once the TestContext framework loads an `ApplicationContext` (or `WebApplicationContext`) for a test, the context is cached & reused for all subsequent tests with 
	- same ***unique context config*** in the 
	- same ***test suite***.
- The TestContext framework uses many configuration parameters to check uniqueness and then based on that stores it in static context cache
	- the parameters are : [Context Caching (spring.io)](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-ctx-management-caching)
- **Forked process** : 
	- context is stored in a `static` variable
	- if tests run in separate processes then contexts cannot be cached (e.g. using `forkMode` in Maven)
- **Size of context cache** : 
	- default size is 32; after that LRU
	- set new size using `spring.test.context.cache.maxSize` via JVM or Spring properties
- **View Cache Statistics** : set `org.springframework.test.context.cache` logging to `DEBUG`
- **Dirties Context** : if the test corrupts the context & requires reloading, annotate the class/test with `@DirtiesContext`

### Context Hierarchies
[Context Hierarchies (spring.io)](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-ctx-management-ctx-hierarchies)

### Transaction Management
[Transaction Management (spring.io)](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-tx)

### Execution SQL Scripts
[Execution SQL Scripts (spring.io)](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-executing-sql)
`@Sql`

### Parallel Test Execution
- Spring 5.0 has basic support for executing tests in parallel in a single JVM
- Guidelines by Spring team -- DO NOT run tests in parallel  :
	- if tests use `@DirtiesContext`
	- if tests use `@MockBean` or `@SpyBean`
	- if tests use JUnit 4 `@FixMethodOrder`
	- if tests change state of shared services or systems such as DB, Message broker, files, etc.

### TestContext Support Classes
#### Spring JUnit 4 Runner
- use `@RunWith(SpringRunner.class)` or `@RunWith(SpringJUnit4ClassRunner.class)`
#### Spring JUnit 4 Rules
- `SpringClassRule` 
	- is a JUnit `TestRule`
	- supports class-level features of Spring
- `SpringMethodRule`
	- is a JUnit `MethodRule`
	- supports instance-level or method-level features of Spring
- **Differences** : 
	- In contrast to the `SpringRunner`, Spring’s rule-based JUnit support has the advantage of being independent of any `org.junit.runner.Runner` implementation and can, therefore, be combined with existing alternative runners (such as JUnit 4’s `Parameterized`) or third-party runners (such as the `MockitoJUnitRunner`).
- **Usage** : 
	- To support the full functionality of the TestContext framework, you must combine a `SpringClassRule` with a `SpringMethodRule`.
#### Spring JUnit Jupiter
- use `@ExtendWith(SpringExtension.class)` OR
- meta-annotation for e.g. `@SpringJUnitConfig`, `@SpringJUnitWebConfig`
#### Nested Class Config
- [Nested Class Config (spring.io)](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-junit-jupiter-nested-test-configuration)

## Spring WebTestClient
- wraps `WebClient` to perform end-to-end tests by performing requests and verifying response
- can be used to test MVC or WebFlux without a running server via mock server request and response object
### Setup : Bind to Controller
- allows to test controller(s) via mock request and response objects ***without*** running a server
- For WebFlux, it loads infrastructure equivalent to webflux java config, registers the given controller(s) and creates a "WebHandler chain" to handle requests
```java
WebTestClient client = WebTestClient.bindToController(new TestController()).build();
```
- For Spring MVC, it delegates to `StandaloneMockMvcBuilder` to load infrastructure equivalent of WebMvc java config, registers the given controller(s) and creates an instance of `MockMvc` to handle requests
```java
WebTestClient client = MockMvcWebTestClient.bindToController(new TestController()).build();
```
### Setup : Bind to Application Context
- For WebFlux :
```java
@SpringJUnitConfig(WebConfig.class)  
class MyTests { 
	WebTestClient client; 
	@BeforeEach 
	void setUp(ApplicationContext context) {  
		client = WebTestClient.bindToApplicationContext(context).build();  
	} 
}
```
- For Spring MVC : 
```java
@ExtendWith(SpringExtension.class) 
@WebAppConfiguration("classpath:META-INF/web-resources")  
@ContextHierarchy({ @ContextConfiguration(classes = RootConfig.class), @ContextConfiguration(classes = WebConfig.class) }) 
class MyTests { 
	@Autowired 
	WebApplicationContext wac;  
	WebTestClient client; 
	@BeforeEach 
	void setUp() { 
	  client = MockMvcWebTestClient.bindToApplicationContext(this.wac).build();
	} 
}
```
### Setup : Bind to Router Function
[Router Function (spring.io)](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#webtestclient-fn-config)

### Setup : Bind to Server
[Bind to Server (spring.io)](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#webtestclient-server-config)

### Client Config 
[Client Config (spring.io)](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#webtestclient-client-config)

### Using in Tests
```java
client.get().uri("/persons/1")
	.accept(MediaType.APPLICATION_JSON)
	.exchange()
	.expectStatus().isOk() 	
	.expectHeader().contentType(MediaType.APPLICATION_JSON)
```
- Response Body can be decoded via
	- `expectBody(Class<T>)` : decode to single object
	- `expectBodyList(Class<T>)`: Decode and collect objects to `List<T>`
	- `expectBody()`: Decode to `byte[]` for JSON Context or an empty body
- Perform assertions as : 
```java
client.get().uri("/persons").exchange()
	.expectStatus().isOk() 	
	.expectBodyList(Person.class).hasSize(3).contains(person);
//OR have custom assertions
import org.springframework.test.web.reactive.server.expectBody client.get().uri("/persons/1").exchange()
	.expectStatus().isOk()
	.expectBody(Person.class)
	.consumeWith(result -> { 
		// custom assertions (e.g. AssertJ)... 
	});
//OR get result 
EntityExchangeResult<Person> result = client.get().uri("/persons/1") 
			.exchange()
			.expectStatus().isOk() 
			.expectBody(Person.class) 
			.returnResult();
//OR No Content
client.post().uri("/persons") 
	.body(personMono, Person.class) 
	.exchange() 
	.expectStatus().isCreated() 
	.expectBody().isEmpty();
//OR to ignore response content
client.get().uri("/persons/123") 
	.exchange() 
	.expectStatus().isNotFound() 
	.expectBody(Void.class);
//OR to verify JSON
client.get().uri("/persons/1") 
	.exchange() .expectStatus().isOk() 
	.expectBody() 
	.json("{\"name\":\"Jane\"}")   // slashes required
//OR using JSONPath
client.get().uri("/persons") 
	.exchange() 
	.expectStatus().isOk() 
	.expectBody() 
	.jsonPath("$[0].name").isEqualTo("Jane") 
	.jsonPath("$[1].name").isEqualTo("Jason");
```

#### Streaming Responses
- To test potentially infinite streams such as `"text/event-stream"` or `"application/x-ndjson"`, start by verifying the response status and headers, and then obtain a `FluxExchangeResult`:
```java
FluxExchangeResult<MyEvent> result = client.get().uri("/events") 
					.accept(TEXT_EVENT_STREAM) 
					.exchange() 
					.expectStatus().isOk() 
					.returnResult(MyEvent.class);
// Now use `StepVerifier` from `reactor-test`
Flux<Event> eventFlux = result.getResponseBody();
StepVerifier.create(eventFlux) 
			.expectNext(person) 
			.expectNextCount(4) 
			.consumeNextWith(p -> ...) 
			.thenCancel() 
			.verify();
```

#### MockMvc assertions
- to perform further assertions on the server responsem, start by obtaining an `ExchangeResult` after asserting the body
```java
// For a response with a body 
EntityExchangeResult<Person> result = client.get().uri("/persons/1") 
					.exchange() 
					.expectStatus().isOk() 
					.expectBody(Person.class) 
					.returnResult(); 
// For a response without a body 
EntityExchangeResult<Void> result = client.get().uri("/path") 
					.exchange() 
					.expectBody().isEmpty();
// MockMvc assertions
MockMvcWebTestClient.resultActionsFor(result) 
			.andExpect(model().attribute("integer", 3)) 
			.andExpect(model().attribute("string", "a string value"));
```

## MockMvc
