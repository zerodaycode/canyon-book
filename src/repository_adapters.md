# Repository adapters

A model that derives `Crud` is convenient when the model itself owns database operations. Some applications instead separate domain data from the component that persists it. Canyon supports that shape without requiring a second copy of the entity fields.

The mapped entity needs `CanyonMapper`. This example also uses `#[canyon_entity]` for its table name and primary-key annotation; the [entity chapter](./canyon_entities.md#when-do-you-need-canyon_entity) explains when the attribute is needed:

```rust
#[derive(Debug, CanyonMapper)]
#[canyon_entity(table_name = "teams")]
pub struct Team {
    #[primary_key]
    pub id: i64,
    pub name: String,
}
```

An adapter can derive only the operations it intends to expose. The `maps_to` annotation binds those generated operations to `Team`:

```rust
use canyon_sql::macros::{
    canyon_entity, EntityDelete, EntityInsert, EntityUpdate,
};

#[derive(EntityInsert, EntityUpdate, EntityDelete)]
#[canyon_crud(maps_to = Team)]
#[canyon_entity(table_name = "teams")]
pub struct TeamWriter {
    marker: (),
}
```

With the matching traits in scope, it accepts a `Team` passed to the operation rather than persisting `self`:

```rust
use canyon_sql::crud::{EntityDelete as _, EntityInsert as _, EntityUpdate as _};

TeamWriter::insert_entity(&mut team).await?;
let affected: u64 = TeamWriter::update_entity(&team).await?;
TeamWriter::delete_entity(&team).await?;
```

The adapter has a few useful choices:

- Use a `_with` version to pass a named datasource or compatible connection.
- Use `EntityCrud` when insert, update, and delete are available for the same mapped entity; it is the composite runtime contract.
- Derive `Read` separately on the adapter for `find_all`, `find_by_pk`, `count`, and a select builder that returns the mapped type.

This feature is useful for a repository layer, but it does not create a new database transaction or hide the connection. The [source integration example](https://github.com/zerodaycode/Canyon-SQL/blob/main/tests/crud/hex_arch_example.rs) shows a service and adapter wired together; the [compile fixture](https://github.com/zerodaycode/Canyon-SQL/blob/main/tests/ui/src/bin/selective_entity_operations.rs) shows independently derived operations.
