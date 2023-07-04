+++
title = "Testing in Python"
date = 2023-07-04
updated = 2023-07-04
type = "post"
description = "Notes and lessons learned from writing tests in Python."
in_search_index = true
[taxonomies]
TIL-Tags = ["software-testing"]
+++

I've written this almost a year back. I decided to give an (almost) permanent home here. Since I wrote this mostly for myself, the structure of the blog is not great. However, I believe the content is quite useful for me.

# Python Testing in Visual Studio Code

The [Python extension](https://marketplace.visualstudio.com/items?itemName=ms-python.python) for VSCode (developed by Microsoft) makes it easier to navigate the testing process (especially when there a many). If you have the extension installed, you should be able to see a beaker icon on the side panel.

 ![The beaker icon](/images/til/python-testing/beaker-icon.png)

Some advantages of using this capability over using `pytest` CLI are:


1. **Running a specific test is easier**

   
   1. A tree view of all the test folders, test files, and test functions is displayed in the left panel. You can hover over each item to run (or) run&debug all the test cases that are lower in the hierarchy of the hovered item.

       ![tree view of all the discovered tests](/images/til/python-testing/tree-view-tests.png)
   2. We can hover over `test_fn` and run that test alone with a button click instead of doing it with the [pytest CLI like this](https://stackoverflow.com/a/36539692/10524266). Since I struggle to remember the commands, button clicks are faster.
   3. You can also run a test from the opened file in VSCode itself. You will see a run icon on the left of each discovered test function

       ![A test function with a run icon on the left](/images/til/python-testing/run-icon-test.png)


{% alert(type="info") %}
There are buttons on the UI for the most frequent CLI commands. 
For example: Re-running failed test cases only
{% end %}


2. **You can view the entire output of the testing log.**

   
   1. You can switch to the “OUTPUT“ section of the integrated terminal and select “Python Test Log“ to view the entire output.
       ![Log of pytest](/images/til/python-testing/pytest-log.png)


{% alert(type="info") %}
TIL: The use of the “OUTPUT“ section on the integrated terminal of VSCode
We can view the output logs of all the extensions
{% end %}

 ![Select the log output you want to view](/images/til/python-testing/pytest-log-select-vscode.png)



3. **View and compare the Test History of a test case easily**
   
   1. If a particular test case has failed now, we can see the test result history in the gutter decorations that will be displayed for each failed test case

       ![a linear history of test results being displayed](/images/til/python-testing/test-failure-compare.png)

**Note:** You can learn more about Python testing in VSCode [here](https://code.visualstudio.com/docs/python/testing).


---

# Monkey Patching

We shall see how we can leverage monkey patching to test functions that are expected to spit out different values on each run.
### Monkey patching fixture

Let’s say we have the following function

```python
import uuid

def greet(greeting: str) -> str:
    return greeting + str(uuid.uuid4())
```

Let’s try to create a test for this function

```python
import uuid

def test_greet(monkeypatch: MonkeyPatch):
    sample_uuid = str(uuid.uuid4())
    greeting = "Hello "
    assert greeting + sample_uuid == greet(greeting)
```

Such a test will not pass. You will get an error something like this:

```javascript
./tests/test_basic.py::test_greet Failed: [undefined]AssertionError: assert 'Hello 2639f2...-87ef386cae27' == 'Hello a1436c...-d3b69af84d05'
  - Hello a1436ca0-cf34-47c1-8aca-d3b69af84d05
  + Hello 2639f240-b792-45b6-be49-87ef386cae27
monkeypatch = <_pytest.monkeypatch.MonkeyPatch object at 0x7faf98410730>

    def test_greet(monkeypatch: MonkeyPatch):
        sample_uuid = str(uuid.uuid4())
        # monkeypatch.setattr(uuid, "uuid4", lambda: sample_uuid)
        greeting = "Hello "
>       assert greeting + sample_uuid == greet(greeting)
E       AssertionError: assert 'Hello 2639f2...-87ef386cae27' == 'Hello a1436c...-d3b69af84d05'
E         - Hello a1436ca0-cf34-47c1-8aca-d3b69af84d05
E         + Hello 2639f240-b792-45b6-be49-87ef386cae27

tests/test_basic.py:10: AssertionError
```

This is because the UUID generated in the `test_greet` function is `2639f240-b792-45b6-be49-87ef386cae27` whereas the UUID generated in the `greet` function is `a1436ca0-cf34-47c1-8aca-d3b69af84d05`

This is fair because that is how `uuid4()` works. Every time it is supposed to return us a unique string.

**So how can we test something that keeps changing every time it gets called?**
This is exactly why [pytest fixtures](https://docs.pytest.org/en/6.2.x/fixture.html) are useful. We will now use the Monkey Patching fixture.


Let’s modify the test function like this:

```python
from pytest import MonkeyPatch
import uuid

def test_greet(monkeypatch: MonkeyPatch):
    sample_uuid = str(uuid.uuid4())
    monkeypatch.setattr(uuid, "uuid4", lambda: sample_uuid)
    greeting = "Hello"
    assert greeting + sample_uuid == greet(greeting)
```

**Here is what we did**


1. We made the `test_greet` function accept a pytest fixture called monkey-patch.

   1. Note: The argument has to be `monkeypatch` only. Any other variant or another word will not work.
2. We modified the `uuid4` function with our own implementation.

   1. The original implementation of `uuid4()` is

    ```python
    def uuid4():
        """Generate a random UUID."""
        return UUID(bytes=os.urandom(16), version=4)
    ```

   2. After the line `monkeypatch.setattr(uuid, "uuid4", lambda: sample_uuid)` it will behave like this

    ```javascript
    def uuid4():
        return "2639f240-b792-45b6-be49-87ef386cae27" # the contents of sample_uuid
   ```
3. Let’s analyse point 2 a bit further

   
   1. `monkeypatch.setattr` is helping us replace the implementation of `uuid4()` function from the `uuid` module.
   2. Now every time we call `uuid4()` , it will always give us `sample_uuid` (which we hardcoded to "2639f240-b792-45b6-be49-87ef386cae27")
   3. After the monkey patch is applied, when we call the `greet` function (in the assert statement of the `test_greet` function), we get `2639f240-b792-45b6-be49-87ef386cae27` from the modified `uuid4()` function
4. It is important to note that the effect of monkey patching wears off as soon as we exit the test function. That is to say if we call `uuid.uuid4()` in another test function, we should be expecting the original behavior of `uuid.uuid4()`to occur.

   ```python
   def test_greet(monkeypatch: MonkeyPatch):
   # no error because uuid4 is modified to return a mock value
       sample_uuid = str(uuid.uuid4())
       monkeypatch.setattr(uuid, "uuid4", lambda: sample_uuid)
       greeting = "Hello "
       assert greeting + sample_uuid == greet(greeting)
   
   def test_greet2():
   # error will occur becuse uuid4 is not modified in this test. 
   # The affect of monkeypatch only applies to the test function 
   # where the fixture is applied
       sample_uuid = str(uuid.uuid4())
       greeting = "Hi "
       assert greeting + sample_uuid == greet(greeting)
   ```

### Special Mention

1. This approach can be used to test anything that keeps changing every time we run it. Examples: Functions that involve random integers and timestamps.
2. Monkey patching doesn’t work if a module has immutable objects or attributes. Example: datetime.datetime. We will have to [patch](https://stackoverflow.com/a/20503374/10524266) the entire datetime.datetime class.
3. You can also use [freezegun to freeze datetime](https://stackoverflow.com/a/28080767/10524266) values for testing purposes.

### Monkey Patching Outside Testing

Monkey patching is not unique to testing. Not even unique to Python. [This](https://stackoverflow.com/a/6647776/10524266) has the best explanation of what it is.

**One way we can use this in our day-to-day work is:**

1. Let’s say, we are facing an issue with a function’s implementation coming from an external library.
2. Ideally, the owners of the library have to fix the bug in, write a new test case, wait for the build to pass, and release a new version. After that, we can upgrade the library's version in our codebase.
3. However, sometimes, there is no time to die. In such exceptional cases, we can monkey-patch the problematic function with our implementation and then ship our code.

### Things to be careful about Monkey Patching

There are serious drawbacks to monkey-patching:


1. If two modules attempt to monkey-patch the same method, one of them (whichever one runs last) "wins" and the other patch has no effect. (In some cases, if the "winning" monkey-patch takes care to call the original method, the other patch(es) may also work; but you must hope that the patches do not have contradictory intentions.)
2. It creates a discrepancy between the source code on the disk and the observed behavior. This can be very confusing when troubleshooting, especially for anyone other than the monkey-patch's author. Monkey-patching is therefore a kind of antisocial behavior.
3. Monkey-patches can be a source of upgrade pain when the patch makes assumptions about the patched object which are no longer true.

[This](https://web.archive.org/web/20120730014107/http://wiki.zope.org/zope2/MonkeyPatch) is the source for the above points.

# Test Performance and Coverage

### Coverage

```bash
pytest --cov=slu --cov-report html --cov-report term:skip-covered tests/
```

The above statement gives us a coverage report. For each file, we can see the test report below:           ![Sample test coverage report](/images/til/python-testing/sample-test-coverage.png)

The green lines signify that the tests have touched these lines. It is important to note that it does not mean every possible scenario has been tested for these lines. The red lines are the ones that none of the test functions reached.

### Pinning Down Slowest Tests

You can pass the number with --durations

```powershell
pytest --durations=0 — Show all times for tests and setup and teardown

pytest --durations=1 — Just show me the slowest test

pytest --durations=50 — Slowest 50, with times, … etc
```
Usage:
```python
# example command
pytest --durations=3
# example output

=============================================================================== slowest 3 durations ===============================================================================
0.39s call     tests/test_controller/test_predict_api.py::test_utterances[116918a3bc5138ccced42e825522d0397d5c29327290c0f912d4a5ade97cdf14-payload0]
0.37s call     tests/test_controller/test_predict_api.py::test_utterances[593e45693169a78ef9ac329fd2adee8f6912b370196fefb949fd41348674cdcc-payload3]
0.32s call     tests/test_controller/test_predict_api.py::test_utterances[1b464d8c8d00d0d78915ccb3362b2339c6545fac33c3d7b018096690302b671b-payload2]
```

source: [here](https://stackoverflow.com/a/55095253/10524266)

### Test Performance Visualisation

There is a 3rd party library called [Pytest-benchmark](https://pypi.org/project/pytest-benchmark/) which provides a fixture for accurately benchmarking a test case.

You can use it like this:

```python
from pytest_benchmark.fixture import BenchmarkFixture

def test_greet(monkeypatch: MonkeyPatch, benchmark: BenchmarkFixture):
    sample_uuid = str(uuid.uuid4())
    monkeypatch.setattr(uuid, "uuid4", lambda: sample_uuid)
    greeting = "Hello "
    assert greeting + sample_uuid == benchmark(greet, greeting)
```

When we run pytest:

```javascript
============================= test session starts ==============================
platform linux -- Python 3.9.11, pytest-7.1.2, pluggy-1.0.0
benchmark: 3.4.1 (defaults: timer=time.perf_counter disable_gc=False min_rounds=5 min_time=0.000005 max_time=1.0 calibration_precision=10 warmup=False warmup_iterations=100000)
rootdir: /home/roronoa/Desktop/workspace/personal/python_testing
plugins: benchmark-3.4.1
collected 1 item

tests/test_basic.py .                                                    [100%]

-------------- generated xml file: /tmp/tmp-5172mcnCmBQDSfPj.xml ---------------

------------------------------------------------------ benchmark: 1 tests -----------------------------------------------------
Name (time in ns)          Min          Max      Mean    StdDev    Median      IQR   Outliers  OPS (Mops/s)  Rounds  Iterations
-------------------------------------------------------------------------------------------------------------------------------
test_greet            340.0019  16,096.9976  395.4294  117.2442  390.0022  37.9987  4900;5713        2.5289  168663           1
-------------------------------------------------------------------------------------------------------------------------------

Legend:
  Outliers: 1 Standard Deviation from Mean; 1.5 IQR (InterQuartile Range) from 1st Quartile and 3rd Quartile.
  OPS: Operations Per Second, computed as 1 / Mean
============================== 1 passed in 1.55s ===============================
```

To visualize it:

```javascript
pytest --benchmark-histogram
```

This will provide an SVG file that has the histogram of the performance. Pytest-benchmark runs the code several times to give us a median value.     ![pytest-benchmark histogram visualization of two test cases side by side](/images/til/python-testing/pytest-benchmark-histogram.png)


**Note:** Pytest-benchmark gives us an accurate picture as opposed to finding the difference between the times at the start and end of a function. It accounts for asynchronous code too.


# The Importance of Negative Test Cases

Some time ago, I was watching this video([The Most Common Cognitive Bias](https://www.youtube.com/watch?v=vKA4w2O61Xo)) by Veritasium and I realized that the learnings from the video can directly be applied to writing tests for code.

**Here is what I gathered:**

1. There was a theory that all swans are white. So every white swan that you come across makes you think, “Yeah the theory is pretty good”.
2. People in the video are asking a question for which they expect the answer to be yes.
3. But you want to get to the NOs because that is much more information for you than the yes. A “yes” confirms what you are thinking, and a “no” breaks what you are thinking.
4. The scientific way to prove that something is true is to constantly try and disprove it. Only when we are not able to disprove it, we must be getting closer to something true.
5. If you think something is true, you need to try as hard as you can to disprove it. That is the only way to not fool yourself. Every "no" is part of the answer leading to "yes".
6. Cognitive Bias is such that humans generally want to confirm the truth with similar data (pattern matching), not get closer to the truth by proving false data (edge case detection). Both have value, but the latter is critical. (a comment on the same video)

### Applying it to Writing Tests

Trying to apply what we learned above: Suppose there is a function that takes in a list of numbers and returns True/False based on if the numbers follow a certain pattern or not. Considering the code implementation as a black box (which can happen if you are new to the project or if the code is hard to read), how would you go ahead and write the test cases?


1. You can write a test case with 2, 4, and 8 and you get `True` as the result. This is a positive test case.
2. Now you can write N such test cases where you expect `True` as the result. Let’s say you think the pattern is “**multiply by 2**“. Multiple positive test cases will strengthen your belief that you’ve figured out the pattern. But also this can lead to bias.
3. Instead, once you have a hypothesis, the best way to verify it is to disprove it.
4. So as the second test case, you can write 2, 4, and 7 and expect to get `False`. Surprise! you get `True` again. This negative test case helped you avoid cognitive bias.
5. After some iterations, you’ve formed a hypothesis that the pattern is “**numbers in ascending order**“. You write some positive test cases. Now you have a stronger hypothesis. Then you write a negative test case (input: 2, 5, 1) and expect to get `False`. Once your negative test case also passes, that’s when you can be more confident about your hypothesis.
6. Positive and Negative test cases together give us double confirmation of our hypothesis.

# Misc Learnings
1. If your function code is reading any value from environment variables, configuring pytest will have issues. To solve this, you can create a `.env` file with all the environment variables. Post this, you can configure pytest with VSCode python.
2. Every statement in a Makefile runs in separate sessions.