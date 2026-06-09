# Magic Methods - __str__, __repr__, __add__, __getitem__, __call__

## Introduction

Magic methods (also called dunder methods, short for "double underscore") are special methods in Python that begin and end with double underscores. They allow user-defined classes to integrate seamlessly with Python's built-in functions, operators, and protocols. By implementing magic methods, your objects can behave like built-in types — supporting iteration, indexing, arithmetic, string representation, context management, and more.

## __str__ and __repr__

### What It Is

`__str__` defines the "informal" string representation of an object (used by `print()` and `str()`). `__repr__` defines the "official" representation (used by `repr()` and the interactive interpreter), ideally one that can recreate the object.

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __str__(self):
        return f"({self.x}, {self.y})"

    def __repr__(self):
        return f"Point({self.x}, {self.y})"
```

### Why It Is Important

Proper string representations make objects display meaningfully in logs, debug output, and user interfaces. `__repr__` is especially important for debugging — it should be unambiguous. The rule: `__repr__` is for developers, `__str__` is for users.

### How It Works Internally

When `print(obj)` is called, Python checks for `__str__`. If not found, it falls back to `__repr__`. When `repr(obj)` is called, it looks for `__repr__`. The interactive interpreter displays `repr(obj)` for expressions.

```python
p = Point(3, 4)
print(p)          # __str__ → (3, 4)
str(p)            # __str__ → (3, 4)
repr(p)           # __repr__ → Point(3, 4)
f"{p!r}"          # __repr__ → Point(3, 4)  (!r forces repr)
f"{p}"            # __str__ → (3, 4)
```

### Syntax

```python
class MyClass:
    def __str__(self):
        """User-friendly string."""
        return "informal"

    def __repr__(self):
        """Unambiguous, often eval()-able string."""
        return "MyClass()"
```

### Beginner Examples

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def __str__(self):
        return f"{self.name} ({self.age})"

    def __repr__(self):
        return f"Person('{self.name}', {self.age})"

p = Person("Alice", 30)
print(p)           # Alice (30)
print(repr(p))     # Person('Alice', 30)
```

### Intermediate Examples

```python
from datetime import datetime

class Task:
    def __init__(self, title, priority=0, due_date=None):
        self.title = title
        self.priority = priority
        self.due_date = due_date or datetime.now()
        self.completed = False

    def __str__(self):
        status = "✓" if self.completed else "○"
        return f"{status} {self.title} (priority: {self.priority})"

    def __repr__(self):
        return (
            f"Task(title={self.title!r}, "
            f"priority={self.priority}, "
            f"due_date={self.due_date!r})"
        )

t = Task("Write report", priority=2)
print(t)        # ○ Write report (priority: 2)
print(repr(t))  # Task(title='Write report', priority=2, due_date=datetime.datetime(...))
```

### Advanced Examples

```python
import json
from typing import List, Optional

class Node:
    def __init__(self, value: int, left: Optional["Node"] = None, right: Optional["Node"] = None):
        self.value = value
        self.left = left
        self.right = right

    def __repr__(self):
        parts = [f"Node({self.value}"]
        if self.left or self.right:
            left_repr = repr(self.left) if self.left else "None"
            right_repr = repr(self.right) if self.right else "None"
            parts.append(f", left={left_repr}")
            parts.append(f", right={right_repr}")
        parts.append(")")
        return "".join(parts)

    def __str__(self):
        # Level-order traversal
        if not self:
            return "Empty"
        result = []
        queue = [self]
        while queue:
            node = queue.pop(0)
            result.append(str(node.value))
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)
        return "[" + ", ".join(result) + "]"

# Build tree
root = Node(1,
    Node(2, Node(4), Node(5)),
    Node(3, None, Node(6))
)
print(str(root))   # [1, 2, 3, 4, 5, 6]
print(repr(root))  # Full nested representation
```

### Real-World Use Cases

- **Django models**: `__str__` shows human-readable model instances in admin
- **API responses**: `__repr__` for debugging serialized objects
- **Collections**: containers display their contents via `__repr__`
- **Error messages**: custom exceptions with informative `__str__`

### Common Mistakes

1. `__repr__` doesn't return a string (must always return `str`)
2. `__repr__` that isn't informative enough (should include all significant fields)
3. `__str__` that raises exceptions (always handle edge cases)
4. Recursive `__str__` for objects with circular references

### Best Practices

- Always implement `__repr__`; `__str__` is optional (falls back to `__repr__`)
- `__repr__` should be unambiguous, ideally eval-able
- `__str__` should be readable for end users
- Use `!r` in f-strings when you need repr output
- For collections, show length and type

## __add__ and arithmetic

### What It Is

`__add__` defines behavior for the `+` operator. Python provides a full suite of arithmetic magic methods for `+`, `-`, `*`, `/`, `//`, `%`, `**`, `@`, and their in-place (`+=`) and reverse variations.

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)
```

### Why It Is Important

Arithmetic magic methods enable custom types to use intuitive math syntax. This is essential for mathematical objects (vectors, matrices, complex numbers) and domain-specific types (money, quantities, dates).

### How It Works Internally

For `a + b`, Python calls `a.__add__(b)`. If it returns `NotImplemented`, Python tries `b.__radd__(a)`. If both fail, `TypeError` is raised. In-place operators like `+=` call `__iadd__` if available, falling back to `__add__`.

```python
result = a + b
# Step 1: a.__add__(b)
# Step 2: if NotImplemented → b.__radd__(a)
# Step 3: if both fail → TypeError
```

### Syntax

```python
class MyNumber:
    def __init__(self, value):
        self.value = value

    def __add__(self, other):       # self + other
        ...
    def __radd__(self, other):      # other + self
        ...
    def __iadd__(self, other):      # self += other
        ...
    def __sub__(self, other):       # self - other
        ...
    def __mul__(self, other):       # self * other
        ...
    def __truediv__(self, other):   # self / other
        ...
```

### Beginner Examples

```python
class Dollar:
    def __init__(self, amount):
        self.amount = amount

    def __add__(self, other):
        if isinstance(other, Dollar):
            return Dollar(self.amount + other.amount)
        return NotImplemented

    def __str__(self):
        return f"${self.amount:.2f}"

    def __repr__(self):
        return f"Dollar({self.amount})"

d1 = Dollar(10)
d2 = Dollar(20)
print(d1 + d2)  # $30.00
```

### Intermediate Examples

```python
class Distance:
    def __init__(self, meters=0, kilometers=0):
        self._meters = meters + kilometers * 1000

    @property
    def meters(self):
        return self._meters

    @property
    def kilometers(self):
        return self._meters / 1000

    def __add__(self, other):
        if isinstance(other, Distance):
            return Distance(meters=self._meters + other._meters)
        return NotImplemented

    def __sub__(self, other):
        if isinstance(other, Distance):
            return Distance(meters=max(0, self._meters - other._meters))
        return NotImplemented

    def __mul__(self, scalar):
        if isinstance(scalar, (int, float)):
            return Distance(meters=self._meters * scalar)
        return NotImplemented

    def __rmul__(self, scalar):
        return self.__mul__(scalar)

    def __eq__(self, other):
        if isinstance(other, Distance):
            return abs(self._meters - other._meters) < 0.001
        return NotImplemented

    def __lt__(self, other):
        if isinstance(other, Distance):
            return self._meters < other._meters
        return NotImplemented

    def __str__(self):
        if self._meters >= 1000:
            return f"{self.kilometers:.2f} km"
        return f"{self._meters:.1f} m"

d1 = Distance(kilometers=1.5)
d2 = Distance(meters=500)
print(d1 + d2)       # 2.00 km
print(d1 - d2)       # 1.00 km
print(d1 * 3)        # 4.50 km
print(3 * d1)        # 4.50 km
print(d1 > d2)       # True
```

### Advanced Examples

```python
class Polynomial:
    def __init__(self, *coeffs):
        # coeffs[0] + coeffs[1]*x + coeffs[2]*x^2 + ...
        self.coeffs = list(coeffs)

    def __add__(self, other):
        if isinstance(other, Polynomial):
            max_len = max(len(self.coeffs), len(other.coeffs))
            a = self.coeffs + [0] * (max_len - len(self.coeffs))
            b = other.coeffs + [0] * (max_len - len(other.coeffs))
            return Polynomial(*[x + y for x, y in zip(a, b)])
        if isinstance(other, (int, float)):
            return Polynomial(self.coeffs[0] + other, *self.coeffs[1:])
        return NotImplemented

    def __radd__(self, other):
        return self.__add__(other)

    def __sub__(self, other):
        if isinstance(other, Polynomial):
            max_len = max(len(self.coeffs), len(other.coeffs))
            a = self.coeffs + [0] * (max_len - len(self.coeffs))
            b = other.coeffs + [0] * (max_len - len(other.coeffs))
            return Polynomial(*[x - y for x, y in zip(a, b)])
        return NotImplemented

    def __mul__(self, other):
        if isinstance(other, Polynomial):
            result = [0] * (len(self.coeffs) + len(other.coeffs) - 1)
            for i, a in enumerate(self.coeffs):
                for j, b in enumerate(other.coeffs):
                    result[i + j] += a * b
            return Polynomial(*result)
        if isinstance(other, (int, float)):
            return Polynomial(*[c * other for c in self.coeffs])
        return NotImplemented

    def __call__(self, x):
        result = 0
        for power, coeff in enumerate(self.coeffs):
            result += coeff * (x ** power)
        return result

    def __str__(self):
        terms = []
        for power, coeff in enumerate(self.coeffs):
            if coeff == 0:
                continue
            if power == 0:
                terms.append(f"{coeff}")
            elif power == 1:
                terms.append(f"{coeff}x")
            else:
                terms.append(f"{coeff}x^{power}")
        return " + ".join(terms) if terms else "0"

    def __repr__(self):
        return f"Polynomial({', '.join(str(c) for c in self.coeffs)})"

p1 = Polynomial(1, 2, 3)   # 1 + 2x + 3x^2
p2 = Polynomial(0, 1, 1)   # 0 + 1x + x^2
print(f"p1 = {p1}")
print(f"p2 = {p2}")
print(f"p1 + p2 = {p1 + p2}")
print(f"p1 * p2 = {p1 * p2}")
print(f"p1(2) = {p1(2)}")  # 1 + 4 + 12 = 17
```

### Real-World Use Cases

- **Unit conversion**: `Distance(km=1) + Distance(m=500)`
- **Money arithmetic**: `Price(10, "USD") + Price(5, "EUR")` with auto-conversion
- **Vector graphics**: `Point(1,2) + Point(3,4)` for translations
- **Date/time**: `datetime + timedelta`
- **Database query building**: `Query("users") & Query(age=21)` for compound queries

### Common Mistakes

1. Not returning `NotImplemented` for incompatible types
2. Forgetting `__radd__` — `5 + obj` breaks without it
3. Mutating `self` instead of returning a new instance
4. Not implementing `__iadd__` for `+=` (falls back to `__add__`, but may be slower)

### Best Practices

- Return `NotImplemented`, don't raise `TypeError` for unsupported types
- Always return a new instance from arithmetic methods
- Implement `__radd__` whenever `__add__` is implemented
- Implement `__iadd__` for mutable objects to optimize `+=`
- Keep operations type-consistent (return same class)

## __getitem__ and indexing

### What It Is

`__getitem__` enables indexing on custom objects using `obj[key]` syntax. It also enables iteration and the `in` operator. `__setitem__` enables assignment `obj[key] = value`, and `__delitem__` enables `del obj[key]`.

```python
class MyList:
    def __init__(self, items):
        self._items = items

    def __getitem__(self, index):
        return self._items[index]
```

### Why It Is Important

Indexing is how Python accesses elements in sequences and mappings. Implementing `__getitem__` makes custom collections behave like built-in lists, tuples, and dicts. It's also the gateway to iteration — any class with `__getitem__` is iterable.

### How It Works Internally

`obj[key]` calls `type(obj).__getitem__(obj, key)`. For slices, Python passes a `slice(start, stop, step)` object. For iteration, Python calls `__getitem__` with increasing integer indices starting from 0 until `IndexError` is raised.

```python
class SimpleSequence:
    def __getitem__(self, idx):
        if 0 <= idx < 5:
            return f"item_{idx}"
        raise IndexError

for item in SimpleSequence():  # Works! Calls __getitem__(0), (1), ...
    print(item)
```

### Syntax

```python
class Container:
    def __getitem__(self, key):     # obj[key]
        ...

    def __setitem__(self, key, value):  # obj[key] = value
        ...

    def __delitem__(self, key):     # del obj[key]
        ...

    def __len__(self):             # len(obj)
        ...
```

### Beginner Examples

```python
class Playlist:
    def __init__(self):
        self._songs = []

    def add(self, song):
        self._songs.append(song)

    def __getitem__(self, index):
        return self._songs[index]

    def __setitem__(self, index, song):
        self._songs[index] = song

    def __len__(self):
        return len(self._songs)

pl = Playlist()
pl.add("Song A")
pl.add("Song B")
pl.add("Song C")
print(pl[0])       # Song A
print(pl[-1])      # Song C
pl[1] = "Song D"
print(len(pl))     # 3
```

### Intermediate Examples

```python
class SparseMatrix:
    def __init__(self, rows, cols):
        self.rows = rows
        self.cols = cols
        self._data = {}

    def __getitem__(self, key):
        if isinstance(key, tuple) and len(key) == 2:
            row, col = key
            if 0 <= row < self.rows and 0 <= col < self.cols:
                return self._data.get((row, col), 0)
            raise IndexError("Index out of bounds")
        raise TypeError("Expected (row, col) tuple")

    def __setitem__(self, key, value):
        if isinstance(key, tuple) and len(key) == 2:
            row, col = key
            if 0 <= row < self.rows and 0 <= col < self.cols:
                if value != 0:
                    self._data[(row, col)] = value
                else:
                    self._data.pop((row, col), None)
                return
            raise IndexError("Index out of bounds")
        raise TypeError("Expected (row, col) tuple")

    def __str__(self):
        lines = []
        for r in range(self.rows):
            row = [str(self[r, c]) for c in range(self.cols)]
            lines.append("[" + ", ".join(row) + "]")
        return "\n".join(lines)

m = SparseMatrix(4, 4)
m[0, 0] = 1
m[1, 1] = 2
m[3, 3] = 3
print(m)
```

### Advanced Examples

```python
class TrieNode:
    __slots__ = ("children", "is_end")

    def __init__(self):
        self.children = {}
        self.is_end = False

class Trie:
    def __init__(self):
        self._root = TrieNode()

    def insert(self, word):
        node = self._root
        for char in word:
            if char not in node.children:
                node.children[char] = TrieNode()
            node = node.children[char]
        node.is_end = True

    def __getitem__(self, prefix):
        """Return all words starting with given prefix."""
        node = self._root
        for char in prefix:
            if char not in node.children:
                return []
            node = node.children[char]
        return self._collect_words(node, prefix)

    def __contains__(self, word):
        node = self._root
        for char in word:
            if char not in node.children:
                return False
            node = node.children[char]
        return node.is_end

    def __len__(self):
        return self._count_words(self._root)

    def _collect_words(self, node, prefix):
        results = []
        if node.is_end:
            results.append(prefix)
        for char, child in sorted(node.children.items()):
            results.extend(self._collect_words(child, prefix + char))
        return results

    def _count_words(self, node):
        count = 1 if node.is_end else 0
        for child in node.children.values():
            count += self._count_words(child)
        return count

t = Trie()
t.insert("apple")
t.insert("app")
t.insert("apricot")
t.insert("banana")
print(t["ap"])          # ['app', 'apple', 'apricot']
print("apple" in t)     # True
print("ap" in t)        # False
print(len(t))           # 4
```

### Real-World Use Cases

- **Custom collections**: `IndexedList`, `UniqueList` with special indexing behavior
- **Database cursors**: `cursor[10:20]` for pagination
- **Pandas DataFrames**: `df["column"]`, `df[0:10]`
- **Configuration objects**: `config["database.host"]` for dot-path access
- **Tree structures**: `tree["path/to/node"]` for path-based access

### Common Mistakes

1. Not handling `slice` objects in `__getitem__`
2. Raising wrong exception type (should raise `IndexError` or `KeyError`)
3. Not implementing `__setitem__` for mutable collections
4. Inconsistent behavior between `__getitem__` and `__len__`

### Best Practices

- Support slicing via `isinstance(key, slice)` checks
- Raise `KeyError` for dict-like access, `IndexError` for sequence-like access
- Implement `__contains__` for the `in` operator (faster than iterating)
- For multi-dimensional indexing, accept tuple keys
- Document whether your container is sequence-like or mapping-like

### Performance Considerations

- `__getitem__` method call overhead per index operation
- For hot loops, pre-fetch items into local variables
- `__contains__` should be O(1) for hash-based collections
- Slice access creates a new slice object each time

## __call__

### What It Is

`__call__` allows an instance of a class to be called like a function. Objects that implement `__call__` are called "callables."

```python
class Greeter:
    def __call__(self, name):
        return f"Hello, {name}!"

greet = Greeter()
print(greet("World"))  # Hello, World!
```

### Why It Is Important

`__call__` enables objects to behave like functions while maintaining state. This is useful for decorators, function factories, and objects that represent operations. Callable objects can have configuration and state that plain functions cannot.

### How It Works Internally

`obj(args)` calls `type(obj).__call__(obj, args)`. The check is: does the class have `__call__` in its `__dict__` (or MRO)? If so, the instance is callable. `callable(obj)` returns `True` for objects with `__call__`.

```python
def is_callable(obj):
    return hasattr(type(obj), "__call__")
```

### Syntax

```python
class Callable:
    def __call__(self, *args, **kwargs):
        return result

obj = Callable()
result = obj(args)  # Calls __call__
```

### Beginner Examples

```python
class Counter:
    def __init__(self, start=0):
        self.count = start

    def __call__(self):
        self.count += 1
        return self.count

counter = Counter(10)
print(counter())  # 11
print(counter())  # 12
print(counter())  # 13
```

### Intermediate Examples

```python
import time

class TimedFunction:
    def __init__(self, func):
        self.func = func
        self.total_time = 0

    def __call__(self, *args, **kwargs):
        start = time.perf_counter()
        result = self.func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        self.total_time += elapsed
        print(f"{self.func.__name__} took {elapsed:.6f}s")
        return result

@TimedFunction
def slow_function(n):
    total = 0
    for i in range(n):
        total += i ** 2
    return total

result = slow_function(10_000_000)
print(f"Result: {result}")
print(f"Total time: {slow_function.total_time:.4f}s")
```

### Advanced Examples

```python
import re
from typing import Dict, Pattern

class RegexValidator:
    def __init__(self):
        self._cache: Dict[str, Pattern] = {}

    def __call__(self, pattern: str, text: str) -> bool:
        if pattern not in self._cache:
            self._cache[pattern] = re.compile(pattern)
        return bool(self._cache[pattern].search(text))

    def invalidate(self, pattern: str = None):
        if pattern:
            self._cache.pop(pattern, None)
        else:
            self._cache.clear()

    @property
    def cache_size(self) -> int:
        return len(self._cache)


class Partial:
    """Partial function application via callable object."""
    def __init__(self, func, *args, **kwargs):
        self.func = func
        self.args = args
        self.kwargs = kwargs

    def __call__(self, *args, **kwargs):
        merged_kwargs = {**self.kwargs, **kwargs}
        return self.func(*self.args, *args, **merged_kwargs)

    def __repr__(self):
        return f"Partial({self.func.__name__}, {self.args}, {self.kwargs})"


# Callable as a state machine
class StateMachine:
    def __init__(self):
        self._state = "idle"
        self._transitions = {
            "idle": {"start": "running"},
            "running": {"pause": "paused", "stop": "idle"},
            "paused": {"resume": "running", "stop": "idle"},
        }

    def __call__(self, action: str) -> str:
        if action in self._transitions.get(self._state, {}):
            self._state = self._transitions[self._state][action]
            return f"Transitioned to {self._state}"
        raise ValueError(f"Cannot {action} from {self._state}")

    def __str__(self):
        return f"StateMachine({self._state})"


validator = RegexValidator()
print(validator(r"\d+", "abc123"))       # True
print(validator(r"^[a-z]+$", "Hello"))   # False
print(f"Cached patterns: {validator.cache_size}")

add = lambda x, y: x + y
add5 = Partial(add, 5)
print(add5(10))  # 15

sm = StateMachine()
print(sm("start"))   # Transitioned to running
print(sm("pause"))   # Transitioned to paused
print(sm("resume"))  # Transitioned to running
```

### Real-World Use Cases

- **Decorators**: `@LoggedFunction` as a callable decorator class
- **Function factories**: `Multiplier(factor)` returns callable that multiplies by factor
- **GUI callbacks**: button click handlers as callable objects with state
- **Middleware**: callable objects that wrap request handling
- **Partial function application**: binding arguments to functions

### Common Mistakes

1. Forgetting `self` parameter in `__call__` definition
2. Making objects callable unnecessarily (use a plain function when stateless)
3. Mutable default arguments in `__call__` (same issue as regular functions)
4. Side effects in `__call__` that surprise users

### Best Practices

- Use `__call__` when your object represents an operation with state
- Prefer plain functions for stateless operations
- Document what arguments `__call__` expects
- Make callable objects idempotent where possible
- Consider `functools.partial` for simple argument binding

### Performance Considerations

- Calling a callable object has the same overhead as calling a method
- `__call__` is slightly slower than a plain function call (attribute lookup)
- For hot loops, pre-bind the callable to a local variable
- `callable()` check is fast (just checks `tp_call` slot)

### Interview Questions

1. What is the difference between `__str__` and `__repr__`? When would you implement each?
2. How does Python dispatch `a + b`? What is the role of `NotImplemented`?
3. How do magic methods enable iteration without explicitly implementing `__iter__`?
4. What is a callable object and when would you use one instead of a function?
5. How would you implement a class that supports `obj[1:5:2]`?

### Coding Challenges

1. Create a `Range` class that supports indexing, iteration, and slicing like `range()`.
2. Implement a `Fraction` class with full arithmetic operator support.
3. Build a `MemoizedFunction` callable that caches results of expensive computations.

### Related Topics

Properties, Descriptors, Iterators, Generators, Context Managers
