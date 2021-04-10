### Overview
- Spring’s `@Conditional` annotation allows us to define conditions under which a certain bean is included into that object graph.

### Examples
- Some beans work in test context only
- Another use case is to enable or disable certain cross cutting concerns

#### Annotate Locations
- can annotate `@Configuration` classes or individual `@Bean` methods

## Evaluation
- The `@ConditionalOn..` evaluations for auto-configuration on the `@Configuration` classes are evaluated based on the conditions and in short-circuiting fashion. 
- Configuration class/Bean creation : 
	- Also, if the conditions don't match then the `@Configuration` class is not configured and also the beans

### Configuration & Bean loading on condition-match
- No effect on creation
	- `@ConditionalOnBean` and `@ConditionalOnMissingBean` do not prevent `@Configuration` classes from being created. 
- Bean registration 
	- The only difference between using these conditions at the class level and marking each contained `@Bean` method with the annotation is that the former prevents registration of the `@Configuration` class ***as a bean*** if the condition does not match.
	- 
## Declaring Conditional Beans
### Class Conditions
- The `@ConditionalOnClass` and `@ConditionalOnMissingClass` annotations let `@Configuration` classes be included based on the presence or absence of specific classes. 
- Class loading : 
	- due to the fact that annotation metadata is parsed by using ASM, you can use the `value` attribute to refer to the real class, even though that class might not appear on the running application classpath. You can also use the `name` attribute if you prefer to specify the class name by using `String` value
	- does not apply to `@Bean` methods where typically the return type is the target of the condition : before the condition on the method applies, the JVM will have loaded the class and potentially processed the method references which will fail if the class is not present
```java
@Configuration
// Some conditions
public class MyAutoConfiguration {
	// Auto-configured beans
	@Configuration
	@ConditionalOnClass(EmbeddedAcmeService.class)
	static class EmbeddedConfiguration {
		@Bean
		@ConditionalOnMissingBean
		public EmbeddedAcmeService embeddedAcmeService() { ... }
	}
}
```
### Bean Conditions
- The `@ConditionalOnBean` and `@ConditionalOnMissingBean` annotations let a bean be included based on the presence or absence of specific beans. You can use the `value` attribute to specify beans by type or `name` to specify beans by name. 
- The `search` attribute lets you limit the `ApplicationContext` hierarchy that should be considered when searching for beans.
- When placed on a `@Bean` method, the target type defaults to the return type of the method
```java
@Configuration
class ConditionalBeanConfiguration {
  @Bean
  @ConditionalOnMissingBean   // <-- no need to specify type
  public MyService myService() { ... }
}
```
#### Bean definition order
- Which bean definitions are considered are based on what has been processed so far. 
> Use `@ConditionalOnBean` and `@ConditionaOnMissingBean` ONLY on auto-configuration classes as these are guaranteed to load after the user-defined bean definitions have been added
#### @CondionalOnBean
- load a bean only if a certain other bean is available in the application context
```java
@Configuration
@ConditionalOnBean(OtherModule.class)
class DependantModule {
  ...
}
```

#### @ConditionalOnMissingBean
- if we want to load a bean only if a certain other bean is _not_ in the application context
```java
@Configuration
class OnMissingBeanModule {

  @Bean
  @ConditionalOnMissingBean
  DataSource dataSource() {
    return new InMemoryDataSource();
  }
}
```


### Property Conditions
```java
@Configuration
@ConditionalOnProperty(
    value="module.enabled", 
    havingValue = "true", 
    matchIfMissing = true)
class CrossCuttingConcernModule {
  ...
}
```
- The `@ConditionalOnProperty` annotation lets configuration be included based on a Spring Environment property. 
- Usage : Use the `prefix` and `name` attributes to specify the property that should be checked.
- The `CrossCuttingConcernModule` is only loaded if the `module.enabled` property has the value `true`. If the property is not set at all, it will still be loaded, because we have defined `matchIfMissing` as `true`.

### SpEL expression Conditions
```java
@Configuration
@ConditionalOnExpression(
    "${module.enabled:true} and ${module.submodule.enabled:true}"
)
class SubModule {
  ...
}
```
- By appending `:true` to the properties we tell Spring to use `true` as a default value in the case the properties have not been set.

### Resource Conditions, Web Conditions
- `@ConditionalOnResource`
- `@ConditionalOnWebApplication`, `@ConditionalOnNotWebApplication`. We can provide more filtering on servlet & reactive web application

### Conditional @Configuration
- all beans contained within this configuration will only be loaded if the condition is met
```java
@Configuration
@Conditional... // <--
class ConditionalConfiguration {
  
  @Bean
  Bean bean(){
    ...
  };
  
}
```

### Conditional @Component
```java
@Component
@Conditional... // <--
class ConditionalComponent {
}
```

## Testing Conditional 
- [49. Creating Your Own Auto-configuration (spring.io)](https://docs.spring.io/spring-boot/docs/2.1.11.RELEASE/reference/html/boot-features-developing-auto-configuration.html#boot-features-test-autoconfig)

## Condition evaluation order
- The `@Conditional..` annotations have a `Condition` implementation class which has the order specified using `@Order`. (lower the number, higher the precedence)
- The conditions are evaluated based on that order -- `@ConditionalOnClass`, `@ConditionalOnProperty` have the highest order. 
- Shortcut evaluation : since `Conditional` annotations evaluation is short-circuited, it's preferable to use `ConditionalOn...` annotations which have higher-order (& are typically faster)
- Custom order on `@ConditionalOn` classes
	- The `@Order` annotation is placed on the `Condition` implementation associated with the `@ConditionalOn..` configuration. To place order on the `@Configuration` itself, use the `@AutoConfigureAfter` or `@AutoConfigureBefore` annotations
	- You can use the `@AutoConfigureAfter` or `@AutoConfigureBefore` annotations if your configuration needs to be applied in a specific order. For example, if you provide web-specific configuration, your class may need to be applied after `WebMvcAutoConfiguration`
	- If you want to order certain auto-configurations that should not have any direct knowledge of each other, you can also use `@AutoConfigureOrder`. That annotation has the same semantic as the regular `@Order` annotation but provides a dedicated order for auto-configuration classes.
- Example of Spring's own `WebMvcAutoConfiguration`
```java
@ConditionalOnClass({Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class})  
@ConditionalOnMissingBean({WebMvcConfigurationSupport.class})  
@AutoConfigureOrder(-2147483638)     // highest precedence
@AutoConfigureAfter({DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class, ValidationAutoConfiguration.class})  // RELATIVE ORDER
public class WebMvcAutoConfiguration {
...
}
```

## Custom Conditions
[[Spring Boot Custom Conditions]]