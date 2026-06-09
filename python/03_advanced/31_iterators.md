# Iterators - __iter__, __next__, iter(), next(), custom iterators

## Introduction

An iterator is an object that defines a sequence of values and implements the **iterator protocol**, consisting of two methods: `__iter__()` (which returns itself) and `__next__()` (which returns the next value or raises `StopIteration`). Iterators allow uniform traversal of container types, streams, and custom data sources. The `for` loop in Python works by calling `iter()` on the target object to get an iterator, then repeatedly calling `next()` until `StopIteration` is raised.

## Why It Is Important

The iterator protocol is the foundation of iteration in Python. It enables lazy evaluation (values produced on demand), uniform access to diverse data structures (lists, strings, files, dictionaries, sets), and integration with the `for` loop, comprehensions, and `map()`/`filter()` functions. Understanding iterators is essential for creating custom iterable objects and for grasping how generators, `itertools`, and async iteration work under the hood.

## Syntax

```python
# Iterator protocol
class MyIterator:
    def __iter__(self):
        return self

    def __next__(self):
        if no_more_items:
            raise StopIteration
        return next_item

# Iterable protocol
class MyIterable:
    def __iter__(self):
        return MyIterator()

# Built-in functions
iterator = iter(iterable)
item = next(iterator)
item = next(iterator, default_value)  # No StopIteration if provided
```

## Examples

```python
from typing import Iterator, Any, Optional
import sys
```

### Manually Using an Iterator

```python
numbers = [10, 20, 30]
it = iter(numbers)
print(next(it))  # 10
print(next(it))  # 20
print(next(it))  # 30
# print(next(it))  # Raises StopIteration
```

### Using next() with Default

```python
it = iter([1, 2, 3])
print(next(it, "END"))
print(next(it, "END"))
print(next(it, "END"))
print(next(it, "END"))  # "END" instead of StopIteration
```

### for Loop Internals

```python
# Internally, this:
for x in [1, 2, 3]:
    print(x)

# Is equivalent to:
_it = iter([1, 2, 3])
while True:
    try:
        x = next(_it)
    except StopIteration:
        break
    print(x)
```

## Beginner Examples

### Custom Iterator for a Range-like Object

```python
class MyRange:
    """Custom iterator that behaves like built-in range."""
    def __init__(self, start: int, stop: int, step: int = 1) -> None:
        self.current = start
        self.stop = stop
        self.step = step

    def __iter__(self) -> 'MyRange':
        return self

    def __next__(self) -> int:
        if self.current >= self.stop:
            raise StopIteration
        value = self.current
        self.current += self.step
        return value

for i in MyRange(0, 10, 2):
    print(i)
```

### Iterable vs Iterator

```python
# A list is iterable (has __iter__), but not an iterator (no __next__)
my_list = [1, 2, 3]
print(hasattr(my_list, '__iter__'))   # True
print(hasattr(my_list, '__next__'))   # False

# Getting an iterator from it
it = iter(my_list)
print(hasattr(it, '__iter__'))   # True
print(hasattr(it, '__next__'))   # True
print(it is iter(it))            # True -- iterators are also iterable
```

### Iterator for Fibonacci Sequence

```python
class FibonacciIterator:
    def __init__(self, max_count: int) -> None:
        self.max_count = max_count
        self.count = 0
        self.a, self.b = 0, 1

    def __iter__(self) -> 'FibonacciIterator':
        return self

    def __next__(self) -> int:
        if self.count >= self.max_count:
            raise StopIteration
        value = self.a
        self.a, self.b = self.b, self.a + self.b
        self.count += 1
        return value

for fib in FibonacciIterator(10):
    print(fib)
```

### Infinite Iterator

```python
class NaturalNumbers:
    def __iter__(self) -> 'NaturalNumbers':
        return self

    def __next__(self) -> int:
        self.n = getattr(self, 'n', 0) + 1
        return self.n

import itertools
nat = NaturalNumbers()
first_10 = list(itertools.islice(nat, 10))
print(first_10)
```

## Intermediate Examples

### Sentinel-based Iteration

```python
def until_sentinel() -> Iterator[str]:
    """Read input lines until empty string."""
    return iter(input, "")  # iter(callable, sentinel)

# print("Enter text (blank to stop):")
# for line in until_sentinel():
#     print(f"> {line}")
```

### Bidirectional Iterator

```python
class BidirectionalIterator:
    def __init__(self, data: list[Any]) -> None:
        self.data = data
        self._index = 0

    def __iter__(self) -> 'BidirectionalIterator':
        return self

    def __next__(self) -> Any:
        if self._index >= len(self.data):
            raise StopIteration
        value = self.data[self._index]
        self._index += 1
        return value

    def prev(self) -> Any:
        if self._index <= 0:
            raise StopIteration
        self._index -= 1
        return self.data[self._index]

it = BidirectionalIterator([10, 20, 30, 40])
print(next(it))
print(next(it))
print(it.prev())
print(next(it))
```

### Iterator with Peek Support

```python
class PeekableIterator:
    def __init__(self, iterable: Iterable[Any]) -> None:
        self._iterator = iter(iterable)
        self._cache: Optional[Any] = None
        self._has_cache = False

    def __iter__(self) -> 'PeekableIterator':
        return self

    def __next__(self) -> Any:
        if self._has_cache:
            self._has_cache = False
            return self._cache
        return next(self._iterator)

    def peek(self) -> Any:
        if not self._has_cache:
            try:
                self._cache = next(self._iterator)
                self._has_cache = True
            except StopIteration:
                raise StopIteration("No items left to peek")
        return self._cache

it = PeekableIterator([1, 2, 3])
print(it.peek())  # 1
print(next(it))   # 1
print(it.peek())  # 2
```

### Reusable Iterator via Iterable

```python
class Squares:
    """Iterable that can be iterated multiple times (not an iterator itself)."""
    def __init__(self, n: int) -> None:
        self.n = n

    def __iter__(self) -> Iterator[int]:
        for i in range(1, self.n + 1):
            yield i * i

sq = Squares(5)
print(list(sq))
print(list(sq))  # Works again -- fresh iterator each time
```

## Advanced Examples

### Iterator for a Binary Tree Traversal

```python
class TreeNode:
    def __init__(self, value: Any, left: Optional['TreeNode'] = None, right: Optional['TreeNode'] = None) -> None:
        self.value = value
        self.left = left
        self.right = right

class InOrderIterator:
    def __init__(self, root: Optional[TreeNode]) -> None:
        self.stack: list[TreeNode] = []
        self._push_left(root)

    def _push_left(self, node: Optional[TreeNode]) -> None:
        while node:
            self.stack.append(node)
            node = node.left

    def __iter__(self) -> 'InOrderIterator':
        return self

    def __next__(self) -> Any:
        if not self.stack:
            raise StopIteration
        node = self.stack.pop()
        self._push_left(node.right)
        return node.value

root = TreeNode(1,
    TreeNode(2, TreeNode(4), TreeNode(5)),
    TreeNode(3)
)
print(list(InOrderIterator(root)))  # [4, 2, 5, 1, 3]
```

### Iterator for Cartesian Product

```python
class ProductIterator:
    def __init__(self, *iterables: Iterable[Any], repeat: int = 1) -> None:
        self.pools = [tuple(iterable) for iterable in iterables] * repeat
        self.indices = [0] * len(self.pools)
        self._done = any(len(p) == 0 for p in self.pools)

    def __iter__(self) -> 'ProductIterator':
        return self

    def __next__(self) -> tuple[Any, ...]:
        if self._done:
            raise StopIteration
        result = tuple(pool[i] for pool, i in zip(self.pools, self.indices))
        i = len(self.pools) - 1
        while i >= 0:
            self.indices[i] += 1
            if self.indices[i] < len(self.pools[i]):
                break
            self.indices[i] = 0
            i -= 1
        if i < 0:
            self._done = True
        return result

for p in ProductIterator([1, 2], ['a', 'b']):
    print(p)
```

### Lazy CSV File Iterator

```python
import csv
from typing import TextIO

class CSVRowIterator:
    def __init__(self, file_path: str) -> None:
        self.file_path = file_path
        self._file: Optional[TextIO] = None
        self._reader: Optional[Iterator[list[str]]] = None

    def __iter__(self) -> 'CSVRowIterator':
        self._file = open(self.file_path, 'r', newline='', encoding='utf-8')
        self._reader = csv.reader(self._file)
        return self

    def __next__(self) -> list[str]:
        if not self._reader:
            raise StopIteration
        try:
            return next(self._reader)
        except StopIteration:
            if self._file:
                self._file.close()
            raise
```

### Cyclic Iterator

```python
class CyclicIterator:
    def __init__(self, iterable: Iterable[Any]) -> None:
        self._items = list(iterable)
        self._index = 0

    def __iter__(self) -> 'CyclicIterator':
        return self

    def __next__(self) -> Any:
        if not self._items:
            raise StopIteration("Empty iterable")
        value = self._items[self._index]
        self._index = (self._index + 1) % len(self._items)
        return value

import itertools
colors = CyclicIterator(['red', 'green', 'blue'])
first_8 = list(itertools.islice(colors, 8))
print(first_8)
```

### Iterator with Filtering (Predicate-based)

```python
class FilterIterator:
    def __init__(self, iterable: Iterable[Any], predicate) -> None:
        self._iterator = iter(iterable)
        self._predicate = predicate

    def __iter__(self) -> 'FilterIterator':
        return self

    def __next__(self) -> Any:
        while True:
            item = next(self._iterator)
            if self._predicate(item):
                return item

fil = FilterIterator(range(10), lambda x: x % 2 == 0)
print(list(fil))
```

### Chained Iterator

```python
class ChainIterator:
    def __init__(self, *iterables: Iterable[Any]) -> None:
        self._iterators = [iter(it) for it in iterables]
        self._current_idx = 0

    def __iter__(self) -> 'ChainIterator':
        return self

    def __next__(self) -> Any:
        while self._current_idx < len(self._iterators):
            try:
                return next(self._iterators[self._current_idx])
            except StopIteration:
                self._current_idx += 1
        raise StopIteration

chained = ChainIterator([1, 2], [3, 4], [5, 6])
print(list(chained))
```

## Real-World Use Cases

- **Database cursor iteration**: `for row in cursor:` works because cursors implement the iterator protocol.
- **File reading**: `for line in file:` reads line by line without loading the whole file.
- **Pagination**: API pagination wrappers that yield pages or records on demand.
- **Stream processing**: Processing network streams, sensor data, or log files incrementally.
- **Custom collection classes**: Making custom data structures iterable via `__iter__`.
- **Tree/graph traversal**: Depth-first, breadth-first, or custom order traversal using iterators.
- **Lazy evaluation frameworks**: Django QuerySets, pandas iterrows(), etc.

## Common Mistakes

- Confusing iterables with iterators — an iterable can produce many iterators, but itself is not exhausted.
- Forgetting to raise `StopIteration` in `__next__` — causing infinite loops.
- Not implementing `__iter__` in an iterator (should return `self`).
- Attempting to reuse an exhausted iterator — must call `iter()` again.
- Modifying a collection while iterating over it — raises `RuntimeError` for some types.
- Returning `None` instead of raising `StopIteration` at the end.
- Creating iterators that hold file handles or network connections without cleanup.

## Best Practices

- Use `Iterable` and `Iterator` from `typing` for type hints.
- Always implement `__iter__` returning `self` for iterators; for iterables, return a fresh iterator.
- Use `next(iterator, default)` to avoid manual `StopIteration` handling.
- Prefer generators over custom iterator classes for simplicity.
- Document whether your class is an iterable (can be iterated multiple times) or an iterator (single-use).
- Use context managers for iterators that manage resources (files, connections).
- Implement `__length_hint__` for performance hints when possible.

## Interview Questions

1. What is the difference between an iterable and an iterator?
2. How does a `for` loop work internally?
3. What methods must an iterator implement?
4. What is the purpose of `StopIteration`?
5. Can you make an object both iterable and an iterator?
6. How does `iter(callable, sentinel)` work?
7. What is the difference between an iterator and a generator?
8. How do you create a custom iterator?
9. Why does `list(iterator)` exhaust it?
10. What is `__length_hint__` and where is it used?

## Coding Challenges

1. **Range Iterator**: Reimplement `range()` as a custom iterator class.
2. **Zip Iterator**: Implement `zip()` as an iterator class.
3. **Paginated API Iterator**: Create an iterator that fetches pages from a fake API.
4. **Flatten Iterator**: Build an iterator that flattens nested lists.
5. **Round-Robin Iterator**: Create an iterator that cycles through multiple iterables in turn.
6. **Skip Iterator**: Implement an iterator that skips every Nth element.
7. **Windowed Iterator**: Create an iterator that yields sliding windows over a sequence.
8. **Chunked Iterator**: Build an iterator that yields chunks of fixed size from an iterable.

## Summary

Iterators are the backbone of iteration in Python, using the `__iter__()`/`__next__()` protocol and `StopIteration` to signal completion. They enable lazy, memory-efficient traversal and work seamlessly with `for` loops, comprehensions, and built-in functions. Understanding the distinction between iterables and iterators, and how to implement both, is fundamental for advanced Python programming.

## Related Topics

- Generators
- Generator expressions
- itertools module
- for loop internals
- Collections abstract base classes
- Async iterators (__aiter__, __anext__)