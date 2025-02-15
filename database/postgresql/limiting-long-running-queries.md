# Limiting Long Running Queries in PostgreSQL

By default the PostgreSQL database doesn't limit the execution time of a query. Even if the client
session is disconnected the query continues to run for good or bad.

## 1. Canceling Long Running Queries

To find out the list of currently running queries sorted by duration you can use:

```sql
select now() - query_start as duration, *
from pg_stat_activity
where state = 'active'
order by query_start;
```

Now, if you find a query that is running for too long and should be canceled you can do so using the PID from the previous query as shown below:

```sql
select pg_cancel_backend(<pid>);
```

> canceling statement due to user request

## 2. Preventing Long Running Queries to Happen

Instead of actively canceling long running queries you can limit the time queries can run per database, per session, or even  per transaction.

For example, to set a timeout of 60 seconds for all queries in a datasource you can add the parameter `statement_timeout=60000` to the JDBC URL. An example URL could look like:

```
jdbc:postgresql://10.26.56.200:5432/mydatabase?statement_timeout=60000
```

All queries within sessions started in this datasource will stop after 60 seconds of execution time. For example, the following long running query will automatically stop after 60 seconds with an error:

```sql
select * from my_big_table;
```

> ERROR: canceling statement due to statement timeout

The datasource setting acts as a default for all running queries using the connection. If a specific query needs more time to run you can change the default using the `SET` clause.

The following settings will affect the connection settings temporarily or permanently.

| Command | Description |
| -- | -- |
| `set local statement_timeout = 60000` | Sets a 60-second query timeout within the current transaction. It resets to the default settings after commit/rollback. Remember to turn off auto-commit for this to work |
| `set statement_timeout = 60000` | Sets a 60-second query timeout to the whole session |
| `set statement_timeout = DEFAULT` | Resets the session timeout to the session's default value |

