# Enums - Enum class, auto(), IntEnum, Flag (Python 3.4+)

## Introduction

Enumerations, or enums, are a way to define a set of symbolic names bound to unique, constant values. Introduced in Python 3.4 via PEP 435, the `Enum` class provides a type-safe way to work with fixed sets of constants. Unlike simple string or integer constants, enums are self-documenting, provide better IDE support, and prevent invalid values from being used.

## Enum Class

### What It Is

`Enum` is the base class for creating enumerated constants in Python. An enum class is created by subclassing `Enum` and defining class attributes as members. Each member has a `name` and a `value`. Enum members are singletons — they are the only instances of the enum class and are compared by identity.

### Why It Is Important

Enums replace error-prone patterns of using string or integer constants. They provide type safety (functions can declare they accept an enum type), better documentation (member names reveal intent), iteration over all members, and protection against invalid values. They also integrate well with type checkers and IDEs.

### How It Works Internally

When Python creates an `Enum` subclass, it uses a metaclass (`EnumMeta`) that collects all class attributes during class creation. Each attribute becomes an enum member, an instance of the enum class. The metaclass ensures that members are unique, sets up `__members__` (an ordered dict of all members), and prevents user instantiation of the enum class.

### Syntax

```python
from enum import Enum

class Color(Enum):
    RED = 1
    GREEN = 2
    BLUE = 3

# Access
Color.RED           # <Color.RED: 1>
Color.RED.name      # 'RED'
Color.RED.value     # 1
```

### Beginner Examples

```python
from enum import Enum

class Direction(Enum):
    NORTH = 1
    SOUTH = 2
    EAST = 3
    WEST = 4

# Accessing members
print(Direction.NORTH)         # Direction.NORTH
print(Direction.NORTH.name)    # NORTH
print(Direction.NORTH.value)   # 1

# Iteration
for d in Direction:
    print(f"{d.name}: {d.value}")
# NORTH: 1, SOUTH: 2, EAST: 3, WEST: 4

# Comparison
print(Direction.NORTH is Direction.NORTH)  # True (singleton)
print(Direction.NORTH == Direction.NORTH)  # True
print(Direction.NORTH == Direction.SOUTH)  # False

# Lookup by name or value
d = Direction['EAST']
print(d)  # Direction.EAST

d = Direction(3)
print(d)  # Direction.EAST
```

### Intermediate Examples

```python
from enum import Enum

class HttpStatus(Enum):
    OK = 200
    CREATED = 201
    BAD_REQUEST = 400
    UNAUTHORIZED = 401
    FORBIDDEN = 403
    NOT_FOUND = 404
    INTERNAL_SERVER_ERROR = 500

    @property
    def is_success(self):
        return 200 <= self.value < 300

    @property
    def is_client_error(self):
        return 400 <= self.value < 500

    @classmethod
    def from_code(cls, code):
        try:
            return cls(code)
        except ValueError:
            return None

status = HttpStatus.OK
print(status.is_success)         # True
print(status.is_client_error)    # False
print(HttpStatus.from_code(404)) # HttpStatus.NOT_FOUND

# Using enums in functions
def handle_response(status: HttpStatus) -> str:
    if status == HttpStatus.OK:
        return "Success"
    elif status.is_client_error:
        return "Client error"
    elif status == HttpStatus.INTERNAL_SERVER_ERROR:
        return "Server error"
    return "Unknown"

print(handle_response(HttpStatus.CREATED))  # Success
```

### Advanced Examples

```python
from enum import Enum, unique

@unique
class Planet(Enum):
    MERCURY = ("Mercury", 0.39, 0.0)
    VENUS = ("Venus", 0.72, 0.0)
    EARTH = ("Earth", 1.0, 1.0)
    MARS = ("Mars", 1.52, 2.0)
    JUPITER = ("Jupiter", 5.2, 67.0)
    SATURN = ("Saturn", 9.54, 62.0)
    URANUS = ("Uranus", 19.19, 27.0)
    NEPTUNE = ("Neptune", 30.07, 14.0)

    def __new__(cls, name_str, au, moons):
        obj = object.__new__(cls)
        obj._name_str = name_str
        obj._au = au
        obj._moons = moons
        return obj

    @property
    def name_str(self):
        return self._name_str

    @property
    def distance_from_sun_au(self):
        return self._au

    @property
    def moon_count(self):
        return self._moons

    def light_travel_time_minutes(self):
        return self._au * 8.317

for planet in Planet:
    print(f"{planet.name_str}: {planet.distance_from_sun_au} AU, "
          f"{planet.moon_count} moons, light time: "
          f"{planet.light_travel_time_minutes():.1f} min")
```

### Real-World Use Cases

- **HTTP status codes**: Self-documenting status code constants with helper methods.
- **Days of week / Months**: Fixed sets in scheduling applications.
- **Order status**: Track order lifecycle states (PENDING, SHIPPED, DELIVERED).
- **User roles**: Define system roles (ADMIN, USER, MODERATOR) with permissions.
- **Configuration modes**: DEBUG, PRODUCTION, TEST modes with associated behaviours.
- **Database field choices**: ORMs like Django use enum classes for field choices.

### Common Mistakes

- Using `class MyEnum(Enum):` without importing `Enum`.
- Expecting enum values to be auto-incremented integers (use `auto()` for that).
- Using `==` with non-enum values (won't match, even if value is the same).
- Assuming enum members are mutable (they are not — enums are immutable by design).
- Forgetting that enum members are singletons — comparing with `is` is safe.

### Best Practices

- Use `Enum` over integer/string constants for type safety and clarity.
- Use `auto()` when only uniqueness matters, not specific values.
- Use `@unique` decorator to ensure no duplicate values.
- Add methods to enums for behaviour associated with each member.
- Use `enum_name.MEMBER` syntax consistently throughout the codebase.

### Performance Considerations

Enum member access is a class attribute lookup — very fast. Enum member comparison uses identity (`is`) and is equivalent to comparing any two Python objects. Enum creation at class definition time has negligible overhead. The `@unique` decorator adds validation at class definition time.

### Interview Questions

**Q: How are enum members compared?**

A: Enum members are singletons — they are compared by identity (`is`) and equality (`==`). Two enum members are equal if they are the same member (same enum class, same name). A member is never equal to its value (e.g., `Color.RED == 1` is `False`).

**Q: Can you add methods to an Enum class?**

A: Yes. Enums are classes and can have methods, properties, dunder methods, and even `__new__` for complex member construction. This is a powerful feature for associating behavior with each enum member.

### Coding Challenges

1. Create an `OrderStatus` enum that tracks state transitions (which states can transition to which).
2. Implement a `Permission` enum with methods that check role-based access.
3. Build an enum-driven state machine for a traffic light system with timing.

### Related Topics

- `auto()` function
- `IntEnum`
- `Flag` enum
- `@unique` decorator
- `EnumMeta` metaclass

## auto() Function

### What It Is

`auto()` is a function that generates unique values for enum members automatically. It delegates to `_generate_next_value_` which, by default, assigns incrementing integers starting from 1. Custom enum classes can override `_generate_next_value_` to use different schemes.

### Why It Is Important

`auto()` eliminates the need to manually assign values to enum members when only uniqueness matters. It reduces boilerplate, prevents duplicate values, and makes adding new members simpler (no need to manage value assignments).

### How It Works Internally

When `auto()` is used as a member value, `EnumMeta` calls `_generate_next_value_(name, start, count, last_values)` during class creation. The default implementation returns `count + 1` (1-indexed). Custom enums can override this class method to implement custom auto-numbering schemes.

### Syntax

```python
from enum import Enum, auto

class Color(Enum):
    RED = auto()
    GREEN = auto()
    BLUE = auto()

# Color.RED.value == 1
# Color.GREEN.value == 2
# Color.BLUE.value == 3
```

### Beginner Examples

```python
from enum import Enum, auto

class Priority(Enum):
    LOW = auto()
    MEDIUM = auto()
    HIGH = auto()
    CRITICAL = auto()

for p in Priority:
    print(f"{p.name}: {p.value}")
# LOW: 1, MEDIUM: 2, HIGH: 3, CRITICAL: 4

# Adding a new priority is easy
class ExtendedPriority(Enum):
    LOW = auto()
    MEDIUM = auto()
    HIGH = auto()
    URGENT = auto()
    CRITICAL = auto()
```

### Intermediate Examples

```python
from enum import Enum, auto

class State(Enum):
    PENDING = auto()
    PROCESSING = auto()
    COMPLETED = auto()
    FAILED = auto()

    def __str__(self):
        return self.name.lower()

    def can_transition_to(self, new_state):
        transitions = {
            State.PENDING: {State.PROCESSING, State.FAILED},
            State.PROCESSING: {State.COMPLETED, State.FAILED},
            State.COMPLETED: set(),
            State.FAILED: {State.PENDING},
        }
        return new_state in transitions[self]

state = State.PENDING
new_state = State.PROCESSING
print(state.can_transition_to(new_state))  # True
```

### Advanced Examples

```python
from enum import Enum, auto

class AutoName(Enum):
    def _generate_next_value_(name, start, count, last_values):
        return name.lower()

class ApiEndpoint(AutoName):
    USERS = auto()
    PRODUCTS = auto()
    ORDERS = auto()
    AUTH = auto()

print(ApiEndpoint.USERS.value)     # 'users'
print(ApiEndpoint.PRODUCTS.value)  # 'products'

# Custom auto with formatted values
class FormattedAuto(Enum):
    def _generate_next_value_(name, start, count, last_values):
        return f"{name}_{count + 1}"

class TransactionStatus(FormattedAuto):
    CREATED = auto()
    PENDING = auto()
    SETTLED = auto()
    CANCELLED = auto()

print(TransactionStatus.CREATED.value)   # 'CREATED_1'
print(TransactionStatus.SETTLED.value)   # 'SETTLED_3'
```

### Real-World Use Cases

- **Simple state machines**: When only state uniqueness matters, not specific values.
- **Categories/Tags**: Grouping-related constants where values are irrelevant.
- **Event types**: Event-driven systems where event names are the key identifier.
- **Log levels**: When you need ordered but irrelevant actual values.
- **Options/Flags**: When adding new options frequently (auto handles numbering).

### Common Mistakes

- Mixing `auto()` with explicit values (works but defeats auto's purpose).
- Expecting `auto()` to start at 0 (it starts at 1 by default).
- Using `auto()` with `IntEnum` and expecting specific integer values.
- Forgetting that `auto()` values are determined at class definition time, not runtime.

### Best Practices

- Use `auto()` for enum members when only uniqueness is required.
- Override `_generate_next_value_` for custom auto-value schemes.
- Use `auto()` with `IntEnum` for integer-compatible enums.
- Document the auto-numbering scheme if overriding `_generate_next_value_`.

### Performance Considerations

`auto()` resolves at class definition time with no runtime overhead. Custom `_generate_next_value_` methods add a small amount of class-creation time but no per-access cost.

### Interview Questions

**Q: What values does `auto()` generate by default?**

A: By default, `auto()` generates incrementing integers starting at 1. The first member gets 1, the second gets 2, etc. The implementation is `_generate_next_value_` which returns `len(class.__members__) + 1`.

**Q: How can you customize `auto()` value generation?**

A: Override `_generate_next_value_` as a `@staticmethod` on your enum class. It receives `name` (member name), `start` (initial value), `count` (how many auto values so far), and `last_values` (list of previous auto values).

### Coding Challenges

1. Create an enum using `auto()` that generates string values matching member names (lowercased).
2. Implement an enum that uses `auto()` with a custom counter starting at 100 and incrementing by 10.
3. Build a priority enum where `auto()` values represent a Fibonacci sequence.

### Related Topics

- `Enum` class
- `_generate_next_value_` method
- `IntEnum` (integer-compatible enum)
- `@unique` decorator (prevent duplicate values)

## IntEnum

### What It Is

`IntEnum` is an enum class whose members are also subclasses of `int`. This means IntEnum members can be used anywhere integers are expected, including in arithmetic operations, as list indices, and for comparison with plain integers.

### Why It Is Important

`IntEnum` bridges the gap between enum type safety and integer compatibility. It's useful for replacing integer constants (like HTTP status codes or exit codes) while maintaining backward compatibility with code that expects plain integers. It also allows integer-specific operations like bitwise operations.

### How It Works Internally

`IntEnum` inherits from both `int` and `Enum`. The metaclass creates members that are actual `int` instances, so `isinstance(Status.OK, int)` is `True`. This allows IntEnum values to be used in arithmetic, as dict keys with plain integers, and in any context requiring an integer.

### Syntax

```python
from enum import IntEnum

class Priority(IntEnum):
    LOW = 1
    MEDIUM = 2
    HIGH = 3

assert Priority.LOW == 1    # True (unlike regular Enum)
```

### Beginner Examples

```python
from enum import IntEnum

class ExitCode(IntEnum):
    SUCCESS = 0
    GENERAL_ERROR = 1
    USAGE_ERROR = 2

# Works with integers
print(ExitCode.SUCCESS == 0)  # True (unlike regular Enum)

# Can be used in arithmetic
print(ExitCode.GENERAL_ERROR + ExitCode.GENERAL_ERROR)  # 2

# Can be used with built-in functions
numbers = [ExitCode.SUCCESS, ExitCode.GENERAL_ERROR, ExitCode.USAGE_ERROR]
print(min(numbers))  # ExitCode.SUCCESS
print(max(numbers))  # ExitCode.USAGE_ERROR

# Can be used as list indices
items = ["zero", "one", "two"]
print(items[ExitCode.SUCCESS])  # zero
```

### Intermediate Examples

```python
from enum import IntEnum

class LogLevel(IntEnum):
    DEBUG = 10
    INFO = 20
    WARNING = 30
    ERROR = 40
    CRITICAL = 50

logger_level = LogLevel.INFO

def log(level: LogLevel, message: str):
    if level >= logger_level:
        print(f"[{level.name}] {message}")

log(LogLevel.DEBUG, "debug info")     # Not printed (level < INFO)
log(LogLevel.WARNING, "warning msg")   # Printed
log(LogLevel.ERROR, "error msg")      # Printed

# Comparison with plain integers works
if LogLevel.ERROR >= 40:
    print("Yes, ERROR >= 40")  # True
```

### Advanced Examples

```python
from enum import IntEnum

class Permissions(IntEnum):
    NONE = 0
    READ = 4
    WRITE = 2
    EXECUTE = 1
    ALL = READ | WRITE | EXECUTE  # 7

class FilePermissions(IntEnum):
    OWNER_READ = 0o400
    OWNER_WRITE = 0o200
    OWNER_EXEC = 0o100
    GROUP_READ = 0o040
    GROUP_WRITE = 0o020
    GROUP_EXEC = 0o010

# Bitwise operations work because they're integers
current = Permissions.READ | Permissions.WRITE
print(current)             # 6
print(Permissions(current))  # <Permissions.ALL? No, ALL is 7

# Check permission
has_write = bool(current & Permissions.WRITE)
has_execute = bool(current & Permissions.EXECUTE)
print(f"Write: {has_write}, Execute: {has_execute}")  # Write: True, Execute: False
```

### Real-World Use Cases

- **HTTP status codes**: Backward-compatible with integer handling in web frameworks.
- **Exit codes**: System exit codes that need comparison with integer standards.
- **Bitmask flags**: Combine multiple flags using bitwise OR (use `Flag` or `IntFlag` for better support).
- **Log levels**: Numeric log levels that can be compared with `>=`.
- **Protocol constants**: Network protocol constants that must match integer specifications.

### Common Mistakes

- Using `IntEnum` when a regular `Enum` would suffice (only use when integer compatibility is needed).
- Forgetting that `IntEnum` members are `int` instances and can be used in unintended arithmetic.
- Using `IntEnum` for bitmask operations without understanding the values (use `Flag` instead).
- Passing `IntEnum` to functions that modify the integer (creates plain `int`, not enum).

### Best Practices

- Use `IntEnum` only when you need integer compatibility for existing code or protocols.
- Prefer regular `Enum` for most new code to maintain type safety.
- Use `Flag` or `IntFlag` for bitmask operations instead of manual bitwise `IntEnum`.
- Document that the enum values are integers and must match specific numeric specifications.

### Performance Considerations

`IntEnum` members are plain `int` instances with no performance overhead for integer operations. Enum operations (member access, name/value lookup) are as fast as regular `Enum`.

### Interview Questions

**Q: How is `IntEnum` different from regular `Enum`?**

A: `IntEnum` members are also `int` instances, making them compatible with integers in comparisons, arithmetic, and as sequence indices. Regular `Enum` members are not comparable to their values — `Color.RED == 1` is `False`.

**Q: What is the risk of using `IntEnum`?**

A: `IntEnum` members can be compared to and used as plain integers, which means they can be passed to `int`-typed parameters anywhere. This weakens the type safety that enums provide and can lead to accidentally passing enum members where integers are expected.

### Coding Challenges

1. Create an `IntEnum` for Unix process signals (SIGTERM=15, SIGKILL=9, etc.) with appropriate descriptions.
2. Implement a `HttpStatus` `IntEnum` with methods for checking response categories.
3. Build a simple logging system that uses `LogLevel(IntEnum)` for filtering.

### Related Topics

- `Enum` class
- `IntFlag` (for integer-compatible bitmask operations)
- `Flag` (for bitmask operations without integer compatibility)
- `auto()` with `IntEnum`
- `@unique` decorator

## Flag

### What It Is

`Flag` is an enum class designed for bitmask operations. Members should have values that are powers of two (1, 2, 4, 8, ...), allowing combination via bitwise OR (`|`) and testing via bitwise AND (`&`). `Flag` supports the standard bitwise operators and provides `__contains__` for membership testing.

### Why It Is Important

`Flag` provides a type-safe way to work with combinations of options. Instead of passing multiple boolean parameters or integer bitmasks, you can pass a single `Flag` value representing the set of enabled options. It is more readable, type-safe, and self-documenting than raw integer bitmasks.

### How It Works Internally

`Flag` is a subclass of `Enum`. Its metaclass (`FlagMeta`) ensures that member values are powers of two. Combined values (from OR operations) are `Flag` instances. The `__contains__` method checks if a flag's bits are all set in the combined flags. Aliases (combinations) are automatically tracked.

### Syntax

```python
from enum import Flag, auto

class Permission(Flag):
    READ = auto()
    WRITE = auto()
    EXECUTE = auto()

# Combine flags
perm = Permission.READ | Permission.WRITE

# Check flags
Permission.READ in perm  # True
Permission.EXECUTE in perm  # False
```

### Beginner Examples

```python
from enum import Flag, auto

class Color(Flag):
    RED = auto()
    GREEN = auto()
    BLUE = auto()
    WHITE = RED | GREEN | BLUE
    YELLOW = RED | GREEN
    CYAN = GREEN | BLUE
    MAGENTA = RED | BLUE

# Combine
purple = Color.RED | Color.BLUE
print(purple)  # Color.MAGENTA (or Color.RED|BLUE)

# Check membership
print(Color.RED in purple)    # True
print(Color.GREEN in purple)  # False

# Iterate over individual flags in a combined value
for c in Color:
    if c in purple and c.name.isupper():
        print(c.name)  # RED, BLUE
```

### Intermediate Examples

```python
from enum import Flag, auto

class Access(Flag):
    NONE = 0
    READ = auto()
    WRITE = auto()
    DELETE = auto()
    ADMIN = READ | WRITE | DELETE

class Document:
    def __init__(self, title, content):
        self.title = title
        self.content = content
        self._access_control = {}

    def set_access(self, user, access: Access):
        self._access_control[user] = access

    def can(self, user, access: Access) -> bool:
        user_access = self._access_control.get(user, Access.NONE)
        return access in user_access

doc = Document("Report", "Confidential data")
doc.set_access("alice", Access.READ | Access.WRITE)
doc.set_access("bob", Access.READ)
doc.set_access("charlie", Access.ADMIN)

print(doc.can("alice", Access.WRITE))  # True
print(doc.can("bob", Access.DELETE))   # False
print(doc.can("charlie", Access.ADMIN))  # True
```

### Advanced Examples

```python
from enum import Flag, auto

class Style(Flag):
    BOLD = auto()
    ITALIC = auto()
    UNDERLINE = auto()
    STRIKETHROUGH = auto()
    MONOSPACE = auto()

class TextFormatter:
    def __init__(self):
        self.default_styles = Style.BOLD | Style.MONOSPACE

    def format(self, text: str, styles: Style) -> str:
        result = text
        if Style.BOLD in styles:
            result = f"**{result}**"
        if Style.ITALIC in styles:
            result = f"*{result}*"
        if Style.UNDERLINE in styles:
            result = f"_{result}_"
        if Style.STRIKETHROUGH in styles:
            result = f"~~{result}~~"
        return result

    def format_with_defaults(self, text: str, extra: Style = Style.NONE) -> str:
        return self.format(text, self.default_styles | extra)

fmt = TextFormatter()
print(fmt.format("Hello", Style.BOLD | Style.ITALIC))
# **Hello** with *...* (order depends on implementation)

print(fmt.format_with_defaults("Code", Style.UNDERLINE))
# **Code** with _..._ and fixed-width
```

### Real-World Use Cases

- **User permissions**: Combine READ, WRITE, DELETE, ADMIN flags for access control.
- **File system attributes**: Represent file flags (HIDDEN, READ_ONLY, SYSTEM, ARCHIVE).
- **UI component states**: Combine VISIBLE, ENABLED, FOCUSED, SELECTED flags.
- **Network socket options**: Represent socket flags (NONBLOCK, REUSEADDR, KEEPALIVE).
- **Configuration flags**: Enable/disable feature combinations.

### Common Mistakes

- Using non-power-of-2 values (combinations won't work correctly — use `auto()` to avoid this).
- Forgetting `NONE = 0` for an empty flag set (without it, you can't represent "no flags").
- Using `Flag` when only one flag can be set at a time (use regular `Enum` instead).
- Expecting `Flag` members to be comparable to integers (they are not — use `IntFlag` for that).

### Best Practices

- Use `auto()` to ensure values are powers of two.
- Define `NONE = 0` for the empty flag set.
- Define named combinations for commonly used flag sets (e.g., `ALL`, `NONE`, `READ_WRITE`).
- Use `Flag` over `IntFlag` when integer compatibility is not needed.
- Use `x in y` syntax for checking if a flag is set (`Flag.__contains__`).

### Performance Considerations

`Flag` operations are bitwise integer operations in C — very fast. The `__contains__` check is a single bitwise AND. Named combinations are computed at class definition time. `auto()` for flags resolves at class definition time.

### Interview Questions

**Q: How is `Flag` different from `IntFlag`?**

A: `Flag` does not inherit from `int`, so flag values are not directly comparable to integers. `IntFlag` does inherit from `int`, making it compatible with integer bitwise operations and comparisons. Use `Flag` for type-safe flags, `IntFlag` for backward compatibility with integer-based code.

**Q: How do you check if a specific flag is set in a combined value?**

A: Use the `in` operator: `Flag.READ in combined_flags`. This is the `__contains__` method which checks if all bits of the tested flag are set in the combined value.

### Coding Challenges

1. Implement a `FilePermission` flag with OWNER, GROUP, OTHER and READ, WRITE, EXECUTE access levels.
2. Create a notification system that uses `Flag` for notification types (EMAIL, SMS, PUSH, IN_APP).
3. Build an `OrderOption` flag (RUSH, GIFT_WRAP, INSURED, PRIORITY) and a method to calculate total cost.
4. Implement a feature toggle system using `Flag` for managing feature flags.

### Related Topics

- `IntFlag` (integer-compatible flags)
- `auto()` for automatic power-of-2 values
- `Enum` base class
- Bitwise operations in Python
- `@unique` decorator
