# Getting started

We'll use PostgreSQL to get the first query running. MySQL and SQL Server follow the same path: enable the matching Cargo feature and configure a datasource for that backend.

To get a query running, you need:

1. A project with a SQL backend [enabled](./initial_setup.md).
2. A datasource in [`canyon.toml`](./the_configuration_file.md).
3. A [table](./configuring_the_database.md) that matches the Rust model.

Once those three pieces are in place, call `Canyon::init().await?` to open the connection pools. Then `Team::find_all().await?` can read your table.

> **Already have a database?** Use its connection details and an existing table. You do not need Canyon's experimental migrations feature.
