# Repository adapters

Sometimes a `Team` should hold team data and nothing else. You can put database operations on a separate type without copying the fields into it. Canyon calls this a repository adapter.

Let's use a table named `team`, Canyon's default name for the Rust type `Team`. Only the row model is a Canyon entity:

```rust
use canyon_sql::macros::{canyon_entity, CanyonMapper};

#[derive(Debug, CanyonMapper)]
#[canyon_entity]
pub struct Team {
    #[primary_key]
    pub id: i64,
    pub name: String,
}
```

`CanyonMapper` reads rows into `Team`. The `#[primary_key]` marker tells generated operations which field identifies a row; `#[canyon_entity]` processes that marker on the model.

Now define a type for writes. It has no database fields of its own:

```rust
use canyon_sql::macros::{EntityDelete, EntityInsert, EntityUpdate};

#[derive(EntityInsert, EntityUpdate, EntityDelete)]
#[canyon_crud(maps_to = Team)]
pub struct TeamWriter {
    marker: (),
}
```

`maps_to = Team` says which type the operations receive. It does **not** turn `TeamWriter` into another entity. With the operation traits in scope, you can write:

```rust
use canyon_sql::crud::{EntityDelete as _, EntityInsert as _, EntityUpdate as _};

TeamWriter::insert_entity(&mut team).await?;
let affected: u64 = TeamWriter::update_entity(&team).await?;
TeamWriter::delete_entity(&team).await?;
```

There is no need to construct a `TeamWriter` here. The generated methods take a `Team` as an argument. You can derive just one or two operations if that is all the adapter should expose. Deriving all three also gives it the composite `EntityCrud` trait.

As with model methods, each operation has a `_with` form for a named datasource or compatible connection. The adapter does not start a transaction for you.

## Before using an adapter with another model

The example above works with the conventional table name `team`. Check these two cases before copying the pattern:

1. **Your table has a custom name.** Earlier in this book, `Team` maps to `teams`. Today's adapter derives still generate SQL for `team`; they do not pick up `Team`'s `table_name`. Write those repository operations explicitly for now.
2. **You want to derive `Read` on a fieldless adapter.** Its generated `find_all` selects the adapter's fields, not `Team`'s. A `marker: ()` field would become a selected column. Keep `Read` on the model or implement the repository read yourself.

> **Don't annotate the adapter as an entity.** Adding `#[canyon_entity(table_name = "teams")]` to `TeamWriter` happens to point its SQL at the right table, but it also registers the adapter as an entity. That is a macro limitation to fix, not a pattern to copy.
