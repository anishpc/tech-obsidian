## Declarative transaction implementation
- Declarative transaction support is enabled via AOP proxies and is driven by metadata (XML- or Annotation-based)
- Combination of AOP & transactional metadata yields an AOP proxy that uses a `TransactionInterceptor` in conjunction with an appropriate `TransactionManager` implementation
- Reactive/Imperative : 
	- Spring Framework’s `TransactionInterceptor` provides transaction management for imperative and reactive programming models.
	- Model is chosen based on return type :
		- Reactive : `Publisher` or Kotlin `Flow` 
		- Imperative : All other return types (including `void`)
	- Imperative : `PlatformTransactionManager`
	- Reactive : `ReactiveTransactionManager`

### Threading Model
- `@Transactional` commonly works with thread-bound transactions managed by `PlatformTransactionManager`, exposing a transaction to all data access operations within the current execution thread. 
> Note: This does ***not*** propagate to newly started threads within the method.
- A reactive transaction managed by `ReactiveTransactionManager` uses the Reactor context instead of thread-local attributes. As a consequence, all participating data access operations need to execute within the same Reactor context in the same reactive pipeline.

### Transactional proxy
![tx](https://docs.spring.io/spring-framework/docs/current/reference/html/images/tx.png)

### Using `@Transactional`
> The standard `javax.transaction.Transactional` annotation is also supported as a drop-in replacement to Spring’s own annotation.
- Method OR class-level : 
	- Class-level : 
		- applies to all methods of the class as well as its ***subclasses***
		- does not apply to super class though
	- Method level : each individual method
- Enabling : Use `@EnableTransactionManagement` annotation in a `@Configuration` class

#### Method visibility & ``@Transactional`
- When you use proxies, you should apply the `@Transactional` annotation only to methods with public visibility. 
- If you do annotate protected, private or package-visible methods with the `@Transactional` annotation, no error is raised, but the annotated method does not exhibit the configured transactional settings. 
- If you need to annotate non-public methods, consider using AspectJ

#### Annotation on Interface/Class
- The Spring team recommends that you annotate only concrete classes (and methods of concrete classes) with the `@Transactional` annotation, as opposed to annotating interfaces. 
- You certainly can place the `@Transactional` annotation on an interface (or an interface method), but this works only as you would expect it to if you use ***interface-based proxies***. 
- The fact that Java annotations are not inherited from interfaces means that, if you use class-based proxies (`proxy-target-class="true"` or `proxyTargetClass`) or the weaving-based aspect (`mode="aspectj"`), the transaction settings are ***not*** recognized by the proxying and weaving infrastructure, and the object is ***not*** wrapped in a transactional proxy.

#### Self-invocation
- In proxy mode (which is the default), only external method calls coming in through the proxy are intercepted. This means that self-invocation (in effect, a method within the target object calling another method of the target object) does not lead to an actual transaction at runtime even if the invoked method is marked with `@Transactional`.
- Use AspectJ if this is required because the target class is woven (that is, its byte code is modified) to turn `@Transactional` into runtime behavior on any kind of method.

#### Proper Initialization
- The proxy must be fully initialized to provide the expected behavior, so you should not rely on this feature in your initialization code (that is, `@PostConstruct`).

### Settings for `@Transactional`

| Attribute     | Default | Description |
| ----------- | ----------- |----------|
| transactionManager      | transactionManager| - Name of the transaction manager to use. <br/>- Required only if the name of the transaction manager is not `transactionManager`       |
| mode   | proxy        | Other mode is `aspectj`
| proxyTargerClass| false|- Applies to proxy mode only. If true then class-based proxies. <br/>- If false or omitted then standard JDK interface-based proxies
|order|Ordered.LOWEST_PRECEDENCE| Ordering of AOP advice
|propagation|REQUIRED|Propagation level
|isolation|DEFAULT|Applies only to `REQUIRED` or `REQUIRES_NEW`.
|readOnly|false|read-write or read-only (Only applicable to `REQUIRED` or `REQUIRES_NEW`)
|timeout(secs)|-1|Applies only to `REQUIRED` or `REQUIRES_NEW`.
|rollbackFor(`Class<? extends Throwable>[]`)|{}|array of exception classes that must cause rollback
|rollbackForClassName (`String[]`)|{}|array of ***names*** of exception classes that must cause rollback
|noRollbackFor (`Class<? extends Throwable>[]`)|{}|array of exception classes that must not cause rollback
|noRollbackForClassName(`String[]`)|{}|array of ***names*** of exception classes that must not cause rollback
|label(`String[]`)since Spring 5.3|{}|Labels may be used to describe a transaction and they can be evaluated by individual transaction manager

### Multiple Transaction managers
[Data Access (spring.io)](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#tx-multiple-tx-mgrs-with-attransactional)

## Programmatic Transaction Management
Two ways : 
1. `TransactionTemplate` or `TransactionalOperator`
2. Directly using `TransactionManager`
> RECOMMENDATION :
>  `TransactionTemplate` for Imperative flows
>  `TransactionalOperator` for Reactive flows

More info at : [Data Access (spring.io)](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-programmatic)

## Programmatic vs Declarative
- Number of transaction operations : 
	- Small Number of transaction operations  : if there are only few operations then instead of setting up transactional proxies in Spring or something else, we can use `TransactionTemplate`
	- Large number of transaction operations : if your application has numerous transactional operations, declarative transaction management is usually worthwhile. It keeps transaction management out of business logic and is not difficult to configure.
- Set transaction name explicitly : 
	- Being able to set the transaction name explicitly is also something that can be done only by using the programmatic approach to transaction management.

## Transaction bound Events
- As of Spring 4.2, the listener of an event can be bound to a phase of the transaction
- Bind to transaction : 
	- You can register a regular event listener by using the `@EventListener` annotation. 
	- If you need to bind it to the transaction, use `@TransactionalEventListener`. When you do so, the listener is bound to the ***commit*** phase of the transaction by default.
- Example : handle event once transaction has committed successfully
```java
@Component 
public class MyComponent { 
	@TransactionalEventListener 
	public void handleOrderCreatedEvent(CreationEvent<Order> creationEvent) { 				// ... 
	} 
}
```
- Bind phase : 
	- The `@TransactionalEventListener` annotation exposes a `phase` attribute that lets you customize the phase of the transaction to which the listener should be bound. 
	- The valid phases are `BEFORE_COMMIT`, `AFTER_COMMIT` (default), `AFTER_ROLLBACK`, as well as `AFTER_COMPLETION` which aggregates the transaction completion (be it a commit or a rollback).
- No transaction?
	- If no transaction is running, the listener is not invoked at all, since we cannot honor the required semantics. You can, however, override that behavior by setting the `fallbackExecution` attribute of the annotation to `true`.
> `@TransactionalEventListener` only works with thread-bound transactions managed by `PlatformTransactionManager`. A reactive transaction managed by `ReactiveTransactionManager` uses the Reactor context instead of thread-local attributes, so from the perspective of an event listener, there is no compatible active transaction that it can participate in.