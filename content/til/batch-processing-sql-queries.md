+++
title = "Batch processing long running SQL queries"
date = 2022-11-17
updated = 2022-11-17
type = "post"
description = "Reliable code snippets to use whenever there is a need to download huge datasets in batches"
in_search_index = true
[taxonomies]
TIL-Tags = ["SQL", "pandas"]
+++

There is often a need to run a SQL query to save raw data on the disk which can further be used for some kind of analysis. The most common problems we face are [timeout issues](/til/mvcc-timeout-postgres/) (either statement timeout or mvcc timeout when reading from a read replica).

**The following shows how you can process a query in batches:**

Imagine there is a table called the `game` with columns like: `id`, `name`, `place`, `animal`, and `thing`. Let's say, the data present in the `name`, `place`, `animal`, and `thing` columns is too big which is causing your query unable to be executed at once. Your `id` column is a primary key and is also indexed.

In such a case, this is how you can download all the data in batches:

```python

# First get all the ids of the row you want to fetch

import psycopg2 as pg

def connect(
    host,
    port,
    user,
    password,
    db_name
):
    return pg.connect(
        host=host, port=port, user=user, password=password, dbname=db_name
    )

def get_ids(params) -> Tuple[int]:
    #  params = {"name": ("abc", "dfe"), "place": ("uvw", "xyz")}
    query = """
        select id from game
        where 
            name in %(name)s
            and palce in %(place)s
    """
    ids_ = set()
    with connect() as conn:
        with conn.cursor() as cursor:
            cursor.execute(query, {**params})
            return tuple(id_[0] for id_ in cursor.fetchall())
```
Since we are just fetching the `id` column (which is also indexed), this query will (hopefully) not run into timeout issues. If it is still running into timeout issues, you can consider adding extra dimensions to help you filter out data faster.

Now that we have fetched all the ids of the rows we are interested in, we can fetch the row data in batches of these ids

```python

def download_data():
    query = """select * from game where id in %(id)s"""

    ids = get_ids(params) # function we defined above
    num_ids = len(ids)
    batch_size = 1000

    num_batches = (
        num_ids // batch_size
        if num_ids % batch_size == 0
        else num_ids // batch_size + 1
    )
    pandas_header = True
    with tqdm(total=num_batches, desc="Downloading the dataset.") as pbar:
        while i < num_ids:
            batch = ids[i : i + batch_size]
            try:
                with connect() as conn:
                    with conn.cursor(cursor_factory=NamedTupleCursor) as cursor:
                        cursor.execute(query, {id: batch})
                        result_set = cursor.fetchall()

                        if use_pandas:
                            df_chunk = pd.DataFrame.from_records(result_set, columns=["id", "name", "place", "animal", "thing"])
                            df_chunk.to_csv(file_name, header=pandas_header, index=False, mode="a")
                            pandas_header = False
                        else:
                            yield from process_data(result_set) # if you want to process the data further before saving it to the disk
                        pbar.update(1)
                i += batch_size
            except (SerializationFailure, OperationalError) as e:
                # retry this batch incase there is a timeout
                # if this error is occuring too many times, consider reducing the batch size
                logger.error(e)
                logger.error(f"This error is common if you are requesting a large dataset. We will retry the batch in a while.")
                time.sleep(0.2)

data = download_data()
```

### Pandas Alternative

Pandas also supports reading the result from a SQL query in chunks directly as pandas data frames. If you know the optimal batch size, this is how you can proceed:

```python

from sqlalchemy import create_engine

 def get_engine(
        host,
        port,
        user,
        password,
        db_name
    ):
        return create_engine(f"postgresql://{user}:{password}@{host}:{port}/{db_name}") # return sql alchemy engine instead of the psycopgy engine

def download_data(
        db_name, query_file_name, params, output_file_name, processing_fn=None
    ):
        pandas_header = True
        with get_engine(db_name=db_name).connect().execution_options(
            stream_results=True
        ) as conn:
            # If we need to modify something manually in the same connection session, we can do it here
            # example: conn.execute("SELECT statement_timeout = 0;")
            for df_chunk in pd.read_sql_query(
                get_query(query_file_name), conn, params=params, chunksize=1000
            ):
                if processing_fn:
                    df_chunk = processing_fn(df_chunk) # apply any processing on the df_chunk 
                df_chunk = df_chunk.applymap(serialize) # to properly format the json values
                df_chunk.to_csv(output_file_name, header=pandas_header, index=False, mode="a")
                pandas_header = False

download_data()
```

## Resource
1. [Pandas SQL Chunking](https://pythonspeed.com/articles/pandas-sql-chunking/)
2. [Django bulk_update to update all records](https://stackoverflow.com/questions/61040570/django-using-bulk-update-to-update-all-records)

**Note:** These are rough snippets. almost like pseudo-code. This is just to communicate the idea across. Please ensure that you structure this properly and provide the right params when you try to use this code snippet :)