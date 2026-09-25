# Getting started

We will use PostgreSQL for the first working example. The same model code can be used with MySQL or SQL Server; the Cargo feature and datasource configuration decide which driver Canyon uses.

To get a query running, you need:

1. A project with a SQL backend [enabled](./initial_setup.md).
2. A datasource in [`canyon.toml`](./the_configuration_file.md).
3. A [table](./configuring_the_database.md) that matches the Rust model.

Once these are in place, `Canyon::init().await?` opens the configured pools and the model can query the database.

> **Already have a database?** Use your own connection details and existing table. None of these steps requires Canyon's experimental migrations feature.
