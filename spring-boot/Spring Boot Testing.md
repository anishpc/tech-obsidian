
## Scratch
- Use `spring-boot-starter-test` "Starter", which imports both Spring Boot test module as well as JUnit Jupiter, AssertJ, Hamcrest, and other libraries

## Testing Spring Applications
- integration testing can be done with `ApplicationContext`

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