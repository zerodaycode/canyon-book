# The final boss. Canyon migrations

*NOTE: Please, take in mind that this feature it's still unstable. Read the last section*
*of this document (the # Disclaimer one) to get more information about this.*

As a modern ORM framework, `Canyon` must implement something that completly manages all the
relations with the database developer-agnostic.

That concept has a term probably familiar to the most of you. `Migrations`. But, for the ones
who don't know about it, it's a way to say that the framework it's the ultimate responsable
for handle every single aspect managing the database. This includes the creation and the dropping
of the tables, alter it's name, columns, columns name, columns types, generates `GRANT` permissions,
and the list goes on.

But, unlike other `ORM's`, `Canyon` proposes a new way of handling the migrations for you.

Typically, others comes with a built in cmd tool to manage it. Others has a separate cmd tool
for it that you must download, install and manage.

What `Canyon` proposes for the `migrations` concept it's new. Everything it's managed by
`Canyon` itself at compile time, and executed just in the beginning of the client's code.
But this is a feature that must be enabled, it does not comes active by default.

## The #[canyon] annotation

As always, the behaviour of the components of the framework are annotation-driven. This means,
if you need something, probably `Canyon` has an annotation that makes the hard work for you.

But, how `Canyon` knows that he must enable the `migrations` feature, and starting to take
control of whatever action is necessary in your program?

Simple answer. A new annotation. That's probably the most special one, and it must be placed
in the `Rust` more special part of a program. The `main()` function.

```rust
#[canyon]
fn main() { 
    /* code in main */ 
}
```
*Canyon's #[canyon] annotation, that unlocks the 'full mode'*


## The 'full mode' concept

When talking about the *framework*, the `full mode` words are used to describe that `Canyon`
has enabled all of his features to take the control about everything database related in
your program.

## The migrations

`Canyon` handles the migrations when the *full mode* it's enabled. For the complete lifecycle
of your program, it will take care about everything... for the full lifecycle? Wow, wow...
wait a moment... it will `Canyon` interfer at runtime on my program?

Respecting to the migrations, (obviously, your queries are executed at runtime) the answer is...
just as the beginning (for now!).

> Note: We are trying to look for a solution that this process is only executed when `cargo build` or
`cargo run` are invoked. Code static analizers makes use of the `cargo check` functionality, which
leads to the execution of the migrations in a obscure way, due that we don't receive any feedback.

`Canyon` migrations re designed to manage every entity for you at compile time, and (for now) generate
the necessary queries for apply at the start of your application.
This it's done via complex macro operations that you don't have yo worry about. It will be done for you.

But, for this to works, there's two extra requisites required.
The structs that you want that `Canyon` manages, must be annotated with something that we've already
learn in the *Canyon Entities* chapter, the `#[canyon_entity]` macro.

```rust
#[canyon_entity]  // ready to be tracked and managed by Canyon
pub struct League { /* fields... */ }
```

- You are only allowed to have one `Canyon Entity` per `.rs` source file. So, whenever you're
defining a new entity, please, don't implement more that one in the same source file, or the
compiler will refuses to build your program.

When the *full mode* it's enabled, `Canyon` will look for your `canyon_entity` annotated structs.
That's when your type is really an entity in `Canyon`!

Then, it will check your provided database info in your `canyon.toml` file, and it will generate
everything that is necessary for that entities in the database.

Everytime that you compile your program, `Canyon` performs a quick search to determine if there
are new entities, if an entity was deleted from your code (this will delete it in your database!),
if an entity was renamed, if an entity field was renamed or changed it's type, and finally, if
an entity has a new `field annotation` or a pre-existent one was removed from your code.

Then, when it's still compiling, `Canyon` will generate the queries to the database.

And that's all. Even if it's by far the most complicated feature of `Canyon`, there's no
much more explanation about how affects you when are developing, and that's because,
as already mentioned before, the idea of `Canyon` it's pretty simple.

*Make the developer's life easier*

## Disclaimer

As you read at the beggining of this file, `Canyon` migrations are still an unstable and a baby feature.

That's still code to rework in order to make them work as intended, and there's still
mentioned features that aren't available for the `0.1.0` release.

There are already a lot of things implemented, like change the table name, create tables, drop tables,
changes columns datatypes, drop and add constraints, modify (create, alter or delete) sequences or identities,
but no alter a column name.

Also, some of the mentioned above are ready for `PostgreSQL` databases, but no for `MSSQL` ones.
If `Canyon` encounters some action after detecting changes in the entities that should be processed
for the Microsoft's database engine, a `todo!()` panic will be raised.

Remember that you are able to disable migrations for a whole datasource in the configuration file,
to avoid those kind of situations.

Also please, remember the limitation of having one annotated entity with `#[canyon_entity]`
per file.
This is a restriction imposed because how the migrations works, and what `Canyon`
needs in order to know and relate the data for every entity managed in your program,
compilation after compilation, because, if they are modified, `Canyon` has to
update them in the database to make everything works correctly.
