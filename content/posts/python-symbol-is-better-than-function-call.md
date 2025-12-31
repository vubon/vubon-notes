---
title: 'Python: Symbol is better than function call'
date: 2019-09-15 21:24:56
lastmod: 2021-12-14 00:46:16
draft: false
categories:
  - 'Python'
aliases:
  - '/python-symbol-is-better-than-function-call/'
---

## Benchmarking Function Calls vs. Symbols

[The time difference is from my Core-i3 machine; it may vary on yours.]

![Function vs symbol benchmark](/vubon-notes/images/function_vs_symbol.png)

```python
# Example 1
from timeit import timeit

print(timeit("my_tuple = tuple([1, 2, 3, 4, 5, 6, 7, 8, 9, 0])"))
print(timeit("my_tuple = ([1, 2, 3, 4, 5, 6, 7, 8, 9, 0])"))

# Output
# 0.3789963669996723
# 0.1345829190004224

# Example 2
print(timeit("my_set = set([1, 2, 3, 4, 5, 6, 7, 8, 9, 0])"))
print(timeit("my_set = {*[1, 2, 3, 4, 5, 6, 7, 8, 9, 0]}") )

# Output
# 0.6751592210002855
# 0.547103707000133
```

You will see a probably trivial difference—but a difference is still a difference. Think about huge datasets; the cost adds up.

**What is Python actually doing?**

1. Look up the function in memory.
2. Initialize and manage a frame for the function.
3. Store all local variables.
4. Store activation records (call frames) on the runtime stack.
5. Push the frame to the top of the stack when it’s called.

That’s a lot of work for `set()` and `tuple()`. With literal syntax, Python just loads constants (constant folding) and skips much of that overhead.

In the set example, I used `*` unpacking to turn a list into a set literal; otherwise you can’t directly convert a list to a set literal.

I hope the purpose of this article is clear. If it is, great—see you in the next one.
