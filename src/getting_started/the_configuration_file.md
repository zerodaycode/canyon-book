# Configure datasources

A datasource gives a name to a database connection. Put its settings in a TOML file whose name starts with `canyon` and ends with `.toml`.

Canyon searches the current working directory and its immediate descendants, up to depth two. Keep one matching file in that area so you know which configuration it will find.

## Your first datasource

For the PostgreSQL example, put this in `canyon.toml` where you run the application:

```toml
[canyon_sql]

[[canyon_sql.datasources]]
name = "main"

[canyon_sql.datasources.auth]
postgresql = { basic = { username = "postgres", password = "postgres" } }

[canyon_sql.datasources.properties]
host = "localhost"
port = 5432
db_name = "app"
```

The first active datasource becomes the default for calls such as `Team::find_all()`. Give each additional datasource a different `name`; an operation's `_with` form selects one by name.

> **Backend features matter here too:** A datasource for a backend that was not enabled at compile time is ignored. If none remain, there is no usable default connection.

All three backends currently use `basic` username/password authentication. The keys and standard ports are:

| Backend | Authentication key | Standard port |
| --- | --- | ---: |
| PostgreSQL | `postgresql` (or `postgres`) | 5432 |
| MySQL | `mysql` | 3306 |
| SQL Server | `sqlserver` (or `mssql`) | 1433 |

`host` and `db_name` are required; `port` is optional.

## Add another database

A second datasource follows the same shape. For example, a MySQL connection alongside `main` could be declared as:

```toml
[[canyon_sql.datasources]]
name = "analytics"

[canyon_sql.datasources.auth]
mysql = { basic = { username = "reporter", password = "replace-me" } }

[canyon_sql.datasources.properties]
host = "localhost"
port = 3306
db_name = "analytics"
```

For SQL Server, use `sqlserver` as the auth key, supply its credentials and database name, and choose a TLS policy. Both examples require the matching Cargo feature.

After initialization, you can see which datasources are active:

```rust
use canyon_sql::core::Canyon;

let canyon = Canyon::instance()?;
for datasource in canyon.datasources() {
    println!("{}: {:?}", datasource.name, datasource.get_db_type());
}
```

Use `get_connection("analytics")` to select by name or `get_default_connection()` for the first active datasource. An unknown name returns an error.

## Connection pools

You can tune each connection pool separately:

```toml
[canyon_sql.datasources.properties.pool]
min_size = 2
max_size = 10
```

The example shows the defaults. `max_size` must be positive and at least `min_size`; invalid bounds produce a configuration error.

## SQL Server TLS

Set `mssql_tls` under `[canyon_sql.datasources.properties]`:

- `"required"` (the default) encrypts and validates the server certificate.
- `"trust_server_certificate"` encrypts without validating the certificate.
- `"disabled"` turns encryption off.

> **Only for local tests:** The repository's Docker fixture uses `"disabled"`. For a server outside a controlled test environment, use a valid certificate and `"required"`.

## Credentials are application secrets

> **Keep credentials private:** The values above are examples. A real `canyon.toml` contains credentials; do not publish it or commit production passwords. Arrange file permissions and deployment accordingly.

If startup fails, the error tells you where to look:

- `Configuration` means Canyon could not find, read, or parse the file, or rejected one of its settings.
- `Connection` means it read the configuration but could not open a datasource.

We'll handle those errors in [Handle errors](../errors.md).
