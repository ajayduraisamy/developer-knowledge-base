# Collections Module - namedtuple, Counter, deque, defaultdict

## Introduction

The `collections` module provides specialized container datatypes that extend and enhance Python's built-in containers (list, dict, set, tuple). These data structures are optimized for specific use cases, offering improved performance, memory efficiency, and convenience compared to their built-in counterparts. The module includes `namedtuple`, `defaultdict`, `Counter`, `deque`, `ChainMap`, `OrderedDict`, and user-customizable base classes (`UserList`, `UserDict`, `UserString`).

## Why It Is Important

The `collections` module is essential for writing idiomatic, efficient Python code. Each container solves a specific problem: `namedtuple` creates lightweight, immutable data objects; `defaultdict` eliminates missing-key checks; `Counter` simplifies counting and frequency analysis; `deque` provides O(1) appends/pops from both ends; `ChainMap` combines multiple dictionaries into a single view; and `OrderedDict` maintains insertion order. Understanding when to use each is a hallmark of advanced Python programming.

## Syntax

```python
from collections import (
    namedtuple, defaultdict, Counter, deque,
    ChainMap, OrderedDict, UserList, UserDict, UserString
)

# namedtuple
Point = namedtuple('Point', ['x', 'y'])
p = Point(10, 20)

# defaultdict
dd = defaultdict(int)
dd['key'] += 1

# Counter
cnt = Counter('abracadabra')

# deque
dq = deque([1, 2, 3])
dq.appendleft(0)

# ChainMap
combined = ChainMap(dict1, dict2)

# OrderedDict
od = OrderedDict()
od['a'] = 1
```

## Examples

```python
from collections import (
    namedtuple, defaultdict, Counter, deque,
    ChainMap, OrderedDict, UserDict, UserList, UserString
)
from typing import Any, Optional
import csv
import json
```

### namedtuple

```python
Point = namedtuple('Point', ['x', 'y'])
p = Point(10, 20)
print(p.x, p.y)
print(p[0], p[1])
print(p)

# Named tuple with defaults
Person = namedtuple('Person', ['name', 'age', 'city'], defaults=['Unknown'])
alice = Person('Alice', 30)
print(alice)
print(alice._asdict())

# _replace
bob = alice._replace(name='Bob', age=25)
print(bob)
```

### defaultdict

```python
# Default factory: int (default value 0)
word_counts = defaultdict(int)
text = "the quick brown fox jumps over the lazy dog"
for word in text.split():
    word_counts[word] += 1
print(dict(word_counts))

# Default factory: list
groups = defaultdict(list)
for i in range(10):
    groups[i % 3].append(i)
print(dict(groups))

# Default factory: set
sets = defaultdict(set)
sets['a'].add(1)
sets['a'].add(2)
sets['b'].add(3)
print(dict(sets))

# Custom default factory
def default_value() -> str:
    return "N/A"
config = defaultdict(default_value)
print(config['host'])
```

### Counter

```python
# Counting elements
cnt = Counter('abracadabra')
print(cnt)
print(cnt['a'])
print(cnt.most_common(3))

# Counter operations
c1 = Counter(['a', 'b', 'c', 'a', 'b', 'a'])
c2 = Counter(['a', 'b', 'd', 'd'])
print(c1 + c2)  # Union (sum counts)
print(c1 - c2)  # Difference (keep positive)
print(c1 & c2)  # Intersection (min)
print(c1 | c2)  # Union (max)

# Update and subtract
cnt.update('abc')
print(cnt)
cnt.subtract('a' * 3)
print(cnt)
```

### deque

```python
dq = deque(maxlen=5)

# Append from both sides
dq.append(1)
dq.appendleft(2)
dq.extend([3, 4, 5])
dq.extendleft([0, -1])
print(dq)  # Dropped oldest (1) due to maxlen

# Pop from both sides
print(dq.pop())
print(dq.popleft())

# Rotation
dq2 = deque(range(1, 6))
dq2.rotate(2)   # Rotate right
print(dq2)
dq2.rotate(-1)  # Rotate left
print(dq2)
```

## Beginner Examples

### namedtuple for CSV Data

```python
# Reading CSV as named tuples
csv_data = [
    "name,age,city",
    "Alice,30,New York",
    "Bob,25,London",
    "Charlie,35,Tokyo",
]

Record = namedtuple('Record', csv_data[0].split(','))
records = [Record(*row.split(',')) for row in csv_data[1:]]
for r in records:
    print(f"{r.name} ({r.age}) from {r.city}")
```

### defaultdict for Grouping

```python
# Group items by category
data = [
    ("fruit", "apple"),
    ("fruit", "banana"),
    ("vegetable", "carrot"),
    ("fruit", "cherry"),
    ("vegetable", "broccoli"),
]

grouped = defaultdict(list)
for category, item in data:
    grouped[category].append(item)
print(dict(grouped))
```

### Counter for Text Analysis

```python
text = "Python is amazing and Python is powerful. Python is easy to learn."
words = text.lower().replace('.', '').split()
word_freq = Counter(words)
print(word_freq.most_common(5))
```

### deque as a Queue

```python
# Efficient FIFO queue
queue = deque()
queue.append("task1")
queue.append("task2")
queue.append("task3")

while queue:
    task = queue.popleft()
    print(f"Processing {task}")

# Efficient stack (LIFO)
stack = deque()
stack.append("item1")
stack.append("item2")
print(stack.pop())
print(stack.pop())
```

## Intermediate Examples

### defaultdict for Nested Structures

```python
# Nested defaultdict
nested = lambda: defaultdict(nested)
data = nested()
data['users']['alice']['email'] = 'alice@example.com'
data['users']['bob']['email'] = 'bob@example.com'
data['users']['alice']['age'] = 30
print(json.dumps(data, indent=2))

# Tree structure
Tree = lambda: defaultdict(Tree)
tree = Tree()
tree['root']['child1']['leaf'] = 1
tree['root']['child2']['leaf'] = 2
print(dict(tree))
```

### Counter with Custom Objects

```python
from math import floor

class ShoppingCart:
    def __init__(self) -> None:
        self.items: Counter = Counter()

    def add(self, item: str, count: int = 1) -> None:
        self.items[item] += count

    def remove(self, item: str, count: int = 1) -> None:
        self.items[item] -= count
        if self.items[item] <= 0:
            del self.items[item]

    def total_items(self) -> int:
        return sum(self.items.values())

    def most_popular(self, n: int = 3) -> list[tuple[str, int]]:
        return self.items.most_common(n)

cart = ShoppingCart()
cart.add('apple', 5)
cart.add('banana', 3)
cart.add('orange', 8)
cart.add('apple', 2)
cart.remove('banana', 1)
print(cart.most_popular())
print(cart.total_items())
```

### ChainMap for Layered Configuration

```python
# Configuration with override layers
defaults = {
    'host': 'localhost',
    'port': 8080,
    'debug': False,
    'database': 'sqlite',
}

user_settings = {
    'port': 9090,
    'debug': True,
}

environment_vars = {
    'host': 'prod.example.com',
}

# Priority: env vars > user settings > defaults
config = ChainMap(environment_vars, user_settings, defaults)
print(config['host'])
print(config['port'])
print(config['debug'])
print(config['database'])

# New child (context-specific)
request_config = config.new_child({'debug': False})
print(request_config['debug'])
print(request_config['host'])
```

### deque for Sliding Window

```python
def sliding_max(values: list[int], k: int) -> list[int]:
    """Return max for each sliding window of size k."""
    dq = deque()
    result = []

    for i, v in enumerate(values):
        # Remove indices outside window
        while dq and dq[0] <= i - k:
            dq.popleft()

        # Remove smaller values from back
        while dq and values[dq[-1]] <= v:
            dq.pop()

        dq.append(i)

        # Start recording once we have a full window
        if i >= k - 1:
            result.append(values[dq[0]])

    return result

print(sliding_max([1, 3, -1, -3, 5, 3, 6, 7], 3))
```

### OrderedDict for LRU Cache

```python
class LRUCache:
    def __init__(self, capacity: int) -> None:
        self.capacity = capacity
        self.cache: OrderedDict[str, Any] = OrderedDict()

    def get(self, key: str) -> Optional[Any]:
        if key not in self.cache:
            return None
        self.cache.move_to_end(key)
        return self.cache[key]

    def put(self, key: str, value: Any) -> None:
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        if len(self.cache) > self.capacity:
            self.cache.popitem(last=False)

    def __repr__(self) -> str:
        return f"LRUCache({dict(self.cache)})"

cache = LRUCache(3)
cache.put('a', 1)
cache.put('b', 2)
cache.put('c', 3)
cache.get('a')       # Move 'a' to end
cache.put('d', 4)    # Evicts 'b' (least recently used)
print(cache)
```

## Advanced Examples

### Custom UserDict for Validated Dictionary

```python
class ValidatedDict(UserDict):
    """Dictionary that validates all values match expected types."""

    def __init__(self, expected_type: type, *args: Any, **kwargs: Any) -> None:
        self.expected_type = expected_type
        super().__init__(*args, **kwargs)

    def __setitem__(self, key: Any, value: Any) -> None:
        if not isinstance(value, self.expected_type):
            raise TypeError(
                f"Value for '{key}' must be {self.expected_type.__name__}, "
                f"got {type(value).__name__}"
            )
        super().__setitem__(key, value)

    def update(self, *args: Any, **kwargs: Any) -> None:
        for k, v in dict(*args, **kwargs).items():
            self[k] = v

vd = ValidatedDict(int)
vd['a'] = 1
vd['b'] = 2
# vd['c'] = "hello"  # Raises TypeError
print(vd)
```

### Custom UserList for Constrained List

```python
class BoundedList(UserList):
    """List with a maximum size."""

    def __init__(self, max_size: int, initlist: Optional[list[Any]] = None) -> None:
        self.max_size = max_size
        super().__init__(initlist if initlist else [])

    def append(self, item: Any) -> None:
        if len(self.data) >= self.max_size:
            raise RuntimeError(f"List full (max {self.max_size} items)")
        super().append(item)

    def extend(self, other: list[Any]) -> None:
        if len(self.data) + len(other) > self.max_size:
            raise RuntimeError(f"Cannot extend: would exceed max size {self.max_size}")
        super().extend(other)

    def insert(self, i: int, item: Any) -> None:
        if len(self.data) >= self.max_size:
            raise RuntimeError(f"List full (max {self.max_size} items)")
        super().insert(i, item)

bl = BoundedList(3)
bl.append(1)
bl.append(2)
bl.append(3)
# bl.append(4)  # Raises RuntimeError
print(bl)
```

### Custom UserString for Wrapped String

```python
class PrefixString(UserString):
    """String that always has a prefix."""

    def __init__(self, seq: str, prefix: str = "[PREFIX]") -> None:
        super().__init__(seq)
        self.prefix = prefix

    def __str__(self) -> str:
        return f"{self.prefix}{self.data}"

    def __repr__(self) -> str:
        return f"PrefixString('{self.data}', prefix='{self.prefix}')"

    def upper(self) -> str:
        return f"{self.prefix}{self.data.upper()}"

    def split(self, sep: Optional[str] = None, maxsplit: int = -1) -> list[str]:
        return super().split(sep, maxsplit)[1:]  # Skip prefix

ps = PrefixString("hello world")
print(ps)
print(ps.upper())
print(ps.split())
```

### ChainMap for Template Rendering

```python
def render_template(template: str, **context: Any) -> str:
    """Simple template rendering with ChainMap scoping."""
    builtins = {
        'upper': str.upper,
        'lower': str.lower,
        'len': len,
        'range': range,
    }

    global_context = {
        'site_name': 'MySite',
        'year': 2024,
    }

    # Scoped lookup: local > global > builtins
    scope = ChainMap(context, global_context, builtins)

    # Simple template substitution
    for key, value in scope.items():
        placeholder = f"{{{{ {key} }}}}"
        if callable(value):
            continue
        template = template.replace(placeholder, str(value))

    return template

result = render_template(
    "Welcome to {{ site_name }}, {{ name }}! ({{ year }})",
    name="Alice"
)
print(result)
```

### Counter for Anomaly Detection

```python
import random

class AnomalyDetector:
    def __init__(self, threshold: float = 2.0) -> None:
        self.data: Counter = Counter()
        self.total = 0
        self.threshold = threshold

    def record(self, value: str) -> None:
        self.data[value] += 1
        self.total += 1

    def is_anomaly(self, value: str) -> bool:
        if self.total == 0:
            return False
        freq = self.data[value] / self.total
        expected = 1.0 / len(self.data) if self.data else 0
        if expected == 0:
            return False
        return freq < expected / self.threshold

    def frequencies(self) -> dict[str, float]:
        return {k: v / self.total for k, v in self.data.items()}

detector = AnomalyDetector(threshold=3.0)
events = ['login', 'logout', 'view', 'login', 'login', 'edit', 'login']
for e in events:
    detector.record(e)

print(detector.is_anomaly('delete'))  # Likely True (unseen)
print(detector.frequencies())
```

### deque for Thread-Safe Task Queue

```python
import threading
import time

class TaskQueue:
    def __init__(self) -> None:
        self._queue: deque = deque()
        self._lock = threading.Lock()

    def add_task(self, task: Any) -> None:
        with self._lock:
            self._queue.append(task)

    def get_task(self) -> Optional[Any]:
        with self._lock:
            return self._queue.popleft() if self._queue else None

    def size(self) -> int:
        with self._lock:
            return len(self._queue)

# def worker(queue: TaskQueue):
#     while True:
#         task = queue.get_task()
#         if task is None:
#             break
#         print(f"Processing: {task}")
#         time.sleep(0.1)

# q = TaskQueue()
# for i in range(10):
#     q.add_task(f"Task-{i}")
# t = threading.Thread(target=worker, args=(q,), daemon=True)
# t.start()
# time.sleep(0.5)
```

### OrderedDict for Tracking Insertion Order in Legacy Code

```python
# In Python 3.7+, regular dicts maintain insertion order, but OrderedDict
# provides extra methods like move_to_end and popitem

class ExpiringCache:
    """Cache that removes oldest items when full."""
    def __init__(self, maxsize: int = 100) -> None:
        self._cache: OrderedDict[str, Any] = OrderedDict()
        self._maxsize = maxsize

    def set(self, key: str, value: Any) -> None:
        if key in self._cache:
            self._cache.move_to_end(key)
        self._cache[key] = value
        while len(self._cache) > self._maxsize:
            self._cache.popitem(last=False)

    def get(self, key: str) -> Optional[Any]:
        if key in self._cache:
            self._cache.move_to_end(key)
            return self._cache[key]
        return None

    def __repr__(self) -> str:
        return f"ExpiringCache({dict(self._cache)})"

ec = ExpiringCache(3)
ec.set('a', 1)
ec.set('b', 2)
ec.set('c', 3)
ec.get('a')
ec.set('d', 4)  # Removes oldest (b)
print(ec)
```

## Real-World Use Cases

- **namedtuple**: Lightweight data objects for CSV rows, database records, coordinate systems.
- **defaultdict**: Grouping items, building nested structures, counting without key checks.
- **Counter**: Word frequency analysis, inventory counting, finding duplicates.
- **deque**: Sliding window algorithms, task queues, undo/redo history, breadth-first search.
- **ChainMap**: Layered configuration (defaults + user settings + env vars), namespace management.
- **OrderedDict**: LRU caches, ordered serialization, maintaining insertion order in older Python versions.
- **UserDict/UserList/UserString**: Creating custom dictionary/list/string subclasses with validation.

## Common Mistakes

- `namedtuple._asdict()` returns an `OrderedDict` in older Python, plain `dict` in newer — be cautious.
- Forgetting that `defaultdict` default factory applies to any missing key access, not just `__getitem__`.
- Using `Counter` subtraction can produce zero/negative counts (use `-` operator or `subtract()` carefully).
- Assuming `deque` is thread-safe without a lock (operations are not atomic across multiple operations).
- Passing mutable default factories (like `list`) without `lambda` or callable.
- Forgetting that `OrderedDict.move_to_end()` requires the key to exist.
- Overusing `ChainMap` for simple dictionary merging (use `{**d1, **d2}` for simple cases).

## Best Practices

- Use `namedtuple` for immutable data containers; use `dataclass` when you need mutability.
- Use `defaultdict` for grouping and counting to avoid explicit key checks.
- Use `Counter` for all frequency/multi-set operations (supports arithmetic).
- Use `deque` for queues, stacks, and sliding windows (O(1) at both ends).
- Use `ChainMap` for layered/priority-based lookups, not for simply merging dicts.
- Use `OrderedDict` when you need `move_to_end()` or `popitem(last=...)` methods.
- Subclass `UserDict`/`UserList`/`UserString` instead of `dict`/`list`/`str` for custom behavior.
- Use `Counter.most_common()` for ranked frequency results.

## Interview Questions

1. What is the difference between `defaultdict` and regular `dict`?
2. How does `Counter` work and what operations does it support?
3. When would you use `deque` instead of a list?
4. What is `namedtuple` and how is it different from a regular tuple?
5. How does `ChainMap` handle key lookups?
6. What is `OrderedDict` and when do you need it (given Python 3.7+ dicts are ordered)?
7. What is the difference between `UserDict` and subclassing `dict` directly?
8. How do you create a nested `defaultdict`?
9. How does `deque.rotate()` work?
10. Implement an LRU cache using `OrderedDict`.

## Coding Challenges

1. **Word Frequency**: Use `Counter` to find the top 10 most common words in a text.
2. **LRU Cache**: Implement a Least Recently Used cache with `OrderedDict`.
3. **Group By**: Use `defaultdict` to group a list of dictionaries by a key.
4. **Sliding Average**: Use `deque` to compute a running average over a sliding window.
5. **Configuration Manager**: Use `ChainMap` to implement layered configuration (defaults, file, env, CLI).
6. **CSV Data Reader**: Use `namedtuple` to read and access CSV data.
7. **Bounded History**: Use `deque(maxlen=N)` to implement a command history.
8. **Inventory Counter**: Use `Counter` to track inventory changes (additions, removals, adjustments).

## Summary

The `collections` module provides specialized, high-performance container datatypes that solve common programming problems. `namedtuple` creates immutable data objects; `defaultdict` simplifies missing-key handling; `Counter` provides powerful counting and multi-set operations; `deque` offers O(1) operations on both ends; `ChainMap` enables layered lookups; and `OrderedDict` maintains insertion order with extra methods. Mastering these tools leads to cleaner, more efficient Python code.

## Related Topics

- Dictionary and set internals
- List and tuple basics
- Hash tables
- Queue data structures
- LRU caching
- Frequency analysis
- Configuration management patterns