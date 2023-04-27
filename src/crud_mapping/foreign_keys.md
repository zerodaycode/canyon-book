# Relations: Foreign Keys

`Canyon` provides an easy-to-implement solution for enforcing rules over relations or associations in a database. This chapter will explore how to add SQL `CONSTRAINTS` to the classic relation in the SQL standard, the `FOREIGN KEY`.

## Index
- [Relations: Foreign Keys](#relations-foreign-keys)
  - [Index](#index)
  - [Foreign Key](#foreign-key)
  - [The `foreign_key` annotation and the `ForeignKeyable` derive macro](#the-foreign_key-annotation-and-the-foreignkeyable-derive-macro)
  - [Foreign Key Queries](#foreign-key-queries)
    - [Retrieve data for a `League` referenced by a `Tournament`](#retrieve-data-for-a-league-referenced-by-a-tournament)
    - [Find out how many `Tournament`s are associated with a specific `League`](#find-out-how-many-tournaments-are-associated-with-a-specific-league)

## Foreign Key
[(back to top)](#relations-foreign-keys)

The `FOREIGN KEY` constraint is used to prevent actions that would destroy links between tables. A `FOREIGN KEY` is a field or collection of fields in one table that refers to the `PRIMARY KEY` in another table. The table with the foreign key is called the child table, and the table with the `PRIMARY KEY` is called the referenced or parent table.

To demonstrate how `Canyon` implements this relation, two entities will be used:

```rust
#[derive(CanyonCrud, CanyonMapper, ForeignKeyable)]
#[canyon_macros::canyon_entity]
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
*league.rs*


```rust
#[derive(CanyonCrud, CanyonMapper)]
#[canyon_entity]
pub struct Tournament {
    #[primary_key]
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

## The `foreign_key` annotation and the `ForeignKeyable` derive macro
[(back to top)](#relations-foreign-keys)

As you may have noticed, a new annotation has been introduced in the `Tournament` entity: 

```rust
#[foreign_key(table = "league", column = "id")]
```

This annotation generates a new parent-child relation between `League` and `Tournament`, where `League` is the parent or the referenced entity, where `League` is the parent or the referenced entity, and `Tournament` is the child entity. This is specified through the `table` argument. Which indicates the parent table, and the `column` argument, which indicates the referred field.

`foreign_key` annotations describe a parent-child relation between two tables. It requires two arguments:

 - `table` : The parent table. In the above example, it is "league";
 - `column` : The column or field that is the `id` of the parent table;

The current table is `Tournament`. Therefore `Tournament` is the child and `League` is the parent. `Tournament` can not exist without a `League` to reference. But `League` can exist without any `Tournament` existing.

> Note: The `table` and `column` arguments currently accept an `&str` to create the data necessary for generating the SQL for each operation. However, this behavior will change in upcoming releases, where the `table` argument will accept a valid Rust identifier, and the `column` argument will accept a `FieldIdentifier` discussed in previous chapters.

For this relation to be implemented successfully, a new derive macro must be included along with the ones presented in past chapters. Entities that behave like parents must have the `ForeignKeyable` derive macro.

Whenever there is a foreign key relation in the code, you must tell `Canyon` that this entity `T` at some point will be the parent for some other entity `T`. There is no need to specify the child because `Canyon` will be able to resolve that question through the first annotation.

However, do not forget to annotate your parents with the `ForeignKeyable` macro, or `Canyon` will issue a warning later when trying to query the database given this relation.

## Foreign Key Queries
[(back to top)](#relations-foreign-keys)

In `Canyon`, a `SQL CONSTRAINT` operation offers two advantages:

Firstly, it ensures data integrity by preventing invalid data insertion and removing data that depends on other records. This feature is particularly useful when working with relational databases.

Secondly, it enables querying based on the relation. For instance, given the `League` and `Tournament` types defined at the beginning of this chapter, a user may query:

- Retrieve data for a `League` referenced by a `Tournament`.
- Find out how many `Tournament`s are associated with a specific `League`.

### Retrieve data for a `League` referenced by a `Tournament`
[(back to top)](#relations-foreign-keys)

To retrieve data for a `League` referenced by a `Tournament`. Manually, a query similar to this could be made:

```sql
SELECT * FROM league WHERE league.id = tournament.league
```

In the above example `tournament.league` is the id of the league stored on the tournament instance.

With `Canyon`, a method for this will be generated called `search_parent_table_league`. It returns an option that will be `Some` if it finds the `League` instance.

```rust
// You can search the 'League' that is the parent of a concrete instance of 'Tournament'
let parent_league: Option<League> = tournament_itce.search_league().await;

println!(
    "The related League queried through a method of tournament: {:?}", 
    &parent_league
);
```

In a more complete example, suppose you retrieve some `Tournament` from the database and then ask `Canyon` to find its `League` parent:

```rust
let tournament: Option<Tournament> = Tournament::find_by_id(1).await;
println!("Tournament: {:?}", &tournament);

if let Some(trnmt) = tournament {
    let result: Option<League> = trnmt.search_league().await;
    println!("The related League as method if tournament is some: {:?}", &result);
} else { println!("`tournament` variable contains a None value") }
```
*Note that the identifier for the autogenerated method is called 'search_league'*

### Find out how many `Tournament`s are associated with a specific `League`
[(back to top)](#relations-foreign-keys)

On the type declarations for `Tournament` and `League`. It can be noticed that `Tournament` stores the id of a foreign `League`. It is possible to have several tournaments that reference the **same** `League`. This query is about retrieving these entries.

This is usually known as "the reverse side of a Foreign Key". `Canyon` offers the possibility to find the children of a specific type or, in other words, the `Tournaments` associated with a particular `League`:

```rust
    /* Find all the tournaments that are pointing to the same `League` record. This is usually known as the reverse side of a foreign key. It is a many-to-one relation on this side, not a one-to-one */

    // Find league with id 1
    let some_league: League = League::find_by_pk(&1)
        .await
        .expect("Result variant of the query is err")
        .expect("No result found for the given parameter");

    // Retrieve all tournaments that reference to some_league
    let child_tournaments: Vec<Tournament> = Tournament::search_league_childrens(&some_league)
        .await
        .expect("Result variant of the query is err");

    assert!(!child_tournaments.is_empty());
    child_tournaments
        .iter()
        .for_each(|t| assert_eq!(t.league, some_league.id));    
```
*Finds the records that are directly pointing to an existing League instance.*

It is important to note that the associated function follows a naming convention of `search_` + `parent_type` + `childrens`. Furthermore, it receives a reference to an instance of `League`(`&lec`), necessitating the existence of a valid record of `League` prior to executing the function.