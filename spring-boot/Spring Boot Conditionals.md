### Overview
- Spring’s `@Conditional` annotation allows us to define conditions under which a certain bean is included into that object graph.

### Examples
- Some beans work in test context only
- Another use case is to enable or disable certain cross cutting concerns

## Declaring Conditional Beans
### Conditional @Bean

```java
@Configuration
class ConditionalBeanConfiguration {

  @Bean
  @Conditional... // <--
  ConditionalBean conditionalBean(){
    return new ConditionalBean();
  };
}
```

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

## Pre-defined Conditions
### @ConditionalOnProperty
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
- It allows to load beans conditionally depending on a certain environment property
- The `CrossCuttingConcernModule` is only loaded if the `module.enabled` property has the value `true`. If the property is not set at all, it will still be loaded, because we have defined `matchIfMissing` as `true`.

### @ConditionalOnExpression
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

### @CondionalOnBean
- load a bean only if a certain other bean is available in the application context
```java
@Configuration
@ConditionalOnBean(OtherModule.class)
class DependantModule {
  ...
}
```

### @ConditionalOnMissingBean
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