## Consistent Exception Hierarchy
- Spring wraps the original exception to `DataAccessException`. 
- In addition to JDBC exceptions, Spring also wraps JPA & Hibernate specific exceptions
- To convert an exception to Spring exception, use the `convertHibernateAccessException` or `convertJpaAccessException` methods of `SessionFactoryUtils`
![DataAccessException](https://docs.spring.io/spring-framework/docs/current/reference/html/images/DataAccessException.png)

## Annotations for DAO or Repository Classes
- Use `@Repository`annotation to guarantee that DAOs and repository classes provide exception translation