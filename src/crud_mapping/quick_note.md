# Choosing a datasource

Methods without `_with` use the first active datasource in `canyon.toml`. To choose another database, pass its name—or a compatible connection—to the `_with` form:

```rust
use canyon_sql::crud::Read;

let default_teams: Vec<Team> = Team::find_all().await?;
let analytics_teams: Vec<Team> = Team::find_all_with("analytics").await?;
```

`find_by_pk`, `count`, `insert`, `update`, and `delete` have the same choice. A misspelled name produces `ConnectionError::DatasourceNotFound`; it does not silently use the default.

> **Two meanings of `_with`:** `Team::find_all_with("analytics")` chooses a connection. `Team::select_query_with(DatabaseType::MySQL)` chooses a SQL *dialect*, not a connection. The same is true of `update_query_with` and `delete_query_with`.

Build a query for the intended dialect, then run it with `launch_with("mysql_datasource")` or send its SQL and parameters through a matching connection. `select_query()` followed by `launch_default()` uses the default datasource for both steps.

Passing an existing connection to `_with` keeps that operation on the connection you chose. It does not make two different datasources part of one transaction.
