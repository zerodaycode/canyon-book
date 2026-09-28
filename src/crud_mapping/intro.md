# Working with data

We have a `Team` that Canyon can map from a row. Deriving `Crud` adds the common read and write operations to that model:

```rust
#[derive(Debug, Fields, Crud, CanyonMapper)]
#[canyon_entity(table_name = "teams")]
pub struct Team {
    #[primary_key]
    pub id: i64,
    pub name: String,
}
```

You need not expose all four. The smaller derives are:

- `Read` for lookups.
- `Insert` for new rows.
- `Update` for changes.
- `Delete` for removals.

When you call a generated method, import its matching trait from `canyon_sql::crud`. The derive creates the implementation; the trait makes the method available at the call site.

All database calls are asynchronous and return `CanyonResult<T>`. A query can succeed without finding a row: `find_all()` returns an empty vector, and `find_by_pk()` returns `Ok(None)`. A connection or mapping failure returns `Err` instead.

We'll start with reads, then move through writes and relationships. Once those methods feel familiar, the query builder will make more sense.
