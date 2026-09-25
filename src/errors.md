# Handle errors

Every public operation that can fail returns `CanyonResult<T>`, an alias for `Result<T, CanyonError>`. The outer enum tells you where a failure occurred:

| Family | Typical cause |
| --- | --- |
| `Configuration` | Missing or invalid `canyon.toml`, incompatible authentication, invalid pool bounds |
| `Connection` | Uninitialized Canyon, unknown datasource, busy or failed connection |
| `Query` | The database rejected a statement or a driver could not convert a query value |
| `QueryBuilder` | An invalid query shape, such as empty `IN` or `SET` |
| `Mapping` | A required column is absent, `NULL` where a value is required, or a type conversion failed |

The families are `#[non_exhaustive]`, so match with a fallback arm. Driver errors remain available through the standard `std::error::Error::source()` chain; Canyon does not collapse them into an unstructured `Other` variant.

An absent row is different from an error:

```rust
use canyon_sql::{CanyonError, crud::Read};

match Team::find_by_pk(&42_i64).await {
    Ok(Some(team)) => println!("Found {}", team.name),
    Ok(None) => println!("No team has that key"),
    Err(CanyonError::Connection(error)) => eprintln!("Connection: {error}"),
    Err(error) => eprintln!("Query failed: {error}"),
}
```

The return type tells you what “nothing found” means:

- **A list lookup** — `find_all()` or `Player::find_all_by_team(&team)` — succeeds with an empty `Vec` when there are no matching rows. Looping over it simply does nothing.
- **A single-row lookup** — `find_by_pk()` or `player.find_team()` — succeeds with `Ok(None)` when no row matches. Decide at the call site whether that is acceptable or should become an application-level “not found” error.
- **A scalar lookup** such as `query_one_for()` expects a value. No row is reported as an error; there is no `Option` in its return type.

None of these rules excuses a broken row. If a selected column is missing, unexpectedly `NULL`, or has an incompatible type, mapping returns `Err(CanyonError::Mapping(...))`. Check the error rather than treating it as an empty result.

An empty `IN` list or a duplicated `SET` fails while building the query, before any SQL reaches the driver. A builder can reject those malformed shapes; it cannot check that your live schema has the selected columns or that the connection has permission to use them.

When an application needs its own error type, convert `CanyonError` at the boundary where it has context—an HTTP handler, service, or repository. Preserve the original source for logs and diagnostics rather than replacing it with a generic string.
