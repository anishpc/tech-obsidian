## Using JDBC core classes
- `JdbcTemplate` :
	- classic and most popular
	- lowest-level and all others use this under the covers
- `NamedParameterJdbcTemplate`
	- wraps a `JdbcTemplate` to provide named parameters instead of the traditional JDBC `?` placeholders
	- provides good documentation, ease of use for multiple parameters
- `SimpleJdbcInsert` & `SimpleJdbcCall`
	- need to provide only name of the table or procedure and map of attributes
- RDBMS objects : `MappingSqlQuery`, `SqlUpdate` and `StoredProcedure`

## Using `JdbcTemplate`
The JdbcTemplate class : 
- runs SQL queries
- updates statements and stored procedure calls
- performs iteration over `ResultSet` instances and extraction of returned parameter values
- catches JDBC exceptions and translates them to generic, Spring exception hierarchy