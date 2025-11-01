+++
title = "Python .format() doesn't raise error on unused arguments"
date = 2025-10-09
updated = 2025-10-09
type = "post"
in_search_index = true
[taxonomies]
TIL-Tags = ["python"]
+++

When you pass extra args to an .format() method, it will not raise an error. It will just ignore the extra args. This allows for good flexibility. Easier to write code but also easier to produce bugs.

This happened to me a few times in the past. Two such examples are:

```python
id = "some_id"
name = None
idea = None

prompt_template = """
* Current Timestamp: {current_timestamp}

* id: {id}
* name: {name}
* idea: {idea}
"""

# update the values based on some logic
id, name, idea = get_details()

prompt = prompt_template.format(id=id, name=name, idea=idea)

print(prompt)
```

This should've worked well. Except that I had mistakenly made prompt_template already formatted. Like this

```python

prompt_template = f"""
* Current Timestamp: {current_timestamp}

* id: {id}
* name: {name}
* idea: {idea}
"""
```

So the line `prompt = prompt_template.format(id=id, name=name, idea=idea)` was effectively doing nothing.

Another case:

```python
id = "some_id"
name = None
idea = None

prompt_template = """
* Current Timestamp: {current_timestamp}

* id: {id}
* name: {name}
"""

# update the values based on some logic
id, name, idea = get_details()

prompt = prompt_template.format(id=id, name=name, idea=idea) # I was thinking that `idea` is being passed here.
# but infact the llm was never seeing the idea.

print(prompt)
```

Who is responsible for such bugs? me. I quiclly made some changes and pushed without testing since it was a small change. But it would be great if the language helps me reduce such bugs.