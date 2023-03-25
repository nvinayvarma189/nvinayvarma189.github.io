+++
title = "Timeouts in Postgres RDS"
date = 2023-03-25
updated = 2023-03-25
type = "post"
description = "How to trouble shoot different kind of time outs"
in_search_index = true
[taxonomies]
TIL-Tags = ["database"]
+++

When you are running some long running (SELECT or UPDATE) queries on the read replica (or slave) of a PostgresDB (even AWS RDS Postgres), sometimes you may run into the below error:

```
canceling statement due to conflict with recovery
```

I misunderstood this as statement timeout and was looking to optimise the query. While that is one of the options to counter this problem, the problem itself is not coming because of a statement timeout. This is actually because of one of the properties of [MVCC in Postgres](https://www.postgresql.org/docs/7.1/mvcc.html). 

## What causes this issue?

Imagine a Write statement is executed on a table in the master Postgresinstance when there is a running select query on the same table in the read replica instance, there are two options:

1. Wait for the SELECT statement to be finished before applying the WAL record. In this case, the replication lag increases.

2. Apply the WAL record, and then cancel the SELECT statement. In this case, you get the error "canceling statement due to conflict with recovery".

You get this error typically due to long running queries on the read replica.

[Reference](https://repost.aws/knowledge-center/rds-postgresql-error-conflict-recovery)

## How to get around statement timeout?

The above is different from a regular statement timeout. Each Postgres instance has a default timeout of 30 seconds (if you are using Postgresversion < 9.3). Post 9.3 version, the default statement timeout is set to 0 seconds (which means there is no time limit).

1. You can configure the `statement_timeout` parameter in the `postgresql.conf` file. However, this will be applied to all connections and all Postgresusers. This can cause long standing resource intensive queries slowing down your DB.
2. You can configure the timeout just for your session. You can do it with psql like this:
    ```sql
        psql "postgresql://username:passwrod@hostname:port/db_name"
    ```
    and then in the session shell: 
    ```sql
        psql (12.9 (Ubuntu 12.9-0ubuntu0.20.04.1), server 12.11)
        SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
        Type "help" for help.

        db_name=> set statement_timeout=2000;
        SET
    ```
    Now in this session, all the subsequent queries will have a timeout of 2000 milliseconds (2 seconds).

3. You can do the same from Python like this:
    ```python
        import psycopg2

        # Establish a connection to the database
        conn = psycopg2.connect(
            host="your_host_name",
            port="your_port_number",
            database="your_database_name",
            user="your_user_name",
            password="your_password"
        )

        # Open a cursor to perform database operations
        cur = conn.cursor()

        # Set the statement timeout to 2 seconds
        cur.execute("SET statement_timeout = '2000'")

        # Execute a SELECT query
        cur.execute("SELECT * FROM your_table_name")

        # Fetch all the results from the query
        rows = cur.fetchall()

        # Print out the results
        for row in rows:
            print(row)

        # Close the cursor and connection to the database
        cur.close()
        conn.close()
    ```


### Misc
Below is a query that you can use to get statistics on how many times statement conflict between master and read replica has occurred.

```sql
select
    datname as "db",
    confl_snapshot as "queries_cancelled"    
from pg_stat_database_conflicts
where 
    confl_snapshot > 0
order by 
    confl_snapshot desc
```