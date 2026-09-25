# Configure datasources

A datasource is a named connection to one database. Canyon reads datasources from a TOML file whose name starts with `canyon` and ends with `.toml`.

It searches the current working directory and its immediate descendants, to a maximum depth of two. Keep one matching file in that area; otherwise the first file found may not be the one you intended.

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

The first active datasource becomes the default for calls such as `Team::find_all()`. Give every additional datasource a distinct `name` and use an operation's `_with` form to select it.

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

For SQL Server, change the auth key to `sqlserver`, supply its credentials and database name, and choose a TLS policy appropriate to the server. Both examples require the matching Cargo backend feature to be enabled.

## Connection pools

You can tune each connection pool separately:

```toml
[canyon_sql.datasources.properties.pool]
min_size = 2
max_size = 10
```

These are also the defaults. `max_size` must be greater than zero and no smaller than `min_size`; Canyon returns a configuration error for invalid bounds.

## SQL Server TLS

Set `mssql_tls` under `[canyon_sql.datasources.properties]`:

- `"required"` (the default) encrypts and validates the server certificate.
- `"trust_server_certificate"` encrypts without validating the certificate.
- `"disabled"` turns encryption off.

> **For local tests only:** The repository's Docker fixture uses `"disabled"`. Outside a controlled test environment, prefer a valid certificate with `"required"`.

## Credentials are application secrets

> **Keep credentials private:** The values above are examples. A real `canyon.toml` contains credentials; do not publish it or commit production passwords. Arrange file permissions and deployment accordingly.

If Canyon cannot find, read, or parse the file, `Canyon::init()` returns a typed configuration error. If it can read the file but cannot open a datasource, it returns a connection error. The [error chapter](../errors.md) shows how to distinguish them.
