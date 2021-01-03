### Customizing Config metadata with BeanFactoryPostProcessor

-   Spring container lets a `BeanFactoryPostProcessor` read the configuration metadata and potentially change it before the container instantiates any beans other than `BeanFactoryPostProcessor` instances.
-   Can implement multiple `BeanFactoryPostProcessor` and order is via `order` property of `Ordered` interface
-   Bean instance note : While it is technically possible to work with bean instances within a `BeanFactoryPostProcessor` (for example, by using `BeanFactory.getBean()`), doing so causes premature bean instantiation, violating the standard container lifecycle. This may cause negative side effects, such as bypassing bean post processing.
-   Container-hierarchies:
    -   `BeanFactoryPostProcessor` instances are scoped per-container.
    -   If you define a `BeanFactoryPostProcessor` in one container, it is applied only to the bean definitions in that container.
    -   Bean definitions in one container are not post-processed by `BeanFactoryPostProcessor` instances in another container, even if both containers are part of the same hierarchy.
-   Spring includes a number of predefined bean factory post-processors, such as `PropertyOverrideConfigurer` and `PropertySourcesPlaceholderConfigurer`. `PropertySourcesPlaceholderConfigurer` file replaces the property values with the actual values from the property files.
    -   The `PropertySourcesPlaceholderConfigurer` not only looks for properties in the `Properties` file you specify. By default, if it cannot find a property in the specified properties files, it checks against Spring `Environment` properties and regular Java `System` properties.