# Getting started

We will use PostgreSQL for the first working example. The same model code can be used with MySQL or SQL Server; the Cargo feature and datasource configuration decide which driver Canyon uses.

There are three things to prepare: a project with a SQL backend enabled, a `canyon.toml` datasource, and a table that matches the Rust model. None of these steps invokes migrations. Once they are in place, `Canyon::init().await?` opens the configured pools and the model can query the database.

The following chapters take those steps in order. If you already have a running database, start with [installation](./initial_setup.md) and use your own connection details in the [configuration](./the_configuration_file.md).
