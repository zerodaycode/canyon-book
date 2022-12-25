# CRUD and mapping operations

The main goal of Canyon, apart from our slogan of `*Make the developer's life easier*`, it's just to query databases, and retrive back data from them.

But we know that it's a little bit dissapointing and tedious to write raw `SQL` statements every time that you need to fetch some data from some entity, being error prone, tedious and hard to debug.

That's were come the first `Canyon` solution to make your life easier. Create queries for you.

You already know about the `#[canyon_entity]` macro, the `#[primary_key]` attribute...
`Canyon` it's able to recognize a Rust struct that it's candidate to model some database table.
So it will wire into your Rust object an impl block with nice and neat methods and associated functions to make your type able to communicate with the database, and futhermore, `Canyon` it's designed to automatically translate the result of the query
into elements of some type T whenever that type implements some specific derive traits.

> Note: In Canyon's common language, an entity represents a Rust struct that models a real world object. This is a common term also used when you want to refer to some type that maps against a database table

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

This struct represents the data of some League of the profesional competitive league from the `League of Legends` MOBA, obtained from the `LolEsports API`.

Suppose now, that you have a web application that requests that information, and the first thing that you want to do it's store it in your database, and later serve it to another application that you want to build with that data, for example, a mobile front-end application.

So, you need a mechanism to save your data, read and write data, update data...

How can `Canyon` can help you to reduce the incredible larger amount of *Rust* code that we usually need when we need to do operations against a database?

## The CanyonCrud and CanyonMapper derive macros

Canyon has two powerful macros to the service of the developer that will do all the hard job for him.

`#[derive(CanyonCrud, CanyonMapper)]`

That two derive macro annotations are like one, because one *can't* work without the other.
The fact of being sepparated is an implementation design decision, because we like to read from the name of that macros that our types are not only capable of communicate with the database, also they are able to deserialize the results
into Rust readable data automatically for us!

Now, rewrite our struct with the necessary annotations to make it work!

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

## The asyncronous behaviour of Canyon

You will find that every Canyon action, will always be followed by an `.await` expresion.
Every operation in Canyon it's an async operation, due to the inheret design of the client libraries, so, for obtaining the result, it must be awaited.

Remember from here until the end, that every Canyon's usage that leads to a database query execution must be followed by the await keyword. That's because *futures* are lazy by default, meaning that if they're not awaited, they're not consumed!

If you are struggling with this concept, you can take a look to the following link of
[The Rust async book](https://rust-lang.github.io/async-book/03_async_await/01_chapter.html),
or if this a new concept for you, consider take yourself a time to read it and comprehend it.

## The `datasource` concept

One thing that we want to discuss before we get our hands on job, it's de concept of `datasource`.

Datasources in `Canyon` may be see as the channel that connects `Canyon` with a specific database. And you can have multiple ones, even to different database engines! You will be able to query one database in a statement, and query a completly different one in the next statement.

> Note: All the CRUD operations (being a method of your type or an associated function) that will be presented in the next chapters will have a duplicate of themselves ending with `_datasource`. For example, the `::find_all()` operation
will have a replica as `::find_all_datasource(datasource_name: &str)`, where you can pass a string slice that matches one of the datasource names defined in the configuration file.

> Note: Canyon will take the first declared datasource as the default one. That's why exists code like `::find_all()`, without no need to specify a datasource, Canyon will use that one directly on the queries.

For avoid duplication in the next chapters, remember, you will have an `_datasource` alternative to every CRUD operation where you can specify a concrete datasource when you need to use more than one of a different one of the defaulted.

## In summary

We create a new type that it's able to query the database, and map the data retrieved into Rust objects automatically!

But now we need to know what Canyon it's really capable of do with that annotations!

Let's explore it in the next subchapters!
