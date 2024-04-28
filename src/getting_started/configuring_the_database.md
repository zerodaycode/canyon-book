# Setting up a Working Environment

To test the functionality of `Canyon-SQL`, it is necessary to have a working database with some data stored in it.

One quick way to set up such an environment is by using Docker. Docker is an open-source project that automates the
deployment of applications as portable, self-sufficient containers that can run almost anywhere.

For those who are not yet familiar with Docker, official documentation is available [here](https://www.docker.com/),
along with installers for every supported platform.

## Creating a DataBase container

Assuming your `Docker` environment is ready, the next step is to create a container with a supported database
installation.

To accomplish this, we provide an example of a `docker-compose` file located in the [scripts folder](../../scripts) in
the root of this repository. You can use this file to create a new container and populate it with data.

The file contains information about a `PostgreSQL`, `SQLServer` and `MySql` containers to start.

In the same folder, you will also find an sql folder containing some SQL scripts. These scripts will be automatically
detected by the docker-compose file and used to fill the tables of the examples.

> Please note that we assume you have already installed `docker-compose` and copied the scripts folder into your
> project. Adjust the paths according to your preferences.


Build and start containers by running:

```bash
docker-compose -f ./scripts/docker-compose.yml up
```

Lastly, you will need to create a `canyon.toml` file as mentioned in the previous chapter. You can use the following
snippet on it:

```toml
[canyon_sql]
datasources = [
    { name = 'PostgresDS', auth = { postgresql = { basic = { username = "postgres", password = "postgres" } } }, properties.host = 'localhost', properties.db_name = 'triforce', properties.migrations = 'enabled' },
    { name = 'SqlServerDS', auth = { sqlserver = { basic = { username = "sa", password = "SqlServer-10" } } }, properties.host = '192.168.0.250.1', properties.port = 3340, properties.db_name = 'triforce2', properties.migrations = 'disabled' },
    { name = 'MysqlDS', auth = { mysql = { basic = { username = "root", password = "root" } } }, properties.host = '192.168.0.250.1', properties.port = 3340, properties.db_name = 'triforce2', properties.migrations = 'disabled' }
]
```
> Note: In `SqlServer` the data will not be loaded automatically, user interaction is required.
> Note: Please ensure that the inlined tables are correctly formatted without any line breaks.
> Next time the code is compiled, Canyon will:

- Connect to the databases;
- Create the tables if needed;
- Populate each table if needed;

Now it is time to start writing the code...