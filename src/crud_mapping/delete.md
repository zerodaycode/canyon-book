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

If the record don't exists in the database, it will fail performing the operation,
and an error it's returned. 
If we try to delete our hand written instance, it will fails because if you remember,
it has a default vale for the `id` field.

So, what's going on?

We must query the database first with a `SELECT` operation, and retrieve an instance
that represents one record.
If we want to delete that record, we can performn a `DELETE` operation over the instance
with the data in such an easy way like this:
`lec.delete().await;`

`Canyon` will delete that row from the database where the id matches our instance id.

It just simply makes no sence to create a random instance and try to perform a `DELETE`
operation on it, because only exists in our code.

If we go the way where we insert that record, we must query the database to retrieve
an instance with the correct `id` (the real one), so always take care with this
kind of action whenever you're working with `Canyon`.