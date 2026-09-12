# Setting up a PostgreSQL Database in a Container

These are simple instructions to set up a database in a docker container. It's tailored for a PostgreSQL database, but the steps can be modified to run other databases.

The database we'll create have the following settings:

- PostgreSQL version: `16.10`
- PostgreSQL port: `5416`
- PostgreSQL super user password: `rootroot`
- Database Name: `data`
- Service Account Username: `user1`
- Service Account Password: `pass1`

**Note**: These values are used in the commands below. Change accordingly to your needs.

## 1. Create the Container

```bash
docker create --name postgresql16.10 -e POSTGRES_PASSWORD=rootroot -p 5416:5432 postgres:16.10
```

The container is created in stopped state.

## 2. Start the Container

```bash
docker start postgresql16.10
```

That's it! The database engine is now up and running and listening on port 5416. However, there's no database or service account created yet. Let's do that now.

## 3. Create the Database and Service Account

Create the database `data` with the service account `user1`/`pass1`:

```bash
docker exec -it postgresql16.10 /bin/bash

su -  postgres
psql

create user user1 password 'pass1';
create database data with owner user1 template template0;
revoke connect on database data from public;
quit
exit
exit
```

That's it! The database `data` is now created, and the service account is ready. For example, you can now connect from Java using the URL:

```
jdbc:postgresql://<host>:5416/data
```


