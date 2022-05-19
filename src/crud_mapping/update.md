# UPDATE operations

Update operations consists in alter the values written in the database for a new ones.

`Canyon` provides the developer a really easy way to perform operations over a concrete
instance of some `T` type.

Let's refresh our `lec` instance for the `League` entity:
```
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

Notice how our `lec` instance is now `mut`. We are modifying the value by accesing
directly the declared `pub` field. We can avoid this by generating `getters` and
`setters` to provide a more `object-oriented` feel, but it serves to our purposes
right now.

To update the new modified data into the already existing record in the database,
we jus need to write:
`lec.update().await;`

Again, just one line of code to performn such an action! It's true that every
column has been updated in the process (even the ones that were not modified, are rewriting
the same value), but for just one row, the relation between `effort-reward` 
it's more than worth it!

Obviously, we can modify every field that we want and perform an `update` operation,
and every new data will be updated in the corresponding database row in it's
corresponding column.
