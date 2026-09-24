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

`query::<_, Team>(sql, params)` maps many rows to a `Vec<Team>`. `query_one::<Team>` maps zero or one row to `Option<Team>`. `query_one_for::<i64>` reads one scalar value, and `execute` returns a `u64` affected-row count. These methods return `CanyonResult`, so driver errors and mapping errors remain visible.

For a query whose shape is not yet represented by a model, `query_rows` returns `CanyonRows`, a wrapper over the active driver's rows. Its `len()`, `is_empty()`, and `get_row_at()` inspect the result. `first::<Team>()` maps the first row and returns `CanyonResult<Option<Team>>`, preserving the difference between an empty result and a failed conversion. Backend-specific row accessors exist for advanced cases and return a mapping error if used with the wrong backend.

Advanced users who depend on `canyon_core` directly can use `canyon_core::row::RowExt` at the individual-row level; the root `canyon_sql` crate does not currently re-export this trait. Its `get_postgres`, `get_mysql`, and `get_mssql` methods read a required value; the corresponding `_opt` methods read a nullable value. They return `CanyonResult` rather than panicking when a column is absent, unexpectedly `NULL`, or incompatible with the requested Rust type. These are lower-level escape hatches, not a replacement for `CanyonMapper` on a normal entity.

Values should be passed as parameters rather than interpolated into SQL. Backend syntax and identifier quoting remain your responsibility for handwritten statements. The parameterized query builder performs those steps for the SQL it generates.

`Transaction` is a low-level proxy over connection operations; it does not, by itself, begin or commit a database transaction. If you need an atomic multi-statement transaction, use a compatible backend connection and its transaction facilities deliberately.
