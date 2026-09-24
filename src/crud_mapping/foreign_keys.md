# Relationships

A foreign key has two sides: a child stores a value, and a parent row owns the referenced field. Canyon turns that relationship into lookup methods on the child entity. It does **not** create the database constraint; define that in your schema.

Suppose `Player.team_id` refers to `Team.id`:

```rust
use canyon_sql::macros::{canyon_entity, CanyonMapper, Crud, Fields};

#[derive(Debug, Fields, Crud, CanyonMapper)]
#[canyon_entity(table_name = "teams")]
pub struct Team {
    #[primary_key]
    pub id: i64,
    pub name: String,
}

#[derive(Debug, Fields, Crud, CanyonMapper)]
#[canyon_entity(table_name = "players")]
pub struct Player {
    #[primary_key]
    pub id: i64,
    #[foreign_key(references = Team::id)]
    pub team_id: i64,
    pub name: String,
}
```

The child needs `Read` (or `Crud`, which includes it); the parent needs Canyon mapping metadata. The annotation points to a Rust entity and field, not to an arbitrary table string. From `team_id`, Canyon derives the relation name `team` and generates four inherent methods:

```rust
let parent: Option<Team> = player.find_team().await?;
let children: Vec<Player> = Player::find_all_by_team(&team).await?;

let parent_on_other_db = player.find_team_with("reporting").await?;
let children_on_other_db =
    Player::find_all_by_team_with(&team, "reporting").await?;
```

No matching parent is `Ok(None)`; no matching children is `Ok(vec![])`. A failed query or mapping operation remains an error. The `_with` variants also accept a compatible connection.

The referenced field need not be the parent's primary key, but it should identify the parent as your schema intends—normally through a unique constraint. A fully qualified path works too:

```rust
#[foreign_key(references = crate::models::Team::external_id)]
pub team_external_id: i64,
```

Canyon takes the parent's physical table name and schema from its `#[canyon_entity(...)]` metadata. A type such as `TournamentDetails` maps to `tournament_details` by default; a custom physical name is equally valid. Keep the Rust relationship annotation and the actual database constraint consistent. The [relationship integration tests](https://github.com/zerodaycode/Canyon-SQL/blob/main/tests/crud/foreign_key_operations.rs) cover both naming cases on PostgreSQL, MySQL, and SQL Server.
