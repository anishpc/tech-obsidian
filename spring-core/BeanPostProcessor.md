BeanPostProcessor
------------------------------

-   Annotation injection is performed before XML injection. Thus, the XML configuration overrides the annotations for properties wired through both approaches.
-   Beans can be registered individually with bean definitions but they can also be implicitly registered by including the `annotation-config` tag

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans ..>

    <context:annotation-config/>
</beans>
```

-   The implicitly registered post-processors include `AutowiredAnnotationBeanPostProcessor`, `CommonAnnotationBeanPostProcessor`, `PersistenceAnnotationBeanPostProcessor`, and the aforementioned `RequiredAnnotationBeanPostProcessor`
-   `<context:annotation-config/>` only looks for annotations on beans in the same application context in which it is defined. This means that, if it's in a `WebApplicationContext` for a `DispatcherServlet`, it only checks for `@Autowired` beans in your controllers, and not your services.
-   NOTE : The use of `<context:component-scan>` implicitly enables the functionality of `<context:annotation-config>` There is usually no need to include the `<context:annotation-config>` element when using `<context:component-scan>`.
-   Component scan in annotation

```java
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    // ...
}
```





### Customizing Beans with BeanPostProcessor

-   The `BeanPostProcessor` interface defines callback methods that you can implement to provide your own (or override the container’s default) instantiation logic, dependency resolution logic, and so forth.
    -   To change the actual "bean definition", use `BeanFactoryPostProcessor`
-   If you want to implement some custom logic after the Spring container finishes instantiating, configuring, and initializing a bean, you can plug in one or more custom `BeanPostProcessor` implementations.
-   Can control order by setting the `order` property. Can set this property only if the `BeanPostProcessor` implements the `Ordered` interface
-   Container-hierarchies :
    -   `BeanPostProcessor` instances are scoped per-container.
    -   In other words, beans that are defined in one container are not post-processed by a `BeanPostProcessor` defined in another container, even if both containers are part of the same hierarchy. You can copy & register programmatically across hierarchies
-   The post-processor can take any action with the bean instance, including ignoring the callback completely
-   A bean post-processor typically checks for callback interfaces, or it may wrap a bean with a proxy, or do AOP
-   Registration :
    -   An `ApplicationContext` automatically detects any beans that are defined in the configuration metadata that implements the `BeanPostProcessor` interface and registers them (recommended)
    -   Register programmatically against a `ConfigurableBeanFactory` by using the `addBeanPostProcessor` method.
        -   Useful to evaluate conditional logic before registration
        -   copying bean post processors across contexts in a hierarchy
        -   NOTE : programmatical registration don't respect the `Ordered` interface & is handled by order of registration
        -   NOTE : programmatically registered instances are always processed before those registered through auto-detection, regardless of any explicit ordering
-   AOP auto-proxying
    -   AOP auto-proxying is implemented as `BeanPostProcessor` so other `BeanPostProcessor` instances nor the beans referenced are eligible for auto-proxying and do not have aspects woven into them