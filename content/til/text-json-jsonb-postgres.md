+++
title = "TEXT vs JSON vs JSONB column data types in postgres"
date = 2023-11-29
updated = 2023-11-29
type = "post"
description = "What data type to choose for storing dictionary information in postgres"
in_search_index = true
[taxonomies]
TIL-Tags = ["postgres", "SQL"]
+++

1. If you have a python dict you want to store:
    1. TEXT is not ideal as you would have to json dump your dict and then to query based on attributes of the column you have to do something like `where data::jsonb ->> 'attribute' = 'value'`. You have to convert to json or jsonb while querying/filtering. It is the fastest to insert though. May not be storage efficient as indendation of json dumps is also stored.
    2. JSON column allows to save the dict as it is (with same key order, indentation spaces, etc). It is faster to insert as compared to JSONB but slower than TEXT (because it needs to check for json validity). However, it is not storage efficient as JSONB. It offers basic querying and filtering capability. No need to json dumps and json loads everytime.
    3. JSONB is the best and should be the go to choice most of the time. It is storage efficient (removes indentation spaces for example). But it doesn’t guarantee order of the keys. It also allows for indexing based on the attributes of JSONB. The only disadvantage compared to JSON is it can be a little slow when inserting. If your input data isn’t too large, this is negligable.

[Reference](https://chat.openai.com/share/dff0f491-50db-4a17-9075-4849fa41b25d)