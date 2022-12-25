# DELETE operations

Delete operations consists in removing rows of data from the database.

`Canyon` provides an instance method of your `T` type to delete one concrete record at a time.

Let's refresh our `lec` instance for the `League` entity:

```rust
let mut lec: League = League::new(
    134524353253, 
    "LEC".to_string(),
    "League Europe Champions".to_string(),
    "EU West".to_string(),
    image_url: "https://lec.eu".to_string()
);
```

So, the `delete` method works by getting the `primary_key` field and make a query based on `DELETE FROM table_name WHERE table_name.<pk_name> = value`.

`Canyon` will delete that row from the database where the id matches our instance id.

In summary, to delete one record of the database, that record must exists and being mapped to an instance. If the record for that row was already deleted, any operation on the database is perfomed, but the query is executed.

- Remeber that you have the `_datasource(datasource_name: &str)` alternatives.
- Remember that, if you don't provide a `#[primary_key]` operation, the *delete* methods won't be generated for your type!
