## Global Transactions
- Global Transactions let you work with multiple transactional resources, typically relational databases and message queues
- The application server manages global transactions through the JTA
- Furthermore, a JTA `UserTransaction` needs to be sourced from JNDI, meaning that you also need to use JNDI in order to use JTA
- Previously, the preferred way to use global transactions was through EJB CMT (Container Managed Transaction)

## Local Transactions
- Local transactions are resource-specific, such as a transaction associated with a JDBC connection. 
- They cannot work across multiple transactional resources. For example, code that manages transactions by using a JDBC connection cannot run within a global JTA transaction

## Transaction Manager
- The key to the Spring transaction abstraction is the notion of a transaction strategy
- `TransactionManager` implementations normally require knowledge of the environment in which they work: JDBC, JTA, Hibernate, and so on
- A transaction strategy is defined by a `TransactionManager`, specifically the `org.springframework.transaction.PlatformTransactionManager` interface for imperative transaction management and the `org.springframework.transaction.ReactiveTransactionManager` interface for reactive transaction management
```java
public interface PlatformTransactionManager extends TransactionManager { 
	TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException; 
	void commit(TransactionStatus status) throws TransactionException; 
	void rollback(TransactionStatus status) throws TransactionException; 
}
```
- `TransactionException` is unchecked
- The `TransactionDefinition` interface specifies : 
	- Propagation
	- Isolation
	- Timeout
	- Read-only status
- The `TransactionStatus` interface provides a simple way for transactional code to control transaction execution and to query transaction status
```java
public interface TransactionStatus extends TransactionExecution, SavepointManager, Flushable { 
	@Override 
	boolean isNewTransaction(); 
	
	boolean hasSavepoint(); 
	
	@Override 
	void setRollbackOnly(); 
	
	@Override 
	boolean isRollbackOnly(); 
	
	void flush(); 
	
	@Override 
	boolean isCompleted(); 
}
```

### Transaction Manager implementations
- For a JDBC `DataSource`
- The transaction maanger needs a reference to `DataSource`
```xml
<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager" <property name="dataSource" ref="dataSource"> 
</bean>
```

### Hibernate Transaction Setup
- Need to define a Hibernate `LocalSessionFactoryBean` which the application code can use to obtain Hibernate `Session` instances
- The `HibernateTransactionManager` needs a reference to `SessionFactory`
```xml
<bean id="txManager" class="org.springframework.orm.hibernate5.HibernateTransactionManager">
    <property name="sessionFactory" ref="sessionFactory"/>
</bean>
```

## Synchronizing Resources with Transactions
- How can application code ensure that the resources are created, reused, and cleaned up properly

### High-level synchronization approach
- Spring's template based APIs (e.g. `JdbcTemplate`) or to use native ORM APIs with transaction-aware factory beans or proxies

### Low-level synchronization approach
- Classes such as `DataSourceUtils` (for JDBC), `EntityManagerFactoryUtils` (for JPA), `SessionFactoryUtils` (for Hibernate), and so on exist at a lower level.
- When you want the application code to deal directly with the resource types of the native persistence APIs, you use these classes to ensure that proper Spring Framework-managed instances are obtained, transactions are (optionally) synchronized, and exceptions that occur in the process are properly mapped to a consistent API.
```java
Connection conn = DataSourceUtils.getConnection(dataSource);
```

### Lowest-level approach `TransactionAwareDataSourceProxy`
- At the very lowest level exists the `TransactionAwareDataSourceProxy` class. This is a proxy for a target `DataSource`, which wraps the target `DataSource` to add awareness of Spring-managed transactions. In this respect, it is similar to a transactional JNDI `DataSource`, as provided by a Java EE server.
- You should almost never need or want to use this class