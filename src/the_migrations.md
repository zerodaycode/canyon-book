# The final boss. Canyon migrations.

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
`Canyon` itself at compile time. But this is a feature that must be enabled, it does not
comes active by default.


## The #[canyon] annotation

As always, the behaviour of the components of the framework are annotation-driven. This means,
if you need something, probably `Canyon` has an annotation that makes the hard work for you.

But, how `Canyon` knows that he must enable the `migrations` feature, and starting to take
control of whatever action is necessary in your program?

Simple answer. A new annotation. That's probably the most special one, and it must be placed
in the `Rust` more special part of a program. The `main()` function.

```
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

Respecting to the migrations, (obviously, your queries are executed at runtime) the answer is `NO`.

`Canyon` migrations re designed to manage every entity for you at compile time. This it's
done via complex macro operations that you don't have yo worry about. It will be done for you.

But, for this to works, there's two extra requisites required. 
- The structs that you want that `Canyon`
manages, must be annotated with something that we've learn in the *Canyon Entities* chapter.

```
#[canyon_macros::canyon_entity]
pub struct League { /* fields... */ }
```

- You are only allowed to have one `Canyon Entity` per `.rs` source file. So, whenever you're
defining a new entity, please, don't implement more that one in the same source file, or the
compiler will refuses to compile your program.


When the *full mode* it's enabled, `Canyon` will look for your `canyon_entity` annotated structs.
That's when your type is really an entity in `Canyon`!

Then, it will check your provided database info in your `secrets.toml` file, and it will generate
everything that is necessary for that entities in the database.

Everytime that you compile your program, `Canyon` performs a quick search to determine if there
are new entities, if an entity was deleted from your code (this will delete it in your database!),
if an entity was renamed, if an entity field was renamed or changed it's type, and finally, if
an entity has a new `field annotation` or a pre-existent one was removed from your code.

Then, when it's still compiling, `Canyon` will query the database adding, removing or updating
the necessary, letting everything `up-to-date` for when your program starts it's execution.

And that's all. Even if it's by far the most complicated feature of `Canyon`, there's no
much more explanation about how affects you when are developing, and that's because,
as already mentioned before, the idea of `Canyon` it's pretty simple.

*Make the developer's life easier*


# Disclaimer

As you read at the beggining of this file, `Canyon` migrations are still an unstable feature.

That's still code to rework in order to make them work as intended, and there's still
mentioned features that aren't available for the `1.0.0` release.

Those ones are altering the table name, and altering a column name.

The first one has an easy implementation, and will be released soon. 
The second one it's a little more tricky, but will be released *ASAP*

Also please, remember the limitation of having one annotated entity with `#[canyon_entity]`
per file.
This is a restriction imposed because how the migrations works, and what `Canyon`
needs in order to know and relate the data for every entity managed in your program, 
compilation after compilation, because, if they are modified, `Canyon` has to 
update them in the database to make everything works correctly, and that's not
an easy-to-do job.