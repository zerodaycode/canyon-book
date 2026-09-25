# Working with data

Once an entity and `CanyonMapper` are in place, Canyon can perform the common database operations without a handwritten SQL statement. The `Crud` derive is the convenient choice when you want the complete set:

```rust
#[derive(Debug, Fields, Crud, CanyonMapper)]
#[canyon_entity(table_name = "teams")]
pub struct Team {
    #[primary_key]
    pub id: i64,
    pub name: String,
}
```

You can derive only the operations a model or adapter should expose:

- `Read` for lookups.
- `Insert` for new rows.
- `Update` for changes.
- `Delete` for removals.

The matching trait from `canyon_sql::crud` must be in scope when you call its methods.

All database calls are asynchronous and return `CanyonResult<T>`. An empty result is not a failed query: `find_all()` returns an empty vector, while `find_by_pk()` returns `Ok(None)`.

> **Absence is not failure:** Errors remain errors, with the driver source preserved when there is one.

The following chapters start with the simplest operation and move toward writes and relationships. The query builder comes afterward, where it is easier to see what it adds.
