## Managed Components

-   Beans can be defined using
    -   `@Configuration`
    -   `@Bean`
    -   `@Import`
    -   `@DependsOn`
    -   Component scanning

### Component Scanning

-   `@Component` is a generic stereotype for any Spring-managed component
-   `@Repository`, `@Service`, `@Controller` are specialisations of `@Component` for specific use-cases — in persistence, service & presentation layers respectively. These are defined in the `org.springframework.stereotype` package.
-   All these specializations are annotated with `@Component` annotation. So, effectively, we are just scanning `@Component` annotation.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
**@Component** 
public @interface Service {
    // ...
}
```

-   These stereotypes make ideal targets for pointcuts
-   Also, these specializations can carry additional semantics in future Spring releases
-   `@Repository` is already supported as a annotation-pointcut for automatic exception translation in the persistence layer. It uses the `PersistenceExceptionTranslationPostProcessor`. The postprocessor automatically looks for all exception translators (implementations of the `PersistenceExceptionTranslator` interface) and advises all beans marked with the `@Repository` annotations so that the discovered translators can intercept and apply the appropriate translations on the thrown exceptions
-   More : [](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/data-access.html#orm-exception-translation)[https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/data-access.html#orm-exception-translation](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/data-access.html#orm-exception-translation)

### Meta-annotations & Composed Annotations

-   **_Meta-annotations_**
    
    -   A meta-annotation is an annotation that can be applied to an other annotation. For example, the `@Service` annotation mentioned earlier is meta-annotated with `@Component`
    
    ```java
    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    **@Component** 
    public @interface Service {
        // ...
    }
    ```
    
-   **_Composed annotations_**
    
    -   A composed annotation is an annotation that is _meta-annotated_ with one or more annotations with the intent of combining the behavior associated with those meta-annotations into a single custom annotation
    -   For example, the `@RestController` annotation from Spring MVC is composed of `@Controller` and `@ResponseBody`.
    -   In addition, composed annotations can optionally redeclare attributes from meta-annotations to allow customization. This can be particularly useful when you want to only expose a subset of the meta-annotation’s attributes. For example, Spring’s `@SessionScope` annotation hardcodes the scope name to `session` but still allows customization of the `proxyMode`. The following listing shows the definition of the `SessionScope` annotation:
    
    ```java
    @Target({ElementType.TYPE, ElementType.METHOD})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Scope(WebApplicationContext.SCOPE_SESSION)
    public @interface SessionScope {
    
        /**
         * Alias for {@link Scope#proxyMode}.
         * <p>Defaults to {@link ScopedProxyMode#TARGET_CLASS}.
         */
        @AliasFor(annotation = Scope.class)
        ScopedProxyMode proxyMode() default ScopedProxyMode.TARGET_CLASS;
    
    }
    
    // USAGE1 ============
    @Service
    @SessionScope
    public class SessionScopedService {
        // ...
    }
    
    // USAGE2 ============
    @Service
    @SessionScope(**proxyMode** = ScopedProxyMode.INTERFACES)
    public class SessionScopedUserService implements UserService {
        // ...
    }
    ```
    
-   More details : [](https://github.com/spring-projects/spring-framework/wiki/Spring-Annotation-Programming-Model)[https://github.com/spring-projects/spring-framework/wiki/Spring-Annotation-Programming-Model](https://github.com/spring-projects/spring-framework/wiki/Spring-Annotation-Programming-Model)
    

### Component Scanning

-   Add `@ComponentScan` to the `@Configuration` class to scan; `basePackages` attribute is parent package for the classes. Alternatively, specifiy comma or semicolon or space separated list of parent packages

```java
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    // ...
}
```

-   Java-9 :
    -   On JDK 9’s module path (Jigsaw), Spring’s classpath scanning generally works as expected. However, make sure that your component classes are exported in your module-info descriptors. If you expect Spring to invoke non-public members of your classes, make sure that they are 'opened' (that is, that they use an opens declaration instead of an exports declaration in your module-info descriptor).
-   Furthermore, the `AutowiredAnnotationBeanPostProcessor` and `CommonAnnotationBeanPostProcessor` are both implicitly included when you use the component-scan element. That means that the two components are autodetected and wired together — all without any bean configuration metadata provided in XML.

### Component Scanning Filters

-   By default, classes annotated with `@Component`, `@Repository`, `@Service`, `@Controller`, `@Configuration`, or a custom annotation that itself is annotated with `@Component` are the only detected candidate components. However, you can modify and extend this behavior by applying custom filters.
-   Add them as `includeFilters` or `excludeFilters` attributes of the `@ComponentScan` annotation (or as `<context:include-filter />` or `<context:exclude-filter />` child elements of the `context:component-scan` element in XML configuration). Each filter element requires the `type` and `expression` attributes.

```java
@Configuration
@ComponentScan(basePackages = "org.example",
	includeFilters=@Filter(type = FilterType.REGEX, pattern = ".*Stub.*Repository"),
	excludeFilters=@Filter(Repository.class))
public class AppConfig {
    ...
}
```

Filter Type|Expression|Description
--------|-----|-----|
annotation(default)|`org.example.SomeAnnotation`|An annotation to be present or meta-present at the type level in target components.
assignable|`org.example.SomeClass`|A class (or interface) that the target components are assignable to (extend or implement).
aspectj|`org.example.*Service+`|An AspectJ type expression to be matched by the target components.
regex|`org\.example\.Default.*`|A regex expression to be matched by the target components' class names.
custom|`org.example.MyTypeFilter`|A custom implementation of the `org.springframework.core.type.TypeFilter` interface.

### Bean metadata within Components

-   Can do this with the same `@Bean` annotation used to define bean metadata within `@Configuration` annotated classes.

```java
@Component
public class FactoryMethodComponent {

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    public void doWork() {
        // Component method implementation omitted
    }
}
```

-   Difference with components in `@Bean` & `@Configuration` : the component classes are not enhanced with CGLIB to intercept the invocation of methods and fields. The components have standard Java semantics, with no special CGLIB processing or other constraints
-   CGLIB proxying is the means by which invoking methods or fields within `@Bean` methods in `@Configuration` classes creates bean metadata references to collaborating objects. Such methods are not invoked with normal Java semantics but rather go through the container in order to provide the usual lifecycle management and proxying of Spring beans, even when referring to other beans through programmatic calls to `@Bean` methods. In contrast, invoking a method or field in a `@Bean` method within a plain `@Component` class has standard Java semantics, with no special CGLIB processing or other constraints applying.

### Static `@Bean` definitions

-   `BeanFactoryPostProcessor` & `BeanPostProcessor` may be declared as `static` in the `@Configuration` classes, since such beans get initialized early in the container lifecycle and should avoid triggering other parts of the configuration at that point
-   CGLIB subclassing can override only non-static methods — hence, a direct call to another `@Bean` method has standard Java semantics, resulting in an independent instance being returned straight from the factory method itself.
-   Regular `@Bean` methods in `@Configuration` classes need to be overrideable & hence cannot be declared as `private` or `final`
-   Inheritance scanning : `@Bean` methods are discovered on
    -   base classes of a given component or configuration class
    -   Java 8 default methods in interfaces implemented by component or configuration class
-   Collision : Single class may hold multiple `@Bean` methods for the same bean. The variant with the largest number of satisfiable dependencies is picked at construction time

### Component Index

-   While classpath scanning is very fast, it is possible to improve the startup performance of large applications by creating a static list of candidates at compilation time. In this mode, all modules that are target of component scan must use this mechanism.
-   [](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-scanning-index)[https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-scanning-index](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-scanning-index)

### Component annotation equivalent
- Instead of `@Component`, can use `javax.inject.@Named` OR `javax.annotation.ManagedBean`
```java
@Named("movieListener")  // @ManagedBean("movieListener") could be used as well
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
    // ...
}
```
- Component scanning works with `@Named` & `@ManagedBean` as well

### Limitations of JSR-330 annotations
Spring annotation|JSR-330 annotation|Comments
----|---|---
`@Autowired`|`@Inject`|`@Inject` has no `required` attribute. Can use `Optional` instead
`@Component`|`@Named`,`@ManagedBean`|JSR-330 does not provide composable model, only a way to identify named components
`@Scope("singleton")`|`@Singleton`|JSR-330 default scope is "prototype". To keep consistency with Spring, JSR-330 bean is declared as singleton in Spring container by default. To change, use the Spring's `@Scope` annotation. The `@Scope` annotation provided by JSR-330 is only for creating custom annotations
`@Qualifier`|`@Qualifier`, `@Named`|`javax.inject.Qualifier` is a meta annotation for building custom qualifiers. Concrete qualifiers can be associated through `@Named`
`@Value`|-|no equivalent
`@Required`|-|no equivalent
`@Lazy`|-|no equivalent
ObjectFactory|Provider|`javax.inject.Provider` is a direct alternative to Spring’s `ObjectFactory`, only with a shorter `get()` method name. It can also be used in combination with Spring’s `@Autowired` or with non-annotated constructors and setter methods.
