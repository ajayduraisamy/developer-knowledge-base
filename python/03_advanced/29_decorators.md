# Decorators - @decorator, @wraps, decorators with arguments

## Introduction

A decorator in Python is a function that takes another function (or class) and extends or modifies its behavior without explicitly changing its source code. Decorators are a form of metaprogramming and are applied using the `@decorator_name` syntax, which is syntactic sugar for `function = decorator(function)`. They are callable objects that wrap the target, allowing you to run code before and after the wrapped function executes, inject arguments, modify return values, or even prevent execution entirely.

## Why It Is Important

Decorators promote the DRY (Don't Repeat Yourself) principle by enabling reusable cross-cutting concerns. Common use cases include logging, access control (authentication/authorization), memoization/caching, timing, input validation, rate limiting, and registering functions in frameworks (e.g., Flask routes, pytest fixtures). They keep core business logic clean by separating orthogonal concerns, making code more modular, testable, and maintainable. Every major Python web framework and library, from Flask to Django to Click, relies heavily on decorators.

## Syntax

```python
# Basic decorator structure
def decorator(func):
    def wrapper(*args, **kwargs):
        # pre-processing
        result = func(*args, **kwargs)
        # post-processing
        return result
    return wrapper

@decorator
def target():
    pass

# Equivalent to: target = decorator(target)

# Decorator with arguments
def decorator_with_args(arg1, arg2):
    def actual_decorator(func):
        def wrapper(*args, **kwargs):
            # uses arg1, arg2
            return func(*args, **kwargs)
        return wrapper
    return actual_decorator

@decorator_with_args("a", "b")
def target2():
    pass

# Equivalent to: target2 = decorator_with_args("a", "b")(target2)

# Class-based decorator
class ClassDecorator:
    def __init__(self, func):
        self.func = func
    def __call__(self, *args, **kwargs):
        # pre-processing
        result = self.func(*args, **kwargs)
        # post-processing
        return result
```

## Examples

```python
import functools
import time
import inspect
from typing import Any, Callable, Tuple, Dict
```

### Basic Function Decorator

```python
def timer(func: Callable) -> Callable:
    @functools.wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__} took {elapsed:.6f}s")
        return result
    return wrapper

@timer
def slow_task(n: int) -> int:
    total = 0
    for i in range(n):
        total += i ** 2
    return total

print(slow_task(10_000_000))
```

### Decorator with Arguments

```python
def repeat(count: int = 1) -> Callable:
    def decorator(func: Callable) -> Callable:
        @functools.wraps(func)
        def wrapper(*args: Any, **kwargs: Any) -> Any:
            for _ in range(count):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(count=3)
def greet(name: str) -> None:
    print(f"Hello, {name}!")

greet("Alice")
```

### Using @wraps to Preserve Metadata

```python
def bad_decorator(func: Callable) -> Callable:
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        return func(*args, **kwargs)
    return wrapper

def good_decorator(func: Callable) -> Callable:
    @functools.wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        return func(*args, **kwargs)
    return wrapper

@bad_decorator
def hello_bad() -> str:
    """Returns a greeting."""
    return "Hi"

@good_decorator
def hello_good() -> str:
    """Returns a greeting."""
    return "Hi"

print(hello_bad.__name__)   # "wrapper" -- BAD
print(hello_bad.__doc__)    # None -- BAD
print(hello_good.__name__)  # "hello_good"
print(hello_good.__doc__)   # "Returns a greeting."
```

### Class Decorator

```python
def add_method(cls: type) -> type:
    def new_method(self) -> str:
        return f"Added to {self.__class__.__name__}"
    cls.new_method = new_method
    return cls

@add_method
class MyClass:
    pass

obj = MyClass()
print(obj.new_method())
```

### Class-Based Decorator

```python
class CountCalls:
    def __init__(self, func: Callable) -> None:
        self.func = func
        self.count = 0

    def __call__(self, *args: Any, **kwargs: Any) -> Any:
        self.count += 1
        print(f"Call {self.count} of {self.func.__name__}")
        return self.func(*args, **kwargs)

@CountCalls
def say(message: str) -> None:
    print(message)

say("Hello")
say("World")
```

### Decorator that Returns a Different Type

```python
def bool_returning(func: Callable) -> Callable:
    @functools.wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> bool:
        result = func(*args, **kwargs)
        return bool(result)
    return wrapper

@bool_returning
def get_value() -> int:
    return 42

print(get_value())         # True
print(type(get_value()))   # <class 'bool'>
```

## Beginner Examples

### Simple Logging Decorator

```python
def log_call(func: Callable) -> Callable:
    @functools.wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        print(f"Calling {func.__name__} with {args} {kwargs}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result}")
        return result
    return wrapper

@log_call
def add(a: int, b: int) -> int:
    return a + b

add(3, 5)
```

### Authorization Check

```python
USERS = {"alice": "admin", "bob": "user"}

def require_role(role: str) -> Callable:
    def decorator(func: Callable) -> Callable:
        @functools.wraps(func)
        def wrapper(username: str, *args: Any, **kwargs: Any) -> Any:
            user_role = USERS.get(username, "guest")
            if user_role != role:
                raise PermissionError(f"{username} needs role {role}")
            return func(username, *args, **kwargs)
        return wrapper
    return decorator

@require_role("admin")
def delete_database(username: str) -> str:
    return f"{username} deleted the database"

print(delete_database("alice"))
```

### Retry Decorator

```python
import random

def retry(max_attempts: int = 3, delay: float = 0.1) -> Callable:
    def decorator(func: Callable) -> Callable:
        @functools.wraps(func)
        def wrapper(*args: Any, **kwargs: Any) -> Any:
            last_exception = None
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    last_exception = e
                    print(f"Attempt {attempt} failed: {e}")
                    if attempt < max_attempts:
                        time.sleep(delay)
            raise last_exception  # type: ignore
        return wrapper
    return decorator

@retry(max_attempts=3)
def flaky_api_call() -> str:
    if random.random() < 0.7:
        raise ConnectionError("Network error")
    return "Success"

print(flaky_api_call())
```

## Intermediate Examples

### Decorator that Validates Arguments

```python
def validate_types(**type_hints: type) -> Callable:
    def decorator(func: Callable) -> Callable:
        sig = inspect.signature(func)
        @functools.wraps(func)
        def wrapper(*args: Any, **kwargs: Any) -> Any:
            bound = sig.bind(*args, **kwargs)
            bound.apply_defaults()
            for name, value in bound.arguments.items():
                expected = type_hints.get(name)
                if expected and not isinstance(value, expected):
                    raise TypeError(
                        f"Argument {name} must be {expected.__name__}, "
                        f"got {type(value).__name__}"
                    )
            return func(*args, **kwargs)
        return wrapper
    return decorator

@validate_types(x=int, y=int)
def divide(x: int, y: int) -> float:
    return x / y

print(divide(10, 2))
# divide(10, "2")  # Would raise TypeError
```

### Caching / Memoization Decorator

```python
def memoize(func: Callable) -> Callable:
    cache: Dict[Tuple[Any, ...], Any] = {}
    @functools.wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        key = (args, tuple(sorted(kwargs.items())))
        if key not in cache:
            cache[key] = func(*args, **kwargs)
        return cache[key]
    return wrapper

@memoize
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

print(fibonacci(100))
```

### Decorator with State (Property-like)

```python
def property_decorator(func: Callable) -> property:
    return property(func)

class Circle:
    def __init__(self, radius: float) -> None:
        self._radius = radius

    @property_decorator
    def area(self) -> float:
        return 3.14159 * self._radius ** 2

c = Circle(5)
print(c.area)
```

### Chaining Multiple Decorators

```python
def bold(func: Callable) -> Callable:
    @functools.wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> str:
        return f"<b>{func(*args, **kwargs)}</b>"
    return wrapper

def italic(func: Callable) -> Callable:
    @functools.wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> str:
        return f"<i>{func(*args, **kwargs)}</i>"
    return wrapper

@bold
@italic
def render(text: str) -> str:
    return text

# Equivalent to: render = bold(italic(render))
print(render("Hello"))  # <b><i>Hello</i></b>
```

## Advanced Examples

### Decorator that Adds Methods to a Class

```python
def add_property(name: str, getter: Callable) -> Callable:
    def decorator(cls: type) -> type:
        setattr(cls, name, property(getter))
        return cls
    return decorator

def get_full_name(self) -> str:
    return f"{self.first} {self.last}"

@add_property("full_name", get_full_name)
class Person:
    def __init__(self, first: str, last: str) -> None:
        self.first = first
        self.last = last

p = Person("John", "Doe")
print(p.full_name)
```

### Parameterized Validation Decorator with Error Messages

```python
def validate_range(name: str, minimum: float = 0, maximum: float = 100) -> Callable:
    def decorator(func: Callable) -> Callable:
        @functools.wraps(func)
        def wrapper(*args: Any, **kwargs: Any) -> Any:
            bound_args = inspect.signature(func).bind(*args, **kwargs)
            bound_args.apply_defaults()
            value = bound_args.arguments.get(name)
            if value is not None and not (minimum <= value <= maximum):
                raise ValueError(
                    f"{name}={value} out of range [{minimum}, {maximum}]"
                )
            return func(*args, **kwargs)
        return wrapper
    return decorator

class TemperatureSensor:
    @validate_range("temp", minimum=-50, maximum=150)
    def read(self, temp: float) -> str:
        return f"Temperature: {temp}C"

sensor = TemperatureSensor()
print(sensor.read(25.0))
```

### Decorator that Automatically Registers Functions

```python
PLUGINS: Dict[str, Callable] = {}

def register(func: Callable) -> Callable:
    PLUGINS[func.__name__] = func
    @functools.wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        return func(*args, **kwargs)
    return wrapper

@register
def greet(name: str) -> str:
    return f"Hello, {name}"

@register
def farewell(name: str) -> str:
    return f"Goodbye, {name}"

for name, plugin in PLUGINS.items():
    print(f"{name}: {plugin('Alice')}")
```

### Decorator as a Class with __getattr__ Passthrough

```python
class DelegatingDecorator:
    def __init__(self, func: Callable) -> None:
        self.func = func
        functools.update_wrapper(self, func)

    def __call__(self, *args: Any, **kwargs: Any) -> Any:
        print(f"Delegating: {self.func.__name__}")
        return self.func(*args, **kwargs)

    def __getattr__(self, name: str) -> Any:
        return getattr(self.func, name)

@DelegatingDecorator
def upper(text: str) -> str:
    return text.upper()

print(upper("hello"))
```

### Decorator Using Abstract Base Classes

```python
from abc import ABC, abstractmethod

class AbstractDecorator(ABC):
    def __init__(self, func: Callable) -> None:
        self.func = func

    @abstractmethod
    def __call__(self, *args: Any, **kwargs: Any) -> Any:
        pass

class LoggingDecorator(AbstractDecorator):
    def __call__(self, *args: Any, **kwargs: Any) -> Any:
        print(f"Calling {self.func.__name__}")
        return self.func(*args, **kwargs)

@LoggingDecorator
def compute(x: int) -> int:
    return x * x

print(compute(9))
```

## Real-World Use Cases

- **Flask route registration**: `@app.route('/')` registers a view function with a URL rule.
- **Django view decorators**: `@login_required`, `@csrf_exempt`, `@require_http_methods`.
- **Click CLI framework**: `@click.command()`, `@click.option('--name')`.
- **Pytest fixtures and marks**: `@pytest.fixture`, `@pytest.mark.parametrize`.
- **Celery task definitions**: `@celery.task`.
- **Property decorator**: `@property` for computed attributes.
- **Logging and monitoring**: Automatically log function calls, arguments, and timing.
- **Caching**: `@functools.lru_cache` for memoization.
- **Rate limiting**: Throttle API calls per user/IP.
- **Type checking**: Validate function arguments at runtime.

## Common Mistakes

- Forgetting to use `@functools.wraps` — losing function metadata (name, docstring, signature).
- Applying decorators to a function that returns a different type without considering the impact on callers.
- Nesting decorators in the wrong order — decorators apply bottom-up (closest to the function first).
- Writing a decorator with arguments when a simple decorator is sufficient, adding unnecessary complexity.
- Mutable default arguments inside decorators persisting across calls (e.g., list or dict accumulators).
- Not preserving the original function's signature for tools that inspect it (use `functools.wraps`).

## Best Practices

- Always apply `@functools.wraps` in your wrapper to preserve `__name__`, `__doc__`, `__module__`, and `__dict__`.
- Use `*args` and `**kwargs` in wrappers to accept arbitrary arguments.
- Prefer composing multiple simple decorators over a single complex one.
- When creating a decorator with optional arguments, support using it with or without parentheses: `@deco` and `@deco(args)`.
- Document what the decorator does and whether it changes the function's signature or return type.
- Use class-based decorators when you need to maintain state across calls.
- Keep decorators focused on a single responsibility.

## Interview Questions

1. What is a decorator and how does it work under the hood?
2. What is the difference between `@decorator` and `@decorator()`?
3. Why do we need `functools.wraps`?
4. How do you chain multiple decorators and what is the execution order?
5. Write a decorator that measures execution time.
6. How would you implement a retry decorator with configurable attempts?
7. What's the difference between a function decorator and a class decorator?
8. How can a class be used as a decorator?
9. How do decorators interact with class methods (especially `self`)?
10. Can you write a decorator that accepts both arguments and functions flexibly?

## Coding Challenges

1. **Memoization Decorator**: Implement `@memoize` that caches results based on arguments.
2. **Rate Limiter**: Create `@rate_limit(max_per_minute=N)` that limits function calls.
3. **Deprecation Warning**: Write `@deprecated(replacement="new_func")` that prints a warning.
4. **Singleton**: Implement `@singleton` that ensures a class has only one instance.
5. **Timeout**: Write `@timeout(seconds=5)` that raises an exception if the function takes too long.
6. **Debug Info**: Create `@debug` that prints function name, arguments, and return value.
7. **Memoization with TTL**: Extend `@memoize` to support a time-to-live for cache entries.
8. **Permission Checker**: Implement `@require_permission("admin")` for a mock system.

## Summary

Decorators are a powerful Python feature for modifying or extending functions and classes without altering their source code. They use the `@` syntax and rely on higher-order functions, closures, and the `functools.wraps` utility for proper metadata preservation. Mastery of decorators is essential for writing clean, reusable, and maintainable Python code in both library and application development.

## Related Topics

- Closures
- Higher-order functions
- functools module
- Metaclasses
- Context managers
- Properties and descriptors
- Monkey patching