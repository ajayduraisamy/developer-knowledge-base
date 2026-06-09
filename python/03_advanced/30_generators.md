# Generators - yield, generator expressions, send(), yield from

## Introduction

A generator is a special type of iterable that produces a sequence of values lazily — on demand — rather than storing them all in memory at once. Generators are defined using regular functions with the `yield` keyword instead of `return`. When called, a generator function returns a generator object that can be iterated over. Each time `yield` is encountered, the function's state is frozen, and the yielded value is sent to the caller. On the next iteration, execution resumes from where it left off.

## Why It Is Important

Generators are crucial for memory efficiency when working with large or infinite data streams. Instead of building an entire list in memory, generators produce items one at a time, dramatically reducing memory footprint. They enable lazy evaluation — values are computed only when needed, improving performance for expensive computations. Generators also simplify code for sequences that would be cumbersome to produce with traditional iteration, such as recursive tree traversals, Fibonacci sequences, or reading huge files line by line.

## Syntax

```python
# Generator function
def generator_function():
    yield value
    yield value2

# Generator expression (like a list comprehension but with parentheses)
gen = (x * 2 for x in range(10))

# yield from - delegate to another generator
def delegating():
    yield from sub_generator

# send() - send value back into generator
gen.send(value)

# throw() - raise exception inside generator
gen.throw(ExceptionType)

# close() - stop the generator
gen.close()
```

## Examples

```python
from typing import Generator, Any, Optional, Iterator, Union
import sys
import math
```

### Basic Generator Function

```python
def count_up_to(n: int) -> Generator[int, None, None]:
    i = 1
    while i <= n:
        yield i
        i += 1

for num in count_up_to(5):
    print(num)
```

### Generator Expression

```python
squares = (x ** 2 for x in range(1, 6))
print(list(squares))

# Compare memory
list_sq = [x ** 2 for x in range(1000)]
gen_sq  = (x ** 2 for x in range(1000))
print(f"List size: {sys.getsizeof(list_sq)} bytes")
print(f"Gen size: {sys.getsizeof(gen_sq)} bytes")
```

### yield from

```python
def generator_a() -> Generator[str, None, None]:
    for ch in "ABC":
        yield ch

def generator_b() -> Generator[str, None, None]:
    for num in "123":
        yield num

def combined() -> Generator[str, None, None]:
    yield from generator_a()
    yield from generator_b()

print(list(combined()))
```

### send() Method

```python
def echo() -> Generator[str, str, str]:
    """Receives values via send and echoes them back."""
    result = ""
    while True:
        received = yield result
        if received is None:
            break
        result = f"Echo: {received}"
    return "Done"

gen = echo()
next(gen)           # Initialize
print(gen.send("Hello"))
print(gen.send("World"))
try:
    gen.send(None)  # Stops the generator
except StopIteration as e:
    print(e.value)
```

### throw() Method

```python
def safe_divide() -> Generator[float, None, None]:
    x = 0
    while True:
        yield 1.0 / x if x != 0 else float('inf')
        x += 1

gen = safe_divide()
print(next(gen))  # inf
print(next(gen))  # 1.0
print(next(gen))  # 0.5
```

### close() Method

```python
def infinite_counter() -> Generator[int, None, None]:
    i = 0
    try:
        while True:
            yield i
            i += 1
    except GeneratorExit:
        print("Generator closed")

gen = infinite_counter()
print(next(gen))
print(next(gen))
gen.close()
# next(gen)  # Raises StopIteration
```

## Beginner Examples

### Reading Large Files Line by Line

```python
def read_lines(filename: str) -> Generator[str, None, None]:
    with open(filename, 'r', encoding='utf-8') as f:
        for line in f:
            yield line.rstrip('\n')

# for line in read_lines('large_file.txt'):
#     process(line)
```

### Generating Fibonacci Numbers

```python
def fibonacci(limit: Optional[int] = None) -> Generator[int, None, None]:
    a, b = 0, 1
    while limit is None or a < limit:
        yield a
        a, b = b, a + b

print(list(fibonacci(limit=100)))
```

### Generating Infinite Sequence

```python
def natural_numbers() -> Generator[int, None, None]:
    n = 1
    while True:
        yield n
        n += 1

gen = natural_numbers()
for _ in range(5):
    print(next(gen))
```

### Filtering with Generator Expression

```python
numbers = range(1, 21)
evens = (x for x in numbers if x % 2 == 0)
squares_of_evens = (x * x for x in evens)
print(list(squares_of_evens))
```

## Intermediate Examples

### Generator for Prime Numbers (Sieve)

```python
def primes() -> Generator[int, None, None]:
    yield 2
    seen: list[int] = []
    candidate = 3
    while True:
        is_prime = True
        for p in seen:
            if p * p > candidate:
                break
            if candidate % p == 0:
                is_prime = False
                break
        if is_prime:
            seen.append(candidate)
            yield candidate
        candidate += 2

import itertools
first_10_primes = list(itertools.islice(primes(), 10))
print(first_10_primes)
```

### Generator with send() for Coroutine Pattern

```python
def accumulator() -> Generator[float, float, float]:
    total = 0.0
    while True:
        value = yield total
        if value is None:
            return total
        total += value

acc = accumulator()
next(acc)  # Prime it
print(acc.send(10))
print(acc.send(20))
print(acc.send(30))
```

### Chaining Generators for Pipelines

```python
def read_values() -> Generator[int, None, None]:
    for i in range(1, 11):
        yield i

def square(stream: Iterator[int]) -> Generator[int, None, None]:
    for value in stream:
        yield value * value

def even_only(stream: Iterator[int]) -> Generator[int, None, None]:
    for value in stream:
        if value % 2 == 0:
            yield value

pipeline = even_only(square(read_values()))
print(list(pipeline))
```

### Generator for Tree Traversal

```python
class TreeNode:
    def __init__(self, value: Any, left: Optional['TreeNode'] = None, right: Optional['TreeNode'] = None) -> None:
        self.value = value
        self.left = left
        self.right = right

def inorder(node: Optional[TreeNode]) -> Generator[Any, None, None]:
    if node is not None:
        yield from inorder(node.left)
        yield node.value
        yield from inorder(node.right)

# Build tree:
#     1
#    / \
#   2   3
#  / \
# 4   5
root = TreeNode(1,
    TreeNode(2, TreeNode(4), TreeNode(5)),
    TreeNode(3)
)
print(list(inorder(root)))
```

## Advanced Examples

### Two-Way Generator Communication (Coroutine)

```python
from typing import Generator

def coroutine_example() -> Generator[str, str, str]:
    """Demonstrates two-way communication between caller and generator."""
    items = []
    while True:
        item = yield f"Received: {items[-1] if items else 'nothing'}"
        if item == "STOP":
            return f"Final list: {items}"
        items.append(item)

co = coroutine_example()
next(co)  # Prime
print(co.send("apple"))
print(co.send("banana"))
print(co.send("cherry"))
try:
    co.send("STOP")
except StopIteration as e:
    print(e.value)
```

### Generator for Lazy Evaluation of Recursive Combinatorics

```python
def combinations_generator(pool: list[Any], r: int) -> Generator[list[Any], None, None]:
    """Generate combinations without itertools."""
    n = len(pool)
    if r > n:
        return
    indices = list(range(r))
    yield [pool[i] for i in indices]
    while True:
        i = r - 1
        while i >= 0 and indices[i] == i + n - r:
            i -= 1
        if i < 0:
            return
        indices[i] += 1
        for j in range(i + 1, r):
            indices[j] = indices[j - 1] + 1
        yield [pool[i] for i in indices]

for combo in combinations_generator([1, 2, 3, 4], 2):
    print(combo)
```

### Generator-based Stream Processing

```python
def file_line_stream(filename: str) -> Generator[str, None, None]:
    with open(filename, 'r') as f:
        for line in f:
            yield line.strip()

def tokenize(stream: Iterator[str]) -> Generator[list[str], None, None]:
    for line in stream:
        yield line.split()

def filter_by_length(stream: Iterator[list[str]], min_len: int) -> Generator[list[str], None, None]:
    for tokens in stream:
        filtered = [t for t in tokens if len(t) >= min_len]
        if filtered:
            yield filtered

# stream = filter_by_length(tokenize(file_line_stream("data.txt")), 3)
# for line in stream:
#     print(line)
```

### Generator for Batch Processing

```python
def batched(iterable: Iterator[Any], n: int) -> Generator[list[Any], None, None]:
    """Yield successive n-sized chunks from an iterable."""
    batch: list[Any] = []
    for item in iterable:
        batch.append(item)
        if len(batch) == n:
            yield batch
            batch = []
    if batch:
        yield batch

for batch in batched(range(10), 3):
    print(batch)
```

### Generator with Exception Handling

```python
def safe_divide_all(dividends: list[float], divisor: float) -> Generator[Union[float, str], None, None]:
    for d in dividends:
        try:
            yield d / divisor
        except ZeroDivisionError:
            yield "Division by zero"

results = list(safe_divide_all([10, 20, 30, 40], 0))
print(results)
```

### Generator for Flattening Nested Structures

```python
def flatten(nested: Union[list[Any], Any]) -> Generator[Any, None, None]:
    for item in nested:
        if isinstance(item, list):
            yield from flatten(item)
        else:
            yield item

deep = [1, [2, [3, 4], 5], 6]
print(list(flatten(deep)))
```

## Real-World Use Cases

- **Reading large CSV/JSON/XML files** line by line without loading into memory.
- **Streaming data processing** (log analysis, sensor data, real-time feeds).
- **Lazy evaluation of mathematical sequences** (Fibonacci, prime numbers, fractals).
- **Web scraping pipelines** that crawl, parse, and store data incrementally.
- **Database cursor iteration** where each row is yielded one at a time.
- **Log file monitoring** (tail -f equivalent).
- **Paginated API consumers** that fetch and yield pages on demand.
- **Pipeline processing** — chaining transformations via generator functions.

## Common Mistakes

- Using `return` with a value inside a generator — the value is available only through `StopIteration.value`, not the iteration loop.
- Forgetting to prime a coroutine (calling `next()` or `.send(None)`) before using `.send()`.
- Infinite loops in generator expressions that consume resources when iterated eagerly.
- Converting generators to lists unnecessarily, defeating their memory-saving purpose.
- Reusing a generator after exhaustion — generators are single-use; you must recreate them.
- Assuming generators support indexing or len() — they do not.

## Best Practices

- Use generator expressions for simple transformations instead of list comprehensions when you don't need all values at once.
- Prefer delegating with `yield from` for composing generators.
- Name generator functions clearly to indicate they return generators (e.g., `read_file`, `generate_ids`).
- Use type hints: `Generator[YieldType, SendType, ReturnType]`.
- Use `itertools.islice()` to take a finite number of items from infinite generators.
- Close generators explicitly when done if cleanup is needed.
- Avoid mixing `return value` and `yield` unless you understand how `StopIteration` works.

## Interview Questions

1. What is the difference between a generator and a regular function?
2. How does `yield` work and what happens to the function state?
3. What is `yield from` and why is it useful?
4. Explain the `.send()`, `.throw()`, and `.close()` methods.
5. How do generators improve memory efficiency?
6. What is the difference between a generator expression and a list comprehension?
7. Can a generator have a `return` statement?
8. How do you create an infinite generator?
9. What is the difference between `Iterable`, `Iterator`, and `Generator`?
10. How would you implement `range()` using a generator?

## Coding Challenges

1. **Fibonacci Generator**: Write a generator that yields Fibonacci numbers up to N.
2. **File Reader**: Create a generator that reads a file in chunks of N bytes.
3. **Progressive Average**: Write a generator that yields the running average of values sent to it.
4. **Permutations Generator**: Implement a generator that yields all permutations of a list without itertools.
5. **Sliding Window**: Create a generator that yields sliding windows of size K from an iterable.
6. **Lazy CSV Parser**: Write a generator that yields rows from a CSV file as dictionaries.
7. **Generator-based Pipeline**: Build a word count pipeline using chained generators.
8. **Circular Buffer**: Implement a circular buffer generator that keeps the last N items.

## Summary

Generators provide a powerful, lazy, memory-efficient way to produce sequences of values in Python. Defined with `yield` instead of `return`, they maintain state between calls and can receive values through the `.send()` protocol. Combined with generator expressions and `yield from`, generators enable clean, composable data pipelines. They are a cornerstone of Python's iteration protocol and asynchronous programming.

## Related Topics

- Iterators and Iterables
- Coroutines and async/await
- itertools module
- Generator expressions
- Lazy evaluation
- Streaming data processing
- Memory-efficient programming