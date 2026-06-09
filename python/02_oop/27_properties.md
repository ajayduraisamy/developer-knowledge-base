# Properties - @property, getters, setters, computed attributes

## Introduction

Properties are Python's mechanism for controlled attribute access. The `@property` decorator transforms a method into a "getter" that looks like a plain attribute to users, while enabling validation, computed values, lazy loading, and other behaviors. Properties implement the concept of "managed attributes" — they let you start with simple attribute access and later add logic without changing the public API.

## @property decorator

### What It Is

`@property` is a built-in decorator that converts a method into a read-only attribute. The decorated method is called whenever the attribute is accessed, allowing dynamic computation or validation.

```python
class Circle:
    def __init__(self, radius):
        self._radius = radius

    @property
    def radius(self):
        return self._radius
```

### Why It Is Important

Properties enable the "Uniform Access Principle" — callers access attributes and computed values the same way (`obj.attr`). You can start with a simple attribute and later add getter/setter logic without breaking existing code. Properties are essential for encapsulation in Python.

### How It Works Internally

`@property` creates a `property` descriptor object. When you access `obj.prop`, Python finds the descriptor on the class, calls its `__get__` method, which executes the getter function. The property descriptor stores references to the getter, setter, and deleter functions.

```python
# Internally, this:
@property
def radius(self):
    return self._radius

# Is equivalent to:
radius = property(lambda self: self._radius)
```

### Syntax

```python
class MyClass:
    def __init__(self):
        self._value = 0

    @property
    def value(self):          # Getter
        return self._value

    @value.setter
    def value(self, new_val):  # Setter
        self._value = new_val

    @value.deleter
    def value(self):           # Deleter
        del self._value
```

### Beginner Examples

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
            raise ValueError("Temperature below absolute zero")
        self._celsius = value

    @property
    def fahrenheit(self):
        return self._celsius * 9 / 5 + 32

t = Temperature(100)
print(t.celsius)      # 100
print(t.fahrenheit)   # 212.0
t.celsius = 0
print(t.fahrenheit)   # 32.0
# t.fahrenheit = 50   # AttributeError: can't set attribute
```

### Intermediate Examples

```python
import re

class Email:
    def __init__(self, address):
        self.address = address  # Uses the setter

    @property
    def address(self):
        return self._address

    @address.setter
    def address(self, value):
        if not re.match(r"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$", value):
            raise ValueError(f"Invalid email: {value}")
        self._address = value

    @property
    def domain(self):
        return self._address.split("@")[1]

    @property
    def username(self):
        return self._address.split("@")[0]

e = Email("alice@example.com")
print(e.domain)      # example.com
print(e.username)    # alice
# e.address = "invalid"  # ValueError
e.address = "bob@test.org"
print(e.address)     # bob@test.org
```

### Advanced Examples

```python
import threading
from typing import Optional

class ThreadSafeSingleton:
    _instance: Optional["ThreadSafeSingleton"] = None
    _lock = threading.Lock()

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self):
        self._config = {}

    @property
    def config(self):
        return dict(self._config)  # Return a copy

    @config.setter
    def config(self, config_dict: dict):
        if not isinstance(config_dict, dict):
            raise TypeError("Config must be a dict")
        with self._lock:
            self._config = config_dict.copy()

    @config.deleter
    def config(self):
        with self._lock:
            self._config = {}

class LazyProperty:
    """Descriptor that caches the computed value."""
    def __init__(self, func):
        self.func = func
        self.name = func.__name__

    def __get__(self, obj, cls):
        if obj is None:
            return self
        value = self.func(obj)
        obj.__dict__[self.name] = value  # Cache in instance dict
        return value

class DataAnalyzer:
    def __init__(self, data):
        self.data = data

    @LazyProperty
    def mean(self):
        print("Computing mean...")
        return sum(self.data) / len(self.data)

    @LazyProperty
    def std(self):
        print("Computing std...")
        m = self.mean
        variance = sum((x - m) ** 2 for x in self.data) / len(self.data)
        return variance ** 0.5

da = DataAnalyzer([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
print(da.mean)  # Computes once
print(da.mean)  # Returns cached value
print(da.std)
```

### Real-World Use Cases

- **Django model fields**: properties for computed fields like `full_name`
- **Pydantic models**: validation via property setters
- **API clients**: `response.data` deserializes lazily
- **ORM models**: `user.posts` lazy-loads related objects
- **Configuration objects**: validation on setting, caching on reading

### Common Mistakes

1. **Recursion**: property and backing store with same name
```python
@property
def value(self):
    return self.value  # Recursion! Should be self._value
```

2. **Expensive operations in getter**: users don't expect attribute access to be slow
3. **Setter without validation**: defeats the purpose of using a property
4. **Not using property when API stability is needed**: exposing raw attributes makes changes harder

### Best Practices

- Prefix backing attributes with `_` (e.g., `_radius`)
- Keep getters simple and fast; defer heavy computation to explicit methods
- Validate in setters immediately
- Document if a property might raise exceptions or be expensive
- Use properties for interface stability, not for every attribute

### Performance Considerations

- Property access is ~5x slower than direct attribute access
- @LazyProperty caches result after first computation
- Property overhead is negligible in I/O-bound code
- For CPU-bound hot loops, access `_backing` directly

## Getters and setters

### What It Is

Getters and setters are methods that control access to an attribute. In Python, they're implemented via the `@property` getter and `@attr.setter` decorator, providing controlled read and write access to an attribute.

```python
class Person:
    @property
    def name(self):     # Getter
        return self._name

    @name.setter
    def name(self, v):  # Setter
        self._name = v.strip().title()
```

### Why It Is Important

Getters and setters enforce invariants, validate data, and provide a stable API. Unlike Java-style `get_name()`/`set_name()`, Python's property-based getters/setters are syntactically transparent — callers use simple attribute syntax.

### How It Works Internally

The setter is called when `obj.attr = value` is executed. Python finds the property descriptor on the class, checks if it has a setter function, and calls it with the assigned value. Without a setter, assignment raises `AttributeError`.

```python
obj.attr = 42
# Internally: type(obj).__dict__['attr'].fset(obj, 42)
```

### Syntax

```python
class ClassName:
    @property
    def attr(self):          # getter
        return self._attr

    @attr.setter
    def attr(self, value):   # setter
        # validation, transformation
        self._attr = value

    @attr.deleter
    def attr(self):          # deleter
        del self._attr
```

### Beginner Examples

```python
class Student:
    def __init__(self, name, grade):
        self.name = name
        self.grade = grade   # Uses setter for validation

    @property
    def grade(self):
        return self._grade

    @grade.setter
    def grade(self, value):
        if not (0 <= value <= 100):
            raise ValueError("Grade must be 0-100")
        self._grade = value

    @property
    def is_passing(self):
        return self._grade >= 60

s = Student("Alice", 85)
print(s.grade)       # 85
print(s.is_passing)  # True
s.grade = 95
# s.grade = 150      # ValueError
```

### Intermediate Examples

```python
import hashlib
import os

class User:
    def __init__(self, username, password):
        self.username = username
        self.password = password  # Uses setter — hashed immediately

    @property
    def password(self):
        raise AttributeError("Password is write-only")

    @password.setter
    def password(self, value):
        if len(value) < 8:
            raise ValueError("Password must be at least 8 characters")
        salt = os.urandom(32)
        hash_obj = hashlib.pbkdf2_hmac("sha256", value.encode(), salt, 100000)
        self._password_hash = salt + hash_obj

    def verify_password(self, candidate):
        salt = self._password_hash[:32]
        stored_hash = self._password_hash[32:]
        candidate_hash = hashlib.pbkdf2_hmac("sha256", candidate.encode(), salt, 100000)
        return candidate_hash == stored_hash

u = User("alice", "securepass123")
print(u.verify_password("wrong"))      # False
print(u.verify_password("securepass123"))  # True
# print(u.password)  # AttributeError
```

### Advanced Examples

```python
from datetime import datetime

class Document:
    def __init__(self, content=""):
        self._content = content
        self._modified_at = datetime.now()
        self._version = 1

    @property
    def content(self):
        return self._content

    @content.setter
    def content(self, value):
        if value == self._content:
            return
        self._version += 1
        self._content = value
        self._modified_at = datetime.now()

    @property
    def version(self):
        return self._version

    @property
    def modified_at(self):
        return self._modified_at

    @property
    def word_count(self):
        return len(self._content.split())

    @property
    def char_count(self):
        return len(self._content)

    @content.deleter
    def content(self):
        self._content = ""
        self._version += 1
        self._modified_at = datetime.now()

# Inheritance with properties
class ReadOnlyDocument(Document):
    @Document.content.setter
    def content(self, value):
        raise AttributeError("This document is read-only")

class LoggedDocument(Document):
    @Document.content.setter
    def content(self, value):
        old = self._content
        super(type(self), type(self)).content.__set__(self, value)
        print(f"Content changed. Old length: {len(old)}, New length: {len(value)}")

doc = Document("Hello world")
print(doc.word_count)  # 2
doc.content = "Hello Python world"
print(doc.version)     # 2
```

### Real-World Use Cases

- **Validation**: age, price, email validation on set
- **Transformation**: automatically normalizing strings (trim, lowercase)
- **Lazy loading**: loading data from database only when accessed
- **Change tracking**: logging modifications, incrementing version counters
- **Access control**: permission checks before returning sensitive data

### Common Mistakes

1. **Creating getter but no setter unintentionally**: immutable properties are fine, but be intentional
2. **Throwing exceptions from getters**: users don't expect attribute access to fail
3. **Side effects in getters**: reading an attribute should not modify state
4. **Property and method with same name**: will shadow one or the other

### Best Practices

- Keep getters side-effect-free
- Validate all input in setters immediately
- Use setters for data normalization (trimming, lowercasing, etc.)
- Consider read-only properties for computed values
- Don't use properties for expensive operations without documentation

### Performance Considerations

- Each property access is a method call
- For frequently accessed attributes in tight loops, cache or use direct `_attr` access
- Property setter validation adds overhead proportional to validation complexity

## Computed attributes

### What It Is

Computed attributes (also called derived attributes) are properties whose value is computed on-the-fly from other attributes. They look like regular attributes to callers but reflect dynamic state.

```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    @property
    def area(self):
        return self.width * self.height
```

### Why It Is Important

Computed attributes keep data consistent. Instead of storing both `width * height` and `area` separately (risking stale data), area is always computed from the authoritative values. This eliminates synchronization bugs.

### How It Works Internally

Computed properties work identically to regular properties — `@property` creates a getter-only descriptor. Each access calls the function, computes the result, and returns it. Computed attributes are not cached by default.

```python
rect = Rectangle(3, 4)
print(rect.area)   # 12 — computed each time
rect.width = 5
print(rect.area)   # 20 — always up to date
```

### Syntax

```python
class ClassName:
    @property
    def computed(self):
        return self._dep1 + self._dep2
```

### Beginner Examples

```python
class Circle:
    def __init__(self, radius):
        self.radius = radius

    @property
    def diameter(self):
        return self.radius * 2

    @property
    def circumference(self):
        return 2 * 3.14159 * self.radius

    @property
    def area(self):
        return 3.14159 * self.radius ** 2

c = Circle(10)
print(c.diameter)       # 20
print(c.circumference)  # 62.8318
print(c.area)           # 314.159
```

### Intermediate Examples

```python
class OrderItem:
    def __init__(self, name, price, quantity):
        self.name = name
        self.price = price
        self.quantity = quantity

    @property
    def subtotal(self):
        return self.price * self.quantity

    @property
    def tax(self):
        return self.subtotal * 0.08

    @property
    def total(self):
        return self.subtotal + self.tax

class Order:
    def __init__(self, items=None):
        self._items = items or []

    def add_item(self, item):
        self._items.append(item)

    @property
    def subtotal(self):
        return sum(item.subtotal for item in self._items)

    @property
    def tax(self):
        return sum(item.tax for item in self._items)

    @property
    def total(self):
        return sum(item.total for item in self._items)

    @property
    def item_count(self):
        return sum(item.quantity for item in self._items)

    @property
    def average_item_price(self):
        if not self._items:
            return 0
        return self.subtotal / self.item_count

order = Order()
order.add_item(OrderItem("Widget", 10, 2))
order.add_item(OrderItem("Gadget", 25, 1))
print(f"Total: ${order.total:.2f}")        # $48.60
print(f"Items: {order.item_count}")         # 3
print(f"Avg: ${order.average_item_price:.2f}")  # $15.00
```

### Advanced Examples

```python
from datetime import datetime, timedelta
from typing import Optional

class Task:
    def __init__(self, title, due_date=None, estimated_hours=0):
        self.title = title
        self.due_date = due_date
        self.estimated_hours = estimated_hours
        self._status = "todo"
        self._completed_at = None
        self._started_at = None

    @property
    def status(self):
        return self._status

    @status.setter
    def status(self, value):
        if value == "in_progress" and self._status == "todo":
            self._started_at = datetime.now()
        elif value == "done" and self._status == "in_progress":
            self._completed_at = datetime.now()
        self._status = value

    @property
    def is_overdue(self) -> bool:
        if not self.due_date:
            return False
        return datetime.now() > self.due_date and self._status != "done"

    @property
    def is_blocked(self) -> bool:
        return self._status == "blocked"

    @property
    def days_remaining(self) -> Optional[int]:
        if not self.due_date or self._status == "done":
            return None
        delta = self.due_date - datetime.now()
        return max(0, delta.days)

    @property
    def progress_pct(self) -> float:
        status_values = {"todo": 0, "in_progress": 50, "blocked": 25, "done": 100}
        return status_values.get(self._status, 0)

    @property
    def time_spent(self) -> timedelta:
        if not self._started_at:
            return timedelta()
        end = self._completed_at or datetime.now()
        return end - self._started_at

    @property
    def efficiency_ratio(self) -> Optional[float]:
        if not self.estimated_hours:
            return None
        actual_hours = self.time_spent.total_seconds() / 3600
        if actual_hours == 0:
            return None
        return self.estimated_hours / actual_hours


# Graph traversal with computed paths
class GraphNode:
    def __init__(self, name):
        self.name = name
        self._neighbors: list["GraphNode"] = []

    def connect(self, other: "GraphNode"):
        self._neighbors.append(other)
        other._neighbors.append(self)

    @property
    def degree(self) -> int:
        return len(self._neighbors)

    @property
    def neighbor_names(self) -> list:
        return [n.name for n in self._neighbors]

    def bfs_distance_to(self, target: "GraphNode") -> Optional[int]:
        from collections import deque
        visited = {self}
        queue = deque([(self, 0)])
        while queue:
            node, dist = queue.popleft()
            if node is target:
                return dist
            for neighbor in node._neighbors:
                if neighbor not in visited:
                    visited.add(neighbor)
                    queue.append((neighbor, dist + 1))
        return None

# Build graph
a = GraphNode("A")
b = GraphNode("B")
c = GraphNode("C")
a.connect(b)
b.connect(c)
print(a.bfs_distance_to(c))  # 2
```

### Real-World Use Cases

- **ORM models**: `user.full_name = f"{user.first_name} {user.last_name}"`
- **E-commerce**: `order.total`, `order.tax`, `shipping_cost`
- **Analytics**: `dashboard.monthly_revenue`, `dashboard.growth_rate`
- **Graphics**: `image.aspect_ratio`, `image.megapixels`
- **Machine learning**: `model.accuracy`, `model.feature_importance`

### Common Mistakes

1. **Computing expensive values without caching**: `@property` recomputes every access
2. **Storing computed values redundantly**: leads to sync issues
3. **Using properties when a method is clearer**: `image.save()` should not be a property
4. **Side effects in computed properties**: reading should not modify state

### Best Practices

- Computed properties should be fast (< 1ms). Cache slow computations
- Use methods for operations (verbs), properties for attributes (nouns)
- Document if the property is expensive or has side effects
- Consider `functools.cached_property` for one-time expensive computations
- Chain computed properties for complex derived state

### Performance Considerations

- Each access recomputes; use `@cached_property` for expensive computations
- `cached_property` (Python 3.8+) caches on first access, invalidates on `del obj.attr`
- For properties that depend on mutable state, manual cache invalidation is needed
- Computed property chains (A → B → C) recompute entire chain each access

### Interview Questions

1. How is a Python property different from Java-style getters/setters?
2. What happens when you access `obj.prop` where `prop` is a property?
3. How would you implement lazy initialization using a property?
4. What is the difference between `@property` and `@cached_property`?
5. Can you have a property without a setter? What happens on assignment?

### Coding Challenges

1. Create a `BankAccount` class with properties for `balance` (validated), `interest_rate`, and computed `monthly_interest`.
2. Implement a `Product` class with properties for `price` (must be positive), `discount` (0-100%), and computed `final_price`.
3. Build a `TimeSheet` class with properties for `hours_worked`, `overtime`, `daily_overtime_limit`, and computed `total_pay`.

### Related Topics

Descriptors, `__getattr__`, `__setattr__`, `cached_property`, Encapsulation
