+++
title = "typing.cast in python"
date = 2025-07-01
updated = 2025-07-01
type = "post"
in_search_index = true
[taxonomies]
TIL-Tags = ["python"]
+++

```python

from typing import List, Union, cast

a : List[Union[int, None]] = [1, 2, 3, None]

b: List[int]

b = [i for i in a] # lint error

b = [i for i in a if i is not None] # no lint error
```

Sometimes the static type checker is not able to infer the type of the variable. In such cases, you can use `typing.cast` to cast the type of the variable. Static checkers don’t narrow types for arbitrary indexed expressions.

```python
data_dfs: Dict[str, Dict[str, Union[None, pd.Series]]] = {}

data_dfs[entity_id]["capital"] = None
data_dfs[entity_id]["ratio"] = None

# lint error is there. Even when we are filtering out the None values even though at runtime, this will not cause any issues.
sums: List[pd.Series] = [
        data_dfs[entity_id]["sum"]
        for entity_id in data_dfs
        if data_dfs[entity_id]["sums"] is not None
    ]

# You can explicitly cast the type to the expected type to make sure 
sums: List[pd.Series] = cast(List[pd.Series], [
        data_dfs[entity_id]["sum"]
        for entity_id in data_dfs
        if data_dfs[entity_id]["sums"] is not None
    ])
```

but the onus is on us to make sure that this is not being used excessivley for convinience. It is an act of fooling ourselves if we do that.

from the first example:
```python

b = cast(List[int], b) # no lint error. but values og b can be None during runtime.

```