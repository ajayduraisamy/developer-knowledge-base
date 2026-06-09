# Dictionaries - dict() constructor, key-value pairs, methods

## Introduction

Dictionaries (dicts) are unordered (insertion-ordered as of Python 3.7), mutable collections of key-value pairs. Each key must be unique and hashable (immutable like strings, numbers, tuples), while values can be any type, including other dicts, lists, or custom objects. Dicts are optimized for O(1) average-time key lookups and are one of Python's most versatile and heavily used data structures. They underpin JSON handling, configuration management, caching, and virtually every domain in Python programming.

## dict() constructor

### What It Is

The `dict()` constructor creates dictionaries from keyword arguments, iterables of key-value pairs, mappings, or other dict objects. It provides a flexible, uniform interface for dictionary creation across multiple data sources.

### Why It Is Important

The constructor enables creating dicts from diverse formats — lists of tuples, zipped sequences, existing mappings, and keyword arguments — without manual looping. This is essential for data parsing, transformation pipelines, and API response handling.

### How It Works Internally

When `dict()` is called with keyword arguments, the CPython `PyDict_Update()` function inserts each key-value pair sequentially. With an iterable of pairs, `PyDict_MergeFromSeq2()` iterates the input and inserts each 2-element sequence. For a mapping argument, `PyDict_Update()` copies entries. Under the hood, Python uses a sparse table with `PyDictEntry` structs, where hash collisions are resolved via open addressing and quadratic probing. The table resizes when the load factor exceeds 2/3.

### Syntax

```python
# Empty dict
d = dict()

# From keyword arguments
d = dict(name="Alice", age=30)

# From iterable of pairs
d = dict([("a", 1), ("b", 2)])
d = dict(zip(["x", "y"], [10, 20]))

# From mapping (copy)
original = {"a": 1}
copy = dict(original)

# From comprehension
d = {x: x**2 for x in range(5)}
```

### Beginner Examples

```python
# Keyword arguments
person = dict(name="Bob", age=25, city="London")
print(person)  # {'name': 'Bob', 'age': 25, 'city': 'London'}

# List of tuples
pairs = [("apple", 1), ("banana", 2), ("cherry", 3)]
fruit_counts = dict(pairs)
print(fruit_counts)

# Zip two lists
keys = ["name", "age", "city"]
values = ["Alice", 30, "NYC"]
combined = dict(zip(keys, values))
print(combined)

# Empty dict
empty = dict()
print(len(empty))  # 0
```

### Intermediate Examples

```python
# Dict comprehension vs dict()
squares = {n: n**2 for n in range(5)}
squares2 = dict((n, n**2) for n in range(5))
print(squares == squares2)  # True

# From list of tuples with transformation
data = [("alice", 30), ("bob", 25)]
d = {name.upper(): age for name, age in data}
print(d)  # {'ALICE': 30, 'BOB': 25}

# Merging with dict()
base = {"a": 1}
update = dict({"b": 2}, **base)  # Merged
print(update)

# Default factory pattern
def make_config(**kwargs):
    defaults = dict(host="localhost", port=8080)
    defaults.update(kwargs)
    return defaults

print(make_config(port=9090))
```

### Advanced Examples

```python
# Deep copy with dict() (shallow only)
original = {"items": [1, 2, 3]}
shallow = dict(original)
original["items"].append(4)
print(shallow["items"])  # [1, 2, 3, 4] -- shared!

# Custom key transformation
def normalize_keys(d, fn=str.lower):
    return dict((fn(k), v) for k, v in d.items())

raw = {"Name": "Alice", "Age": 30}
normalized = normalize_keys(raw)
print(normalized)  # {'name': 'Alice', 'age': 30}

# type() based dict creation
Record = dict.fromkeys(["id", "name", "email"], None)
print(Record)  # {'id': None, 'name': None, 'email': None}

# Ordered dict from constructor
from collections import OrderedDict
od = OrderedDict([("z", 1), ("a", 2), ("m", 3)])
print(list(od.keys()))  # ['z', 'a', 'm']
```

### Real-World Use Cases

- **JSON deserialization**: `json.loads()` returns dicts, often passed to `dict()` for normalization
- **Configuration merging**: Combining default config with user overrides
- **Columnar data conversion**: Converting CSV rows (list of lists) into list of dicts
- **API parameter construction**: Building query parameters from keyword args

### Common Mistakes

```python
# Mistake 1: dict() with non-2-element sequences
# dict([(1, 2, 3)])  # ValueError: dictionary update sequence element

# Mistake 2: Using mutable default arguments with dict()
def bad(d=dict()):  # Single dict shared across calls!
    d["key"] = "value"
    return d

print(bad())  # {'key': 'value'}
print(bad())  # {'key': 'value'} -- wait, same dict!

# Fix:
def good(d=None):
    d = dict() if d is None else d
    d["key"] = "value"
    return d

# Mistake 3: Confusing dict() with {}
# Both create dicts, but dict() is more explicit for conversions
```

### Best Practices

- Use `dict()` when converting from other formats (zip, list of pairs)
- Use `{}` literals for static dicts
- Use `dict.fromkeys()` for initializing with default values
- Avoid mutable default arguments; use `dict()` inside function body

### Performance Considerations

`dict()` with keyword arguments is slightly slower than `{}` literal (the bytecode `LOAD_CONST` + `BUILD_MAP` vs function call overhead). For conversion from pairs, `dict()` is the most efficient approach. `dict()` allocates a minimum-size table and resizes as needed during insertion.

### Interview Questions

1. What arguments does `dict()` accept?
2. How does `dict()` handle duplicate keys in the input?
3. What is the difference between `dict()` and `{}`?
4. How do you create a dict from two lists?
5. What happens with `dict.fromkeys(["a", "b"], [])`?
6. Does `dict()` deep-copy when given a mapping?

### Coding Challenges

```python
# Challenge 1: CSV rows to dicts
def csv_to_dicts(headers, rows):
    return [dict(zip(headers, row)) for row in rows]

headers = ["name", "age"]
data = [["Alice", 30], ["Bob", 25]]
print(csv_to_dicts(headers, data))

# Challenge 2: Nested dict() construction
def deep_dict(keys, value):
    if len(keys) == 1:
        return {keys[0]: value}
    return {keys[0]: deep_dict(keys[1:], value)}

print(deep_dict(["a", "b", "c"], 42))  # {'a': {'b': {'c': 42}}}

# Challenge 3: Dict from object attributes
class User:
    def __init__(self, name, age):
        self.name = name
        self.age = age

u = User("Alice", 30)
d = dict((k, v) for k, v in vars(u).items())
print(d)
```

### Related Topics

- collections.OrderedDict
- dict comprehensions
- zip() function
- JSON serialization
- Mutable vs immutable types

## Key-value pairs

### What It Is

A dictionary in Python is an associative array mapping unique, hashable keys to arbitrary values. Each key-value pair is an entry. Keys must be immutable and hashable (strings, numbers, tuples of immutables, frozensets). Values can be any Python object — including lists, dicts, functions, or class instances.

### Why It Is Important

Key-value pairs model real-world relationships: mapping IDs to records, words to frequencies, configuration names to settings, and URLs to handlers. This abstraction is fundamental to databases, caching, web frameworks, and data processing.

### How It Works Internally

CPython stores key-value pairs in a `PyDictObject` using a hash table with open addressing. The table is an array of `PyDictKeyEntry` structs (hash, key, value). Python 3.6+ uses a compact dict implementation: two arrays — an indices array (sparse) and an entries array (dense, insertion-ordered). This preserves insertion order and reduces memory usage. Lookup computes `hash(key)`, indexes into the sparse array, and follows the probe sequence until finding the matching key or an empty slot.

### Syntax

```python
# Creating with key-value pairs
d = {"name": "Alice", "age": 30}

# Mixed key types
d = {1: "int", "s": "str", (1, 2): "tuple"}

# Accessing
value = d["name"]      # KeyError if missing
value = d.get("name")  # None if missing

# Adding/updating
d["new_key"] = "new_value"
d.update({"k": "v"})

# Deleting
del d["name"]
popped = d.pop("age")
```

### Beginner Examples

```python
# Phonebook
phonebook = {
    "Alice": "555-1234",
    "Bob": "555-5678",
}
print(phonebook["Alice"])

# Check key existence
if "Bob" in phonebook:
    print("Found Bob")

# Safe access
number = phonebook.get("Charlie", "Not found")
print(number)

# Iteration
for name, number in phonebook.items():
    print(f"{name}: {number}")

# Adding entries
phonebook["Charlie"] = "555-9012"
```

### Intermediate Examples

```python
# Nested key-value pairs
users = {
    "alice": {"age": 30, "email": "alice@x.com"},
    "bob": {"age": 25, "email": "bob@x.com"},
}

# Chained access
email = users.get("alice", {}).get("email")
print(email)

# Keys with special requirements
cache = {}
import hashlib
def make_key(*args):
    return hashlib.md5(str(args).encode()).hexdigest()

cache[make_key("user", 1)] = "Alice"
print(cache)

# Inverting key-value pairs
original = {"a": 1, "b": 2, "c": 3}
inverted = {v: k for k, v in original.items()}
print(inverted)  # {1: 'a', 2: 'b', 3: 'c'}
```

### Advanced Examples

```python
# Dict as primary data structure for graph
graph = {
    "A": {"B": 5, "C": 3},
    "B": {"A": 5, "D": 2},
    "C": {"A": 3, "D": 1},
}

def shortest_path(graph, start, end):
    distances = {node: float("inf") for node in graph}
    distances[start] = 0
    unvisited = set(graph.keys())
    while unvisited:
        current = min(unvisited, key=lambda n: distances[n])
        unvisited.remove(current)
        for neighbor, weight in graph[current].items():
            distances[neighbor] = min(distances[neighbor], distances[current] + weight)
    return distances[end]

print(shortest_path(graph, "A", "D"))  # 4

# Memoization with key-value pairs
def fib(n, memo={}):
    if n in memo:
        return memo[n]
    if n < 2:
        return n
    memo[n] = fib(n-1, memo) + fib(n-2, memo)
    return memo[n]

print(fib(100))  # 354224848179261915075

# Factory pattern with dict
class Operation:
    _registry = {}

    @classmethod
    def register(cls, name):
        def wrapper(op_cls):
            cls._registry[name] = op_cls
            return op_cls
        return wrapper

    @classmethod
    def create(cls, name, *args):
        return cls._registry[name](*args)

@Operation.register("add")
class Add:
    def __init__(self, a, b):
        self.result = a + b

op = Operation.create("add", 5, 3)
print(op.result)  # 8
```

### Real-World Use Cases

- **Database ORM**: Mapping column names to values for each row
- **HTTP headers**: Key-value pairs for request/response metadata
- **Caching**: URL → cached response, computation key → result
- **Configuration**: Nested key-value pairs in YAML/JSON configs
- **Counters**: Word → frequency, category → total
- **State machines**: State → handler mapping

### Common Mistakes

```python
# Mistake 1: Unhashable key
# d = {[1, 2]: "value"}  # TypeError
d = {(1, 2): "value"}  # Correct

# Mistake 2: Modifying dict during iteration
d = {"a": 1, "b": 2}
# for k in d:
#     if k == "a":
#         del d[k]  # RuntimeError
for k in list(d):
    if k == "a":
        del d[k]
print(d)  # {'b': 2}

# Mistake 3: Dict copy is shallow
original = {"data": [1, 2, 3]}
copy = original.copy()
original["data"].append(4)
print(copy["data"])  # [1, 2, 3, 4]

# Mistake 4: KeyError on missing key
# d["missing"]  # KeyError
d.get("missing", "default")  # Safe

# Mistake 5: Duplicate keys (last wins)
d = {"a": 1, "a": 2}
print(d)  # {'a': 2}
```

### Best Practices

- Always use `.get()` for access when key may be missing
- Use `.setdefault()` for initialization patterns
- Use `defaultdict` for auto-initialization
- Use `key in dict` for membership (not `.keys()`)
- Prefer `dict.items()` for iteration
- Use immutable keys (strings, ints, tuples)
- Avoid modifying dict during iteration — iterate over `list(dict.items())`

### Performance Considerations

Key lookup is O(1) average, O(n) worst-case (hash collisions). Insertion is amortized O(1). Memory overhead is significant (the compact dict is ~72 bytes + 8 bytes per entry overhead). For millions of keys, consider specialized structures (shelve, dbm, Redis). `dict.items()` returns a view (no copy), while `list(dict.items())` creates a full copy.

### Interview Questions

1. What types can be dictionary keys?
2. How does Python handle hash collisions in dicts?
3. What is the time complexity of key lookup?
4. How do insertion-order dicts work (Python 3.7+)?
5. What is the difference between `.get()` and `.setdefault()`?
6. How do you safely remove items while iterating?
7. Why are dict views (`.keys()`, `.values()`, `.items()`) dynamic?

### Coding Challenges

```python
# Challenge 1: Deep key existence check
def key_exists(d, keys):
    current = d
    for key in keys:
        if isinstance(current, dict) and key in current:
            current = current[key]
        else:
            return False
    return True

config = {"db": {"host": "localhost", "port": 5432}}
print(key_exists(config, ["db", "host"]))   # True
print(key_exists(config, ["db", "user"]))   # False

# Challenge 2: Group by key function
def group_by(items, key_fn):
    result = {}
    for item in items:
        k = key_fn(item)
        result.setdefault(k, []).append(item)
    return result

data = [1, 2, 3, 4, 5, 6]
grouped = group_by(data, lambda x: "even" if x % 2 == 0 else "odd")
print(grouped)

# Challenge 3: Key path setter
def set_path(d, keys, value):
    for key in keys[:-1]:
        d = d.setdefault(key, {})
    d[keys[-1]] = value

d = {}
set_path(d, ["a", "b", "c"], 42)
print(d)  # {'a': {'b': {'c': 42}}}
```

### Related Topics

- Hashable types
- Hash tables
- collections.defaultdict
- collections.Counter
- JSON serialization

## Dictionary methods

### What It Is

Python dicts expose a rich set of built-in methods for access, modification, iteration, and copy operations. These include `.get()`, `.setdefault()`, `.update()`, `.pop()`, `.popitem()`, `.clear()`, `.copy()`, `.keys()`, `.values()`, `.items()`, and `.fromkeys()`.

### Why It Is Important

These methods provide a complete API for dict manipulation — safe access, bulk updates, complex initialization, and view-based iteration. Mastering them is essential for writing idiomatic, bug-free Python.

### How It Works Internally

View objects (`.keys()`, `.values()`, `.items()`) are dynamic — they reflect changes to the underlying dict because they hold a reference to the dict's internal table and re-compute on iteration. Modifying methods like `.update()` and `.pop()` directly manipulate the hash table entries. `.setdefault()` is atomic — it performs a lookup and inserts only if the key is missing, all within the GIL.

### Syntax

```python
d = {"a": 1, "b": 2, "c": 3}

# Access
d.get("a")                    # 1
d.get("z", "default")         # 'default'
d.setdefault("d", 4)          # Sets d["d"] = 4 if missing

# Modification
d.update({"e": 5, "f": 6})   # Bulk update
d.pop("a")                    # Removes and returns 1
d.popitem()                   # Removes and returns last inserted (key, value)
d.clear()                     # Removes all items

# Iteration
d.keys()                      # dict_keys(['b', 'c', ...])
d.values()                    # dict_values([2, 3, ...])
d.items()                     # dict_items([('b', 2), ...])

# Copy
d.copy()                      # Shallow copy
dict(d)                       # Same as copy

# Class method
dict.fromkeys(["a", "b"], 0)  # {'a': 0, 'b': 0}
```

### Beginner Examples

```python
d = {"apple": 5, "banana": 3}

# get() with defaults
print(d.get("apple"))            # 5
print(d.get("cherry"))           # None
print(d.get("cherry", 0))        # 0

# update() to merge
d.update({"cherry": 7, "banana": 10})
print(d)  # {'apple': 5, 'banana': 10, 'cherry': 7}

# pop() to remove
count = d.pop("apple")
print(count, d)  # 5 {'banana': 10, 'cherry': 7}

# keys(), values(), items()
print(list(d.keys()))    # ['banana', 'cherry']
print(list(d.values()))  # [10, 7]

# clear()
d.clear()
print(d)  # {}
```

### Intermediate Examples

```python
# setdefault() initialization
text = "apple banana apple cherry banana apple"
word_positions = {}
for i, word in enumerate(text.split()):
    word_positions.setdefault(word, []).append(i)
print(word_positions)
# {'apple': [0, 2, 5], 'banana': [1, 4], 'cherry': [3]}

# pop() with default
d = {"a": 1}
print(d.pop("b", "N/A"))  # N/A
print(d.pop("a"))         # 1

# popitem() for FIFO (Python 3.7+)
d = {"x": 10, "y": 20}
key, value = d.popitem()
print(key, value)  # y 20 (LIFO)

# fromkeys() with shared mutable
# BAD: all values reference same list
bad = dict.fromkeys(["a", "b", "c"], [])
bad["a"].append(1)
print(bad)  # {'a': [1], 'b': [1], 'c': [1]}

# GOOD: comprehension creates independent lists
good = {k: [] for k in ["a", "b", "c"]}
good["a"].append(1)
print(good)  # {'a': [1], 'b': [], 'c': []}
```

### Advanced Examples

```python
# Chaining methods
class ChainDict(dict):
    def get_or_set(self, key, default):
        return self.setdefault(key, default())

cd = ChainDict()
val = cd.get_or_set("count", lambda: 0)
print(val)  # 0

# Custom fromkeys with factory
def fromkeys_factory(keys, factory):
    return {k: factory() for k in keys}

d = fromkeys_factory(["a", "b"], list)
d["a"].append(1)
print(d)

# View object dynamism
d = {"a": 1}
keys = d.keys()
d["b"] = 2
print(list(keys))  # ['a', 'b'] -- dynamic!

# Update with callable
class UpdateDict(dict):
    def update_with(self, other, fn=lambda old, new: new):
        for k, v in other.items():
            self[k] = fn(self.get(k), v)

ud = UpdateDict({"a": 1, "b": 2})
ud.update_with({"a": 10, "c": 3}, lambda old, new: old + new if old else new)
print(ud)  # {'a': 11, 'b': 2, 'c': 3}
```

### Real-World Use Cases

- **Configuration**: `.get()` with defaults for optional settings
- **Caching**: `.setdefault()` for populating cache entries atomically
- **Merging configs**: `.update()` for layering defaults → user config → env overrides
- **Counting**: `.get(key, 0) + 1` pattern for frequency counters
- **Form data**: `.pop()` to extract keys from parsed forms
- **API responses**: `.copy()` before modifying cached response dicts

### Common Mistakes

```python
# Mistake 1: Using .keys() unnecessarily
if "key" in d.keys():  # Works but wasteful
    pass
if "key" in d:  # Better, direct hash lookup

# Mistake 2: .setdefault() evaluates default eagerly
d = {}
d.setdefault("key", [])  # Creates [] even if key exists
# Use default factory pattern for expensive defaults

# Mistake 3: .copy() is shallow
d = {"data": {"nested": 1}}
c = d.copy()
c["data"]["nested"] = 99
print(d["data"]["nested"])  # 99 -- shared!

# Mistake 4: Popitem() order assumption
# Python 3.7+ LIFO, but don't rely on it for critical logic

# Mistake 5: Forgetting .pop() raises KeyError without default
d = {"a": 1}
# d.pop("b")  # KeyError
d.pop("b", None)  # Safe
```

### Best Practices

- Use `.get()` over direct indexing when key may be missing
- Use `.setdefault()` for single-step initialization
- Use `.update()` for bulk merges
- Use `.pop(key, None)` for safe removal
- Use `.copy()` for shallow copies; `copy.deepcopy()` for deep
- Prefer `key in d` over `key in d.keys()`
- Use `dict.fromkeys()` with caution (shared mutable trap)

### Performance Considerations

`.get()` is O(1) — identical cost to direct indexing when key exists, slightly more when missing (None creation). `.setdefault()` is O(1) — lookup + optional insert. `.update()` is O(k) for k keys. `.keys()`, `.values()`, `.items()` return views (O(1), no copy). `list(.keys())` creates a full copy (O(n)). `.copy()` is O(n) — it copies all key-value pair references.

### Interview Questions

1. What is the difference between `.get()` and `.setdefault()`?
2. How are dict view objects different from lists?
3. What happens with `dict.fromkeys(["a"], [])` when you modify the list?
4. Why is `key in d` better than `key in d.keys()`?
5. How does `.popitem()` differ from `.pop()`?
6. Is `.copy()` deep or shallow?
7. When should you use `.update()` vs the `|` operator?

### Coding Challenges

```python
# Challenge 1: Deep update
def deep_update(base, overlay):
    for key, value in overlay.items():
        if key in base and isinstance(base[key], dict) and isinstance(value, dict):
            deep_update(base[key], value)
        else:
            base[key] = value
    return base

d1 = {"a": 1, "b": {"c": 2}}
d2 = {"b": {"d": 3}, "e": 4}
merged = deep_update(d1, d2)
print(merged)

# Challenge 2: Counter class
class Counter(dict):
    def __init__(self, iterable=None):
        super().__init__()
        if iterable:
            for item in iterable:
                self[item] = self.get(item, 0) + 1

    def most_common(self, n=None):
        items = sorted(self.items(), key=lambda x: x[1], reverse=True)
        return items[:n]

c = Counter("mississippi")
print(c.most_common(3))  # [('i', 4), ('s', 4), ('p', 2)]

# Challenge 3: Dot-accessible dict
class DotDict(dict):
    def __getattr__(self, key):
        try:
            return self[key]
        except KeyError:
            raise AttributeError(key) from None
    def __setattr__(self, key, value):
        self[key] = value
    def __delattr__(self, key):
        try:
            del self[key]
        except KeyError:
            raise AttributeError(key) from None

d = DotDict({"name": "Alice", "age": 30})
print(d.name)  # Alice
d.city = "NYC"
print(d)  # {'name': 'Alice', 'age': 30, 'city': 'NYC'}
```

### Related Topics

- collections.defaultdict
- collections.OrderedDict
- collections.Counter
- Shallow vs deep copy
- View objects
- collections.abc.Mapping
