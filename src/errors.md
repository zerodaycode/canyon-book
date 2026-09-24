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

`find_all()` uses an empty vector for no rows. A relationship parent lookup returns `Ok(None)`, and a reverse child lookup returns an empty vector. The mapper does not turn missing columns, unexpected `NULL` values, or incompatible types into absence.

Handle a `QueryBuilder` error before executing anything. In particular, an empty `IN` list or a duplicated `SET` is rejected as a construction problem. This keeps invalid SQL from reaching a driver, but a valid builder cannot guarantee your schema or permissions are correct.

When an application needs its own error type, convert `CanyonError` at the boundary where it has context—an HTTP handler, service, or repository. Preserve the original source for logs and diagnostics rather than replacing it with a generic string.
