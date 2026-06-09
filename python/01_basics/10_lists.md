# Lists - list() constructor, indexing, methods, comprehensions

## Introduction

Lists are mutable, ordered sequences that can hold heterogeneous items. They are the most versatile built-in collection type in Python, used everywhere from simple data storage to complex algorithm implementation. Lists support indexing, slicing, a rich set of mutating methods, and concise comprehension syntax for creating lists from iterables.

## `list()` Constructor

### What It Is

The `list()` constructor creates a new list from any iterable. With no arguments, it returns an empty list. It converts strings, tuples, sets, dict keys, generators, range objects, and custom iterables into a list.

### Why It Is Important

`list()` materializes iterables into a concrete sequence, enabling random access, mutation, and list-specific operations. It is essential for consuming generators (to reuse items), converting between collection types, and creating copies of other sequences.

### How It Works Internally

`list(iterable)` calls `PyList_New()` to allocate a list object, then iterates over the argument using `PyObject_GetIter()` / `PyIter_Next()`, appending each item with `list_appends()`. The list grows dynamically using the over-allocation strategy (0, 4, 8, 16, 25, 35, 46, ... extra slots; ~12.5% overallocation for large lists). If the iterable has a known length (e.g., `__len__` is defined), Python pre-allocates the exact capacity to avoid resizing.

### Syntax

```python
list()                            # []
list(iterable)                    # new list from iterable
list(string)                      # ['c', 'h', 'a', 'r', 's']
list(range(n))                    # [0, 1, 2, ..., n-1]
```

### Beginner Examples

```python
# Empty list
empty = list()
print(empty)  # []

# From string
chars = list("hello")
print(chars)  # ['h', 'e', 'l', 'l', 'o']

# From range
nums = list(range(5))
print(nums)  # [0, 1, 2, 3, 4]

# From tuple
items = list((1, 2, 3))
print(items)  # [1, 2, 3]

# From set (order not preserved)
unique = list({3, 1, 2})
print(unique)  # [1, 2, 3] (arbitrary order)

# From dict keys
keys = list({"a": 1, "b": 2})
print(keys)  # ['a', 'b']
```

### Intermediate Examples

```python
# Converting dict items to list of tuples
d = {"x": 10, "y": 20}
pairs = list(d.items())
print(pairs)  # [('x', 10), ('y', 20)]

# Materializing a generator
squares = (x**2 for x in range(5))
squares_list = list(squares)
print(squares_list)  # [0, 1, 4, 9, 16]
print(list(squares))  # []  — generator exhausted

# From custom iterable
class Countdown:
    def __init__(self, n):
        self.n = n
    def __iter__(self):
        while self.n > 0:
            yield self.n
            self.n -= 1

print(list(Countdown(5)))  # [5, 4, 3, 2, 1]

# Copying a list
original = [1, 2, 3]
copy_constructor = list(original)
print(copy_constructor)  # [1, 2, 3]
print(original is copy_constructor)  # False
```

### Advanced Examples

```python
# list() vs [:] — performance and intent
import timeit

def list_copy():
    return list(range(1000))

def slice_copy():
    return range(1000)[:]

# list() is slightly faster for materializing range objects
print(timeit.timeit(list_copy, number=100000))
print(timeit.timeit(slice_copy, number=100000))

# Using list() with map and filter
nums = [1, 2, 3, 4, 5]
doubled = list(map(lambda x: x * 2, nums))
evens = list(filter(lambda x: x % 2 == 0, nums))
print(doubled)  # [2, 4, 6, 8, 10]
print(evens)    # [2, 4]

# list() on memoryview
data = memoryview(b"hello world")
print(list(data))  # [104, 101, 108, 108, 111, ...]

# Deep materialization of nested structures
def flatten_and_list(nested):
    result = []
    for item in nested:
        if isinstance(item, (list, tuple, set)):
            result.extend(flatten_and_list(item))
        else:
            result.append(item)
    return result

nested = [(1, 2), [3, [4, 5]], {6}]
print(flatten_and_list(nested))  # [1, 2, 3, 4, 5, 6]
```

### Real-World Use Cases

- **API Responses**: Converting JSON deserialized dicts/iterables into lists for processing
- **Database Cursors**: Materializing query results from generators into lists for pagination
- **Configuration Parsing**: Converting command-line arguments or config file sections into lists
- **Data Pipelines**: Using `list()` to cache generator results for multiple passes

### Common Mistakes

```python
# Mistake 1: list() on a list creates a new list (not always obvious)
a = [1, 2, 3]
b = list(a)  # New object, but shallow copy
print(a is b)  # False

# Mistake 2: list() on a nested list is still a shallow copy
nested = [[1, 2], [3, 4]]
shallow = list(nested)
shallow[0][0] = 99
print(nested)  # [[99, 2], [3, 4]]  — modified!

# Mistake 3: Forgetting list() on a generator exhausts it
gen = (x for x in [1, 2, 3])
first = list(gen)
second = list(gen)  # Already exhausted
print(first)   # [1, 2, 3]
print(second)  # []

# Mistake 4: list() with no arguments is rarely needed
# [] is more idiomatic than list()
```

### Best Practices

- Use `[]` for empty lists (more idiomatic and slightly faster)
- Use `list(iterable)` to explicitly materialize iterables
- Prefer `list()` over list comprehension when you just need to materialize (e.g., `list(map(...))` vs `[x for x in map(...)]`)
- Use `list(d.keys())` when you need a mutable snapshot of dict keys
- Remember `list()` makes a shallow copy — use `copy.deepcopy()` for nested structures

### Performance Considerations

- `list()` on a sized iterable pre-allocates capacity (avoids resizing)
- `list()` is O(n) where n is the iterable length
- Python 3.12+ optimized `list()` construction from certain iterables (e.g., `range`) using fast paths
- For small iterables, list literal `[...]` may be faster than `list(iterable)`
- `list()` creates a new object — avoid in hot paths where aliasing is acceptable

### Interview Questions

1. What happens when you call `list()` on a generator?
2. How does `list()` handle memory allocation internally?
3. What is the difference between `list(iterable)` and `[iterable]`?
4. Does `list()` create a deep or shallow copy?
5. How does CPython optimize `list(range(n))`?
6. What types can `list()` accept as arguments?
7. How does `list()` behave with an iterator vs an iterable?
8. Can `list()` accept any object? What if it has no `__iter__`?
9. How do you convert a string to a list of characters? To a list of words?
10. What is the time complexity of `list()` on various input types?

### Coding Challenges

```python
# Challenge 1: Custom list() behavior
def materialize(iterable, max_items=None):
    """Convert iterable to list with optional cap."""
    result = []
    for i, item in enumerate(iterable):
        if max_items is not None and i >= max_items:
            break
        result.append(item)
    return result

print(materialize(range(100), 5))  # [0, 1, 2, 3, 4]

# Challenge 2: Lazy materialization with progress
def materialize_with_progress(iterable, label="Progress"):
    items = []
    for i, item in enumerate(iterable):
        items.append(item)
        if (i + 1) % 1000 == 0:
            print(f"{label}: {i + 1} items")
    return items

# Challenge 3: Recursive materialization
def deep_list(obj):
    """Convert nested iterables to nested lists."""
    if hasattr(obj, '__iter__') and not isinstance(obj, (str, bytes)):
        return [deep_list(item) for item in obj]
    return obj

nested = ((1, 2), (3, (4, 5)))
print(deep_list(nested))  # [[1, 2], [3, [4, 5]]]
```

### Related Topics

- `tuple()` constructor
- `set()`, `dict()` constructors
- Generators and Iterators
- Shallow vs Deep Copy
- `collections.abc.Iterable`

## Indexing

### What It Is

Indexing accesses individual elements of a list by their position using square brackets with zero-based indices. Negative indices count from the end (`-1` is the last element). Out-of-range access raises `IndexError`.

### Why It Is Important

Indexing provides O(1) random access to any element, which is the fundamental advantage of arrays/lists over linked structures. It enables direct manipulation, swapping, and positional operations essential for algorithms.

### How It Works Internally

List indexing calls `PyList_GetItem()` which checks bounds, then returns `list->ob_item[index]` — a direct pointer into the underlying C array of `PyObject*` pointers. Negative indices are converted to positive via `index += len(list)`. The C array stores only pointers; the actual objects exist elsewhere on the heap. CPython 3.12+ uses `PyObject*` array with inlined small-object optimization for some types.

### Syntax

```python
lst[i]        # Element at index i (0-based)
lst[-i]       # Element at position len(lst) - i from the end
lst[i] = x    # Assign to position i
```

### Beginner Examples

```python
fruits = ["apple", "banana", "cherry", "date", "elderberry"]

# Positive indexing
print(fruits[0])   # apple
print(fruits[1])   # banana
print(fruits[4])   # elderberry

# Negative indexing
print(fruits[-1])  # elderberry
print(fruits[-2])  # date
print(fruits[-5])  # apple

# Modifying via index
fruits[1] = "blueberry"
print(fruits)  # ['apple', 'blueberry', 'cherry', 'date', 'elderberry']

# IndexError
# print(fruits[10])  # IndexError: list index out of range
# print(fruits[-10])  # IndexError: list index out of range
```

### Intermediate Examples

```python
# Swapping elements
nums = [1, 2, 3, 4, 5]
nums[0], nums[-1] = nums[-1], nums[0]
print(nums)  # [5, 2, 3, 4, 1]

# Using index for in-place algorithm (reverse in place)
def reverse_in_place(lst):
    left, right = 0, len(lst) - 1
    while left < right:
        lst[left], lst[right] = lst[right], lst[left]
        left += 1
        right -= 1

nums = [1, 2, 3, 4, 5]
reverse_in_place(nums)
print(nums)  # [5, 4, 3, 2, 1]

# Nested indexing
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]
print(matrix[1][2])  # 6 (row 1, col 2)

# Using index in loops
words = ["hello", "world", "python"]
for i in range(len(words)):
    words[i] = words[i].upper()
print(words)  # ['HELLO', 'WORLD', 'PYTHON']

# enumerate — better than manual indexing
for i, word in enumerate(words):
    words[i] = word.lower()
print(words)  # ['hello', 'world', 'python']
```

### Advanced Examples

```python
# Implementing __getitem__ and __setitem__ for custom indexing behavior
class RingBuffer:
    def __init__(self, capacity):
        self._buffer = [None] * capacity
        self._capacity = capacity
        self._start = 0
        self._size = 0

    def append(self, item):
        idx = (self._start + self._size) % self._capacity
        self._buffer[idx] = item
        if self._size < self._capacity:
            self._size += 1
        else:
            self._start = (self._start + 1) % self._capacity

    def __getitem__(self, index):
        if isinstance(index, slice):
            return [self[i] for i in range(*index.indices(self._size))]
        if not -self._size <= index < self._size:
            raise IndexError("ring buffer index out of range")
        if index < 0:
            index += self._size
        return self._buffer[(self._start + index) % self._capacity]

    def __setitem__(self, index, value):
        if not -self._size <= index < self._size:
            raise IndexError("ring buffer index out of range")
        if index < 0:
            index += self._size
        self._buffer[(self._start + index) % self._capacity] = value

buf = RingBuffer(3)
buf.append(1); buf.append(2); buf.append(3); buf.append(4)
print(buf[0], buf[1], buf[2])  # 2 3 4
buf[0] = 99
print(buf[0])  # 99

# Advanced: using slice assignment for list rotation
def rotate(lst, k):
    if not lst:
        return
    k %= len(lst)
    lst[:] = lst[-k:] + lst[:-k]

nums = [1, 2, 3, 4, 5]
rotate(nums, 2)
print(nums)  # [4, 5, 1, 2, 3]

# Indexing with custom objects as indices
class Index:
    def __init__(self, value):
        self.value = value
    def __index__(self):
        return self.value

items = ["a", "b", "c", "d"]
idx = Index(2)
print(items[idx])  # c
```

### Real-World Use Cases

- **Data Access**: Random access to records in a dataset
- **In-Place Algorithms**: Sorting, reversing, partitioning
- **Matrix Operations**: Accessing rows, columns, individual cells
- **Buffer Management**: Circular buffers, stacks, queues
- **UI Components**: Accessing list items in a GUI framework

### Common Mistakes

```python
# Mistake 1: Off-by-one errors
items = ["a", "b", "c"]
# print(items[3])  # IndexError — last index is len-1

# Mistake 2: Assuming negative index 0 wraps around
# items[0] is the first element, not items[-0] (which is also 0)

# Mistake 3: Not checking bounds before access
def safe_get(lst, index, default=None):
    try:
        return lst[index]
    except IndexError:
        return default

# Mistake 4: Confusing index with value
nums = [10, 20, 30]
# if nums[nums[0]]:  # Interpreted as nums[10] — IndexError

# Mistake 5: Forgetting assignment by index works in-place
items = [1, 2, 3]
items[1] = 99  # Modifies the list, does NOT return a new list
```

### Best Practices

- Use negative indices for last/from-end access
- Use `enumerate()` instead of manual `range(len(lst))`
- Use slice assignment for bulk modifications instead of element-by-element
- Validate indices when accessing user-provided positions
- Prefer `list[i] = x` over `list.insert(i, x)` when replacing an existing element

### Performance Considerations

- Indexing is O(1) — direct pointer offset into the C array
- Assignment by index is O(1)
- Negative index conversion is O(1) (single addition)
- Python 3.12+ bounds checking is optimized (single branch, predictable)
- Repeated indexing in a loop is fast but caching the value in a local variable is even faster

### Interview Questions

1. What is the time complexity of list indexing?
2. How does Python handle negative indices internally?
3. What happens when you access an out-of-range index?
4. Can you use custom objects as list indices? What protocol do they need?
5. How does slicing differ from indexing in terms of return type?
6. How does indexing work on multi-dimensional lists?
7. What is the difference between `lst[i]` and `lst[i:i+1]`?
8. How does CPython implement `__getitem__` and `__setitem__` for lists?
9. Can you use a float as a list index?
10. How does Python compute the effective index from a negative index?

### Coding Challenges

```python
# Challenge 1: Safe indexing wrapper
class SafeList:
    def __init__(self, items):
        self._items = list(items)

    def __getitem__(self, index, default=None):
        try:
            return self._items[index]
        except IndexError:
            return default

    def __setitem__(self, index, value):
        if isinstance(index, slice):
            self._items[index] = value
        elif 0 <= index < len(self._items):
            self._items[index] = value
        else:
            raise IndexError(f"index {index} out of range")

    def __repr__(self):
        return repr(self._items)

sl = SafeList([1, 2, 3])
print(sl[10])  # None (no crash)

# Challenge 2: Circular indexing
class CircularList:
    def __init__(self, items):
        self._items = list(items)

    def __getitem__(self, index):
        if isinstance(index, slice):
            indices = range(*index.indices(len(self._items)))
            return [self[i] for i in indices]
        return self._items[index % len(self._items)]

    def __setitem__(self, index, value):
        self._items[index % len(self._items)] = value

    def __repr__(self):
        return repr(self._items)

cl = CircularList([1, 2, 3])
print(cl[5])    # 3 (5 % 3 = 2)
print(cl[-1])   # 3 (-1 % 3 = 2)
cl[100] = 99    # 100 % 3 = 1
print(cl)       # [1, 99, 3]

# Challenge 3: Strided indexing
def gather_indices(lst, indices):
    """Gather multiple indices at once."""
    return [lst[i] for i in indices]

print(gather_indices(["a", "b", "c", "d", "e"], [0, 2, 4]))  # ['a', 'c', 'e']
```

### Related Topics

- Slicing
- `enumerate()` Built-in
- Sequence Protocol
- `__getitem__`, `__setitem__` Methods
- `operator.index()`

## List Methods

### What It Is

List methods are built-in functions on `list` objects that provide mutation, searching, and reordering operations. Key methods include `append`, `extend`, `insert`, `remove`, `pop`, `sort`, `reverse`, `index`, `count`, and `copy`.

### Why It Is Important

These methods enable in-place manipulation of lists, which is critical for memory-efficient algorithms, sorting, searching, and data transformation. They are implemented in C and are significantly faster than equivalent Python code.

### How It Works Internally

Each method is a C function in `listobject.c`. `append` calls `app1()` which ensures capacity (reallocating with over-allocation if needed) then stores the pointer. `extend` iterates and appends each item. `insert` calls `ins1()` which shifts elements right via `memmove()`. `pop` retrieves the element and shifts elements left. `sort` uses Timsort (hybrid stable sort, O(n log n) worst-case, O(n) best-case). `remove` scans linearly, finds the first match, and shifts elements.

### Syntax

```python
lst.append(x)           # Add x to end
lst.extend(iterable)    # Add all elements from iterable
lst.insert(i, x)        # Insert x at index i
lst.remove(x)           # Remove first occurrence of x
lst.pop(i)              # Remove and return element at i (or last)
lst.clear()             # Remove all elements
lst.index(x)            # Return index of first x (or ValueError)
lst.count(x)            # Count occurrences of x
lst.sort()              # Sort in-place (modifies list)
lst.reverse()           # Reverse in-place
lst.copy()              # Return shallow copy
```

### Beginner Examples

```python
# Adding elements
nums = []
nums.append(1)
nums.append(2)
nums.append(3)
print(nums)  # [1, 2, 3]

# Extending
nums.extend([4, 5, 6])
print(nums)  # [1, 2, 3, 4, 5, 6]

# Inserting
nums.insert(0, 0)
print(nums)  # [0, 1, 2, 3, 4, 5, 6]

# Removing
nums.remove(3)
print(nums)  # [0, 1, 2, 4, 5, 6]

# Popping
last = nums.pop()
print(last, nums)  # 6 [0, 1, 2, 4, 5]

at_index = nums.pop(2)
print(at_index, nums)  # 2 [0, 1, 4, 5]

# Searching
colors = ["red", "blue", "green", "blue"]
print(colors.index("green"))  # 2
print(colors.count("blue"))   # 2

# Sorting and reversing
nums = [3, 1, 4, 1, 5]
nums.sort()
print(nums)  # [1, 1, 3, 4, 5]
nums.reverse()
print(nums)  # [5, 4, 3, 1, 1]

# Copying
original = [1, 2, 3]
copied = original.copy()
copied.append(4)
print(original)  # [1, 2, 3]
print(copied)    # [1, 2, 3, 4]

# Clearing
nums.clear()
print(nums)  # []
```

### Intermediate Examples

```python
# Custom sort with key
words = ["banana", "apple", "cherry", "date"]
words.sort(key=len)
print(words)  # ['date', 'apple', 'banana', 'cherry']

words.sort(key=lambda w: w[-1])
print(words)  # ['banana', 'apple', 'date', 'cherry']

# Sort with multiple keys (tuple key)
students = [
    ("Alice", 85, "A"),
    ("Bob", 92, "B"),
    ("Charlie", 85, "A"),
]
students.sort(key=lambda s: (-s[1], s[0]))
print(students)
# [('Bob', 92, 'B'), ('Alice', 85, 'A'), ('Charlie', 85, 'A')]

# Using list as stack (LIFO)
stack = []
stack.append("a")
stack.append("b")
stack.append("c")
print(stack.pop())  # c
print(stack.pop())  # b
print(stack)        # ['a']

# Using list as queue (FIFO) — inefficient for large lists
queue = []
queue.append("a")
queue.append("b")
queue.append("c")
print(queue.pop(0))  # a — O(n) shift!
print(queue.pop(0))  # b — O(n) shift!

# Better: collections.deque
from collections import deque
dq = deque(["a", "b", "c"])
print(dq.popleft())  # a — O(1)
print(dq.popleft())  # b — O(1)

# Remove all occurrences
nums = [1, 2, 3, 2, 4, 2]
while 2 in nums:
    nums.remove(2)
print(nums)  # [1, 3, 4]

# Better: list comprehension
nums = [1, 2, 3, 2, 4, 2]
nums[:] = [x for x in nums if x != 2]
```

### Advanced Examples

```python
# Custom sort with __lt__ on objects
class Task:
    def __init__(self, name, priority, deadline):
        self.name = name
        self.priority = priority
        self.deadline = deadline

    def __repr__(self):
        return f"Task({self.name}, pri={self.priority}, due={self.deadline})"

tasks = [
    Task("write_code", 1, 5),
    Task("fix_bug", 3, 1),
    Task("review_pr", 2, 3),
]
tasks.sort(key=lambda t: (t.deadline, -t.priority))
print(tasks)
# [Task(fix_bug, pri=3, due=1), Task(review_pr, pri=2, due=3), Task(write_code, pri=1, due=5)]

# Sort stability guarantee
records = [
    ("Alice", 2020),
    ("Bob", 2019),
    ("Charlie", 2020),
    ("Diana", 2019),
]
records.sort(key=lambda r: r[1])  # Sort by year (stable)
print(records)
# [('Bob', 2019), ('Diana', 2019), ('Alice', 2020), ('Charlie', 2020)]

# extend with generator
def fibonacci(n):
    a, b = 0, 1
    for _ in range(n):
        yield a
        a, b = b, a + b

fibs = []
fibs.extend(fibonacci(10))
print(fibs)  # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

# pop with Sentinel pattern
def process_items(items):
    while items:
        item = items.pop()
        yield item

processed = list(process_items([1, 2, 3, 4]))
print(processed)  # [4, 3, 2, 1]

# Copy vs deep copy with copy method
import copy

class MutableObj:
    def __init__(self, val):
        self.val = val
    def __repr__(self):
        return f"Mutable({self.val})"

objs = [MutableObj(1), MutableObj(2)]
shallow = objs.copy()
deep = copy.deepcopy(objs)

shallow[0].val = 99
print(objs[0].val)   # 99 (shared object)
print(deep[0].val)   # 1 (independent copy)
```

### Real-World Use Cases

- **Data Collection**: `append`/`extend` for building lists from loops
- **Sorting**: `.sort(key=...)` for ordering records by custom criteria
- **Stack Implementation**: `append`/`pop` for LIFO operations
- **Queue Implementation**: `append`/`pop(0)` for FIFO (small lists only)
- **Data Cleaning**: `remove`/comprehension to filter out unwanted items
- **Undo/Redo**: Stack of state snapshots with `append`/`pop`

### Common Mistakes

```python
# Mistake 1: sort() returns None, sorts in-place
nums = [3, 1, 2]
result = nums.sort()
print(result)  # None
print(nums)    # [1, 2, 3]

# Mistake 2: append vs extend
a = [1, 2]
a.append([3, 4])
print(a)  # [1, 2, [3, 4]] — nested!

b = [1, 2]
b.extend([3, 4])
print(b)  # [1, 2, 3, 4]

# Mistake 3: remove only removes first occurrence
nums = [1, 2, 3, 2, 4]
nums.remove(2)
print(nums)  # [1, 3, 2, 4] — second 2 still there

# Mistake 4: pop on empty list
# empty = []
# empty.pop()  # IndexError

# Mistake 5: index raises ValueError, not return -1
colors = ["red", "green"]
# colors.index("blue")  # ValueError

# Mistake 6: insert at negative index
items = [1, 2, 3]
items.insert(-1, 99)  # Inserts before the last element
print(items)  # [1, 2, 99, 3]

# Mistake 7: Forgetting copy() creates shallow copy
```

### Best Practices

- Use `key` parameter for complex sorts (avoid `cmp` functions)
- Use `collections.deque` for queue operations on large lists
- Use list comprehension instead of `remove` in a loop (O(n) vs O(n²))
- Prefer `extend` over repeated `append` in loops
- Use `sorted()` when you need to preserve the original list
- Use `lst[:] = ...` to replace contents without creating a new list object
- Use `lst.copy()` or `lst[:]` for shallow copies (explicit intent)

### Performance Considerations

- `append` is amortized O(1)
- `extend(iterable)` is O(k) where k is iterable length
- `insert(0, x)` is O(n) — shifts all elements right
- `pop()` is O(1); `pop(0)` is O(n)
- `remove(x)` is O(n) — linear scan + shift
- `sort()` is O(n log n) using Timsort
- `reverse()` is O(n)
- `index(x)` is O(n)
- `count(x)` is O(n)
- Python 3.11+ has improved `list.sort()` for partially sorted data
- `copy()` is O(n) — creates new list but shares references

### Interview Questions

1. What is the time complexity of `list.append()` and why is it amortized O(1)?
2. How does `list.sort()` work internally (what algorithm)?
3. What is the difference between `append()` and `extend()`?
4. How does `pop(0)` differ from `pop()` in performance?
5. What does `list.insert()` do with a negative index?
6. How does `list.remove()` find the element to remove?
7. What does `list.sort(key=...)` do vs `sorted(list, key=...)`?
8. Is list.sort() stable? What does that mean?
9. How does `list.copy()` differ from `copy.deepcopy()`?
10. What happens internally when you call `list.clear()`?

### Coding Challenges

```python
# Challenge 1: Custom sort with multiple criteria
def multi_sort(items, *key_funcs):
    """Sort items by multiple keys in order of priority."""
    items = list(items)
    for key_func in reversed(key_funcs):
        items.sort(key=key_func)
    return items

records = [
    ("Alice", 85, 1990),
    ("Bob", 92, 1985),
    ("Charlie", 85, 1992),
]
sorted_recs = multi_sort(records, lambda r: r[2], lambda r: -r[1], lambda r: r[0])
print(sorted_recs)

# Challenge 2: Batch operations
class BatchList:
    def __init__(self, items=None):
        self._items = list(items) if items else []

    def append_many(self, *items):
        self._items.extend(items)

    def remove_all(self, value):
        self._items[:] = [x for x in self._items if x != value]

    def pop_many(self, n):
        result = []
        for _ in range(min(n, len(self._items))):
            result.append(self._items.pop())
        return result

    def __repr__(self):
        return repr(self._items)

bl = BatchList([1, 2, 2, 3, 2, 4])
bl.remove_all(2)
print(bl)  # [1, 3, 4]
print(bl.pop_many(2))  # [4, 3]
print(bl)  # [1]

# Challenge 3: Observable list (logging wrapper)
class ObservableList:
    def __init__(self, items=None):
        self._items = list(items) if items else []
        self._observers = []

    def append(self, item):
        self._items.append(item)
        self._notify("append", item)

    def pop(self, index=-1):
        item = self._items.pop(index)
        self._notify("pop", item)
        return item

    def _notify(self, action, item):
        for obs in self._observers:
            obs(action, item)

    def observe(self, callback):
        self._observers.append(callback)

    def __repr__(self):
        return repr(self._items)

ol = ObservableList([1, 2, 3])
ol.observe(lambda action, item: print(f"Observed: {action} -> {item}"))
ol.append(4)  # Observed: append -> 4
ol.pop()      # Observed: pop -> 4
```

### Related Topics

- `collections.deque`
- Sorting Algorithms (Timsort)
- `sorted()` Built-in
- `operator` Module (`itemgetter`, `attrgetter`)
- `bisect` Module

## List Comprehensions

### What It Is

A list comprehension is a concise syntax for creating lists by applying an expression to each element of an iterable, optionally with filtering. It follows the form `[expression for item in iterable if condition]`.

### Why It Is Important

List comprehensions are more readable, faster, and more Pythonic than equivalent `for` loop + `append()` patterns. They are a hallmark of Python code style and are extensively used in data processing, filtering, and transformation.

### How It Works Internally

CPython compiles list comprehensions into a separate code object (a nested function scope in Python 3). The `LIST_APPEND` bytecode opcode appends each computed value. The comprehension runs in its own scope, avoiding variable leakage (fixed in Python 3). The C implementation avoids function call overhead per iteration compared to `map()`/`filter()` with lambdas. Python 3.12+ further optimizes comprehension bytecode.

### Syntax

```python
[expression for item in iterable]
[expression for item in iterable if condition]
[expression for item1 in iterable1 for item2 in iterable2]
[expr if cond else alt for item in iterable]
```

### Beginner Examples

```python
# Basic comprehension
squares = [x**2 for x in range(10)]
print(squares)  # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# With filter
evens = [x for x in range(20) if x % 2 == 0]
print(evens)  # [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]

# Transforming strings
names = ["alice", "bob", "charlie"]
capitalized = [name.capitalize() for name in names]
print(capitalized)  # ['Alice', 'Bob', 'Charlie']

# Cartesian product
colors = ["red", "green"]
sizes = ["S", "M", "L"]
items = [(c, s) for c in colors for s in sizes]
print(items)
# [('red', 'S'), ('red', 'M'), ('red', 'L'), ('green', 'S'), ('green', 'M'), ('green', 'L')]

# Applying function
nums = [1, 2, 3, 4, 5]
doubled = [x * 2 for x in nums]
print(doubled)  # [2, 4, 6, 8, 10]
```

### Intermediate Examples

```python
# Conditional expression (ternary in comprehension)
nums = [1, 2, 3, 4, 5]
label = ["even" if x % 2 == 0 else "odd" for x in nums]
print(label)  # ['odd', 'even', 'odd', 'even', 'odd']

# Nested comprehensions (flatten matrix)
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat = [num for row in matrix for num in row]
print(flat)  # [1, 2, 3, 4, 5, 6, 7, 8, 9]

# Multiple conditions
nums = range(50)
result = [x for x in nums if x % 2 == 0 if x % 3 == 0]
# Equivalent to: x % 2 == 0 and x % 3 == 0
print(result)  # [0, 6, 12, 18, 24, 30, 36, 42, 48]

# Dict to list transformation
data = {"a": 1, "b": 2, "c": 3}
pairs = [(k, v) for k, v in data.items()]
print(pairs)  # [('a', 1), ('b', 2), ('c', 3)]

# Using enumerate
words = ["hello", "world", "python"]
indexed = [(i, w.upper()) for i, w in enumerate(words)]
print(indexed)  # [(0, 'HELLO'), (1, 'WORLD'), (2, 'PYTHON')]

# Flatten uneven nesting
mixed = [[1, 2], 3, [4, [5, 6]], 7]
def flatten(lst):
    result = []
    for item in lst:
        if isinstance(item, list):
            result.extend(flatten(item))
        else:
            result.append(item)
    return result
print(flatten(mixed))  # [1, 2, 3, 4, 5, 6, 7]
```

### Advanced Examples

```python
# Walrus operator (:=) in comprehensions (Python 3.8+)
import math
nums = [1, 4, 9, 16, 25]
roots = [root for x in nums if (root := math.isqrt(x)) > 2]
print(roots)  # [3, 4, 5]

# Comprehension with side effects (generally avoided)
processed = []
def process(x):
    processed.append(x)
    return x * 2

result = [process(x) for x in [1, 2, 3]]
print(processed)  # [1, 2, 3]
print(result)     # [2, 4, 6]

# Using zip in comprehension
names = ["Alice", "Bob", "Charlie"]
scores = [85, 92, 78]
grades = [(n, s, "A" if s >= 90 else "B" if s >= 80 else "C") for n, s in zip(names, scores)]
print(grades)
# [('Alice', 85, 'B'), ('Bob', 92, 'A'), ('Charlie', 78, 'C')]

# Chained comprehensions (transposed matrix)
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
transposed = [[row[i] for row in matrix] for i in range(3)]
print(transposed)  # [[1, 4, 7], [2, 5, 8], [3, 6, 9]]

# Set-like deduplication with order preserved
def unique_preserve_order(items):
    seen = set()
    return [x for x in items if not (x in seen or seen.add(x))]

print(unique_preserve_order([3, 1, 2, 1, 3, 4]))  # [3, 1, 2, 4]

# Comprehensions with itertools for complex logic
import itertools

def powerset(iterable):
    """Generate all subsets of a set."""
    s = list(iterable)
    return [list(comb) for r in range(len(s) + 1) for comb in itertools.combinations(s, r)]

print(powerset([1, 2]))  # [[], [1], [2], [1, 2]]

# Memory-efficient comprehension with generator expression fallback
# For large iterables, use generator instead:
big = (x**2 for x in range(10_000_000))  # Generator — lazy, no memory allocation
first_10 = [next(big) for _ in range(10)]
print(first_10)
```

### Real-World Use Cases

- **Data Cleaning**: `[clean(x) for x in raw_data]`
- **Filtering**: `[x for x in data if valid(x)]`
- **Feature Engineering**: `[x**2 for x in numeric_features]`
- **Matrix Operations**: Flattening, transposing, mapping over rows
- **API Response Transformation**: Reshaping JSON data into lists
- **Parsing**: `[line.strip() for line in file if line.strip()]`

### Common Mistakes

```python
# Mistake 1: Parentheses vs brackets (generator vs list)
# (x for x in range(10)) — generator, not list!
# [x for x in range(10)] — list

# Mistake 2: Overly complex comprehensions (readability)
# Bad: [[func(x) for x in row if cond(x)] for row in matrix if any(cond(x) for x in row)]
# Better: break into steps or use a function

# Mistake 3: Multiple for clauses order
# [x for y in list1 for x in y] — outer loop first, inner loop second
# This is equivalent to:
# for y in list1:
#     for x in y:
#         result.append(x)

# Mistake 4: Variable leakage (Python 2 only — fixed in Python 3)
# In Python 3, comprehension variables don't leak to outer scope

# Mistake 5: Modifying the source while iterating
# Never modify the list you're iterating over in a comprehension

# Mistake 6: Using comprehension just for side effects
# Bad: [print(x) for x in items]  # Creates a list of None!
# Good: for x in items: print(x)
```

### Best Practices

- Keep comprehensions under 80 characters; break into multiple lines or use a for loop for complex logic
- Use generator expressions `(...)` instead of list comprehensions for large/unbounded iterables
- Prefer `any()`/`all()` over comprehensions with `if` for boolean checks
- Use `dict` and `set` comprehensions when creating non-list collections
- Use `:=` (walrus operator) sparingly in comprehensions
- Nest comprehensions at most 2 levels deep

### Performance Considerations

- List comprehensions are faster than `for` loop + `append()` (~30-50% faster)
- Filtering with `if` is efficient — the condition is evaluated per item
- Nested comprehensions create intermediate lists in memory
- For very large results, use generator expressions to avoid memory blowup
- Python 3.12+ comprehension bytecode is further optimized (faster local variable access)
- `map()` with a pre-defined function can be faster than comprehension for simple transformations
- `filter()` with `None` as predicate is faster than comprehension for truthiness filtering

### Interview Questions

1. How does a list comprehension differ from a generator expression?
2. What is the scope of variables inside a list comprehension?
3. How do you write a nested loop in a list comprehension?
4. How do you add an if-else condition in a list comprehension?
5. What is the order of evaluation in `[f(x) for x in items if g(x)]`?
6. How do list comprehensions perform compared to `map()` and `filter()`?
7. Can you use `break` or `continue` inside a list comprehension?
8. How does Python compile list comprehensions internally (Python 3 vs 2)?
9. How do you create a list of all combinations from multiple lists?
10. How does the walrus operator (`:=`) work inside a comprehension?

### Coding Challenges

```python
# Challenge 1: One-liner prime sieve using comprehension
def primes_up_to(n):
    """Return list of primes <= n using comprehension + all()."""
    return [x for x in range(2, n + 1) if all(x % d != 0 for d in range(2, int(x**0.5) + 1))]

print(primes_up_to(30))  # [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]

# Challenge 2: Flatten list of arbitrary depth
def deep_flatten(nested):
    return [item for sublist in nested for item in (deep_flatten(sublist) if isinstance(sublist, list) else [sublist])]

print(deep_flatten([1, [2, [3, 4]], 5]))  # [1, 2, 3, 4, 5]

# Challenge 3: Comprehensions for data validation
def validate_and_clean(records, *, validators, cleaners):
    """Apply validators and cleaners to a list of records."""
    return [
        cleaner(record)
        for record in records
        if all(validator(record) for validator in validators)
        for cleaner in cleaners
    ]

records = [{"name": "Alice", "age": 30}, {"name": "B", "age": 15}, {"name": "Charlie", "age": 25}]
validators = [lambda r: len(r["name"]) > 1, lambda r: r["age"] >= 18]
cleaners = [lambda r: {**r, "name": r["name"].strip()}]
cleaned = validate_and_clean(records, validators=validators, cleaners=cleaners)
print(cleaned)
# [{'name': 'Alice', 'age': 30}, {'name': 'Charlie', 'age': 25}]

# Challenge 4: Matrix multiplication with comprehensions
def matrix_multiply(A, B):
    """Multiply two matrices using nested comprehensions."""
    n = len(A)
    m = len(B[0])
    p = len(B)
    return [[sum(A[i][k] * B[k][j] for k in range(p)) for j in range(m)] for i in range(n)]

A = [[1, 2], [3, 4]]
B = [[5, 6], [7, 8]]
print(matrix_multiply(A, B))  # [[19, 22], [43, 50]]
```

### Related Topics

- Generator Expressions
- `map()`, `filter()` Built-in Functions
- Dict Comprehensions
- Set Comprehensions
- `itertools` Module
- Functional Programming in Python
- Walrus Operator (`:=`)
