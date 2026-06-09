# Tuples - tuple() constructor, immutability, packing/unpacking

## Introduction

Tuples are ordered, immutable sequences in Python, created with parentheses `()` or the `tuple()` constructor. Unlike lists, tuples cannot be modified after creation — no item assignment, append, remove, or pop. Their immutability makes them hashable (usable as dictionary keys and set elements) and suitable for representing fixed collections of data. Tuples also support packing (creating a tuple without parentheses) and unpacking (assigning tuple elements to multiple variables), enabling elegant, concise code patterns. Python's `namedtuple` and typing extensions further enhance tuple capabilities for production use.

## tuple() constructor

### What It Is

The `tuple()` constructor creates a tuple from any iterable — lists, strings, ranges, generators, or other tuples. It accepts zero or one argument. With no argument, it returns an empty tuple.

### Why It Is Important

The constructor provides a uniform interface for converting other sequence types into immutable tuples, enabling hashability, safe dictionary-key usage, and memory-efficient storage of fixed data.

### How It Works Internally

When `tuple()` is called on an iterable, Python iterates over the argument and copies each element into a new `PyTupleObject` internal structure. This is a contiguous array of `PyObject*` pointers, allocated exactly once at creation time — which is why tuples are fixed-size and cannot grow or shrink. The conversion involves an O(n) copy but yields a compact, immutable structure that shares elements (not deep-copies them).

### Syntax

```python
# Empty tuple
t = tuple()

# From iterable
t = tuple([1, 2, 3])
t = tuple("hello")
t = tuple(range(5))
t = tuple(x**2 for x in range(3))
```

### Beginner Examples

```python
# Converting a list to tuple
colors = ["red", "green", "blue"]
colors_tuple = tuple(colors)
print(colors_tuple)  # ('red', 'green', 'blue')

# Converting a string to tuple of characters
word = "Python"
chars = tuple(word)
print(chars)  # ('P', 'y', 't', 'h', 'o', 'n')

# Converting a range
nums = tuple(range(1, 6))
print(nums)  # (1, 2, 3, 4, 5)

# Empty tuple
empty = tuple()
print(len(empty))  # 0
```

### Intermediate Examples

```python
# Generator expression to tuple
squares = tuple(x**2 for x in range(10))
print(squares)  # (0, 1, 4, 9, 16, 25, 36, 49, 64, 81)

# Tuple of tuples from nested list
matrix = tuple(tuple(row) for row in [[1, 2], [3, 4]])
print(matrix)  # ((1, 2), (3, 4))

# Using tuple() for deduplication via set round-trip
items = [1, 2, 2, 3, 3, 3]
unique_tuple = tuple(set(items))
print(unique_tuple)

# Converting dict views
d = {"a": 1, "b": 2}
keys_tuple = tuple(d.keys())
values_tuple = tuple(d.values())
print(keys_tuple, values_tuple)
```

### Advanced Examples

```python
# Memory comparison with lists
import sys

t = tuple(range(1000))
l = list(range(1000))
print(f"Tuple size: {sys.getsizeof(t)} bytes")
print(f"List size: {sys.getsizeof(l)} bytes")

# tuple() with map
nums = tuple(map(lambda x: x * 2, range(5)))
print(nums)  # (0, 2, 4, 6, 8)

# Recursive tuple flattening
def flatten_tuple(t):
    result = []
    for item in t:
        if isinstance(item, tuple):
            result.extend(flatten_tuple(item))
        else:
            result.append(item)
    return tuple(result)

nested = (1, (2, 3), (4, (5, 6)))
print(flatten_tuple(nested))  # (1, 2, 3, 4, 5, 6)
```

### Real-World Use Cases

- **Database row representation**: Converting query results into tuples for immutable data transfer
- **API response parsing**: Converting JSON arrays to tuples for safe caching
- **Memoization keys**: Transforming list-based arguments into tuple keys for cached functions
- **Coordinate data**: Creating hashable point representations from user input

### Common Mistakes

```python
# Mistake 1: Using tuple() on a non-iterable
# tuple(5)  # TypeError: 'int' object is not iterable

# Mistake 2: Forgetting tuple() with generator (double wrapping)
gen = (x for x in range(3))
wrong = tuple(gen)
print(wrong)  # (0, 1, 2) -- correct, but if you pass a tuple of generators...

# Mistake 3: Assuming tuple() deep-copies
inner = [1, 2]
t = tuple([inner, 3])
inner.append(99)
print(t)  # ([1, 2, 99], 3) -- inner list is shared!
```

### Best Practices

- Use `tuple()` over list literal `[]` when the sequence must be immutable
- Prefer `tuple(iterable)` over manual loops for readability
- Use `tuple()` for safe conversion before using as dict keys or set members
- Combine `tuple()` with `map()` or comprehensions for clean transformations

### Performance Considerations

Tuple creation via `tuple()` is O(n). Tuples consume less memory than lists (about 4–8 bytes per element less on CPython) because lists allocate capacity overhead for future appends, while tuples allocate exactly the needed space. Use `tuple()` when the resulting collection will never change size.

### Interview Questions

1. What iterables can `tuple()` accept?
2. Does `tuple()` create a shallow or deep copy?
3. How does `tuple()` handle generators?
4. Why is `tuple([1, 2, 3])` preferred over manual looping?
5. Can `tuple()` be used to deduplicate elements?

### Coding Challenges

```python
# Challenge 1: Custom tuple() implementation
def make_tuple(iterable):
    result = []
    for item in iterable:
        result.append(item)
    return tuple(result)

print(make_tuple("abc"))  # ('a', 'b', 'c')

# Challenge 2: tuple() with filter
def filtered_tuple(iterable, predicate):
    return tuple(item for item in iterable if predicate(item))

print(filtered_tuple(range(10), lambda x: x % 2 == 0))  # (0, 2, 4, 6, 8)

# Challenge 3: Batched tuple conversion
def batch_to_tuples(data, batch_size):
    return [tuple(data[i:i+batch_size]) for i in range(0, len(data), batch_size)]

print(batch_to_tuples(range(10), 3))  # [(0,1,2), (3,4,5), (6,7,8), (9,)]
```

### Related Topics

- Lists and list() constructor
- Iterables and iterators
- Generator expressions
- Memory management in CPython
- Hashable types

## Immutability

### What It Is

Tuple immutability means that once a tuple is created, its elements cannot be added, removed, or replaced. The tuple object itself maintains a fixed identity and sequence of references for its entire lifetime. However, if a tuple contains a mutable object (like a list), that mutable object's contents can still be changed.

### Why It Is Important

Immutability guarantees data integrity — a tuple cannot be accidentally modified after creation. This makes tuples safe to share across functions, threads, and contexts without defensive copying. Immutability also makes tuples hashable, which is a prerequisite for use as dictionary keys and set elements.

### How It Works Internally

CPython implements `PyTupleObject` as a fixed-size C array of `PyObject*` pointers. The `ob_item` array is allocated once during `PyTuple_New()` and the individual slots are filled during initialization. No mutation methods exist in the C API for tuples — `PyTuple_SetItem()` is used only during initialization and raises an error on an already-initialized tuple. The `tp_hash` slot is populated by computing a hash over all elements at first request and caching it on the object, which is safe precisely because the contents never change.

### Syntax

```python
# Immutability means no item assignment
t = (1, 2, 3)
# t[0] = 10   # TypeError

# No append, extend, pop, remove, insert, clear
# t.append(4)   # AttributeError
# t.remove(2)   # AttributeError

# Mutable objects inside tuples CAN change
t = (1, [2, 3], "hello")
t[1].append(4)
print(t)  # (1, [2, 3, 4], "hello")
```

### Beginner Examples

```python
# Attempting to modify
t = (10, 20, 30)
try:
    t[0] = 99
except TypeError as e:
    print(f"Cannot modify: {e}")

# Reassignment is not modification
t = (1, 2)
t = (3, 4)    # New tuple, original unchanged
print(t)      # (3, 4)

# Concatenation creates new tuple
a = (1, 2)
b = (3, 4)
c = a + b
print(c)      # (1, 2, 3, 4) -- new object
print(a)      # (1, 2) -- unchanged

# Tuple as dictionary key
cache = {}
cache[(1, 2)] = "value"  # Works
# cache[[1, 2]] = "value"  # TypeError: unhashable type: 'list'
```

### Intermediate Examples

```python
# Mutable inside immutable
t = ([1, 2], [3, 4])
t[0].append(99)
print(t)  # ([1, 2, 99], [3, 4])

# Immutability prevents accidental modification in functions
def process(data):
    # data is tuple -- safe from modification
    result = sum(data)
    return result

data = (1, 2, 3, 4, 5)
result = process(data)
print(data)  # Still (1, 2, 3, 4, 5) -- guaranteed

# Thread safety example
import threading
shared_data = (1, 2, 3)

def reader():
    print(shared_data[0])

threads = [threading.Thread(target=reader) for _ in range(10)]
for t in threads: t.start()
for t in threads: t.join()
# No race condition possible -- tuple is immutable

# Hash caching
t = (1, 2, 3)
h = hash(t)  # Computed once, cached on the tuple object
print(h)
```

### Advanced Examples

```python
# Immutability enables __slots__ optimization
class Point:
    __slots__ = ("x", "y")
    def __init__(self, x, y):
        self.x = x
        self.y = y

# Tuple-based lightweight alternative
from collections import namedtuple
PointNT = namedtuple("PointNT", ["x", "y"])
p = PointNT(10, 20)
print(p.x, p.y)

# Using tuple for caching/memoization
def memoize(fn):
    cache = {}
    def wrapper(*args):
        key = tuple(args)  # Immutable key
        if key not in cache:
            cache[key] = fn(*args)
        return cache[key]
    return wrapper

@memoize
def fib(n):
    return n if n < 2 else fib(n-1) + fib(n-2)

print(fib(100))  # Fast due to tuple-based memoization

# Shallow immutability verification
def is_deeply_immutable(obj):
    if isinstance(obj, tuple):
        return all(is_deeply_immutable(item) for item in obj)
    if isinstance(obj, (int, float, str, bool, type(None))):
        return True
    return False

t = (1, "hello", (2, 3))
print(is_deeply_immutable(t))  # True
t2 = (1, [2, 3])
print(is_deeply_immutable(t2))  # False
```

### Real-World Use Cases

- **Dictionary keys**: Geographic coordinates, composite identifiers
- **Configuration constants**: Immutable application settings
- **Multi-threaded data**: Safe sharing without locks
- **Function memoization**: Tuple argument keys for cached results
- **Namedtuples**: Lightweight immutable data records

### Common Mistakes

```python
# Mistake 1: Assuming tuples are fully immutable
t = ([1], [2])
t[0].append(99)  # This works!
print(t)  # ([1, 99], [2])

# Mistake 2: Thinking += creates a new tuple for mutable elements
t = ([1], [2])
t[0] += [3]  # TypeError + the list IS modified!
# The += on the list modifies it in-place, then tries to assign back

# Mistake 3: Forgetting that tuple immutability prevents sorting in-place
t = (3, 1, 2)
# t.sort()  # AttributeError
sorted_t = tuple(sorted(t))  # Correct approach
```

### Best Practices

- Use tuples when you need guaranteed immutability
- Avoid placing mutable objects (lists, dicts, sets) inside tuples
- Prefer namedtuples or dataclasses with `frozen=True` for structured immutable data
- Use tuples for multi-threaded constant data sharing
- Leverage tuple immutability for cache keys

### Performance Considerations

Tuples are faster than lists for iteration (CPython can skip bounds-checking in certain paths due to fixed size). Access is O(1). The hash is computed once per tuple and cached, making repeated hash lookups fast. Tuple creation overhead is less than list growth overhead because no overallocation occurs. For temporary immutable collections, tuples are always the right choice.

### Interview Questions

1. Are tuples truly immutable? Explain with examples.
2. Why can tuples be dictionary keys but lists cannot?
3. How does tuple immutability affect thread safety?
4. What happens when you put a list inside a tuple?
5. How does Python cache tuple hashes?
6. Why is tuple immutability considered "shallow"?

### Coding Challenges

```python
# Challenge 1: Deep-freeze a nested structure
def deep_freeze(obj):
    if isinstance(obj, (list, tuple)):
        return tuple(deep_freeze(item) for item in obj)
    if isinstance(obj, dict):
        return tuple(sorted((k, deep_freeze(v)) for k, v in obj.items()))
    if isinstance(obj, set):
        return frozenset(deep_freeze(item) for item in obj)
    return obj

data = [1, [2, 3], {"a": [4, 5]}]
frozen = deep_freeze(data)
print(frozen)  # (1, (2, 3), (('a', (4, 5)),))
print(hash(frozen))  # Hashable

# Challenge 2: Safe container wrapper
class ImmutableContainer:
    def __init__(self, data):
        self._data = tuple(data)

    def get(self, index):
        return self._data[index]

    def __iter__(self):
        return iter(self._data)

    def __len__(self):
        return len(self._data)

c = ImmutableContainer([1, 2, 3])
print(list(c))

# Challenge 3: Verify tuple safety in threads
import threading

results = []
def worker(t):
    results.append(t[0] + t[1])

shared = (10, 20)
threads = [threading.Thread(target=worker, args=(shared,)) for _ in range(100)]
for t in threads: t.start()
for t in threads: t.join()
print(sum(results))  # 3000 -- no race conditions
```

### Related Topics

- Lists (mutable counterpart)
- Frozenset
- Hashable types
- Thread safety
- Namedtuple and frozen dataclasses

## Packing and unpacking

### What It Is

Packing is creating a tuple by listing comma-separated values, optionally without parentheses. Unpacking is the reverse — assigning a tuple's elements to individual variables in a single statement. Python also supports extended unpacking with `*` to capture remaining elements, and starred expressions for unpacking into function calls.

### Why It Is Important

Packing and unpacking enable clean, readable code for multiple assignment, function return values, variable swapping, and iterable destructuring. They eliminate boilerplate indexing and make intent explicit.

### How It Works Internally

When Python encounters comma-separated values without brackets, the compiler generates a `BUILD_TUPLE` bytecode instruction that creates a tuple from the stack values. On the unpacking side, `UNPACK_SEQUENCE` pops a tuple from the stack and pushes each element individually. Extended unpacking (`*`) uses `UNPACK_EX`, which splits the sequence into prefix, middle (starred), and suffix sections based on the number of variables.

### Syntax

```python
# Packing
t = 1, 2, 3
t = 1,                 # Single element tuple

# Unpacking
a, b, c = (1, 2, 3)
a, b, c = 1, 2, 3      # Works without parentheses on right

# Extended unpacking
first, *middle, last = (1, 2, 3, 4, 5)

# Unpacking into function call
def f(a, b, c): pass
f(*some_tuple)

# Swapping
a, b = b, a
```

### Beginner Examples

```python
# Basic unpacking
point = (10, 20)
x, y = point
print(f"x={x}, y={y}")

# Packing without parentheses
coordinates = 10, 20, 30
print(type(coordinates))  # <class 'tuple'>

# Swapping variables
a, b = 5, 10
a, b = b, a
print(f"a={a}, b={b}")  # a=10, b=5

# Returning multiple values
def min_max(nums):
    return min(nums), max(nums)

low, high = min_max([3, 7, 1, 9, 4])
print(f"low={low}, high={high}")

# Single-element tuple packing
single = 5,
print(type(single), single)  # <class 'tuple'> (5,)
```

### Intermediate Examples

```python
# Extended unpacking
first, *rest = (1, 2, 3, 4, 5)
print(f"first={first}, rest={rest}")

first, *middle, last = (1, 2, 3, 4, 5)
print(f"first={first}, middle={middle}, last={last}")

# Unpacking in for loops
pairs = [(1, 2), (3, 4), (5, 6)]
for x, y in pairs:
    print(f"Point({x}, {y})")

# Unpacking with enumerate
fruits = ("apple", "banana", "cherry")
for idx, fruit in enumerate(fruits):
    print(f"{idx}: {fruit}")

# Unpacking with zip
names = ("Alice", "Bob")
scores = (85, 92)
for name, score in zip(names, scores):
    print(f"{name}: {score}")

# Unpacking dict items
d = {"a": 1, "b": 2}
for key, value in d.items():
    print(f"{key}={value}")

# Ignoring values with underscore
_, second, _ = (1, 2, 3)
print(second)  # 2
```

### Advanced Examples

```python
# Starred unpacking in function calls
def f(a, b, c, d):
    return a + b + c + d

nums = (1, 2, 3, 4)
print(f(*nums))  # 10

# Combining multiple unpackings
first = (1, 2)
second = (3, 4)
combined = (*first, *second)
print(combined)  # (1, 2, 3, 4)

# Nested unpacking
point = (1, (2, 3), 4)
a, (b, c), d = point
print(f"a={a}, b={b}, c={c}, d={d}")

# Unpacking with * in assignment (Python 3.10+)
match point:
    case (x, y):
        print(f"2D: {x}, {y}")
    case (x, y, z):
        print(f"3D: {x}, {y}, {z}")

# Advanced: head/tail decomposition
def head_tail(iterable):
    it = iter(iterable)
    first = next(it)
    return first, tuple(it)

h, t = head_tail([1, 2, 3, 4, 5])
print(f"head={h}, tail={t}")

# Multiplexing with zip and unpacking
columns = [("Alice", "Bob", "Charlie"), (85, 92, 78)]
names, scores = zip(*columns)
print(names)   # ('Alice', 'Bob', 'Charlie')
print(scores)  # (85, 92, 78)
```

### Real-World Use Cases

- **Function returns**: Returning multiple values (status, result) from functions
- **Data destructuring**: Extracting fields from database rows
- **Configuration**: Parsing fixed-format strings into typed variables
- **Batch processing**: Unpacking batches of records into separate lists
- **Pattern matching**: Destructuring complex nested data (Python 3.10+ match/case)

### Common Mistakes

```python
# Mistake 1: Unpacking mismatch
# a, b = (1, 2, 3)  # ValueError: too many values to unpack
# Fix with extended unpacking:
a, *rest = (1, 2, 3)

# Mistake 2: Forgetting comma for single-element tuple
single = (5)    # int, not tuple
single = (5,)   # tuple

# Mistake 3: Multiple starred expressions
# *a, *b = (1, 2, 3, 4)  # SyntaxError: two starred expressions in assignment

# Mistake 4: Unpacking non-iterable
# a, b = 5  # TypeError
a, b = 5, 6  # Correct

# Mistake 5: Modifying unpacked mutable elements
inner = [1]
a, b = (inner, [2])
a.append(99)  # Modifies the tuple's element!
```

### Best Practices

- Use unpacking instead of indexed access for cleaner code
- Use `_` for unused values in unpacking
- Use `*` to capture variable-length remaining elements
- Use `a, b = b, a` for swapping instead of a temp variable
- Use starred expressions for merging tuples
- Avoid more than one starred expression per unpacking

### Performance Considerations

Packing and unpacking are highly optimized in CPython. `BUILD_TUPLE` and `UNPACK_SEQUENCE` are simple bytecodes with minimal overhead. Swapping via `a, b = b, a` is faster than using a temporary variable because it avoids a separate store and load. Extended unpacking (`UNPACK_EX`) has slightly more overhead but is still efficient for typical tuple sizes.

### Interview Questions

1. What is tuple packing and unpacking?
2. How does extended unpacking with `*` work?
3. Can you use unpacking in function definitions and calls?
4. What is the difference between `a, b = b, a` and using a temp variable?
5. How do you handle variable-length unpacking?
6. What are the limitations of starred expressions?

### Coding Challenges

```python
# Challenge 1: Rotate tuple elements
def rotate(t):
    if len(t) < 2:
        return t
    first, *rest = t
    return (*rest, first)

print(rotate((1, 2, 3, 4)))  # (2, 3, 4, 1)

# Challenge 2: Interleave two tuples
def interleave(t1, t2):
    result = []
    for a, b in zip(t1, t2):
        result.extend([a, b])
    return tuple(result)

print(interleave((1, 3, 5), (2, 4, 6)))  # (1, 2, 3, 4, 5, 6)

# Challenge 3: Split tuple at index
def split_at(t, index):
    *front, _ = t[:index+1]
    return tuple(front), t[index:]

print(split_at((1, 2, 3, 4, 5), 2))  # ((1, 2), (3, 4, 5))

# Challenge 4: Batch unpacking
def process_batch(batch):
    ids, names, scores = zip(*batch)
    return list(ids), list(names), list(scores)

data = [(1, "Alice", 85), (2, "Bob", 92), (3, "Charlie", 78)]
ids, names, scores = process_batch(data)
print(ids, names, scores)

# Challenge 5: Transpose with unpacking
def transpose(matrix):
    return tuple(zip(*matrix))

m = ((1, 2, 3), (4, 5, 6))
print(transpose(m))  # ((1, 4), (2, 5), (3, 6))
```

### Related Topics

- Extended iterable unpacking (PEP 3132)
- Star expressions in function definitions (`*args`)
- Pattern matching (match/case)
- Zip and enumerate
- Variable swapping techniques
