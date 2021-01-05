## Proxy Mechanism (Q. 19)

### Basics
-   Spring AOP uses either JDK dynamic proxies or CGLIB to create the proxy for a given target object. JDK dynamic proxies are built into the JDK, whereas CGLIB is a common open-source class definition library
-   If the target object to be proxied implements at least one interface, a JDK dynamic proxy is used. All of the interfaces implemented by the target type are proxied. If the target object does not implement any interfaces, a CGLIB proxy is created.
-   If you want to force the use of CGLIB proxying (for example, to proxy every method defined for the target object, not only those implemented by its interfaces), you can do so. However, you should consider the following issues:
    -   With CGLIB, `final` methods cannot be advised, as they cannot be overridden in runtime-generated subclasses.
    -   As of Spring 4.0, the constructor of your proxied object is NOT called twice anymore, since the CGLIB proxy instance is created through Objenesis. Only if your JVM does not allow for constructor bypassing, you might see double invocations and corresponding debug log entries from Spring’s AOP support.

### Limitations of JDK Dynamic Proxy
- Requires proxy object to implement the interface
- Only interface methods will be proxied
- No support for self-invocation

###   Limitations of CGLIB proxy
- Does not work for final classes
- Does not work for final methods
- No support for self-invocation
- CGLIB proxies intercept only public method calls! Do not call non-public methods on such a proxy. They are not delegated to the actual scoped target object.
- In classes marked with `@Configuration`, Spring uses CGLIB when invoking methods annotated with `@Bean` so that the bean is returned without instantiating everytime for singleton instances

### Proxy Advantages
- Ability to change behaviour of existing beans without changing original code
- Separation of concerns (logging, transactions, security)

### Proxy Disadvantages
- May create code hard to debug
- Needs to use unchecked exception for exceptions not declared in original method
- May cause performance issues if before/after section in proxy code is using IO (network, disk)
- May cause unexpected equals operator (==) results since Proxy Object and Proxied Object are two different objects