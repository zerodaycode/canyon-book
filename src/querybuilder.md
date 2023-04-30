# The QueryBuilder

So far, queries have been executed by mapping fields and their corresponding data for each type or instance of that type. Generating the necessary SQL sentences to perform the `CRUD` operations discussed earlier. 

However, the `SQL` language offers a wealth of additional power to analyze and manipulate data stored in tables. Specifically, we are referring to the `SQL clauses` that function as filters.

## Index

- [The QueryBuilder](#the-querybuilder)
  - [Index](#index)
  - [Introduction](#introduction)
  - [How does it work?](#how-does-it-work)
  - [Available methods to work with the builder](#available-methods-to-work-with-the-builder)
  - [The QueryBuilder implementors](#the-querybuilder-implementors)
  - [The Comp enum type](#the-comp-enum-type)
  - [Better by example](#better-by-example)
  - [The #\[derive(Fields)\] macro](#the-derivefields-macro)
  - [The Field enum](#the-field-enum)
  - [The FieldValue enum](#the-fieldvalue-enum)
  - [Next steps](#next-steps)

## Introduction
[(back to top)](#the-querybuilder)

`Canyon` provides support for making queries over your entities more flexible. These queries are also defined in the `CanyonCrud` macro and belong to a family called `QueryBuilder`, which include three types representing the `SELECT`, `UPDATE` and `DELETE` operations. These types implement the `QueryBuilder` trait.

+ The `SelectQueryBuilder`, is invoked after calling `T::select_query()`
+ The `UpdateQueryBuilder`, is invoked after calling `T::update_query()`
+ The `DeleteQueryBuilder`, is invoked after calling `T::delete_query()`

## How does it work?
[(back to top)](#the-querybuilder)

The macro implementations provide various associated functions or methods that return an implementor of the `QueryBuilder` type. These can be used as a builder pattern to construct the query that `Canyon` will use to retrieve data from the database. These functions also contain a `query()` consumer method, which consumes the builder and executes the query against the database, returning the data.

The `QueryBuilder` works by generating the base `SQL`, similar to the `CRUD` operations presented in the previous chapters. However, instead of executing the query, it allows the user to chain methods over a `QueryBuilder` instance. With each method call, the query builder generates more `SQL` content, enabling an efficient and elegant solution to build more complex queries.

## Available methods to work with the builder
[(back to top)](#the-querybuilder)

> "Hablar de la raíz de la familia"

Method calls can be *chained* over a `QueryBuilder` implementor. This allows building a more complex query when needed. By returning `&mut Self`, the user may access the same instance of the `QueryBuilder` and modify its internal state until consumed.

To understand the capabilities of the `QueryBuilder`, let's review the available declarations, which are the available methods in all the implementors.

To see more about the capabilities of the `QueryBuilder`, below are the available methods of its implementors:

```rust
pub trait QueryBuilder<'a, T>
    where T: CrudOperations<T> + Transaction<T> + RowMapper<T> 
{
    /// Returns a read-only reference to the underlying SQL sentence with the same lifetime as self.
    fn read_sql(&'a self) -> &'a str;

    /// Allows the user to append the content of a string slice to the
    /// end of the underlying SQL sentence.
    ///
    /// # Arguments
    ///
    /// * `sql` - The [`&str`] to be appended to the SQL query.
    fn push_sql(&mut self, sql: &str);

    /// Generates a `WHERE` SQL clause for constraining the query based on a column name and a binary comparison operator.
    ///
    /// # Arguments
    ///
    /// * `column` - A `FieldValueIdentifier` that will provide the target column name and the value for the filter.
    /// * `op` - Any type that implements the `Operator` trait.
    fn r#where<Z: FieldValueIdentifier<'a, T>>(&mut self, column: Z, op: impl Operator) -> &mut Self
        where T: Debug + CrudOperations<T> + Transaction<T> + RowMapper<T>;

    /// Generates an `AND` SQL clause for constraining the query based on a column name and a binary comparison operator.
    ///
    /// # Arguments
    ///
    /// * `column` - A `FieldValueIdentifier` that will provide the target column name and the value for the  filter.
    /// * `op` - A type that implements `Operator` for the comparison.
    fn and<Z: FieldValueIdentifier<'a, T>>(&mut self, column: Z, op: impl Operator) -> &mut Self;

    /// Generates an `OR` SQL clause for constraining the query based on a column name and a set of values that are contained within the column.
    ///
    /// # Arguments
    ///
    /// * `column` - A `FieldIdentifier` for the column name.
    /// * `values` - An array of `QueryParameter` with the values that are contained within the column.
    fn and_values_in<Z, Q>(&mut self, column: Z, values: &'a [Q]) -> &mut Self
        where 
            Z: FieldIdentifier<T>,
            Q: QueryParameters<'a>;

    /// Generates an `OR` SQL clause for constraining the query based on a column name and a set of values that are contained within the column.
    ///
    /// # Arguments
    ///
    /// * `column` - A `FieldIdentifier` that will provide the target column name for the filter.
    /// * `values` - An array of `QueryParameter` with the values to filter.
    fn or_values_in<Z, Q>(&mut self, r#or: Z, values: &'a [Q]) -> &mut Self
        where Z: FieldIdentifier<T>, Q: QueryParameters<'a>;

    /// Generates an `OR` SQL clause for constraining the query based on a column name and a binary comparison operator.
    ///
    /// # Arguments
    ///
    /// * `column` - A `FieldValueIdentifier` that will provide the target column name and the value for the filter.
    /// * `op` - Any type that implements `Operator` for the comparison
    fn or<Z: FieldValueIdentifier<'a, T>>(&mut self, column: Z, op: impl Operator) -> &mut Self;

    /// Generates an `ORDER BY` SQL clause for ordering the results of the query.
    ///
    /// # Arguments
    ///
    /// * `order_by`: A `FieldIdentifier` for the column name.
    /// * `desc`: A boolean indicating whether ordering should be ascending or descending.
    fn order_by<Z: FieldIdentifier<T>>(&mut self, order_by: Z, desc: bool) -> &mut Self;
}
```

> Note: The 'Like' clause will be included in a later release.

This will provide you with an idea of the functions performed by the chained methods on the instance. These methods essentially append additional SQL code to the base code to filter the results or execute a particular operation based on your requirements.


## The QueryBuilder implementors
[(back to top)](#the-querybuilder)

Each `QueryBuilder` implementor provides a public interface that contains the methods discussed above, as well as its logical operations. For instance, calling `T::select_query()` returns a `SelectQueryBuilder`, which offers an interface to add `JOIN` clauses to the SQL statement. Currently, `LEFT`, `RIGHT`, `INNER` and `FULL` joins are available. the `inner_join()` method signature is:

```rust
pub fn inner_join(&mut self, join_table: &str, col1: &str, col2: &str) -> &mut Self;
```

Where `join_table` is the entity to perform the join, and `col1` and `col2` are the columns to declare in the `ON` clause. The other join methods have the same parameters.

The `T::update_query()` builds an `UPDATE` statement, providing all the methods defined in the `QueryBuilder` trait declaration. Additionally, it includes a `set()` method:

```rust
pub fn set<Z, Q>(&mut self, columns: &'a [(Z, Q)]) -> &mut Self
    where
        Z: FieldIdentifier<T> + Clone,
        Q: QueryParameter<'a>
```

You may use it as follows:

```rust
let mut q: UpdateQueryBuilder<Player> = Player::update_query_datasource(SQL_SERVER_DS);
    q.set(&[
        (PlayerField::summoner_name, "Random updated player name"),
        (PlayerField::first_name, "I am an updated first name"),
    ])
    .r#where(PlayerFieldValue::id(&1), Comp::Gt)
    .and(PlayerFieldValue::id(&8), Comp::Lt)
    .query()
    .await
    .expect("Failed to update records with the querybuilder");
```

Notice the array of tuples. The first element is the column to target, and the second is the value to update in the database column.

Finally, `T::delete_query()` constructs a `DELETE` statement.

```rust
Tournament::delete_query()
    .r#where(TournamentFieldValue::id(&14), Comp::Gt)
    .and(TournamentFieldValue::id(&16), Comp::Lt)
    .query()
    .await
    .expect("Error connecting to the database during the delete operation");
```

## The Comp enum type
[(back to top)](#the-querybuilder)

The `QueryBuilder` require a comparison operator in some of its methods. This is created from the `Comp` enum type, which is passed as a parameter to the methods that require it. Allowing you to generate comparison operators in a procedural manner.

The available operators are:

```rust
pub enum Comp {
    /// Operator "=" equal
    Eq,
    /// Operator "!=" not equal
    Neq,
    /// Operator ">" greater than <value>
    Gt,
    /// Operator ">=" greater than or equal to <value>
    GtEq,
    /// Operator "<" less than <value>
    Lt,
    /// Operator "=<" less than or equal to <value>
    LtEq,
}
```

## Better by example
[(back to top)](#the-querybuilder)

Using the `QueryBuilder` might be confusing at first. Specially for developers that are not accustomed to the `builder pattern`. However, with practice, developers will see that it is a very practical way of querying.

Let's consider an example where we are working once again with the `League` type: we want to retrieve all records in the `league` table with `id` less than 20 and `slug` equal to `LCK`.

```rust
let leagues: Result<Vec<League>, _> = League::select_query()
    .r#where(
        LeagueFieldValue::id(20),    // This will create a filter -> `WHERE league.id < 20`
        Comp::Lt                     // where the `<` symbol is generated by Canyon when it sees this variant
    ).and(
        LeagueFieldValue::slug("LCK".to_string()),
        Comp::Eq
    ).query()
    .await
    .expect("Query failed");

println!("League elements: {:?}", &leagues);
```

The remaining methods that can be chained are self-explanatory. You can chain as many methods as needed. Make sure that the methods are chained properly to avoid errors.

In addition, if needing more examples, you can visit our repository's `/tests/crud` folder and refer to the `querybuilder_operations.rs` tests. There you will find code examples for all available operations within the `Canyon` query builders.

Now, you may be wondering where `LeagueField`, `LeagueFieldValue(T)`, `FieldIdentifier`, and `FieldValueIdentifier` types and bounds come from.

## The #[derive(Fields)] macro
[(back to top)](#the-querybuilder)

When working with SQL, it is often necessary to specify the column name or type of a column in some operations. To avoid using raw or literal values, we created the `#[derive(Fields)]` derive macro. This macro generates special enumerated types that allows the user to refer to a database column name, type or value in a procedural way. It replaces the string that is usually used with a piece of valid Rust code.

Rest assured that `Canyon` translates these procedures into the corresponding identifiers whenever they are needed. Let's take a closer look at these two types.

## The Field enum
[(back to top)](#the-querybuilder)

The field enum is an autogenerated enum that serves to relate each field of a type with a database column name in a procedural way, making it easier to reflect them through the code.

The identifier of the Field enum is generated as the concatenation of the type's identifier with "Field". It is important to note that there is a naming convention in `Canyon`, where the variants of an enumeration in Rust are typically written in `PascalCase`, while in `Canyon` they are written in `snake_case` to match the way that the field is written.

For example:

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

When directly interacting with a database, it is common to refer to a column using a plain string. That is a problem, because the compiler wouldn't be able to detect any mistake until it attempts to make the query. We, Rust developers, didn't choose this language in order to have runtime errors.

Therefore, whenever a concrete column of the `league` table must be specified, such as the `slug` column, you could write `LeagueField::slug`. While this may seem like more code, it removes the potential errors that can come with using literals in the code. IDEs can also offer autocompletion, making it easier to write and detect wrong variants.

The `impl FieldIdentifier<LeagueField> for LeagueField {}` trait implementation is part of Canyon's method of identifying when it must accept a `FieldIdentifier` type as an argument for functions that generate filters for queries, or when constraining the concrete arguments of some operations of the `QueryBuilder` to the same type. This helps to avoid potential errors of specifying variants or different types, and is almost always used when you generate a filter for your SQL query.

## The FieldValue enum
[(back to top)](#the-querybuilder)

In addition to the `Field` enum, another enum is generated by the `canyon_entity` proc-macro annotation, known as the `FieldValue` enum. This enum serves as a procedural way to determine which field is being referred to, but with the added capability of accepting a value as an argument in every variant. This value can be any supported literal or type for which the `QueryParameter<'a>` is implemented. The main purpose of this enum is to create filters for an `SQL` query where the filter needs a value to perform the filtering.

The `FieldValue` enum looks like this:

```rust
#[derive(Clone, Debug)]
#[allow(non_camel_case_types)]
pub enum LeagueFieldValue {
    id(&dyn QueryParameter<'_>),
    ext_id(&dyn QueryParameter<'_>),
    slug(&dyn QueryParameter<'_>), 
    name(&dyn QueryParameter<'_>), 
    region(&dyn QueryParameter<'_>), 
    image_url(&dyn QueryParameter<'_>)
}

impl FieldIdentifierValue<LeagueFieldValue> for LeagueFieldValue {}
```

Using the `FieldValue` enum provides a powerful way of writing filters for `SQL` queries. For instance, `LeagueFieldValue::id(1)` can be used to generate a `WHERE` clause filter: `WHERE league.id = 1`, among other operations.

## Next steps
[(back to top)](#the-querybuilder)

Now that `Querybuilder` has been discussed, the next chapter is about `Migrations`. Where everything that have been discussed up until now will be wrapped into the full package that `Canyon` provides. See you there!