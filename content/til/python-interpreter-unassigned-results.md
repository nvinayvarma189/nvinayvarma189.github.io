+++
title = "Python interpreter stores unassigned results to '_'"
date = 2024-06-05
updated = 2024-06-05
type = "post"
in_search_index = true
[taxonomies]
TIL-Tags = ["python"]
+++

I have the habit of using the python interpreter to do some quick experiments. Sometimes I do rather ambitious things, this one time I was running an expensive operation ~20 mins and I was supposed to deploy that in the next few mins.

I was in a hurry and just executed the code (a function with a loop) and forgot to assign it to some variable so that I can write the results to a file.

That's when this knowledge came in handy. I simply assigned the value of `_` to a variable and wrote it to a file.

```python
>>> def expensive_func(a):
...     return a
...     
>>> expensive_func(1)
1
>>> _
1
>>> print(_)
1
>>> 
```
