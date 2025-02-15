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
select * from my_big_table; -- my big, heavy, and/or complex query
```

> ERROR: canceling statement due to statement timeout

### Configuration Options

The `statement_timeout` parameter can be set at the following levels:

1. At the PostgreSQL Instance Configuration

    This is set as an installation parameter in the `postgresql.conf` file. This value becomes a default for all databases, all accounts, all sessions, and all transactions in it.

2. At the Database Level

    Use `alter database <database> set statement_timeout = 60000` to set the default timeout for one database.

3. At the Account Level

    Use `alter role <account> set statement_timeout = 60000` to set the default timeout for one account that connects to the database.

4. At the Session Level

    Use `set statement_timeout = 60000` SQL query to set timeout for the current session. This is equivalent
    to set this parameter in the connection string, as in `jdbc:postgresql://10.26.56.200:5432/mydatabase?statement_timeout=60000`.

5. At the Transaction Level

    Use `set local statement_timeout = 60000` SQL query to set the timeout for the current transaction only. It
resets to the default settings after commit/rollback. Remember to turn off auto-commit for this to
work.

**Note**: A more fine grained overrides the more general settings, that act as a default behavior.

**Note**: Typically this parameter is not set at the installation or database level, but per account. Different accounts can differentiate between online queries that should have a relatively short timeout compared to batch jobs that could have a much higher timeout.











