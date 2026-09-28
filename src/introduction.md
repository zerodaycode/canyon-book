# Introduction

Writing one SQL query is rarely the hard part. The work piles up around it: opening connections, passing parameters, turning rows into Rust values, and repeating the same column names across the application.

Canyon-SQL uses a Rust model to generate much of that routine work. You still own the database schema and can still write SQL when it is the clearest tool.

There are three ways to work with data in Canyon:

- **Generated operations** handle common reads and writes on a mapped Rust struct.
- **The query builder** adds filters, joins, ordering, and projections.
- **A connection** lets you send SQL directly when neither of those fits.

Canyon is asynchronous and supports PostgreSQL, MySQL, and SQL Server. You can enable more than one backend and configure several named datasources. The first active datasource is the default; an explicit connection or datasource name lets an operation use another one.

## How to read this book

We'll connect to PostgreSQL, define a `Team`, and use it to read and write rows. Once that works, we'll add relationships and build queries that go beyond the generated methods. Later chapters cover errors, direct SQL, and repository adapters. Examples target Canyon-SQL 0.5.1.

> **Before you start:** Create the tables with your usual schema tool. Canyon's `migrations` feature is [experimental and incomplete](./the_migrations.md); none of the examples depend on it.

> **New to async Rust?** The [Async Book](https://rust-lang.github.io/async-book/) is useful background. You need not know how Canyon's procedural macros are implemented, but their generated code is still Rust: traits must be in scope, types must match, and a database error is not an absent row.

Canyon-SQL is [MIT licensed](https://github.com/zerodaycode/Canyon-SQL/blob/main/LICENSE). If an example no longer matches the code, open an issue or a pull request in the [book repository](https://github.com/zerodaycode/canyon-book); the [Canyon-SQL repository](https://github.com/zerodaycode/Canyon-SQL) holds the implementation.
