# Initial Setup

- [Initial Setup](#initial-setup)
  - [Minimum Requirements](#minimum-requirements)
  - [Supported database engines](#supported-database-engines)
  - [Downloading the PostgreSQL engine](#downloading-the-postgresql-engine)
  - [Add Canyon-SQL as a dependency](#add-canyon-sql-as-a-dependency)
    - [Fetching from Github](#fetching-from-github)
  - [Building the project](#building-the-project)

---

## Minimum Requirements

`Canyon` is a versatile tool that combines a database manager, ORM, and query builder to create your persistence layer with ease. However, before using the framework, a minimum setup is required.

---

## Supported database engines

At present, Canyon supports the following database engines:

 - PostgreSQL
 - MSSQL (SqlServer)
 - MySql

The development team is planning to add support for additional database engines in the future. For the purposes of this tutorial, we will be using `PostgreSQL`.

---

## Downloading the PostgreSQL engine

To use Canyon with PostgreSQL, an active installation of PostgreSQL is required.  If you are an experienced user and have already installed PostgreSQL, you can skip this section.

Installation and instructions can be found in the [official website](https://www.postgresql.org/download/). Choose your operation system. If using Linux, more instructions can be found for each specific distro by searching in the web.

---

## Add Canyon-SQL as a dependency

To create a new Rust project from scratch. Run:

```bash
# Replace project-name with the project name
cargo new project-name
```

In the existing project, open `Cargo.toml` to include the new dependency.

```toml
# Cargo.toml

# More dependencies and configurations

canyon_sql = 0.1.0^
```

Compiling will now download `canyon_sql` to the project. The `^` symbol tells the compiler to use version "`0.1.0` or higher".

```bash
cargo build
```

### Fetching from Github

Cargo can fetch the latest updates directly from the Canyon-SQL GitHub repository. These will include the latest changes that aren't included in a specific release.

Include the following lines to `Cargo.toml`:

```toml
# Cargo.toml

[dependencies]
canyon_sql = { git = "https://github.com/zerodaycode/Canyon-SQL" }
```

To fetch from a specific branch, such as the development branch, specify as follows:

```toml
# Cargo.toml

[dependencies]
canyon_sql = { git = "https://github.com/zerodaycode/Canyon-SQL", branch = "development" }
```

---

## Building the project

After including Canyon-SQL as a dependency in `Cargo.toml`. The crate can be built with:

```bash
# When in project root
cargo build
```

If everything is set up correctly, the build process will complete successfully. However, if the `canyon.toml` file is not found, the following error will appear:

```rust
error: custom attribute panicked
  --> src\main.rs:17:1
   |
17 | #[canyon]
   | ^^^^^^^^^
   |
   = help: message:

           No file --> canyon.toml <-- founded on the root of this project.
           Please, ensure that you created one .toml file with the necesary properties needed in order to connect to the database.

           : Os { code: 2, kind: NotFound, message: "The system can't found the specified file." }

    ...
```

A file named `canyon.toml` is required to run `Canyon-SQL`. It should be located in the root directory of the project. `canyon.toml` contains the necessary properties required to connect to the database. Next chapter will explain how to configure it.