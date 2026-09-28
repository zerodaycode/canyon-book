# Repository adapters

You can keep database operations on a separate type instead of deriving `Crud` on the row model. That type is a repository adapter: it receives an entity to insert, update, or delete, but it is **not** another Canyon entity.

Assume this example has a table named `team`, following Canyon's default name for `Team`. The row model is the only type annotated with `#[canyon_entity]`:

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

`CanyonMapper` maps database rows into `Team`. The entity attribute processes the `#[primary_key]` field marker and registers this row type for Canyon's experimental migrations machinery. The primary-key marker itself identifies the key used by generated operations.

The adapter does not need `#[canyon_entity]`. Its derives provide operations, while `maps_to` specifies the entity supplied to them:

```rust
use canyon_sql::macros::{EntityDelete, EntityInsert, EntityUpdate};

#[derive(EntityInsert, EntityUpdate, EntityDelete)]
#[canyon_crud(maps_to = Team)]
pub struct TeamWriter {
    marker: (),
}
```

With the corresponding traits in scope, these operations take a `Team`; they do not persist the `TeamWriter` value:

```rust
use canyon_sql::crud::{EntityDelete as _, EntityInsert as _, EntityUpdate as _};

TeamWriter::insert_entity(&mut team).await?;
let affected: u64 = TeamWriter::update_entity(&team).await?;
TeamWriter::delete_entity(&team).await?;
```

There are two important boundaries to this API:

- Each operation also has a `_with` form for a named datasource or compatible connection. The adapter does not create a transaction or hide connection management.
- Deriving all three operations also gives the adapter the composite `EntityCrud` trait. Derive only the operations your repository needs.

## Custom table names and reads

The earlier `Team` example uses a table named `teams`. A generated adapter for that model is **not** equivalent to the one above: today its operation derives infer `team` from the mapped Rust type, rather than reading the `teams` metadata on `Team`. Adding `#[canyon_entity(table_name = "teams")]` to `TeamWriter` makes the SQL target `teams`, but also registers the adapter as an entity. That is a workaround in the current macros, not a sound way to model a repository, so this chapter does not present it as the recommended API. Generated adapters need a fix to reuse the mapped entity's table metadata before custom names work cleanly.

There is a separate limitation with `Read`: `find_all` currently builds its selected columns from the fields of the type deriving `Read`. A marker-only adapter would therefore select `marker`, not the fields of `Team`. Do not derive `Read` on that shape expecting a working `find_all`; implement that repository read explicitly, or derive `Read` on the row model when keeping reads there fits your design.

The [repository integration example](https://github.com/zerodaycode/Canyon-SQL/blob/main/tests/crud/hex_arch_example.rs) illustrates the service/repository split. It still contains the custom-table-name workaround described above and implements its repository `find_all` explicitly; treat it as an example of the boundary, not a model of the final annotation API.
