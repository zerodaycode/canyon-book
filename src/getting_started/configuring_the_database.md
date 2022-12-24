# Creating a working environment

In order to test the `Canyon-SQL` funcionalities, we need to have a working database with some data stored in to have a little fun.

The quickest way of getting such environment is by using `Docker`.
Docker is an open-source project for automating the deployment of applications as portable, self-sufficient containers that can run almost everywhere.

For those who are not familiar with Docker yet, you can find it's official documentation [here](https://www.docker.com/), along with the installers for every supported platform.

## Creating a PostgreSQL container

Now, we will assume that your `Docker` environment is ready to start have some fun, next step will
be creating a container with a `PostgreSQL` installation.

This isn't a `Docker` tutorial, so we will provide you an example of a `docker-compose` file at the [scripts folder](../../scripts) that is located at the root of this repository, so you can use that to create a new container and populated with data.

There's in the same folder an sql folder that contains some `SQL` scripts, that will be automatically detected by the `docker-compose` file and will be used to fill the tables of the examples.

> Note: We are assuming that you have docker-compose already installed, and that you copied the scripts folder into your project. Adecuate the paths according your desires.

You only need to run:

```bash
docker-compose -f ./scripts/docker-compose.yml up
```

Finally, as you remember from the last chapter, we will need a `canyon.toml` file. You can use the following snippet and place it inside of such file:

```toml
[canyon_sql]
datasources = [
    {name = 'postgres_docker', properties.db_type = 'postgresql', properties.username = 'postgres', properties.password = 'postgres', properties.host = 'localhost', properties.port = 5438, properties.db_name = 'postgres'}
]
```

> Note: Remember about formatting correctly the inlined tables, without not jump lines, if you copy the snippet

Now, everything will start to run, creating a `PostgreSQL` database, creating the tables and populating them.
When everything finishes, you will be ready to start to write `Canyon-SQL` *Rust* code, and start to finally have fun!
