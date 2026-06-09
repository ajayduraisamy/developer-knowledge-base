# Itertools - count, cycle, chain, groupby, permutations (std lib)

## Introduction
`itertools` is a standard library module that provides fast, memory-efficient tools for working with iterators. It implements iterator building blocks inspired by constructs from APL, Haskell, and SML.

## Why It Is Important
- Avoids nested loops with cleaner functional composition
- Lazy evaluation minimizes memory usage
- Combinatorial generators eliminate boilerplate
- Essential for data pipelines and functional programming

## Syntax
```python
import itertools
```

## Examples

### Beginner Examples

**Counting and cycling:**
```python
from itertools import count, cycle, repeat

for i in count(5):       # 5, 6, 7, ...
    if i > 10:
        break
    print(i)

# Cycle through a sequence infinitely
for idx, color in enumerate(cycle(['red', 'green', 'blue'])):
    if idx > 5:
        break
    print(color)

# Repeat a value
for _ in range(3):
    print(next(repeat('hello')))  # hello
```

**Accumulate:**
```python
from itertools import accumulate
import operator

print(list(accumulate([1, 2, 3, 4])))  # [1, 3, 6, 10]
print(list(accumulate([1, 2, 3, 4], operator.mul)))  # [1, 2, 6, 24]
```

### Intermediate Examples

**Chain and compress:**
```python
from itertools import chain, compress, zip_longest

# Chain multiple iterables
combined = chain([1, 2], [3, 4], [5])
print(list(combined))  # [1, 2, 3, 4, 5]

# Compress filters one iterable by a selectors iterable
data = ['A', 'B', 'C', 'D']
selectors = [1, 0, 1, 0]
print(list(compress(data, selectors)))  # ['A', 'C']

# Zip longest fills missing values
a = [1, 2, 3]
b = ['x', 'y']
print(list(zip_longest(a, b, fillvalue='Z')))
```

### Advanced Examples

**GroupBy — grouping sorted data:**
```python
from itertools import groupby

data = [
    {'name': 'Alice', 'dept': 'Engineering'},
    {'name': 'Bob', 'dept': 'Engineering'},
    {'name': 'Charlie', 'dept': 'Marketing'},
    {'name': 'Diana', 'dept': 'Marketing'},
    {'name': 'Eve', 'dept': 'Engineering'},
]

# Must sort by the grouping key first
data.sort(key=lambda x: x['dept'])

for dept, group in groupby(data, key=lambda x: x['dept']):
    names = [p['name'] for p in group]
    print(f"{dept}: {names}")
```

**Combinatorial generators:**
```python
from itertools import permutations, combinations, combinations_with_replacement, product

items = ['A', 'B', 'C']

# Permutations: order matters
print(list(permutations(items, 2)))

# Combinations: order does not matter
print(list(combinations(items, 2)))

# Combinations with replacement
print(list(combinations_with_replacement(items, 2)))

# Cartesian product
print(list(product(items, repeat=2)))
```

## Real-World Use Cases
- Paginating through API results with `islice`
- Generating test data with product/permutations
- Processing large CSV files in chunks with `islice`
- Sliding window operations with `tee` and `islice`

## Common Mistakes
- Forgetting that itertools returns iterators, not lists
- Not sorting data before using `groupby`
- Consuming an iterator twice without recreating it

## Best Practices
- Convert to list only when you need to store results
- Chain itertools functions for complex pipelines
- Use `takewhile`/`dropwhile` for conditional slicing over `filter` when order matters

## Interview Questions
1. Implement a sliding window function using itertools.
2. What is the difference between `zip` and `zip_longest`?
3. How do you flatten a list of lists with itertools?
4. Implement `chunked` using itertools.

## Coding Challenges
1. **Chunked:** Write a function that splits an iterable into chunks of size n.
2. **Unique Everseen:** Return items from an iterable, skipping duplicates.
3. **N Cycles:** Cycle through n different iterables, taking one from each.

## Summary
`itertools` is an indispensable module for efficient, memory-conscious iteration. Its building-block design lets you compose complex iteration patterns from simple, tested pieces.

## Related Topics
- 30_generators.md — yield expressions power many itertools patterns
- 31_iterators.md — itertools builds on the iterator protocol
- 33_lambdas.md — often used as key functions in itertools
