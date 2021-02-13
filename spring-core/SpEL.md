### What is SpEL
- Spring Expression Language (SpEL) allows to to query and manipulate object graphs during runtime
- SpEL can be used independently by using `ExpressionParser` and `EvaluationContext` or can be used on top of fields, method parameters, constructor arguments via `@Value` annotation
- Usage in `#{ <expression string> }`

### SpEL Compilation
- Why? : SpEL expressions are usually interpreted during runtime which provides a lot of dynamic flexibility but does not provide optimum performance
- Solution : To address this, Spring 4.1 introduced possibility to compile expressions. 
- How : During evaluation, the compiler generates a Java class that embodies the expression behaviour at runtime and uses that class to achieve much faster Expression evaluation
- Type inference : Since during compilation, reference types of properties are unknown, the compiler finds out the types during the first interpreted evaluation
- Issues with type inference : if the types of various expression elements change over time, this can cause issues and hence compilation is suited to expressions whose type information is not going to change on repeated evaluations

#### Compiler Configuration
- Compiler is turned off by default, you can turn it on by : 
	- Parser configuration
	- System property -- `spring.expression.compiler.mode`

#### Compiler Operation Modes
- Compiler can operate in 3 modes (`SpelCompilerMode`)
	1. Off - default
	2. `IMMEDIATE` - expressions are compiled as soon as possible - typically after first interpreted evaluation. If the compiled expression fails (typically due to type changing), the caller of the expression evaluation receives an exception
	3. `MIXED` - the expressions silently switch between interpreted and compiled mode over time. After some number of interpreted runs, they switch to compiled form and, if something goes wrong with the compiled form (like type changing), the expression automatically switches back to interpreted form again. Sometime later, it may generate another compile form and switch to it. The exception thrown in the `IMMEDIATE` mode is handled internally in this mode
- Partial Success `MIXED` - `IMMEDIATE` mode exists because `MIXED` mode could cause issues for expressions that have side effects. If a compiled expression blows up after partially succeeding, it may have already done something that has affected the state of the system. If this has happened, the caller may not want it to silently re-run in interpreted mode.

#### Compiler Limitations
The initial focus has been on the common expressions that are likely to be used in performance-critical contexts. The following kinds of expression cannot be compiled at the moment:
- Expressions involving assignment
- Expressions relying on the conversion service
- Expressions using custom resolvers or accessors
- Expressions using selection or projection

### Annotation Configuration
```java
//Method or Constructor parameters
public class FieldValueTestBean { 
	@Value("#{ systemProperties['user.region'] }") 
	private String defaultLocale;
}

//Setter method
public class PropertyValueTestBean { 
	
	@Value("#{ systemProperties['user.region'] }") 
	public void setDefaultLocale(String defaultLocale) { 
		this.defaultLocale = defaultLocale; 
	}
}
//Autowired methods and constructors can also use the "@Value" annotation
public class SimpleMovieLister { 
	
	@Autowired 
	public void configure(MovieFinder movieFinder, @Value("#{ systemProperties['user.region'] }") String defaultLocale) { 
		this.movieFinder = movieFinder; 
		this.defaultLocale = defaultLocale; 
	}
}
//Constructor
public class MovieRecommender {

	public MovieRecommender(CustomerPreferenceDao customerPreferenceDao, @Value("#{systemProperties['user.country']}") String defaultLocale) { 		
		this.customerPreferenceDao = customerPreferenceDao; 
		this.defaultLocale = defaultLocale; 
	}
}
```

### References in SpEL
Expression format : `@Value("#{ <expr> }")`. The `<expr>` value is as below
- Static fields from class - `T(com.example.Person).DEFAULT_NAME`
- Static methods from class - `T(com.example.Person).getDefaultName()`
- Spring Bean property - `@person.name`
- Spring Bean method - `@person.getName()`
- SpEL variables - `#personName`
- Object property on reference assigned to SpEL variables - `#person.name`
- Object method on reference assigned to SpEL variables - `#person.getName()`
- Spring application env. properties - `environment['app.file.property']`
- System properties - `systemProperties['app.vm.property']`
- System environment properties - `systemEnvironment['JAVA_HOME']`