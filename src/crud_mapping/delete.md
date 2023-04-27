# DELETE operations

Delete operations consists in removing rows of data from the database.

With `Canyon`, developers can delete a single record at a time using an instance method of the given `T` type that was properly set up as an entity.

Once again, using the `League` entity:

```rust
let mut lec: League = League::new(
    134524353253, 
    "LEC".to_string(),
    "League Europe Champions".to_string(),
    "EU West".to_string(),
    image_url: "https://lec.eu".to_string()
);
```

The existing entry can be deleted from the database by running:

```rust
lec.delete();
```

The `delete` method will run a query similar to `DELETE FROM table_name WHERE table_name.<pk_name> = value`, where `pk_name` and `value` comes from the `#[primary_key]` set on the type declaration for `League`. `Canyon` will delete the row from the database where the id matches the instance id.

In summary, to delete a record from the database using `Canyon`, the record must exist and be mapped to an instance. If the record for that row doesn't exist, the query will still be executed on the database.

> Note: Don't forget about using `_datasources` methods when not using the default `datasource`.

> Note: If a `#[primary_key]` does not exist on the type declaration, the *delete* methods for it will not be generated.