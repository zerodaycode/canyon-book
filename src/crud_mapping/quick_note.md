# Choosing a datasource

Every operation without `_with` uses the first active datasource in `canyon.toml`. The `_with` form accepts a datasource name or another compatible `DbConnection`:

```rust
use canyon_sql::crud::Read;

let default_teams: Vec<Team> = Team::find_all().await?;
let analytics_teams: Vec<Team> = Team::find_all_with("analytics").await?;
```

You will find the same `_with` form on `find_by_pk`, `count`, `insert`, `update`, and `delete`. A misspelled name produces `ConnectionError::DatasourceNotFound`; Canyon does not quietly fall back to the default.

Query builders need one extra distinction. `Team::select_query_with(DatabaseType::MySQL)` chooses the SQL *dialect*; it does not select a connection. Build the query, then execute it with `launch_with("mysql_datasource")`. When using `select_query()` and `launch_default()` together, both target the default datasource.

If you already hold a compatible connection, passing it to a `_with` method keeps that operation on the same connection. This is useful in repository adapters and explicit transaction handling. It does not make two separate datasources part of one transaction.
