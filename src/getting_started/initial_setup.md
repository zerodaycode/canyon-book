# Install Canyon

Add Canyon to your application's `Cargo.toml`. Choose at least one SQL backend; we'll use PostgreSQL:

```toml
[dependencies]
canyon_sql = { version = "0.5.1", features = ["postgres"] }
tokio = { version = "1", features = ["macros", "rt-multi-thread"] }
```

The other choices are `mysql` and `mssql`. If one application talks to several kinds of database, enable more than one:

```toml
canyon_sql = { version = "0.5.1", features = ["postgres", "mysql", "mssql"] }
```

> **At least one backend is required.** Canyon has no default SQL backend; omitting all three features produces a compile-time error.

## Start the runtime and Canyon

The explicit startup form gives your application a chance to report a configuration or connection error:

```rust
use canyon_sql::{core::Canyon, CanyonResult};

#[tokio::main]
async fn main() -> CanyonResult<()> {
    Canyon::init().await?;
    // Queries can run here.
    Ok(())
}
```

`Canyon::init()` reads the configuration and opens the pools. Calling it again does not replace an initialized instance. Run it before a generated query method.

> **A shorter `main`:** `#[canyon_sql::main]` creates the Tokio runtime and initializes Canyon before the function body. Use it only on a function named `main`. Unlike the explicit form above, it panics if initialization fails.

Next, [configure a datasource](./the_configuration_file.md) and [prepare a table](./configuring_the_database.md) for the first query.
