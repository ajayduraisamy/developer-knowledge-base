# Data Classes - @dataclass, field(), __post_init__ (Python 3.7+)

## Introduction

Data classes, introduced in Python 3.7 via PEP 557, provide a decorator and functions that automatically generate special methods for classes that primarily store data. By simply annotating fields with type hints, `@dataclass` generates `__init__`, `__repr__`, `__eq__`, and optionally `__hash__`, `__lt__`, and other comparison methods. Data classes reduce boilerplate significantly while maintaining explicit field declarations.

## @dataclass Decorator

### What It Is

`@dataclass` is a decorator that inspects a class's type annotations and automatically generates common special methods. It was added to the standard library in Python 3.7 and is defined in the `dataclasses` module. The decorator accepts parameters to control which methods are generated and how they behave.

### Why It Is Important

Data classes eliminate repetitive boilerplate code for simple data containers. Before dataclasses, developers had to manually write `__init__`, `__repr__`, `__eq__`, and other methods for every data-holding class. Data classes make these classes more concise, less error-prone, and easier to maintain.

### How It Works Internally

When `@dataclass` is applied to a class, it analyzes the class's `__annotations__` to find all fields. For each field, it collects field information (type, default, metadata). It then generates `__init__`, `__repr__`, `__eq__`, and optionally `__hash__`, `__lt__`, `__le__`, `__gt__`, `__ge__` methods. The generated methods are set on the class, potentially overwriting any manually defined ones.

### Syntax

```python
from dataclasses import dataclass

@dataclass
class ClassName:
    field1: type
    field2: type = default
    field3: type = field(default=default, ...)

# Parameters
@dataclass(init=True, repr=True, eq=True, order=False,
           frozen=False, unsafe_hash=False, slots=False)
```

### Beginner Examples

```python
from dataclasses import dataclass

@dataclass
class Person:
    name: str
    age: int
    email: str = ""

p1 = Person("Alice", 30, "alice@example.com")
p2 = Person("Bob", 25)

print(p1)             # Person(name='Alice', age=30, email='alice@example.com')
print(p1 == p2)       # False
print(p1.name)        # Alice

# Without dataclass (boilerplate)
class PersonOld:
    def __init__(self, name, age, email=""):
        self.name = name
        self.age = age
        self.email = email

    def __repr__(self):
        return f"PersonOld(name={self.name!r}, age={self.age!r}, email={self.email!r})"

    def __eq__(self, other):
        if not isinstance(other, PersonOld):
            return NotImplemented
        return (self.name, self.age, self.email) == (other.name, other.age, other.email)
```

### Intermediate Examples

```python
from dataclasses import dataclass

# Ordering support
@dataclass(order=True)
class Score:
    value: int
    name: str

scores = [Score(85, "Alice"), Score(92, "Bob"), Score(78, "Charlie")]
scores.sort()
print(scores)
# [Score(value=78, name='Charlie'), Score(value=85, name='Alice'), Score(value=92, name='Bob')]

# Frozen (immutable) dataclass
@dataclass(frozen=True)
class Point:
    x: float
    y: float

    def distance_from_origin(self) -> float:
        return (self.x ** 2 + self.y ** 2) ** 0.5

p = Point(3, 4)
print(p.distance_from_origin())  # 5.0

# Inheritance with dataclasses
@dataclass
class Base:
    id: int
    created: str = ""

@dataclass
class User(Base):
    username: str
    email: str = ""

user = User(1, "2024-01-01", "alice", "alice@example.com")
print(user)
```

### Advanced Examples

```python
from dataclasses import dataclass, field
from typing import List, Optional
import uuid

@dataclass
class Organization:
    name: str
    members: List[str] = field(default_factory=list)

@dataclass
class Employee:
    name: str
    employee_id: str = field(default_factory=lambda: str(uuid.uuid4())[:8])
    manager: Optional['Employee'] = None
    subordinates: List['Employee'] = field(default_factory=list)

    def add_subordinate(self, emp: 'Employee') -> None:
        self.subordinates.append(emp)
        emp.manager = self

alice = Employee("Alice")
bob = Employee("Bob")
alice.add_subordinate(bob)
print(alice)

# Slots dataclass (Python 3.10+)
@dataclass(slots=True)
class FastPoint:
    x: float
    y: float
    z: float = 0.0

p = FastPoint(1, 2, 3)
```

### Real-World Use Cases

- **Configuration objects**: Store application configuration with typed fields.
- **DTOs (Data Transfer Objects)**: Transfer data between layers of an application.
- **API request/response models**: Define structured data for web APIs.
- **Database models**: Lightweight models for simple database interactions.
- **Value objects**: Immutable objects representing domain values (money, coordinates).
- **Test fixtures**: Structured test data with clear field definitions.

### Common Mistakes

- Using mutable default values (`[]`, `{}`) without `default_factory` (they're shared across instances).
- Forgetting that `frozen=True` doesn't make contained mutable objects immutable.
- Inheriting from a frozen dataclass with a non-frozen one (causes `TypeError`).
- Expecting `__hash__` to be generated automatically (disabled by default when `eq=True`).

### Best Practices

- Always use `field(default_factory=list)` instead of `field(default=[])` for mutable defaults.
- Use `frozen=True` for value objects that should be immutable.
- Use `order=True` when sorting instances by field values makes sense.
- Use `slots=True` (Python 3.10+) for memory optimization with many instances.

### Performance Considerations

Dataclass-generated methods are as fast as handwritten equivalents. `__init__` uses `setattr` for each field, which is slightly slower than direct assignment but negligible for typical use. `slots=True` reduces memory usage and improves attribute access speed. Frozen dataclasses add an immutability check per set operation.

### Interview Questions

**Q: When should you use a dataclass vs. a namedtuple?**

A: Dataclasses are more flexible — they support mutable fields, default factories, inheritance, and `__post_init__`. Namedtuples are immutable, more memory-efficient, and support tuple unpacking. Use dataclasses for complex data models; use namedtuples for simple, immutable, tuple-like structures.

**Q: How does `frozen=True` affect `__hash__`?**

A: With `frozen=True`, `__hash__` is generated automatically (using all fields) unless `eq=False` or `unsafe_hash=True` is specified. With `frozen=False` and `eq=True`, `__hash__` is set to `None` (unhashable) by default.

### Coding Challenges

1. Create a dataclass for a `ShoppingCart` with products and quantities, including a computed total.
2. Implement an immutable `Money` dataclass with currency and amount, supporting arithmetic.
3. Build a dataclass-based configuration system with validation in `__post_init__`.

### Related Topics

- `field()` function
- `__post_init__` method
- Namedtuples (alternative for immutable data)
- Pydantic (runtime validation with dataclass-like syntax)
- Type hints (required for dataclass fields)

## field() Function

### What It Is

`field()` is a function from the `dataclasses` module that provides fine-grained control over individual fields. It allows specifying defaults (including mutable defaults via `default_factory`), marking fields as excluded from `__init__`, `__repr__`, or comparison, and attaching arbitrary metadata.

### Why It Is Important

`field()` solves several problems that arise with simple type-annotated fields: mutable defaults (shared lists/dicts), fields that shouldn't be in `__init__` (computed fields), fields excluded from comparison (internal IDs), and attaching metadata for serialization or validation.

### How It Works Internally

`field()` returns a `Field` object that stores all configuration. When `@dataclass` processes the class, it checks if a field's default value is a `Field` instance. If so, it uses the Field's attributes (`default`, `default_factory`, `init`, `repr`, `compare`, `hash`, `metadata`) instead of the raw default.

### Syntax

```python
from dataclasses import field

@dataclass
class MyClass:
    normal_field: type = field(default=value)
    factory_field: list = field(default_factory=list)
    computed_field: int = field(init=False)
    private_field: str = field(repr=False)
    compare_exclude: int = field(compare=False)
    meta_field: str = field(metadata={"description": "useful info"})
```

### Beginner Examples

```python
from dataclasses import dataclass, field
from typing import List

@dataclass
class Student:
    name: str
    grades: List[int] = field(default_factory=list)
    average: float = field(init=False)

    def __post_init__(self):
        if self.grades:
            self.average = sum(self.grades) / len(self.grades)
        else:
            self.average = 0.0

s = Student("Alice", [85, 90, 92])
print(s.average)  # 89.0

s2 = Student("Bob")
print(s2.grades)  # [] (fresh list per instance)
```

### Intermediate Examples

```python
from dataclasses import dataclass, field
from typing import List, ClassVar
import time

@dataclass
class Request:
    url: str
    method: str = "GET"
    headers: dict = field(default_factory=dict, repr=False)
    timestamp: float = field(default_factory=time.time, init=False)
    _internal_id: int = field(default=0, repr=False, compare=False)
    # Class variable (not a field)
    request_count: ClassVar[int] = 0

    def __post_init__(self):
        Request.request_count += 1
        self._internal_id = Request.request_count

req1 = Request("https://api.example.com")
req2 = Request("https://api.example.com", "POST", {"Content-Type": "application/json"})
print(req1.timestamp)  # Different from req2.timestamp
print(Request.request_count)  # 2
```

### Advanced Examples

```python
from dataclasses import dataclass, field, InitVar
from typing import Optional
import hashlib

@dataclass
class User:
    username: str
    email: str
    password_hash: str = field(init=False, repr=False)
    password: InitVar[str] = None

    def __post_init__(self, password: Optional[str]):
        if password:
            self.password_hash = hashlib.sha256(password.encode()).hexdigest()
        else:
            self.password_hash = ""

    def check_password(self, password: str) -> bool:
        return hashlib.sha256(password.encode()).hexdigest() == self.password_hash

u = User("alice", "alice@example.com", "secret123")
print(u.password_hash)  # Hashed value
print(u.check_password("secret123"))  # True
print(u.check_password("wrong"))      # False
```

### Real-World Use Cases

- **Mutable defaults**: `field(default_factory=list)` for collections.
- **Computed fields**: `field(init=False)` for values computed in `__post_init__`.
- **Internal IDs**: `field(compare=False, repr=False)` for database IDs.
- **Serialization metadata**: `field(metadata={"json_key": "user_name"})` for custom serializers.
- **Init-only fields**: `InitVar` types for constructor-only parameters.

### Common Mistakes

- Using `field(default=[])` instead of `field(default_factory=list)` (shared mutable default).
- Forgetting that `init=False` fields must be set in `__post_init__`.
- Using `field()` on ClassVars (class variables don't use `field()` — they use `ClassVar`).
- Expecting `metadata` to be used by dataclasses itself (it's only for user/third-party use).

### Best Practices

- Always use `default_factory` for mutable types (list, dict, set).
- Use `init=False` for computed or derived fields.
- Use `repr=False` for sensitive fields (passwords, tokens) or very large fields.
- Use `compare=False` for fields that shouldn't participate in equality or ordering.
- Use `metadata` to attach schema information for serialization/validation tools.

### Performance Considerations

`field()` itself has negligible overhead (one function call per field at class definition time). `default_factory` calls the factory function for each new instance, which is equivalent to the cost of creating the default value manually in `__init__`.

### Interview Questions

**Q: What's the difference between `default` and `default_factory`?**

A: `default` is used for immutable default values (int, str, bool, None). `default_factory` is a zero-argument callable called each time a new instance is created, needed for mutable defaults to avoid shared state.

**Q: What is `InitVar` and when would you use it?**

A: `InitVar` is a pseudo-field that is passed to `__post_init__` but not stored as an instance field. It's used for constructor-only parameters like a password to hash, a raw config string to parse, or a database session for loading related data.

### Coding Challenges

1. Create a dataclass with a `default_factory` that generates unique IDs using `uuid4`.
2. Implement a dataclass with `init=False` fields that are computed from `InitVar` inputs.
3. Build a dataclass with `metadata` for JSON serialization that maps field names to different JSON keys.

### Related Topics

- `@dataclass` decorator
- `__post_init__` method
- `InitVar` type
- `ClassVar` type (for class-level data)
- Pydantic's `Field` (alternative with runtime validation)

## __post_init__ Method

### What It Is

`__post_init__` is a method that dataclasses call after the generated `__init__` method. It allows custom initialization logic, such as validation, derived field computation, normalization, or interaction with other fields. It receives `InitVar` fields as additional arguments.

### Why It Is Important

`__post_init__` bridges the gap between automatic field initialization and custom initialization logic. Without it, you would need to either override `__init__` (defeating the purpose of dataclasses) or use properties for computed fields. It enables validation and data transformation while keeping the dataclass declarative.

### How It Works Internally

After `@dataclass` generates `__init__`, it checks if the class defines `__post_init__`. If so, the generated `__init__` calls `self.__post_init__()` (or `self.__post_init__(*initvars)` if there are `InitVar` fields) as the last step before returning.

### Syntax

```python
@dataclass
class MyClass:
    field1: type
    field2: type = field(init=False)
    init_only: InitVar[type] = None

    def __post_init__(self, init_only: Optional[type] = None):
        # Custom initialization
        self.field2 = computed_value
```

### Beginner Examples

```python
from dataclasses import dataclass

@dataclass
class Rectangle:
    width: float
    height: float
    area: float = field(init=False)

    def __post_init__(self):
        self.area = self.width * self.height

r = Rectangle(3.0, 4.0)
print(r.area)  # 12.0

# Validation in __post_init__
@dataclass
class Temperature:
    celsius: float

    def __post_init__(self):
        if self.celsius < -273.15:
            raise ValueError(f"Temperature {self.celsius}C is below absolute zero")

t = Temperature(25)
# t = Temperature(-300)  # ValueError
```

### Intermediate Examples

```python
from dataclasses import dataclass, field
from typing import List, Optional

@dataclass
class Order:
    items: List[str]
    quantities: List[int]
    prices: List[float]
    total: float = field(init=False)

    def __post_init__(self):
        if not (len(self.items) == len(self.quantities) == len(self.prices)):
            raise ValueError("Items, quantities, and prices must have the same length")
        self.total = sum(q * p for q, p in zip(self.quantities, self.prices))

    def add_item(self, name: str, quantity: int, price: float) -> None:
        self.items.append(name)
        self.quantities.append(quantity)
        self.prices.append(price)
        self.total += quantity * price

order = Order(["Apple", "Banana"], [3, 2], [0.5, 0.75])
print(order.total)  # 3.0
order.add_item("Cherry", 5, 0.25)
print(order.total)  # 4.25
```

### Advanced Examples

```python
from dataclasses import dataclass, field, InitVar
from typing import Optional
import re

@dataclass
class EmailContact:
    name: str
    email_raw: InitVar[str]
    email: str = field(init=False)
    domain: str = field(init=False)

    def __post_init__(self, email_raw: str):
        email_clean = email_raw.strip().lower()
        if not re.match(r"^[\w.+-]+@[\w-]+\.[\w.]+$", email_clean):
            raise ValueError(f"Invalid email: {email_raw}")
        self.email = email_clean
        self.domain = email_clean.split("@")[1]

@dataclass
class Config:
    raw: InitVar[str]
    host: str = field(init=False)
    port: int = field(init=False)
    debug: bool = field(init=False)

    def __post_init__(self, raw: str):
        import json
        parsed = json.loads(raw)
        self.host = parsed.get("host", "localhost")
        self.port = parsed.get("port", 8080)
        self.debug = parsed.get("debug", False)

config = Config('{"host": "example.com", "port": 3000, "debug": true}')
print(config.host)   # example.com
print(config.port)   # 3000
print(config.debug)  # True
```

### Real-World Use Cases

- **Validation**: Validate field combinations (start_date < end_date).
- **Derived fields**: Compute `full_name` from `first_name` and `last_name`.
- **Normalization**: Trim whitespace, lowercase emails, format phone numbers.
- **Dependency injection**: Parse and inject configuration from InitVar.
- **Lazy loading**: Load related data from a database in `__post_init__`.

### Common Mistakes

- Forgetting to handle `InitVar` parameters in `__post_init__`.
- Raising exceptions in `__post_init__` that are not caught (makes instance creation fail).
- Using `__post_init__` for complex business logic (kepp it for initialization only).
- Relying on field order in `__post_init__` (fields may not be set in declaration order).
- Not using `field(init=False)` for computed fields (causes `__init__` to expect them).

### Best Practices

- Use `__post_init__` for validation and derived field computation only.
- Keep `__post_init__` focused — extract complex logic to separate methods.
- Always use `field(init=False)` for fields set in `__post_init__`.
- Document what `__post_init__` does, especially for `InitVar` fields.
- Use `InitVar` for parameters that configure initialization but aren't stored.

### Performance Considerations

`__post_init__` runs as part of `__init__`, adding to instance creation time. For frequently created instances, keep it efficient. Expensive operations (file loading, network calls) should be deferred to explicit methods rather than `__post_init__`.

### Interview Questions

**Q: What is the difference between `__post_init__` and overriding `__init__` in a dataclass?**

A: If you override `__init__` in a dataclass, the dataclass-generated `__init__` is not used. `__post_init__` runs after the generated `__init__`, allowing you to augment initialization without losing the auto-generated `__init__`.

**Q: Can `__post_init__` accept arguments other than `InitVar` fields?**

A: No. The generated `__init__` only passes `InitVar` arguments to `__post_init__`. Regular fields are already set when `__post_init__` is called.

### Coding Challenges

1. Create a dataclass that validates and normalizes a phone number in `__post_init__`.
2. Implement a dataclass with `__post_init__` that establishes bidirectional relationships between objects.
3. Build a configuration dataclass that parses environment variables in `__post_init__`.

### Related Topics

- `field()` function (used with `init=False` for `__post_init__` fields)
- `InitVar` type
- `@dataclass` decorator
- Property decorators (alternative for computed fields)
- Pydantic validators (runtime validation alternative)
