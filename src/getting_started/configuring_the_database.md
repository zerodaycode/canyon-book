# Prepare a database

Canyon maps existing tables; the ordinary CRUD path does not provision your schema. The PostgreSQL `teams` table in the [installation example](./initial_setup.md) has an identity key and a non-null name, matching the model in the next chapter. If your table uses different names, tell Canyon with `#[canyon_entity(table_name = "...", schema = "...")]`.

For local development of Canyon itself, the source repository includes a Docker Compose setup with PostgreSQL, MySQL, and SQL Server:

```sh
docker compose -f docker/docker-compose.yml up -d --wait
```

The PostgreSQL and MySQL fixtures are loaded by their container startup scripts. SQL Server needs an additional, ignored initializer test:

```sh
cargo test -p tests --test canyon_integration_tests --all-features \
  initialize_sql_server_docker_instance -- --ignored --test-threads=1
```

These commands belong to a checkout of the [Canyon-SQL source repository](https://github.com/zerodaycode/Canyon-SQL), not to an application that depends on the published crate. The test datasource settings live in `tests/canyon.toml` and use local-only credentials and TLS choices.

Migrations remain [experimental](../the_migrations.md). Do not enable them just to make the examples in this book work: create your tables with a schema tool you trust, then let Canyon read and write the rows.
