# The secrets.toml file

`Canyon` needs a very special file from there to work with, the `canyon.toml` file.
This is just a regular *.toml* file where store the *secrets* of the project, or, in another words,
the user configuration for every datasource declared, that every one represents a connection against
a database.

## The format

A `Canyon-SQL` `canyon.toml` configuration file should be structured like this:

```toml
[canyon_sql]
datasources = [
    {name = 'postgres_docker', properties.db_type = 'postgresql', properties.username = 'postgres', properties.password = 'postgres', properties.host = 'localhost', properties.port = 5438, properties.db_name = 'postgres'},
    {name = 'sqlserver_docker', properties.db_type = 'sqlserver', properties.username = 'sa', properties.password = 'SqlServer-10', properties.host = 'localhost', properties.port = 1434, properties.db_name = 'master'}
]
```

were you should declare:

- **name** - The unique identifier for the `datasource` that you are configuring
- **db_type** - One of the available string values within `Canyon-SQL` to represent a database engine.

  - `'postgres' or 'postgresql` for `PostgreSQL` engines
  - `'sqlserver' or 'mssql` for `Microsoft SQL Server` engines

- **username** - The database username
- **password** - The password associated with the user above
- **host** - OPTIONAL - The address where the database lives. Defaults to `localhost`
- **port** - OPTIONAL - A number representing the addreess port value. Defaults to the default value of the engine
- **da_name** - The name of the database that you want to connect with
- **migrations** - OPTIONAL - Allows you to disable a whole datasource for the migrations. Receives an string value that can be:

  - `'Enabled' or 'enabled'` for explictly enable migrations for this datasource (not necessary, it's the default)
  - `'Disabled' or 'disabled'`, for avoid processing migrations for this datasource

We still didn't talk about migrations, and for now, it's completly fine. We will learn about this powerful tool in some chapters later, but for now, just notice that is an optional property that you must desire to skip.

> NOTE: Recommend toml syntax for Canyon-SQL configuration file it's the one presented above, one table, called [canyon_sql], a property, `datasources`, which is a collection of `datasources`, and json-like objects inside the collection represent every datasource that you want to configure.

> NOTE: Inlined toml tables, that here are the objects inside our `datasources` collection, should be
in a unique line, without no jump lines. This is an imposed restriction by the `toml` standard itself. You can read [more here](https://toml.io/en/v1.0.0#inline-table). You may arbitrary choose to use the classic table notation if you prefer. Probably, the example above appears with new lines jumps, because the toml formatting, so take this in consideration.

And that's all! Your program now should compile successfully if you run `cargo build` again, noticing that everything it's reading to start to write some code.
