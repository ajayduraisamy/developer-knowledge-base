# Iterators - __iter__, __next__, iter(), next(), custom iterators

## Introduction

Iterators are the engine behind Python's `for` loop and all iteration constructs. Any object that can produce a sequence of values one at a time via the iterator protocol is an iterator. The protocol consists of two methods: `__iter__()` (return the iterator object itself) and `__next__()` (return the next value or raise `StopIteration`). Understanding iterators is fundamental to mastering Python's data processing capabilities.

## __iter__ and __next__

### What It Is

`__iter__` and `__next__` are the two methods that define the iterator protocol in Python. `__iter__` should return the iterator object (usually `self`). `__next__` should return the next value in the sequence, raising `StopIteration` when there are no more items. Together, they allow an object to be used in `for` loops and other iteration contexts.

### Why It Is Important

The iterator protocol provides a uniform interface for iterating over any sequence of values, whether the underlying data is a list, a file, a database cursor, or a generated sequence. This abstraction allows functions and loops to work with any iterable without knowing its internal structure, enabling polymorphic iteration.

### How It Works Internally

When Python executes a `for x in obj` loop, it calls `iter(obj)` which invokes `obj.__iter__()` to get an iterator. Then it repeatedly calls `next(iterator)` which invokes `iterator.__next__()`, catching `StopIteration` to exit the loop. This protocol is implemented at the C level for built-in types and is the foundation of all iteration in Python.

### Syntax

```python
class IteratorClass:
    def __iter__(self):
        return self  # Usually return self for iterator objects

    def __next__(self):
        if not more_items:
            raise StopIteration
        return next_value
```

### Beginner Examples

```python
class CountDown:
    def __init__(self, start):
        self.current = start

    def __iter__(self):
        return self

    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        value = self.current
        self.current -= 1
        return value

for num in CountDown(5):
    print(num)  # 5, 4, 3, 2, 1

# Equivalent manual iteration
counter = iter(CountDown(3))
print(next(counter))  # 3
print(next(counter))  # 2
print(next(counter))  # 1
# print(next(counter))  # StopIteration
```

### Intermediate Examples

```python
class FileLineReader:
    def __init__(self, filename):
        self.filename = filename
        self.file = None

    def __iter__(self):
        self.file = open(self.filename)
        return self

    def __next__(self):
        line = self.file.readline()
        if not line:
            self.file.close()
            raise StopIteration
        return line.strip()

# Usage: for line in FileLineReader("data.txt"): print(line)


class Range:
    """Custom range implementation."""
    def __init__(self, start, stop, step=1):
        self.current = start
        self.stop = stop
        self.step = step

    def __iter__(self):
        return self

    def __next__(self):
        if (self.step > 0 and self.current >= self.stop) or \
           (self.step < 0 and self.current <= self.stop):
            raise StopIteration
        value = self.current
        self.current += self.step
        return value

print(list(Range(1, 10, 2)))  # [1, 3, 5, 7, 9]
```

### Advanced Examples

```python
class CircularBuffer:
    def __init__(self, items):
        self.items = items
        self.index = 0

    def __iter__(self):
        self.index = 0
        return self

    def __next__(self):
        if self.index >= len(self.items):
            raise StopIteration
        value = self.items[self.index]
        self.index = (self.index + 1) % len(self.items)
        return value

    def infinite_cycle(self):
        while True:
            yield from self.items

# Iterator vs iterable separation
class IterableCollection:
    def __init__(self, data):
        self.data = data

    def __iter__(self):
        return Iterator(self.data)

class Iterator:
    def __init__(self, data):
        self.data = data
        self.index = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.index >= len(self.data):
            raise StopIteration
        value = self.data[self.index]
        self.index += 1
        return value

# Multiple independent iterators
collection = IterableCollection([1, 2, 3])
it1 = iter(collection)
it2 = iter(collection)
print(next(it1))  # 1
print(next(it2))  # 1 (independent)
```

### Real-World Use Cases

- **Database cursors**: Database drivers implement iterators over query results.
- **File objects**: Python's file objects are iterators over lines.
- **Network streams**: Iterator over incoming data packets.
- **Paginated API responses**: Iterator that transparently fetches next pages.
- **Lazy data loading**: Iterators over chunks of large datasets.

### Common Mistakes

- Making an object iterable but not an iterator (needs `__iter__` only for iterable, both for iterator).
- Returning `self` from `__iter__` when the object is not an iterator (use separate iterator class).
- Not resetting state when `__iter__` is called on a reusable iterator.
- Forgetting to raise `StopIteration` (causes infinite loop).
- Implementing `__next__` without `__iter__` (object won't work in `for` loops directly).

### Best Practices

- Separate iterables (have `__iter__` return a new iterator) from iterators (implement both).
- Use generators instead of custom iterator classes for simple cases.
- Ensure `__iter__` returns a fresh iterator for iterables (supports multiple passes).
- Raise `StopIteration` cleanly — don't use `return` as a substitute.
- Implement `__length_hint__` if the iterator can estimate remaining items.

### Performance Considerations

Iterator overhead is minimal — each `__next__` call is a single Python function call. The real cost is in the computation per item. File iterators buffer reads for efficiency. Custom iterators should avoid per-item overhead by processing in batches where possible.

### Interview Questions

**Q: What is the difference between an iterable and an iterator?**

A: An iterable has `__iter__()` that returns an iterator. An iterator has both `__iter__()` (returns self) and `__next__()` (raises `StopIteration` when done). All iterators are iterables, but not vice versa. Lists, tuples, and strings are iterables but not iterators.

**Q: What happens when you call `next()` on a generator?**

A: It executes the generator function until the next `yield`, returns the yielded value, and suspends. On `StopIteration`, the generator is exhausted.

**Q: Can you reset an iterator?**

A: No standard way. You must create a new iterator from the iterable. Some custom iterators can add a `reset()` method, but that's non-standard.

### Coding Challenges

1. Implement an `Iterator` class that iterates over only the even-indexed elements of a sequence.
2. Create a `PeekableIterator` that implements `__next__` and has a `peek()` method to look at the next value without consuming it.
3. Implement a `ZipIterator` that yields tuples from multiple iterables, stopping when any is exhausted.
4. Build an iterator that yields all permutations of a list without using `itertools`.

### Related Topics

- Generators (the easiest way to create iterators)
- `StopIteration` exception
- `for` loop internals
- `itertools` module (advanced iterator tools)
- Async iterators (`__aiter__`, `__anext__`)

## iter() and next()

### What It Is

`iter()` is a built-in function that returns an iterator for a given object. `next()` is a built-in function that retrieves the next item from an iterator. These are the standard Python interfaces for working with iterators without explicit `for` loop syntax.

### Why It Is Important

`iter()` and `next()` give programmers fine-grained control over iteration. They are essential for manual iteration, implementing custom iteration patterns (such as partial consumption or conditional advancement), and working with iterators in contexts where `for` loops are inconvenient.

### How It Works Internally

`iter(obj)` calls `obj.__iter__()`. If `obj` supports the sequence protocol (`__getitem__` with integer indices starting at 0), Python creates a sequence iterator that calls `__getitem__` with increasing indices until `IndexError`. `next(iterator)` calls `iterator.__next__()`.

### Syntax

```python
iterator = iter(iterable)           # Get iterator
item = next(iterator)               # Get next item
item = next(iterator, default)      # Get next or default if exhausted
```

### Beginner Examples

```python
fruits = ["apple", "banana", "cherry"]
it = iter(fruits)

print(next(it))  # apple
print(next(it))  # banana
print(next(it))  # cherry
# print(next(it))  # StopIteration

# With default value
it = iter([1, 2, 3])
print(next(it, "done"))  # 1
print(next(it, "done"))  # 2
print(next(it, "done"))  # 3
print(next(it, "done"))  # done (no exception)
```

### Intermediate Examples

```python
def consume(iterator, n):
    """Consume n items from an iterator and return them as a list."""
    result = []
    for _ in range(n):
        try:
            result.append(next(iterator))
        except StopIteration:
            break
    return result

it = iter(range(100))
first_10 = consume(it, 10)
print(first_10)  # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
next_5 = consume(it, 5)
print(next_5)    # [10, 11, 12, 13, 14]

# iter() with callable and sentinel
def read_blocks(file_obj, block_size=1024):
    return iter(lambda: file_obj.read(block_size), "")

with open("data.txt") as f:
    for block in read_blocks(f):
        print(f"Read block of {len(block)} bytes")
```

### Advanced Examples

```python
class PaginatedAPI:
    def __init__(self, base_url, page_size=50):
        self.base_url = base_url
        self.page_size = page_size
        self.page = 0
        self.items = []
        self.index = 0

    def fetch_page(self, page_num):
        import requests
        resp = requests.get(
            f"{self.base_url}?page={page_num}&size={self.page_size}"
        )
        return resp.json().get("results", [])

    def __iter__(self):
        return self

    def __next__(self):
        if self.index >= len(self.items):
            self.page += 1
            self.items = self.fetch_page(self.page)
            self.index = 0
            if not self.items:
                raise StopIteration
        value = self.items[self.index]
        self.index += 1
        return value

# Manual control with next() and sentinel
def find_first(predicate, iterable):
    """Return the first item matching predicate, or None."""
    return next((item for item in iterable if predicate(item)), None)

first_even = find_first(lambda x: x % 2 == 0, [1, 3, 5, 7, 8, 9])
print(first_even)  # 8
```

### Real-World Use Cases

- **Processing head of a stream**: Use `next()` to check first item before processing rest.
- **Combining iterators**: `zip()` and `map()` work with `iter()` internally.
- **Sentinel-based reading**: `iter(f.read, '')` for reading file chunks until empty.
- **Default values**: `next(it, default)` to avoid `StopIteration` when iterable might be empty.
- **Skipping items**: Call `next()` multiple times to skip header lines.

### Common Mistakes

- Forgetting that `iter()` raises `TypeError` for non-iterables.
- Using `next()` on an exhausted iterator without a default (raises `StopIteration`).
- Calling `iter()` on an iterator (returns `self`, which is already consumed).
- Expecting `iter()` to work on objects without `__iter__` or `__getitem__`.

### Best Practices

- Use the two-argument form of `next(iterator, default)` to avoid `StopIteration`.
- Prefer `for` loops over manual `next()` for full iteration.
- Use `iter(callable, sentinel)` for repeated function calls until a termination condition.
- Use `next()` with generator expressions for "find first" patterns.

### Performance Considerations

`iter()` and `next()` are C-optimized and very fast. The two-argument `next()` avoids exception handling overhead by returning a default instead of raising `StopIteration`. The `iter(callable, sentinel)` form is more efficient than an equivalent `while` loop because the sentinel check is done in C.

### Interview Questions

**Q: What is the sentinel form of `iter()` and when is it useful?**

A: `iter(callable, sentinel)` repeatedly calls `callable` until it returns `sentinel`. It's useful for reading blocks from files (`iter(f.read, '')`) or processing streams until a termination marker.

**Q: How does `iter()` handle objects without `__iter__`?**

A: If `__iter__` is not defined, `iter()` looks for `__getitem__`. If found, it creates a sequence iterator that calls `__getitem__` with increasing indices starting from 0, catching `IndexError` to stop. This allows iteration over legacy sequence-like objects.

### Coding Challenges

1. Implement `iter()` manually using only `__getitem__` and `IndexError`.
2. Use `iter(callable, sentinel)` to implement a `until` function that calls a function until it returns a specific value.
3. Write a function `batched(iterable, n)` that uses `iter()` and `next()` to yield chunks of `n` items.

### Related Topics

- `for` loop internals
- Generator functions
- `StopIteration` exception
- `itertools.islice` (for slicing iterators)
- Sequence protocol (`__getitem__`)

## Custom Iterators

### What It Is

Custom iterators are user-defined classes that implement the iterator protocol (`__iter__` and `__next__`). They allow creating objects that can produce sequences of values tailored to specific business logic, such as traversing custom data structures, generating algorithmic sequences, or wrapping external resources.

### Why It Is Important

Custom iterators encapsulate iteration logic in a reusable, testable class. They provide a clean interface for complex iteration patterns that cannot be expressed with simple generators or built-in types. They are essential when the iteration logic involves maintaining state, managing resources, or implementing domain-specific traversal rules.

### How It Works Internally

Custom iterators work exactly like built-in iterators at the protocol level. The Python interpreter calls `__iter__` to obtain an iterator and `__next__` to advance it. The `StopIteration` exception signals completion. The key difference is that custom iterators can maintain arbitrary state, interact with external systems, and implement complex termination logic.

### Syntax

```python
class CustomIterator:
    def __init__(self, *args):
        # Initialize state
        pass

    def __iter__(self):
        return self  # Usually self for iterators

    def __next__(self):
        if not self._has_more():
            raise StopIteration
        return self._next_value()
```

### Beginner Examples

```python
class EvenNumbers:
    def __init__(self, limit):
        self.limit = limit
        self.current = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.current > self.limit:
            raise StopIteration
        value = self.current
        self.current += 2
        return value

for num in EvenNumbers(10):
    print(num)  # 0, 2, 4, 6, 8, 10
```

### Intermediate Examples

```python
class FibonacciIterator:
    def __init__(self, max_value=None, max_count=None):
        self.max_value = max_value
        self.max_count = max_count
        self.count = 0
        self.a, self.b = 0, 1

    def __iter__(self):
        return self

    def __next__(self):
        if self.max_count and self.count >= self.max_count:
            raise StopIteration
        if self.max_value is not None and self.a > self.max_value:
            raise StopIteration
        value = self.a
        self.a, self.b = self.b, self.a + self.b
        self.count += 1
        return value

fib = FibonacciIterator(max_count=10)
print(list(fib))  # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]


class BatchIterator:
    """Iterate over an iterable in batches."""
    def __init__(self, iterable, batch_size):
        self.iterator = iter(iterable)
        self.batch_size = batch_size
        self.exhausted = False

    def __iter__(self):
        return self

    def __next__(self):
        if self.exhausted:
            raise StopIteration
        batch = []
        for _ in range(self.batch_size):
            try:
                batch.append(next(self.iterator))
            except StopIteration:
                self.exhausted = True
                break
        if not batch:
            raise StopIteration
        return batch

data = list(range(10))
for batch in BatchIterator(data, 3):
    print(batch)  # [0,1,2], [3,4,5], [6,7,8], [9]
```

### Advanced Examples

```python
class SmarterFibonacci:
    """Fibonacci iterator implementing the full iterator protocol."""
    def __init__(self, max_value=None):
        self.max_value = max_value

    def __iter__(self):
        return FibonacciIterator(self.max_value)


class TreeIterator:
    """Iterator over a binary tree (in-order traversal)."""
    def __init__(self, root):
        self.stack = []
        self._push_left(root)

    def _push_left(self, node):
        while node:
            self.stack.append(node)
            node = node.left

    def __iter__(self):
        return self

    def __next__(self):
        if not self.stack:
            raise StopIteration
        node = self.stack.pop()
        self._push_left(node.right)
        return node.value
```

### Real-World Use Cases

- **Tree/graph traversal**: In-order, pre-order, post-order, BFS, DFS iterators over custom data structures.
- **Database result sets**: Iterators that fetch rows lazily from a database cursor.
- **Sensor data streams**: Iterators over hardware device readings.
- **Protocol parsers**: Iterators over parsed messages from a byte stream.
- **Backtracking solvers**: Iterators that yield valid solutions to constraint satisfaction problems.

### Common Mistakes

- Making an iterator that cannot be reused (needs separate iterable/iterator classes for multiple passes).
- Not resetting state in `__iter__` when the iterator should be restartable.
- Implementing resource cleanup only in `__del__` instead of providing a `close()` method.
- Yielding mutable internal state that the caller modifies (defensive copy needed).
- Implementing `__getitem__` for iteration but not `__iter__` (works but is slower and less idiomatic).

### Best Practices

- Separate the iterable (creates iterators) from the iterator (consumes the iterable).
- Use generators for simple iteration patterns, custom classes for complex state management.
- Support `with` statements for resource cleanup in iterators that manage external resources.
- Provide a `close()` method and implement `__del__` for proper resource cleanup.
- Consider implementing `__length_hint__` for performance optimization.

### Performance Considerations

Custom iterators avoid creating intermediate collections, saving memory. The function call overhead per `__next__` is the same as for generators. For compute-intensive `__next__` methods, consider batching computations or using caching. Resource-based iterators should use buffering to reduce system call overhead.

### Interview Questions

**Q: How would you implement a reusable iterator that can be iterated over multiple times?**

A: Create a separate iterable class with `__iter__` that returns a new iterator instance each time. The iterator class implements `__next__` with independent state.

**Q: What is the advantage of a custom iterator over a generator function?**

A: Custom iterators can maintain complex state across methods, support resource cleanup protocols, be serializable, and implement additional methods beyond `__iter__` and `__next__` (like `peek()`, `close()`, or `reset()`).

### Coding Challenges

1. Implement a `ReverseIterator` that iterates backwards over a sequence.
2. Create a `MergeIterator` that takes two sorted iterables and yields items in sorted order.
3. Implement a `WindowIterator` that yields sliding windows of a fixed size over an iterable.
4. Build an iterator that yields all subsets of a given set.

### Related Topics

- Iterable protocol (`__iter__` only)
- Generator functions (simpler alternative to custom iterators)
- `itertools` module
- Sequence protocol (`__getitem__`)
- Async iterators (`__aiter__`, `__anext__`)
