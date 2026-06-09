# Dictionaries - dict() constructor, key-value pairs, methods

## Introduction

Dictionaries (dicts) are unordered, mutable collections of key-value pairs. Each key must be unique and hashable (immutable like strings, numbers, tuples), while values can be any type. Dicts are optimized for fast lookups by key and are one of Python's most powerful and frequently used data structures.

## Why It Is Important

Dictionaries are essential because:
- They provide O(1) average-time key lookups
- They model real-world relationships (mappings)
- They are flexible and dynamic
- They support dict comprehensions for concise creation
- They form the backbone of JSON data handling
- They are used extensively in web frameworks, APIs, and data processing

## Syntax

```python
# Creating dictionaries
empty = {}
person = {"name": "Alice", "age": 30}

# Using dict() constructor
items = dict(name="Bob", age=25)

# From list of pairs
pairs = dict([("a", 1), ("b", 2)])

# Dict comprehension
squares = {x: x**2 for x in range(5)}

# Accessing values
name = person["name"]  # Raises KeyError if missing
name = person.get("name")  # Returns None if missing
name = person.get("name", "default")  # With default

# Modifying
person["age"] = 31
person["city"] = "New York"
```

## Examples

### Creating Dictionaries

```python
# Different creation methods
empty = {}
print(f"Empty dict: {empty}")

person = {
    "name": "Alice",
    "age": 30,
    "city": "New York",
    "is_student": False
}
print(f"Person: {person}")

# Using dict() with keyword arguments
config = dict(debug=True, port=8080, host="localhost")
print(f"Config: {config}")

# From list of tuples
pairs = [("apple", 1), ("banana", 2), ("cherry", 3)]
fruit_counts = dict(pairs)
print(f"Fruit counts: {fruit_counts}")

# Using zip with keys and values
keys = ["name", "age", "city"]
values = ["Bob", 25, "London"]
combined = dict(zip(keys, values))
print(f"Combined: {combined}")

# Mixed key types
mixed = {
    "string": "value",
    42: "integer key",
    (1, 2): "tuple key",
    True: "boolean key (same as 1!)"
}
print(f"Mixed keys: {mixed}")
```

### Accessing and Modifying

```python
person = {"name": "Alice", "age": 30, "city": "NYC"}

# Accessing values
print(f"Name: {person['name']}")
print(f"Age: {person['age']}")

# Safe access with get()
print(f"Country: {person.get('country')}")  # None
print(f"Country: {person.get('country', 'USA')}")  # USA

# Modifying values
person["age"] = 31
print(f"Updated: {person}")

# Adding new keys
person["email"] = "alice@example.com"
person["phone"] = "555-0123"
print(f"After adding: {person}")

# Updating with another dict
person.update({"city": "Boston", "country": "USA"})
print(f"After update: {person}")

# Deleting keys
del person["phone"]
print(f"After del: {person}")

# Pop (remove and return)
email = person.pop("email")
print(f"Popped email: {email}")
print(f"After pop: {person}")

# Clear all items
# person.clear()  # Would empty the dict
```

## Beginner Examples

```python
# Dictionary as a phonebook
phonebook = {
    "Alice": "555-1234",
    "Bob": "555-5678",
    "Charlie": "555-9012"
}

print("Phonebook:")
for name, number in phonebook.items():
    print(f"  {name}: {number}")

# Looking up a number
name = input("Enter name to look up: ")
number = phonebook.get(name)
if number:
    print(f"{name}'s number is {number}")
else:
    print(f"Contact {name} not found")

# Checking key existence
if "Bob" in phonebook:
    print("Bob is in the phonebook")

# Counting word frequencies
text = "the quick brown fox jumps over the lazy dog the quick"
words = text.split()
word_count = {}
for word in words:
    word_count[word] = word_count.get(word, 0) + 1
print(f"Word counts: {word_count}")

# Simple inventory
inventory = {"apples": 10, "bananas": 5, "oranges": 8}
inventory["apples"] -= 2
inventory["grapes"] = 12
print(f"Inventory: {inventory}")
print(f"Total items: {sum(inventory.values())}")
```

## Intermediate Examples

```python
# Dictionary methods overview
d = {"a": 1, "b": 2, "c": 3}

print(f"Keys: {d.keys()}")
print(f"Values: {d.values()}")
print(f"Items: {d.items()}")

# Converting to lists
print(f"Keys list: {list(d.keys())}")
print(f"Values list: {list(d.values())}")

# Iterating over dictionaries
for key in d:  # Iterates over keys
    print(f"  {key}: {d[key]}")

for key, value in d.items():
    print(f"  {key}: {value}")

# Setdefault
d = {}
d.setdefault("count", 0)
d["count"] += 1
print(f"Count: {d['count']}")

# Practical setdefault example
text = "apple banana apple cherry banana apple"
words = text.split()
word_positions = {}
for i, word in enumerate(words):
    word_positions.setdefault(word, []).append(i)
print(f"Word positions: {word_positions}")

# Defaultdict
from collections import defaultdict

# defaultdict with int
word_counts = defaultdict(int)
for word in "apple banana apple".split():
    word_counts[word] += 1
print(f"Defaultdict counts: {dict(word_counts)}")

# defaultdict with list
groups = defaultdict(list)
for i in range(10):
    groups[i % 2].append(i)
print(f"Groups: {dict(groups)}")

# OrderedDict (remembers insertion order)
from collections import OrderedDict
ordered = OrderedDict()
ordered["first"] = 1
ordered["second"] = 2
ordered["third"] = 3
for key, value in ordered.items():
    print(f"  {key}: {value}")

# Merging dictionaries (Python 3.9+)
dict1 = {"a": 1, "b": 2}
dict2 = {"c": 3, "d": 4}
merged = dict1 | dict2
print(f"Merged: {merged}")
```

## Advanced Examples

```python
# Dict comprehensions
numbers = range(10)
squares = {n: n**2 for n in numbers}
print(f"Squares: {squares}")

# Conditional comprehension
even_squares = {n: n**2 for n in numbers if n % 2 == 0}
print(f"Even squares: {even_squares}")

# Inverting a dictionary (swap keys and values)
original = {"a": 1, "b": 2, "c": 3}
inverted = {v: k for k, v in original.items()}
print(f"Inverted: {inverted}")

# Using dictionary for memoization (caching)
def memoized_fibonacci(n, cache=None):
    if cache is None:
        cache = {}
    if n in cache:
        return cache[n]
    if n <= 1:
        return n
    result = memoized_fibonacci(n-1, cache) + memoized_fibonacci(n-2, cache)
    cache[n] = result
    return result

print(f"Fibonacci(10): {memoized_fibonacci(10)}")
print(f"Fibonacci(50): {memoized_fibonacci(50)}")

# Nested dictionaries
users = {
    "alice": {
        "age": 30,
        "email": "alice@example.com",
        "roles": ["admin", "user"]
    },
    "bob": {
        "age": 25,
        "email": "bob@example.com",
        "roles": ["user"]
    }
}

for username, info in users.items():
    print(f"{username}: {info['age']} years, roles: {', '.join(info['roles'])}")

# Chaining get() for nested access
bob_role = users.get("bob", {}).get("roles", [])
print(f"Bob's roles: {bob_role}")

# Counter for frequency analysis
from collections import Counter
letters = Counter("mississippi")
print(f"Letter frequencies: {dict(letters)}")
print(f"Most common: {letters.most_common(3)}")

# Dictionary as switch/case
def get_operation(op):
    operations = {
        "add": lambda a, b: a + b,
        "subtract": lambda a, b: a - b,
        "multiply": lambda a, b: a * b,
        "divide": lambda a, b: a / b if b != 0 else "Error"
    }
    return operations.get(op, lambda a, b: "Unknown operation")(10, 5)

print(f"Add: {get_operation('add')}")
print(f"Divide: {get_operation('divide')}")

# Deep merging dictionaries
from collections.abc import Mapping

def deep_merge(d1, d2):
    result = dict(d1)
    for key, value in d2.items():
        if key in result and isinstance(result[key], Mapping) and isinstance(value, Mapping):
            result[key] = deep_merge(result[key], value)
        else:
            result[key] = value
    return result

config1 = {"database": {"host": "localhost", "port": 5432}}
config2 = {"database": {"port": 5433, "name": "mydb"}, "debug": True}
merged_config = deep_merge(config1, config2)
print(f"Merged config: {merged_config}")
```

## Real-World Use Cases

- **Configuration Management**: Application settings, environment variables
- **API Responses**: Parsing JSON responses from web APIs
- **Caching**: Memoization, request caches, session storage
- **Database Operations**: Row-to-dict mapping for queries
- **Counting/Frequency Analysis**: Word counts, analytics
- **Lookup Tables**: Mapping codes to descriptions, IDs to records
- **Graph Representation**: Adjacency lists for graph algorithms

## Common Mistakes

```python
# Mistake 1: Accessing missing key without get()
d = {"a": 1}
# print(d["b"])  # KeyError
# Correct:
print(d.get("b", "not found"))

# Mistake 2: Using mutable keys
# d = {[1, 2]: "value"}  # TypeError: unhashable type: 'list'
# Use tuple instead:
d = {(1, 2): "value"}

# Mistake 3: Modifying dict while iterating
d = {"a": 1, "b": 2, "c": 3}
# Wrong:
# for key in d:
#     if key == "b":
#         del d[key]  # RuntimeError: dictionary changed size during iteration
# Correct:
for key in list(d.keys()):
    if key == "b":
        del d[key]
print(f"After safe deletion: {d}")

# Mistake 4: Confusing dict views with lists
d = {"a": 1, "b": 2}
keys = d.keys()
# keys.append("c")  # AttributeError: 'dict_keys' object has no attribute 'append'
# Convert to list first:
keys_list = list(d.keys())

# Mistake 5: Forgetting that dicts maintain insertion order (Python 3.7+)
# This is guaranteed now, but older code may assume unordered

# Mistake 6: Using = to copy dicts (creates reference)
original = {"a": [1, 2]}
shallow = original  # Reference!
shallow["b"] = 3
print(original)  # Also modified!

# Use .copy() or dict() for shallow copy
copy = original.copy()
copy["c"] = 4
print(original)  # Unchanged
```

## Best Practices

- Use `dict.get()` for safe access with defaults
- Use `collections.defaultdict` for automatic default values
- Use `collections.Counter` for counting tasks
- Use `dict.setdefault()` for initialization patterns
- Use `dict.update()` for merging multiple updates
- Use `|` operator (Python 3.9+) for merging dicts
- Use dict comprehensions for concise transformations
- Avoid modifying dicts during iteration
- Use `isinstance(value, Mapping)` for abstract dict types
- Prefer `key in dict` over `key in dict.keys()`
- Use `for key, value in dict.items()` instead of iterating keys

## Interview Questions

1. How do dictionaries work internally in Python?
2. What is the time complexity of dictionary operations?
3. What types can be used as dictionary keys?
4. What is the difference between `dict.get()` and `dict[key]`?
5. How do defaultdict and Counter work?
6. What is a dict comprehension?
7. How do you merge two dictionaries?
8. What is the difference between OrderedDict and regular dict?
9. How do you invert a dictionary?
10. What are dictionary view objects?

## Coding Challenges

```python
# Challenge 1: Word frequency counter
def word_frequency(text):
    words = text.lower().split()
    freq = {}
    for word in words:
        word = word.strip(".,!?;:'\"")
        freq[word] = freq.get(word, 0) + 1
    return freq

text = "The quick brown fox jumps over the lazy dog. The quick!"
print(f"Word frequency: {word_frequency(text)}")

# Challenge 2: Group by key
def group_by(items, key_func):
    result = {}
    for item in items:
        key = key_func(item)
        result.setdefault(key, []).append(item)
    return result

people = [
    {"name": "Alice", "age": 25},
    {"name": "Bob", "age": 30},
    {"name": "Charlie", "age": 25},
    {"name": "Diana", "age": 30}
]
by_age = group_by(people, lambda p: p["age"])
for age, group in by_age.items():
    names = [p["name"] for p in group]
    print(f"Age {age}: {names}")

# Challenge 3: Dictionary to object converter
class DictToObject:
    def __init__(self, d):
        for key, value in d.items():
            if isinstance(value, dict):
                setattr(self, key, DictToObject(value))
            else:
                setattr(self, key, value)

config = {"database": {"host": "localhost", "port": 5432}, "debug": True}
obj = DictToObject(config)
print(f"Host: {obj.database.host}")  # localhost

# Challenge 4: Find keys with maximum values
def max_value_keys(d):
    if not d:
        return []
    max_val = max(d.values())
    return [k for k, v in d.items() if v == max_val]

scores = {"Alice": 85, "Bob": 92, "Charlie": 92, "Diana": 78}
print(f"Top scorers: {max_value_keys(scores)}")  # ['Bob', 'Charlie']

# Challenge 5: Deep flatten a nested dictionary
def flatten_dict(d, parent_key="", sep="."):
    items = []
    for key, value in d.items():
        new_key = f"{parent_key}{sep}{key}" if parent_key else key
        if isinstance(value, dict):
            items.extend(flatten_dict(value, new_key, sep).items())
        else:
            items.append((new_key, value))
    return dict(items)

nested = {"a": 1, "b": {"c": 2, "d": {"e": 3}}}
flat = flatten_dict(nested)
print(f"Flattened: {flat}")  # {'a': 1, 'b.c': 2, 'b.d.e': 3}
```

## Summary

Dictionaries are key-value mappings with O(1) average lookups, mutable and dynamic. They support creation via literals, comprehensions, and constructors. Methods like `get()`, `update()`, `setdefault()`, and `pop()` provide flexible access and modification. Specialized variants like `defaultdict`, `Counter`, and `OrderedDict` extend dict functionality. Dicts are fundamental to Python programming and appear in virtually every non-trivial application.

## Related Topics

- Sets (related key concepts)
- JSON and Serialization
- defaultdict and Counter
- Dict Comprehensions
- Hashable Types
- Collections Module
- Mutability and Hashing
