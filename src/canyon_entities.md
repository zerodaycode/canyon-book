# Entities and mapping

A Canyon entity is a Rust struct whose fields describe a database row. It is not a second schema language: the table must exist, and the Rust field names and types must agree with the columns you intend to read.

For the `teams` table from the previous chapter:

```rust
use canyon_sql::macros::{canyon_entity, CanyonMapper, Crud, Fields};

#[derive(Debug, Fields, Crud, CanyonMapper)]
#[canyon_entity(table_name = "teams")]
pub struct Team {
    #[primary_key]
    pub id: i64,
    pub name: String,
}
```

Each annotation has a different job:

- `#[canyon_entity]` supplies optional table/schema overrides and registers the struct for the experimental migrations machinery. It also processes field annotations such as `#[primary_key]` and `#[foreign_key]` in the current macro setup.
- `CanyonMapper` converts driver rows to `Team`.
- `Crud` generates read, insert, update, and delete operations.
- `Fields` generates names and typed values for the query builder. Ordinary `find_all()` or `insert()` calls do not need it.

## When do you need `#[canyon_entity]`?

The attribute is **not** what makes every Canyon derive work. Without it, `CanyonMapper`, CRUD derives, and `Fields` can infer a table name from the Rust type. For a plain model with no field annotations and a conventional table name, that default is enough.

Use `#[canyon_entity]` when:

- The physical table or schema differs from the inferred name: pass `table_name = "..."` or `schema = "..."`.
- You annotate fields with Canyon markers such as `#[primary_key]` or `#[foreign_key(...)]`. The current attribute macro processes these markers; keep it on models that use them.

The attribute also registers an entity for the **experimental** migrations feature. That is an implementation detail today, not a reason to enable migrations for normal CRUD.

The derives read table metadata from the struct's attributes or fall back to the inferred name. Ordinary query execution does not consult a runtime entity registry.

## Table names and schemas

Without an explicit name, Canyon derives a snake_case table name from the Rust type: `TournamentDetails` maps to `tournament_details`. Existing schemas do not always follow that rule. Use explicit metadata when they do not:

```rust
#[canyon_entity(table_name = "tournament_entries", schema = "public")]
```

That metadata is used by generated CRUD and relationship operations. It is better to state a non-standard physical name once than to repeat a string in every query.

## Primary keys

`#[primary_key]` identifies the field used by key-based reads, updates, and deletes. For a numeric key, it is treated as database-generated and auto-incrementing unless you say otherwise. After a successful `insert(&mut self)`, Canyon writes the generated key back to the Rust instance.

```rust
#[primary_key(autoincremental = false)]
pub external_id: i64,
```

Use the non-incrementing form when your application supplies the key. Without a primary-key annotation, methods that need a key cannot perform their normal operation and return a typed query-builder error. A mapper can still represent tables without one.

## What `Fields` generates

For `Team`, the derive exposes three types:

- `TeamTable` represents table metadata.
- `TeamField` names a column, for example `TeamField::name`. Joins and ordering need this form.
- `TeamFieldValue` pairs a column with a value of its Rust type, for example `TeamFieldValue::name("Blue".to_owned())`. Predicates need both pieces.

`Fields` is the supported API today, and several models may derive it in one Rust module. A future major Canyon version may move toward typed column descriptors instead of these enums. That is a direction under consideration, not a removal scheduled for this release.

The [query-builder chapter](./querybuilder.md) shows how the enums work now.

## Mapping failures are real errors

A database row can contain extra columns; the derived mapper only reads the fields declared on the struct. But a missing required column, an unexpected `NULL`, or an incompatible value produces a `CanyonError::Mapping`. It does not silently become an empty result.

> **Nullable column?** Use `Option<T>` on the model field when the SQL column can contain `NULL`.

The next chapter uses `Team` to read and write rows. When code examples omit its definition, they refer to the model above.

If a derived mapper cannot represent a specialized projection, `canyon_sql::core::RowMapper` is the lower-level contract. Its backend-specific deserialization methods return `CanyonResult` and let you supply your own mapping rules.

Implementing it by hand makes you responsible for column names, nullability, and conversion errors on every enabled backend. Start with `CanyonMapper` unless the row shape genuinely demands something else.
