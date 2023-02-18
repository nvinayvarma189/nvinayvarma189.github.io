+++
title = "Sync data between two k8s clusters"
date = 2023-02-19
updated = 2023-02-19
type = "post"
description = "Some useful commands and notes on how to sync data between two DBs of different clusters"
in_search_index = true
[taxonomies]
TIL-Tags = ["data-pipelines"]
+++

**Task:** You have to sync data from a table in a DB from one k8s cluster to a table (of the same schema) in a DB from another k8s cluster.

**Usecase:** When you want to have the pre-production environment (commonly known as the Staging environment) have real production data so that you can test your code/config changes in the pre-production environment before actually pushing your changes to production.

Why is it important? Sometimes unit tests are not enough as the production data usually has some quirks that we fail to expect/guess when mocking the dependencies.

## Levels of Sophistication

### Level 1

Let's say production DB is running on 5432 port and Pre-production DB is running on 5433 port on your local machine. You can achieve this by SSH tunneling.

The below command will export the data from a table called `flows` in a postgres DB running on 5432 port as insert statements into a SQL file.

```
pg_dump --column-inserts --data-only -h localhost -p 5432 -U postgres -d db_name -t flows >  flows.sql
```

You can then load this data into the pre-production DB running at 5433 with the below command

```
psql -h localhost -U psotgres -p 5433 -d db_name -f flows.sql
```

The dump file looks something like this

```
--
-- PostgreSQL database dump
--

-- Dumped from database version 12.11
-- Dumped by pg_dump version 12.9 (Ubuntu 12.9-0ubuntu0.20.04.1)

SET statement_timeout = 0;
SET lock_timeout = 0;
SET idle_in_transaction_session_timeout = 0;
SET client_encoding = 'UTF8';
SET standard_conforming_strings = on;
SELECT pg_catalog.set_config('search_path', '', false);
SET check_function_bodies = false;
SET xmloption = content;
SET client_min_messages = warning;
SET row_security = off;

--
-- Data for Name: flows; Type: TABLE DATA; Schema: public; Owner: postgres
--

INSERT INTO public.flows (column1, column2, column3) VALUES (value11, value12, value 13);
INSERT INTO public.flows (column1, column2, column3) VALUES (value21, value22, value 23);
```

### Level 2

This is very slow if you have a large number of datapoints (even 1000s). You can instead use:

```
pg_dump --data-only -h localhost -p 5432 -U postgres -d db_name -t flows >  flows.sql
```

You can also dump data from multiple tables like this:
```
pg_dump --data-only -h localhost -p 5432 -U postgres -d db_name -t flows -t clients >  flows_clients.sql
```

You will get all the data in a single copy command. This is much much faster. The exported data will look like this:

```
--
-- PostgreSQL database dump
--

-- Dumped from database version 12.11
-- Dumped by pg_dump version 14.6 (Ubuntu 14.6-0ubuntu0.22.04.1)

SET statement_timeout = 0;
SET lock_timeout = 0;
SET idle_in_transaction_session_timeout = 0;
SET client_encoding = 'UTF8';
SET standard_conforming_strings = on;
SELECT pg_catalog.set_config('search_path', '', false);
SET check_function_bodies = false;
SET xmloption = content;
SET client_min_messages = warning;
SET row_security = off;

--
-- Data for Name: clients; Type: TABLE DATA; Schema: public; Owner: postgres
--

COPY public.clients (column1, column2, column3) FROM stdin;
value11   value12   value13
value21   value22   value23
\.


--
-- Data for Name: flows; Type: TABLE DATA; Schema: public; Owner: postgres
--

COPY public.flows (column1, column2, column3) FROM stdin;
value11   value12   value13
value21   value22   value23
\.


--
-- Name: clientz_id_seq; Type: SEQUENCE SET; Schema: public; Owner: postgres
--

SELECT pg_catalog.setval('public.clientz_id_seq', 257, true);


--
-- Name: flowz_id_seq; Type: SEQUENCE SET; Schema: public; Owner: postgres
--

SELECT pg_catalog.setval('public.flowz_id_seq', 1478, true);


--
-- PostgreSQL database dump complete
--

```

### Level 3

It is usually a good idea to try this on a dummy table first. On the DB into which you are trying to dump the data, you can create a dummy `flows` table from the actual `flows` table like this.

```
CREATE TABLE dupe_flows (LIKE flows INCLUDING ALL);
```

and then 

```
pg_dump --data-only -h localhost -p 5432 -U postgres -d db_name -t flows | sed 's/public.flows/public.dupe_flows/g' > flows.sql
```

It is changing the name of the table from "public.flows" to "public.dupe_flows" in the SQL file. Please be vary that it can have unintended affects if your table name is a substring of some other string. Use it with caution.


### Level 4
If you already have a IaC (Infrastructure as Code) setup, it is likely that you don't need the schema to be created again. You just need to have the data synced.

### Level 5
Sometimes you would want to apply filters on the data from a table. `pg_dump` doesn't seem to have this feature. You can instead use:

```
psql -h localhost -p 5432 -U postgres -d db_name -c "\copy (select * from flows where flow_id in ('1', '2', '3') TO flow_dump.csv delimiter ',' csv header;"
```

You can then load the data like this:
```
psql -h localhost -U postgres -p 5433 -d db_name -c "\copy flows from 'flow_dump.csv' delimeter ',' csv header;"
```

### Level 6
The above assumes that the order of the columns between the two tables is the same. In case, it is not, you can specify the order of the columns when loading the data into pre-production DB like this:

```
psql -h localhost -U postgres -p 5433 -d db_name -c "\copy flows (column1, column3, column2) from 'flow_dump.csv' delimiter ',' csv header;"
```


Note: You will most likely run into issues of timing out if the amount of data you are trying to dump is huge. However, it is not a good idea to dump all the data from production. It would almost never be necessary.

### Putting it together

We have the data dumping and loading sorted. How do we stitch them together and how do we run them on a daily basis?

For the teams that use k8s, you can setup the dumping script as a cron job in the production k8s and configure it to dump the data into a storage location like AWS S3 (or any other storage location to which both k8s clusters have access) and you can setup the loading script to load the data from AWS S3.