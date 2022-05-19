# CRUD and mapping operations

The main goal of Canyon, apart from our slogan of `Make the developer's life easier', it's query a database.
When queryng, you usually recover your desired data when a query it's performed.

But we know that it's a little bit dissapointing and tedious to write raw `SQL` statements every time that you need
to fetch some data from some entity, being error prone to write a bad statement and have to recompile again and
again your program.

That's were come the first `Canyon` solution to make your life easier. Create queries for you.

`Canyon` it's able to recognize a Rust struct that it's candidate to model some database table.
So it will wire into your Rust object an impl block with nice and neat methods and associated
functions to allow to that object to communicate with the database, and not only that, 
Canyon it's designed to automatically translate the result of the query
into elements of some type T whenever that type implements some specific derive traits.


```
NOTE: In Canyon's common language, an entity represents a Rust struct that fits the content on some database table,
or in a raw sense, an entity represents some structure that models a real world object
```

Consider the following Rust type declaration:

```
pub struct League {
    pub id: i32,
    pub ext_id: i64,
    pub slug: String,
    pub name: String,
    pub region: String,
    pub image_url: String
}
```

This struct represents the data of some League of the profesional competitive league from the `League of Legends` MOBA,
obtained from the `LolEsports API`.

Suppose now, that you have a web application that requests that information, and the first thing that you want to do it's 
store it in your database, to serve it to another application that you want to build with that data.

So, you need a mechanism to save your data, read and write data, update data...

How can Canyon help us to reduce the indecent amount of Rust code that we usually need when we need to do operations
against a database?


## The CanyonCrud and CanyonMapper derive macros

Canyon has two powerful macros to the service of the developer that will do all the hard job for him.

`#[derive(CanyonCrud, CanyonMapper)]`

That two derive macro annotations are like one, because one *can't* work without the other.
That's a implementation design decision, because we like to specify to our types that we are making
them not only capable of communicate with the database, also they are able to deserialize the results
into Rust readable data automatically for us!

Now, rewrite our struct with the necessary annotations to make it work!

```
#[derive(Debug, Clone, CanyonCrud, CanyonMapper)]
pub struct League {
    pub id: i32,
    pub ext_id: i64,
    pub slug: String,
    pub name: String,
    pub region: String,
    pub image_url: String
}
```

```
NOTE: Notice how Canyon also need to make the type debuggable and clonable. That's also a design
decision for more complicated technical reasons than the scope of this tutorial when the code 
behind the scenes was written, involving several trait bounds to make the things work properly.
```

## The asyncronous behaviour of Canyon

You will find that every Canyon action, will always be followed by an `.await` expresion.
Every operation in Canyon it's an async operation, due to the inheret design of the client 
libraries, so, for obtaining the result, this must be awaited.

Remember from here until the end, that every Canyon's usage must be followed by the await
keyword. That's because the futures are lazy by default, meaning that if they're not awaited,
they're not consumed!

If you are struggling with this concept, please review the following link of
[The Rust async book](https://rust-lang.github.io/async-book/03_async_await/01_chapter.html),
or if this a concept new to you, take yourself a time to read it and comprehend it.


## In summary

We create a new type that it's able to query the database, and map the data retrieved into
Rust objects automatically!

But now we need to know what Canyon it's really capable of do with that annotations!

Let's explore it in the next subchapters!

