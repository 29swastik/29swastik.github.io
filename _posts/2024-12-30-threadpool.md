---
title: "Understanding Python’s ThreadPoolExecutor Context Manager"
date: 2024-12-30 00:00:00 +0000
categories: [Python, Concurrency]
tags: [threading, concurrency, threadpoolexecutor]
pin: true
math: true
mermaid: true
image:
  path: assets/img/posts/2024-12-30-threadpool/threadpool.png
  alt: ThreadPoolExecutor Context Manager  
---


## Introduction
A thread pool helps run tasks in parallel in Python, but sometimes your code might end up running tasks one after another instead of in parallel. Let’s see why this happens and how to fix it.

Imagine a product search service that needs to:

1.  Search product vectors
2.  Perform full-text search
3.  Fetch sponsored products

## Initial Code

```python
from concurrent.futures import ThreadPoolExecutor
import time
def vector_search(query):
    time.sleep(1)
    return ["product1", "product2"]
def full_text_search(query):
    time.sleep(2)
    return ["product3", "product4"]
def get_sponsored_products():
    time.sleep(3)
    return ["sponsored1"]
def search_products(query):
    with ThreadPoolExecutor(max_workers=2) as executor:
        vector_future = executor.submit(vector_search, query)
        fts_future = executor.submit(full_text_search, query)
    sponsored = get_sponsored_products()
    return {
        "vector_results": vector_future.result(),
        "fts_results": fts_future.result(),
        "sponsored": sponsored
    }
start = time.time()
results = search_products("sneakers")
print(f"Total time: {time.time() - start:.2f} seconds")
```

## Output:

```
Total time: 5.00 seconds --- 😱
```

## What’s Happening? 🤔

You expect 3 seconds, but it takes 5! Moving out of the with block pauses execution until all tasks submitted inside it are complete. This causes the sponsored products call to start only after the first two tasks are done, delaying it by 2 seconds.

## The Fix ✅

```python
def search_products_fixed(query):
    with ThreadPoolExecutor(max_workers=3) as executor:
        vector_future = executor.submit(vector_search, query)
        fts_future = executor.submit(full_text_search, query)
        sponsored = get_sponsored_products()
        return {
            "vector_results": vector_future.result(),
            "fts_results": fts_future.result(),
            "sponsored": sponsored
        }
start = time.time()
results = search_products_fixed("sneakers")
print(results)
print(f"Total time: {time.time() - start:.2f} seconds")
```

## Output:
```
Total time: 3.00 seconds --- 🚀
```

## Visualizing the Execution 🔍


To better understand the difference between the before and after scenarios, here’s a graphical representation of how tasks are executed:

![Execution Time](assets/img/posts/2024-12-30-threadpool/execution-time.png)

## Key Takeaways 💡

1.  Moving out of the with block waits for tasks to complete before proceeding.
2.  Submit all parallel tasks inside the with block to avoid delays.
3.  Plan workflows carefully for true parallelism.

For more details, check the [official documentation](https://docs.python.org/3/library/concurrent.futures.html).

Have you faced similar surprises? Share your thoughts below!