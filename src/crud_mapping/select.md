# SELECT operations

The most basic usage that everyones understand when querying a database, it's a read operation.

By *"read operation"* we mean to ask the database for a specific set of data stored in some (or multiple)
tables.

What the database returns is some data type of type `Row`, that it's the internal implementation
of the libraries that we are wrapping, and, in the end, the responsable ones for handling the connection with the database at a lower level.

> Note: That Row type is just a generic name for a type that contains a database row with the data.
Every client library will have it's own type with a different identifier, but notice how isn't a relevant data. The important thing here it's the concept of Row.

Usually, that comes in the form of a collection (a `Vec<Row>`).
What `Canyon` will do it's to translate that lower-level data into your `Rust` type,
so, in the end, you will have some `T` or `Vec<T>`(where T it's your defined type, just by being correctly annotated. Macros will do all the job for you.

Did you remember the created type that will serve us for the examples from the past chapter?

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

Let's review what *SELECT* operations are automatically available now, focusing
on those who are well know in the `CRUD` world.

## Find all

This is the most widely-used query of all times! Please, database, give me all the data that you have
for a concrete table, including all the columns!

In `Canyon`, this is implemented as an associated function for your type `T`.

The `type::find_all()` associated function can be translated to a SQL `SELECT * FROM {table_name}`,
so we will end having a `Vec<T>` containing N instances of `T`, so, inside the vector is stored
the data of every database row!

We will write the following `Rust` code, considering our example type:

`let leagues: Result<Vec<League>, _> = League::find_all().await;`

That will bring us all the leagues inside the `league` table, in just one line of code!

## Unchecked alternatives

The are two more operations associated with the `find_all()` associated functions, that are the `unchecked` alternatives.

You already know that every database operation in `Canyon` returns as a `Result<T, E>`. But sometimes, when you are debugging or prototyping software, we want to access results without not too much complication. `Canyon` offers:

- `T::find_all_unchecked()`
- `T::find_all_unchecked_datasource()`

where both returns directly a `Vec<T>`. Take in consideration that, if something fails in the connection with the database, the program will panic. But, if you need to quickly profile some software design, those two are provided just in case.

## Find by PK

Annoter common data read pattern, it's to find a record by it's `id`. Almost always, every table contains
a column called `id`, which it's primary responsability it's to identify every record in the table with an `unique identifier`. This is achieved by making an auto-increment of the `id` for every new record, so when you write some new data, a new generated `id` it's associated with that data, making that new row unique because the `id` will always be different! So this deserve a named operation within `Canyon`.

> Note: Other patterns for make unique keys exists, but are out of the scope for this tutorial
. Don't worry, we will review in a latter chapter how Canyon deals with this concept.

We will write the following `Rust` code, considering our example type:

```rust
let league: Result<Option<League>, _> = League::find_by_id(&1).await;
```

Note the reference on the function argument. The find by primary key associated functions
must receive a reference in order to work with its argument of type `QueryParameters<'a>`
