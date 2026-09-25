# API map

The examples in this book begin with the high-level derives. When you need to find a particular type, this map shows where Canyon exposes it from the root `canyon_sql` crate. Feature-gated exports only exist when their backend or experimental feature is enabled.

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

The derive macro named `Crud` and the trait named `Crud` live in different namespaces. Import the derive from `macros` and the trait from `crud`. The same distinction applies to `EntityInsert`, `EntityUpdate`, and `EntityDelete`.

If a generated method appears to be missing, check whether its trait is in scope.

`CanyonMapper` maps rows; `Fields` supplies query identifiers. `#[canyon_entity]` is needed for explicit table/schema metadata and for models using Canyon's field annotations, but not for every derive with default naming. See [Entities and mapping](./canyon_entities.md#when-do-you-need-canyon_entity).

On a repository adapter, `#[canyon_crud(maps_to = Team)]` tells an operation derive which mapped entity it acts on.

For the exact signatures, consult the public Rust API in the [Canyon-SQL source](https://github.com/zerodaycode/Canyon-SQL) or the published crate documentation. This book concentrates on how the pieces fit together and on the behavior that matters at call sites.
