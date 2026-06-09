# Data Structures - list vs dict vs set performance, Big O analysis

## Introduction
Choosing the right data structure is the single most impactful performance decision in Python. Lists, dicts, and sets each have different strengths: lists excel at ordered access and sequence operations, dicts provide O(1) key-value lookups, and sets offer fast membership tests and set algebra. Understanding the Big O performance of each operation — insertion, deletion, lookup, iteration — allows you to write code that scales predictably with data size.

## list performance

### What It Is
A Python `list` is a dynamic array of `PyObject*` pointers. It provides O(1) index access, O(1) amortised append at the end, but O(n) insertion or removal at arbitrary positions. Lists are the default choice for ordered sequences.

### Why It Is Important
Lists are the most commonly used data structure in Python. Misusing them — repeatedly inserting at the front, checking membership with `in` on large lists, or concatenating in a loop — leads to O(n^2) or worse performance. Understanding list internals helps you avoid these pitfalls.

### How It Works Internally
A `PyListObject` holds:
- `ob_item`: pointer to an array of `PyObject*`
- `allocated`: current capacity (may be larger than `len`)
- `ob_size`: number of items

When `append` is called and `ob_size == allocated`, the array is reallocated to approximately `1.125 * allocated` (the growth factor). Indexing is pointer arithmetic: `ob_item[index]`. Insert at position 0 requires `memmove` of all existing elements one slot to the right — O(n).

### Syntax
```python
lst = [1, 2, 3]
lst.append(4)          # O(1) amortised
lst.insert(0, 0)       # O(n)
x = lst[2]             # O(1)
lst.pop()              # O(1)
lst.pop(0)             # O(n)
lst.remove(2)          # O(n)
2 in lst               # O(n) average
lst.sort()             # O(n log n)
```

### Beginner Examples
```python
# Append (efficient)
data = []
for i in range(10000):
    data.append(i)

# Prepend (inefficient)
data = []
for i in range(10000):
    data.insert(0, i)  # O(n^2) total!

# Membership test
items = list(range(10000))
print(9999 in items)    # O(n) — checks element by element
```

### Intermediate Examples
```python
import time

# Building a list efficiently
def build_slow(n):
    result = []
    for i in range(n):
        result = result + [i]   # O(n^2) — creates new list each time!
    return result

def build_fast(n):
    result = []
    for i in range(n):
        result.append(i)        # O(n) — amortised
    return result

# List comprehension vs loop
n = 10000
t0 = time.perf_counter()
squares = [i**2 for i in range(n)]
t1 = time.perf_counter()

squares2 = []
for i in range(n):
    squares2.append(i**2)
t2 = time.perf_counter()

print(f'Comprehension: {t1-t0:.4f}s, Loop: {t2-t1:.4f}s')

# Deleting in reverse avoids shifting
def delete_from_front(lst, indices):
    for i in sorted(indices, reverse=True):
        del lst[i]

items = list(range(100))
delete_from_front(items, [0, 10, 20, 30])
```

### Advanced Examples
```python
import bisect
import array

# Using bisect for sorted insert
sorted_list = []
for val in [3, 1, 4, 1, 5, 9, 2, 6]:
    bisect.insort(sorted_list, val)

print(sorted_list)  # [1, 1, 2, 3, 4, 5, 6, 9]

# Custom allocator for performance
class PreallocatedList:
    def __init__(self, capacity):
        self._data = [None] * capacity
        self._size = 0

    def append(self, item):
        if self._size >= len(self._data):
            self._data.extend([None] * len(self._data))  # double
        self._data[self._size] = item
        self._size += 1

    def __getitem__(self, idx):
        if idx < 0 or idx >= self._size:
            raise IndexError
        return self._data[idx]

    def __len__(self):
        return self._size

# array.array for memory-efficient numeric storage
import array
arr = array.array('i', range(1000000))
print(f'List size: {sys.getsizeof(list(range(1000000)))}')
print(f'Array size: {sys.getsizeof(arr)}')

# Slice assignment for batch operations
lst = list(range(10))
lst[2:5] = [100, 200]      # replaces 3 elements with 2
print(lst)
```

### Real-World Use Cases
- **Queue**: use `collections.deque` (O(1) both ends) instead of list for FIFO.
- **Sorted data**: use `bisect.insort` on a list for O(log n) insert + O(n) shift.
- **Matrix storage**: list of lists for 2D data; consider `numpy.ndarray` for numerical work.

### Common Mistakes
- Using `list.index()` in a hot loop — O(n) per call, leading to O(n^2).
- Concatenating lists with `+` in a loop — creates a new list each iteration.
- Using `list.remove()` which scans from the beginning — O(n) per removal.

### Best Practices
- Use `append` and `pop()` (from the end) for stack-like behaviour.
- Use `collections.deque` for queues (O(1) at both ends).
- Use list comprehensions over manual `for` + `append`.
- Use `array.array` or `numpy.ndarray` for homogeneous numeric data.

### Performance Considerations
- Index access: O(1) — fastest possible.
- Append (amortised): O(1) — occasional reallocation.
- Insert at beginning: O(n) — avoid for large lists.
- Membership (`in`): O(n) — use `set` if you need fast membership.
- Sort: O(n log n) — Timsort, very fast for partially sorted data.

### Interview Questions
- **Q**: Why is inserting at the beginning of a list O(n)?  
  **A**: The underlying array must shift all existing elements right by one slot via `memmove`.
- **Q**: How does Python grow a list when it runs out of capacity?  
  **A**: It over-allocates by a factor of ~1.125 (the exact formula is `new_allocated = (size_t)newsize + (newsize >> 3) + (newsize < 9 ? 3 : 6)`).

### Coding Challenges
- Implement a `CircularBuffer` class that supports O(1) append and pop from both ends.
- Write a function that merges two sorted lists in O(n) time without using built-in `sort` or `heapq.merge`.

### Related Topics
- [dict performance](#dict-performance)
- [set performance](#set-performance)
- [Big O analysis](#big-o-notation)

---

## dict performance

### What It Is
A Python `dict` is a hash table mapping hashable keys to values. Insertion, deletion, and lookup are O(1) average case. Dictionaries are the foundation of Python's namespace (function locals, globals, attribute access) and are used extensively for structured data.

### Why It Is Important
dict performance determines the speed of attribute access, function call resolution, JSON parsing, and many core language operations. Understanding when a dict is the right choice (fast key lookup) and when it is not (ordered iteration, memory efficiency) is critical.

### How It Works Internally
CPython 3.6+ uses a compact hash table with two arrays:
1. **Indices array**: sparse `Py_ssize_t` array (size = 2^n, load factor ~2/3).
2. **Entries array**: dense array of `(hash, key, value)` tuples stored in insertion order.

Lookup:
1. Compute `hash(key)`.
2. Mask to index: `idx = indices[hash & mask]`.
3. If `idx == 0` and not a dummy sentinel, probe with open addressing (usually one step).

The compact representation uses ~30% less memory than the old (pre-3.6) implementation and preserves insertion order.

### Syntax
```python
d = {'a': 1, 'b': 2}
d['c'] = 3            # O(1) average
x = d['a']            # O(1) average
del d['b']            # O(1) average
'a' in d              # O(1) average
d.get('x', default)   # O(1) average
```

### Beginner Examples
```python
# dict as counter
text = "hello world"
counter = {}
for ch in text:
    counter[ch] = counter.get(ch, 0) + 1

# dict membership is fast
valid_ids = {1, 2, 3, 4, 5}  # prefer set for membership
# but dict also works:
valid_map = {1: True, 2: True, 3: True}
```

### Intermediate Examples
```python
import time

# dict vs list for lookup
n = 100000
keys = list(range(n))
values = list(range(n))
lookup_dict = dict(zip(keys, values))
lookup_list = values

# List lookup is O(n)
t0 = time.perf_counter()
for k in keys:
    _ = k in lookup_list
t1 = time.perf_counter()

# Dict lookup is O(1)
t2 = time.perf_counter()
for k in keys:
    _ = k in lookup_dict
t3 = time.perf_counter()

print(f'List membership: {t1-t0:.3f}s, Dict membership: {t3-t2:.3f}s')

# Defaultdict for grouped data
from collections import defaultdict

groups = defaultdict(list)
for i in range(1000):
    groups[i % 10].append(i)

# Missing keys with setdefault
data = {}
for i in range(100):
    data.setdefault(i // 10, []).append(i)
```

### Advanced Examples
```python
import sys

# __slots__ vs __dict__ memory comparison
class WithDict:
    def __init__(self, x, y):
        self.x = x
        self.y = y

class WithSlots:
    __slots__ = ('x', 'y')
    def __init__(self, x, y):
        self.x = x
        self.y = y

a = WithDict(1, 2)
b = WithSlots(1, 2)
print(f'WithDict: {sys.getsizeof(a)} + {sys.getsizeof(a.__dict__)}')
print(f'WithSlots: {sys.getsizeof(b)}')

# Custom hash for dict keys
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    def __hash__(self):
        return hash((self.x, self.y))
    def __eq__(self, other):
        return self.x == other.x and self.y == other.y

grid = {}
grid[Point(0, 0)] = 'origin'
grid[Point(1, 0)] = 'east'

# OrderedDict for order-sensitive operations
from collections import OrderedDict
od = OrderedDict()
od['a'] = 1
od['b'] = 2
od.move_to_end('a')   # move 'a' to end

# dict comprehension for filtering
original = {i: i**2 for i in range(100)}
filtered = {k: v for k, v in original.items() if v % 2 == 0}

# ChainMap for layered lookups
from collections import ChainMap
defaults = {'theme': 'dark', 'lang': 'en'}
user_prefs = {'theme': 'light'}
settings = ChainMap(user_prefs, defaults)
print(settings['theme'])   # 'light'
print(settings['lang'])    # 'en'
```

### Real-World Use Cases
- **JSON parsing**: `json.loads` produces nested dicts; attribute-style access via `__getitem__` is O(1).
- **Caching/memoization**: dicts store computed results keyed by function arguments.
- **Graph representation**: adjacency list implemented as `{node: [neighbors]}`.
- **Database indexes**: in-memory index as `{key: row_id}` for O(1) lookups.

### Common Mistakes
- Using mutable objects as dict keys — requires `__hash__` and `__eq__`; lists are unhashable.
- Iterating over a dict while modifying it — use `list(d.items())` to snapshot.
- Assuming dicts are unordered — they are insertion-ordered as of Python 3.7 (language guarantee), but don't rely on this if pre-3.7 compatibility is needed.

### Best Practices
- Use `dict.get(key, default)` or `dict.setdefault` to avoid `KeyError`.
- Use `collections.defaultdict` for automatic missing-key handling.
- Use `dict.fromkeys(seq, value)` to initialise a dict with keys from a sequence.
- Use `sys.getsizeof` to monitor memory usage for large dicts.

### Performance Considerations
- Average O(1) for get/set/del, worst-case O(n) if hash collisions are pathological (rare).
- Resizing: when load factor exceeds ~2/3, dict doubles its index array and rehashes all entries — O(n) but amortised.
- Memory overhead: each key-value pair costs ~72 bytes (PyDictEntry) plus the keys/values themselves.

### Interview Questions
- **Q**: How does Python handle hash collisions in dicts?  
  **A**: CPython uses open addressing with pseudo-random probing. When a collision occurs, it probes the next index in a deterministic sequence derived from the hash.
- **Q**: Why does Python 3.6+ preserve insertion order in dicts?  
  **A**: The compact table implementation stores entries in insertion order in a dense array, while the sparse index array points to them. This was a memory optimisation that incidentally preserved order.

### Coding Challenges
- Implement a `LRUCache` using `OrderedDict` that supports `get` and `put` in O(1).
- Write a function that merges two dicts deeply (nested keys) with `ChainMap`-like semantics.

### Related Topics
- [set performance](#set-performance)
- [list performance](#list-performance)
- [Big O analysis](#big-o-notation)

---

## set performance

### What It Is
A Python `set` is an unordered collection of unique, hashable elements backed by a hash table (identical to dict keys with no values). Sets provide O(1) average membership tests and support standard set operations: union, intersection, difference, and symmetric difference.

### Why It Is Important
Sets are the go-to data structure for membership testing, deduplication, and set algebra. Using a set instead of a list for `in` checks reduces O(n) to O(1) — the single most impactful optimisation for many programs.

### How It Works Internally
A `PySetObject` is essentially a `PyDictObject` with `NULL` values. The hash table is the same compact structure as dicts (Python 3.6+). Elements are stored in insertion order (though sets are conceptually unordered). The same open-addressing collision resolution applies.

### Syntax
```python
s = {1, 2, 3}
s.add(4)              # O(1) average
s.remove(3)           # O(1) average — raises KeyError if missing
s.discard(3)          # O(1) average — no error if missing
1 in s                # O(1) average
len(s)                # O(1)
s1 | s2               # union — O(len(s1) + len(s2))
s1 & s2               # intersection — O(min(len(s1), len(s2)))
s1 - s2               # difference
s1 ^ s2               # symmetric difference
```

### Beginner Examples
```python
# Fast membership test
valid_users = {'alice', 'bob', 'charlie'}
print('alice' in valid_users)   # True — O(1)

# Deduplication
numbers = [1, 2, 2, 3, 3, 3, 4]
unique = list(set(numbers))
print(unique)

# Removing duplicates while preserving order
def unique_ordered(seq):
    seen = set()
    result = []
    for item in seq:
        if item not in seen:
            seen.add(item)
            result.append(item)
    return result
```

### Intermediate Examples
```python
import time

# set vs list for membership
n = 1000000
items_list = list(range(n))
items_set = set(range(n))

t0 = time.perf_counter()
for i in range(10000):
    _ = i in items_list
t1 = time.perf_counter()

t2 = time.perf_counter()
for i in range(10000):
    _ = i in items_set
t3 = time.perf_counter()

print(f'List: {t1-t0:.3f}s, Set: {t3-t2:.3f}s')

# Set operations for data analysis
all_users = {'a', 'b', 'c', 'd', 'e'}
admins = {'a', 'c'}
moderators = {'b', 'd'}

admins_or_mods = admins | moderators
both = admins & moderators
plain_users = all_users - admins - moderators
exactly_one = admins ^ moderators

# Finding duplicates across multiple lists
def find_duplicates(*lists):
    seen = set()
    duplicates = set()
    for lst in lists:
        for item in lst:
            if item in seen:
                duplicates.add(item)
            seen.add(item)
    return duplicates
```

### Advanced Examples
```python
# FrozenSet as dict key
groups = {
    frozenset({'alice', 'bob'}): 'team_a',
    frozenset({'charlie', 'dave'}): 'team_b',
}

# Subset/superset checks
a = {1, 2, 3}
b = {1, 2}
print(b.issubset(a))   # True
print(a.issuperset(b)) # True

# Set as sparse graph
graph = {
    1: {2, 3},
    2: {1, 4, 5},
    3: {1, 6},
    4: {2},
    5: {2},
    6: {3},
}

def bfs(start, graph):
    visited = set()
    queue = [start]
    while queue:
        node = queue.pop(0)
        if node not in visited:
            visited.add(node)
            queue.extend(graph[node] - visited)
    return visited

# Performance tuning with set operations
def common_elements(*lists):
    """Return items that appear in all lists."""
    if not lists:
        return set()
    result = set(lists[0])
    for lst in lists[1:]:
        result &= set(lst)
    return result

# Counting unique elements in stream
from collections import Counter
stream = [1, 2, 2, 3, 3, 3, 1, 4]
unique_count = len(set(stream))

# Bloom filter alternative (approximate membership)
# For very large datasets where set memory is prohibitive
```

### Real-World Use Cases
- **Web crawler**: track visited URLs in a set for O(1) deduplication.
- **Data validation**: check if a value is in a whitelist/blacklist.
- **Graph algorithms**: track visited nodes during BFS/DFS.
- **Spell checker**: store dictionary words in a set for O(1) lookup.

### Common Mistakes
- Adding unhashable types (list, dict, set) to a set — wrap in `tuple` or `frozenset`.
- Expecting set iteration order to be deterministic — it is insertion-ordered in CPython 3.6+ but conceptually unordered.
- Using `set.remove` without checking membership (`set.discard` is safer).

### Best Practices
- Always use `set` for membership tests when order and duplicates are not needed.
- Use `frozenset` when you need an immutable, hashable set (e.g., as a dict key).
- Use `s1.intersection(s2)` for clarity; the `&` operator works but may be less readable.
- For large sets, consider memory overhead (~72 bytes per element).

### Performance Considerations
- Membership: O(1) average, O(n) worst case (hash collisions).
- Union: O(len(s1) + len(s2)) — copies all elements.
- Intersection: O(min(len(s1), len(s2))) — iterates the smaller set.
- Difference: O(len(s1)) — iterates the left set.
- Deduplication via `set(list)` is O(n).
- Memory: ~72 bytes per element plus hash table overhead.

### Interview Questions
- **Q**: What is the difference between `set` and `frozenset`?  
  **A**: `set` is mutable (add/remove/discard), `frozenset` is immutable and hashable (usable as dict key).
- **Q**: How does Python implement `s1 & s2` (intersection) efficiently?  
  **A**: It iterates over the smaller set and checks membership in the larger set (O(min size)), swapping if one set is much smaller.

### Coding Challenges
- Implement a `FuzzySet` that uses a Bloom filter for approximate membership queries with a configurable false-positive rate.
- Write a function that finds all pairs from two lists whose sum is in a target set, using sets for O(n + m) performance.

### Related Topics
- [dict performance](#dict-performance)
- [list performance](#list-performance)
- [Big O analysis](#big-o-notation)

---

## Big O Analysis

### What It Is
Big O notation describes the upper bound of an algorithm's time or space requirements as the input size grows. It abstracts away constants and lower-order terms to focus on scaling behaviour. Common classes: O(1), O(log n), O(n), O(n log n), O(n^2), O(2^n).

### Why It Is Important
Big O analysis lets you predict how your code will perform with real-world data sizes without running benchmarks. An O(n^2) algorithm that works for 100 items will be catastrophically slow for 100,000 items. Choosing an O(n log n) algorithm over O(n^2) is often the difference between a responsive service and a timeout.

### How It Works Internally
Big O counts the number of basic operations (comparisons, assignments, array accesses) as a function of input size `n`. Constants and lower-order terms are dropped: `3n^2 + 5n + 2 = O(n^2)`. Common derivations:
- **O(1)**: dict/set lookup, list index access, arithmetic.
- **O(log n)**: binary search, balanced tree operations.
- **O(n)**: linear search, list copy, single loop.
- **O(n log n)**: sorting (Timsort, mergesort, heapsort).
- **O(n^2)**: nested loops, bubble sort, naive matrix multiply.
- **O(2^n)**: naive recursion (Fibonacci without memoisation).

### Syntax
```python
# O(1) — constant time
def lookup(d, key):
    return d[key]

# O(n) — linear time
def linear_search(lst, target):
    for item in lst:
        if item == target:
            return True
    return False

# O(n^2) — quadratic time
def nested_loop(n):
    for i in range(n):
        for j in range(n):
            pass

# O(n log n) — linearithmic
def sort_and_search(lst):
    lst.sort()              # O(n log n)
    return bisect.bisect(lst, 42)  # O(log n)
```

### Beginner Examples
```python
import time
import random

# O(1): dict lookup
d = {i: i**2 for i in range(1000)}
t0 = time.perf_counter()
_ = d[500]
t1 = time.perf_counter()

# O(n): list lookup
lst = list(range(1000))
t2 = time.perf_counter()
_ = 500 in lst
t3 = time.perf_counter()

# O(n^2): naive nested loop
def quadratic(n):
    total = 0
    for i in range(n):
        for j in range(n):
            total += i * j
    return total

t4 = time.perf_counter()
quadratic(1000)
t5 = time.perf_counter()

print(f'O(1): {t1-t0:.6f}s, O(n): {t3-t2:.6f}s, O(n^2): {t5-t4:.6f}s')
```

### Intermediate Examples
```python
import time
import random
import bisect

# Demonstrating growth rates
def measure_growth():
    for n in [1000, 2000, 4000, 8000]:
        data = list(range(n))
        random.shuffle(data)

        # O(n log n) sort
        t0 = time.perf_counter()
        data.sort()
        t1 = time.perf_counter()

        # O(log n) bisect
        t2 = time.perf_counter()
        bisect.bisect(data, n // 2)
        t3 = time.perf_counter()

        print(f'n={n:5d}: sort={t1-t0:.5f}s, bisect={t3-t2:.7f}s')

# Amortised analysis example
def amortised_append(n):
    lst = []
    total_copies = 0
    prev_capacity = 0
    for i in range(n):
        lst.append(i)
        # Check if capacity changed (reallocation happened)
        import sys
        cap = sys.getsizeof(lst) // 8
        if cap != prev_capacity:
            total_copies += prev_capacity
            prev_capacity = cap
    return total_copies

# Space complexity measurement
def space_complexity_demo():
    for n in [1000, 10000, 100000]:
        d = {i: i**2 for i in range(n)}
        import sys
        size = sys.getsizeof(d) + sum(sys.getsizeof(k) + sys.getsizeof(v) for k, v in d.items())
        print(f'n={n:6d}: dict size ~{size//1024} KB')
```

### Advanced Examples
```python
# Master theorem analysis
# T(n) = aT(n/b) + f(n)
# Example: Merge sort T(n) = 2T(n/2) + O(n) => O(n log n)

# Benchmarking framework
import time
import functools

def benchmark(func, sizes=[100, 1000, 10000, 100000]):
    for n in sizes:
        data = list(range(n))
        t0 = time.perf_counter()
        func(data)
        t1 = time.perf_counter()
        rate = n / (t1 - t0) if (t1 - t0) > 0 else float('inf')
        print(f'{func.__name__}(n={n:6d}): {t1-t0:.5f}s ({rate:.0f} items/s)')

@benchmark
def builtin_sort(data):
    return sorted(data)

# Constant factor analysis
def constant_factors():
    n = 10_000_000
    data = list(range(n))

    # List comprehension vs for loop — both O(n) but constants differ
    import time
    t0 = time.perf_counter()
    result = [x * 2 for x in data]
    t1 = time.perf_counter()

    result2 = []
    for x in data:
        result2.append(x * 2)
    t2 = time.perf_counter()

    print(f'Comprehension: {t1-t0:.3f}s, Loop: {t2-t1:.3f}s')

# Worst-case vs average-case
def worst_case_dict():
    d = {i: i for i in range(1000)}
    # Average case: O(1)
    # Worst case: O(n) if all keys collide (rare, requires hash DoS)

# Space-time tradeoff
def space_time_tradeoff():
    # Precompute factorials (space) vs compute on demand (time)
    import math
    from functools import lru_cache

    @lru_cache(maxsize=1000)
    def factorial(n):
        return n * factorial(n-1) if n > 1 else 1
```

### Real-World Use Cases
- **API design**: document Big O guarantees in function docstrings so callers know the scaling cost.
- **Code review**: flag O(n^2) patterns (nested loops over the same iterable) that will not scale.
- **System design**: choose data structures and algorithms based on expected data volume — e.g., use a set for O(1) lookup when the dataset fits in memory, use a Bloom filter for approximate membership when it does not.

### Common Mistakes
- Forgetting that `in` on a list is O(n), not O(1) — the single most common Big O mistake.
- Ignoring constant factors: an O(n) algorithm with a huge constant may be slower than O(n^2) for small n.
- Treating nested loops as O(n^2) when the inner loop bounds are independent — always check the loop limits.

### Best Practices
- Always ask: "What is the input size and how will it grow?" before choosing an algorithm.
- Prefer O(n log n) sorting over O(n^2) for any n > 1000.
- Use `set` or `dict` instead of `list` when you need membership tests or unique constraints.
- Profile with real data sizes — Big O is a guide, but constants and platform details matter.

### Performance Considerations
- Big O ignores constants: an O(n) linear scan over a list may be faster than an O(1) dict lookup if the list is tiny and hash computation is expensive.
- Real-world performance is influenced by cache locality: arrays (lists) are cache-friendly; hash tables (dicts, sets) involve random memory access.
- Python's interpreter overhead adds a constant factor of ~50x compared to C, but Big O analysis remains the same.

### Interview Questions
- **Q**: What is the Big O of `list.sort()`?  
  **A**: O(n log n) — Python uses Timsort, a hybrid stable sorting algorithm derived from merge sort and insertion sort.
- **Q**: How do you compute the Big O of a recursive function?  
  **A**: Define a recurrence relation T(n) and apply the Master Theorem or the Akra-Bazzi method.

### Coding Challenges
- Implement a function that, given a list of Big O classes, sorts them from fastest to slowest and gives an example algorithm for each.
- Write a decorator that measures the empirical time complexity of a function by running it with increasingly large inputs and printing the estimated Big O.

### Related Topics
- [list performance](#list-performance)
- [dict performance](#dict-performance)
- [set performance](#set-performance)
- [Algorithms](#algorithms---sorting-searching-recursion-dynamic-programming)
