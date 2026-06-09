# Magic Methods - __str__, __repr__, __add__, __getitem__, __call__

## Introduction

**Magic methods** (also called **dunder methods** — short for "double underscore") are special methods in Python that begin and end with double underscores, like `__init__` or `__str__`. They are not meant to be called directly by your code; instead, Python calls them automatically in response to specific operations such as `len(obj)`, `str(obj)`, `obj == other`, or `obj[key]` — including operators like `+` and `-`.

Magic methods allow you to make your custom objects behave like built-in types, enabling an expressive and intuitive API.

## Why It Is Important

- **Natural syntax**: Make your objects work with Python's built-in functions and operators.
- **Interoperability**: Custom objects can be used with `len()`, `str()`, `in`, iteration, slicing, and more.
- **Duck typing alignment**: Implement the same protocols as built-in types.
- **Context managers**: `with` statement support via `__enter__` / `__exit__`.
- **Callable objects**: Use objects as functions via `__call__`.
- **Memory efficiency**: `__slots__` reduces memory footprint for many instances.

## Syntax

```python
class MyClass:
    def __str__(self):
        return "User-friendly string"

    def __repr__(self):
        return "MyClass()"

    def __len__(self):
        return 0

    def __add__(self, other):
        return ...

    def __eq__(self, other):
        return ...

    def __getitem__(self, key):
        return ...

    def __call__(self, *args, **kwargs):
        return ...

    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        return False
```

## Examples

### `__str__` and `__repr__`

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"Point({self.x}, {self.y})"

    def __str__(self):
        return f"({self.x}, {self.y})"


p = Point(3, 4)
print(repr(p))   # Point(3, 4)
print(str(p))    # (3, 4)
print(p)         # (3, 4)  (uses __str__)
```

## Beginner Examples

### 1. `__len__` and `__bool__`

```python
class Team:
    def __init__(self, members):
        self.members = members

    def __len__(self):
        return len(self.members)

    def __bool__(self):
        return len(self.members) > 0


team = Team(["Alice", "Bob"])
print(len(team))   # 2
print(bool(team))  # True

empty = Team([])
print(bool(empty))  # False
```

### 2. `__eq__` and `__hash__`

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def __eq__(self, other):
        if not isinstance(other, Person):
            return NotImplemented
        return self.name == other.name and self.age == other.age

    def __hash__(self):
        return hash((self.name, self.age))

    def __repr__(self):
        return f"Person({self.name}, {self.age})"


p1 = Person("Alice", 30)
p2 = Person("Alice", 30)
p3 = Person("Bob", 25)
print(p1 == p2)       # True
print(p1 == p3)       # False
print(hash(p1))       # some integer
print({p1, p2, p3})   # deduplicated: {Person(Bob, 25), Person(Alice, 30)}
```

### 3. `__getitem__` and `__setitem__`

```python
class SimpleDict:
    def __init__(self):
        self._data = {}

    def __getitem__(self, key):
        return self._data[key]

    def __setitem__(self, key, value):
        self._data[key] = value

    def __contains__(self, key):
        return key in self._data

    def __len__(self):
        return len(self._data)


d = SimpleDict()
d["name"] = "Alice"
d["age"] = 30
print(d["name"])        # Alice
print("age" in d)       # True
print(len(d))           # 2
```

## Intermediate Examples

### 1. `__call__` — callable objects

```python
class Counter:
    def __init__(self):
        self.count = 0

    def __call__(self, *args, **kwargs):
        self.count += 1
        return self.count


counter = Counter()
print(counter())  # 1
print(counter())  # 2
print(counter())  # 3
```

### 2. `__enter__` / `__exit__` — context manager

```python
class ManagedFile:
    def __init__(self, filename, mode="r"):
        self.filename = filename
        self.mode = mode
        self.file = None

    def __enter__(self):
        self.file = open(self.filename, self.mode)
        return self.file

    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.file:
            self.file.close()
        # Return False to propagate exceptions
        return False


with ManagedFile("test.txt", "w") as f:
    f.write("Hello, magic methods!")
```

### 3. `__iter__` and `__next__` — iteration protocol

```python
class Fibonacci:
    def __init__(self, max_count):
        self.max_count = max_count
        self.count = 0
        self.a, self.b = 0, 1

    def __iter__(self):
        return self

    def __next__(self):
        if self.count >= self.max_count:
            raise StopIteration
        self.count += 1
        self.a, self.b = self.b, self.a + self.b
        return self.a


for n in Fibonacci(10):
    print(n, end=" ")
# 1 1 2 3 5 8 13 21 34 55
```

### 4. `__add__`, `__sub__`, `__mul__`

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)

    def __sub__(self, other):
        return Vector(self.x - other.x, self.y - other.y)

    def __mul__(self, scalar):
        if isinstance(scalar, (int, float)):
            return Vector(self.x * scalar, self.y * scalar)
        return NotImplemented

    def __rmul__(self, scalar):
        return self.__mul__(scalar)

    def __repr__(self):
        return f"Vector({self.x}, {self.y})"


v1 = Vector(1, 2)
v2 = Vector(3, 4)
print(v1 + v2)    # Vector(4, 6)
print(v2 - v1)    # Vector(2, 2)
print(v1 * 3)     # Vector(3, 6)
print(3 * v1)     # Vector(3, 6)  # via __rmul__
```

## Advanced Examples

### 1. `__slots__` for memory efficiency

```python
import sys


class PointSlots:
    __slots__ = ("x", "y")

    def __init__(self, x, y):
        self.x = x
        self.y = y


class PointDict:
    def __init__(self, x, y):
        self.x = x
        self.y = y


p1 = PointSlots(1, 2)
p2 = PointDict(1, 2)
print(sys.getsizeof(p1))  # 48  (no __dict__)
print(sys.getsizeof(p2))  # 56  (has __dict__)
# print(p1.__dict__)      # AttributeError!
```

### 2. Custom numeric type with all operators

```python
import math


class Fraction:
    def __init__(self, num, den=1):
        if den == 0:
            raise ZeroDivisionError("Denominator cannot be zero")
        g = math.gcd(num, den) if hasattr(math, 'gcd') else abs(num if num else 1)
        self.num = num // g
        self.den = den // g
        if self.den < 0:
            self.num = -self.num
            self.den = -self.den

    def __add__(self, other):
        if isinstance(other, int):
            other = Fraction(other)
        if not isinstance(other, Fraction):
            return NotImplemented
        return Fraction(self.num * other.den + other.num * self.den,
                        self.den * other.den)

    def __radd__(self, other):
        return self.__add__(other)

    def __sub__(self, other):
        return self.__add__(-other)

    def __neg__(self):
        return Fraction(-self.num, self.den)

    def __mul__(self, other):
        if isinstance(other, int):
            other = Fraction(other)
        if not isinstance(other, Fraction):
            return NotImplemented
        return Fraction(self.num * other.num, self.den * other.den)

    def __truediv__(self, other):
        if isinstance(other, int):
            other = Fraction(other)
        if not isinstance(other, Fraction):
            return NotImplemented
        return Fraction(self.num * other.den, self.den * other.num)

    def __eq__(self, other):
        if isinstance(other, int):
            other = Fraction(other)
        if not isinstance(other, Fraction):
            return NotImplemented
        return self.num == other.num and self.den == other.den

    def __float__(self):
        return self.num / self.den

    def __repr__(self):
        return f"Fraction({self.num}, {self.den})"

    def __str__(self):
        return f"{self.num}/{self.den}"


f1 = Fraction(1, 2)
f2 = Fraction(1, 3)
print(f1 + f2)    # 5/6
print(f1 * 3)     # 3/2
print(3 * f1)     # 3/2
print(float(f1))  # 0.5
```

### 3. `__getattr__` and `__setattr__` for dynamic attributes

```python
class DynamicObject:
    def __init__(self):
        self._data = {}

    def __getattr__(self, name):
        if name.startswith("_"):
            raise AttributeError(name)
        return self._data.get(name, f"Attribute '{name}' not found")

    def __setattr__(self, name, value):
        if name.startswith("_"):
            super().__setattr__(name, value)
        else:
            self._data[name] = value

    def __delattr__(self, name):
        if name in self._data:
            del self._data[name]
        else:
            raise AttributeError(name)


obj = DynamicObject()
obj.name = "Alice"
obj.age = 30
print(obj.name)    # Alice
print(obj.age)     # 30
print(obj.missing)  # Attribute 'missing' not found
```

## Real-World Use Cases

- **SQLAlchemy models**: `__tablename__`, `__repr__`, `__eq__`, `__hash__` define ORM behaviour.
- **Requests library**: `__enter__` / `__exit__` on response objects enable `with requests.get(...) as resp`.
- **NumPy arrays**: Extensive operator overloading via `__add__`, `__mul__`, etc.
- **Decorators**: Classes with `__call__` act as parameterised decorators.
- **Pathlib**: `Path` objects implement `__truediv__` so `path / "subdir"` works.

## Common Mistakes

| Mistake | Why it's wrong | Fix |
|---|---|---|
| Implementing `__eq__` without `__hash__` | Makes class unhashable (can't use in sets/dict keys) | Implement both or set `__hash__ = None` |
| Returning `True` from `__exit__` | Suppresses all exceptions, making debugging hard | Return `False` to propagate exceptions |
| Forgetting `__repr__` | Default `<ClassName object at 0x...>` is unhelpful | Always implement `__repr__` for debugging |
| Implementing only `__str__` but not `__repr__` | `repr(obj)` falls back to ugly default | `__repr__` should be unambiguous; `__str__` readable |
| Using `__slots__` but forgetting `__dict__` in a subclass | Subclass with `__dict__` negates memory savings | Either all classes in the hierarchy use `__slots__` |
| Implementing `__getattr__` vs `__getattribute__` incorrectly | `__getattr__` is called only when normal lookup fails; `__getattribute__` always | Use `__getattr__` for dynamic attributes |

## Best Practices

- **Always implement `__repr__`** — it should return an unambiguous string that ideally recreates the object.
- Implement `__str__` for user-friendly output; it falls back to `__repr__` if absent.
- When implementing `__eq__`, also implement `__hash__` if the object is immutable and should be hashable.
- For mutable objects that implement `__eq__`, set `__hash__ = None` (it's automatically set to `None` by Python 3).
- Return `NotImplemented` (not `NotImplementedError`) from operator overloads when the operation isn't supported for the given type.
- Use `__slots__` only when you have **many instances** (thousands+) and memory matters.
- Always call `super().__init__()` in `__init__` when using inheritance if you want parent initialisation.

## Interview Questions

1. **What is the difference between `__str__` and `__repr__`?**
   *`__repr__` is for developers (unambiguous, should recreate the object). `__str__` is for end users (readable). `__repr__` is used as a fallback for `__str__`.*

2. **Why must `__eq__` and `__hash__` be consistent?**
   *Objects that compare equal must have the same hash value, otherwise dicts and sets behave incorrectly.*

3. **What does `__enter__` return and what does `__exit__` do with exceptions?**
   *`__enter__` returns the resource (often `self`). `__exit__` receives exception info; if it returns `True`, the exception is suppressed.*

4. **Explain `__slots__` and its trade-offs.**
   *`__slots__` prevents `__dict__` creation, saving memory per instance. Trade-offs: no dynamic attributes, must be inherited carefully, can't have `__weakref__` by default.*

5. **What is the difference between `__getattr__` and `__getattribute__`?**
   *`__getattribute__` is called for every attribute access; `__getattr__` is called only when normal lookup fails.*

6. **How does `__call__` make an object callable?**
   *Defining `__call__` allows an instance to be called like a function: `obj(args)`. This is used for callable classes, decorators, and closures.*

## Coding Challenges

1. **Polynomial Class**: Implement a `Polynomial` class with `__add__`, `__sub__`, `__mul__`, `__call__` (evaluate), and `__repr__`.

2. **Sparse Matrix**: Build a `SparseMatrix` with `__getitem__`, `__setitem__`, `__add__`, `__matmul__` (`@`), and `__len__`.

3. **Custom Range**: Create `MyRange` that implements `__iter__`, `__next__`, `__len__`, `__getitem__`, `__reversed__`, and `__contains__`.

4. **Timed Context Manager**: Build `Timer` context manager using `__enter__` / `__exit__` that prints elapsed time on exit.

5. **Attribute History**: Create `HistoryObj` using `__setattr__` and `__getattr__` that tracks every attribute change with timestamps.

## Summary

- **Magic methods** (dunder methods) customise object behaviour for built-in operations.
- Key categories: **string representation** (`__str__`, `__repr__`), **operators** (`__add__`, `__eq__`), **container** (`__getitem__`, `__len__`), **callable** (`__call__`), **context manager** (`__enter__`, `__exit__`), **iteration** (`__iter__`, `__next__`).
- Always return `NotImplemented` from operator overloads when type is unsupported.
- Always implement `__repr__`; implement `__str__` for readable output.
- `__eq__` and `__hash__` must be consistent for correct set/dict behaviour.
- `__slots__` saves memory but prevents dynamic attribute creation.

## Related Topics

- Classes and Objects — magic methods define how objects behave
- Polymorphism — operator overloading is a form of polymorphism
- Encapsulation — magic methods often access private attributes
- Properties — `@property` interacts with attribute access magic
- Inheritance — magic methods are inherited like regular methods
- Abstraction — ABCs can declare abstract magic methods
