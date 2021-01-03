Container
---------

### Container > BeanFactory

-   Spring's job is to parse configuration files and then instantiate your managed classes, resolving their interdependencies.
-   Spring is often called a **_container_**, since it is designed to create and manage all the dependencies within your application, serving as a foundation and a context through which beans may be resolved & injected
-   This core engine is represented by a base interface called `BeanFactory`
-   `BeanFactory` itself has sub-interfaces like `ApplicationContext` but also other bean factories like `ConfigurableBeanFactory`, `ConfigurableListableBeanFactory`

### Container > ApplicationContext

-   The `BeanFactory` interface provides a configuration mechanism capable of managing any type of object. The `ApplicationContext` is a sub-interface of `BeanFactory`. It adds :
    -   integration with AOP
    -   i8n
    -   event publication
    -   application-layer specific contexts such as `WebApplicationContext` for use in web applications
    -   load file resources using relative or absolute paths/URLs (`ResourceLoader` support)
-   The `ApplicationContext` is a complete superset of the `BeanFactory`
-   The `ApplicationContext` extends the `BeanFactory` interface, providing more robust features. The separation can come in handy if you are building a very lightweight application and you don’t need some of these more advanced features
-   Implementations of `ApplicationContext` manage a number of bean definitions uniquely identified by their name.

### Container > ApplicationContext Types

-   The container gets its instructions on what objects to instantiate, configure, and assemble by reading configuration metadata. The configuration metadata is represented in XML, Java annotations, or Java code
-   Non-web applications
    -   `AnnotationConfigApplicationContext`
    -   `ClassPathXmlApplicationContext`
    -   `FileSystemXmlApplicationContext`
-   Web applications
    -   Servlet 2 : web.xml, `ContextLoaderListener`, `DispatcherServlet`
    -   Servlet 3 : `XmlWebApplicationContext`
    -   Servlet 3 : `AnnotationConfigWebApplicationContext`
-   Spring Boot
    -   SpringBootConsoleApplication — `CommandLineRunner`
    -   SpringBootWebApplication — Embedded Tomcat

![container-magic.png](container-magic.png)

### Container > ApplicationContext config loading

-   An `ApplicationContext` is initialized with a configuration provided by a resource that can be an XML file (or more) or a configuration class (or more) or both.
-   When the resource is provided as a `String` value, the Spring container tries to load the resource based on the prefix of the string value
-   To provide functionality of loading resources, an application context must implement `org.springframework.core.io.ResourceLoader` interface
-   The resource-type depends on the context being used to load the resource; however if we want to fix the resource type no matter which app-context is used to load then use prefix
-   Prefix :
    -   no-prefix :
        -   location : In root directory where the class creating the context is executed
        -   notes : resource type will depend on the application-context used to load the resource
    -   `classpath:`
        -   location : The resource should be obtained from the class-path
        -   notes : in the `resources` directory; resource type `ClassPathResource`; `ClassPathXmlApplicationContext` class is suitable
    -   `file:`
        -   location : In the absolute location following the prefix
        -   notes : resource type `UrlResource`; `FileSystemXmlApplicationContext` is suitable
    -   `http:`
        -   location : in the web location following the prefix
        -   notes : resource type `UrlResource`; `WebApplicationContext` is suitable

### Container > WebApplicationContext bootstrap

-   Prior to Servlet 3.0, it was necessary to configure your web application's `web.xml` file to bootstrap `WebApplicationContext`. Since Servlet 3.0 (& Spring 3.1), it is possible to bootstrap a `WebApplicationContext` programmatically using `WebApplicationInitializer`

### Container > Injection

-   Spring supports types of dependency injection
    1.  constructor injection
    2.  setter injection
    3.  field injection : container injects the dependency directly as a value for the field (via reflection and this requires the `open` directive in the `[module-info.java](<http://module-info.java>)` file)
    4.  method parameters
-   Starting with Spring 4.3, using `@Autowired` if the class has a single constructor is **_no longer_** necessary
-   If the Spring IoC container cannot find a bean to inject into when creating a bean declared to use constructor injection, an exception of type `org.springframework.beans.factory.UnsatisfiedDependencyException` will be thrown and the application will fail to start.
-   The `@Autowired`, `@Inject`, `@Value`, and `@Resource` annotations are handled by Spring `BeanPostProcessor` implementations. This means that you cannot apply these annotations within your own `BeanPostProcessor` or `BeanFactoryPostProcessor` types (if any). These types must be 'wired up' explicitly by using XML or a Spring `@Bean` method.

### Container > Injection > Fine-tuning autowiring

1.  `@Primary`
    
    -   `@Primary` indicates that a particular bean should be given preference when multiple beans are candidates to be autowired to a single-valued dependency. If exactly one primary bean exists among the candidates, it becomes the autowired value.
        
        ```java
        @Configuration
        public class MovieConfiguration {
        
            @Bean
            @Primary
            public MovieCatalog firstMovieCatalog() { ... }
        
            @Bean
            public MovieCatalog secondMovieCatalog() { ... }
        }
        
        public class MovieRecommender {
        
            @Autowired
            private MovieCatalog movieCatalog;  // <-- firstMovieCatalog 
        }
        
        //in xml
        <bean class="example.SimpleMovieCatalog" primary="true">
        ```
        
        -   Self-reference : [](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-autowired-annotation-qualifiers)[https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-autowired-annotation-qualifiers](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-autowired-annotation-qualifiers)
2.  `@Qualifier`
    
    -   `@Primary` is an effective way to use autowiring by type with several instances when one primary candidate can be determined. When you need more control over the selection process, you can use Spring’s `@Qualifier` annotation
    -   Creating custom qualifier
    
    ```java
    @Target({ElementType.FIELD, ElementType.PARAMETER})
    @Retention(RetentionPolicy.RUNTIME)
    @Qualifier
    public @interface Genre {
    
        String value();
    }
    
    		//Usage
    		@Autowired
        @Genre("Action")
        private MovieCatalog actionCatalog;
    ```
    
3.  Generics as Autowiring Qualifiers
    
    -   Java generic types can be used as an implicit form of qualification
    
    ```java
    @Configuration
    public class MyConfiguration {
    
        @Bean
        public StringStore stringStore() {
            return new StringStore();
        }
    
        @Bean
        public IntegerStore integerStore() {
            return new IntegerStore();
        }
    }
    // Class declaration
    public class IntegerStore implements Store<Integer> {}
    public class StringStore implements Store<String> {}
    
    //Injection
    @Autowired
    private Store<String> s1; // <String> qualifier, injects the stringStore bean
    
    @Autowired
    private Store<Integer> s2; // <Integer> qualifier, injects the integerStore bean
    ```
    

### Injection Annotations
Type|@Resource|@Inject|@Autowired
-----|------------|--------|-----------
Source|part of JSR-250 packaged with Jakarta EE|@Inject is from JSR-330 from javax.inject.| part of Spring framework
Usage places| Can use `@Resource` on : - field  setters; not on constructors|`@Inject` can be used on field, setter, constructor|`@Autowired` can be used on field, setter, constructor, method parameters
Paramters| The @Resource annotation takes 2 important optional parameter name and type. \* If no name is explicitly specified, the default name is derived from the field name or setter method. \* Similarly, if no type is specified explicitly, it will do a type match and try to resolve it.|Does not take any parameters|Takes one parameter required (default value is true)
Precedence| 1. By Name 2. By Type 3. By Qualifier| 1. By Type 2. By Qualifier(`@Qualifier`) 3.By Name(`@Named`)| 1. By Name, 2. By Qualifier, 3. By Name

### Container > Injection > `@Resource` (JSR-250)

```java
//classses : FileReader(abstract) <-- PdfFileReader
//classses : FileReader(abstract) <-- WordFileReader

public class ResourceTest {
	//Will throw exception due to ambiguity
	@Resource
	private FileReader fReader;

  //Resolve by property name (implicit)
	@Resource
	private FileReader pdfFileReader;  //PdfFileReader class would be 

	//Resolve by explicit name
  @Resource(name = "wordFileReader")
  private FileReader reader;

	//Resolve by type auto detection
  @Resource
  private WordFileReader fileReader;

	//Resolve by explicit Type
  @Resource(type = PdfFileReader.class)
  private FileReader fileReader2;

	//Resolve by Qualifier
  @Qualifier("pdfFileReader")
  @Resource
  private FileReader myFileReader;

}
```

### Container > Injection > `@Inject` (JSR-330)

```java
public class ResourceTest {
	//Inject by Type
  @Inject
  private PdfFileReader fileReader;

	//Inject by @Qualifier
  @Inject
  @Qualifier("wordFileReader")
  private FileReader fileReader3;

	//Inject by name with @Named annotation
  @Inject
  @Named("pdfFileReader")
  private FileReader fileReader2;
}
```

### Container > Injection > `@Autowired` (Spring)

```java
public class ResourceTest {
	//Inject by Type
  @Autowired
  private PdfFileReader pdfFileReader;

	//Inject by @Qualifier
  @Autowired
  @Qualifier("wordFileReader")
  private FileReader fileReader;

	//Inject by Field name
  @Autowired
  private FileReader wordFileReader;

	@Autowired
  private MovieCatalog[] movieCatalogs;

  // Standard Spring objects
	@Autowired
  private ApplicationContext context;

	@Autowired
  public void setMovieCatalogs(Set<MovieCatalog> movieCatalogs) {
		this.movieCatalogs = movieCatalogs;
	}

	@Autowired
  public void setMovieCatalogs(Map<String, MovieCatalog> movieCatalogs) {
		this.movieCatalogs = movieCatalogs;
	}

	// Not required -- fine to skip
	@Autowired(required = false)
  public void setMovieFinder(MovieFinder movieFinder) {
	  this.movieFinder = movieFinder;
  }

	// Not required -- fine to skip
	@Autowired
	public void setMovieFinder(Optional<MovieFinder> movieFinder) {
	}

	// Not required -- fine to skip
	@Autowired
	public void setMovieFinder(@Nullable MovieFinder movieFinder) {
  }
}
```

-   **_Set, List, Arrays :_** For the case of `MovieCatalog[]` array, the bean `MovieCatalog` can implement `Ordered` interface or use `@Order`or standard `@Priority` annotation if the items have to be in a specific order. Otherwise, registration order is used
    -   NOTE : `@Order` doesn't influence startup order — that is managed using `@DependsOn`
    -   NOTE : `javax.annotation.Priority` annotation is not available at the `@Bean` level, since it cannot be declared on methods. Its semantics can be modeled through `@Order` values in combination with `@Primary` on a single bean for each type.
-   **_Map :_** Even typed `Map` instances can be autowired as long as the expected key type is `String`.
-   Not-required :
    -   `@Autowired(required = false)` can be used to indicate that its fine to skip a non-satisfiable injection
    -   `@Autowired` with field as `Optional<>` would be considered fine to skip
    -   As of Spring Framework 5.0, you can also use a `@Nullable` annotation (of any kind in any package — for example, `javax.annotation.Nullable` from JSR-305) or just leverage Kotlin builtin null-safety support with `@Autowired`
-   Standard objects :
    -   You can also use `@Autowired` for interfaces that are well-known resolvable dependencies: `BeanFactory,` `ApplicationContext`, `Environment`, `ResourceLoader`, `ApplicationEventPublisher`, and `MessageSource`. These interfaces and their extended interfaces, such as `ConfigurableApplicationContext` or `ResourcePatternResolver`, are automatically resolved, with no special setup necessary.

### Container > Injection > Other Annotations

-   `@Required`
    -   Can be applied only on setter methods & it marks that dependency as mandatory
    -   Since Spring 5.1, this has been deprecated in favour of using constructor injection for required settings

### Container > Misc

-   `@Description` annotation adds a description to a bean, which is quite useful when beans are exposed for monitoring puposes. It can be used with `@Bean` and `@Component` (and its specializations)

```java
@Description("Salary for an employee might change,
      so this is a suitable example for a prototype scoped bean")
@Component
public class Salary {
...
}
OR
public class SimpleDependentCfg {
    @Description("This bean depends on 'simpleBean'")
    @Bean
    DependentBean dependentBean(){
      return new DependentBeanImpl(simpleBean());
    }
    ...
}
```

