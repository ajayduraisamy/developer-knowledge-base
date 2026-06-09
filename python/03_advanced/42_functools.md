# Functools - partial, lru_cache, singledispatch, wraps

## Introduction

The `functools` module provides higher-order functions that act on or return other functions. It is a collection of tools for functional programming in Python, enabling partial application, memoization, single-dispatch generic functions, and wrapper function metadata preservation. These tools help write cleaner, more modular, and more efficient code.

## partial

### What It Is

`functools.partial` is a function that takes a callable and some arguments (positional and/or keyword) and returns a new callable with those arguments pre-filled. It creates a partial application of the original function, where some arguments are already supplied.

### Why It Is Important

`partial` reduces code duplication by specializing general functions. Instead of writing wrapper functions that hard-code certain arguments, you can create partials that bind those arguments. This is particularly useful for callbacks, event handlers, and interfaces that require a specific function signature.

### How It Works Internally

`partial` returns a `functools.partial` object that stores the original function and the pre-filled arguments. When called, it merges the stored arguments with any new arguments. If more arguments are provided than the partial expects, they are appended. The `partial` object has `func`, `args`, and `keywords` attributes for introspection.

### Syntax

```python
from functools import partial

new_func = partial(original_func, arg1, arg2, kwarg1=value1)
```

### Beginner Examples

```python
from functools import partial

# Basic partial application
def power(base, exponent):
    return base ** exponent

square = partial(power, exponent=2)
cube = partial(power, exponent=3)

print(square(5))   # 25
print(cube(5))     # 125

# Partial with positional args
def greet(greeting, name):
    return f"{greeting}, {name}!"

say_hello = partial(greet, "Hello")
say_goodbye = partial(greet, "Goodbye")

print(say_hello("Alice"))    # Hello, Alice!
print(say_goodbye("Bob"))    # Goodbye, Bob!
```

### Intermediate Examples

```python
from functools import partial
import os

# Creating specialized functions
base_dir = "/var/log"
list_logs = partial(os.listdir, base_dir)
# Now: list_logs() == os.listdir("/var/log")

# Function that prints to a specific stream
import sys
print_err = partial(print, file=sys.stderr)
print_err("This is an error message")  # Prints to stderr

# Sorting with a pre-configured key
from operator import attrgetter
sort_by_name = partial(sorted, key=attrgetter('name'))

class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

people = [Person("Charlie", 30), Person("Alice", 25), Person("Bob", 35)]
sorted_people = sort_by_name(people)
print([p.name for p in sorted_people])  # ['Alice', 'Bob', 'Charlie']

# Partial with map
from decimal import Decimal
format_currency = partial("{:.2f}".format)
prices = [10.5, 20.0, 30.75]
formatted = list(map(format_currency, prices))
print(formatted)  # ['10.50', '20.00', '30.75']
```

### Advanced Examples

```python
from functools import partial

# Callback specialization
class Button:
    def __init__(self, label):
        self.label = label
        self._callbacks = []

    def on_click(self, callback):
        self._callbacks.append(callback)

    def click(self, event_data=None):
        for cb in self._callbacks:
            cb(self, event_data)

def log_button_click(message, button, event_data):
    print(f"{message}: {button.label} clicked ({event_data})")

save_btn = Button("Save")
cancel_btn = Button("Cancel")

save_btn.on_click(partial(log_button_click, "User saved document"))
cancel_btn.on_click(partial(log_button_click, "User cancelled"))

save_btn.click({"ctrl": True})   # User saved document: Save clicked ({'ctrl': True})
cancel_btn.click({})             # User cancelled: Cancel clicked ({})

# Partial for API client configuration
class APIClient:
    def __init__(self, base_url):
        self.base_url = base_url

    def request(self, method, endpoint, **kwargs):
        url = f"{self.base_url}/{endpoint}"
        print(f"{method} {url}")
        return {"status": 200, "method": method, "endpoint": endpoint}

    def get(self, endpoint, **kwargs):
        return self.request("GET", endpoint, **kwargs)

    def post(self, endpoint, **kwargs):
        return self.request("POST", endpoint, **kwargs)

api = APIClient("https://api.example.com")
get_users = partial(api.get, "/users")
create_user = partial(api.post, "/users")

print(get_users(params={"role": "admin"}))  # GET https://api.example.com/users
print(create_user(json={"name": "Alice"}))  # POST https://api.example.com/users
```

### Real-World Use Cases

- **Callback specialization**: Bind arguments to event handlers.
- **API client methods**: Create specialized HTTP method functions.
- **Configuration**: Pre-configure logging formatters or database connections.
- **Functional transformations**: Create specialized `map`/`filter` functions.
- **Test fixtures**: Create reusable test helpers with pre-configured arguments.
- **Thread pool workers**: Pre-configure worker functions for `concurrent.futures`.

### Common Mistakes

- Using `partial` when keyword arguments would suffice (simpler is better).
- Expecting `partial` to work with methods as the first argument (use `partial(method, instance)` or bound methods).
- Creating partials with mutable default arguments (captured by reference, not value).
- Forgetting that `partial` returns a callable, not the result.
- Using `partial` inside loops defined with lambda (closure issues — use `functools.partial` to avoid late binding).

### Best Practices

- Use `partial` over lambda for simple argument binding (better introspection).
- Use keyword arguments with `partial` for clarity: `partial(func, arg=value)`.
- Use `partial` to simplify function signatures for callbacks and event handlers.
- Combine `partial` with `map` and `filter` for functional-style programming.
- Use `partial.method` for method specialization: `partial(MyClass.method, instance)`.

### Performance Considerations

`partial` creates a new object at call time (negligible overhead). Calling a partial is slightly slower than calling the original function directly due to argument merging. For performance-critical code, consider writing a dedicated function instead of using `partial`.

### Interview Questions

**Q: What is the difference between `partial` and a lambda?**

A: `partial` provides better introspection — it has `func`, `args`, and `keywords` attributes. Lambdas are anonymous and harder to debug. `partial` also handles keyword arguments more cleanly. However, lambdas can contain arbitrary expressions, while `partial` only binds arguments.

**Q: How does `partial` handle additional arguments at call time?**

A: Additional positional arguments are appended to the ones stored in the partial. Additional keyword arguments are merged (call-time kwargs override partial kwargs if there's a conflict).

### Coding Challenges

1. Use `partial` to create a `multiply_by(n)` function factory.
2. Create a logging system where different log levels are partials of a base logger function.
3. Implement a simple RPC client using `partial` to create remote method callers.

### Related Topics

- `partialmethod` (for method partials)
- `wraps` (for preserving function metadata)
- Lambda functions (alternative to partial)
- `functools.reduce` (another functional tool)

## lru_cache

### What It Is

`functools.lru_cache` is a decorator that caches the results of function calls using a least-recently-used (LRU) eviction strategy. It stores return values based on the arguments passed to the function, returning cached results when the same arguments are seen again.

### Why It Is Important

`lru_cache` provides automatic memoization with minimal code changes. It can dramatically improve performance for expensive, pure functions that are called repeatedly with the same arguments. The LRU eviction prevents unbounded memory growth by discarding the least recently used entries when the cache is full.

### How It Works Internally

`lru_cache` uses a dictionary (for O(1) lookup) combined with a doubly-linked list (for tracking access order). When a function is called, the arguments are hashed and used as a key. If found, the cached result is returned and the entry is moved to the front of the LRU list. If not found, the function is called, the result is cached, and if the cache exceeds `maxsize`, the least recently used entry is evicted.

### Syntax

```python
from functools import lru_cache

@lru_cache(maxsize=128, typed=False)
def expensive_function(args):
    ...
```

### Beginner Examples

```python
from functools import lru_cache

# Fibonacci with memoization (exponential -> linear)
@lru_cache(maxsize=None)  # No limit
def fib(n):
    if n < 2:
        return n
    return fib(n - 1) + fib(n - 2)

print(fib(100))  # 354224848179261915075 (fast, even for n=100)

# Without caching (would be incredibly slow)
def fib_no_cache(n):
    if n < 2:
        return n
    return fib_no_cache(n - 1) + fib_no_cache(n - 2)

# Cache introspection
print(fib.cache_info())
# CacheInfo(hits=98, misses=101, maxsize=None, currsize=101)
```

### Intermediate Examples

```python
from functools import lru_cache
import time

# Memoizing an expensive computation
@lru_cache(maxsize=32)
def compute_factorization(n):
    """Expensive prime factorization."""
    result = []
    d = 2
    while d * d <= n:
        while n % d == 0:
            result.append(d)
            n //= d
        d += 1
    if n > 1:
        result.append(n)
    return result

# Time comparison
start = time.time()
r1 = compute_factorization(123456789)
t1 = time.time() - start

start = time.time()
r2 = compute_factorization(123456789)  # Cached
t2 = time.time() - start

print(f"First call: {t1:.4f}s, Second call: {t2:.6f}s")
print(f"Called {r1}")

# Cache info
print(compute_factorization.cache_info())

# Clearing the cache
compute_factorization.cache_clear()
print(compute_factorization.cache_info())  # All zeros
```

### Advanced Examples

```python
from functools import lru_cache

# Dynamic programming with lru_cache
@lru_cache(maxsize=None)
def edit_distance(s1, s2):
    """Compute Levenshtein distance between two strings."""
    if not s1:
        return len(s2)
    if not s2:
        return len(s1)
    if s1[0] == s2[0]:
        return edit_distance(s1[1:], s2[1:])
    return 1 + min(
        edit_distance(s1[1:], s2),       # Delete
        edit_distance(s1, s2[1:]),       # Insert
        edit_distance(s1[1:], s2[1:])    # Replace
    )

print(edit_distance("kitten", "sitting"))  # 3

# Typed caching (distinguishes 1 and 1.0)
@lru_cache(maxsize=128, typed=True)
def cached_function(x):
    return type(x).__name__

class Grid:
    @lru_cache(maxsize=None)
    def get_cell(self, x, y):
        """Compute cell value (expensive)."""
        return (x * 1000 + y)  # Dummy computation

grid = Grid()
print(grid.get_cell(1, 2))  # Computed
print(grid.get_cell(1, 2))  # Cached (hit)

# Cache size management
@lru_cache(maxsize=1000)
def expensive_api_call(query, page=1):
    """Simulated API call."""
    import time
    time.sleep(0.1)
    return {"query": query, "page": page, "results": []}

# First 500 calls are computed, then cached hits
for i in range(1500):
    expensive_api_call(f"query_{i % 500}", page=1)

print(expensive_api_call.cache_info())
# CacheInfo(hits=1000, misses=500, maxsize=1000, currsize=500)
```

### Real-World Use Cases

- **Dynamic programming**: Memoize recursive solutions to subproblems.
- **API response caching**: Cache responses to external API calls (with `maxsize` to control memory).
- **Configuration parsing**: Cache parsed configuration files.
- **Resource-intensive computations**: Cache results of complex calculations.
- **Database query results**: Cache query results for frequently accessed data.
- **Parsing/compliation**: Cache parsed expressions or compiled patterns.

### Common Mistakes

- Using `lru_cache` on functions with mutable arguments (lists, dicts) — they're not hashable.
- Using `lru_cache` on functions with side effects (results should be deterministic).
- Forgetting that `maxsize=None` can lead to unbounded memory growth.
- Using `lru_cache` on functions that return mutable objects (the cached object is shared).
- Applying `lru_cache` to methods without understanding `self` is part of the cache key (separate cache per instance).

### Best Practices

- Use `maxsize=None` only when you're certain the argument space is limited.
- Use `typed=True` when you need to distinguish between `1` and `1.0`, `True` and `1`.
- Use `cache_clear()` for cache invalidation when the underlying data changes.
- Use `cache_info()` for monitoring cache effectiveness.
- Apply `lru_cache` only to pure functions (same args always produce same result, no side effects).

### Performance Considerations

The cache lookup is O(1) with a very fast hash lookup. The overhead for a cache miss is one dict insertion and (if at maxsize) one LRU eviction. For functions that take very little time to compute (microseconds), the caching overhead may outweigh the benefit. Profile before and after to ensure caching improves performance.

### Interview Questions

**Q: What is the difference between `lru_cache` and `cache` (Python 3.9+)?**

A: `functools.cache` is a shorthand for `lru_cache(maxsize=None)` — unbounded cache with no LRU eviction. Use `cache` when the argument space is small and bounded; use `lru_cache` with a specific `maxsize` for memory-constrained scenarios.

**Q: How does `lru_cache` handle keyword arguments?**

A: Keyword arguments are converted to a canonical form and hashed as part of the cache key. The order of keyword arguments doesn't matter — `f(a=1, b=2)` and `f(b=2, a=1)` map to the same cache entry.

### Coding Challenges

1. Use `lru_cache` to implement a memoized version of the "minimum cost to reach the top of stairs" dynamic programming problem.
2. Implement a cache for an API client with a TTL (time-to-live) check alongside `lru_cache`.
3. Use `lru_cache` to optimize a recursive function that computes binomial coefficients.
4. Build a throttled memoizer that limits how often a function can be called, combining `lru_cache` with a rate limiter.

### Related Topics

- `functools.cache` (unbounded cache, Python 3.9+)
- `functools.cached_property` (lazy property caching)
- `functools.singledispatch` (dispatch based on type)
- Custom caching strategies (TTL, size-based, etc.)

## singledispatch

### What It Is

`functools.singledispatch` is a decorator that transforms a function into a single-dispatch generic function. A generic function dispatches to different implementations based on the type of the first argument. It provides a form of ad-hoc polymorphism, similar to method overloading but implemented as standalone functions.

### Why It Is Important

`singledispatch` allows you to write functions that behave differently based on the type of the first argument, without modifying the original classes or creating a complex class hierarchy. It enables the "visitor pattern" without the boilerplate and is particularly useful for serialization, formatting, and type-specific processing.

### How It Works Internally

`singledispatch` registers implementations for specific types using a registry (a dictionary mapping types to functions). When the generic function is called, it looks up the type of the first argument in the registry. If a match is found, the registered function is called. If not, it checks the MRO for registered base classes, finally falling back to the default implementation.

### Syntax

```python
from functools import singledispatch

@singledispatch
def process(arg):
    """Default implementation."""
    ...

@process.register(int)
def _(arg):
    """Implementation for int."""
    ...

@process.register(str)
def _(arg):
    """Implementation for str."""
    ...
```

### Beginner Examples

```python
from functools import singledispatch

@singledispatch
def describe(obj):
    return f"Unknown type: {type(obj).__name__}"

@describe.register(int)
def _(obj):
    return f"Integer: {obj}"

@describe.register(str)
def _(obj):
    return f"String: '{obj}'"

@describe.register(list)
def _(obj):
    return f"List of {len(obj)} items"

@describe.register(bool)  # Must be before int (bool is subclass of int)
def _(obj):
    return f"Boolean: {obj}"

print(describe(42))       # Integer: 42
print(describe("hello"))  # String: 'hello'
print(describe([1, 2]))   # List of 2 items
print(describe(True))     # Boolean: True (not Integer: True)
print(describe(3.14))     # Unknown type: float
```

### Intermediate Examples

```python
from functools import singledispatch
from collections.abc import Iterable

@singledispatch
def to_json(obj):
    """Convert object to JSON-compatible representation."""
    return f"\"{str(obj)}\""

@to_json.register(int)
@to_json.register(float)
def _(obj):
    return str(obj)

@to_json.register(str)
def _(obj):
    return f"\"{obj}\""

@to_json.register(bool)
def _(obj):
    return "true" if obj else "false"

@to_json.register(list)
def _(obj):
    items = ", ".join(to_json(item) for item in obj)
    return f"[{items}]"

@to_json.register(dict)
def _(obj):
    items = ", ".join(f"{to_json(k)}: {to_json(v)}" for k, v in obj.items())
    return f"{{{items}}}"

@to_json.register(type(None))
def _(obj):
    return "null"

# Works with nested structures
data = {
    "name": "Alice",
    "age": 30,
    "active": True,
    "scores": [85, 92, 78],
    "address": None
}
print(to_json(data))
# {"name": "Alice", "age": 30, "active": true, "scores": [85, 92, 78], "address": null}
```

### Advanced Examples

```python
from functools import singledispatch
from collections.abc import Iterable, Sequence, Mapping
from numbers import Number

@singledispatch
def process_data(data):
    raise TypeError(f"Unsupported type: {type(data)}")

@process_data.register(str)
def _(data):
    return f"Processed string: {data.upper()}"

@process_data.register(Number)
def _(data):
    return f"Processed number: {data * 2}"

# Registering for abstract base classes
@process_data.register(Sequence)
def _(data):
    if isinstance(data, (str, bytes)):
        return process_data.dispatch(str)(data)
    return f"Processed sequence: {list(data)}"

@process_data.register(Mapping)
def _(data):
    return f"Processed mapping: {dict(data)}"

@process_data.register(Iterable)
def _(data):
    return f"Processed iterable: {list(data)[:3]}..."

# Stacking type registrations
@process_data.register(tuple)
@process_data.register(set)
def _(data):
    return f"Processed collection: {len(data)} items"

# Runtime type checking
print(process_data("hello"))         # Processed string: HELLO
print(process_data(42))              # Processed number: 84
print(process_data([1, 2, 3]))       # Processed sequence: [1, 2, 3]
print(process_data({"a": 1}))        # Processed mapping: {'a': 1}
print(process_data({1, 2, 3}))       # Processed collection: 3 items

# Dispatch method resolution
print(process_data.dispatch(list))   # <function _ at ...> (sequence version)
print(process_data.registry.keys())  # All registered types
```

### Real-World Use Cases

- **Serialization**: Custom JSON, YAML, or XML serializers per type.
- **Logging formatters**: Format log messages differently based on argument type.
- **Data conversion**: Convert data between different formats based on type.
- **Plugin systems**: Register handlers for different data types dynamically.
- **Pretty printing**: Custom display formats for different data types.
- **Validation**: Type-specific validation functions without modifying original classes.

### Common Mistakes

- Forgetting that `register` decorates a function and returns it (can chain `@register`).
- Registering for a type that is already handled by a parent class (more specific types take precedence).
- Using `singledispatch` with the wrong argument position (only dispatches on the first argument).
- Expecting `singledispatch` to work with type hints instead of explicit registration (it doesn't — use `typing` protocols for that).
- Creating circular dependencies between registered implementations.

### Best Practices

- Use `singledispatch` for functions that need different implementations for different types but don't belong as methods.
- Prefer `singledispatch` over chains of `isinstance` checks for cleaner code.
- Register implementations for abstract base classes (like `Sequence`, `Mapping`) for broad coverage.
- Use `dispatch(type)` to call a specific implementation directly.
- Stack decorators: `@func.register(type)` applies `register` which is a decorator, so you can use `@func.register(type)` followed by `def _(arg):`.

### Performance Considerations

Dispatch lookup involves a type lookup in the registry dictionary (O(1)) followed by MRO traversal if the exact type isn't registered. This is fast — typically 100-200 nanoseconds. The overhead is negligible compared to the actual function logic. Building the registry at definition time has no runtime cost beyond import.

### Interview Questions

**Q: How is `singledispatch` different from method overloading in other languages?**

A: Python doesn't support method overloading by argument types. `singledispatch` provides dispatch based on the first argument at runtime (not compile time). It's more flexible than static overloading because it supports registering new types after the function is defined.

**Q: How does `singledispatch` handle inheritance?**

A: When the exact type is not registered, `singledispatch` walks the MRO looking for a registered parent class. For example, if `bool` is not registered but `int` is, and `bool` is a subclass of `int`, calling the generic function with a `bool` argument will dispatch to the `int` implementation.

### Coding Challenges

1. Write a `serialize` function using `singledispatch` that handles int, float, str, list, dict, and datetime.
2. Implement a `math_operation` dispatcher that performs different operations (add, multiply, etc.) based on operand types.
3. Create a type-based validator using `singledispatch` that validates different data types.
4. Build an event handler dispatch system using `singledispatch`.

### Related Topics

- `singledispatchmethod` (for methods)
- `functools.wraps` (used in dispatch registration)
- `functools.partial` (functional tooling)
- Abstract base classes (collections.abc)
- Type-based dispatch patterns

## wraps

### What It Is

`functools.wraps` is a decorator that copies metadata from a wrapped function to a wrapper function. It updates `__name__`, `__doc__`, `__module__`, `__qualname__`, `__annotations__`, and `__dict__` from the original function to the wrapper, and sets `__wrapped__` to reference the original function.

### Why It Is Important

Without `wraps`, decorators cause the decorated function to lose its identity — its name becomes the wrapper's name, its docstring is lost, and its annotations disappear. This breaks debugging, introspection tools, documentation generators, and any code that relies on function metadata.

### How It Works Internally

`wraps` is implemented using `functools.update_wrapper`. It assigns attributes from the wrapped function to the wrapper function using `setattr`. The `WRAPPER_ASSIGNMENTS` tuple defines which attributes to copy (`__module__`, `__name__`, `__qualname__`, `__doc__`, `__annotations__`). The `WRAPPER_UPDATES` tuple defines which attributes to update (by updating the wrapper's `__dict__`).

### Syntax

```python
from functools import wraps

def decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

### Beginner Examples

```python
from functools import wraps

def log_calls(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@log_calls
def add(a, b):
    """Add two numbers together."""
    return a + b

print(add.__name__)  # 'add' (not 'wrapper')
print(add.__doc__)   # 'Add two numbers together.'
add(3, 5)            # Calling add

# Without wraps (broken)
def log_calls_broken(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@log_calls_broken
def multiply(a, b):
    """Multiply two numbers."""
    return a * b

print(multiply.__name__)  # 'wrapper' (BAD)
print(multiply.__doc__)   # None (BAD)
```

### Intermediate Examples

```python
from functools import wraps

def retry(max_attempts=3):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    print(f"Attempt {attempt + 1} failed: {e}")
            return None
        return wrapper
    return decorator

@retry(max_attempts=3)
def unstable_network_call(url):
    """Make a network call with retry logic."""
    import random
    if random.random() < 0.7:
        raise ConnectionError("Timeout")
    return f"Response from {url}"

print(unstable_network_call.__name__)  # 'unstable_network_call'
print(unstable_network_call.__doc__)   # 'Make a network call with retry logic.'
print(unstable_network_call.__wrapped__)  # <function unstable_network_call at ...>

# Using __wrapped__ for introspection
original = unstable_network_call.__wrapped__
print(original)  # Original undecorated function
```

### Advanced Examples

```python
from functools import wraps
import inspect

def validate_types(func):
    """Decorator that validates argument types based on annotations."""
    sig = inspect.signature(func)

    @wraps(func)
    def wrapper(*args, **kwargs):
        bound = sig.bind(*args, **kwargs)
        bound.apply_defaults()

        for name, value in bound.arguments.items():
            param = sig.parameters[name]
            if param.annotation is not param.empty:
                expected = param.annotation
                if not isinstance(value, expected):
                    raise TypeError(
                        f"{name} must be {expected.__name__}, "
                        f"got {type(value).__name__}"
                    )
        return func(*args, **kwargs)
    return wrapper

@validate_types
def process(name: str, count: int) -> str:
    """Process items with validation."""
    return f"{name} x {count}"

# Metadata is preserved
print(process.__name__)    # 'process'
print(process.__doc__)     # 'Process items with validation.'
print(process.__annotations__)  # {'name': str, 'count': int, 'return': str}

# Signature is preserved via wraps + inspect
print(inspect.signature(process))  # (name: str, count: int) -> str
```

### Real-World Use Cases

- **All decorators**: Every non-trivial decorator should use `@wraps`.
- **Logging decorators**: Preserve function name and signature in logs.
- **Authorization decorators**: Maintain endpoint metadata in web frameworks.
- **Caching decorators**: Keep function identity for cache key generation.
- **Timing/profiling decorators**: Report accurate function names in profiling output.
- **API route decorators**: Flask, FastAPI route decorators use `wraps` internally.

### Common Mistakes

- Using `@wraps` on the wrong function (must be the innermost wrapper).
- Forgetting `@wraps` entirely in custom decorators.
- Expecting `wraps` to preserve the function signature (it doesn't — only name, docstring, annotations).
- Overwriting `__wrapped__` manually instead of letting `wraps` handle it.
- Using `wraps` with `partial` directly (partial doesn't have the same attributes).

### Best Practices

- Always use `@wraps(func)` on the wrapper function in every decorator.
- Use `inspect.signature` for signature preservation in addition to `wraps`.
- Use `func.__wrapped__` for unwrapping decorator chains in debuggers.
- Combine `wraps` with `functools.update_wrapper` for custom attribute copying.
- Stack `@wraps` correctly when composing decorators.

### Performance Considerations

`@wraps` adds attribute-copying overhead at decoration time (once per decorated function). The runtime performance of the wrapper is unaffected. The `__wrapped__` attribute enables efficient introspection of decorator chains.

### Interview Questions

**Q: What attributes does `@wraps` copy?**

A: `__module__`, `__name__`, `__qualname__`, `__doc__`, `__annotations__` are assigned (copied). `__dict__` is updated (wrapper's dict is updated with wrapped function's entries). `__wrapped__` is set to the original function.

**Q: Does `@wraps` preserve the function signature?**

A: No. `wraps` copies `__name__`, `__doc__`, `__module__`, `__qualname__`, `__annotations__`, and `__dict__`, but the actual signature (parameter names, defaults, kinds) is determined by the wrapper function's code object. For signature preservation, use `inspect.signature` combined with `wraps`.

### Coding Challenges

1. Write a decorator factory `logged(level)` that uses `@wraps` and logs function calls with the correct name.
2. Implement a decorator that adds type checking to a function and preserves metadata using `wraps` and `inspect.signature`.
3. Create a decorator that caches results and uses `@wraps` to maintain proper function identity.
4. Build a decorator that registers functions in a central registry and preserves all original metadata.

### Related Topics

- `functools.update_wrapper` (lower-level function)
- `functools.partial` (often used with wraps)
- Decorators (wraps is essential for proper decorators)
- `inspect.signature` (for full signature preservation)
- `__wrapped__` attribute (for unwrapping chains)
