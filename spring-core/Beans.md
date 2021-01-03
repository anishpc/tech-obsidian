Bean Definition
-----------------------

-   Within the container, bean definitions are represented as `BeanDefinition` objects, which contain (among other information) the following metadata :
    -   A package-qualified class name: typically, the actual implementation class of the bean being defined.
    -   Bean behavioral configuration elements, which state how the bean should behave in the container (scope, lifecycle callbacks, and so forth).
    -   References to other beans that are needed for the bean to do its work. These references are also called collaborators or dependencies.
    -   Other configuration settings to set in the newly created object — for example, the size limit of the pool or the number of connections to use in a bean that manages a connection pool.
-   Register beans :
    -   Can register beans defined outside through the `BeanFactory`.
    -   This is done by accessing the ApplicationContext’s `BeanFactory` through the `getBeanFactory()` method, which returns the BeanFactory `DefaultListableBeanFactory` implementation. `DefaultListableBeanFactory` supports this registration through the `registerSingleton(..)` and `registerBeanDefinition(..)` methods. However, typical applications work solely with beans defined through regular bean definition metadata.
-   Runtime registration :
    -   Bean metadata and manually supplied singleton instances need to be registered as early as possible, in order for the container to properly reason about them during autowiring and other introspection steps. While overriding existing metadata and existing singleton instances is supported to some degree, the registration of new beans at runtime (concurrently with live access to the factory) is not officially supported and may lead to concurrent access exceptions, inconsistent state in the bean container, or both.

Instantiating Beans
---------------------------

Bean Naming
-------------------

-   Spring bean naming :
    
    -   class name with first letter lower-cased
    -   bean method name when annotated with `@Bean`
    -   specify name with `@Component("myNewName")`
-   Spring bean name aliases :
    
    -   Declaring aliases using stereotype annotations is not supported at the moment, but it is possible using the `@Bean` annotation
    -   The caveat of declaring aliases using the `@Bean` annotation is that the method name will **no** longer be used as a bean name, only the names declared as values for the `name` attribute in the annotation will be used.
    
    ```java
    @Configuration
    public class AliasesCfg {
        @Bean(name= {"beanOne", "beanTwo"})
        SimpleBean simpleBean(){
            return new SimpleBeanImpl();
        }
    }
    // there is no bean with name `simpleBean`
    ```
    
-   Rule of thumb :
    
    -   Consider specifying the name with the annotation whenever other components may be making explicit references to it.
    -   Auto-generated names are adequate whenever the container is responsible for wiring.
-   Custom name :
    
    -   Create custom class which implements `BeanNameGenerator` and include a no-arg constructor
    -   Configure the custom as follows:
    
    ```java
    @Configuration
    @ComponentScan(basePackages = "org.example", nameGenerator = MyNameGenerator.class)
    public class AppConfig {
        // ...
    }
	
    === XML
    <beans>
        <context:component-scan base-package="org.example"
            name-generator="org.example.MyNameGenerator" />
    </beans>
    ```
    
-   Conflicts :
    
    -   Conflicts due to multiple auto-detected components having the same non-qualified class name (i.e. classes with identical names but residing in different packages) then as of Spring 5.2.3, can configure `FullyQualifiedAnnotationBeanNameGenerator` provided by Spring

Bean Scopes
-------------------

-   `@Scope` annotations are only introspected on the concrete bean class (for annotated components) or the factory method (for `@Bean` methods).
    
-   Singleton :
    
    -   a single bean managed by Spring container
    -   suited for stateless beans
-   Prototype :
    
    -   new bean instance on every new request
    -   suited for stateful beans
    -   initialization callback methods are called from Spring; however, destruction lifecycle callbacks are not called;
        -   to do clean up during destruction, use a custom bean post-processor which holds a reference to beans that need to be cleaned up
-   Request, Session, Application, WebSocket
    
    -   only valid in context of a web-aware Spring `ApplicationContext`
-   Custom scope
    
    -   Write a custom `Scope` and register with Spring
    -   Registration is via `void registerScope(String scopeName, Scope scope);` method which is defined on `ConfigurableBeanFactory` interface which is available via the `BeanFactory` property on most concrete `ApplicationContext` implementations.
    
    ```java
    Scope threadScope = new SimpleThreadScope();
    beanFactory.registerScope("thread", threadScope);
    ```
    
    -   Can also register declaratively : [](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-custom-using)[https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-custom-using](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-custom-using)
-   Scopes code
    

```java
//inside @Configuration class
@Bean
@Scope("prototype")
public Item item() {
  return new Item();
}

@Scope("singleton")
@Component
public class ItemDao {
  public void doSomething(){ //... }
}

// Web-aware : 
@RequestScope
@SessionScope
@ApplicationScope

@Scope("websocket")
@Component
public class UpdateNotifier {
 //do tasks
}

```

### Bean Scopes > Scoped bean dependencies

-   **_AOP Proxy_** : If you want to inject a short-lived scope into another bean of a longer-lived scope, you may choose to inject an AOP proxy in place of the scoped bean. This proxy needs to expose the same interface as the scoped object but can also retrieve the real target object from relevant scope & delegate calls to the real object
    -   Singleton example : Using `<aop:scoped-proxy/>` on `singleton` scope -> the reference then goes through an intermediate proxy that is serializable and therefore able to re-obtain the target singleton bean on deserialization
    -   Prototype example : Using `<aop:scoped-proxy/>` on `prototype` scope -> every method call on the shared proxy leads to the creation of a new target instance to which the call is then being forwarded
-   **_ObjectFactory_** : You may also declare your injection point (that is, the constructor or setter argument or autowired field) as `ObjectFactory<MyTargetBean>`, allowing for a `getObject()` call to retrieve the current instance on demand every time it is needed — without holding on to the instance or storing it separately.
    -   As an extended variant, you may declare `ObjectProvider<MyTargetBean>`, which delivers several additional access variants, including `getIfAvailable` and `getIfUnique`
-   **_JSR-330 Provider_** : The JSR-330 variant of this is called `Provider` and is used with a `Provider<MyTargetBean>` declaration and a corresponding `get()` call for every retrieval attempt.

Customize Bean
---------------------

### Lifecyle

![bean-lifecycle.png](bean-lifecycle.png)

### Phases

-   Initialization phase is divided into two steps
    1.  Load bean definitions
    2.  Initialize bean instances

![bean-lifecycle.png](bean-lifecycle.png)

### Load Bean definitions

-   All class files with `@Configuration` and XML files are processed
-   Component scanning is done for `@Component` and subtype annotations
-   Bean definitions are added to `BeanFactory`
-   Each bean is indexed by its `id`
-   Spring runs it's `BeanFactoryPostProcessor` beans & `BeanFactoryPostProcessor` can modify the definition of any bean before any objects are created
    -   examples: reading properties from property file, registering custom scope
-   Beans from POJOs can be configured in one of three ways: XML, methods annotated with `@Bean` in a configuration Java class annotated with `@Configuration`, or when using component scanning, you can add an annotation such as `@Component` or `@Service` on the POJO class itself.
    -   The most recommended way is using one or more Java configuration classes for infrastructure and component scanning for business classes.

### Instantiate Bean instances

![bean-lifecycle-phases.png](bean-lifecycle-phases.png)

-   To interact with the bean's lifecycle, there are options:
    1.  JSR-250 `@PostConstruct` & `@PreDestroy` anntations (RECOMMENDED way)
    2.  Lifecycle callbacks in `<bean/>` or `@Bean` declarations
    3.  `InitializingBean` & `DisposableBean` interfaces
    4.  Using default initialization & destroy methods
-   Internally Spring uses `BeanPostProcessor` implementations to process any callback interface it can find and call the appropriate methods. You can implement a `BeanPostProcessor` for any feature Spring does not provide
-   Order of above options for initialization
    1.  Methods annotated with `@PostConstruct`
    2.  `afterPropertiesSet()` method from `InitializingBean` callback interface (couples code with Spring framework)
    3.  custom configured init method (`init-method` in XML and `@Bean(initMethod = "initMethod")`)
-   Order of above options for shutdown
    1.  Methods annotated with `@PreDestroy`
    2.  `destroy()` method from `DisposableBean` callback interface (couples code with Spring framework)
    3.  custom configured method

#### JSR-250

-   These annotations are managed in Spring by `CommonAnnotationBeanPostProcessor`

```java
@Component
public class JsrExampleBean {
  @PostConstruct
  public void initCacheData() {
    System.out.println("Your custom initialization code goes here...");
  }
  @PreDestroy
  public void clearCacheData() {
    System.out.println("Your custom destroy code goes here...");
  }
}
```

-   If it's a non-web spring application then the application context should be shutdown using `ConfigurableApplicationContext.registerShutdownHook();`

#### Init-Destroy config

-   Spring uses `BeanPostProcessor` to process callback interfaces and appropriate methods for this config as well

```java
@Configuration
@ComponentScan("bean.lifecycle.callbacks")
public class BLCConfigClass {

  @Bean(initMethod = "init", destroyMethod = "destroy")
  public ExampleBean1 exampleBean1() {
    return new ExampleBean1();
  }
}

//XML option
<bean id="exampleBean" class="bean.lifecycle.callbacks.ExampleBean1" 
init-method="init" destroy-method="destroy"/>
```

#### Initializing-Disposable

-   The bean has to implement the interfaces `InitializingBean` and `DisposableBean`
-   This creates tight coupling & binds the beans to the Spring framework interfaces as opposed to other methods above

```java
@Component
public class ExampleBean2 implements InitializingBean, DisposableBean {
  @Override
  public void afterPropertiesSet() throws Exception {
    System.out.println("Initialization method of " + this.getClass().getName());
  }
  @Override
  public void destroy() throws Exception {
    System.out.println("Destroy method of " + this.getClass().getName());
  }
}
```

#### Global init-destroy

-   This can be configured once and would apply to all beans
-   The presence of the `default-init-method` attribute on the top-level `<beans/>` element attribute causes the Spring IoC container to recognize a method called `init` on the bean class as the initialization method callback. When a bean is created and assembled, if the bean class has such a method, it is invoked at the appropriate time.
-   Where existing bean classes already have callback methods that are named at variance with the convention, you can override the default by specifying (in XML, that is) the method name by using the `init-method` and `destroy-method`attributes of the `<bean/>` itself.

```xml
<beans default-init-method="init" default-destroy-method="destroy">

    <bean id="blogService" class="com.something.DefaultBlogService">
        <property name="blogDao" ref="blogDao" />
    </bean>

</beans>
```

-   Each bean is instantiated eagerly by default (for lazy needs to marked as lazy). Beans are created in the order based on dependencies
-   `BeanPostProcessor` can change the instance of the bean. They work on bean instances after instantiation of the bean by the container
-   `BeanPostProcessor` Code :

```java
public interface BeanPostProcessor { 
  Object postProcessBeforeInitialization(Object bean, String beanName) 
throws BeansException; 
  Object postProcessAfterInitialization(Object bean, String beanName) 
throws BeansException; 
}
```

### Shutdown

![bean-shutdown.png](bean-shutdown.png)

### Startup & Shutdown Callbacks

-   The `Lifecycle` interface defines the essential methods for any object that has its own lifecycle requirements like starting and stopping background processes

```java
public interface Lifecycle {

    void start();
    void stop();
    boolean isRunning();
}
```

-   Any Spring-managed object may implement the `Lifecycle` interface. Then, when the `ApplicationContext` itself receives start and stop signals (for example, for a stop/restart scenario at runtime), it cascades those calls to all `Lifecycle` implementations defined within that context. It does this by delegating to a `LifecycleProcessor`

```java
public interface LifecycleProcessor extends Lifecycle {

    void onRefresh();
    void onClose();
}
```

-   `SmartLifecycle` : `Lifecycle` interface is a plain contract for explicit start and stop notifications and does not imply auto-startup at context refresh time. For fine-grained control over auto-startup of a specific bean (including startup phases), consider implementing `SmartLifecycle`
-   Also, please note that stop notifications are not guaranteed to come before destruction. On regular shutdown, all `Lifecycle` beans first receive a stop notification before the general destruction callbacks are being propagated. However, on hot refresh during a context’s lifetime or on stopped refresh attempts, only destroy methods are called.
-   More details : [](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-processor)[https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-processor](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-processor)

### -Aware interfaces

-   When an `ApplicationContext` creates an object instance that implements the `org.springframework.context.ApplicationContextAware` interface, the instance is provided with a reference to that `ApplicationContext`
    
    -   Beans can programmatically manipulate the `ApplicationContext` or cast to known subclass (such as `ConfigurableApplicationContext`, which exposes additional functionality)
    -   General advice to avoid it as it leads to tight couping with Spring and does not follow IoC
    -   Additional capabilities of `ApplicationContext` : [](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#context-introduction)[https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#context-introduction](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#context-introduction)
    
    ```java
    public interface ApplicationContextAware {
        void setApplicationContext(ApplicationContext applicationContext) 
    throws BeansException;
    }
    ```
    
-   Autowiring is another alternative to obtain a reference to the `ApplicationContext`
    
-   When an `ApplicationContext` creates a class that implements the `org.springframework.beans.factory.BeanNameAware` interface, the class is provided with a reference to the name defined in its associated object definition.
    
    ```java
    public interface BeanNameAware {
        void setBeanName(String name) throws BeansException;
    }
    ```
    
-   Order : The callback is invoked after population of normal bean properties but before an initialization callback such as `InitializingBean`, `afterPropertiesSet`, or a custom init-method.
    
-   Other `Aware` interfaces : [](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aware-list)[https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aware-list](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aware-list)
    

Bean Definition Inheritance
---------------------------

-   [](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-child-bean-definitions)[https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-child-bean-definitions](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-child-bean-definitions)
-   In contrast to XML bean definitions, there is no notion of bean definition inheritance, and inheritance hierarchies at the class level are irrelevant for metadata purposes.