# Insert

An insert starts with a mutable entity. For an auto-generated numeric key, initialize the key with its default value and let the database supply the real one:

```rust
use canyon_sql::crud::Insert;

let mut team = Team {
    id: 0,
    name: "Blue".to_owned(),
};

team.insert().await?;
println!("new key: {}", team.id);
```

The result is `CanyonResult<()>`. On success, Canyon assigns the generated primary key back to `team.id`. If `#[primary_key(autoincremental = false)]` is used, the key is included in the inserted values rather than fetched from the database.

Use `team.insert_with("reporting").await?` to target a named datasource. It also accepts a compatible connection. The table, column types, and any database constraints must agree with the entity; Canyon does not create the table or suppress a constraint failure.

There is no current bulk `insert_into` API on `Crud`. Older examples that use it predate the present interface. For multiple rows, insert them individually or use an explicit SQL statement through `DbConnection`, taking responsibility for transaction boundaries and backend syntax.
