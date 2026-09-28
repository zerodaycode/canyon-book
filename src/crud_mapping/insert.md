# Insert

Give the new `Team` a name and leave its auto-generated key at the default value:

```rust
use canyon_sql::crud::Insert;

let mut team = Team {
    id: 0,
    name: "Blue".to_owned(),
};

team.insert().await?;
println!("new key: {}", team.id);
```

The result is `CanyonResult<()>`. On success, Canyon writes the generated key into `team.id`. If you mark the key `#[primary_key(autoincremental = false)]`, provide its value yourself; Canyon inserts it instead of fetching one.

Use `team.insert_with("reporting").await?` to target a named datasource, or pass a compatible connection. Database constraints still apply: a duplicate key or missing required column will fail the insert.

> **Older examples:** `Crud` does not provide the former `insert_into` API. For multiple rows, insert them individually or write SQL through `DbConnection`. If the rows must succeed or fail together, manage the transaction explicitly.
