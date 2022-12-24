# The minimum requirements

`Canyon` it's a database manager, an ORM, a querybuilder... and your best tool to construct
you persistence layer without pain.

But, as obvious, the framework only will work after a minimum set-up.

## Supported database engines

For now, `Canyon` supports the following database engines to work with:
    - PostgreSQL
    - MSSQL (SqlServer)

The development teams is planning to add support for more database engines, but at this time, the available ones are the listed above.
For this tutorial, we will be using our most beloved one: `PostgreSQL`.

## Download your engine from internet

You must have an active installation of `PostgreSQL`. If you never work with it, you can download
and install it from [here](https://www.postgresql.org/download/).

Choose your operating system and follow the installation program. If you're working on Linux, you will find a tutorial for your distro just by searching the topic *postgresql installation guide on [my-linux-distro]* on the web.

If you're a more experienced user, or you already have installed `PostgreSQL`, just omit this part.

## Add Canyon-SQL as a dependency to your project

The next thing it's code. YES! So, as usual, in `Rust`, you will generate a new project with the invocation of the command `cargo new --project-name`.
If you already have a project made, and you desires to add `Canyon` to your project, the next step for
the two points of view are the same: Add Canyon to your *Cargo.toml* file

So, we just need to add the following lines:

```toml
# Cargo.toml

# More dependencies and configurations

canyon_sql = 0.1.0^
```

This will notify `Cargo` that he must bring to your project `Canyon` in the version `0.1.0` or higher. You can replace this for a concrete version that you desire, but we are talking in general terms to
make the very basic explanations, so it will be fine for the moment.

### Directly from GitHub

If you prefer, you can ask `Cargo` to fetch the code directly from the GitHub repo and have the latest
available version in whatever branch you want.

```toml
# Cargo toml.file


[dependencies]
canyon_sql = { git = "https://github.com/zerodaycode/Canyon-SQL" }

```

You could also desire to have the source code from an specific branch, for example, the *development* brach:

```toml
[dependencies]
canyon_sql = { git = "https://github.com/zerodaycode/Canyon-SQL", branch = "development" }
```

## Check the project

`Cargo` automatically do everything for you, so, from the root of your project, run `cargo build` and you will see a very similar error:

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

This error it's telling us that `Canyon-SQL` needs the `canyon.toml` file in order to be built properly.
We are ready to know how `Canyon-SQL` handles the connection with the databases.

Yay! Let's go ahead and solve the error in the next chapter.
