# Introduction

Most applications do not struggle with one SQL query. They struggle with the repetition around a hundred of them: opening connections, naming the same columns again, translating rows into Rust values, and keeping those pieces in step as the schema changes. Canyon-SQL takes that repetitive work and generates it from the model you write.

An entity in Canyon is an ordinary Rust struct annotated with its database identity. Derives add row mapping, CRUD operations, and typed field names. When the generated operations are too narrow, a query builder lets you compose SQL while keeping values separate from the statement. You can also use a connection directly for SQL that does not fit either path.

Canyon is asynchronous and currently supports PostgreSQL, MySQL, and SQL Server. A project may enable more than one backend and configure several named datasources. The first active datasource is the default; an explicit connection or datasource name lets an operation use another one.

This book follows the route an application usually takes: connect, define a model, read and write data, then reach for relationships and more expressive queries. Later chapters cover typed errors, direct SQL, and repository adapters. The examples use the current API of Canyon-SQL 0.5.1 and return `CanyonResult` rather than hiding failures with `unwrap()`.

> Canyon's `migrations` feature is experimental and incomplete. It is not a production schema-management system. The [migrations chapter](./the_migrations.md) explains the present boundary; the rest of the book assumes tables already exist.

If you are new to Rust's asynchronous code, the [Async Book](https://rust-lang.github.io/async-book/) is useful background. You do not need to know how Canyon's procedural macros are implemented to use them, but it helps to remember that their generated code is still Rust: traits must be in scope, types must match, and a database error is not the same as an absent row.

Canyon-SQL is [MIT licensed](https://github.com/zerodaycode/Canyon-SQL/blob/main/LICENSE). The [source repository](https://github.com/zerodaycode/Canyon-SQL) and [book repository](https://github.com/zerodaycode/canyon-book) welcome corrections, especially when an example drifts from a tested API.
