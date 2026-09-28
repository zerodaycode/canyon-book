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

`affected` is the database's affected-row count. It can be zero even though the statement succeeded—for example, if another operation removed that team after you read it. If your application expects exactly one row, check the count.

`update_with("reporting")` chooses a named datasource or compatible connection. If there is no `#[primary_key]`, Canyon cannot safely construct the key predicate and returns a typed error.

To update rows by some condition other than the model's primary key, start with `Team::update_query()?`. The [query-builder chapter](../querybuilder.md) shows how to set values and add the predicate. Always check that predicate before running a multi-row update.
