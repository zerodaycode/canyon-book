# UPDATE operations

Update operations involve altering the values in the database with new ones. `Canyon` offer developers an easy way to perform operations on a specific instance of a given `T` type that was properly set up.

Considering the following `League` instance:

```rust
let mut lec: League = League::new(
    134524353253, 
    "LEC".to_string(),
    "League Europe Champions".to_string(),
    "EU West".to_string(),
    image_url: "https://lec.eu".to_string()
);
```

Suppose that the `image_url` field has to be modified. It can be done as follows:

```rust
// modify the field first
lec.image_url = "https://new_lec_url.eu".to_string();
// synchronize the changes with the database
lec.update().await; // unused result
```

`image_url` is a public field, so it can be modified at will. Some objects require validation before making changes. In such cases, usually the fields will be private. Although `getters` and `setters` should be also available for interacting with the fields.

When running the `update` instructions, an "update row request" with all the new fields will be made with the database. Synchronizing the new state with the database.

There is no need to call `update` more than once. Make all the changes that are required, call `update` once, then all the changes will be reflected to the database.

> Note: Don't forget about using `_datasources` methods when not using the default `datasource`.

> Note: If a `#[primary_key]` does not exist on the type declaration, the update methods for it will not be generated.
