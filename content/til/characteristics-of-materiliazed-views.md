+++
title = "Characteristics of Materiliazed Views"
date = 2022-09-29
updated = 2023-11-02
type = "post"
description = "Some pointers on when to use them and when to not"
in_search_index = true
[taxonomies]
TIL-Tags = ["database"]
+++

A Materiliazed View (MV) is a result of an editable SQL query (on an exisitng table) stored on disk. We can execute SQL queries on a MV to get information faster.

You can list all the MVs present along with their defining queries on PostgreSQL like [this](https://dataedo.com/kb/query/postgresql/list-materialized-views):
```sql
select schemaname as schema_name,
       matviewname as view_name,
       matviewowner as owner,
       ispopulated as is_populated,
       definition
from pg_matviews
order by schema_name,
         view_name;
```

### Characteristics:
1. They are best used when a complex query (joins on multiple tables) has to be
   executed frequently on not so frequently changing data. Refreshing a MV every time (or every 5 minutes) defeats the purpose of using a MV.
   1. In contrast, a View is used when a simple query (joins on 1 or 2 tables) has to be executed frequently on not so frequently changing data. A View is just a named query and the query is executed every time the View is called. The result you get is from the actual tables. [reference](https://stackoverflow.com/a/43053443/10524266)
2. They are refreshable. Meaning, the (complex) query is executed again on the
   underlying tables to reflect the updates.
3. You can create an MV on an exisitng MV.


### Limitations: 
1. Refreshing a MV would mean we are polling the underlying DB/tables for every period of time. Polling adds a load onto the underlying (master) DB. Lesser the refresh period, more the load on the DB.
2. When we refresh a MV, it would block all select queries on the MV during that period. While there is an option to do the refresh concurrently (does not block queries), it would put additional load on our machines.
3. If there is an ORDER BY clause in the materialized view's defining query, the original contents of the materialized view will be ordered that way, but `REFRESH MATERIALIZED VIEW` does not guarantee to preserve that ordering.
4. You cannot use {% codehighlight() %}JOIN{% end %} between two MVs or between a MV and a table.
5. When we refresh a MV, we won’t be getting a history of all the changes that a datapoint has went through in the underlying tables or even how many times did a datapoint get updated since its last MV state. The datapoint might’ve even be deleted.