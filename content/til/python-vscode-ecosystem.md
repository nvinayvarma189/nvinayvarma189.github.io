+++
title = "Python VsCode Ecosystem"
date = 2024-08-21
updated = 2024-08-21
type = "post"
description = "Notes on Python VSCode Ecosystem and defining linters, type checkers, formatters, refactoring tools, etc."
in_search_index = true
[taxonomies]
TIL-Tags = ["python", "ide"]
+++

## The Core: Python Extension & Pylance

Pylance is an (official) VS Code extension from Microsoft that comes along with the official Python VS Code extension (it also downloads the Python debugger).

The Python VS Code extension provides many features:
- **Linter/Checker**: Powered by Pylance. The linter part of Pylance is internally powered by Pyright (Microsoft's static type checker, alternative to Mypy) and Pylint.
- **Code Formatting**: Pylance can leverage extensions like `autopep8`, `black`, or `ruff`.
- **Code Refactoring**: Pylance can leverage extensions like `isort` or `ruff`.
- **Intellisense**: Code navigation and autocomplete (included in Pylance).
- **Debugging**: Python Debugger extension.
- **Testing Support**.
- **Virtual Env Support**: Automatic detection and putting the relevant Python location in the sys path.

## Linting & Type Checking

### What is a Linter?
A linter highlights potential issues at code editing time (before actually running the code). Static type checking is a subset of the responsibilities of a linter.

Most linters (Pyright, Mypy, Ruff) have VS Code extensions for inline errors and also come as a CLI for terminal use.

### Categories of Linting
- **Type Checkers**: Verify type annotations/hints. (Mypy, Pyright, Pyre, Pytype)
- **Error Linters**: Point out syntax errors or code that results in crashes. (Pylint, Pyflakes, Flake8)
- **Style Linters**: Point out PEP 8 or readability issues. (Pylint, Flake8)
- **Packaging Linters**: Issues related to PyPI packaging/metadata. (Pyroma)
- **Security Linters**: Potential security vulnerabilities. (Bandit, Dodgy)
- **Code Formatters**: Change style (whitespace) without affecting behavior. (Black, autopep8)
- **Dead Code Linters**: Remove commented-out code. (Vulture, eradicate)
- **Docstring Linters**: Style issues in docstrings (PEP 257). (pydocstringformatter, docformatter)
- **Complexity Analyzers**: Point out complex, unreadable code. (mccabe, Radon)

### Configuring Type Checking in VS Code
**IMPORTANT:** To enable static code checking at coding time, set `Python > Analysis: Type Checking Mode` to **"Basic"** or **"Strict"**.
*Note:* This setting comes from the Python extension rather than Pylance itself.

#### Strict Mode Considerations
Strict mode can take away some flexibility that Python allows at runtime. 
**Example with Pydantic:**
```python
class OnboardBankRequest(BaseModel):
    bankIdrssd: int
    onboardedBy: str

request = OnboardBankRequest(bankIdrssd="123", onboardedBy="John Doe")
```
Pydantic will convert the string `"123"` into int `123` at runtime. However, **Strict Mode** will complain about this. I believe it's better to adhere to strict mode and not rely on such runtime flexibilities.

Pylance automatically works with Pydantic models as well.

## Modern Tooling: Ruff
Ruff provides both linting and formatting options. It is widely used to replace `isort`, `flake8`, and others.

- **Type Checking Limitations**: Ruff cannot do static type checking (yet). Even though type checking is a branch of linting, you still need the **Pyright extension** (default with Pylance) or **Mypy extension** if you want inline type errors at coding time.
- **Replacement**: Ruff replaces `isort` and many `flake8` plugins.

## Formatting & Refactoring

### Code Formatter
Restructures the code (not refactoring) to fall in line with best practices defined in PEPs.
*Examples:* `autopep8`, `black`, `ruff`.

### Code Restructurer (Refactoring)
Allows automatic modifications like:
- Sorting imports.
- Extracting a block of code into a function.
*Examples:* `isort`, `ruff`.

## Intellisense & Autocomplete

Intellisense provides autocompleting variables, parameters, and attribute names, as well as "Go to Definition".

### Observations: Autocomplete Behavior
Static type checking integrates well with native types, but autocomplete doesn't always behave as expected.

#### Example 1: Pydantic BaseModel (Works)
For autocomplete to work in constructors, I had to subclass `BaseModel` from Pydantic.
```python
class PydanticUser(BaseModel):
    id: int
    name: str
    email: Optional[str] = None

# Autocomplete works here
pydantic_user = PydanticUser(na 
```

#### Example 2: Pydantic without Type Hints (Fails)
Even with Pydantic, if type hints are missing, it fails:
```python
class PydanticUserIn(BaseModel):
    name = "lol"
    secret = 1

# Autocomplete for 'name' attribute doesn't work here
pydantic_user = PydanticUserIn(nam
```
**Conclusion:** Autocomplete seems to only work reliably when using `BaseModel` AND explicit type hints.

This is beacause un-annotated fields are treated as class attributes, not constructor arguments by pydantic.

## References
- [GPXZ: Ruff Guide](https://www.gpxz.io/blog/ruff)
- [YouTube: Python Tooling](https://www.youtube.com/watch?v=udP4cD2JAFM)
