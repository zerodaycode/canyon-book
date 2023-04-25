# INSERT operations

Now it is time to write data into the database.

Canyon provides a convenient way to insert data directly from an instantiated object.

Remember this?

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

Let's create a `League` instance:

```rust
let mut lec: League = League {
    id: Default::default(),
    ext_id: 134524353253,
    slug: "LEC".to_string(),
    name: "League Europe Champions".to_string(),
    region: "EU West".to_string(),
    image_url: "https://lec.eu".to_string(),
};
```

Notice how the `id` is `Default::default()`. For the type i32, this just means `0`.

Now, with that value, we will call the following instruction:

```rust
lec.insert().await;
```

And that record is now inserted into the database!

But note one thing: the `lec` instance was declared as mutable. Why?

It is important to remember that the `insert` method from `Canyon` will automatically update the `self.id` field. The previous `id` was replaced by the newly generated ID after the insert.

Thanks to it, the `insert` method is a really convenient method for writing data into the database. For example, a common scenario is when you make a call to an external service and parse the data into a new instance of your Rust type. With this you can insert it directly in a single expression!

## The new associated function pattern

A common alternative to writing better and more maintainable code is to use the convention of "constructors" in Rust.

Although there is no exact concept of constructors in the language, a convention has been established for creating new instances of a given type `T`.

This convention is to write an associated function `::new(params)` in the `impl` block for initializing objects of the given type. Through this approach, an initializer may be implemented as follows:

```rust
impl League {
    // Notice how id is not included in the parameters
    pub fn new(
        ext_id: i64,
        slug: String,
        name: String,
        region: String,
        image_url: String
    ) -> Self {
        Self {
            id: Default::default(),
            ext_id: ext_id,
            slug: slug,
            name: name,
            region: region,
            image_url: image_url
        }
    }
}
```

With this in mind, the previous example can be written and inserted into the database as follows:

```rust
// Create new instance
let lec: League = League::new(
    134524353253,
    "LEC".to_string(),
    "League Europe Champions".to_string(),
    "EU West".to_string(),
    image_url: "https://lec.eu".to_string()
);

// Insert new instance into the database
lec.insert().await;
```

The `insert` instruction returns a `Result<(), Err>` type. For simplicity, we are omiting handling results. For more information on the topic, see in [The Official Book](https://doc.rust-lang.org/book/ch09-00-error-handling.html).

## Performing Multiple Inserts

In addition to allowing the insertion of individual entities, `Canyon` also enables users to insert multiple entities with a single operation. The multi-insert feature is implemented as an associated function rather than a method of the instantiated object.

You can pass instances of your type as a reference to a raw Rust array, and then await the result. The values generated for fields declared as primary keys will then be assigned to the corresponding field of each instance.

The syntax for the multi-insert feature is shown below:

```rust
/// Demonstration on how to perform an insert of multiple items on a table
async fn _multi_insert_example() {
    // Create an instance
    let new_league = League {
        id: Default::default(),
        ext_id: 392489032,
        slug: "DKC".to_owned(),
        name: "Denmark Competitive".to_owned(),
        region: "Denmark".to_owned(),
        image_url: "https://www.an_url.com".to_owned()
    };
    // Create a second instance
    let new_league2 = League {
        id: Default::default(),
        ext_id: 392489032,
        slug: "IreL".to_owned(),
        name: "Ireland ERL".to_owned(),
        region: "Ireland".to_owned(),
        image_url: "https://www.yet_another_url.com".to_owned()
    };
    // Create a third instance
    let new_league3 = League {
        id: Default::default(),
        ext_id: 9687392489032,
        slug: "LEC".to_owned(),
        name: "League Europe Champions".to_owned(),
        region: "EU".to_owned(),
        image_url: "https://www.lag.com".to_owned()
    };

    // Unused Result<T, E>
    // Insert all three instances in a single transaction
    League::insert_into(
        &[new_league, new_league2, new_league3]
    ).await;
}
```

The `insert_into` instruction returns a result with an array of the updated values that were inserted.

## Notes on the `#[primary_key]` annotation

It is important to note that if the database has a primary key, the `#[primary_key]` annotation is mandatory. By default, `Canyon` assumes that the column holding the primary key has a sequence or similar database concept and will omit the `PK` value on the insert operation. This will result in the database generating a unique and auto-incremental value.

However, you can still insert into tables that do not have any columns marked as primary key. In such cases, you should not declare a `#[primary_key]` in your entity.

Therefore, it is important to include the `#[primary_key]` annotation when performing insert operations. If a table's column has a primary key and `Canyon` does not find the `#[primary_key]` annotation, it will serialize the value to insert it, which is an illegal or incomplete operation.

It is also worth noting that `_datasource(datasource_name: &str)` alternatives are available, and if you do not provide a `#[primary_key]` annotation, the insert methods will not be generated for that type. However, the associated function `T::multi_insert()` will still be available.