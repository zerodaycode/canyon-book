# Introduction

Most applications do not struggle with one SQL query. They struggle with the repetition around a hundred of them: opening connections, naming columns again, translating rows into Rust values, and keeping everything in step as the schema changes.

Canyon-SQL takes that repetitive work and generates it from the model you write.

An entity in Canyon is an ordinary Rust struct annotated with its database identity. From that model, Canyon gives you several ways to work:

- Derives for row mapping, CRUD operations, and typed field names.
- A query builder for filters, joins, and other queries that need more shape.
- Direct connections for SQL that does not fit either path.

Canyon is asynchronous and supports PostgreSQL, MySQL, and SQL Server. You can enable more than one backend and configure several named datasources. The first active datasource is the default; an explicit connection or datasource name lets an operation use another one.

## Finding your way through the book

Start by connecting to a database and defining a model. Then read and write rows, explore relationships, and build more expressive queries. The later chapters cover typed errors, direct SQL, and repository adapters. Examples use the Canyon-SQL 0.5.1 API and return `CanyonResult` so failures remain visible.

> **About migrations:** Canyon's `migrations` feature is experimental and incomplete. It is not a production schema-management system. The [migrations chapter](./the_migrations.md) explains its current boundary; the rest of the book assumes tables already exist.

> **New to async Rust?** The [Async Book](https://rust-lang.github.io/async-book/) is useful background. You need not know how Canyon's procedural macros are implemented, but their generated code is still Rust: traits must be in scope, types must match, and a database error is not an absent row.

Canyon-SQL is [MIT licensed](https://github.com/zerodaycode/Canyon-SQL/blob/main/LICENSE). The [source repository](https://github.com/zerodaycode/Canyon-SQL) and [book repository](https://github.com/zerodaycode/canyon-book) welcome corrections, especially when an example drifts from a tested API.
