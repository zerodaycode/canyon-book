# Delete

Delete uses the annotated primary key, just as update does:

```rust
use canyon_sql::crud::Delete;

team.delete().await?;
```

The generated method returns `CanyonResult<()>`. It does not return an affected-row count, so check with `find_by_pk` afterward if your application must prove that the row is gone. A missing primary-key annotation is an error, not permission to delete the whole table.

`delete_with("reporting")` targets a named datasource or compatible connection. Database foreign-key constraints still apply: Canyon's relationship annotation generates lookup methods; it does not bypass or create those constraints.

For a filtered delete rather than one identified by the model's primary key, use `Team::delete_query()?` and add a predicate before execution. Inspect the generated SQL when the scope matters.
