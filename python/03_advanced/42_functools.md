# Functools - partial, lru_cache, singledispatch, wraps

## Introduction
`functools` is a standard library module for higher-order functions — functions that act on or return other functions. It provides tools for function caching, partial application, and function comparison.

## Why It Is Important
- Reduces boilerplate for common functional patterns
- `lru_cache` and `cache` give memoization with one decorator
- `partial` lets you freeze function arguments
- `singledispatch` enables type-based dispatch without OOP

## Syntax
```python
import functools
```

## Examples

### Beginner Examples

**Partial function application:**
```python
from functools import partial

def power(base, exponent):
    return base ** exponent

square = partial(power, exponent=2)
cube = partial(power, exponent=3)

print(square(5))  # 25
print(cube(5))    # 125
```

**Wraps decorator:**
```python
from functools import wraps

def log_calls(func):
    @wraps(func)  # Preserves func's metadata
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@log_calls
def greet(name):
    """Say hello to someone."""
    return f"Hello {name}"

print(greet.__name__)   # greet (not wrapper)
print(greet.__doc__)    # Say hello to someone.
```

### Intermediate Examples

**Caching with lru_cache:**
```python
from functools import lru_cache
import time

@lru_cache(maxsize=128)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

start = time.time()
print(fibonacci(35))  # 9227465 — instant with cache
print(f"Time: {time.time() - start:.4f}s")
print(fibonacci.cache_info())  # CacheInfo(hits=..., misses=..., ...)
```

**Single dispatch:**
```python
from functools import singledispatch

@singledispatch
def process(data):
    raise TypeError(f"Unsupported type: {type(data)}")

@process.register(int)
def _(data):
    return f"Processing integer: {data * 2}"

@process.register(str)
def _(data):
    return f"Processing string: {data.upper()}"

@process.register(list)
def _(data):
    return f"Processing list: {sum(data)}"

print(process(42))
print(process("hello"))
print(process([1, 2, 3]))
```

### Advanced Examples

**Total ordering:**
```python
from functools import total_ordering

@total_ordering
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def __eq__(self, other):
        if not isinstance(other, Person):
            return NotImplemented
        return self.age == other.age

    def __lt__(self, other):
        if not isinstance(other, Person):
            return NotImplemented
        return self.age < other.age

p1 = Person("Alice", 30)
p2 = Person("Bob", 25)
print(p1 > p2)   # True
print(p1 >= p2)  # True
```

**Cached property:**
```python
from functools import cached_property

class DataLoader:
    def __init__(self, filename):
        self.filename = filename

    @cached_property
    def data(self):
        print(f"Loading {self.filename}...")
        return list(range(1000000))

loader = DataLoader("test.txt")
print(len(loader.data))  # Loads the data
print(len(loader.data))  # Uses cached property — no reload
```

## Real-World Use Cases
- Memoizing expensive database queries
- Building API clients with partial (pre-filling base URL, auth tokens)
- Creating type-specific serializers with singledispatch
- Decorators that preserve function metadata

## Common Mistakes
- Forgetting @wraps in decorators, losing function metadata
- Using lru_cache on functions with unhashable arguments
- Mutating cached mutable return values

## Best Practices
- Set `maxsize=None` on lru_cache only when you understand memory implications
- Use `@cache` (Python 3.9+) as a shorthand for `@lru_cache(maxsize=None)`
- Use `cached_property` for expensive, one-time computed attributes
- Prefer `partial` over lambda when freezing function arguments

## Interview Questions
1. How does lru_cache work under the hood?
2. What's the difference between partial and lambda?
3. How would you implement a simple version of `@wraps`?
4. When would you use singledispatch instead of isinstance checks?

## Coding Challenges
1. **Memoize:** Implement your own `@memoize` decorator.
2. **Compose:** Write a `compose(*funcs)` that chains functions left-to-right.
3. **Pipe:** Create a `pipe(data, *funcs)` that passes data through a pipeline.

## Summary
`functools` provides essential functional programming tools that integrate seamlessly with Python's object system. They reduce boilerplate, improve performance, and make code more declarative.

## Related Topics
- 08_functions.md — functions are the foundation
- 29_decorators.md — wraps is essential for proper decorators
- 37_annotations.md — singledispatch uses type annotations
