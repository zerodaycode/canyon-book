# The QueryBuilder

`Canyon-SQL` comes with a powerful querybuilder.

Until now, queries are executed by mapping fields and the data of the fields
for every type or instance of that type, and generating the SQL senteces needed to perform
the operations discussed before.

But the `SQL` idiom offers a lot of more power to analyze and manipulate the data stored in its tables.
We are concretly talking now about the `SQL Clauses` that acts like filters.

`Canyon` provides support for make the queries over your entities more flexible. They're also
defined in the `CanyonCrud` macro. They belongs to a family called `QueryBuilder`, and exists three types that represents the `SELECT`, `UPDATE` and `DELETE` operations and that implements the base operations designed by the `QueryBuilder` trait, among their particular ones.

+ The `SelectQueryBuilder`, invoked after call `T::select_query()`
+ The `UpdateQueryBuilder`, invoked after call `T::update_query()`
+ The `DeleteQueryBuilder`, invoked after call `T::delete_query()`

## How does it works?

Every associated function or method provided through the macro implementations
that returns an implementor of the `QueryBuilder` type, can be used as a builder pattern to construct
the query that `Canyon` will use to retrive data from the database. Those associated
functions contains a `query()` consumer method, which will consume the builder and will execute the query against the database, returning the data.

The `QueryBuilder` works by first, generate the base `SQL` as the rest of the `CRUD` operations
presented in the past chapters. But, instead of executing the query, it lets you to chain
methods over a `QueryBuilder` instance that, for every method call, generates more `SQL` content
for your query, being an efficient an elegant solution to build more complex queries.

## Available methods to work with the builder

// Hablar de la raíz de la familia

You can *chain* calls over your specific `QueryBuilder` implementor to build a more complex query when you need it.
Every method will return `&mut Self` to make you able to aggregate and modify your query according
to your needs. By returning `&mut Self`, we can have access to the same instance of the `QueryBuilder`,
modifying it's internal state until it's consumed.

To start to see what the `QueryBuilder` it's capable to do, let's review
the available declarations, that are the available methods in all the implementors.

```rust
pub trait QueryBuilder<'a, T>
    where T: CrudOperations<T> + Transaction<T> + RowMapper<T> 
{
    /// Returns a read-only reference to the underlying SQL sentence,
    /// with the same lifetime as self
    fn read_sql(&'a self) -> &'a str;

    /// Public interface for append the content of an slice to the end of
    /// the underlying SQL sentece.
    /// 
    /// This mutator will allow the user to wire SQL code to the already
    /// generated one
    /// 
    /// * `sql` - The [`&str`] to be wired in the SQL
    fn push_sql(&mut self, sql: &str);

    /// Generates a `WHERE` SQL clause for constraint the query.
    /// 
    /// * `column` - A [`FieldValueIdentifier`] that will provide the target
    /// column name and the value for the filter
    /// * `op` - Any element that implements [`Operator`] for create the comparison
    /// or equality binary operator 
    fn r#where<Z: FieldValueIdentifier<'a, T>>(&mut self, column: Z, op: impl Operator) -> &mut Self
        where T: Debug + CrudOperations<T> + Transaction<T> + RowMapper<T>;

    /// Generates an `AND` SQL clause for constraint the query.
    /// 
    /// * `column` - A [`FieldValueIdentifier`] that will provide the target
    /// column name and the value for the filter
    /// * `op` - Any element that implements [`Operator`] for create the comparison
    /// or equality binary operator 
    fn and<Z: FieldValueIdentifier<'a, T>>(&mut self, column: Z, op: impl Operator) -> &mut Self;

    /// Generates an `AND` SQL clause for constraint the query that will create
    /// the filter in conjunction with an `IN` operator that will ac
    /// 
    /// * `column` - A [`FieldIdentifier`] that will provide the target
    /// column name for the filter, based on the variant that represents
    /// the field name that maps the targeted column name
    /// * `values` - An array of [`QueryParameters`] with the values to filter
    /// inside the `IN` operator
    fn and_values_in<Z, Q>(&mut self, column: Z, values: &'a [Q]) -> &mut Self
        where 
            Z: FieldIdentifier<T>,
            Q: QueryParameters<'a>;

    /// Generates an `OR` SQL clause for constraint the query that will create
    /// the filter in conjunction with an `IN` operator that will ac
    /// 
    /// * `column` - A [`FieldIdentifier`] that will provide the target
    /// column name for the filter, based on the variant that represents
    /// the field name that maps the targeted column name
    /// * `values` - An array of [`QueryParameters`] with the values to filter
    /// inside the `IN` operator
    fn or_values_in<Z, Q>(&mut self, r#or: Z, values: &'a [Q]) -> &mut Self
        where Z: FieldIdentifier<T>, Q: QueryParameters<'a>;

    /// Generates an `OR` SQL clause for constraint the query.
    /// 
    /// * `column` - A [`FieldValueIdentifier`] that will provide the target
    /// column name and the value for the filter
    /// * `op` - Any element that implements [`Operator`] for create the comparison
    /// or equality binary operator 
    fn or<Z: FieldValueIdentifier<'a, T>>(&mut self, column: Z, op: impl Operator) -> &mut Self;

    /// Generates a `ORDER BY` SQL clause for constraint the query.
    /// 
    /// * `order_by` - A [`FieldIdentifier`] that will provide the target
    /// column name
    /// * `desc` - a boolean indicating if the generated `ORDER_BY` must be
    /// in ascending or descending order
    fn order_by<Z: FieldIdentifier<T>>(&mut self, order_by: Z, desc: bool) -> &mut Self;
}
```

> Note: If you miss the 'LIKE' clause, don't worry. It will be released soon.*

That's will make you and idea on what the chained methods do over the instance. It really just appends more `SQL` code to the base one, to filter the results or to concretize an operation acording to your needs.

## The Comp enum type

The `QueryBuilder` usually works receiving in some methods a comparation operator.
In `Canyon` it's created from the `Comp` enum type, which it's received by parameter by the methods that required them, and allows you to generate comparation operators
in a procedural way.

This are the available ones:

```rust
pub enum Comp {
    /// Operator "=" equals
    Eq,
    /// Operator "!=" not equals
    Neq,
    /// Operator ">" greather than <value>
    Gt,
    /// Operator ">=" greather or equals than <value>
    GtEq,
    /// Operator "<" less than <value>
    Lt,
    /// Operator "=<" less or equals than <value>
    LtEq,
}
```

## Better by example

We know that it could be confusing how the `QueryBuilder` works, essentially if you not
fight too often with `Builders`, but, wait... probably you will use it a lot!

Let's put an use case, working with our beloved `League` type: We want to recover
all the `league` table records which `id` are less than 20 and it's `slug` it is
equals to `LCK`.

```rust
let leagues: Result<Vec<League>, _> = League::select_query()
        .r#where(
            LeagueFieldValue::id(20),    // This will create a filter -> `WHERE league.id < 20`
            Comp::Lt                     // where the `<` symbol it's generated by Canyon when sees this variant
        ).and(
            LeagueFieldValue::slug("LCK".to_string()),
            Comp::Eq
        ).query()
        .await
        .expect("Query failed");

println!("League elements: {:?}", &leagues);
```

The rest of methods that you may desire to chain are trivial. You can chain as many as you need, but
obviously, try to use those one who makes sense for your queries.

Also, if you need to look at more examples, go to our repository, to the `/tests/crud` folder, and look for the `querybuilder_operations.rs` tests. You eventually will have any of the available operations within the `Canyon` querybuilders written there.

The `update multiple columns` it's avaiable within the `UpdateQueryBuilder` type.
It just simply works by setting the values of the provided columns (via FieldIdentifier items)
to the values provided in the special `SET` clause.

`SET` clause arguments are a little bit tricky. It will accept a reference to an
array of tuples, being every tuple compound of a implementor of a `FieldIdentifier` trait and an implementor of the `QueryParameter<'a>` trait.

Any thing that implements `FieldIdentifier` at the `.0` position, and anything that implements
`QueryParameter<'a>` at the `.1` position of the tuple passed in. A basic example:

```rust
    League::update_query()
        .set_clause(
            &[
                (LeagueField::ext_id, 394039),
                (LeagueField::image_url, "https://random_updated_url.up")
            ]
        ).where_clause(
            LeagueFieldValue::id(3), Comp::Gt
        ).query()
        .await;  // Unused result
```

*Updating the values of multiple columns with Canyon at the same time*

Hmmm... but where does all of the `LeagueField`, `LeagueFieldValue(T)` and the `FieldIdentifier` and `FieldValueIdentifier` types or bounds comes from?

## The #[derive(Fields)] derive macro

Usually, when working with `SQL`, there's a lot of places where you want to
specify the column of the type of the column for some reason in some operation.

We liked to have a different way of specifiying those values without being just raw or literal values.
So we created the #[derive(Fields)] derive macro.
This macro provides the autogeneration of a very special enumerated types. They are the way of refer
to a database column name, it's type or it's value in a procedural way, replacing
the string that you usually will put in that place to reference it for a
piece of valid `Rust` code.

Ah, don't worry, `Canyon` already translates that procedures for the correspondent
identifiers whenever it needs it.

Let's now review those two beasts!

## The Field enum

The field enum it's an autogenerated enum, which main purpose it's to relate
every field of your type with a database column name, as a procedural way of reflecting them through the code.

It's identifier it's generated as the concatenation of your type's identifier + "Field".

> Note: There's an important convenction here in Canyon about the way on how the variants
are written. By default, variants of an enumeration in Rust are written in PascalCase form,
while in Canyon are written exactly the same on how the field it's written (convenction here
applies snake_case).

```rust
#[derive(Clone, Debug)]
#[allow(non_camel_case_types)]
pub enum LeagueField {
    id,
    ext_id,
    slug, 
    name, 
    region, 
    image_url
}

impl FieldIdentifier<LeagueField> for LeagueField {}
```

In any place that you must specify a concrete column of the `league` table, for example, the
`slug` column, you will write `"slug"`. Any literal in the code it's error prone, since a lot of
bad things can go wrong with them, plus IDE's don't offer autocompletation of information about
them (we're not refering to the AI autocompletion tools, which are out of scope for this).

So now, you only need to type `LeagueField::slug`.

Yes, we know. "It's more code"... HA! But we removing a literal from our code, and potencially
every literal written in this situation. The IDE now will help us if we write a bad one, or
autocompleting (God bless the `Ctrl + spacebar`).

Also, it's easier to detect a wrong variant. Even they are translated into literals, when used,
you can only put the wrong one and make a bad operation, but now you can't write a literal
that does not even exists and makes your code panic!

About the `impl FieldIdentifier<LeagueField> for LeagueField {}` trait impl, it's just part of the `Canyon's` way of knowing when it must accept a FieldIdentifier
type as an argument of the functions that generates the filters for the queries, and for constraint the concrete arguments of some operations of the `QueryBuilder`
to the same type, avoding the potential error of specify variants of different types.
Almost always are used when you generate some kind of filter for your query in the `SQL` language.

## The FieldValue enum

In a complementary way to the `Field` enum, another one it's generated with the `canyon_entity`
proc-macro annotation.

This new one it's known as the `FieldValue` enum, and represents the same as the `Field`
enum (a procedural way to determine to what field we are refering) but accepts a value
as an argument in every variant. That argument must be of the exact same type of
the field's type.

The main purpose of this enum, it's to make filters for an `SQL` query, where the
filter needs a value to perform the filtering.

It looks like this:

```rust
#[derive(Clone, Debug)]
#[allow(non_camel_case_types)]
pub enum LeagueFieldValue {
    id(i32),
    ext_id(i64),
    slug(String), 
    name(String), 
    region(String), 
    image_url(String)
}

impl FieldIdentifierValue<LeagueFieldValue> for LeagueFieldValue {}
```

It's just a powerful way of write filters for the `SQL` queries.
For example, a `LeagueFieldValue::id(1);` could be used for generate
a `WHERE` clause filter: `WHERE league.id = 1`, among other operations.
