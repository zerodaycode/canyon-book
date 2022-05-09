# The secrets.toml file

`Canyon` needs a very special file from there to work with, the `secrets.toml` file.
This is just a regular *.toml* file where store the *secrets* of the project, or, in another words,
the user credentials to be able to work with the database.


## The file fields

Until know, a `Canyon` project needs the following mandatory fields to work with:

```
username = <your_database_user_username_here>
password = <the_user's_password>
db_name = <the_database_name_that_you_want_to_work_with>
```

were you should put:
- your database user (or the admin one, but obviously, not recomended)
- the user's password
- the database you want to work with

were you must replace the angle brackets for *nothing* and put a raw identifier in the place of the
content that you're seeing inside those angle brackets, looking similar to the following lines:

```
username = my_username
password = some_secure_pass
db_name = my_database
```

where there's no need to enquote the literals inside string quotes.


## Remote host
This file has another field that its an optional one: `the host`

In simple terms, the `host ` it's refering to the server that it's hosting the database.
Usually, when you develop and work in your personal computer, or even when you deploy in production some kind of database, this is connected from within the machine. This kind of connection it's 
known as `localhost`, refering that the database is running an hosted in the same place that the 
code (or tool) that it's working with.

But sometimes, the database it's hosted on a remote server, that you must connect via SSL or whatever
kind of connection you desire, but isn't locally to the tool that it's invoking it. 

If this is the case, we can add a last property to our `secrets.toml` file to indicate
`Canyon` that it must modify the "connection chain" and specify the address of where it's 
the database that you want to work with. So, the final file will look similar to:

```
username = <your_database_user_username_here>
password = <the_user's_password>
db_name = <the_database_name_that_you_want_to_work_with>
host = <some_non_local_direction>
```


# Setting up the database credentials
Now that we are ready to go, we need to create this resources in the database. In a future release,
even if the database isn't configured to work with a new user and a new database, `Canyon` will be
able to handle those issues for you. 

But, for now, let's see how to configure the engine to properly being able to work with.