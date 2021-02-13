## Using DataSource





## Configuring DataSource
- Standalone : data source is configured in the `@Configuration` class
- Spring Boot : Data source is auto-configured by providing details in `application.properties`
- Application Server : data source to be fetched via JNDI via `JndiDataSourceLookup`/`JndiTemplate`



## Initializing DataSource for tests
```java
@Configuration
public class DataSourceTestDataConfiguration {
	@Value("classpath:/db-schema.sql")
	private Resource schemaScript;
	@Value("classpath:/db-test-data.sql")
	private Resource dataScript;
	@Bean
	public DataSourceInitializer dataSourceInitalizer(DataSource dataSource) {
		DataSourceInitializer dsInitializer = new DataSourceInitializer();
		dsInitializer.setDataSource(dataSource);
		dsInitializer.setDatabasePopulator(databasePopulator());
		return dsInitializer;
	}
	@Bean
	public DatabasePopulator databasePopulator() {
		ResourceDatabasePopulator dbPopulator = new ResourceDatabasePopulator();
		dbPopulator.addScript(schemaScript);
		dbPopulator.addScript(dataScript);
		return dbPopulator;
	}
}

```