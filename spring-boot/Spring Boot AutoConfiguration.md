
- `@EnableAutoConfiguration` loads `/META-INF/spring.factories`
- `spring.factories` declares `@Configuration` classes
- Each `@Configuration` is `@Conditional`
- Typically, if you provide a custom implementation, Spring Boot would back away. If that's not happening then you can use the `exclude` parameter in the `@EnableAutoConfiguration` annotation to exclude that configuration explicitly

## AutoConfiguration
- which files should be considered as configuration classes
- which situation should these configuration classes be applied

### Configuration classes
- Spring looks for `spring.factories` in the `META-INF` directory of all jars that the application depends on
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