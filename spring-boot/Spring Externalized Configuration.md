
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
- For example, if your application activates a profile named `prod` and uses YAML files, then both `application.yml` and `application-prod.yml` will be considered.
- Profile-specific properties are loaded from the same locations as standard `application.properties`, with profile-specific files always overriding the non-specific ones. If several profiles are specified, a last-wins strategy applies.
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