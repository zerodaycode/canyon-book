# Migrations: experimental

Canyon has a `migrations` Cargo feature. It is **incomplete and experimental**, so this book does not give you a migration workflow to use in an application.

Create and change tables with an established schema tool or SQL scripts. Canyon's mapping, CRUD methods, and query builder work without its migrations feature.

If you experiment with migrations, keep two things in mind:

- You still need a backend feature: `postgres`, `mysql`, or `mssql`.
- Older examples of compile-time table creation and data loading describe an earlier design; do not use them as a guide for the current release.

The feature needs a redesign, including integration with the typed `CanyonError` API, before it can have a supported workflow. Until then, migration code in the repository is implementation in progress, not an application contract.
