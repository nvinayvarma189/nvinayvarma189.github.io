+++
title = "Python profiling"
date = 2023-11-30
updated = 2023-11-30
type = "post"
description = "How to get a line by line profile of a python function"
in_search_index = true
[taxonomies]
TIL-Tags = ["python"]
+++

```python
from line_profiler import LineProfiler
import time

profiler = LineProfiler()

def profile(func):
    def inner(*args, **kwargs):
        profiler.add_function(func)
        profiler.enable_by_count()
        return func(*args, **kwargs)
    return inner

@profile
def a():
    b=1
    for i in range(2):
        b+=i
        time.sleep(1)
    return b

print(a())
profiler.print_stats()
```

[Reference](https://medium.com/uncountable-engineering/pythons-line-profiler-32df2b07b290)