# The database configuration

Before we can jump to write code as crazy devs, we need to tell the database about our intentions.

The database needs users, permissions, roles, databases, schemas, tables, columns... woof! 
Too much crazy things.

But, sticking with past chapter, we will guide you (if you don't know) on how to set up the 
database to start to work with.


## Creating a database an a user

When you install for the first time a new database backend, during the installation always requires
you to create an *admin* user to manage the database. This let's you to have a `super-user` to work
with the database, but, for obviously reasons, isn't recommended to use the *admin* user for almost
nothing (at least, talking in production).

`Note: if you only want to test code or tools to work in development stage, it's fine to go ahead an
use that `admin` to quickly set up an environment where you can work.`

But, being consistent and trying to stick always with the best possible practices, we are going to create a new user with it's password, and a new database to work with the examples *in this book*.

So, open a command prompt, or your favourite GUI database manager application. Let's determine a few
constraints, for simplycity and for clarifiying this guide, we will use the next `credentials` and 
`database` to work with.

```
username = canyon_user
password = canyon_pass
db_name = triforce
```

`NOTE: Why `triforce`? You will find all the details following [this link](https://zerodaycode.github.io/canyon-book/real_world_example.html)`

Type secuentially the following commands on your command prompt or database manager:

### The commands

```
DROP database IF EXISTS triforce;

DROP USER IF EXISTS canyon_user;

CREATE ROLE canyon_user WITH LOGIN SUPERUSER PASSWORD 'canyon_pass';

CREATE DATABASE triforce WITH TEMPLATE = template0 ENCODING = 'UTF8' LOCALE = 'es_ES.UTF-8';

ALTER DATABASE triforce OWNER TO canyon_user;
```

And voilá! You have configured your database server to be able to work with a new user `canyon-user`
that it's not the `admin` user, having it's own permissions and roles. 


## Creating some data to work with

We need to have some tables inside our new `triforce` database to store information. Usually, the database tables are called `entities`, which will become a powerful concept in `Canyon`. 

An `entity` usually represents a *real world* object, like a Car, a Country or a User. Those entities can have multiple relations (or not) between them, and that's the reason because the `SQL` language it's colloquially knows as a `relational database language`.

`NOTE: More advanced `Canyon` features that will be presented later are able to create tables and columns for you, directly from `Rust` structs, but we are growing from the more basic concepts to the more advanced ones`

So, let's gonna create a couple of tables to have something to work.

As usual, type the following commands in your command prompt or in your database manager:

### PostgreSQL - syntax example
```
\c triforce

CREATE TABLE public.league (
	id					INTEGER PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
	ext_id				BIGINT NOT NULL,
	slug				TEXT NOT NULL,
	name				TEXT NOT NULL,
	region				TEXT NOT NULL,
	image_url			TEXT NOT NULL
);
CREATE TABLE public.tournament (
	id					INTEGER PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
	ext_id				BIGINT NOT NULL,
	slug				TEXT NOT NULL,
	start_date			DATE NOT NULL,
	end_date			DATE NOT NULL,
	league				INTEGER REFERENCES league(id)
);

CREATE TABLE public.player (
    id					INTEGER PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    ext_id				BIGINT NOT NULL,
    first_name			TEXT NOT NULL,
    last_name			TEXT NOT NULL,
    summoner_name		TEXT NOT NULL,
    image_url			TEXT,
    role				TEXT NOT NULL
);

CREATE TABLE public.team (
    id					INTEGER PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    ext_id				BIGINT NOT NULL,
    slug				TEXT NOT NULL,
    name				TEXT NOT NULL,
    code				TEXT NOT NULL,
    image_url			TEXT NOT NULL,
    alt_image_url		TEXT,
    bg_image_url		TEXT,
    home_league		    INTEGER REFERENCES league(id)
);

CREATE TABLE public.team_player (
    id					INTEGER PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    team_id			    INTEGER REFERENCES team(id) ON DELETE CASCADE,
    player_id			INTEGER REFERENCES player(id) ON DELETE CASCADE
);

ALTER TABLE public.league OWNER TO triforce;
ALTER TABLE public.tournament OWNER TO triforce;
ALTER TABLE public.player OWNER TO triforce;
ALTER TABLE public.team OWNER TO triforce;
ALTER TABLE public.team_player OWNER TO triforce;
```

were we just basically created some *tables* that represents some entities and it's relations
between them, with it's columns, to store concrete data in there. 

Finally, we asked to assign our `canyon_user` as the owner of the tables, in order to have
all the necessary permissions to work with the tables.


## In recap

In this chapter, we setted up a `Rust` project to work with `Canyon`. We installed and configured
a database engine to work with, created an user and a database with some tables to start having fun
querying them with `Canyon`.

In the next chapter, we will write the first lines of `Rust` code to work with the already configured database, presenting the most basic operations but yet powerful available with `Canyon`, the
classical `CRUD` operations.