# Run SQL directly

Generated operations and the query builder are not a promise to hide SQL. A report, maintenance task, or database-specific feature may be clearer as an explicit statement. Canyon exposes the configured connection through `Canyon::instance()` and the `DbConnection` trait.

The following SQL uses PostgreSQL's `$1` placeholder. Adapt placeholder syntax when using another backend:

```rust
use canyon_sql::{connection::DbConnection, core::Canyon};

let connection = Canyon::instance()?.get_connection("main")?;
let id = 42_i64;
let team: Option<Team> = connection
    .query_one::<Team>("SELECT id, name FROM teams WHERE id = $1", &[&id])
    .await?;
```

Choose the call by the shape you expect back:

| Call | Result on success |
| --- | --- |
| `query::<_, Team>(sql, params)` | All mapped rows, as `Vec<Team>` |
| `query_one::<Team>(sql, params)` | One mapped row or `None` |
| `query_one_for::<i64>(sql, params)` | One scalar value |
| `execute(sql, params)` | Number of affected rows, as `u64` |

All four return `CanyonResult`, so driver and mapping failures remain visible.

For a query whose shape is not yet represented by a model, `query_rows` returns `CanyonRows`, a wrapper over the active driver's rows. Use `len()`, `is_empty()`, or `get_row_at()` to inspect them. `first::<Team>()` maps the first row and returns `CanyonResult<Option<Team>>`: `None` means there was no first row; `Err` means mapping failed.

Backend-specific row accessors are available for advanced cases. Calling one for the wrong backend returns a mapping error.

Advanced users who depend on `canyon_core` directly can use `canyon_core::row::RowExt` at the individual-row level; the root `canyon_sql` crate does not currently re-export it.

- `get_postgres`, `get_mysql`, and `get_mssql` read required values.
- Their `_opt` counterparts read nullable values.

All return `CanyonResult`. A missing column or incompatible type is an error; the required getters also reject `NULL`, while the `_opt` getters represent it as `None`. These are lower-level escape hatches, not a replacement for `CanyonMapper` on a normal entity.

Values should be passed as parameters rather than interpolated into SQL. Backend syntax and identifier quoting remain your responsibility for handwritten statements. The parameterized query builder performs those steps for the SQL it generates.

`Transaction` is a low-level proxy over connection operations; it does not, by itself, begin or commit a database transaction. If you need an atomic multi-statement transaction, use a compatible backend connection and its transaction facilities deliberately.
