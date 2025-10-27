+++
title = "Safe operations on pandas Series"
date = 2024-08-23
updated = 2024-08-23
type = "post"
description = "Small gotchas when doing ops on pandas series"
in_search_index = true
[taxonomies]
TIL-Tags = ["database"]
+++

When dealing with a lot of financial data, it is important to pay close attention to how nulls and 0s are being treated while doing ips like sum and divide

One easy way is to just do a fillna(0) before doing the ops but you might want to retain the null values as null itself

```python
import pandas as pd

a = pd.Series([1, 2, 3, None, 5])
b = pd.Series([1, 2, 3, 4, 5])

result = a + b

# result at index 3 will be NaN but I want it to be treated as 0 during the summation
```

Use this when you want to ignore NaN values but retain NaN if all values in a row are NaN: The min_count=1 parameter ensures that if all values in a row are NaN, the result will be NaN
```python
df.sum(axis=1, min_count=1) # sum multiple columns at once

a.add(b, fill_value=0) # add two series and treat NaN as 0
```

When doing a division, we should handle none/none, value/none, none/value, anything by 0 cases

```python
a = pd.Series([1, 2, 3, None, 5])
b = pd.Series([1, 2, 3, 4, 5])

def divide_series_by_series(series1: pd.Series, series2: pd.Series) -> pd.Series:
    """Divides a series by another series. handles null values and division by 0

    Args:
        series1 (pd.Series): The numerator series
        series2 (pd.Series): The denominator series

    Returns:
        pd.Series: The result of the division
    """
    return series1.divide(series2.replace(0, np.nan)).fillna(np.nan)

result = divide_series_by_series(a, b)
```

It is important to do these safe poerations without actually modifying the base values because keep a tracking of values with large data transformation flow is hard.




