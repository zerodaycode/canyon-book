# Build a query

The generated CRUD methods are deliberately small. They cover common operations, but they do not know whether your next read needs a filter, a join, or a particular ordering. The query builder is the point where you describe that extra shape.

`Read` supplies `select_query()`; `Update` and `Delete` supply `update_query()` and `delete_query()`. These methods choose the default datasource's SQL dialect and return builders. They do not execute anything. `build()?` emits a `Query` containing both the SQL and its bound parameters.

## A filtered read

Given the `Team` model from [Entities and mapping](./canyon_entities.md):

```rust
use canyon_sql::{
    crud::Read,
    query::{
        operators::Operator,
        querybuilder::QueryBuilderExt,
    },
};

let name = TeamFieldValue::name("Blue".to_owned());
let teams: Vec<Team> = Team::select_query()?
    .where_value(&name, Operator::Eq)
    .build()?
    .launch_default()
    .await?;
```

`where_value` receives a generated `FieldValue` and collects its value as a bound parameter. Keep `name` alive until the query is built and executed. `and` and `or` add further conditions. `and_values_in` and `or_values_in` accept a `Field` and a non-empty slice of values; an empty list is a `QueryBuilderError::EmptyInClause`.

The lower-level `r#where(column, operator)` adds a placeholder **without** collecting a value. Prefer `where_value` for ordinary application code; with `r#where`, you must provide the matching parameter when executing the SQL yourself.

The `Operator` enum includes equality, inequality, greater/less-than comparisons, `IN`, and `LIKE` / `NOT LIKE`. For a contains-style pattern, use `Operator::Like(LikeKind::Full)`; `Left` and `Right` select the other wildcard placements. Canyon renders backend-specific placeholders and quoting when it builds the query.

## What `Fields` contributes

For `Team`, `Fields` generates three public enums:

- `TeamTable`: table metadata such as `TeamTable::DbName`.
- `TeamField`: a column name, such as `TeamField::name`, for ordering or a join.
- `TeamFieldValue`: the same column with a value of its Rust field type, such as `TeamFieldValue::name("Blue".to_owned())`, for a predicate.

They remain the current query-builder API. We may replace this generated shape with typed column descriptors in a future major version, but there is no deprecation in 0.5.1. Multiple models can derive `Fields` in the same Rust module.

## Joins and projection

`SelectQueryBuilderExt` adds `inner_join`, `left_join`, `right_join`, `full_join`, `with_columns`, `with_distinct`, `count`, and `order_by`. A join identifies its table and the two columns of the equality:

```rust
use canyon_sql::query::querybuilder::{QueryBuilderExt, SelectQueryBuilderExt};

let query = Team::select_query()?
    .inner_join(PlayerTable::DbName, TeamField::id, PlayerField::team_id)
    .order_by(TeamField::id, false)
    .build()?;

println!("{}", query.sql());
```

This example assumes the `Player` model from [Relationships](./crud_mapping/foreign_keys.md). A join can return more columns than `Team` declares; the mapper ignores extras, but it cannot invent required missing fields. When you need values from both tables, map the intended projection into an appropriate result type. Be explicit about selected columns when names overlap.

## Updates and deletes

A conditional update must say what changes. Prefer `set_values`, which records each target column and its bound value together:

```rust
use canyon_sql::{
    connection::DbConnection,
    core::Canyon,
    crud::Update,
    query::{operators::Operator, querybuilder::{QueryBuilderExt, UpdateQueryBuilderExt}},
};

let changes = [(TeamField::name, "Blue Tigers")];
let target = TeamFieldValue::id(42_i64);
let query = Team::update_query()?
    .set_values(&changes)?
    .where_value(&target, Operator::Eq)
    .build()?;

let connection = Canyon::instance()?.get_default_connection()?;
let affected = connection.execute(query.sql(), query.params()).await?;
```

`set` is a lower-level alternative that lists columns but does **not** collect matching values. Use it only when you supply parameters yourself. Canyon rejects an empty `SET` and a second `SET` on the same builder.

`delete_query()?` uses the shared `QueryBuilderExt` predicates; add a `WHERE` condition before executing unless you truly intend to delete every row. `execute` returns the affected-row count. For single-row changes identified by a model's key, `update()` and `delete()` are simpler.

There is also a lower-level `InsertQueryBuilder` for cases where a generated entity insert is not suitable. Construct it with a table and `DatabaseType`, then use `InsertQueryBuilderExt::with_columns`, `with_values`, and optionally `returning` before `build()`. The explicit column list and bound-value count must match. Ordinary entity inserts should use `insert()` or `insert_with()`; there is no generated `insert_query()` method on `Crud`.

## Dialect and connection are separate choices

`Team::select_query_with(DatabaseType::MySQL)?` chooses MySQL SQL syntax. It does not connect to MySQL. After `build()`, use `launch_with("mysql_datasource")` to execute on the matching datasource. The `Query` also exposes `sql()` and `params()` for inspection or direct execution. Do not send SQL generated for one dialect to another backend.

For operations outside these patterns, use [a connection directly](./raw_queries.md). The builder validates known mistakes, but it does not prove that every selected table or column exists in your live schema.
