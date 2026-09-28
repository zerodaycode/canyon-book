# Read

Import `Read`, then call its methods on `Team`:

```rust
use canyon_sql::crud::Read;

let teams: Vec<Team> = Team::find_all().await?;
let team: Option<Team> = Team::find_by_pk(&42_i64).await?;
let total: i64 = Team::count().await?;
```

Notice the three result shapes:

- `find_all()` reads every mapped row; no matches gives you an empty `Vec<Team>`.
- `count()` returns the row count as `i64`.
- `find_by_pk()` uses the field marked `#[primary_key]` and returns `Ok(None)` if no row has that key.

> **No row versus failed query:** A missing table, failed connection, or failed row conversion is an `Err`, not an empty collection or `None`.

Each function has a `_with` counterpart for a named datasource or compatible connection:

```rust
let team = Team::find_by_pk_with(&42_i64, "reporting").await?;
```

Need a filter or an ordering? `select_query()` and `select_query_with(...)` start a builder instead of executing immediately. We'll use one in [Build a query](../querybuilder.md).

Without `#[primary_key]`, `find_all` and `count` can still work. `find_by_pk` cannot guess which field is the key; it returns a typed error.
