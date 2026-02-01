+++
title = "Avoiding Async Waterfall with asyncio.gather"
date = 2026-01-07
updated = 2026-01-07
type = "post"
in_search_index = true
[taxonomies]
TIL-Tags = ["python", "async"]
+++


This is a case of wanting to run multiple sync functions in an async way. But the problem, is some_sync_func , some_other_sync_func , etc execute one after the other in an async loop. If they have no dependency, you can create async tasks out of them and use async.gather  to wait until all of them are done.

```python

import asyncio
import time

# ignore time.sleep. This is only a problem when there are multiple requests. Then it will block the event loop.

def some_sync_func(a: int) -> int:
    time.sleep(1)
    return a + 1
def some_other_sync_func(b: int) -> int:
    time.sleep(1)
    return b + 1
def another_sync_func(a: int, b: int) -> int:
    time.sleep(1)
    return a + b
def yet_another_sync_func(z: int) -> int:
    time.sleep(1)
    return z + 1

async def func_1(a: int, b: int) -> int:
    loop = asyncio.get_event_loop()
    x = await loop.run_in_executor(None, some_sync_func, a)
    y = await loop.run_in_executor(None, some_other_sync_func, b)
    z = await loop.run_in_executor(None, lambda: another_sync_func(a,b))
    w = await loop.run_in_executor(None, yet_another_sync_func, z)
    return w

async def func_2(a: int, b: int) -> int:
    loop = asyncio.get_event_loop()
    task_x = loop.run_in_executor(None, some_sync_func, a)
    task_y = loop.run_in_executor(None, some_other_sync_func, b)
    task_z = loop.run_in_executor(None, lambda: another_sync_func(a,b))
    task_w = loop.run_in_executor(None, yet_another_sync_func, 1)
    x, y, z, w = await asyncio.gather(task_x, task_y, task_z, task_w)
    return w


if __name__ == "__main__":
    start_time = time.time()
    asyncio.run(func_1(1, 2))
    end_time = time.time()
    print(f"func_1 took {end_time - start_time} seconds") # 4 seconds
    start_time = time.time()
    asyncio.run(func_2(1, 2))
    end_time = time.time()
    print(f"func_2 took {end_time - start_time} seconds") # 1 second. Avoids the waterfall when there is no dependency between the tasks.
```