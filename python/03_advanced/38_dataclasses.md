# Data Classes - @dataclass, field(), __post_init__ (Python 3.7+)

## Introduction

A data class is a class decorated with `@dataclass` that automatically generates special methods like `__init__`, `__repr__`, `__eq__`, `__hash__`, and `__order__` based on class annotations. Introduced in Python 3.7 via PEP 557, dataclasses are designed to reduce boilerplate for classes that primarily serve as data containers. They are similar to named tuples but are mutable by default and fully featured classes.

## Why It Is Important

Dataclasses eliminate the tedious boilerplate of writing `__init__`, `__repr__`, `__eq__`, and other methods for data-holding classes. They provide sensible defaults while remaining customizable through parameters like `frozen`, `order`, and `slots`. They integrate with type annotations, making code self-documenting. For data-oriented classes (DTOs, value objects, configuration), dataclasses are often the best choice.

## Syntax

```python
from dataclasses import dataclass, field, asdict, astuple, replace, fields, FrozenInstanceError

@dataclass
class Point:
    x: float
    y: float
    z: float = 0.0

# Parameters
@dataclass(frozen=True, order=True, slots=True)
class ImmutablePoint:
    x: float
    y: float
```

## Examples

```python
from dataclasses import dataclass, field, asdict, astuple, replace, fields, FrozenInstanceError, InitVar
from typing import Any, ClassVar, Optional, list
import json
```

### Basic Dataclass

```python
@dataclass
class Person:
    name: str
    age: int
    email: str

p = Person("Alice", 30, "alice@example.com")
print(p)
print(p.name)
```

### Auto-generated Methods

```python
@dataclass
class Book:
    title: str
    author: str
    pages: int
    isbn: str = ""

b1 = Book("Python 101", "John", 300, "123-456")
b2 = Book("Python 101", "John", 300, "123-456")
b3 = Book("Advanced Python", "Jane", 400)

print(b1)             # Auto __repr__
print(b1 == b2)       # Auto __eq__ (True)
print(b1 is b2)       # False (different objects)
```

## Beginner Examples

### Default Values and Immutable Defaults

```python
@dataclass
class Student:
    name: str
    grades: list[int] = field(default_factory=list)
    active: bool = True
    year: int = 2024

s1 = Student("Alice")
s2 = Student("Bob", [85, 90, 78])
s1.grades.append(95)  # OK: each instance has its own list

print(s1)
print(s2)
```

### Frozen (Immutable) Dataclass

```python
@dataclass(frozen=True)
class Config:
    host: str
    port: int
    debug: bool = False

config = Config("localhost", 8080)
print(config)

# config.port = 9090  # Raises FrozenInstanceError
```

### Comparison and Ordering

```python
@dataclass(order=True)
class Product:
    name: str
    price: float
    quantity: int

p1 = Product("Apple", 1.50, 100)
p2 = Product("Banana", 0.75, 200)
p3 = Product("Cherry", 3.00, 50)

products = [p1, p2, p3]
products.sort()
print(products)

print(p1 < p2)  # False (compares all fields in order)
```

### Field with Metadata

```python
@dataclass
class User:
    name: str
    password: str = field(repr=False)  # Don't show in repr
    id: int = field(compare=False)     # Don't use in equality

u = User("Alice", "secret123", 42)
print(u)
```

## Intermediate Examples

### __post_init__ for Post-Initialization

```python
@dataclass
class Rectangle:
    width: float
    height: float

    def __post_init__(self) -> None:
        if self.width <= 0 or self.height <= 0:
            raise ValueError("Dimensions must be positive")
        self._area = self.width * self.height
        self._perimeter = 2 * (self.width + self.height)

    @property
    def area(self) -> float:
        return self._area

    @property
    def perimeter(self) -> float:
        return self._perimeter

r = Rectangle(10, 5)
print(r.area)
print(r.perimeter)
```

### InitVar (Initialization-Only Variables)

```python
@dataclass
class DatabaseConnection:
    host: str
    port: int
    timeout: InitVar[int] = 30
    _connection: Optional[str] = field(init=False, default=None)

    def __post_init__(self, timeout: int) -> None:
        # timeout is provided to __post_init__ as an argument
        self._connection = f"{self.host}:{self.port} (timeout={timeout}s)"

db = DatabaseConnection("localhost", 5432, timeout=60)
print(db._connection)
# print(db.timeout)  # AttributeError: no 'timeout' field
```

### Inheritance with Dataclasses

```python
@dataclass
class Base:
    x: int
    y: int

@dataclass
class Derived(Base):
    z: int

    def __post_init__(self) -> None:
        super().__post_init__()
        self.magnitude = (self.x ** 2 + self.y ** 2 + self.z ** 2) ** 0.5

d = Derived(3, 4, 5)
print(d)
print(d.magnitude)
```

### Factory Pattern with Dataclass

```python
from datetime import datetime

@dataclass
class Order:
    order_id: str
    items: list[str] = field(default_factory=list)
    created_at: datetime = field(default_factory=datetime.now)
    status: str = "pending"

    @classmethod
    def create(cls, items: list[str]) -> 'Order':
        import uuid
        return cls(
            order_id=str(uuid.uuid4())[:8],
            items=items,
        )

order = Order.create(["apple", "banana", "cherry"])
print(order)
```

## Advanced Examples

### Dataclass with Validation

```python
@dataclass
class Employee:
    name: str
    age: int
    salary: float
    department: str

    def __post_init__(self) -> None:
        errors: list[str] = []
        if not self.name.strip():
            errors.append("Name cannot be empty")
        if not (18 <= self.age <= 65):
            errors.append(f"Age {self.age} outside valid range [18, 65]")
        if self.salary < 0:
            errors.append("Salary cannot be negative")
        if self.department not in ("Engineering", "Sales", "HR"):
            errors.append(f"Unknown department: {self.department}")
        if errors:
            raise ValueError("; ".join(errors))

e = Employee("Alice", 30, 75000, "Engineering")
print(e)
```

### Dataclass Serialization

```python
@dataclass
class Address:
    street: str
    city: str
    zip_code: str

@dataclass
class Customer:
    name: str
    age: int
    address: Address
    tags: list[str] = field(default_factory=list)

    def to_json(self) -> str:
        return json.dumps(asdict(self), indent=2)

    @classmethod
    def from_json(cls, data: str) -> 'Customer':
        raw = json.loads(data)
        return cls(**raw)

customer = Customer(
    "Alice",
    30,
    Address("123 Main St", "Springfield", "12345"),
    tags=["vip", "premium"],
)

json_str = customer.to_json()
print(json_str)

restored = Customer.from_json(json_str)
print(restored)
```

### Dataclass with Computed Fields

```python
@dataclass
class Statistics:
    data: list[float] = field(default_factory=list)
    _count: int = field(init=False, repr=False)
    _mean: float = field(init=False, repr=False)
    _variance: float = field(init=False, repr=False)

    def __post_init__(self) -> None:
        n = len(self.data)
        self._count = n
        self._mean = sum(self.data) / n if n > 0 else 0.0
        self._variance = (
            sum((x - self._mean) ** 2 for x in self.data) / n
            if n > 0 else 0.0
        )

    @property
    def count(self) -> int:
        return self._count

    @property
    def mean(self) -> float:
        return self._mean

    @property
    def std(self) -> float:
        return self._variance ** 0.5

stats = Statistics([1.0, 2.0, 3.0, 4.0, 5.0])
print(f"n={stats.count}, mean={stats.mean:.2f}, std={stats.std:.2f}")
```

### Dataclass with Custom __hash__

```python
@dataclass
class MutablePoint:
    x: int
    y: int

# By default, __hash__ is None for mutable dataclasses
p = MutablePoint(1, 2)
# hash(p)  # TypeError

@dataclass(frozen=True)
class ImmutablePoint:
    x: int
    y: int

# Frozen dataclasses have __hash__
p = ImmutablePoint(1, 2)
print(hash(p))

# Manual hash for mutable dataclass
@dataclass(unsafe_hash=True)
class SafeHashPoint:
    x: int
    y: int

p = SafeHashPoint(3, 4)
print(hash(p))
```

### Dataclass with ClassVar and InitVar

```python
@dataclass
class Repository:
    name: str
    url: str
    stars: int = 0
    _cache: ClassVar[dict[str, 'Repository']] = {}
    api_token: InitVar[str] = ""

    def __post_init__(self, api_token: str) -> None:
        if api_token:
            self._setup_api(api_token)

    def _setup_api(self, token: str) -> None:
        self._token = token

    @classmethod
    def get_cached(cls, name: str) -> Optional['Repository']:
        return cls._cache.get(name)

    def __post_init__(self) -> None:
        self._cache[self.name] = self

repo = Repository("opencode", "https://github.com/opencode", api_token="secret")
print(Repository.get_cached("opencode"))
```

### Field with Conversion

```python
@dataclass
class Temperature:
    celsius: float

    @classmethod
    def from_fahrenheit(cls, f: float) -> 'Temperature':
        return cls(celsius=(f - 32) * 5 / 9)

    @property
    def fahrenheit(self) -> float:
        return self.celsius * 9 / 5 + 32

    def __post_init__(self) -> None:
        if self.celsius < -273.15:
            raise ValueError("Below absolute zero")

t = Temperature(100)
print(f"{t.celsius}C = {t.fahrenheit}F")

t2 = Temperature.from_fahrenheit(212)
print(f"{t2.celsius:.1f}C")
```

### Dataclass as DTO with field options

```python
@dataclass
class APIResponse:
    status: int
    message: str
    data: dict[str, Any] = field(default_factory=dict)
    _internal_id: str = field(default="", repr=False, compare=False)
    cache_ttl: int = field(default=300, metadata={"unit": "seconds"})

    def __bool__(self) -> bool:
        return 200 <= self.status < 300

resp = APIResponse(200, "OK", {"user": "Alice"})
print(resp)
print(bool(resp))

# Access metadata
for f in fields(APIResponse):
    print(f.name, f.metadata)
```

### Dataclass with slots (Python 3.10+)

```python
@dataclass(slots=True)
class SlottedPoint:
    x: float
    y: float

p = SlottedPoint(3.0, 4.0)
print(p)
# p.z = 5.0  # AttributeError: 'SlottedPoint' has no attribute 'z'
```

## Real-World Use Cases

- **DTOs (Data Transfer Objects)**: Representing API request/response data.
- **Configuration objects**: Type-safe configuration holders.
- **Value objects**: Domain-driven design with immutable data holders.
- **Database models**: Lightweight ORM models (SQLAlchemy supports dataclass integration).
- **Test fixtures**: Clean data definitions for test cases.
- **Message payloads**: Structuring messages for queues or event systems.
- **Serialization targets**: JSON/YAML/XML serialization with `asdict()`.
- **State representation**: Finite state machine states and transitions.

## Common Mistakes

- Using mutable default values (lists, dicts) without `field(default_factory=...)`.
- Forgetting that `@dataclass` generates `__init__` — can't have another `__init__`.
- Expecting frozen dataclasses to prevent mutation of mutable fields (only prevents reassignment).
- Using `order=True` without understanding it compares all fields in order.
- Adding `@property` methods that conflict with generated field names.
- Not calling `super().__post_init__()` in derived dataclasses.
- Overlooking that `__hash__` is set to `None` when `eq=True` and `frozen=False`.

## Best Practices

- Always use `field(default_factory=...)` for mutable default values.
- Use `frozen=True` for value objects that should be immutable.
- Use `order=True` only when natural ordering makes sense.
- Use `__post_init__` for validation and computed fields.
- Use `InitVar` for parameters needed only during initialization.
- Use `slots=True` (Python 3.10+) for memory efficiency.
- Prefer dataclasses over named tuples when you need mutability or methods.
- Use `asdict()` for serialization, but be careful with deeply nested objects.

## Interview Questions

1. What is a dataclass and what problem does it solve?
2. What methods does `@dataclass` automatically generate?
3. How do you handle mutable default values in dataclasses?
4. What is `__post_init__` and when would you use it?
5. What is the difference between `dataclass` and `namedtuple`?
6. How do you make a dataclass immutable?
7. What is `field()` and what parameters does it accept?
8. What is `InitVar` and how is it different from a regular field?
9. How do dataclasses handle inheritance?
10. What is `asdict()` and how does it handle nested dataclasses?

## Coding Challenges

1. **Validation Dataclass**: Create a dataclass for a `Student` that validates grades are 0-100.
2. **Serializable Config**: Build a `Config` dataclass that can be serialized/deserialized from JSON.
3. **Bank Account**: Create an immutable `Transaction` and a mutable `Account` dataclass.
4. **Nested Dataclasses**: Implement `Company` -> `Department` -> `Employee` with nested dataclasses.
5. **Custom __post_init__**: Write a dataclass that computes a field based on init args.
6. **Database Row**: Create a dataclass that maps database column types to Python types.
7. **Cached Property**: Implement a dataclass with lazy-loaded external data.
8. **Tree Structure**: Build a recursive tree node dataclass with children.

## Summary

Dataclasses reduce boilerplate for data-holding classes by automatically generating `__init__`, `__repr__`, `__eq__`, `__hash__`, and ordering methods. They support type annotations, default values, immutability, initialization hooks, and integration with the `typing` module. Dataclasses are the recommended approach for most data-oriented class definitions in modern Python.

## Related Topics

- Type annotations
- Named tuples
- attrs library (third-party, inspired dataclasses)
- Properties and descriptors
- Serialization
- Value objects and DTOs
- Python data model