# Run SQL directly

You do not have to fit every query into a derive or a builder. A report or database-specific operation may be clearer as SQL. Get a configured connection through `Canyon::instance()` and use `DbConnection`. This example uses PostgreSQL's `$1` placeholder:

```rust
use canyon_sql::{connection::DbConnection, core::Canyon};

let connection = Canyon::instance()?.get_connection("main")?;
let id = 42_i64;
let team: Option<Team> = connection
    .query_one::<Team>("SELECT id, name FROM teams WHERE id = $1", &[&id])
    .await?;
```

There are four common ways to execute a statement:

| Call | Result on success |
| --- | --- |
| `query::<_, Team>(sql, params)` | All mapped rows, as `Vec<Team>` |
| `query_one::<Team>(sql, params)` | One mapped row or `None` |
| `query_one_for::<i64>(sql, params)` | One scalar value |
| `execute(sql, params)` | Number of affected rows, as `u64` |

All four return `CanyonResult`. With no matching rows, `query` returns an empty vector and `query_one` returns `None`. A query failure returns `Err`.

## Binding values

The parameter slice holds references to values implementing `QueryParameter`. Supported values include:

- Numeric types such as `i16`, `i32`, `i64`, `u32`, `f32`, and `f64`.
- `bool`, strings, and several `chrono` date and time types.
- Nullable forms of selected types.

There is **no blanket implementation for every `Option<T>`**. An unsupported type fails to compile.

Keep parameter values alive through the async call. Bind values as shown above; do not interpolate them into the SQL string. On MySQL, Canyon normalizes `DateTime<Utc>` and `DateTime<FixedOffset>` values to UTC before binding them.

## Rows without a model

If a query does not yet have a model, `query_rows` returns `CanyonRows`. Start with `len()`, `is_empty()`, or `get_row_at()`. When the first row *does* match a model, `first::<Team>()` maps it and returns `CanyonResult<Option<Team>>`:

- `Ok(Some(team))`: a row was mapped.
- `Ok(None)`: there was no first row.
- `Err(...)`: reading or mapping failed.

To work with driver rows directly, use `get_postgres_rows()`, `get_tiberius_rows()`, or `get_mysql_rows()`, behind their respective Cargo features. Asking for the wrong backend returns a mapping error.

If you depend on `canyon_core` directly, `canyon_core::row::RowExt` provides individual-row access. The root `canyon_sql` crate does not re-export it.

- `get_postgres`, `get_mysql`, and `get_mssql` read required values.
- Their `_opt` counterparts read nullable values.
- `columns()` returns each column's name and backend-specific `ColumnType`, useful when inspecting an unfamiliar result shape.

These getters return `CanyonResult`. Missing columns and incompatible types are errors. Required getters also reject `NULL`; the `_opt` getters represent it as `None`. Use a `CanyonMapper` model for an ordinary row.

> **Handwritten SQL is your SQL:** You choose backend syntax and quote identifiers correctly. The query builder handles those details only for statements it generates.

`Transaction` is a low-level proxy over connection operations. It does not begin or commit a database transaction by itself. For atomic multi-statement work, use a compatible backend connection and manage its transaction explicitly.
