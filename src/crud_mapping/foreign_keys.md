# Relationships

A player belongs to a team: the `players` table holds `team_id`, which points at a team. Canyon can generate lookups in both directions from that relationship. The database still needs its own foreign-key constraint if you want it enforced.

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

The `references` path names a Rust type and field, not a SQL table string. `Player` needs `Read` (included in `Crud`), and `Team` needs mapping metadata. Canyon derives the name `team` from `team_id` and gives `Player` four methods:

```rust
let parent: Option<Team> = player.find_team().await?;
let children: Vec<Player> = Player::find_all_by_team(&team).await?;

let parent_on_other_db = player.find_team_with("reporting").await?;
let children_on_other_db =
    Player::find_all_by_team_with(&team, "reporting").await?;
```

The return type follows the direction of the lookup:

- `player.find_team()` looks for one parent: `Ok(None)` means the lookup found none.
- `Player::find_all_by_team(&team)` looks for children: `Ok(vec![])` means there are none.

A query or mapping failure is an error in either direction. The `_with` variants also accept a compatible connection.

The referenced field can be something other than the parent's primary key. In that case, make sure your schema identifies a parent unambiguously—usually with a unique constraint. A fully qualified Rust path also works:

```rust
#[foreign_key(references = crate::models::Team::external_id)]
pub team_external_id: i64,
```

Canyon reads the parent's physical table name and schema from its entity metadata. `TournamentDetails` maps to `tournament_details` by default, but an explicit `table_name` works too. Keep that metadata, the Rust relationship, and the database constraint in agreement. The [relationship tests](https://github.com/zerodaycode/Canyon-SQL/blob/main/tests/crud/foreign_key_operations.rs) exercise default and custom names on all three backends.
