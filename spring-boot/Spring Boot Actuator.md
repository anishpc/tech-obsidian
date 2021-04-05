

## Implementing Custom Actuator Endpoints
- You can create or extend a custom actuator endpoint. You need to mark your class with `@Endpoint` and also mark your methods with `@ReadOperation`, `@WriteOperation` or `@DeleteOperation`

### JMX or Web
- By default, your endpoint is exposed over JMX and through the web over HTTP
- To expose only via JMX, use `@JmxEndpoint`.
- To expose only via Web, use `@WebEndpoint`

### Method signature
- **Input parameters** : You can accept parameters, which are converted to the right type needed by the `ApplicationConversionService` instance. These types consume the `application/vnd.spring-boot.actuator.v2+json` and `application/json` content type
- **Return Types** : You can return any type (even `void` or `Void`) in any method signature. Normally, the returned content-type varies depending on the type.
	- for `org.springframework.core.io.ResourceType` -- return is `application/octet-stream` content-type
	- for other types : it returns `application/vnd.spring-boot.actuator.v2+json, application/json` content type
- When using your custom actuator endpoint over the web, the operations have their own HTTP method defined: `@ReadOperation` (Http.GET), `@WriteOperation` (Http.POST) and `@DeleteOperation` (Http.DELETE).