# Update

An update uses the entity's primary key to identify the row. Change the Rust value, then persist it:

```rust
use canyon_sql::crud::{Read, Update};

let mut team = Team::find_by_pk(&42_i64)
    .await?
    .expect("the team must exist");

team.name = "Blue Tigers".to_owned();
let affected: u64 = team.update().await?;
```

`affected` is the number of rows reported by the database. A successful statement may affect zero rows, for instance if the key no longer exists. Check the count when your application requires exactly one update.

`update_with("reporting")` chooses a named datasource or compatible connection. If there is no `#[primary_key]`, Canyon cannot safely construct the key predicate and returns a typed error.

For a conditional update over several rows, `Team::update_query()?` starts the query builder. The [query-builder chapter](../querybuilder.md) explains its `set_values` operation and why an unrestricted update deserves particular care.
