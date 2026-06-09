# Type Hints - function annotations, typing module (Python 3.5+)

## Introduction

Type hints, introduced in Python 3.5 via PEP 484, provide a way to annotate function parameters and return values with expected types. Python remains a dynamically-typed language at runtime — type hints are not enforced by the interpreter. Instead, they enable static type checking with tools like `mypy`, `pyright`, and `pytype`, as well as improved IDE support, code documentation, and early error detection.

## Function Annotations

### What It Is

Function annotations allow you to attach type information to function parameters and return values using the `->` syntax and `:` syntax. These annotations are stored in the function's `__annotations__` attribute as a dictionary and have no effect on runtime behavior.

### Why It Is Important

Annotations make code self-documenting by explicitly stating what types a function expects and returns. They enable static type checkers to catch type-related bugs before runtime, improve IDE autocompletion and refactoring support, and serve as machine-checkable documentation that cannot go out of sync with the code.

### How It Works Internally

When Python compiles a function with annotations, it stores them in `func.__annotations__` as a `dict`. Parameter names map to their type expressions, and the special key `'return'` maps to the return annotation. Type expressions are not evaluated at definition time when `from __future__ import annotations` is used (PEP 563), becoming strings instead.

### Syntax

```python
def func(param1: type1, param2: type2) -> return_type:
    ...

# Variable annotations
name: str = "Alice"
count: int
items: list[int] = []
```

### Beginner Examples

```python
def greet(name: str, age: int) -> str:
    return f"{name} is {age} years old."

result = greet("Alice", 30)
print(result)  # Alice is 30 years old.

# Annotations are stored in __annotations__
print(greet.__annotations__)
# {'name': <class 'str'>, 'age': <class 'int'>, 'return': <class 'str'>}

# Variable annotation
user_id: int
user_id = 42  # OK
# user_id = "hello"  # Type checker would warn, but runtime is fine
```

### Intermediate Examples

```python
from typing import Optional, Union, List, Dict, Tuple

# Optional parameter
def find_user(user_id: int) -> Optional[str]:
    users = {1: "Alice", 2: "Bob"}
    return users.get(user_id)

# Union types
def process(value: Union[int, str]) -> str:
    if isinstance(value, int):
        return f"Number: {value}"
    return f"String: {value}"

# Collection types
def total_scores(scores: List[int]) -> int:
    return sum(scores)

def lookup(entries: Dict[str, int], key: str) -> Optional[int]:
    return entries.get(key)

# Tuple with variable length
def process_pairs(pairs: List[Tuple[str, int]]) -> None:
    for name, value in pairs:
        print(f"{name}: {value}")

# Multiple return values
def min_max(items: List[int]) -> Tuple[int, int]:
    return min(items), max(items)
```

### Advanced Examples

```python
from typing import Callable, TypeVar, Generic, Protocol, Any

# Callable types
def apply_twice(func: Callable[[int], int], x: int) -> int:
    return func(func(x))

print(apply_twice(lambda n: n * 2, 5))  # 20

# TypeVar for generics
T = TypeVar('T')
def first(items: List[T]) -> T:
    return items[0]

value: int = first([1, 2, 3])  # T inferred as int

# Protocol (structural subtyping)
class SupportsStr(Protocol):
    def __str__(self) -> str: ...

def to_string(obj: SupportsStr) -> str:
    return str(obj)

# Any - opt out of type checking
def unsafe_func(data: Any) -> Any:
    return data.dangerous_method()

# Type alias
Vector = List[float]
def scale(v: Vector, factor: float) -> Vector:
    return [x * factor for x in v]

# Literal types
from typing import Literal
def set_mode(mode: Literal["read", "write", "append"]) -> None:
    pass

# set_mode("read")    # OK
# set_mode("delete")  # Type checker error
```

### Real-World Use Cases

- **API endpoints**: Document request/response types in web frameworks.
- **Data processing pipelines**: Specify transformations with clear type signatures.
- **Library APIs**: Provide type information for IDE autocompletion.
- **Configuration systems**: Define typed configuration schemas.
- **Dependency injection**: Type-annotated constructors for automatic wiring.

### Common Mistakes

- Annotating but never type-checking (defeats the purpose).
- Using incorrect or overly complex type expressions.
- Forgetting that type hints are not enforced at runtime.
- Mixing `Optional[X]` with `Union[X, None]` inconsistently.
- Annotating with mutable default arguments without understanding the implications.

### Best Practices

- Use type hints consistently across all public functions and methods.
- Start simple (`str`, `int`, `bool`, `list[str]`) and add complexity as needed.
- Use a static type checker (`mypy`, `pyright`) in CI to enforce type correctness.
- Prefer `list[X]` over `List[X]` (Python 3.9+), `dict[K, V]` over `Dict[K, V]`.
- Use `Optional[X]` for `X | None` (Python 3.10+) or `Union[X, None]`.

### Performance Considerations

Type hints have no runtime performance cost in standard Python (they're just stored in `__annotations__`). With `from __future__ import annotations` (PEP 563), annotations become strings evaluated lazily, avoiding the cost of evaluating complex type expressions at definition time.

### Interview Questions

**Q: Are type hints enforced at runtime?**

A: No. Python ignores type hints at runtime. They are used by static type checkers (mypy, pyright) and IDEs. You can still pass any type to an annotated parameter.

**Q: What is `Optional[str]` equivalent to?**

A: `Optional[str]` is equivalent to `Union[str, None]`. It means the value can be either a `str` or `None`.

### Coding Challenges

1. Add type hints to a complex existing function and run mypy on it.
2. Create a generic function that works with any sequence type and returns the first element.
3. Define a Protocol for objects that support addition and implement it with two different classes.

### Related Topics

- `typing` module
- Static type checking
- PEP 484 (Type Hints)
- PEP 563 (Postponed Evaluation of Annotations)
- `dataclasses` (use type hints for field definitions)

## typing Module

### What It Is

The `typing` module, introduced in Python 3.5, provides a standard set of types, utilities, and constructs for type hinting. It includes generic types (`List`, `Dict`, `Set`, `Tuple`), special forms (`Optional`, `Union`, `Literal`), type aliases, `TypeVar` for generics, `Protocol` for structural subtyping, and `Any` for dynamic typing opt-out.

### Why It Is Important

The `typing` module makes type hints expressive enough to describe complex type relationships. Without it, you could only use simple types like `int` or `str`. With it, you can describe generic containers, callable signatures, optional values, discriminated unions, and custom protocols.

### How It Works Internally

Most typing constructs are implemented as special classes or functions that create type objects. When a type checker encounters `List[int]`, it understands this as "a list whose elements are ints." At runtime, `List[int]` is an instance of `typing._GenericAlias` and is not the same as `list`. Starting from Python 3.9, many typing types are aliases for built-in types with subscript support.

### Syntax

```python
from typing import (
    List, Dict, Tuple, Set, Optional, Union,
    Any, Callable, TypeVar, Generic, Protocol, Literal
)
```

### Beginner Examples

```python
from typing import List, Dict, Tuple, Optional, Set

# Basic container types
def process_list(items: List[int]) -> int:
    return sum(items)

def process_dict(data: Dict[str, int]) -> List[str]:
    return [k for k, v in data.items() if v > 0]

def process_tuple(pair: Tuple[str, int]) -> str:
    name, age = pair
    return f"{name} is {age}"

def process_set(values: Set[int]) -> bool:
    return len(values) == len({v * 2 for v in values})

# Optional
def find_key(d: Dict[str, int], key: str) -> Optional[int]:
    return d.get(key)
```

### Intermediate Examples

```python
from typing import Union, Callable, TypeVar, List

# Union types
def handle(value: Union[int, str, List[int]]) -> str:
    if isinstance(value, int):
        return str(value * 2)
    elif isinstance(value, str):
        return value.upper()
    return ", ".join(map(str, value))

# Callable
def execute_handler(
    handler: Callable[[str, int], bool],
    name: str,
    count: int
) -> bool:
    return handler(name, count)

# TypeVar for generics
T = TypeVar('T')
def first(items: List[T]) -> T:
    return items[0]

# Constrained TypeVar
Number = TypeVar('Number', int, float)
def double(value: Number) -> Number:
    return value * 2

# TypeVar with bound
class Animal:
    def speak(self) -> str: ...
    def make_sound(self) -> str:
        return self.speak()

A = TypeVar('A', bound=Animal)
def animal_sound(animal: A) -> A:
    print(animal.speak())
    return animal
```

### Advanced Examples

```python
from typing import (
    Generic, Protocol, TypeVar, List,
    Iterable, Iterator, overload, Literal
)

# Generic classes
class Stack(Generic[T]):
    def __init__(self) -> None:
        self._items: List[T] = []

    def push(self, item: T) -> None:
        self._items.append(item)

    def pop(self) -> T:
        return self._items.pop()

    def peek(self) -> T:
        return self._items[-1]

stack = Stack[int]()
stack.push(1)
stack.push(2)
print(stack.pop())  # 2

# Protocol for structural subtyping
class Sized(Protocol):
    def __len__(self) -> int: ...

def size(obj: Sized) -> int:
    return len(obj)

print(size([1, 2, 3]))  # 3
print(size("hello"))    # 5

# Overload decorator
@overload
def process(data: int) -> str: ...

@overload
def process(data: str) -> int: ...

def process(data):
    if isinstance(data, int):
        return str(data)
    return len(data)

# Literal types
from typing import Literal

def set_status(status: Literal["active", "inactive", "pending"]) -> None:
    print(f"Status set to {status}")

# Type aliases
JSON = Union[str, int, float, bool, None, List['JSON'], Dict[str, 'JSON']]
def parse_json(data: str) -> JSON:
    import json
    return json.loads(data)
```

### Real-World Use Cases

- **Data validation libraries**: Pydantic uses typing constructs for schema definition.
- **Web frameworks**: FastAPI generates OpenAPI specs from type hints.
- **Dependency injection**: Type hints drive auto-wiring in frameworks.
- **Serialization**: Type hints guide JSON/YAML serializers.
- **Configuration management**: Typed configuration objects with validation.

### Common Mistakes

- Using `List` instead of `list` in Python 3.9+ (deprecated but works).
- Importing everything from typing unnecessarily or using deprecated types.
- Using `TypeVar` without constraints when specific types are expected.
- Overusing `Any` (defeats the purpose of type hints).
- Not understanding variance (covariant, contravariant, invariant) in generic types.

### Best Practices

- Use built-in generic types (`list[X]`, `dict[K, V]`) in Python 3.9+ instead of typing equivalents.
- Use `Optional[X]` for potentially-None values (Python 3.10+: `X | None`).
- Define type aliases for complex repeating type expressions.
- Use `Protocol` for duck typing — define what methods an object must have.
- Run a type checker regularly and fix all reported issues.

### Performance Considerations

At runtime, `typing` types have minimal overhead — they create objects once at module import time. With PEP 563 (lazy evaluation), annotation expressions are not evaluated at definition time, eliminating even that cost. Type checking itself is done offline by external tools.

### Interview Questions

**Q: What is the difference between `TypeVar` and `Any`?**

A: `Any` tells the type checker to skip type checking entirely for that value. `TypeVar` creates a generic type variable that preserves type relationships — the same type must be used consistently (e.g., `def first(items: List[T]) -> T` returns the same type as the list elements).

**Q: What is `Protocol` and how does it differ from ABC?**

A: `Protocol` enables structural subtyping (duck typing at the type level). If an object has the methods defined in the Protocol, it satisfies the Protocol, no explicit inheritance needed. ABCs use nominal subtyping — you must explicitly inherit from the ABC.

### Coding Challenges

1. Write a generic `maybe(value: Optional[T], default: T) -> T` function.
2. Create a `Comparable` Protocol and a generic `max_element` function that works with it.
3. Implement a typed `Observable[T]` class using `Generic[T]`.
4. Define a JSON-serializable type alias and validate a function against it.

### Related Topics

- PEP 484 (Type Hints)
- PEP 526 (Variable Annotations)
- PEP 563 (Postponed Evaluation)
- PEP 585 (Type Hinting Generics In Standard Collections)
- `mypy` static type checker
- `dataclasses` and type hints

## Type Hints Benefits

### What It Is

Type hints provide tangible benefits across the software development lifecycle, from early error detection to improved documentation and developer experience. While optional, they have become a standard practice in professional Python development.

### Benefits List

1. **Early error detection**: Static type checkers catch type errors before runtime.
2. **Improved IDE support**: Better autocompletion, inline documentation, and refactoring.
3. **Self-documenting code**: Types serve as always-current documentation.
4. **Safer refactoring**: Type checkers verify that changes don't break type contracts.
5. **Onboarding efficiency**: New developers understand code faster with type annotations.
6. **Reduced unit testing needs**: Some type-related tests become unnecessary.
7. **Better APIs**: Designing types first leads to cleaner interfaces.
8. **Framework integration**: FastAPI, Pydantic, and others use type hints for validation and serialization.

### Common Mistakes

- Adding type hints without running a type checker (wasted effort).
- Making type hints too complex (defeating readability).
- Using `# type: ignore` too liberally instead of fixing type issues.
- Expecting type hints to prevent all bugs (they only catch type-related issues).
- Not using `reveal_type()` in mypy for debugging type inference.

### Best Practices

- Run type checking in CI as a required step.
- Start with strict mode in mypy (`--strict`) for new projects.
- Gradually add type hints to existing codebases, starting with public APIs.
- Use `cast()` sparingly — it means you know better than the type checker.
- Combine type hints with runtime validation (e.g., Pydantic) for defensive programming.

### Interview Questions

**Q: What are the trade-offs of using type hints?**

A: Benefits: earlier bug detection, better documentation, improved IDE support, safer refactoring. Drawbacks: increased verbosity, learning curve, maintenance overhead, false negatives/positives from type checkers.

**Q: How do you handle third-party libraries without type stubs?**

A: Use stub files (`.pyi`), the `typeshed` repository for popular libraries, or `# type: ignore` for imports. Libraries without stubs will be typed as `Any` by default.

### Coding Challenges

1. Take an untyped Python module and add complete type hints.
2. Configure mypy for a project and fix all type errors.
3. Write a type-annotated decorator that preserves the signature of the decorated function.

### Related Topics

- `mypy` static type checker
- `pyright`/`pylance` for VS Code
- `typeshed` (type stubs for the standard library)
- PEP 484 (specification)
- `dataclasses` integration with type hints
