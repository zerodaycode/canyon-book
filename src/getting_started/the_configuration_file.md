# The secrets.toml file

In order for `Canyon` to work properly, it requires a `canyon.toml` file which stores the configuration details for each datasource declared. Essentially, this file contains the user configuration information for every datasource, each of which represents a connection to a database.

The `canyon.toml` file is a regular `.toml` file and should be kept ***secure***, as it contains sensitive information such as passwords and credentials needed to access the databases. Therefore, it should not be stored in a public repository or be accessible to unauthorized users.

## The format

A `Canyon-SQL` `canyon.toml` configuration file should be structured like this:

```toml
[canyon_sql]
datasources = [
    {name = 'postgres_docker', properties.db_type = 'postgresql', properties.username = 'postgres', properties.password = 'postgres', properties.host = 'localhost', properties.port = 5438, properties.db_name = 'postgres'},
    {name = 'sqlserver_docker', properties.db_type = 'sqlserver', properties.username = 'sa', properties.password = 'SqlServer-10', properties.host = 'localhost', properties.port = 1434, properties.db_name = 'master'}
]
```

Depending on your preferences you may also use "*TOML array of tables*":

``` toml
[[canyon_sql.datasources]]
name = 'postgres_docker'

[canyon_sql.datasources.properties]
db_type = 'postgresql'
username = 'postgres'
password = 'postgres'
host = 'localhost'
port = 5438
db_name = 'postgres'

[[canyon_sql.datasources]]
name = 'sqlserver_docker'

[canyon_sql.datasources.properties]
db_type = 'sqlserver'
username = 'sa'
password = 'SqlServer-10'
host = 'localhost'
port = 1434
db_name = 'master'
```

In this configuration file, you should declare the following properties for each datasource:

 - **name:** A unique identifier for the datasource.
 - **db_type:** A string value that represents the type of database engine being used. Possible values are `'postgres'` or `'postgresql'` for PostgreSQL engines, and `'sqlserver'` or `'mssql'` for *Microsoft SQL Server* engines.
 - **username:** The username used to access the database.
 - **password:** The password associated with the above username.
 - **host:** (Optional) The address where the database lives. Defaults to localhost.
 - **port:** (Optional) A number representing the address port value. Defaults to the default value of the engine.
 - **db_name:** The name of the database that you want to connect with.
 - **migrations:** (Optional) Allows you to disable migrations for a whole datasource. Can be set to `'Enabled'` or `'enabled'` to explicitly enable migrations for this datasource (not necessary, since this is the default value), or `'Disabled'` or `'disabled'` to avoid processing migrations for this datasource.

It is worth noting that migrations are a powerful tool that will be discussed in later chapters, but for now, you may choose to skip them.

> **Note:** It is recommended to use the TOML syntax for Canyon-SQL configuration file as presented [above](#the-format), with one table called `[canyon_sql]`, a property named `datasources`, which is a collection of `datasources`, and json-like objects inside the collection representing every datasource that should be configured.

> **Note:** It should be noted that inlined TOML tables, which are the objects inside the datasources collection, should be in a unique line without any jumps. This is an imposed restriction by the *TOML standard itself*. You can read more about this at the following link: https://toml.io/en/v1.0.0#inline-table. You may choose to use the classic table notation if you prefer. However, note that the example above may appear with new line jumps due to the TOML formatting.

And that's all! Your program now should compile successfully if you run `cargo build` again, noticing that everything it's reading to start to write some code.

Upon completing these steps, your program should now compile successfully. By running `cargo build` again, the compiler should be equipped with the necessary information for us to start writing code.