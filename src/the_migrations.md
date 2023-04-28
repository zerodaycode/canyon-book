# Canyon Migrations: The Final Boss

> *Note: Please be advised that this feature is still unstable. For further information, please refer to the [disclaimer section](#disclaimer) a the end of this document.*

## Index

- [Canyon Migrations: The Final Boss](#canyon-migrations-the-final-boss)
  - [Index](#index)
  - [Introduction](#introduction)
  - [Only one entity per `.rs` file](#only-one-entity-per-rs-file)
  - [The #\[canyon\] annotation](#the-canyon-annotation)
    - [The 'full mode' concept](#the-full-mode-concept)
  - [The migrations](#the-migrations)
  - [Disclaimer](#disclaimer)

## Introduction

As a modern ORM framework, `Canyon` aims to provide a comprehensive solution for managing all relations with the database in a manner that is agnostic to the developer. This concept is commonly referred to as "migrations". Migrations are a way for the framework to take full responsibility for managing every aspect of the database, including creating and dropping tables, altering their names, columns, column names, column types, generating `GRANT` permissions, etc.

However, unlike other ORM frameworks, `Canyon` proposes a new way of handling migrations. Other frameworks come with a built-in command tool or require a separate command tool to be downloaded, installed, and managed. `Canyon`, on the other hand, manages everything at compile time and executes the migrations at the beginning of the client's code. 

> It should be noted that this feature must be enabled, as it is not active by default.
 
## Only one entity per `.rs` file
[(back to top)](#canyon-migrations-the-final-boss)

To enable the framework to manage the entity types, they must be annotated with the `#[canyon_entity]` macro, which was introduced in the [Entities chapter](./canyon_entities.md).

```rust
#[canyon_entity]  // ready to be tracked and managed by Canyon
pub struct League { /* fields... */ }
```

It is important to remember that only one annotated entity with `#[canyon_entity]` is allowed per file due to the way migrations work.  Attempting to implement more than one entity in the same file will result in a compiler error.

## The #[canyon] annotation
[(back to top)](#canyon-migrations-the-final-boss)

As is typical with components of a modern ORM framework. `Canyon` utilizes annotations to drive its behavior. This means that when needing something, `Canyon` most likely has an annotation to solve it.

In order to enable `migrations`, include the following annotation for the `main()` function.

```rust
#[canyon]
fn main() { 
    /* code in main */ 
}
```
*Canyon's #[canyon] annotation. It unlocks the 'full mode'*

###  The 'full mode' concept
[(back to top)](#canyon-migrations-the-final-boss)

When discussing about the *framework*, the term 'full mode' is used to indicate that `Canyon` has activated all of its features to assume complete control over everything related to the database in your program.

## The migrations
[(back to top)](#canyon-migrations-the-final-boss)

When the full mode is enabled, `Canyon` takes care of managing the migrations for the complete lifecycle of the program. Migrations are designed to manage every declared entity during compile time. Thanks to the complex macro system, it will generate all the necessary queries to be applied at the start of the application.

> Note: Currently, we are making efforts to ensure that the migration process is only executed when `cargo build` or `cargo run` are invoked. The execution of migrations by code static analyzers, which make use of the `cargo check` functionality, can lead to obscure feedback.

`Canyon` searches for structs annotated with `canyon_entity`, then checks the database information provided in the `canyon.toml` file. It generates everything necessary for the entities in the database.

During compilation, `Canyon` searches for new entities, entities that have been removed from your code (which will then be deleted from the database), entities that have been renamed, entity fields that have been renamed or changed type, and any new or removed `field annotation` for an entity.

When it is still compiling, `Canyon` generates the necessary queries for the database. Even though this is the most complicated feature of `Canyon`, there is not much more explanation about how it affects development. As mentioned earlier, the idea behind `Canyon` is simple:

*Make developer's life easier*

## Disclaimer
[(back to top)](#canyon-migrations-the-final-boss)

As stated at the beginning of this document, `Canyon` migrations are still an unstable feature under development.

There is still some work to be done to ensure they function as intended, and some of the features mentioned are not yet available in the `0.1.0` release.

Many functionalities are already implemented, such as changing table names, creating, dropping, altering tables, modifying sequences or identities, changing column data types. However, renaming columns is not yet possible.

Furthermore, while some of these functionalities are ready for use with `PostgreSQL` databases, they may not be available for `MSSQL` databases, if `Canyon` encounters an action that requires processing from Microsoft's database engine, it will raise a `todo!()` panic. To avoid such situations, it is possible to disable migrations for a specific datasource in the configuration file.
