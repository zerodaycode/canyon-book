# Relations: Foreign Keys

`Canyon` comes with a easy to implement solution when you need to make rules over some kind of relations
or associations.

In this chapter, we'll take a shoot on how to add SQL `CONSTRAINTS` to the probably, most clasic
relation in the `SQL` standard. The `FOREIGN KEY`.

The `FOREIGN KEY` constraint is used to prevent actions that would destroy links between tables.
A `FOREIGN KEY` is a field (or collection of fields) in one table, that refers to the 
`PRIMARY KEY` in another table.

The table with the foreign key is called the *child* table, and the table with the `PRIMARY KEY`
is called the *referenced* or *parent* table.

For this examples, we'll need two `Canyon Entities` to demonstrate what solution has `Canyon`
for im
plementing this relations.

```
use canyon_sql::*;

#[derive(Debug, Clone, CanyonCrud, CanyonMapper, ForeignKeyable)]
#[canyon_macros::canyon_entity]
pub struct League {
    pub id: i32,
    pub ext_id: i64,
    pub slug: String,
    pub name: String,
    pub region: String,
    pub image_url: String
}
```
*league.rs*


```
use canyon_sql::*;

#[derive(Debug, Clone, CanyonCrud, CanyonMapper)]
#[canyon_macros::canyon_entity]
pub struct Tournament {
    pub id: i32,
    pub ext_id: i64,
    pub slug: String,
    pub start_date: NaiveDate,
    pub end_date: NaiveDate,
    #[foreign_key(table = "league", column = "id")]
    pub league: i32
}
```
*tournament.rs*



## The foreign_key annotation and the ForeignKeyable derive macro.

As you may notice, a new annotation has been introduced in the `Tournament` entity.

`#[foreign_key(table = "league", column = "id")]`

This annotation will generate a new `parent-child` relation between `League` being
the *parent* or the *referenced* and `tournament` being the *child*, where this is
specified through the `table` argument, and the refered field through the `column`
argument.

But, for this to work `Canyon` *full mode* must be enabled. This is because the
creation, modification of delete `CONSTRAINT` relation must be specified querying
the database, in order to this generate such a bound.

The annotation it's a Rust field level annotation, meaning that always must be placed in front
(or above) a field. This annotation accepts two arguments: `table` and `column`.

*NOTE: table and column arguments accepts both an &str to create the data neccesary*
*for generate the SQL behind this operation.*
*But this behaviour will change in the next release, changing the table argument to accept*
*a valid Rust identifier and the column a FieldIdentifier discussed in past chapters.*

For this relation to be enabled, we must tell to `Canyon` one more thing. The `entities`
that behaves like `parents` at some point in your program, must add a new `derive macro`
to the presented in the past chapters. This is the `ForeignKeyable` derive macro.

Whenever you have a Foreign Key relation in your code, you must tell `Canyon` that
this `T`, at some point will be the `parent` for some other `T`. There's no need to specify
the child, because `Canyon` will be able to resolve that doubt with the first annotation, 
but do not forget to annotate your parents with the `ForeignKeyable` macro, or `Canyon`
will complain later when you try to query the database given this relation.


## Queries for the Foreing Key relation.

A `SQL CONSTRAINT` operation in `Canyon` has two virtues. 

The first is all about data integrity. Having such a relation doesn't allow you to insert
invalid data, or two remove things that depends on others. This is a really common scenario
when working with relational databases.

The second one, it's that you can query things, thinking in such a relation.
For example, in the types defined at the beggining of the chapter, `League` and `Tournament`,
you may ask `Canyon` two find:

- The data of the `League` which is referenced in this relation.
- How many `Tournaments` are pointing to a concrete instance of `League`.

The first one it's just a `SELECT * FROM league WHERE league.id = tournamemnt.league`, where
you will replace `tournament.league` for the value contained a concrete `tournament` instance
in its `league` field, `Canyon` already implements you a convenient method to quickly discover
what's the related `league`. It will autogenerate a method following the convention
`search_parent_table_league`, that will return an optional if it's able to found the `League`
parent for the `Tournament` instance.

```
    // You can search the 'League' that it's the parent of a concrete instance of 'Tournament'
    let parent_league: Option<League> = tournament_itce.search_league().await;

    println!(
        "The related League queried through a method of tournament: {:?}", 
        &parent_league
    );
```


A more complete example, when we retrieve some `Tournament` from the database, and then, we
ask `Canyon` to find its `League` parent:

```
    let tournament: Option<Tournament> = Tournament::find_by_id(1).await;
    println!("Tournament: {:?}", &tournament);

    if let Some(trnmt) = tournament {
        let result: Option<League> = trnmt.search_league().await;
        println!("The related League as method if tournament is some: {:?}", &result);
    } else { println!("`tournament` variable contains a None value") }
```
*Note that the identifier for the autogenerated method is called 'search_league'*


`Canyon` also has another way of perform this operation.

```
    /*  
        Finds the League (if exists) that it's the parent side of the FK relation
        for the Tournament entity.
        This does exactly the same as the `.search_league()` method, but implemented
        as an associated function. 
        There's no point on use this way, this it's just provided in the example
        because was codified before the method implementation over a T type, and
        still exists in the Canyon's code. 
        It seems a little more readable to call the method .search_parentname()
        over a T type rather than make the search with the associated method, 
        passing a reference to the instance, but it's up to the developer
        to decide which one likes more.
    */
    let related_tournaments_league: Option<League> = Tournament::belongs_to(&lec).await;
    println!(
        "The related League queried as through an associated function: {:?}", 
        &related_tournaments_league
    );
```
*Same that search_league() instance method, but being requested from an associated function to the Tournament type.*



And the second one, would be find the child tournaments of a concrete `League`. This behaviour it's usually known as
`the reverse side of a Foreign Key`, but being this more specific to the action of access a concrete instance
of `Tournament` given a `League`. 

In `Canyon` this is not posible, neither too much desirable, because we are working directly with the `SQL` 
data but through `Rust` code. Besides that, `Canyon` offers you the posibility of find the childs of a 
concrete type, or in other words, what `Tournaments` are pointing to a concrete `League`:

```
    /* Finds all the tournaments that it's pointing to a concrete `League` record
    This is usually known as the reverse side of a foreign key, but being a
    many-to-one relation on this side, not a one-to-one */

    let tournaments_belongs_to_league: Vec<Tournament> = 
        Tournament::search_by__league(&lec)
            .await
            .ok()
            .unwrap();

    println!("Tournament that belongs to lec: {:?}", &tournaments_belongs_to_league);    
```
*Finds the childs for a concrete League instance.*

Note that the associated function, has a nomemclature convenction of `search_by` + `__` + `parent_type`.
Also, note how it receives a reference to an instance of `League` (&lec), so you will need first
of all a valid `League` instance. 



# The _result variations

For this two operations, exists the `_result` operations discussed in the [chapter 4](result_operations.md).
Just look in your `IDE` for the same methods an associated functions what the described above, just ending
with `_result` in its identifier, an you will be able to perform the same operations but being wrapped
in a `Result<T, E>` type, enabling you to handle an `Err` if exists one when the operation its made.