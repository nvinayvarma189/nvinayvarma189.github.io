+++
title = "Explain Analyze"
date = 2022-09-25
updated = 2022-09-25
type = "post"
description = "Profiling SQL queries on Postgres"
in_search_index = true
[taxonomies]
TIL-Tags = ["SQL"]
+++

Let's say we have this query [1]

```sql
select name, artist, text
from card
where to_tsvector(name || ' ' || artist || ' ' || text) @@ to_tsquery('Avon');
```
If we want to make this run faster, we would probably think of creating a new column which helps us save some computation time (especially if the number of rows is huge)

So we create a new column like this

```sql
ALTER TABLE card
  ADD COLUMN document tsvector;
update card
set document = to_tsvector(name || ' ' || artist || ' ' || text);
```

Now you can just query the document column [2]

```sql
select name, artist, text
from card
where document @@ to_tsquery('Avon');
```

Now, How do we knw how much improvement does [2] get us over [1]? Enter **`explain analyze`**

```sql
explain analyze select name, artist, text
                from card
                where document @@ to_tsquery('Avon');
explain analyze select name, artist, text
                from card
                where document_with_idx @@ to_tsquery('Avon');
```
You will be presented with the time it took to execute both the queries.

**Reference:** [Full Text Search PostgreSQL](https://www.youtube.com/watch?v=szfUbzsKvtE)