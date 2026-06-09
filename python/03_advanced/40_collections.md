# Collections Module - namedtuple, Counter, deque, defaultdict

## Introduction

The `collections` module provides specialized container datatypes that extend Python's built-in containers (dict, list, set, tuple). These high-performance alternatives solve common programming problems with cleaner, more efficient code. The module includes `namedtuple` for self-documenting tuples, `Counter` for counting hashable objects, `deque` for efficient double-ended operations, and `defaultdict` for automatic default values.

## namedtuple

### What It Is

`namedtuple` is a factory function that creates tuple subclasses with named fields. Each field is accessible by name (like an object attribute) and by position (like a regular tuple). Namedtuples are immutable, memory-efficient, and provide helpful `__repr__` and `__doc__` strings automatically.

### Why It Is Important

Namedtuples make code more readable by replacing anonymous tuples with self-documenting data structures. They combine the immutability and memory efficiency of tuples with the readability of objects. They are ideal for simple data containers where you want the benefits of immutability without the overhead of a full class definition.

### How It Works Internally

`namedtuple` uses a template-based approach — it generates a Python class definition as a string and executes it with `exec`. The generated class inherits from `tuple` and implements `__new__`, `__repr__`, `__doc__`, and property descriptors for each named field. The instances are exactly the same size as a regular tuple.

### Syntax

```python
from collections import namedtuple

Point = namedtuple('Point', ['x', 'y'])
# or: namedtuple('Point', 'x y')
# or: namedtuple('Point', 'x, y')

p = Point(10, 20)
p.x     # 10
p[0]    # 10
```

### Beginner Examples

```python
from collections import namedtuple

# Define a namedtuple
Person = namedtuple('Person', ['name', 'age', 'city'])

# Create instances
alice = Person('Alice', 30, 'New York')
bob = Person('Bob', 25, 'San Francisco')

# Access by name
print(alice.name)   # Alice
print(alice.age)    # 30

# Access by index
print(alice[0])     # Alice
print(alice[1])     # 30

# Unpacking
name, age, city = alice
print(f"{name} is {age} from {city}")

# Immutability
# alice.age = 31  # AttributeError: can't set attribute

# Useful methods
print(alice._asdict())  # {'name': 'Alice', 'age': 30, 'city': 'New York'}
print(alice._fields)    # ('name', 'age', 'city')
```

### Intermediate Examples

```python
from collections import namedtuple

# Default values with defaults parameter (Python 3.7+)
Employee = namedtuple('Employee', ['name', 'dept', 'salary'], defaults=['Unknown', 0])
emp1 = Employee('Alice')  # name='Alice', dept='Unknown', salary=0
emp2 = Employee('Bob', 'Engineering', 75000)

print(emp1)  # Employee(name='Alice', dept='Unknown', salary=0)
print(emp2._field_defaults)  # {'dept': 'Unknown', 'salary': 0}

# Renaming invalid field names
Data = namedtuple('Data', ['name', 'class', 'def'], rename=True)
# class becomes _1, def becomes _2
d = Data('Alice', 'A', 'func')
print(d)  # Data(name='Alice', _1='A', _2='func')

# Making new instances with _replace
emp3 = emp2._replace(salary=80000)
print(emp3.salary)  # 80000 (emp2 remains unchanged)

# Inheriting from namedtuple (advanced)
class Point(namedtuple('Point', ['x', 'y'])):
    __slots__ = ()

    @property
    def distance_from_origin(self):
        return (self.x ** 2 + self.y ** 2) ** 0.5

p = Point(3, 4)
print(p.distance_from_origin)  # 5.0
```

### Advanced Examples

```python
from collections import namedtuple
import csv
from typing import List

# Reading CSV files into namedtuples
def read_csv_as_namedtuples(filename: str):
    with open(filename, 'r') as f:
        reader = csv.reader(f)
        headers = next(reader)
        Row = namedtuple('Row', headers)
        return [Row(*row) for row in reader]

# Using namedtuple for tree structures
class Tree:
    Node = namedtuple('Node', ['value', 'left', 'right'])
    Leaf = namedtuple('Leaf', ['value'])

    @classmethod
    def from_list(cls, lst):
        if isinstance(lst, list):
            if len(lst) == 3:
                return cls.Node(lst[0], cls.from_list(lst[1]), cls.from_list(lst[2]))
            return cls.Node(lst[0], None, None)
        return cls.Leaf(lst) if lst is not None else None

tree = Tree.from_list([1, [2, None, None], [3, [4, None, None], None]])
print(tree)  # Node(value=1, left=Node(value=2, left=None, right=None), ...)

# Converting between dict and namedtuple
def namedtuple_to_dict(nt):
    return nt._asdict()

def dict_to_namedtuple(cls, d):
    return cls(**{k: v for k, v in d.items() if k in cls._fields})

pt = Point(10, 20)
d = namedtuple_to_dict(pt)
pt2 = dict_to_namedtuple(Point, d)
print(pt == pt2)  # True
```

### Real-World Use Cases

- **Reading CSV data**: Represent rows as namedtuples for readable column access.
- **Configuration**: Store immutable configuration values with named fields.
- **Database records**: Lightweight, immutable representations of DB rows.
- **Return values**: Functions that return multiple values as a namedtuple for clarity.
- **Graph nodes**: Lightweight representations with named coordinates.

### Common Mistakes

- Using `namedtuple` when mutability is required (use a `dataclass` instead).
- Forgetting that `namedtuple` field names must be valid Python identifiers.
- Using `namedtuple` with many fields (use `dataclass` for 5+ fields, it's more readable).
- Expecting `namedtuple` to be as fast as a manually written C extension (it's close but not identical).
- Modifying the `_fields` class attribute (it's read-only).

### Best Practices

- Use `namedtuple` for immutable, lightweight data containers with 2-5 fields.
- Use `dataclass` when you need mutability, type hints, or complex initialization.
- Use `_replace` for creating modified copies (functional update pattern).
- Use `_asdict()` for serialization to JSON or other formats.
- Consider subclassing `namedtuple` when adding behavior to the data.

### Performance Considerations

Namedtuples have the same memory footprint as regular tuples — they are the most memory-efficient way to create simple objects in Python. Attribute access (by name) is slightly slower than tuple index access but faster than `__dict__`-based attribute access on regular objects.

### Interview Questions

**Q: What are the advantages of namedtuple over a regular class?**

A: Namedtuples are immutable, memory-efficient (no `__dict__`), provide tuple unpacking, indexing, and iteration automatically, and require less code to define. They also provide `_asdict()`, `_replace()`, and `_fields` out of the box.

**Q: How does namedtuple ensure field names are valid?**

A: If `rename=True` is passed, invalid or duplicate field names are renamed to `_1`, `_2`, etc. Without `rename=True`, invalid names cause a `ValueError`. Reserved keywords like `class`, `def` are also rejected unless renamed.

### Coding Challenges

1. Create a namedtuple for a 3D vector with methods for dot product and cross product.
2. Read a CSV file into a list of namedtuples and compute summary statistics.
3. Implement a simple tree structure using recursive namedtuples.

### Related Topics

- `dataclasses` (mutable, type-hinted alternative)
- Regular tuples (anonymous counterpart)
- `typing.NamedTuple` (type-hinted namedtuple)
- `__slots__` (memory optimization similar to namedtuple)

## Counter

### What It Is

`Counter` is a dictionary subclass designed for counting hashable objects. It stores elements as dictionary keys and their counts as dictionary values. It provides methods for common counting operations like `most_common()`, arithmetic between counters, and element enumeration.

### Why It Is Important

`Counter` eliminates boilerplate code for counting operations. Instead of manually using dictionaries with `if key in counts: counts[key] += 1`, `Counter` provides a clean, efficient, and feature-rich API. It includes operations that would otherwise require multiple lines of code: most common elements, arithmetic on counts, and finding unique/duplicate elements.

### How It Works Internally

`Counter` subclasses `dict` and overrides `__missing__` to return 0 for missing keys (instead of raising `KeyError`). The `update()` method adds counts rather than replacing them. The `most_common()` method uses `heapq.nlargest` for efficient top-N extraction. Arithmetic operations (`+`, `-`, `&`, `|`) use `__add__`, `__sub__`, `__and__`, `__or__` methods.

### Syntax

```python
from collections import Counter

# From iterable
Counter(['a', 'b', 'c', 'a', 'b', 'a'])
# Counter({'a': 3, 'b': 2, 'c': 1})

# From mapping
Counter({'a': 3, 'b': 2})

# From keyword args
Counter(a=3, b=2)
```

### Beginner Examples

```python
from collections import Counter

# Basic counting
words = "the quick brown fox jumps over the lazy dog".split()
word_counts = Counter(words)
print(word_counts)
# Counter({'the': 2, 'quick': 1, 'brown': 1, ...})

# Access counts
print(word_counts['the'])    # 2
print(word_counts['cat'])    # 0 (no KeyError)

# Most common
print(word_counts.most_common(3))
# [('the', 2), ('quick', 1), ('brown', 1)]

# Elements (iterator repeating each element count times)
c = Counter(a=3, b=1)
print(list(c.elements()))  # ['a', 'a', 'a', 'b']
```

### Intermediate Examples

```python
from collections import Counter

# Counter arithmetic
c1 = Counter(a=5, b=3, c=1)
c2 = Counter(a=2, b=1, d=3)

# Addition (adds counts)
print(c1 + c2)  # Counter({'a': 7, 'b': 4, 'd': 3, 'c': 1})

# Subtraction (removes counts, zero or negative are excluded)
print(c1 - c2)  # Counter({'a': 3, 'b': 2})

# Intersection (min)
print(c1 & c2)  # Counter({'a': 2, 'b': 1})

# Union (max)
print(c1 | c2)  # Counter({'a': 5, 'd': 3, 'b': 3, 'c': 1})

# Update (add counts)
c = Counter(a=1)
c.update({'a': 2, 'b': 1})
print(c)  # Counter({'a': 3, 'b': 1})
```

### Advanced Examples

```python
from collections import Counter
import re

# Analyzing text
def word_frequency(text: str, n: int = 10):
    words = re.findall(r'\w+', text.lower())
    return Counter(words).most_common(n)

text = """
Python is an interpreted, high-level and general-purpose programming language.
Python's design philosophy emphasizes code readability with its notable use
of significant indentation. Its language constructs and object-oriented approach
aim to help programmers write clear, logical code for small and large-scale projects.
"""

print(word_frequency(text, 5))
# [('and', 2), ('language', 2), ('code', 2), ...]

# Finding duplicates
def find_duplicates(items):
    return [item for item, count in Counter(items).items() if count > 1]

# Anagrams detection
def are_anagrams(s1: str, s2: str):
    return Counter(s1.replace(" ", "").lower()) == Counter(s2.replace(" ", "").lower())

print(are_anagrams("listen", "silent"))    # True
print(are_anagrams("hello", "world"))     # False

# Multi-set operations
inventory = Counter(apples=5, oranges=3, bananas=2)
order = Counter(oranges=2, bananas=2, apples=1)
remaining = inventory - order
print(remaining)  # Counter({'apples': 4, 'oranges': 1})
```

### Real-World Use Cases

- **Word frequency analysis**: Analyze document word distributions.
- **Inventory management**: Track stock levels and process orders.
- **Poll/Vote counting**: Count votes with `most_common()` for winners.
- **Data quality**: Find duplicate records or missing values.
- **Log analysis**: Count error types or access patterns in log files.
- **Genomics**: Count nucleotide or k-mer frequencies in DNA sequences.

### Common Mistakes

- Using `Counter` when a simple `defaultdict(int)` would suffice (Counter is better for its extra methods).
- Forgetting that `Counter` returns 0 for missing keys (unlike regular dict's `KeyError`).
- Expecting `elements()` to return elements in a specific order (it's arbitrary).
- Using `Counter` with unhashable types (lists, dicts) — they must be hashable.
- Assuming `most_common()` with no argument returns all items in order (it does).

### Best Practices

- Use `Counter.most_common()` for frequency-based analysis.
- Use Counter arithmetic (`+`, `-`, `&`, `|`) for multi-set operations.
- Use `Counter` with `re.findall` for text analysis.
- Use `elements()` to expand counts back into individual items.
- Chain Counters with normal dict operations for maximum flexibility.

### Performance Considerations

`Counter` operations are O(n) for counting, O(n log n) for `most_common()` with no argument, and O(n log k) for `most_common(k)`. Arithmetic operations are O(n) where n is the number of unique elements. Memory usage is proportional to the number of unique elements.

### Interview Questions

**Q: How does `Counter` handle missing keys?**

A: `Counter` overrides `__missing__` to return 0 instead of raising `KeyError`. This allows `c[key]` to return 0 for any key not in the counter, making increment operations natural.

**Q: What is the difference between `Counter` and `defaultdict(int)`?**

A: Both can count items, but `Counter` provides additional methods: `most_common()`, `elements()`, and arithmetic operations (+`, `-`, `&`, `|`) that `defaultdict(int)` doesn't have. `Counter` also has a more convenient constructor.

### Coding Challenges

1. Find the most common words in a large text file using `Counter`.
2. Implement a simple spell checker that suggests corrections based on character frequency.
3. Use `Counter` to implement an anagram solver finding all anagrams from a word list.
4. Build a shopping cart inventory system using `Counter` for stock management.

### Related Topics

- `defaultdict` (general-purpose default dict)
- Dictionaries (Counter is a dict subclass)
- Hashable types (Counter keys must be hashable)
- `heapq` (used by `most_common`)

## deque

### What It Is

`deque` (double-ended queue) is a list-like container optimized for fast appends and pops from both ends. It provides O(1) operations for `append`, `appendleft`, `pop`, `popleft`, while Python lists have O(n) for insertions/removals from the left.

### Why It Is Important

`deque` is the go-to data structure when you need fast operations at both ends of a sequence. It's ideal for queues, stacks, sliding windows, rolling buffers, and breadth-first search. Its performance characteristics make it superior to lists for these specific use cases.

### How It Works Internally

`deque` is implemented as a doubly-linked list of fixed-size blocks (arrays). Each block holds multiple elements. This design provides O(1) append/pop at both ends while maintaining good cache locality. The `maxlen` parameter creates a bounded deque that automatically discards elements from the opposite end when full.

### Syntax

```python
from collections import deque

d = deque([1, 2, 3])     # From iterable
d = deque(maxlen=5)       # Bounded deque
```

### Beginner Examples

```python
from collections import deque

# Basic operations
d = deque([1, 2, 3])
d.append(4)           # deque([1, 2, 3, 4])
d.appendleft(0)       # deque([0, 1, 2, 3, 4])
d.pop()               # 4, deque([0, 1, 2, 3])
d.popleft()           # 0, deque([1, 2, 3])

# Bounded deque (maxlen)
recent = deque(maxlen=3)
for i in range(10):
    recent.append(i)
print(recent)  # deque([7, 8, 9], maxlen=3)

# Rotation
d = deque([1, 2, 3, 4, 5])
d.rotate(2)    # deque([4, 5, 1, 2, 3])
d.rotate(-1)   # deque([5, 1, 2, 3, 4])
```

### Intermediate Examples

```python
from collections import deque

# Queue implementation
class Queue:
    def __init__(self):
        self._items = deque()

    def enqueue(self, item):
        self._items.append(item)

    def dequeue(self):
        return self._items.popleft()

    def is_empty(self):
        return len(self._items) == 0

    def __len__(self):
        return len(self._items)

# Stack implementation
class Stack:
    def __init__(self):
        self._items = deque()

    def push(self, item):
        self._items.append(item)

    def pop(self):
        return self._items.pop()

    def peek(self):
        return self._items[-1]

# Sliding window
def sliding_max(numbers, k):
    d = deque()
    result = []
    for i, n in enumerate(numbers):
        # Remove elements outside window
        while d and d[0] <= i - k:
            d.popleft()
        # Remove smaller elements (they won't be needed)
        while d and numbers[d[-1]] <= n:
            d.pop()
        d.append(i)
        if i >= k - 1:
            result.append(numbers[d[0]])
    return result

print(sliding_max([1, 3, -1, -3, 5, 3, 6, 7], 3))
# [3, 3, 5, 5, 6, 7]
```

### Advanced Examples

```python
from collections import deque

# BFS on a graph using deque
def bfs_shortest_path(graph, start, goal):
    queue = deque([(start, [start])])
    visited = {start}
    while queue:
        node, path = queue.popleft()
        for neighbor in graph[node]:
            if neighbor == goal:
                return path + [neighbor]
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append((neighbor, path + [neighbor]))
    return None

graph = {
    'A': ['B', 'C'],
    'B': ['A', 'D', 'E'],
    'C': ['A', 'F'],
    'D': ['B'],
    'E': ['B', 'F'],
    'F': ['C', 'E']
}
print(bfs_shortest_path(graph, 'A', 'F'))  # ['A', 'C', 'F'] or ['A', 'B', 'E', 'F']

# Round-robin scheduler
class RoundRobinScheduler:
    def __init__(self):
        self.tasks = deque()

    def add_task(self, task_id, burst_time):
        self.tasks.append((task_id, burst_time))

    def run(self, quantum=2):
        while self.tasks:
            task_id, burst = self.tasks.popleft()
            if burst <= quantum:
                print(f"Task {task_id} completed")
            else:
                print(f"Task {task_id} ran for {quantum}, remaining {burst - quantum}")
                self.tasks.append((task_id, burst - quantum))

scheduler = RoundRobinScheduler()
scheduler.add_task(1, 5)
scheduler.add_task(2, 3)
scheduler.add_task(3, 8)
scheduler.run(quantum=2)
```

### Real-World Use Cases

- **Undo/Redo**: Store history with bounded deque for memory-efficient undo/redo.
- **Task queues**: Producer-consumer patterns with deque as the queue.
- **BFS traversal**: Graph algorithms rely on O(1) popleft operations.
- **Sliding window**: Efficient sliding window computations (max, min, average).
- **Round-robin scheduling**: Process tasks in a circular manner.
- **Recent items**: `maxlen` keeps the last N items automatically.

### Common Mistakes

- Using `deque` when random access is needed (lists are O(1) for index access, deques are O(n)).
- Expecting `deque` to be faster than list for all operations (random access and insert in middle are slower).
- Using `deque` as a multi-threaded queue without locks (use `queue.Queue` for thread safety).
- Forgetting that `deque` is not thread-safe for multi-producer/consumer scenarios.

### Best Practices

- Use `deque` for FIFO queues and LIFO stacks that require fast appends/pops from both ends.
- Use `maxlen` for bounded buffers and keeping only the most recent items.
- Use `deque` for BFS and sliding window algorithms.
- Use `queue.Queue` for multi-threaded producer-consumer patterns.
- Avoid random access on deques in performance-critical code.

### Performance Considerations

`deque` appends and pops from either end are O(1). Random access is O(n) in the worst case. For comparison, list appends are O(1) amortized, pop from left is O(n). Memory overhead is slightly higher than list due to the linked-list-of-blocks structure.

### Interview Questions

**Q: When would you use a `deque` instead of a list?**

A: Use `deque` when you need fast appends/pops from both ends (e.g., queue, stack, BFS). Use list when you need fast random access (indexing) or insert/remove in the middle.

**Q: How does `maxlen` affect a deque's behavior?**

A: When `maxlen` is set, the deque has a maximum size. When full, adding an item to one end automatically removes an item from the opposite end. This is useful for keeping a rolling history or sliding window.

### Coding Challenges

1. Implement a palindrome checker using `deque`.
2. Implement a BFS to find the shortest path in a maze.
3. Build a task scheduler using `deque` for round-robin task execution.
4. Implement a sliding window algorithm to find the median of each window.

### Related Topics

- `queue.Queue` (thread-safe queue)
- `list` (alternative for random access)
- `collections.deque` methods
- `itertools.islice` (for slicing deques)

## defaultdict

### What It Is

`defaultdict` is a dictionary subclass that calls a factory function to supply default values for missing keys. When a key is accessed that doesn't exist, `defaultdict` calls the factory (e.g., `list`, `int`, `set`) to create and return a default value instead of raising `KeyError`.

### Why It Is Important

`defaultdict` eliminates boilerplate code for handling missing dictionary keys. Common patterns like grouping items, counting occurrences, and building nested dictionaries become one-liners instead of multi-line `if key in dict` blocks. It makes code more concise, readable, and less error-prone.

### How It Works Internally

`defaultdict` overrides `__missing__` (called by `__getitem__` when a key is not found). Instead of raising `KeyError`, it calls the stored factory function with no arguments, inserts the result under the requested key, and returns it. The factory is stored as the `default_factory` attribute.

### Syntax

```python
from collections import defaultdict

dd = defaultdict(int)        # Default value: 0
dd = defaultdict(list)       # Default value: []
dd = defaultdict(set)        # Default value: set()
dd = defaultdict(lambda: 'N/A')  # Custom default
```

### Beginner Examples

```python
from collections import defaultdict

# Counting with defaultdict (vs Counter for more features)
words = "the quick brown fox jumps over the lazy dog".split()
count = defaultdict(int)
for word in words:
    count[word] += 1
print(dict(count))
# {'the': 2, 'quick': 1, 'brown': 1, ...}

# Grouping with defaultdict(list)
students = [
    ("A", "Alice"), ("B", "Bob"), ("A", "Charlie"),
    ("C", "Diana"), ("B", "Eve")
]
by_grade = defaultdict(list)
for grade, name in students:
    by_grade[grade].append(name)
print(dict(by_grade))
# {'A': ['Alice', 'Charlie'], 'B': ['Bob', 'Eve'], 'C': ['Diana']}
```

### Intermediate Examples

```python
from collections import defaultdict

# Nested defaultdict (autovivification)
tree = lambda: defaultdict(tree)
nested = tree()
nested['users']['alice']['email'] = 'alice@example.com'
nested['users']['bob']['email'] = 'bob@example.com'
nested['settings']['theme'] = 'dark'
print(dict(nested))
# {'users': {'alice': {'email': 'alice@example.com'}, 'bob': {'email': 'bob@example.com'}}, ...}

# defaultdict(set) for unique grouping
tags = [
    ("python", "dynamic"), ("python", "interpreted"),
    ("java", "static"), ("python", "high-level"),
    ("java", "verbose")
]
lang_tags = defaultdict(set)
for lang, tag in tags:
    lang_tags[lang].add(tag)
print(dict(lang_tags))
# {'python': {'dynamic', 'interpreted', 'high-level'}, 'java': {'static', 'verbose'}}

# defaultdict with custom factory
from datetime import datetime
timestamps = defaultdict(datetime.now)
timestamps['login']
timestamps['logout']
```

### Advanced Examples

```python
from collections import defaultdict

# Graph representation using defaultdict(list)
class Graph:
    def __init__(self):
        self.adj = defaultdict(list)

    def add_edge(self, u, v):
        self.adj[u].append(v)
        self.adj[v].append(u)

    def dfs(self, start):
        visited = set()
        result = []
        def _dfs(node):
            visited.add(node)
            result.append(node)
            for neighbor in self.adj[node]:
                if neighbor not in visited:
                    _dfs(neighbor)
        _dfs(start)
        return result

g = Graph()
g.add_edge(1, 2)
g.add_edge(1, 3)
g.add_edge(2, 4)
g.add_edge(3, 5)
print(g.dfs(1))  # [1, 2, 4, 3, 5]

# Multimap implementation
class MultiMap:
    def __init__(self):
        self._map = defaultdict(list)

    def add(self, key, value):
        self._map[key].append(value)

    def get(self, key):
        return self._map.get(key, [])

    def items(self):
        for key, values in self._map.items():
            for value in values:
                yield key, value

mm = MultiMap()
mm.add('fruit', 'apple')
mm.add('fruit', 'banana')
mm.add('color', 'red')
print(list(mm.items()))  # [('fruit', 'apple'), ('fruit', 'banana'), ('color', 'red')]
```

### Real-World Use Cases

- **Grouping records**: Group database rows by a field value.
- **Counting**: Count occurrences of items (use `Counter` for extra features).
- **Tree building**: Build nested dictionaries for hierarchical data.
- **Graph representation**: Adjacency list as `defaultdict(list)`.
- **Inverted index**: Map values to lists of keys in search engines.
- **Caching**: Cache with default factory for lazy computation.

### Common Mistakes

- Accessing a missing key via `d[key]` when you want to test for existence without inserting a default (use `key in d` or `d.get(key)` instead).
- Setting `default_factory` to `None` and expecting defaults (raises `KeyError` like a regular dict).
- Using `defaultdict` when `Counter` would be more appropriate for counting.
- Forgetting that `defaultdict` adds a default value for ANY missing key access via `d[key]`.

### Best Practices

- Use `defaultdict(list)` for grouping items by a key.
- Use `defaultdict(int)` for simple counting (or `Counter` for advanced needs).
- Use `defaultdict(set)` for maintaining unique collections per key.
- Use `d.get(key)` or `key in d` for existence checks without side effects.
- Use `defaultdict` over manual `if key in dict` for cleaner grouping code.

### Performance Considerations

`defaultdict` is slightly faster than equivalent `if key in dict` checks because the default generation logic is in C. Memory usage is the same as a regular dict plus one pointer to the factory function. The factory is called every time a missing key is accessed — ensure it's cheap.

### Interview Questions

**Q: How is `defaultdict` different from `dict.setdefault()`?**

A: `defaultdict` automatically calls the factory for missing keys accessed via `d[key]`. `setdefault()` requires an explicit call and provides the default only for that specific access. `defaultdict` is cleaner for repeated access patterns.

**Q: What happens if `default_factory` is `None`?**

A: If `default_factory` is `None`, accessing a missing key raises `KeyError`, just like a regular dict. You can use `dd.default_factory = list` to change the factory at runtime.

### Coding Challenges

1. Build a tree structure using a recursive `defaultdict` (`tree = lambda: defaultdict(tree)`).
2. Implement a simple inverted index for a search engine using `defaultdict(list)`.
3. Parse a log file and group error messages by error type using `defaultdict`.
4. Build a graph and implement Dijkstra's algorithm using `defaultdict`.

### Related Topics

- `Counter` (specialized defaultdict for counting)
- Regular `dict` (base class)
- `dict.setdefault()` (alternative to defaultdict)
- `dict.get()` (safe access without default insertion)
