# Delete

Delete uses the annotated primary key, just as update does:

```rust
use canyon_sql::crud::Delete;

team.delete().await?;
```

The generated method returns `CanyonResult<()>`, not an affected-row count. If you need to confirm that a row is gone, read it again with `find_by_pk`. Without `#[primary_key]`, Canyon returns an error rather than deleting without a key predicate.

`delete_with("reporting")` targets a named datasource or compatible connection. Database foreign-key constraints still apply: Canyon's relationship annotation generates lookup methods; it does not bypass or create those constraints.

For a filtered delete, start with `Team::delete_query()?` and add a predicate. Check the SQL before you execute a statement that could remove many rows.
