
## ConfigurationProperties
- A way to map properties to POJO
- IDE support so properties have recommendation & documentation
- Type-safe (coerced by Spring converters)
- Can be validated (`@Valid`) (JSR-303)

## Property Sources
### Inject property values
- Property values can be 
	- injected directly into your beans by using the `@Value` annotation
	- accessed through Spring's `Environment` abstraction
	- or be bound to structured objects (like POJOs) via `@ConfigurationProperties`

### Property Source Order
- Spring Boot uses a very particular `PropertySource` order that is designed to allow sensible overriding of values. 
Order here : [Spring-Boot-ExternalConfig-Order](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-external-config)

### Config data files
- Config data files are considered in the following order : 
1. Application properties inside the jar (`application.properties` or YAML)
2. Profile-specific application properties inside the jar (`application-{profile}.properties` or YAML)
3. Application properties outside the jar (`application.properties` or YAML)
4. Profile-specific application properties outside the jar (`application-{profile}.properties` or YAML)
Note : If you have configuration files with both `.properties` and `.yml` format in the same location, `.properties` takes precedence. (recommended to stick with one format for the entire application)
> The `env` and `configprops` endpoints can be useful in determining why a property has a particular value. You can use these two endpoints to diagnose unexpected property values.

### Command-line properties
- By default, `SpringApplication` converts any command line option arguments (that is, arguments starting with `--` like `--spring.port=9090` ) to a property and adds them to the Spring `Environment` (command-line properties take precedence over file based property sources)
- If you do not want command line properties to be added to the `Environment`, you can disable them by using `SpringApplication.setAddCommandLineProperties(false)`.

### JSON Application properties
- Environment variables and system properties often have restrictions that mean some property names cannot be used. To help with this, Spring Boot allows you to encode a block of properties into a single JSON structure.
- When your application starts, any `spring.application.json` or `SPRING_APPLICATION_JSON` properties will be parsed and added to the `Environment`.
```
// The property value is "acme.name=test"
SPRING_APPLICATION_JSON='{"acme":{"name":"test"}}' java -jar myapp.jar
//OR
java -Dspring.application.json='{"acme":{"name":"test"}}' -jar myapp.jar
//OR
java -jar myapp.jar --spring.application.json='{"acme":{"name":"test"}}'
```

### External application properties
- Spring boot automatically will find and load `application.properties` & `application.yaml` from the following locations when the application starts
1. classpath root
2. classpath `/config` package
3. current directory
4. `/config` subdirectory
5. immediate child directories of the `/config` subdirectory
- can refer to an explicit location using `spring.config.location`
#### Optional locations 
[Spring-Boot-Config-OptionalLocations](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-external-config-optional-prefix)
#### Wildcard locations
[Spring-Boot-ExternalConfigFiles-Wildcard](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-external-config-files-wildcards)

### Profile specific files
- Which files are loaded
	- For example, if your application activates a profile named `prod` and uses YAML files, then both `application.yml` and `application-prod.yml` will be considered.
- Last-wins in case of multiple profiles
	- if profiles `prod,live` are specified by the `spring.profiles.active` property, values in `application-prod.properties` can be overridden by those in `application-live.properties`.
- Default only if NO active profile
	- If no profile is explicitly activated then `application-default` is considered. Note: `application-default` is NOT considered if any profile is active.

### Importing additional data
[Spring-Boot-ExternalConfigFiles-Importing](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-external-config-files-importing)

### Property Placeholders
- The values in `application.properties` and `application.yml` are filtered through the existing `Environment` when they are used, so you can refer back to previously defined values (for example, from System properties).
```properties
app.name=MyApp app.description=${app.name} is a Spring Boot application
```

## Encrypting properties
- Spring Boot does ***NOT*** provide any built in support for encrypting property values. 
- The `EnvironmentPostProcessor` interface allows you to manipulate the `Environment` before the application starts.
- [Spring Boot-CustomizeEnvironment-Before-AppContext](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#howto-customize-the-environment-or-application-context)

## Actuator Check
- if actuator is enabled then the properties can be checked at `localhost:8080/configprops`

## Type Safe Configuration Properties
- using `@Value("{property}"` can be cumbersome if working with multiple properties or data is hierarchical in nature
- strongly typed beans can be used to govern and validate the configuration

### Java Bean properties binding
```java
@ConfigurationProperties("acme") 
public class AcmeProperties { 
	private boolean enabled;
	private InetAddress remoteAddress;   // can be coerced from String
	private List<String> roles = new ArrayList<>(Collections.singleton("USER"));  //default value is USER
	private final Security security = new Security();
	...
	public static class Security { 	//inner class
		private String username;
		...
	}
}
```
```properties
acme.enabled=false
acme.remote-address="some address"
acme.security.username="someuser"   # nested value
```
- LOMBOK : if using Lombok, make sure that Lombok does not generate any particular constructor for such a type, as it is used automatically by the container to instantiate the object.
- Add setters for the above POJO

### Constructor property binding
- Above example using constructor binding and making it ***immutable***
- the binder will expect to find a constructor with the parameters that you wish to have bound.
- Nested members of a `@ConstructorBinding` class (such as `Security` in the example) will also be bound via their constructor.
- NOTE : If you have more than one constructor for your class you can also use `@ConstructorBinding` directly on the constructor that should be bound.
```java
@ConstructorBinding 
@ConfigurationProperties("acme") 
public class AcmeProperties { 
	private final boolean enabled;
	...
	public AcmeProperties(boolean enabled, 
	InetAddress remoteAddress, 
	Security security,
	@DefaultValue("USER") List<String> roles) { 
		this.enabled = enabled; 
		this.remoteAddress = remoteAddress; 
		this.security = security; 
		this.roles = roles
	}
	public static class Security {
		public Security(String username) {
			this.username = username;	
		}
	}
}
```
> To use constructor binding the class must be enabled using `@EnableConfigurationProperties` or configuration property scanning.

### Enabling `@ConfigurationProperties`-annotated types
- Sometimes, classes annotated with `@ConfigurationProperties` might not be suitable for scanning, for example, if you’re developing your own auto-configuration or you want to enable them conditionally. In these cases, specify the list of types to process using the `@EnableConfigurationProperties` annotation
```java
@Configuration(proxyBeanMethods = false) @EnableConfigurationProperties(AcmeProperties.class) public class MyConfiguration { }
```

NOTE : `proxyBeanMethods` 
- Specify whether `@Bean` methods should get proxied in order to enforce bean lifecycle behavior, e.g. to return shared singleton bean instances even in case of direct `@Bean` method calls in user code. This feature requires method interception, implemented through a runtime-generated CGLIB subclass which comes with limitations such as the configuration class and its methods not being allowed to declare `final`.
- The default is `true`, allowing for 'inter-bean references' via direct method calls within the configuration class as well as for external calls to this configuration's `@Bean` methods, e.g. from another configuration class. If this is not needed since each of this particular configuration's `@Bean` methods is self-contained and designed as a plain factory method for container use, switch this flag to `false` in order to avoid CGLIB subclass processing.
- Turning off bean method interception effectively processes `@Bean` methods individually like when declared on non-`@Configuration` classes, a.k.a. "@Bean Lite Mode

### Using `@ConfigurationProperties`
- Using in service
```java
@Service 
public class MyService { 
	private final AcmeProperties properties; 
	public MyService(AcmeProperties properties) { 
		this.properties = properties; 
	}
	..
}
```
- Using `@ConfigurationProperties` also lets you generate metadata files that can be used by IDEs to offer auto-completion for your own keys. [Spring-Boot-Configuration-Properties-IDE](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#configuration-metadata)

## Relaxed Binding
- uses relaxed rules for binding `Environment` properties to `@ConfigurationProperties`
```java
@ConfigurationProperties(prefix="acme.my-project.person") 
public class OwnerProperties { 
	private String firstName;
	...
}
```
|Property|Details|
|-------|------|
|`acme.my-project.person.first-name`|kebab case (recommended in `.properties` & `.yml`)|
| `acme.myProject.person.firstName`| Standard camel case|
|`acme.my_project.person.first_name`| underscore notation (also used in `.properties` & `.yml`)|
|`ACME_MYPROJECT_PERSON_FIRSTNAME`|Upper case format, which is recommended when using system environment variables.|

### Binding Maps
- [Spring-Boot-Property-Binding-Maps](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-external-config-relaxed-binding-maps)
- The properties above will bind to a `Map` with `/key1`, `/key2` and `key3` as the keys in the map. The slash has been removed from `key3` because it wasn’t surrounded by square brackets.
```properties
acme.map.[/key1]=value1 
acme.map.[/key2]=value2 
acme.map./key3=value3
```

### Binding from Environment Variables
- To convert a property name in the canonical-form to an environment variable name you can follow these rules
	- Replace dots (`.`) with underscores (`_`).
	- Remove any dashes (`-`).
	- Convert to uppercase.
- For example, the configuration property `spring.main.log-startup-info` would be an environment variable named `SPRING_MAIN_LOGSTARTUPINFO`.
- Binding Lists :
	- Environment variables can also be used when binding to object lists. To bind to a `List`, the element number should be surrounded with underscores in the variable name.
	- For example, the configuration property `my.acme[0].other` would use an environment variable named `MY_ACME_0_OTHER`.

## Merging Complex Types
- When lists are configured in more than one place, overriding works by replacing the entire list.
- For `Map` properties, you can bind with property values drawn from multiple sources. However, for the same property in multiple sources, the one with the highest priority is used.
- More details : [Spring-Boot-Merging-Complex-Types](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-external-config-complex-type-merge)

## Properties Conversion
- Spring Boot attempts to coerce the external application properties to the right type when it binds to the `@ConfigurationProperties` beans.
- If you need custom type conversion, you can provide 
	- a `ConversionService` bean (with a bean named `conversionService`) or 
	- custom property editors (through a `CustomEditorConfigurer` bean) or 
	- custom `Converters` (with bean definitions annotated as `@ConfigurationPropertiesBinding`).
- NOTE : bean requested early during application life-cycle, so limit dependencies

### Converting Duration
- Spring Boot has dedicated support for expressing durations (`java.time.Duration`)
	- -   A regular `long` representation (using milliseconds as the default unit unless a `@DurationUnit` has been specified)
	- The standard ISO-8601 format [used by `java.time.Duration`]
	- A more readable format where the value and the unit are coupled (e.g. `10s` means 10 seconds)
```java
@ConfigurationProperties("app.system") 
public class AppSystemProperties { 
	@DurationUnit(ChronoUnit.SECONDS) 
	private Duration sessionTimeout = Duration.ofSeconds(30); 
	
	private Duration readTimeout = Duration.ofMillis(1000);
	...
}
```