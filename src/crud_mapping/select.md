# SELECT operations

The most fundamental operation when querying a database is the `read` operation, which involves requesting a particular set of data stored in one or more tables.

When any of the CRUD operations are processed, the data received as response from the database will be of a generic type referred to as `Row`. `Row` is unique for each *client library*, but each of them refer to a single "row" retrieved from the database.

`Canyon`'s job is to then parse this retrieved response into `T` or `Vec<T>`, where `T` is your defined type. As long as the types are properly annotated, the users don't need to worry about parsing the data themselves.


Here is the type that was described in the previous chapter:

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


Let's review what *SELECT* operations are automatically available now. With focus on those that are well known in the `CRUD` world.

## `find_all`

One of the most commonly used queries when working with databases. In summary, it is the same as saying:

> "Please, database, give me all the data that you have for this table, including all of the columns!"

In `Canyon`, this method is available as an associated function for your defined type `T`. The `type::find_all()` method can be translated to a SQL query `SELECT * FROM {table_name}`, which will return a collection of `Row` instances.

The retrieved `Row` collection will then be automatically mapped into a `Vec<T>`, where `T` is the same type of the object where `type::find_all` was called.

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

We can retrieve all rows of the `league` table with the following line:

```rust
let leagues: Result<Vec<League>, _> = League::find_all().await;
```


## Unchecked alternatives

There are two more operations associated with the `find_all()` function that provide unchecked alternatives.

In `Canyon`, every database operation returns a `Result<T, E>`. However, during software development, debugging, or prototyping, it may be useful to access query results without additional complications. For this purpose, `Canyon` provides:


- `T::find_all_unchecked()`
- `T::find_all_unchecked_datasource()`

Both functions return a `Vec<T>` directly, bypassing the `Result` type. However, if there is any error during the database connection, the program will panic. Therefore, these functions are only recommended for quickly profiling or experimentation.


## Find by PK

Another common pattern for reading data is to find a record by its primary key (`PK`). 

Looking at the previous example `League` again:

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

The primary key in this case is `id`. Therefore, only one `League` row exists for every unique `id` value. The `auto-incrementing` parameter isn't set. So it **is enabled**. Check previous chapter for more information.

To find a record by its primary key, the method `type::find_by_id` can be used:

```rust
// Searching for a row that has id=1
let league: Result<Option<League>, _> = League::find_by_id(&1).await;
```

Note the reference on the function argument. The `find_by_id` associated function doesn't take ownership of the parameter.