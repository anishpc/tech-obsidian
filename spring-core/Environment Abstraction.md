### `Environment` interface

- The `Environment` interface is an abstraction integrated in the container that models two key aspects of the application 
	- profiles
	- properties
- profile
	- profile is a named, logical group of bean definitions to be registered with the container only if the given profile is active. Beans may be assigned to a profile. 
	- The role of the `Environment` object with relation to profiles is determining which profiles (if any) are currently active, and which profiles (if any) should be active by default
- properties
	- The role of the `Environment` object with relation to properties is to provide the user with a convenient service interface for configuring property sources and resolving properties from them.


### Bean Definition Profiles
Spring profiles are configured by
- specifying which beans are part of which profile
- specifying which profiles are active 

> There are no limits on the number of profiles which can be defined. However, the profiles are represented as Integers, so the int limit applies

### Specifying Beans with Profiles
- use `@Profile` annotation at the `@Component` class level -- bean will be part of profile(s) specified in annotation
- use `@Profile` annotation at the `@Configuration` class level -- all beans will be part of the profile(s) mentioned
- use `@Profile` annotation at the `@Bean` method of `@Configuration` class -- instance of bean returned by this method would be part of the profile(s) mentioned
- use `@Profile` to define custom annotation
- not defined : if Bean does not have profile specified, it would be created in every profile

#### Profile Defn : Component level
```java
@Component
@Profile("development")
public class DevDatabaseDao implements UserDataDa0 {
}
```

#### Profile Defn : Configuration level
```java
@Configuration
@Profile("development")
public class StandaloneDataConfig {
	@Bean
	public DataSource dataSource() {
		return new EmbeddedDatabaseBuilder()....;
	}
}

@Configuration
@Profile("production")
public class JndiDataConfig{
	@Bean
	public DataSource dataSource() throws Exception {
		return new InitialContext()....;
	}
}

@Configuration
@Import({StandaloneDataConfig.class,JndiDataConfig.class})
public class AppConfig{
}
```
#### Profile Defn : Bean method level
```java
@Configuration
public class DataConfig{
	@Bean
	@Profile("production")
	public DataSource dataSource() throws Exception {
		return new InitialContext()....;
	}
	
	@Bean
	@Profile("development", "test")  //can also specify multiple profiles
	public DataSource devDataSource() {
		return ...//h2 database
	}
}
```
#### Profile Defn : Custom Meta-annotation
```java
@Target(ElementType.TYPE) 
@Retention(RetentionPolicy.RUNTIME) 
@Profile("production")    // put the profile as a meta-annotation
public @interface Production { }
```
- This custom annotation can be used in any of the above scenarios of profile definition

### Profiles : profile expression
- profile string could be simple profile name (`development`) or a profile expression (`production & us-east`). The following operators are supported :
	- `!`
	- `&`
	- `|`
- NOTE : parenthesis required for mixing `&` and `|` operators
### Activating a profile
Profile can be activated in many ways
- programmatically with the use of `ConfigurableEnvironment`
- using `spring.profiles.active` property
- On JUnit test using `@ActiveProfiles` annotation
- In Spring Boot, programmatically by usage of `SpringApplicationBuilder`

#### Activating profile : Programmatically
- The `getEnvironment` method returns the `ConfigurableEnvironment` which has the method to set the active profiles.
```java
//programmatically
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(); ctx.getEnvironment().setActiveProfiles("development");
//ctx.getEnvironment().setActiveProfiles("dev", "test") // multiple profiles
ctx.refresh();
```
#### Activating profile : properties
- `spring.profiles.active` property can be specified through 
	- system environment variables , 
	- JVM system properties (`-Dspring.profiles.active="profile1,profile2"`)
	- servlet context properties in `web.xml`
#### Activating profiles : JUnit test
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = ApplicationConfig.class)
@ActiveProfiles("development")   //this profile would be active
public class AppConfigTest {
}
```

#### Activating profiles : Spring Boot
```java
@SpringBootApplication
public class Runner implements CommandLineRunner {
	public static void main(String[] args){
		new SpringApplicationBuilder(Runner.class)
							.profiles("production")  //profile activated
							.run(args);
	}
}
```
#### Default profile
```java
@Configuration 
@Profile("default") 
public class DefaultDataConfig {
```
- If no profiles are active, the configuration in `default` profile is used. If any other profile is active then `default` profile does **NOT** apply
- You can change the name of the default profile by using `setDefaultProfiles()` on the `Environment` or ,declaratively, by using the `spring.profiles.default` property.

### Profiles Use cases
- Changing behaviour of system for different environments
- Changing set of beans for testing
- Changing/Activating set of beans for monitoring or additional debugging

## Property Source Abstraction
### Property Source Hierarchy
- Property values are not merged but the highest order souce value is taken
- Default property source is `StandardEnvironment` and for web it is `StandardServletEnvironment` 
- For a common `StandardServletEnvironment`, the full hierarchy is as follows, with the highest-precedence entries at the top:
	1. ServletConfig parameters (if applicable — for example, in case of a `DispatcherServlet` context)
	2. ServletContext parameters (web.xml context-param entries)
	3. JNDI environment variables (`java:comp/env/` entries)
	4. JVM system properties (`-D` command-line arguments)
	5. JVM system environment (operating system environment variables)

### Spring Boot property sources
- [24. Externalized Configuration (spring.io)](https://docs.spring.io/spring-boot/docs/1.5.22.RELEASE/reference/html/boot-features-external-config.html)

### Adding your own property source

#### Adding property source programmatically
```java
ConfigurableApplicationContext ctx = new GenericApplicationContext(); MutablePropertySources sources = ctx.getEnvironment().getPropertySources(); sources.addFirst(new MyPropertySource());
```

#### Adding property source using `@PropertySource`
- The `@PropertySource` annotation provides a convenient and declarative mechanism for adding a `PropertySource` to Spring’s `Environment`.
```java
@Configuration 
@PropertySource("classpath:/com/myco/app.properties") 
public class AppConfig { 
	@Autowired 
	Environment env; 
	
	@Bean 
	public TestBean testBean() { 
		TestBean testBean = new TestBean();
		testBean.setName(env.getProperty("testbean.name")); 
		return testBean; 
	} 
}
```
- Any `${…​}` placeholders present in a `@PropertySource` resource location are resolved against the set of property sources **already registered** against the environment. So `${my.placeholder}` is NOT resolved from app.properties defined
```java
@Configuration @PropertySource("classpath:/com/${my.placeholder:default/path}/app.properties") public class AppConfig {
...
}
```

### Property using `@Value`
- Usage : Inside `@Value`, you can specify:
- Simple value : `@Value("John")`
- Boolean value : `@Value("true")`
- Reference a property : `@Value("${number.of.connections}")`
- SpEL :
	-  `@Value("#{'John'.toUpperCase()}")`
	-  `@Value("#{'${app.department}'.toUpperCase()}")`
-  Inject values in Array, List, Set (might require Spring `ConversionService`)
- NOTE : `@PropertySource` has to be defined & these values are populated from property sources

### Property Value usage
```java
public class SpringBean {

	//Constructor
	public SpringBean(@Value("{app.department}") String department) {}
	
	//Setter method
	@Value("{app.contact}")
	public void setContact(String contactEmail) {}
	
	//Setter param
	@Autowired
	public void setName(@Value("${app.support.name}") String name) {}
}
```