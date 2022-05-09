# The minimum requirements

`Canyon` it's a database manager, an ORM, a querybuilder... and your best tool to construct
you persistence layer without pain.

But, as obvious, the framework only will work after a minimum set-up. 


## Supported database engines

For now, `Canyon` supports the following database engines to work with:
    - PostgreSQL

Ok, we know. This is a list of only one element. But don't worry. We are prettending to increase that list size adding support for more engines, but for know, we will focus on the supported one: `PostgreSQL`.


## Download your engine from internet

You must have an active installation of `PostgreSQL`. If you never work with it, you can download
and install it from [here](https://www.postgresql.org/download/).

Choose your operating system and follow the installation program. If you're working on Linux, you will find a tutorial for your distro just by searching the topic *postgresql installation guide on [my-linux-distro]* on the web.

If you're a more experienced user, or you already have installed `PostgreSQL`, just omit this part. 


## Add Canyon-SQL as a dependency to your project.

The next thing it's code. YES! So, as usual, in `Rust`, you will generate a new project with the invocation of the command `cargo new --project-name`.
If you already have a project made, and you desires to add `Canyon` to your project, the next step for
the two points of view are the same: Add Canyon to your *Cargo.toml* file

So, we just need to add the following lines:
```
# Cargo.toml

# More dependencies and configurations

canyon_sql = 1.0.0^
```

This will notify `Cargo` that he must bring to your project `Canyon` in the version `1.0.0` or higher.. You can replace this for a concrete version that you desire, but we are talking in general terms to 
make the very basic explanations, so it will be fine for the moment. 

### Directly from GitHub
If you prefer, you can ask `Cargo` to fetch the code directly from the GitHub repo and have the latest
available version in whatever branch you want. 

```
# Cargo toml.file


[dependencies]
canyon_sql = { git = "https://github.com/zerodaycode/Canyon-SQL" }

```

If you prefer to have the source code from an specific branch, for example, the *development* brach:
```
[dependencies]
canyon_sql = { git = "https://github.com/zerodaycode/Canyon-SQL", branch = "development" }
```


## Check the project
`Cargo` automatically do everything for you, so, from the root of the project, run `cargo build` and you will see the next error: 

```
error: custom attribute panicked
  --> src\main.rs:17:1
   |
17 | #[canyon]
   | ^^^^^^^^^
   |
   = help: message:

           No file --> secrets.toml <-- founded on the root of this project.
           Please, ensure that you created one .toml file with the necesary properties needed in order to connect to the database.

           : Os { code: 2, kind: NotFound, message: "The system can't found the specified file." }

    ...
```

This error it's telling us that `Canyon` needs the `secrets.toml` file in order to be built properly.
We are ready to know how `Canyon` handles the connection with the database at the user credentials
level.

So, stuck with us, and We'll solve the error in the next chapter.


