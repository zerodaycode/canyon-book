# Test and contribute

Canyon's unit and compile tests catch API errors without starting a database. From a checkout of the [source repository](https://github.com/zerodaycode/Canyon-SQL):

```sh
cargo fmt --all -- --check
cargo test --workspace --lib --all-features
cargo test -p tests --test compile_tests --features postgres
```

At least one SQL backend feature is required. A bare `cargo test --workspace` is expected to fail with Canyon's explicit compile-time backend diagnostic; it is not the command for testing all supported engines.

The source integration tests commonly use `#[canyon_sql::macros::canyon_tokio_test]` on a synchronous `fn`. The macro creates a test, starts Canyon's Tokio runtime, initializes the configured datasources, and runs the body as async code. Initialization or a returned body error fails the test. It is useful for Canyon's own tests, but it still needs the relevant database fixtures to be available.

The integration tests use the Docker setup in `docker/docker-compose.yml`. PostgreSQL and MySQL load their test data at container startup. The SQL Server fixture needs its ignored initializer:

```sh
docker compose -f docker/docker-compose.yml up -d --wait
cargo test -p tests --test canyon_integration_tests --all-features \
  initialize_sql_server_docker_instance -- --ignored --test-threads=1
cargo test --workspace --all-features --no-fail-fast -- --test-threads=1
```

The `--test-threads=1` choice makes the stateful integration suite easier to reason about. The relationship tests also prepare their own idempotent fixture so persisted Docker volumes do not depend on an entrypoint script running again. You may compile the integration binary without connecting to any database:

```sh
cargo test -p tests --test canyon_integration_tests --all-features --no-run
```

When contributing, add a regression test at the layer of the failure. Generated Rust syntax belongs in the compile fixtures under `tests/ui`; SQL behavior belongs in backend integration tests. See [CONTRIBUTING.md](https://github.com/zerodaycode/Canyon-SQL/blob/main/CONTRIBUTING.md) for the repository workflow.
