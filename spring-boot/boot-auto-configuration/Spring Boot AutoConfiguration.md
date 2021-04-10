
- `@EnableAutoConfiguration` loads `/META-INF/spring.factories`
- `spring.factories` declares `@Configuration` classes
- Each `@Configuration` is `@Conditional`
- Typically, if you provide a custom implementation, Spring Boot would back away. If that's not happening then you can use the `exclude` parameter in the `@EnableAutoConfiguration` annotation to exclude that configuration explicitly

## AutoConfiguration
- which files should be considered as configuration classes
- which situation should these configuration classes be applied

### Configuration classes
- Spring looks for `spring.factories` in the `META-INF` directory of all jars that the application depends on
> Auto-configurations must be loaded *only* via `spring.factories` file. Make sure they are defined in a specific package space and not the target of component scanning. Also, auto-configuration should NOT enable component scanning to find additional components. Specific `@Import`s should be used instead.
- `spring.factories` declares a list of `@Configuration` classes
- In addition, these `@Configuration` classes are annotated with an `@Conditional` annotation
- Some conditions are : 
	- if a certain class is on the classpath (e.g. Tomcat)
	- if the user hasn't declared a specific bean
- Example conditional class
	- has `@Configuration` which defines beans as standard configuration classes
	- classes `MimeMessage` & `MimeType` should be on the classpath
	- property `spring.mail.host` should be present (value of property is irrelevant). If value was relevant then use the property `hasValue`
	- Bean of type `MailSender` is missing - this means that user is not overriding it
	- `EnableConfigurationProperties` is for mapping properties-POJO
```java
@Configuration
@ConditionalOnClass({ MimeMessage.class, MimeType.class})
@ConditionalOnProperty(prefix = "spring.mail", value = "host")
@ConditionalOnMissingBean(MailSender.class)
@EnableConfigurationProperties(MailProperties.class)
public class MailSenderAutoConfiguration {
	@Autowired
	MailProperties properties;
	
	@Bean
	public JavaMailSender mailSender() {
		// create the bean
	}
}
```
- The `@ConditionalOnMissingBean` annotation can also be placed on the `@Bean` method `mailSender`.

#### Auto-configure order
- You can use the `@AutoConfigureAfter` or `@AutoConfigureBefore` annotations if your configuration needs to be applied in a specific order. For example, if you provide web-specific configuration, your class may need to be applied after `WebMvcAutoConfiguration`
- If you want to order certain auto-configurations that should not have any direct knowledge of each other, you can also use `@AutoConfigureOrder`. That annotation has the same semantic as the regular `@Order` annotation but provides a dedicated order for auto-configuration classes.

### Spring OOB Auto-configure
- browse the source code of `spring-boot-autoconfigure` to see the `@Configuration` classes that Spring provides (see the `META-INF/spring.factories` file)

### Condition Annotations
- You almost always want to include one or more `@Conditional` annotations on your auto-configuration class. 
- The `@ConditionalOnMissingBean` annotation is one common example that is used to allow developers to override auto-configuration if they are not happy with your defaults.

#### Class Conditions

## Component Scanning
- If the package has a namespace such that the auto-configuration is on the component scanning path then the `AutoConfigurationExcludeFilter` avoids that
- If the class is defined as 'auto-configuration' then the component scanning excludes it
![spring-boot-exclude-filter.png](spring-boot-exclude-filter.png)