# CRUD and Mapping Operations

- [CRUD and Mapping Operations](#crud-and-mapping-operations)
  - [Introduction](#introduction)
  - [The `CanyonCrud` and `CanyonMapper` derive macros](#the-canyoncrud-and-canyonmapper-derive-macros)
  - [The asynchronous behavior of `Canyon`](#the-asynchronous-behavior-of-canyon)
  - [The `datasource` concept](#the-datasource-concept)
    - [Specifying Datasources](#specifying-datasources)
    - [Default Datasources](#default-datasources)
  - [Next steps](#next-steps)


## Introduction
[(back to top)](#introduction)

The primary purpose of `Canyon` is to facilitate database querying and retrieval of data. Writing raw SQL statements every time data is needed can be tedious, error-prone, and challenging to debug. Therefore, `Canyon` provides a solution to make the developer's life easier by automatically generating the desired queries.

As described in previous chapter, the macros `#[canyon_entity]` and `#[primary_key]` describe an entity and the primary key, respectively. In Canyon's common language, an entity represents a Rust type that models a real world object. It is a term used to refer to a type that maps against a database table. `Canyon` recognizes this and generate an implementation block with methods and associated functions to allow the type to communicate with the database. 

Consider the following Rust type declaration:

```rust
#[canyon_entity]
pub struct League {
    #[primary_key]
    pub id: i32,
    pub ext_id: i64,
    pub slug: String,
    pub name: String,
    pub region: String,
    pub image_url: String
}
```

This struct represents the data of a professional competitive league from the `League of Legends` MOBA obtained from the `LolEsports API`. Suppose there is a web application that requests this information. The first thing that can be done is store it in the database and then serve it to the main application. Which could be, for instance, a mobile front-end application.

Methods will have to be implemented for saving, retrieving, and updating data...

To accomplish these tasks, `Canyon` will support the development by reducing the extensive amount of Rust code required to perform such database operations.

## The `CanyonCrud` and `CanyonMapper` derive macros
[(back to top)](#introduction)

Canyon provides two powerful macros to simplify the developer's work:

```rust
#[derive(CanyonCrud, CanyonMapper)]
```

These two derive macros work together and are essential for communicating with the database. They will implement the following traits on the type:

- `CanyonCrud` is responsible for generating the CRUD methods for interacting with the database;
- `CanyonMapper` is responsible for serializing/deserializing the data as it sends/receives messages with the database;

```rust
#[derive(CanyonCrud, CanyonMapper)]
#[canyon_entity]
pub struct League {
    #[primary_key]
    pub id: i32,
    pub ext_id: i64,
    pub slug: String,
    pub name: String,
    pub region: String,
    pub image_url: String
}
```

## The asynchronous behavior of `Canyon`
[(back to top)](#introduction)

It should be noted that every `Canyon` action requires the usage of a `.await` expression. This is because every operation in `Canyon` is designed to be asynchronous, which means that the result must be awaited in order to be obtained.

It is also important to remember that from this point forward, every action with `Canyon` that leads to  the execution of a database query must be followed by the `await` instruction. This is because futures are *lazy by default*, which means that they will *not be consumed unless they are awaited*.

If you are not familiar with the concept of asynchronous programming, we recommend taking a look at [The Rust async book](https://rust-lang.github.io/async-book/03_async_await/01_chapter.html). This resource can provide valuable insight about asynchronous programming.

## The `datasource` concept
[(back to top)](#introduction)

Before we proceed, let's discuss the concept of `datasource` in Canyon.

A `datasource` in Canyon refers to the channel that connects the framework with a specific database. A user may have multiple datasources, even if they use different database engines. With Canyon, you can query one database in a statement and then query a completely different one in the next statement.

### Specifying Datasources
[(back to top)](#introduction)

It is worth noting that all CRUD operations, whether they are a method from your type or an associated function, will have a duplicate ending with `_datasource`. For instance, the `::find_all()` operation will have a replica called `::find_all_datasource(datasource_name: &str)`. In this duplicate, `datasource_name` matches one of the datasource names defined in the configuration file.

### Default Datasources
[(back to top)](#introduction)

`Canyon` takes the first declared datasource as the default one. Therefore, you may find code that uses `::find_all()` without the need to specify a datasource, as `Canyon` will use the default directly in the queries.

To avoid duplication in the next chapters, it is essential to remember that there will be a `_datasource` alternative for every CRUD operation. This can be used to specify a datasource when the default is not the target.

## Next steps
[(back to top)](#introduction)

In this chapter, we have created a new type that is capable of querying the database and properly serializing/deserializing messages as needed.

However it is important to explore what Canyon is really capable of doing with these annotations. The following chapters will discuss more in-depth about each of the CRUD operations...