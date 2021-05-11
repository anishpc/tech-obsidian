# Testing Spring Boot Applications

## Intro
### Basic Annotations
- **Features** : External properties, logging, and other features of Spring Boot are installed in the context by default only if you use `SpringApplication` to create it

## SpringBootTest
- `@SpringBootTest` is an alternative to the standard Spring's `@ContextConfiguration`
- `@SpringBootTest` creates the application context for tests through `SpringApplication`
- **JUnit** : 
	- for JUnit 4, add the `@RunWith(SpringRunner.class)` , otherwise annotations will be ignored
	- for JUnit 5, there’s no need to add the equivalent `@ExtendWith(SpringExtension.class)` as `@SpringBootTest` and the other `@…Test` annotations are already annotated with it.

### SpringBootTest > webEnvironment
- by default, `@SpringBootTest` will ***NOT*** start a server

|`webEnvironment` attribute|Server started?|Description|
|----|-----|----|
|`MOCK`(default)|No|Loads a web `ApplicationContext` and provides a mock web environment.It can be used in conjunction with `@AutoConfigureMockMvc` or `@AutoConfigureWebTestClient` for mock-based testing of your web application.|
|`RANDOM_PORT`|Yes (on random port)|Loads a `WebServerApplicationContext` and provides a real web environment.|
|`DEFINED_PORT`|Yes (on defined port or 8080 by default)|Loads a `WebServerApplicationContext` and provides a real web environment|
|`NONE`|No|Loads an `ApplicationContext` by using `SpringApplication` but does not provide _any_ web environment (mock or otherwise).|

#### Separate client & server threads
- `@Transactional` rolls back the transaction at the end of each test method by default. 
- With either `RANDOM_PORT` or `DEFINED_PORT` provides a real servlet environment, the HTTP client and server run in ***separate threads*** and, thus, in separate transactions. 
- Any transaction initiated on the server ***does not roll back*** in this case.

#### Management port
- `@SpringBootTest` with `webEnvironment = WebEnvironment.RANDOM_PORT` will also start the management server on a separate random port if your application uses a different port for the management server.

### WebApplication type
- if Spring MVC is available then MVC-based application context is configured
- if ONLY Spring WebFlux then WebFlux-based app context
- if BOTH then Spring MVC takes precedence
- Explicit : `@SpringBootTest(properties = "spring.main.web-application-type=reactive")`

## Detecting Test Configuration
- **General Spring** : To load Spring configuration, Spring test framework uses 
	- `ContextConfiguration(classes=...)` OR
	- use nested `@Configuration` classes
- **Spring Boot** : 
	- for Spring Boot, the above is NOT required
	- Spring Boot's `@*Test` annotations search for primary configuration automatically when one is not explicitly defined
	- Search algorithm : works up from the package that contains the test until it finds a class annotated with `@SpringBootApplication` or `@SpringBootConfiguration`.
	- Nested configuration : 
		- Nested `@TestConfiguration` class is used ***in addition to*** the primary config; typically used for additional beans or customization to the primary config
		- Nested `@Configuration` class is used ***instead of*** the primary config

## Excluding Test Configuration
- When `@TestConfiguration` is placed on a top-level class, it indicates that classes in `src/test/java` should NOT be picked up by scanning. You can then import that class explicitly where it is required
```java
@SpringBootTest 
@Import(MyTestsConfiguration.class) 
class MyTests { 
	@Test 
	void exampleTest() { ... } 
}
```

## Using Application Arguments
- For application arguments, you can have `@SpringBootTest` inject them using the `args` attribute
```java
@SpringBootTest(args = "--app.test=one") 
class ApplicationArgumentsExampleTests { 
	@Test 
	void applicationArgumentsPopulated(@Autowired ApplicationArguments args) { 
		assertThat(args.getOptionNames()).containsOnly("app.test"); 
		assertThat(args.getOptionValues("app.test")).containsOnly("one"); 
	} 
}
```

## Testing With a mock environment
- By default, `@SpringBootTest` does NOT start the server
- **Options** : `MockMvc` OR `WebTestClient`
- **Servlet layer testing** : 
	- since mocking occurs at the MVC layer, code that relies on the lower-level Servlet container behaviour cannot be directly tested with `MockMvc`
	- for e.g., you can test that an exception is thrown but cannot test that a specific error page is rendered. For that, we need a fully running server

### MockMvc
- To test web-endpoints against mock environment, you can configure `MockMvc`
```java
@SpringBootTest 
@AutoConfigureMockMvc 
class MockMvcExampleTests { 
	@Test 
	void exampleTest(@Autowired MockMvc mvc) throws Exception { 		
		mvc.perform(get("/"))
			.andExpect(status().isOk())
			.andExpect(content().string("Hello World")); 
	} 
}
```
- For slice testing, use `@WebMvcTest` instead

### WebTestClient
```java
@SpringBootTest 
@AutoConfigureWebTestClient 
class MockWebTestClientExampleTests { 
	@Test 
	void exampleTest(@Autowired WebTestClient webClient) { 		
			webClient.get().uri("/").exchange()
				.expectStatus().isOk()
				.expectBody(String.class).isEqualTo("Hello World"); 
	} 
}
```

## Testing with running server
- recommended to use a random port
	- `@LocalServerPort` can be used to autowire the actual port used in the test
- WebFlux : `WebTestClient` links to the running server for
- MVC : use `TestRestTemplate`
```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT) 
class RandomPortWebTestClientExampleTests { 
	@Test 
	void exampleTest(@Autowired WebTestClient webClient) {    //WebFlux
		webClient.get().uri("/").exchange()
				.expectStatus().isOk()
				.expectBody(String.class).isEqualTo("Hello World"); 	
	}
	@Test 			// MVC
	void exampleTest(@Autowired TestRestTemplate restTemplate) { 
		String body = restTemplate.getForObject("/", String.class); 	
		assertThat(body).isEqualTo("Hello World"); 
	}
}
```

### Customizing WebTestClient
- To customize the `WebTestClient` bean, configure a `WebTestClientBuilderCustomizer` bean. Any such beans are called with the `WebTestClient.Builder` that is used to create the `WebTestClient`.

## Using Metrics
- If you need to export metrics to a different backend as part of an integration test, annotate it with `@AutoConfigureMetrics`.

## Mocking & Spying Beans
- Spring Boot includes a `@MockBean` annotation that can be used to define a Mockito mock for a bean inside your `ApplicationContext`.
- **Can use to **
	- ***add*** new beans or 
	- ***replace*** a single existing bean definition.
- **Usage** : 
	- The annotation can be used directly 
		- on test classes, 
		- on fields within your test, or 
		- on `@Configuration` classes and fields. 
	- When used on a field, the instance of the created mock is also injected. Mock beans are automatically reset after each test method.
- **Cannot be used **: 
	- `@MockBean` cannot be used to mock the behavior of a bean that’s exercised during application context refresh. 
	- By the time the test is executed, the application context refresh has completed and it is too late to configure the mocked behavior. 
	- **Then What** : We recommend using a `@Bean` method to create and configure the mock in this situation.
- **Internal Details**
	- This is done via following 2 `TestExecutionListeners`
	- `@TestExecutionListeners({ MockitoTestExecutionListener.class, ResetMocksTestExecutionListener.class })`
	- (enabled by default when using `@SpringBootTest`)
- **Cached contexts** : 
	- While Spring’s test framework caches application contexts between tests and reuses a context for tests sharing the same configuration, the use of `@MockBean` or `@SpyBean` influences the cache key, which will most likely increase the number of contexts.
- **Note for SpyBean** : 
	- When you are using `@SpyBean` to spy on a bean that is proxied by Spring, you may need to remove Spring’s proxy in some situations, for example when setting expectations using `given` or `when`. Use `AopTestUtils.getTargetObject(yourProxiedSpy)` to do so.

## Slice Testing or Auto-configured Tests
- **How does it work** : 
	- a `@...Test` annotation loads an `ApplicationContext` and one or more `AutoConfigure...` annotations to customize auto-configuration settings
	- Scanning : each slice restricts component scan to appropriate components
- **Excluding from slice** : 
	- To exclude auto-configuration from slice annotations, the `@...Test` annotations have an `excludeAutoConfiguration` attribute. Alternative is `@ImportAutoConfiguration#exclude`
- **Include multiple slice** : 
	- using several slice `@...Test` annotations is NOT supported
	- pick one `@...Test` annotation and include the `@AutoConfigure...` annotations of the other slices manually

### Auto-configured Tests

|Annotation|Injected Fields|AutoConfigure|Description|
|---|---|---|---|
| `@JsonTest`|`JacksonTester<Vehicle>`|`@AutoConfigureJsonTesters`|test object JSON serialization-deserialization (has support for Jackson, Gson, JsonB)|
|`@WebMvcTest`|`MockMvc`|`@AutoConfigureMockMvc`|MVC Controllers;scans only controllers, json, etc.;|
|`@WebFluxTest`|`WebTestClient`|`@AutoConfigureWebTestClient`|WebFlux controllers|
|`@DataJpaTest`|`TestEntityManager`|`@AutoConfigureTestEntityManager`|entity classes & repositories;is Transactional|
|`@JdbcTest`|`JdbcTemplate`|`@AutoConfigureTestDatabase`(in memory DB)|only require a `DataSource` & not Spring-Data JDBC;configures in-memory DB|
|`@DataJdbcTest`|`JdbcTemplate`|`AutoConfigureTestDatabase`(in memory DB)|entity, repositories for tests using Spring-Data JDBC;default in-memory DB|
|`@RestClientTest`|`RestTemplateBuilder` & `MockRestServiceServer`|also auto-configures Jackson|to test REST clients|
|`@WebServiceClientTest`|`MockWebServiceServer`||tests that call web services|


### Auto-configure JSON Tests
- [Auto-configured JSON Tests](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-json-tests)

### Auto-configure Spring MVC Tests
- **Scanned** : 
	- `@WebMvcTest` auto-configures the Spring MVC infrastructure and limits scanned beans to `@Controller`, `@ControllerAdvice`, `@JsonComponent`, `Converter`, `GenericConverter`, `Filter`, `HandlerInterceptor`, `WebMvcConfigurer`, and `HandlerMethodArgumentResolver`. 
- **Not Scanned** : Regular `@Component` and `@ConfigurationProperties` beans are not scanned when the `@WebMvcTest` annotation is used. 
	- `@EnableConfigurationProperties` can be used to include `@ConfigurationProperties` beans.
- Use `@Import` for additional configuration
- **Test what?** :
	- `@WebMvcTest(...)` can specify single or multiple Controller classes to test
	- related beans have to be mocked out using `MockBean`
- **Auto-Configure** : 
	- auto-configuration is done via `AutoConfigureMockMvc`
	- so `MockMvc` can be autowired
```java
@WebMvcTest(UserVehicleController.class)   // SPECIFY CONTROLLER
class MyControllerTests { 
	@Autowired 
	private MockMvc mvc; 
	@MockBean 
	private UserVehicleService userVehicleService;   // MOCK DEPENDENCIES
	@Test 
	void testExample() throws Exception { 
		given(this.userVehicleService.getVehicleDetails("sboot")) 	
			.willReturn(new VehicleDetails("Honda", "Civic"));  //BDDMockito
		this.mvc.perform(get("/sboot/vehicle")  //MockMvc request
			.accept(MediaType.TEXT_PLAIN)) 	
			.andExpect(status().isOk())			//MockMvc verify
			.andExpect(content().string("Honda Civic")); 
	} 
}
```
- Can also use `WebClient` or Selenium `WebDriver`

### Auto-configured WebFlux tests
[Auto-configured Spring WebFlux Tests](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-webflux-tests)