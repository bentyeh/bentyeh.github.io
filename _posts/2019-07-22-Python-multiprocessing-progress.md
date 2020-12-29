---
title: Checking progress of Python multiprocessing pools
layout: post
use_code: true
use_toc: true
excerpt: Use `tqdm` or roll your own code snippets to quickly check the progress of your Python multiprocessing pools!
tags: [python, multiprocessing]
---

Process pools, such as those afforded by Python's [`multiprocessing.Pool`](https://docs.python.org/3/library/multiprocessing.html#multiprocessing.pool.Pool) class, are often used to parallelize loops or map a function over an iterable. It can be helpful sometimes to monitor the progress over the loop or iterable, and we demonstrate below several ways to do so.

In the sample code below, we consider a situation where you want to apply a function `my_func` to each value in a dictionary `data`.

## Option 1: Manually check status of `AsyncResult` objects

This option assumes you are working with one of the `_async` pool methods (`apply_async`, `map_async`, or `starmap_async`). These are non-blocking and return [`AsyncResult`](https://docs.python.org/3/library/multiprocessing.html#multiprocessing.pool.AsyncResult) objects, which allow you to check on the status of results.

Specifically, we take advantage of `AsyncResult.successful()`, which does one of the following:
- Returns `True`: the call completed successfully
- Returns `False`: the call completed but raised an exception
- Raises `ValueError`: the call is still running
  - The current Python 3.7 documentation incorrectly states that an `AssertionError` is raised. The documentation will be presuably fixed for Python 3.8. See [https://github.com/python/cpython/pull/13721](https://github.com/python/cpython/pull/13721).

By keeping track of calls that completed but raised an exception, this option makes it easier to debug what went wrong. For example, if we see that an error occurred when calling `my_func` on the value corresponding to `'key3'`, we can run `results['key3'].get()` to recover and analyze the exception.

Option summary
- Non-blocking
- Easy to debug exceptions
- Supports any `_async` method
- Does not support using the process pool as a context manager

### Sample code

```python
import multiprocessing
import time

# Start the pool.
pool = multiprocessing.Pool(nproc)
results = {}
start_time = time.time()
for key, value in data.items():
    results[key] = pool.apply_async(my_func, (value,))

# This code block can be (re)run whenever you want to check on the progress of the pool.
running, successful, error = [], [], []
current_time = time.time()
for key, result in results.items():
    try:
        if result.successful():
            successful.append(key)
        else:
            error.append(key)
    except ValueError:
        running.append(key)
rate = (len(successful) + len(error)) / (current_time - start_time)
print('Running:', sorted(running))
print('Successful:', sorted(successful))
print('Error:', sorted(error))
print('Rate:', round(rate, 3))
print('Estimated time to completion:', time.strftime('%H:%M:%S', time.gmtime(len(running) / rate)))

# Don't forget to join the pool after it finishes running!
pool.close()
pool.join()
```

Example output
```
Running: ['key4', 'key5']
Successful: ['key1', 'key2']
Error: ['key3']
Rate: 1.234
Estimated time to completion: 00:01:23
```

## Option 2: Using `tqdm`

We take advantage of the very helpful progress bar provided by the [`tqdm` package](https://github.com/tqdm/tqdm). How to use `tqdm`, however, depends on the pool method we choose.
- Each `_async` method includes `callback` and `error_callback` arguments that take functions to be called when the result completes, with or without error, respectively. We can use these callback functions to update our `tqdm` progress bar.
- `imap_unordered` and `imap` each return an iterator, so we can simply wrap it with `tqdm`.

Option summary
- Blocking
- Succinct progress bar
- Supports any non-blocking pool method (i.e., any method except `apply`, `map`, and `starmap`)
- Supports using the process pool as a context manager

### Sample code

```python
import multiprocessing
from tqdm import tqdm

# imap_unordered example
results = []
with multiprocessing.Pool(nproc) as pool:
    for result in tqdm(pool.imap_unordered(my_func, data.values()), total=len(data)):
        results.append(result)
    pool.close()
    pool.join()

# imap example
with mulitprocessing.Pool(nproc) as pool:
    results = list(tqdm(pool.imap(my_func, data.values()), total=len(data)))
    pool.close()
    pool.join()

# apply_async example
pbar = tqdm(total=len(data))
def update(*a):
    pbar.update()

results = []
with multiprocessing.Pool(nproc) as pool:
    for key, value in data.items():
        pool.apply_async(my_func, (value,), callback=update)
    pool.close()
    pool.join()
```

Reference: [https://github.com/tqdm/tqdm/issues/484](https://github.com/tqdm/tqdm/issues/484).