# Java container configuration

## Basic : `@Configuration` with `@Bean`
- Bean method : The `@Bean` annotation is used to indicate that a method instantiates, configures, and initializes a new object to be managed by the Spring IoC container. 
- XML : `@Bean` is much similar to `<bean>` config in XML
- NOT final : Classes annotated with `@Configuration` use CGLIB to create a proxy for `@Configuration` class. CGLIB creates subclass for each class that is supposed to be proxied (hence cannot be `final`). Similarly, methods cannot be final.
- IF final : If methods or class is final then `BeanDefinitionParsingException` is thrown
- Identify class : class name would have  `$$EnhancerBySpringCGLIB`
> NOTE :  As a consequence, `@Configuration` classes and their factory methods ***both*** must not be marked as `final` or `private` in this mode

## Full `@Configuration` vs "lite" `@Bean` mode)
- When `@Bean` ***methods*** are declared within classes that are not annotated with `@Configuration`, they are referred to as being processed in a "lite" mode. Bean methods declared in a `@Component` or even in a plain old class are considered to be "lite"
- Example : service components may expose management views to the container through an additional `@Bean` method on each applicable component class.
- More here : https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-stereotype-annotations

## Instantiating Spring Container : `AnnotationConfigApplicationContext`
- Can pass `@Configuration` or `@Component` annotated classes as constructor parameters
 ```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

### Add classpath Scanning
- Classpath scanning can be added with `@ComponentScan` or programmatically using the `scan` method
- Since `@Configuration` is meta-annotated with `@Component`, the `@Configuration` classes will be scanned & need not be passed as params explicitly

```java
@Configuration 
@ComponentScan(basePackages = "com.acme")  
public class AppConfig { ... }
```
```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(); // NO CONFIGURATION NEEDED AS ITS SCANNED
    ctx.scan("com.acme");   // SCAN PACKAGE
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
}
```

### Web Support in Context
- web variant is `AnnotationConfigWebApplicationContext`
- web.xml config if required : https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-instantiating-container-web
- 