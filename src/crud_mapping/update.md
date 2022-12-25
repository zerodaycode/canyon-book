# UPDATE operations

Update operations consists in alter the values written in the database for a new ones.

`Canyon` provides the developer a really easy way to perform operations over a concrete instance of some `T` type.

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

Imagine that we need to update our `image_url` field. We will write something like:
`lec.image_url = "https://new_lec_url.eu".to_string()`

We are modifying the value by accesing directly the declared `pub` field. We can avoid this by generating `getters` and `setters` to provide a more `object-oriented` feel, but it serves to our purposes right now.

To update the new modified data into the already existing record in the database, we just need to write:
`lec.update().await;  // Unused result`

Again, just one line of code to performn such an action!
Obviously, we can modify every field that we want and perform an `update` operation, and every new data will be updated in the corresponding database row in it's corresponding column.

- Remeber that you have the `_datasource(datasource_name: &str)` alternatives.
- Remember that, if you don't provide a `#[primary_key]` operation, the *update* methods won't be generated for your type!
