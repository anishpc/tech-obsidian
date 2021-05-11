
## Scratch
- Use `spring-boot-starter-test` "Starter", which imports both Spring Boot test module as well as JUnit Jupiter, AssertJ, Hamcrest, and other libraries
- `@RunWith(SpringRunner.class)` for JUnit & `@SpringBootTest` annotation for Springs context
- By default, will find `@SpringBootConfiguration`. To override, use (1) nested configuration or (2) use classes attribute. To add rather than replace, use `@TestConfiguration`
- Use `webEnvironment` attribute to 
	- run a mock servlet environment
	- start a real server on a defined port
	- start a real server on a random port (use `TestRestTemplate` or `@LocalServerPort`)
- For environment properties, you can use the properties or value attribute, or 
	- you can use the `@TestPropertySource` annotation.`
- Mock Beans
	- replaces or adds a mock bean to the Application Context
	- Takes care of 
		- reseting the mock (no need for `@DirtiesContext`)
		- context cache issues
		- AOP Issues
	- `@SpyBean` is also available which does not replace/add beans but can spy on existing beans
- Testing application "slices"
	- test common slices
		- JSON
		- Data JPA
		- Rest Client
		- Spring MVC
- Annotations affecting : 
	- `@SpringBootTest`
	- `@ActiveProfiles`
	- `@TestPropertySource`
	- `@Import`
	- `@ContextConfiguration`

### Testing Spring Applications
- integration testing can be done with `ApplicationContext`

## Scratch : Pro Spring Boot 2
1. Primary goals of Spring Framework in integration testing are : 
	- managing the Spring IoC container caching between test execution
	- transaction management
	- dependency injection of test fixture instances
	- Spring-specific base classes
2. Spring Framework provides way to integrate `ApplicationContext` in tests via : 

Annotation | Description
|--------|-----|
`BootstrapWith`        | class level annotation to configure how the Spring `TestContext` framework is bootstrapped
| `ContextConfiguration` | defines class-level metadata to determine how to load and configure an `ApplicationContext` for integration tests.                                                                                           |
| `@WebAppConfiguration` | class-level annotation to declare that the `ApplicationContext` which loads for an integration test should be a `WebApplicationContext`                                                                      |
| `@ActiveProfile`       | class-level annotation to declare which bean definition profile(s) should be active when loading an `ApplicationContext` for an integration test |
| `@TestPropertySource`  | class-level annotation to configure the locations of properties files and inline properties to be added to the set of `PropertySources` in the `Environment` for an `ApplicationContext` loaded for the test |
| `@DirtiesContext`      | indicates that the underlying `ApplicationContext` has been dirtied during the execution of a test and should be closed |
|`@TestExecutionListeners`| Class-level annotation for configuring `TestExecutionListeners` that should be registered with the `TestContextManager`|
| miscellaneous          | `@TestExecutionListeners`, `@Commit`, `@Rollback`, `@Timed`, `@Repeat`, `@IfProfileValue`, etc.   |

3. Dependencies with `spring-boot-starter-test` : 
	- JUnit, AssertJ, Hamcrest, Mockito, JSONAssert, JsonPath
4. Annotations for running tests : `@RunWith(SpringRunner.class)` & `@SpringBootTest`
5. Testing web endpoints : 
	- use `@AutoConfigureMockMvc` on test class in addition to above 2 annotations; this also configures `MockMvc` which can be autowired in the test class
	- can also autowire `TestRestTemplate` in the test class
6. Mocking beans : 
	- using `@MockBean`, you can mock a new Spring bean or replace an existing definition
7. Testing Slices : 
	- `@JsonTest` : auto-configures JSON mapper (jackson, Gson, Jsonb)
	- `@WebMvcTest` : 
		- testing for controllers
		- limits scanning to `@Controller`, `@ControllerAdvice`, `@JsonComponent`, `Converter`, `GenericConverter`, `Filter`, `WebMvcConfigurer`, `HandlerMethodArgumentResolver`
		- beans marked with `@Component` are NOT scanned with this annotation, but you can still use `@MockBean` if needed
	- `@WebFluxTest` : 
		- similar to above
		- provides `WebTestClient` which can be auto-wired in test class
	- `@DataJpaTest`
		- provides `TestEntityManager` which can be auto-wired in test class
		- by default uses in-memory database engines (H2, Derby, HSQL); for real database testing, add `@AutoConfigureTestDatabase(replace=NONE)` on the test class along with `@DataJpaTest`
	- `@JdbcTest`
		- much similar to `@DataJpaTest` but for pure JDBC-related tests
		- provides `JdbcTemplate` for autowiring in test class
		- omits classes with `@Component`
	- `@RestClientTest(MyService.class)`
		- auto-configures Jackson, GSON, JSONB support
		- configures `RestTemplateBuilder` and support for `MockRestServiceServer`

### Context Caching

## Scratch : LiveLessons
1. `@TestPropertySource`
2. `@RunWith(SpringRunner.class)`
3. `@SpringBootTest(webEnvironment = ...)` -- below
	- this searches in the package upwards to find `@SpringBootApplication` class
	- also gets `TestRestTemplate` auto-configured to invoke this application
4. `@MockBean`
	- replaces or adds a mock bean to the ApplicationContext
	- takes care of
		- resetting the mock ( no need for `@DirtiesContext`)
		- Context cache issues
		- AOP issues
	- `@SpyBean` is also available
5. `@JsonTest`
	- marked on the test class
	- provides `JacksonTester<...>` (also support for `GsonTester`) to be autowired
	- We can test both serialize & deserialize testing with `JacksonTester` with `jacksonTester.write(object)` and `jacksonTester.parseObject(str)`. AssertJ has convenient methods to test equality with json files or to test a specific value with jsonpath.
6. `@DataJpaTest`
	- test domain logic and JPA mapping
	- auto-configures Hibernate, Spring Data and DB
	- uses in-memory DB by default
	- provides `TestEntityManager` bean to be autowired
7. `@RestClientTest` -- testing class which calls remote endpoint
	- the `@RestClientTest` annotation would specify the service class to be tested and also the properties POJO as `@RestClientTest({MyService.class, MyPropPojo.class})`
		- also can use the `@TestPropertySource` annotation on the class to replace the remote endpoint URL
		- use to test `RestTemplate` calls (should be via `RestTemplateBuilder`) 
		- `restTemplate` in the service class is not autowired but created using the `RestTemplateBuilder` which is autowired.
	- auto-configures `RestTemplate` beans like Jackson, HttpMessageConverters, etc
	- provides a `MockRestServiceServer` bean to be autowired
		- mocks out remote rest calls
		```java
		mockRestSvcServer.expect(requestTo("http://remote-endpoint/mypath")).andRespond(withSuccess(getClassPathResource("success_response.json"), APPLICATION_JSON))
		```
8. `@WebMvcTest`
	- used to test Spring MVC mappings (controllers)
		- we can choose a specific controller `@WebMvcTest(MyController.class)` or all controllers
		- service classes could be mocked out using `MockBean`
	- auto-configures web components
	- provides `MockMvc` to be autowired to invoke the application controller
	- optionally will work with HTMLUnit or Selenium
9. Slice Annotations : 
	- slice annotations are composed as follows 
		- usually `@OverrideAutoConfiguration`
			- disables full auto-configuration
			- allows tests to selectively import parts of configuration that is needed
			- reason : for slice annotation, not all beans are required, hence 
		- some `@TypeExcludeFilters`
			- filter beans that are included via component scanning
			- works with `@TypeExcludeFilter`
			- important as `@SpringBootApplication` is used
		- one or more `@AutoConfigure` annotations
			- specific auto-configuration imports (like for using `MockMvc` beans)
			- meta-annotated with `@ImportAutoConfiguration`
			- load configuration classes via `spring.factories`
		- possibly a `@PropertyMapping("my.prop.name")` annotation
			- means that all fields in that class on which this annotation is placed would have the `my.prop.name` prefix
			- often used on `@AutoConfigure` annotations
			- map annotation attributes to `Environment` properties
			- allows test configuration classes to use `@Conditions`
			- all fields
	- Example `@JsonTest` annotation
		![jsontest_annotation.png](jsontest_annotation.png)
		- Top 4 are standard Java annotation
		- `@BootstrapWith` -- helps bootstrap the way the context is loaded; equivalent of `SpringBootTest`
		- `OverrideAutoConfiguration(enabled=false)` -- not to apply the regular auto-configuration it usually would
		- `TypeExcludeFilter` : want to exclude based on `JsonExcludeFilter`
		- `@AutoConfigureCache` : to setup caching; loads the `CacheAutoConfiguration` 
		- `@AutoConfigureJson` : `ImportAutoConfiguration` file loads the file in `spring.factories` file which are `GsonAutoConfiguration` & `JacksonAutoConfiguration`
		- `@AutoConfigureJsonTesters` - internally has `@PropertyMapping("spring.test.jsontesters")` & also has `ImportAutoConfiguration` which reads
	- Can override defaults for existing slice annotation
		- `@DataJpaTest` + `@AutoConfigureTestDatabase(replace=NONE)`
10. Using WireMock
	- add `wiremock-standalone` dependency
	- use JUnit `WireMockRule`
		- can use dynamicport : `WireMockConfiguration.options().dynamicPort()`
		- autowire the properties POJO in the test & set the URL to localhost & port to `wiremock.port()` in the `setup` method
	- define stubs



## Testing Spring Boot Applications
> External properties, logging, and other features of Spring Boot are installed in the context by default only if you use `SpringApplication` to create it
- `SpringBootTest` annotation can be used as an alternative to standard `ContextConfiguration` annotation. The annotation creates the `ApplicationContext` used in the tests through `SpringApplication`
- JUnit : 
	- for JUnit-4, add `@RunWith(SpringRunner.class)`to your test, otherwise the annotations would be ignored
	- for JUnit-5, the equivalent is `@ExtendWith(SpringExtension.class)`, however, NO need to add that as `@SpringBootTest` and the other `@...Test` annotations are already annotated with it
- Server
	- By default, `@SpringBootTest` will NOT start a server. You can use the `webEnvironment` attribute of `@SpringBootTest` to refine how your tests run
		- `MOCK` (Default) :
			- Loads a web `ApplicationContext` and provides a mock web environment
			- Embedded servers are not started
			- If web environment is not on classpath then falls back to non-web `ApplicationContext`
			- Can be used with `AutoConfigureMockMvc` or `AutoConfigureMockWebTestClient` 
		- `RANDOM_PORT` : 
			- Loads a `WebServerApplicationContext` and provides a real web environment
			- Embedded servers are started and listen on a random port
		- `DEFINED_PORT` :
			- Loads a `WebServerApplicationContext` and provides a real web environment
			- Embedded servers are started and listen on a defined port (from your `application.properties`) or on the default port of `8080`
		- `NONE` : 
			- Loads an `ApplicationContext` by using `SpringApplication` but does not provide _any_ web environment (mock or otherwise).
	- Management server 
		- `@SpringBootTest` with `webEnvironment = WebEnvironment.RANDOM_PORT` will also start the management server on a separate random port if your application uses a different port for the management server.

### Detecting Test Configuration
- Spring Boot's `@*Test` annotations search for your primary configuration automatically whenever you do not explicitly define one. (so `@ContextConfiguration` is often not required)
	- It searches ***up from*** the package that contains the test until it finds a class annotated with `@SpringBootApplication` or `@SpringBootConfiguration`
- If you want to customize the primary configuration, you can use a nested `@TestConfiguration` class. A nested `@TestConfiguration` class is used in addition to your application's primary configuration (unlike a nested `@Configuration` class)
- Caching AppContext : 
	- Spring’s test framework caches application contexts between tests. Therefore, as long as your tests share the same configuration (no matter how it is discovered), the potentially time-consuming process of loading the context happens only once

### Excluding Test Configuration
- If your application uses component scanning (for example, if you use `@SpringBootApplication` or `@ComponentScan`), top-level configuration classes that you created only for specific tests get picked up everywhere
- `@TestConfiguration` can be used on an inner class of a test to customize the primary configuration
- When placed on a top-level class, `@TestConfiguration` indicates that classes in `src/test/java` should not be picked up by scanning. You can then import that class explicitly where it is required like : 
```java
@SpringBootTest
@Import(MyTestsConfiguration.class)
class MyTests {
    @Test
    void exampleTest() {
        ...
    }
}
```

