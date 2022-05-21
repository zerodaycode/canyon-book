# DELETE operations

Delete operations consists in removing rows of data from the database.

`Canyon` provides an instance method of your `T` type to delete one concrete 
record at a time.

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

If we didn't insert it already, let's go ahead:
`lec.insert().await;`

So, the `delete` method works by getting the `id` field and make a query based
on `DELETE FROM table_name WHERE table_name.id = id`.

`Canyon` will delete that row from the database where the id matches our instance id.

Note that if you try to make a `DELETE` operation before the `INSERT` one, it will
It just simply makes no sence to create a random instance and try to perform a `DELETE`
operation on it, because that data only exists in our code.

If we didn't insert the data of the instace, we could query the database to retrieve
an entity that exists in the database, to have real data that we can delete.

In summary, to delete one record of the database, that record must exists and 
being mapped to an instance.

Always take care with this kind of action whenever you're working with `Canyon`.