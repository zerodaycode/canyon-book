# Test and contribute

There are two useful levels of tests when working on Canyon itself:

- **Unit and compile tests** check Rust behavior and generated APIs without starting a database.
- **Integration tests** send queries to PostgreSQL, MySQL, and SQL Server. They need the repository's database fixtures.

From a checkout of the [Canyon-SQL source repository](https://github.com/zerodaycode/Canyon-SQL), run the first level with:

```sh
cargo fmt --all -- --check
cargo test --workspace --lib --all-features
cargo test -p tests --test compile_tests --features postgres
```

At least one backend feature is required. A bare `cargo test --workspace` produces Canyon's intentional compile-time diagnostic about the missing backend.

Integration tests often put `#[canyon_sql::macros::canyon_tokio_test]` on a synchronous `fn`. The macro creates a test, starts Canyon's Tokio runtime, initializes datasources, and runs the body asynchronously. An initialization or body error fails the test; the databases still need to be running.

Start the Docker fixtures before running the integration suite. PostgreSQL and MySQL load their test data at startup. SQL Server needs its ignored initializer:

```sh
docker compose -f docker/docker-compose.yml up -d --wait
cargo test -p tests --test canyon_integration_tests --all-features \
  initialize_sql_server_docker_instance -- --ignored --test-threads=1
cargo test --workspace --all-features --no-fail-fast -- --test-threads=1
```

The integration suite changes database state, so `--test-threads=1` helps keep runs predictable. The relationship tests prepare an idempotent fixture of their own, even when a Docker volume already exists. To check that the integration binary compiles without running it:

```sh
cargo test -p tests --test canyon_integration_tests --all-features --no-run
```

When fixing a bug, test it where it failed. Put generated-Rust regressions in `tests/ui`; put SQL behavior in the backend integration suite. [CONTRIBUTING.md](https://github.com/zerodaycode/Canyon-SQL/blob/main/CONTRIBUTING.md) covers the repository workflow.
