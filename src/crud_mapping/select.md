# SELECT operations

The most basic usage that everyones understand when querying a database, it's a read operation.

By *"read operation"* we mean to ask the database for a specific set of data stored in some (or multiple)
tables.

What the database returns is some data type of type `Row`, that it's the internal implementation
of the libraries that, in the end, are handling the connection with the database at a lower level.

```
NOTE: That Row type is just a generic name for a type that contains a database row with the data.
Every client library will have it's own type with a different identifier, but notice how isn't a
relevant data. The important thing here it's the concept of Row
```

Usually, that comes in the form of a collection (a `Vec<Row>`), or just like a `Row`. It will depends
on the specific query, but that's not the relevant part.
What `Canyon` will do it's to translate that lower-level data into your `Rust` type, 
so, in the end, you will have some `T` or `Vec<T>`(where T it's your defined type, just by being
correctly annotated. Macros will do all the job for you.

Did you remember the created type that will serve us for the examples from the past chapter?

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

Let's review what *SELECT* operations are automatically available now, focusing
on those who are well know in the `CRUD` world.


# Find all

This is the most widely-used query of all times! Please, database, give me all the data that you have
for a concrete table, including all the columns!

In `Canyon`, this is implemented as an associated function for your type `T`.

The `type::find_all()` associated function can be translated to a SQL `SELECT * FROM {table_name}`,
so we will end having a `Vec<T>` being every `T` inside the vector, an instance of your type with
the data of one database row inside it!

We will write the following `Rust` code, considering our example type:

`let leagues: Vec<League> = League::find_all().await;`

That will bring us all the leagues inside the `league` table, in just one line of code!

`NOTE: If you are struggling with the `.await` keyword, review the intro chapter of this block`


# Find by ID

Annoted common data read pattern, it's to find a record by it's `ID`. Almost always, every table contains
a column called `ID`, which it's primary responsability it's to identify every record in the table
with an `unique identifier`. This is achieved by making an auto-increment of the `ID` for every new
record, so when you write some new data, a new generated `ID` it's associated with that data, making
that new row unique because the `ID` will always be different!

```
NOTE: Other patterns for make unique keys exists, but are out of the scope for this tutorial,
and in general, for the whole Canyon framework. Don't worry, we will review in more advanced
chapter how Canyon deals with this concept.
```

We will write the following `Rust` code, considering our example type:

```
let some_int_id: IntegralNumber = 1;
let league: Option<League> = League::find_by_id(some_int_id).await;
```

See the function's signature 
```
async fn __find_by_id<N>(table_name: &str, id: N) -> DatabaseResult<T> 
        where N: IntegralNumber;
```

and the definition for the `IntegralNumber` trait bound is the following:
```
pub trait IntegralNumber: ToSql + Sync + Send {}
```

This is telling us, that Rust will be able to accept as parameter any thing that
also it's bounded to `ToSql` + `Sync` + `Send` (this both two are required to safely
send data through a Future<T>).
But, `Canyon` only implements `IntegralNumber` for the following two:
- `impl IntegralNumber for i32 {}`
- `impl IntegralNumber for i64 {}`

This was made because most client libraries thinks about `PRIMARY KEYS` as `i64`, 
and it's a little more clear just to play with `i32`, unless we have trillions of records 
in the database (or not records, but the ID value was a very high number) or use hashes 
instead of raw integers...

In summary, we can pass in as argument to the find by id, a *signed 32 bits integer*,
or *a signed 64 bits integer*.

