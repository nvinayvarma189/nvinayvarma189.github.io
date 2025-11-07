+++
title = "Python type checking for third party libraries"
date = 2024-09-05
updated = 2024-09-05
type = "post"
in_search_index = true
[taxonomies]
TIL-Tags = ["python"]
+++


For mypy to perform static type checking, it needs type annotations in the code. If it is your code, you can just add type annotations and it will start working. But when you are using a python library, mypy also checks the types of that library (say you are using a function from a library) and raises an issue when the return type is not matching, for example.

Most python libraries now have type annotation written in the package code itself. But some of the packages that were being maintained from a long time would not have type annotations from the beginning.

They have two options: 
1. Add type annotations in the code or create pyi files. Creating pyi files has two options again:
    1. Include them in the same code repository of the main package
    2. Release them as a separate types stub package (Example: types-requests for requests module). Check the actual source repo [here](https://github.com/psf/requests/tree/main/src/requests) and the stubs repo [here](https://github.com/python/typeshed/tree/main/stubs/requests/requests). Notice how each `.py` file in the source repo has a corresponding `.pyi` in the stubs repo.
    3. Mypy, by default searches the https://github.com/python/typeshed repository for many industry standard libraries.

### Questions
1. Why would you want to write pyi files instead of writing them in the library code itself?

    1. To maintain backward compatibility. If your packages needs to support python < 3.5 (or even python 2), you can't really add type annotations inline with the code. For this, you can use separate pyi files. That is why a lot of the OG packages like numpy, pandas would have pyi files. Same with some type annotations only available with 3.9

    2. When you have long type annotations, code becomes very clutterd. Though this can be solved with pydantic.

2. How to check if a given python library has built-in types (inline type hints) or makes types available via type stubs?

    1. Just check some random `.py` file and see if there are any type annotations. If it has stubs, they are usually put inside a folder like `stubs`.

    2. If the package has a `py.typed`  file (it is an empty file), it means that the library code is declared to have inline type annotations.

### References

[ChatGPT Conversation](https://chatgpt.com/c/0b935ebe-597e-40a3-95bb-b94a5da1f38e)