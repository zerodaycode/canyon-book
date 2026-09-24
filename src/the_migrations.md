# Migrations: experimental

Canyon contains a `migrations` Cargo feature, but it is **incomplete and experimental**. It is not enabled by default and is not recommended for production schema management. This book intentionally does not present a migration workflow as if it were ready.

For now, create and evolve your tables with an established tool or SQL scripts, then use Canyon for the application queries. The ordinary `Crud`, query-builder, and mapping APIs do not require the `migrations` feature.

Enabling `migrations` alone does not satisfy Canyon's backend requirement: a build still needs at least one of `postgres`, `mysql`, or `mssql`. Some older book pages and examples described compile-time table creation and data loading. They refer to an earlier design and should not be followed for the current release.

The planned direction is to bring migration errors into the same typed `CanyonError` API and redesign the feature before documenting a supported workflow. Until then, treat any migration code in the repository as experimental implementation, not a contract for applications.
