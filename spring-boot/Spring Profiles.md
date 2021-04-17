## Profiles
### Configuring Profiles
- Any `@Component`, `@Configuration`, `@ConfigurationProperties` can be marked with `@Profile` to limit when it is loaded

> If `@ConfigurationProperties` beans are registered via `@EnableConfigurationProperties` instead of automatic scanning, the `@Profile` annotation needs to be specified on the `@Configuration` class that has the `@EnableConfigurationProperties` annotation
> In the case where `@ConfigurationProperties` are scanned, `@Profile` can be specified on the `@ConfigurationProperties` class itself

### Activating Profiles
- You can use `spring.active.profiles` `Environment` property to specify which profile is active
```properties
spring.profiles.active=dev,hsqldb
```
```
//Command line 
--spring.profiles.active=dev,hsqldb
```

### Adding Active Profiles
- `spring.profiles.active` property follows the same ordering rules as other properties. The highest `PropertySource` wins
- `setAdditionalProfiles(String... profiles)` on the `SpringApplication` entry point class can be used to **ADD** new profiles to an existing active profile
- Profile Groups can also be used to add active profiles

### Profile Groups
- A profile group allows you to define logical name for a related group of profiles
- For example, we can create a `production` group that consists of other profiles like `proddb`  and `prodmq` 
```properties
spring.profiles.group.production[0]=proddb 
spring.profiles.group.production[1]=prodmq
```
- Our application can now be started using `--spring.profiles.active=production` to active the `production`, `proddb` and `prodmq` profiles in one hit

### Programmatically Setting Profiles
- using `SpringApplication.setAdditionalProfiles(...)`
- using Spring's `ConfigurableEnvironment` interface

### Profile-specific Configuration Files
- Details in [[Spring Externalized Configuration#Profile specific files]]