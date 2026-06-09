# Properties - @property, getters, setters, computed attributes

## Introduction

**Properties** are a Pythonic way to control access to an object's attributes. The `@property` decorator allows you to define methods that can be accessed like attributes, giving you getter, setter, and deleter behaviour without breaking the public API. This lets you start with simple attribute access and later add validation, computed values, or lazy loading — all without changing how the class is used.

Properties are built on Python's **descriptor protocol** and are one of the hallmarks of idiomatic Python.

## Why It Is Important

- **API stability**: Expose attributes directly and later add logic without breaking callers.
- **Validation**: Reject invalid values at assignment time.
- **Computed attributes**: Present derived data as if it were a stored attribute.
- **Read-only attributes**: Prevent modification by omitting the setter.
- **Lazy evaluation**: Defer expensive computation until the value is first accessed.
- **Backwards compatibility**: Turn stored attributes into computed ones without changing the public interface.

## Syntax

### Decorator style (preferred)

```python
class MyClass:
    @property
    def attr(self):
        """Getter: called when accessing obj.attr."""
        return self._attr

    @attr.setter
    def attr(self, value):
        """Setter: called when assigning obj.attr = value."""
        self._attr = value

    @attr.deleter
    def attr(self):
        """Deleter: called when deleting obj.attr."""
        del self._attr
```

### `property()` function style

```python
class MyClass:
    def get_attr(self):
        return self._attr

    def set_attr(self, value):
        self._attr = value

    attr = property(get_attr, set_attr)
```

## Examples

### Basic read/write property

```python
class Person:
    def __init__(self, name):
        self._name = name

    @property
    def name(self):
        return self._name

    @name.setter
    def name(self, value):
        if not isinstance(value, str):
            raise TypeError("Name must be a string")
        self._name = value


p = Person("Alice")
print(p.name)     # Alice
p.name = "Bob"
print(p.name)     # Bob
# p.name = 42     # TypeError: Name must be a string
```

## Beginner Examples

### 1. Temperature converter

```python
class Temperature:
    def __init__(self, celsius=0):
        self._celsius = celsius

    @property
    def celsius(self):
        return self._celsius

    @celsius.setter
    def celsius(self, value):
        if value < -273.15:
            raise ValueError("Below absolute zero")
        self._celsius = value

    @property
    def fahrenheit(self):
        return self._celsius * 9 / 5 + 32

    @fahrenheit.setter
    def fahrenheit(self, value):
        self._celsius = (value - 32) * 5 / 9


t = Temperature(100)
print(t.fahrenheit)   # 212.0
t.fahrenheit = 32
print(t.celsius)      # 0.0
```

### 2. Read-only property

```python
class User:
    def __init__(self, username):
        self._username = username
        self._login_count = 0

    @property
    def username(self):
        return self._username

    @property
    def login_count(self):
        return self._login_count

    def login(self):
        self._login_count += 1


u = User("alice99")
print(u.username)      # alice99
u.login()
u.login()
print(u.login_count)   # 2
# u.username = "bob"   # AttributeError: can't set attribute
```

### 3. Computed property

```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    @property
    def area(self):
        return self.width * self.height

    @property
    def perimeter(self):
        return 2 * (self.width + self.height)


r = Rectangle(3, 4)
print(r.area)       # 12
print(r.perimeter)  # 14
r.width = 5
print(r.area)       # 20 (recomputed)
```

## Intermediate Examples

### 1. Validation with setter

```python
class Product:
    def __init__(self, name, price):
        self.name = name
        self._price = 0
        self.price = price  # triggers setter

    @property
    def price(self):
        return self._price

    @price.setter
    def price(self, value):
        if not isinstance(value, (int, float)):
            raise TypeError("Price must be a number")
        if value < 0:
            raise ValueError("Price cannot be negative")
        if value > 1000000:
            raise ValueError("Price too high")
        self._price = value


p = Product("Widget", 49.99)
print(p.price)       # 49.99
p.price = 59.99
print(p.price)       # 59.99
```

### 2. Cached / lazy property

```python
class DataAnalyzer:
    def __init__(self, data):
        self._data = data
        self._summary = None

    @property
    def data(self):
        return self._data

    @data.setter
    def data(self, value):
        self._data = value
        self._summary = None  # invalidate cache

    @property
    def summary(self):
        if self._summary is None:
            print("Computing summary...")
            self._summary = {
                "count": len(self._data),
                "sum": sum(self._data),
                "mean": sum(self._data) / len(self._data) if self._data else 0,
                "min": min(self._data),
                "max": max(self._data),
            }
        return self._summary


da = DataAnalyzer([1, 2, 3, 4, 5])
print(da.summary)   # computes
print(da.summary)   # cached (no "Computing")
da.data = [10, 20]  # invalidates cache
print(da.summary)   # re-computes
```

### 3. Deleter property

```python
class SecureConfig:
    def __init__(self):
        self._api_key = "default-key"
        self._settings = {}

    @property
    def api_key(self):
        return "***" + self._api_key[-4:]

    @api_key.setter
    def api_key(self, value):
        if not value or len(value) < 8:
            raise ValueError("API key too short")
        self._api_key = value

    @api_key.deleter
    def api_key(self):
        print("Warning: API key being deleted!")
        self._api_key = None


config = SecureConfig()
config.api_key = "sk-1234567890"
print(config.api_key)   # ***7890
del config.api_key      # Warning: API key being deleted!
```

## Advanced Examples

### 1. Property with `functools.cached_property`

```python
from functools import cached_property
import time


class Report:
    def __init__(self, data):
        self.data = data

    @cached_property
    def expensive_computation(self):
        time.sleep(0.5)  # simulate heavy work
        return sum(x ** 2 for x in self.data)


r = Report(range(1000))
start = time.time()
v1 = r.expensive_computation
t1 = time.time() - start

start = time.time()
v2 = r.expensive_computation
t2 = time.time() - start

print(f"First access: {t1:.2f}s")   # ~0.5s
print(f"Second access: {t2:.4f}s")  # ~0.000s (cached)
```

### 2. Property descriptor (reusable property logic)

```python
class ValidatedString:
    def __set_name__(self, owner, name):
        self.name = f"_{name}"

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.name, "")

    def __set__(self, obj, value):
        if not isinstance(value, str):
            raise TypeError(f"{self.name[1:]} must be a string")
        if len(value) < 2:
            raise ValueError(f"{self.name[1:]} must have at least 2 characters")
        setattr(obj, self.name, value)


class Person:
    name = ValidatedString()
    city = ValidatedString()

    def __init__(self, name, city):
        self.name = name
        self.city = city


p = Person("Alice", "New York")
print(p.name)  # Alice
# p.name = "X" # ValueError: name must have at least 2 characters
```

### 3. Abstract property in ABC

```python
from abc import ABC, abstractmethod


class Shape(ABC):
    @property
    @abstractmethod
    def area(self):
        ...

    @property
    @abstractmethod
    def perimeter(self):
        ...


class Square(Shape):
    def __init__(self, side):
        self._side = side

    @property
    def side(self):
        return self._side

    @side.setter
    def side(self, value):
        if value <= 0:
            raise ValueError("Side must be positive")
        self._side = value

    @property
    def area(self):
        return self._side ** 2

    @property
    def perimeter(self):
        return 4 * self._side


sq = Square(5)
print(sq.area)       # 25
print(sq.perimeter)  # 20
sq.side = 10
print(sq.area)       # 100
```

## Real-World Use Cases

- **Django models**: `@property` for computed fields like `full_name` or `age_from_birthdate`.
- **Pydantic models**: Validated attributes via properties in v1; descriptor-based in v2.
- **SQLAlchemy**: Hybrid properties that work both at the Python level and in SQL queries.
- **Configuration classes**: Read-only properties for settings loaded from environment variables.
- **API responses**: Serialised computed properties like `total`, `discount`, `tax`.

## Common Mistakes

| Mistake | Why it's wrong | Fix |
|---|---|---|
| Writing explicit `get_attr()` / `set_attr()` methods | Not Pythonic; callers must use methods | Use `@property` and `@attr.setter` |
| Naming the attribute the same as the property | Infinite recursion (`self.name` calls the property) | Store in `self._name` |
| Forgetting the `@property` decorator | Method is called with `obj.attr()` instead of `obj.attr` | Add `@property` |
| Modifying the cached value directly | Cache and storage become inconsistent | Invalidate the cache in the setter |
| Making heavy computations in a getter | Unexpected performance cost | Cache or use `@cached_property` |
| Using `@property` for trivial attribute access | Unnecessary overhead and boilerplate | Use plain attributes unless you need validation |

```python
# Infinite recursion — BAD
class Bad:
    @property
    def name(self):
        return self.name   # calls the property again!

    @name.setter
    def name(self, value):
        self.name = value  # infinite recursion

# Correct — GOOD
class Good:
    @property
    def name(self):
        return self._name

    @name.setter
    def name(self, value):
        self._name = value
```

## Best Practices

- Always store the backing attribute with an underscore prefix: `self._attr`.
- Start with **plain attributes** and migrate to `@property` when validation or computation is needed.
- Keep property getters **fast** and **free of side effects**.
- Use `@cached_property` for expensive, idempotent computations.
- **Document** properties like methods, explaining what they return.
- Use the **decorator style** over `property()` function — it's cleaner.
- Prefer `@property` over explicit getter/setter methods.
- Don't raise `AttributeError` from a property; raise `ValueError` or `TypeError` instead.

## Interview Questions

1. **What is the difference between a property and a regular attribute?**
   *A property is a descriptor that intercepts attribute access, allowing getter/setter/deleter logic. A regular attribute is stored directly in `obj.__dict__`.*

2. **How does `@property` work at the implementation level?**
   *`property` is a descriptor class that implements `__get__`, `__set__`, and `__delete__`. The decorator syntax creates a property object and stores it on the class.*

3. **What is the difference between `@property` and `@cached_property`?**
   *`@property` recomputes on every access. `@cached_property` computes once, stores the result in the instance dict, and returns the cached value on subsequent accesses.*

4. **Can you create a read-only property?**
   *Yes — define only the getter with `@property` and omit the setter.*

5. **What happens if a property raises an exception?**
   *The exception propagates to the caller, just like with a regular method.*

6. **Explain the property() function signature.**
   *`property(fget=None, fset=None, fdel=None, doc=None)` creates a property from explicit getter/setter/deleter functions.*

## Coding Challenges

1. **Circle with Validation**: `Circle` class with `radius` property (validated: > 0). Computed read-only properties `diameter`, `circumference`, `area`.

2. **Employee with Computed Properties**: `Employee` with `salary`, `bonus_percent`. Computed `total_compensation` and `tax` (at 30%). Invalidate when `salary` changes.

3. **Blog Post Character Counter**: `BlogPost` with `title` and `content` properties (validated strings). Computed `word_count`, `char_count`, `reading_time` (200 words/min).

4. **Unit Converter**: `Measurement` class with `value`, `from_unit`, `to_unit`. Computed `converted` property that caches the conversion result.

5. **Config with Lazy Loading**: `AppConfig` loads settings from a JSON file only when first accessed via properties. Cache loaded values. Reload when explicitly requested.

## Summary

- **Properties** provide getter/setter/deleter behaviour with attribute-like syntax.
- `@property` decorator is the Pythonic way to define them.
- Use properties for **validation**, **computed values**, **lazy loading**, and **read-only access**.
- `@cached_property` caches the result after the first computation.
- Store backing data in `self._attr` to avoid infinite recursion.
- Start with plain attributes; add properties when the need arises for API stability.

## Related Topics

- Classes and Objects — properties are class-level descriptors
- Encapsulation — properties are the Pythonic encapsulation tool
- Abstraction — abstract properties define required property interfaces
- Magic Methods — properties are built on the descriptor protocol (`__get__`, `__set__`)
- Inheritance — properties are inherited; overrides must use the `@property` decorator again
- Static and Class Methods — alternative types of class-level methods
