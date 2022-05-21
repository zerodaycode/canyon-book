# Production ready operations

Every operation that we've seen in `Canyon` it's nice and neat. Everything works with a 
few lines of code, letting you focus in write your solution or your application, not fighting
against the persistance layer.

But, onw thing that some of you must noticed is that those operations *might* fall.

When querying a database, multiple things can go wrong (connection, parameters, timeouts...)
and our query could resolve as a failure, making or `Rust` code to `panic!`, and 
interrupting it's execution.

`Canyon` already provides you an `out-of-the-bounds` solution for every one of this cases.
The solution it's really simple. Everything that we've seen, it's wrapped in a `Result<T, E>`
`Rust` enum type, indicating that exists the posibility that the operation be a failure.

When you want to quickly prototype your application, at the development stage, or for
some reason you are a crazy dev that always it's pretty sure that your code don't fail,
or don't care about panics, the already commented versions are ok.

But, when you're deploying your app into a more profesional environment, a million of bad things
usually happen, and you must take care about every one in order to make your app running as
intended.

In `Canyon` they are known as `result` operations, and has exactly the same identifier
but ending with an `_result`.

So, for not repeat again all the operations, we will summarice them defining it's signatures.
If you have doubts on how to handling the `Result`, you may want to look at some examples
already written [here](https://github.com/zerodaycode/Canyon-SQL/blob/development/canyon_examples/src/main.rs)
The rest will be up to you!

```
NOTE: Making a type alias for the canyon_sql::tokio_postgres::Error

type QueryError = canyon_sql::tokio_postgres::Error;
```

*NOTE: Instance methods are represented by t., while associated functions are represented as T::*

## SELECT

- T::find_all_result() -> Result<Vec<T>, QueryError>
- T::find_by_id_result(id: i32) -> Result<Option<T>, QueryError>
- T::count_result() -> Result<i32, QueryError>

## INSERT

- t.insert_result(&mut self) -> Result<(), QueryError>

## UPDATE

- t.update_result(&self) -> Result<(), QueryError>

## DELETE

- t.delete_result(&self) -> Result<(), QueryError>