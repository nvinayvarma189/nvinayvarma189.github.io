+++
title = "SQLAlchemy with PG Bouncer"
date = 2024-08-08
updated = 2024-08-08
type = "post"
description = "The default pool settings of SQLAlchemy and PG Bouncer are incompatible"
in_search_index = true
[taxonomies]
TIL-Tags = ["sqlalchemy", "postgres"]
+++

Sqlalachemy (an orm) comes with a connection pooler. This is a client side connection pooler. When we scale our app, we will increases the number of instances of the backend. Which means, each each new instance of backend is creating its own connection pool (which is ultimately creating new processes on postgres server).

This is why you need pg_bouncer. Deploy a pg_bouncer instance and make all of your backend instances connect to the pg_bouncer instance. This will make sure that the db is not getting overloaded with new processes as your backend scales (you just have to adjust your max_connections parameter in pg_bouncer).

But but but, especially when you combine sqlalchemy and pg_bouncer, you can run into issues because there are two connection pools interacting. The default connection pool type of sqlalchemy (QueuePool) is not compatible with pg_bouncer. It is best to disable the connection pooler inside sqlalchemy (by setting it to null pool) and use the connection pooler that pg_bouncer provides.

Good Practices:

1. Add `?application_name={consts.SERVICE_NAME}-{env}` to the postgres connection string. This helps you track how many connections are actually being opened and closed and beind left in idle state as you make requests to your app locally or in an deployed environment
2. Use NullPool from sqlalchemy.pool. This will disable the connection pooler inside sqlalchemy and use the connection pooler that pg_bouncer provides.

```python
from sqlalchemy import create_engine, pool

# A connection string to the pgbouncer instance which in turn connects to the actual postgre    
engine = create_engine("postgresql://user:pass@host:port/db", poolclass=pool.NullPool)
```

References:
[https://pablomarti.dev/sqlalchemy-pgbouncer/](https://pablomarti.dev/sqlalchemy-pgbouncer/)