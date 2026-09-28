# Build a query

`find_all()` is useful until you want only the blue teams, or teams ordered by name. Then you need to describe the query before running it. That is the query builder's job.

The steps are:

1. Start with `select_query()`, `update_query()`, or `delete_query()`. These choose the default datasource's SQL dialect, but execute nothing.
2. Add predicates, joins, ordering, or values.
3. Call `build()?` to get a `Query` containing SQL and bound parameters. Then launch it or pass it to a connection.

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

`where_value` takes the field and value together. It collects `"Blue"` as a bound parameter; no string interpolation is needed. Keep `name` alive through the async call.

For more conditions, use `and` or `or`. The `and_values_in` and `or_values_in` methods take a `Field` and a non-empty slice of values; an empty slice produces `QueryBuilderError::EmptyInClause`.

> **Mind the parameters:** The lower-level `r#where(column, operator)` adds a placeholder but does **not** collect a value. Use `where_value` unless you intend to supply the matching parameter yourself at execution time.

The `Operator` enum includes equality, inequality, greater/less-than comparisons, `IN`, and `LIKE` / `NOT LIKE`. For a contains-style pattern, use `Operator::Like(LikeKind::Full)`; `Left` and `Right` select the other wildcard placements. Canyon renders backend-specific placeholders and quoting when it builds the query.

## What `Fields` contributes

For `Team`, `Fields` generates three public enums:

- `TeamTable`: table metadata such as `TeamTable::DbName`.
- `TeamField`: a column name, such as `TeamField::name`, for ordering or a join.
- `TeamFieldValue`: the same column with a value of its Rust field type, such as `TeamFieldValue::name("Blue".to_owned())`, for a predicate.

These enums are the supported API in 0.5.1. A future major version may use typed column descriptors instead, but `Fields` is not deprecated. Multiple models can derive it in the same Rust module.

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

This uses the `Player` model from [Relationships](./crud_mapping/foreign_keys.md). The query may return more columns than `Team` declares; the mapper ignores extras. It still requires every field in `Team`.

If you need values from both tables, define a result type for that projection. Select overlapping column names explicitly so the mapper can tell them apart.

For a smaller projection, or to remove duplicate rows, use the select-specific methods:

```rust
let query = Team::select_query()?
    .with_columns(vec![TeamField::name])
    .with_distinct()
    .order_by(TeamField::name, false)
    .build()?;
```

This query selects only `name`, so it cannot produce a `Team`: the mapper also needs `id`. Use a result type with the projected shape, or inspect the rows with `query_rows(...)`.

For a count without filters, `Team::count().await?` is simpler and returns `i64` on every backend.

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

The lower-level `set` lists columns but does **not** collect their values. Use it only when you will supply parameters yourself. Canyon rejects an empty `SET` and a second `SET` on the same builder.

> **Check the scope of a write:** Add a predicate to `delete_query()?` unless you mean to delete every row. The same care applies to a multi-row update. `execute` returns the affected-row count; `update()` and `delete()` are simpler when you have one model and its key.

For an insert that does not fit `insert()`, construct `InsertQueryBuilder` with a table and `DatabaseType`. Add columns with `InsertQueryBuilderExt::with_columns`, values with `with_values`, and optionally `returning` before `build()`. The number of columns and bound values must match. `Crud` does not generate an `insert_query()` method.

## Dialect and connection are separate choices

`Team::select_query_with(DatabaseType::MySQL)?` chooses MySQL SQL syntax. It does **not** connect to MySQL. After `build()`, use `launch_with("mysql_datasource")` on a matching datasource.

The `Query` exposes `sql()` and `params()` for inspection or direct execution. Do not send SQL generated for one dialect to another backend.

Choose the launch method by the result you expect:

| Result | Method |
| --- | --- |
| Rows mapped to `Team` | `launch_default::<Team>()` or `launch_with::<_, Team>(...)` |
| One scalar of type `T` | `launch_one_for_default::<Team, T>()` or `launch_one_for_with::<Team, T, _>(...)` |

The scalar type must match the backend's value. A raw SQL Server `COUNT(*)`, for example, is read as `i32`; generated `Team::count()` converts that count to `i64` for you.

If the builder gets in the way, [use a connection directly](./raw_queries.md). Builder validation catches certain malformed shapes, not a table or column that is missing from your live database.
