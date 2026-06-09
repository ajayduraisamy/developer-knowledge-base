# Enums - Enum class, auto(), IntEnum, Flag (Python 3.4+)

## Introduction

An enumeration (enum) is a symbolic name for a set of distinct, constant values. Python's `enum` module (introduced in Python 3.4) provides a robust way to define and work with enumerations. Enums group related constants together in a type-safe manner, providing better readability, maintainability, and debugging compared to plain integers or strings.

## Why It Is Important

Enums replace "magic numbers" and string constants with named, self-documenting values. They provide type safety (preventing accidental comparison with unrelated values), iteration over members, automatic value assignment, and integration with other Python features like `isinstance()` checks. Enums are essential for representing fixed sets of options (days of the week, directions, status codes, HTTP methods, etc.) and are widely used in APIs, configuration, and domain modeling.

## Syntax

```python
from enum import Enum, auto, IntEnum, StrEnum, Flag, unique

class Color(Enum):
    RED = 1
    GREEN = 2
    BLUE = 3

class Status(Enum):
    PENDING = auto()
    ACTIVE = auto()
    INACTIVE = auto()

# Access
Color.RED
Color(1)          # Member by value
Color['RED']      # Member by name
```

## Examples

```python
from enum import Enum, auto, IntEnum, unique, Flag, EnumMeta
from typing import Any
```

### Basic Enum

```python
class Direction(Enum):
    NORTH = 1
    SOUTH = 2
    EAST = 3
    WEST = 4

print(Direction.NORTH)
print(Direction.NORTH.name)
print(Direction.NORTH.value)
print(repr(Direction.NORTH))
```

### Using auto() for Automatic Values

```python
class Priority(Enum):
    LOW = auto()
    MEDIUM = auto()
    HIGH = auto()
    CRITICAL = auto()

for priority in Priority:
    print(f"{priority.name} = {priority.value}")
```

### Member Access

```python
class HTTPStatus(Enum):
    OK = 200
    NOT_FOUND = 404
    INTERNAL_ERROR = 500

# By name
print(HTTPStatus['OK'])

# By value
print(HTTPStatus(200))

# Name and value
s = HTTPStatus.NOT_FOUND
print(s.name)
print(s.value)
```

## Beginner Examples

### Iterating Over Enums

```python
class Weekday(Enum):
    MONDAY = 1
    TUESDAY = 2
    WEDNESDAY = 3
    THURSDAY = 4
    FRIDAY = 5
    SATURDAY = 6
    SUNDAY = 7

for day in Weekday:
    print(f"{day.name}: {day.value}")

# List of members
days_list = list(Weekday)
print(days_list)
```

### Enum Comparison

```python
class Status(Enum):
    PENDING = 1
    ACTIVE = 2
    INACTIVE = 3

s1 = Status.PENDING
s2 = Status.PENDING
s3 = Status.ACTIVE

print(s1 is s2)       # True (same member object)
print(s1 == s2)       # True (same member)
print(s1 == s3)       # False
print(s1 == 1)        # False (comparison with non-enum)
print(s1 is Status.PENDING)  # True
```

### Using Enum in Conditional Statements

```python
class OrderStatus(Enum):
    PENDING = "pending"
    CONFIRMED = "confirmed"
    SHIPPED = "shipped"
    DELIVERED = "delivered"
    CANCELLED = "cancelled"

def process_order(status: OrderStatus) -> str:
    if status == OrderStatus.PENDING:
        return "Order is awaiting confirmation"
    elif status == OrderStatus.CONFIRMED:
        return "Order has been confirmed"
    elif status == OrderStatus.SHIPPED:
        return "Order has been shipped"
    elif status == OrderStatus.DELIVERED:
        return "Order has been delivered"
    elif status == OrderStatus.CANCELLED:
        return "Order was cancelled"
    return "Unknown status"

print(process_order(OrderStatus.SHIPPED))
```

### Enum in Dictionaries

```python
class Color(Enum):
    RED = "#FF0000"
    GREEN = "#00FF00"
    BLUE = "#0000FF"

hex_map = {
    Color.RED: (255, 0, 0),
    Color.GREEN: (0, 255, 0),
    Color.BLUE: (0, 0, 255),
}

print(hex_map[Color.RED])
```

## Intermediate Examples

### IntEnum (Integer-compatible Enum)

```python
from enum import IntEnum

class Priority(IntEnum):
    LOW = 1
    MEDIUM = 2
    HIGH = 3

# Can be used where integers are expected
print(Priority.HIGH > Priority.LOW)
print(Priority.HIGH + 1)
print(Priority.LOW * 2)

# Comparison with integers works
print(Priority.HIGH == 3)
print(1 < Priority.MEDIUM < 4)

# Can be used as list indices
items = ["low", "medium", "high"]
print(items[Priority.LOW - 1])
```

### StrEnum (String-compatible Enum, Python 3.11+)

```python
from enum import StrEnum

class Language(StrEnum):
    PYTHON = "Python"
    JAVASCRIPT = "JavaScript"
    RUST = "Rust"
    GO = "Go"

# Auto converts to string
print(Language.PYTHON.upper())
print(f"Language: {Language.RUST}")

# Comparison with strings
print(Language.PYTHON == "Python")
```

### Unique Enum Values

```python
from enum import unique

@unique
class Status(Enum):
    PENDING = 1
    ACTIVE = 2
    INACTIVE = 3
    # DELETED = 1  # ValueError: duplicate values

print(list(Status))
```

### Enum with Methods

```python
class Planet(Enum):
    MERCURY = (3.303e23, 2.4397e6)
    VENUS = (4.869e24, 6.0518e6)
    EARTH = (5.976e24, 6.37814e6)
    MARS = (6.421e23, 3.3972e6)

    def __init__(self, mass: float, radius: float) -> None:
        self._mass = mass
        self._radius = radius

    @property
    def mass(self) -> float:
        return self._mass

    @property
    def radius(self) -> float:
        return self._radius

    @property
    def surface_gravity(self) -> float:
        G = 6.673e-11
        return G * self.mass / (self.radius ** 2)

for planet in Planet:
    print(f"{planet.name}: g = {planet.surface_gravity:.2f} m/s²")
```

### Enum Custom __init__

```python
class Shape(Enum):
    CIRCLE = ("circle", 0)
    SQUARE = ("square", 4)
    TRIANGLE = ("triangle", 3)

    def __init__(self, name: str, sides: int) -> None:
        self.display_name = name
        self.sides = sides

    @property
    def is_polygon(self) -> bool:
        return self.sides >= 3

for shape in Shape:
    print(f"{shape.display_name}: {shape.sides} sides, polygon: {shape.is_polygon}")
```

## Advanced Examples

### Flag (Bitwise Enum)

```python
from enum import Flag, auto

class Permission(Flag):
    NONE = 0
    READ = auto()
    WRITE = auto()
    EXECUTE = auto()
    ADMIN = READ | WRITE | EXECUTE

# Bitwise operations
perm = Permission.READ | Permission.WRITE
print(perm)
print(Permission.READ in perm)   # True
print(Permission.EXECUTE in perm)  # False

perm |= Permission.EXECUTE
print(perm)
print(Permission.ADMIN in perm)  # True (now has all)

perm &= ~Permission.WRITE
print(perm)
```

### Enum with Custom Behavior (Singleton-like)

```python
class DatabaseType(Enum):
    SQLITE = "sqlite"
    POSTGRES = "postgres"
    MYSQL = "mysql"

    def connect(self, **kwargs: Any) -> str:
        if self == DatabaseType.SQLITE:
            return f"sqlite:///{kwargs.get('path', ':memory:')}"
        elif self == DatabaseType.POSTGRES:
            return f"postgresql://{kwargs.get('user')}:{kwargs.get('password')}@{kwargs.get('host')}:{kwargs.get('port', 5432)}/{kwargs.get('db')}"
        elif self == DatabaseType.MYSQL:
            return f"mysql://{kwargs.get('user')}:{kwargs.get('password')}@{kwargs.get('host')}:{kwargs.get('port', 3306)}/{kwargs.get('db')}"
        raise ValueError(f"Unknown database type: {self}")

db_type = DatabaseType.POSTGRES
url = db_type.connect(
    user="admin",
    password="secret",
    host="localhost",
    db="mydb"
)
print(url)
```

### Enum with Class Methods for Lookup

```python
class Country(Enum):
    USA = ("United States", "USD", 840)
    CANADA = ("Canada", "CAD", 124)
    UK = ("United Kingdom", "GBP", 826)
    JAPAN = ("Japan", "JPY", 392)

    def __init__(self, full_name: str, currency: str, numeric_code: int) -> None:
        self.full_name = full_name
        self.currency = currency
        self.numeric_code = numeric_code

    @classmethod
    def from_currency(cls, currency: str) -> 'Country':
        for member in cls:
            if member.currency == currency.upper():
                return member
        raise ValueError(f"No country with currency: {currency}")

    @classmethod
    def from_numeric(cls, code: int) -> 'Country':
        for member in cls:
            if member.numeric_code == code:
                return member
        raise ValueError(f"No country with code: {code}")

print(Country.from_currency("JPY"))
print(Country.from_numeric(840))
```

### Enum Inheriting from Mixins

```python
class ExtendedEnum(Enum):
    @classmethod
    def values(cls) -> list[Any]:
        return [member.value for member in cls]

    @classmethod
    def names(cls) -> list[str]:
        return [member.name for member in cls]

    @classmethod
    def choices(cls) -> list[tuple[str, Any]]:
        return [(member.name, member.value) for member in cls]

class Color(ExtendedEnum):
    RED = 1
    GREEN = 2
    BLUE = 3

print(Color.values())
print(Color.names())
print(Color.choices())
```

### Enum Validation with Metaclass

```python
class ValidatedEnumMeta(EnumMeta):
    def __new__(mcs, name: str, bases: tuple[type, ...], namespace: dict[str, Any]) -> type:
        cls = super().__new__(mcs, name, bases, namespace)
        values_seen: set[Any] = set()
        duplicates = []
        for member in cls:
            v = member.value
            if v in values_seen:
                duplicates.append(v)
            values_seen.add(v)
        if duplicates:
            raise ValueError(
                f"Duplicate values in {name}: {duplicates}"
            )
        return cls

class Month(metaclass=ValidatedEnumMeta):
    JAN = 1
    FEB = 2
    MAR = 3
    # JAN_DUP = 1  # Would raise ValueError
```

### Enum with Property Shortcuts

```python
class FileType(Enum):
    PYTHON = (".py", "text/x-python")
    JAVASCRIPT = (".js", "text/javascript")
    HTML = (".html", "text/html")
    CSS = (".css", "text/css")
    MARKDOWN = (".md", "text/markdown")

    def __new__(cls, extension: str, mime: str) -> 'FileType':
        obj = object.__new__(cls)
        obj._value_ = extension
        obj.extension = extension
        obj.mime_type = mime
        return obj

    def is_code(self) -> bool:
        return self in (FileType.PYTHON, FileType.JAVASCRIPT)

print(FileType.PYTHON.extension)
print(FileType.PYTHON.mime_type)
print(FileType.CSS.extension)
print(FileType.HTML.is_code())
```

### Enum with Factory Method

```python
from typing import Optional

class TemperatureScale(Enum):
    CELSIUS = "C"
    FAHRENHEIT = "F"
    KELVIN = "K"

    def convert(self, value: float, target: 'TemperatureScale') -> float:
        """Convert temperature from this scale to target scale."""
        # Convert to Celsius first
        if self == TemperatureScale.CELSIUS:
            celsius = value
        elif self == TemperatureScale.FAHRENHEIT:
            celsius = (value - 32) * 5 / 9
        elif self == TemperatureScale.KELVIN:
            celsius = value - 273.15
        else:
            raise ValueError(f"Unknown scale: {self}")

        # Convert from Celsius to target
        if target == TemperatureScale.CELSIUS:
            return celsius
        elif target == TemperatureScale.FAHRENHEIT:
            return celsius * 9 / 5 + 32
        elif target == TemperatureScale.KELVIN:
            return celsius + 273.15
        raise ValueError(f"Unknown target scale: {target}")

c = TemperatureScale.CELSIUS
f = TemperatureScale.FAHRENHEIT
print(f"{c.convert(100, f):.1f}F")
```

## Real-World Use Cases

- **HTTP status codes**: Mapping integer codes to descriptive names.
- **Database column types**: Enumerating supported types.
- **Configuration options**: Setting modes (debug, release, test).
- **API response statuses**: Success, error, pending, rate-limited.
- **Event types**: Different kinds of events in an event system.
- **Permission systems**: Bitwise flags for access control.
- **Protocol implementation**: Fixed set of states in a state machine.
- **Menu options**: Commands or actions available to users.

## Common Mistakes

- Using `value` as an enum member name (it's a reserved attribute).
- Comparing enums with plain values: `Color.RED == 1` is `False` for regular `Enum` (use `IntEnum` for integer compatibility).
- Assuming enum members are ordered — they are not by default (use `IntEnum` for ordering).
- Forgetting `@unique` when duplicate values should be prevented.
- Using mutable values (lists, dicts) as enum values — they are shared.
- Forgetting that `auto()` assigns sequential integers starting from 1.
- Trying to subclass an enum that already has members.

## Best Practices

- Use `auto()` for automatic value assignment when the actual value doesn't matter.
- Use `@unique` to prevent duplicate values.
- Use `IntEnum` when the enum needs to behave like an integer (comparisons, arithmetic).
- Use `StrEnum` (Python 3.11+) for string-compatible enums.
- Use `Flag` for bitwise combination of options.
- Add methods to enums for behavior associated with each member.
- Use `member.name` and `member.value` for access, not string parsing.
- Use `is` for singleton comparison (enum members are singletons).

## Interview Questions

1. What is an Enum and why would you use it?
2. How do you create an enum with automatic values?
3. What is the difference between `Enum` and `IntEnum`?
4. What is `Flag` and how do you use bitwise operations on it?
5. How do you iterate over enum members?
6. What does `@unique` do?
7. Can you add methods to an enum class?
8. What is the difference between `member.name` and `member.value`?
9. How does `auto()` assign values?
10. How do enums ensure singleton behavior?

## Coding Challenges

1. **Direction Navigator**: Create an enum for compass directions and a function that returns the opposite direction.
2. **Roman Numerals**: Map Roman numeral strings to integer values using an enum.
3. **HTTP Methods**: Create an enum for HTTP methods (GET, POST, PUT, DELETE, PATCH) with associated metadata.
4. **Traffic Light**: Create a traffic light enum with a method that returns the next color.
5. **Card Suits**: Create a `Suit` enum and a `Card` dataclass using it.
6. **File Permissions**: Implement Unix file permissions using `Flag`.
7. **Order State Machine**: Create a state machine enum with valid transitions.
8. **Unit Converter**: Build an enum of measurement units with conversion methods.

## Summary

Enums provide a type-safe, readable way to define fixed sets of constants in Python. The `enum` module supports standard enums, integer enums (`IntEnum`), string enums (`StrEnum`), and bitwise flags (`Flag`). Enums support methods, iteration, automatic value assignment, and can be made unique with `@unique`. They are essential for clean, maintainable domain modeling.

## Related Topics

- Constants and named constants
- Bitwise operations (Flag)
- Type safety
- Domain-driven design
- State machines
- Configuration management
- Data validation