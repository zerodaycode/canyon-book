# API map

When an example names a type but not its import, use this map. It lists the main exports of the root `canyon_sql` crate. A feature-gated item exists only when you enable its backend or experimental feature.

| Path | What it contains |
| --- | --- |
| `canyon_sql::{CanyonResult, CanyonError}` | The common result alias and top-level typed error |
| `canyon_sql::macros` | `canyon_entity`, `Fields`, `CanyonMapper`, `Crud`, the individual operation derives, runtime-adapter derives, `main`, and `canyon_tokio_test` |
| `canyon_sql::core` | `Canyon` initialization and datasource access, `RowMapper`, `CanyonRows`, `Transaction`, and error types |
| `canyon_sql::crud` | `Read`, `Insert`, `Update`, `Delete`, `Crud`, plus `EntityInsert`, `EntityUpdate`, `EntityDelete`, and `EntityCrud` |
| `canyon_sql::connection` | `DbConnection`, `DatabaseType`, and `DatabaseConnector` |
| `canyon_sql::query` | `Query`, `QueryParameter`, `ColumnRef`, operators, and query-builder traits and types |
| `canyon_sql::date_time` | Selected `chrono` date and time types for mapped fields |
| `canyon_sql::db_clients` | Enabled low-level PostgreSQL, MySQL, or SQL Server driver re-exports |
| `canyon_sql::runtime` | Tokio and related runtime re-exports used by Canyon |
| `canyon_sql::migrations` | Experimental migration modules, only with the `migrations` feature |

The derive macro `Crud` and the trait `Crud` have the same name but different jobs: import the derive from `macros`, and import the trait from `crud` to call its methods. The same applies to `EntityInsert`, `EntityUpdate`, and `EntityDelete`.

`CanyonMapper` maps rows; `Fields` supplies query identifiers. A model needs `#[canyon_entity]` for explicit table/schema metadata or Canyon field markers, not merely because it has a derive. [Entities and mapping](./canyon_entities.md#when-do-you-need-canyon_entity) works through that choice.

On a repository adapter, `#[canyon_crud(maps_to = Team)]` names the mapped entity. It does not make the adapter an entity or copy `Team`'s custom table name; see [Repository adapters](./repository_adapters.md).

For exact signatures, use the public Rust API in the [Canyon-SQL source](https://github.com/zerodaycode/Canyon-SQL) or the published crate documentation.
