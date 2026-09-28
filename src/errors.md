# Handle errors

Every fallible public operation returns `CanyonResult<T>`, short for `Result<T, CanyonError>`. Start with the outer error variant to find the failing part of the operation:

| Family | Typical cause |
| --- | --- |
| `Configuration` | Missing or invalid `canyon.toml`, incompatible authentication, invalid pool bounds |
| `Connection` | Uninitialized Canyon, unknown datasource, busy or failed connection |
| `Query` | The database rejected a statement or a driver could not convert a query value |
| `QueryBuilder` | An invalid query shape, such as empty `IN` or `SET` |
| `Mapping` | A required column is absent, `NULL` where a value is required, or a type conversion failed |

The error enums are `#[non_exhaustive]`; include a fallback arm when you match them. Canyon preserves the underlying driver error through `std::error::Error::source()`, so you can inspect the original cause.

Here is one way to handle a lookup at the call site:

```rust
use canyon_sql::{CanyonError, crud::Read};

match Team::find_by_pk(&42_i64).await {
    Ok(Some(team)) => println!("Found {}", team.name),
    Ok(None) => println!("No team has that key"),
    Err(CanyonError::Connection(error)) => eprintln!("Connection: {error}"),
    Err(error) => eprintln!("Query failed: {error}"),
}
```

## What does “not found” return?

Read the return type:

- **A list lookup** — `find_all()` or `Player::find_all_by_team(&team)` — succeeds with an empty `Vec` when there are no matching rows. Looping over it simply does nothing.
- **A single-row lookup** — `find_by_pk()` or `player.find_team()` — succeeds with `Ok(None)` when no row matches. Decide at the call site whether that is acceptable or should become an application-level “not found” error.
- **A scalar lookup** such as `query_one_for()` expects a value. No row is reported as an error; there is no `Option` in its return type.

> **A broken row is not an absent row.** A missing required column, unexpected `NULL`, or incompatible type produces `Err(CanyonError::Mapping(...))`.

Some mistakes fail before Canyon sends SQL. An empty `IN` list or a duplicated `SET` returns a query-builder error during `build()`. A nonexistent column or missing database permission is different: only the database can report it.

If your application has its own error type, convert `CanyonError` where you have enough context—for example, in an HTTP handler or service. Keep the source error for logs instead of replacing it with a generic string.
