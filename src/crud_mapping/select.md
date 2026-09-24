# Read

Reading is the least surprising way to meet the generated API. Bring `Read` into scope, then call an associated function on the entity:

```rust
use canyon_sql::crud::Read;

let teams: Vec<Team> = Team::find_all().await?;
let team: Option<Team> = Team::find_by_pk(&42_i64).await?;
let total: i64 = Team::count().await?;
```

`find_all()` reads every mapped row. `count()` returns the number of rows as `i64`. `find_by_pk()` uses the field annotated with `#[primary_key]`; its argument is borrowed because Canyon binds it as a query parameter. It returns `Ok(None)` if no row matches. A missing table, failed connection, or failed row conversion is an `Err` instead.

Each function has a `_with` counterpart for a named datasource or compatible connection:

```rust
let team = Team::find_by_pk_with(&42_i64, "reporting").await?;
```

`Read` also exposes `select_query()` and `select_query_with(...)`. These produce a builder rather than executing immediately. Use them when you need predicates, joins, or ordering; the [query-builder chapter](../querybuilder.md) follows that path.

If your model does not declare a primary key, `find_all` and `count` can still be useful. Key-based reading cannot infer which field identifies a row and reports a typed error instead of guessing.
