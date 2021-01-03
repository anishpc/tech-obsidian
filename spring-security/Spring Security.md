# Basic Security Terms
- `Principal` is the term that signifies the user, device, or system that could perform an action within the application
-    `Credentials` are identification keys that a principal uses to confirm its identity
-   `Authentication` is the process of verifying the validity of the principal's credentials
-   `Authorization` is the process of deciding if an authenticated user is allowed to perform a certain action within the application
-   `Secured item` is the term used to describe any resource that is being secured

```<servlet>
	<servlet-name>pet-dispatcher</servlet-name> 
	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class> 
		<init-param> 
			<param-name>contextConfigLocation</param-name> 
			<param-value> /WEB-INF/spring/mvc-config.xml /WEB-INF/spring/app-config.xml </param-value> 
		</init-param> 
		<load-on-startup>1</load-on-startup> 
	</servlet>

