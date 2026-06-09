# Type Hints - function annotations, typing module (Python 3.5+)

## Introduction

Annotations (also called type hints) are a way to attach metadata to function parameters, return values, and variables in Python. Introduced in Python 3.0 and significantly expanded in Python 3.5+ with the `typing` module, annotations provide a standardized syntax for expressing types. They are not enforced at runtime by the Python interpreter but can be used by static type checkers (mypy, pyright), IDEs, and linters to catch errors before runtime.

## Why It Is Important

Type annotations dramatically improve code readability, documentation, and maintainability. They enable static type checking to catch type-related bugs early, provide better IDE support (autocomplete, refactoring, inline documentation), and serve as living documentation. Annotations are increasingly essential in large codebases and team projects, where they reduce cognitive overhead and make APIs self-documenting. Modern Python development strongly encourages type annotations.

## Syntax

```python
# Function annotations
def func(param: type, param2: type = default) -> return_type:
    pass

# Variable annotations (Python 3.6+)
name: str = "Alice"
age: int
items: list[int] = []

# Forward references (string annotation)
def func() -> "MyClass":
    pass

# from __future__ import annotations (Python 3.7+)
# Makes all annotations strings (lazy evaluation)
```

## Examples

```python
from typing import (
    List, Dict, Tuple, Set, Optional, Union, Any,
    TypeVar, Generic, Callable, Iterator, Iterable,
    Literal, Final, NewType, Sequence, Mapping,
    cast, overload, Type, Protocol,
)
from typing import get_type_hints
import inspect
```

### Basic Annotations

```python
def greet(name: str, greeting: str = "Hello") -> str:
    return f"{greeting}, {name}!"

age: int = 30
pi: float = 3.14159
active: bool = True

print(greet("Alice"))
```

### Collection Type Hints

```python
def process_numbers(numbers: list[int]) -> list[int]:
    return [n * 2 for n in numbers]

def lookup(phonebook: dict[str, str], name: str) -> Optional[str]:
    return phonebook.get(name)

def coordinates() -> tuple[float, float, float]:
    return (10.0, 20.0, 30.0)

def unique(items: set[str]) -> list[str]:
    return sorted(set(items))

print(process_numbers([1, 2, 3]))
```

### Optional, Union, and Any

```python
def find_user(user_id: int) -> Optional[str]:
    """Return username or None if not found."""
    users = {1: "Alice", 2: "Bob"}
    return users.get(user_id)

def process(value: Union[int, str]) -> str:
    if isinstance(value, int):
        return f"Number: {value}"
    return f"String: {value}"

def log(message: Any) -> None:
    print(f"[LOG]: {message}")

print(find_user(1))
print(process(42))
```

## Beginner Examples

### Function with Multiple Parameter Types

```python
def multiply(a: int, b: int) -> int:
    return a * b

def concat(a: str, b: str) -> str:
    return a + b

def repeat(item: str, times: int = 1) -> list[str]:
    return [item] * times

print(multiply(3, 4))
print(concat("Hello ", "World"))
print(repeat("Hi", 3))
```

### Variable Annotations

```python
name: str
name = "Alice"

count: int = 0
items: list[str] = []

# This tells type checker 'maybe' is either str or None
maybe: str | None = None  # Python 3.10+ syntax

# Python 3.9+: use built-in generics
numbers: list[int] = [1, 2, 3]
mapping: dict[str, int] = {"a": 1, "b": 2}
```

### Type Aliases

```python
Vector = list[float]
Matrix = list[list[float]]
UserID = int
JSON = dict[str, Any]

def scale(v: Vector, factor: float) -> Vector:
    return [x * factor for x in v]

def create_user(user_id: UserID, data: JSON) -> None:
    print(f"Creating user {user_id}: {data}")

v = [1.0, 2.0, 3.0]
print(scale(v, 2.0))
```

## Intermediate Examples

### TypeVar and Generics

```python
T = TypeVar('T')  # Any type
S = TypeVar('S', int, float)  # Only int or float

def first(items: list[T]) -> Optional[T]:
    if items:
        return items[0]
    return None

def add_all(a: S, b: S) -> S:
    return a + b

print(first([1, 2, 3]))
print(first(["a", "b", "c"]))
print(add_all(10, 20))
print(add_all(3.14, 2.86))
```

### Generic Class

```python
T = TypeVar('T')

class Stack(Generic[T]):
    def __init__(self) -> None:
        self._items: list[T] = []

    def push(self, item: T) -> None:
        self._items.append(item)

    def pop(self) -> T:
        return self._items.pop()

    def peek(self) -> T:
        return self._items[-1]

    def is_empty(self) -> bool:
        return len(self._items) == 0

stack = Stack[int]()
stack.push(1)
stack.push(2)
print(stack.pop())  # 2

string_stack = Stack[str]()
string_stack.push("hello")
print(string_stack.pop())
```

### Callable Type

```python
def apply(func: Callable[[int], int], value: int) -> int:
    return func(value)

def double(x: int) -> int:
    return x * 2

def square(x: int) -> int:
    return x ** 2

print(apply(double, 5))
print(apply(square, 5))

# Callable with more parameters
def operate(a: int, b: int, op: Callable[[int, int], int]) -> int:
    return op(a, b)

print(operate(10, 5, lambda a, b: a + b))
print(operate(10, 5, lambda a, b: a * b))
```

### Literal Type (Python 3.8+)

```python
from typing import Literal

def set_mode(mode: Literal["read", "write", "append"]) -> None:
    print(f"Mode set to {mode}")

def set_http_status(code: Literal[200, 404, 500]) -> str:
    status_text = {200: "OK", 404: "Not Found", 500: "Server Error"}
    return status_text[code]

# set_mode("delete")  # Type error
set_mode("read")
print(set_http_status(200))
```

### Final and NewType

```python
from typing import Final

VERSION: Final[str] = "1.0.0"
MAX_RETRIES: Final[int] = 3

# VERSION = "2.0"  # Type error (Final)

UserId = NewType('UserId', int)
ProductId = NewType('ProductId', int)

def get_user(user_id: UserId) -> str:
    return f"User {user_id}"

def get_product(product_id: ProductId) -> str:
    return f"Product {product_id}"

uid = UserId(42)
pid = ProductId(100)
print(get_user(uid))
# print(get_user(pid))  # Type error (NewType is distinct)
```

## Advanced Examples

### Runtime Access to Annotations

```python
def show(x: int, y: str, z: float = 3.14) -> bool:
    """Sample function with annotations."""
    return True

# Access function annotations at runtime
print(show.__annotations__)

# Using get_type_hints (handles forward references)
print(get_type_hints(show))

# Using inspect
sig = inspect.signature(show)
for name, param in sig.parameters.items():
    print(f"  {name}: {param.annotation} -> default={param.default}")
print(f"  return: {sig.return_annotation}")
```

### Protocol (Structural Subtyping)

```python
class Drawable(Protocol):
    def draw(self) -> str:
        ...

class Circle:
    def draw(self) -> str:
        return "Drawing Circle"

class Square:
    def draw(self) -> str:
        return "Drawing Square"

class NotDrawable:
    def render(self) -> str:
        return "Rendering"

def render_all(objects: list[Drawable]) -> list[str]:
    return [obj.draw() for obj in objects]

# These work because Circle and Square satisfy the Drawable protocol
print(render_all([Circle(), Square()]))
# render_all([NotDrawable()])  # Type error
```

### Overload Decorator

```python
@overload
def process(data: int) -> str:
    ...

@overload
def process(data: str) -> int:
    ...

@overload
def process(data: list[int]) -> float:
    ...

def process(data: Union[int, str, list[int]]) -> Union[str, int, float]:
    if isinstance(data, int):
        return str(data)
    elif isinstance(data, str):
        return len(data)
    else:
        return sum(data) / len(data) if data else 0.0

print(process(42))
print(process("hello"))
print(process([1, 2, 3, 4]))
```

### TypeGuard (Python 3.10+)

```python
from typing import TypeGuard

def is_string_list(val: list[object]) -> TypeGuard[list[str]]:
    return all(isinstance(x, str) for x in val)

def process_strings(items: list[object]) -> None:
    if is_string_list(items):
        # items is narrowed to list[str] here
        for s in items:
            print(s.upper())
    else:
        print("Not all strings")

process_strings(["hello", "world", "python"])
process_strings([1, 2, 3])
```

### Self Type (Python 3.11+)

```python
from typing import Self

class Shape:
    def set_scale(self, factor: float) -> Self:
        self.scale = factor
        return self

class Circle(Shape):
    def __init__(self, radius: float) -> None:
        self.radius = radius
        self.scale = 1.0

    def area(self) -> float:
        return 3.14159 * (self.radius * self.scale) ** 2

circle = Circle(5).set_scale(2.0)
print(circle.area())
```

### ParamSpec and Concatenate (Python 3.10+)

```python
from typing import ParamSpec, Concatenate

P = ParamSpec('P')

def log_calls(func: Callable[P, Any]) -> Callable[P, Any]:
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@log_calls
def add(a: int, b: int) -> int:
    return a + b

print(add(3, 4))
```

### TypedDict

```python
from typing import TypedDict

class PersonDict(TypedDict):
    name: str
    age: int
    email: Optional[str]

def create_person(data: PersonDict) -> PersonDict:
    return {
        "name": data["name"],
        "age": data["age"],
        "email": data.get("email"),
    }

person = create_person({"name": "Alice", "age": 30})
print(person)
```

### Decorator with Preserved Annotations

```python
import functools

def typed_decorator(func: Callable[P, T]) -> Callable[P, T]:
    @functools.wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        return func(*args, **kwargs)
    return wrapper

@typed_decorator
def process(x: int, y: str) -> bool:
    return str(x) == y

print(get_type_hints(process))
```

### Annotations as Metadata for Validation

```python
def validate(func: Callable) -> Callable:
    hints = get_type_hints(func)
    return_type = hints.pop('return', None)

    @functools.wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        bound = inspect.signature(func).bind(*args, **kwargs)
        bound.apply_defaults()
        for name, value in bound.arguments.items():
            expected = hints.get(name)
            if expected and not isinstance(value, expected):
                raise TypeError(
                    f"Argument '{name}' should be {expected.__name__}, "
                    f"got {type(value).__name__}"
                )
        result = func(*args, **kwargs)
        if return_type and not isinstance(result, return_type):
            raise TypeError(
                f"Return should be {return_type.__name__}, "
                f"got {type(result).__name__}"
            )
        return result
    return wrapper

@validate
def divide(a: int, b: int) -> float:
    return a / b

print(divide(10, 3))
```

## Real-World Use Cases

- **Static type checking**: Using mypy, pyright, or pyre to catch type errors before runtime.
- **IDE support**: Autocomplete, inline documentation, refactoring, and error highlighting.
- **API documentation**: Self-documenting function signatures.
- **Data validation**: Using annotations as metadata for runtime validation libraries (pydantic, marshmallow).
- **Dependency injection**: Frameworks that use annotations to resolve dependencies.
- **Serialization**: Automatically generating serialization/deserialization code from type hints.
- **Command-line interfaces**: Using annotations to define CLI arguments (typer, click).
- **Database ORMs**: Mapping Python types to database column types.

## Common Mistakes

- Using `typing.List` instead of `list` (Python 3.9+ supports built-in generics).
- Forgetting `Optional` vs just using `Union[T, None]`.
- Using mutable default arguments with type annotations (still a bug).
- Not importing `from __future__ import annotations` when doing forward references in Python 3.7-3.10.
- Over-annotating with `Any` (defeats the purpose of type hints).
- Confusing `Type[T]` (the class itself) with `T` (an instance of the class).
- Using `Callable` without specifying parameter types.
- Not using `Protocol` when structural subtyping would be more appropriate.

## Best Practices

- Use built-in generics (`list`, `dict`, `set`, `tuple`) instead of `typing` versions in Python 3.9+.
- Prefer `Optional[X]` over `Union[X, None]`.
- Use `TypeVar` for generic functions and classes.
- Use `Protocol` for duck typing (structural subtyping).
- Use `from __future__ import annotations` for lazy evaluation of annotations.
- Run a static type checker (mypy) regularly in CI.
- Annotate all function parameters and return types in public APIs.
- Use `Final` for constants and `Literal` for constrained values.
- Prefer `NewType` over plain type aliases for distinct concepts.
- Use `cast()` sparingly — it should be the exception, not the rule.

## Interview Questions

1. What are type annotations in Python and how are they used?
2. What is the difference between `typing.List[int]` and `list[int]`?
3. What is `Optional` and when should you use it?
4. What is a `TypeVar` and how does it work with generics?
5. What is `Protocol` and how is it different from ABC?
6. How do you access annotations at runtime?
7. What is `Literal` and when would you use it?
8. What is `NewType` and how is it different from a type alias?
9. How does `cast()` work and when should you use it?
10. What is `from __future__ import annotations` and what problem does it solve?

## Coding Challenges

1. **Type Validator**: Write a function that validates arguments at runtime using annotations.
2. **Generic Stack**: Implement a generic `Stack[T]` class with proper type hints.
3. **Typed Dict Validator**: Create a function that validates `TypedDict` data at runtime.
4. **Overloaded Function**: Write an overloaded function that handles `int`, `str`, and `list`.
5. **Protocol Checker**: Implement a structural subtyping check using `Protocol`.
6. **Annotation Extractor**: Write a function that extracts and prints all annotations from a module.
7. **TypeGuard Filter**: Implement a filter function using `TypeGuard`.
8. **Self-returning Builder**: Create a builder pattern class that uses `Self` type.

## Summary

Type annotations provide a standardized way to express types in Python code, enabling static type checking, better IDE support, and self-documenting APIs. The `typing` module provides a rich set of tools including `Optional`, `Union`, `TypeVar`, `Generic`, `Protocol`, `Literal`, and `TypedDict`. While optional, annotations have become a standard practice in modern Python development.

## Related Topics

- typing module
- Static type checking (mypy, pyright)
- Duck typing vs structural typing
- Generic programming
- Function overloading
- Decorator annotations
- Data classes and attrs