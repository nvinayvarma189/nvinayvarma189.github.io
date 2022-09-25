+++
title = "Application DB v/s Warehouse DB"
date = 2022-09-25
updated = 2022-09-25
type = "post"
description = "Notes on why a Warehouse DB is needed in addition to the application DB"
in_search_index = true
[taxonomies]
TIL-Tags = ["database", "data-engineering"]
+++

A data engineer is often required to write pipelines so that engineers from other teams can consume daily production data for their analysis and modelling needs.

They can either try to read directly from the application DB (the DB that is connected to your main application) or create a new DB (Warehouse DB) to store transformed data which will be ready for analytics.

It is better to maintain a warehouse DB because:
1. Since the uptime and performance of application DB is critical for production usecases, you can keep deleting the old data. The older data can be fetched from the data lake or from the warehouse DB. 
2. Let's say you want to add a new (derived) column which will make things easier for you analytic purposes, it is harder to make changes to the schema of the application DB than to the warehouse DB
3. Application DBs are usually not designed to be performant for analytic purposes. Application DBs are never optimized for long term storage and querying.


### Misc:

**Data Lake:** A storage location where all of the raw and unprocessed data is dumped for a later use. Storage options like S3 are best.

**Data Warehouse:** A storage location used for storing processed data which is ready for immediate consumption (by product/analytics teams).