# Decorators - @decorator, @wraps, decorators with arguments

## Introduction

Decorators are a powerful and expressive feature in Python that allow you to modify or enhance functions and methods without changing their source code. At their core, decorators are functions that take another function as an argument, add some kind of functionality, and return a new function. This functional programming concept enables cleaner, more readable code by separating cross-cutting concerns from business logic.

Python's decorator syntax, using the `@` symbol, provides a concise way to apply wrappers around functions, classes, or methods. Decorators are widely used for logging, access control, memoization, timing, and enforcing validation rules.

## @decorator Syntax

### What It Is

The `@decorator` syntax in Python is syntactic sugar for applying a decorator function to a target function or class. When you write `@decorator` above a function definition, Python automatically passes the defined function to the decorator function and reassigns the result back to the function name.

### Why It Is Important

This syntax makes the intent of code transformation explicit and readable. Without decorators, you would need to write `func = decorator(func)` after every function definition, which is repetitive and obscures the purpose of the transformation. The `@` notation places the decoration right at the definition site, making it immediately clear what behaviour is being added.

### How It Works Internally

When the Python interpreter encounters a decorated function definition:

1. It defines the function normally (compiles and stores it).
2. It evaluates the expression after `@` (the decorator).
3. It calls the decorator with the newly defined function as an argument.
4. It binds the result of the decorator call to the same name as the original function.

This means the decorator can return either the original function (possibly modified) or a completely different callable.

### Syntax

```python
@decorator
def func():
    pass
# Equivalent to: func = decorator(func)


@decorator(arg)
def func():
    pass
# Equivalent to: func = decorator(arg)(func)
```

### Beginner Examples

```python
def simple_decorator(func):
    def wrapper():
        print("Before the function runs")
        func()
        print("After the function runs")
    return wrapper

@simple_decorator
def say_hello():
    print("Hello!")

say_hello()
# Output:
# Before the function runs
# Hello!
# After the function runs
```

### Intermediate Examples

```python
def log_decorator(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__} with args={args}, kwargs={kwargs}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result}")
        return result
    return wrapper

@log_decorator
def add(a, b):
    return a + b

@log_decorator
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}"

print(add(3, 5))
print(greet("Alice"))
```

### Advanced Examples

```python
import time
import random

def retry(max_attempts=3, delay=1):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    print(f"Attempt {attempt + 1} failed: {e}")
                    time.sleep(delay)
            return None
        return wrapper
    return decorator

@retry(max_attempts=5, delay=0.5)
def unstable_network_call():
    if random.random() < 0.6:
        raise ConnectionError("Network timeout")
    return "Success"

print(unstable_network_call())
```

### Real-World Use Cases

- **Web frameworks**: Flask and Django use decorators for routing (`@app.route('/home')`).
- **Authentication**: `@login_required` decorators protect views from unauthorized access.
- **Caching**: `@lru_cache` from functools caches function results.
- **Rate limiting**: Decorators limit API call frequency per user.
- **Validation**: Decorators validate function inputs and outputs automatically.

### Common Mistakes

- Forgetting to return the wrapper function from the decorator (returns `None`).
- Losing the original function's metadata (name, docstring) — use `@wraps`.
- Applying a decorator that doesn't accept arguments to one that needs them.
- Stacking decorators in the wrong order — outermost decorator runs first.

### Best Practices

- Always use `@wraps` from `functools` to preserve metadata.
- Use `*args, **kwargs` in wrapper functions for generality.
- Keep decorators simple and focused on a single responsibility.
- Consider using classes with `__call__` for stateful decorators.
- Write unit tests for decorators in isolation from the decorated functions.

### Performance Considerations

Every decorator adds a layer of function call overhead. For performance-critical code, minimize decorator nesting (each `@` adds a wrapper call). Use `lru_cache` judiciously as it stores results in memory. Decorators that do I/O (logging, network calls) should be asynchronous-friendly. In hot paths, consider applying decorators only where needed rather than decorating every function in a module.

### Interview Questions

**Q: What is the difference between a decorator and a decorator factory?**

A: A decorator is a function that takes a function and returns a function. A decorator factory is a function that returns a decorator, allowing parameterization. Example: `@decorator` vs `@decorator(arg)`.

**Q: How do you preserve metadata when writing decorators?**

A: Use `@wraps` from `functools`, which copies `__name__`, `__doc__`, `__module__`, and `__dict__` from the original function to the wrapper.

**Q: Can decorators be applied to classes?**

A: Yes. A class decorator receives the class as argument and returns a modified class or a wrapper. Common examples include `@dataclass` and `@singleton` implementations.

### Coding Challenges

1. **Timer decorator**: Write a `@timer` decorator that prints the execution time of any function.
2. **Memoize decorator**: Implement a `@memoize` decorator that caches results based on arguments.
3. **Deprecation warning**: Write a `@deprecated` decorator that prints a warning when a function is called.
4. **Rate limiter**: Implement a decorator that limits a function to N calls per second.

### Related Topics

- Closures (decorators rely on closure scoping)
- Functools (`wraps`, `lru_cache`, `singledispatch`)
- Metaclasses (class-level decorators alternative)
- Context managers (similar wrapping pattern for resources)
- Partial functions (decorator factories often use `functools.partial`)

## @wraps

### What It Is

`functools.wraps` is a decorator that copies the metadata (`__name__`, `__doc__`, `__module__`, `__dict__`, `__wrapped__`) from the original function to the wrapper function. Without it, the decorated function loses its identity, making debugging and introspection difficult.

### Why It Is Important

Preserving function metadata is essential for documentation tools (Sphinx, pydoc), debugging (tracebacks show wrapper names), serialization, and any code that inspects function signatures. `@wraps` also sets the `__wrapped__` attribute, which allows unwrapping decorator chains for introspection.

### How It Works Internally

`wraps` is implemented as a decorator factory. It calls `functools.partial(update_wrapper, wrapped=func, assigned=WRAPPER_ASSIGNMENTS, updated=WRAPPER_UPDATES)`. `update_wrapper` copies attributes from the original function to the wrapper, then returns the wrapper. The `WRAPPER_ASSIGNMENTS` tuple includes `__module__`, `__name__`, `__qualname__`, `__doc__`, and `__annotations__`. `WRAPPER_UPDATES` includes `__dict__`.

### Syntax

```python
from functools import wraps

def my_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

### Beginner Examples

```python
from functools import wraps

def debug_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Debug: {func.__name__} called")
        return func(*args, **kwargs)
    return wrapper

@debug_decorator
def calculate(x, y):
    """Multiply two numbers."""
    return x * y

print(calculate.__name__)  # 'calculate' (not 'wrapper')
print(calculate.__doc__)   # 'Multiply two numbers.' (not None)
```

### Intermediate Examples

```python
from functools import wraps

def validate_types(**type_hints):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for arg_name, arg_value in zip(func.__code__.co_varnames, args):
                if arg_name in type_hints:
                    expected = type_hints[arg_name]
                    if not isinstance(arg_value, expected):
                        raise TypeError(
                            f"{arg_name} must be {expected.__name__}, "
                            f"got {type(arg_value).__name__}"
                        )
            return func(*args, **kwargs)
        return wrapper
    return decorator

@validate_types(x=int, y=int)
def divide(x, y):
    return x / y

divide(10, 5)   # Works
divide("10", 5) # Raises TypeError
```

### Advanced Examples

```python
from functools import wraps

def trace_calls(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        args_repr = [repr(a) for a in args]
        kwargs_repr = [f"{k}={v!r}" for k, v in kwargs.items()]
        signature = ", ".join(args_repr + kwargs_repr)
        print(f"→ {func.__name__}({signature})")
        result = func(*args, **kwargs)
        print(f"← {func.__name__} = {result!r}")
        return result
    return wrapper

@trace_calls
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

fibonacci(5)
```

### Real-World Use Cases

- **Logging decorators** that need to report the correct function name.
- **Web framework decorators** that register routes with correct endpoint names.
- **Testing tools** that wrap test functions without losing test identification.
- **Profiling tools** that need accurate function names in reports.

### Common Mistakes

- Forgetting to apply `@wraps` on the innermost wrapper function.
- Applying `@wraps` to the outer function of a decorator factory instead of the wrapper.
- Expecting `wraps` to copy the signature — it does not. Use `inspect.signature` or a third-party library like `decorator` for signature preservation.
- Overwriting `__wrapped__` manually instead of letting `wraps` handle it.

### Best Practices

- Always use `@wraps` on every decorator you write.
- Use `@wraps` on the innermost wrapper, not on the decorator factory.
- Combine `@wraps` with `inspect.signature` when signature preservation is needed.
- Access the original function via `__wrapped__` for introspection in decorator chains.

### Performance Considerations

`@wraps` adds minimal overhead during decoration (copying attributes once). The runtime performance of the wrapper is unchanged. The `__wrapped__` attribute allows tools to skip wrapper layers during deep introspection, improving debugging performance.

### Interview Questions

**Q: Does `@wraps` preserve the function signature?**

A: No. It only copies `__name__`, `__doc__`, `__module__`, `__qualname__`, `__annotations__`, and `__dict__`. For signature preservation, use `inspect.signature` or libraries like `decorator`.

**Q: What is `__wrapped__` and why is it useful?**

A: `__wrapped__` is an attribute set by `@wraps` that references the original unwrapped function. It allows tools to bypass decorator chains and access the original function for introspection.

### Coding Challenges

1. Write a decorator with `@wraps` that logs all calls and their return values.
2. Create a `@logged` decorator that writes to a file the function name and timestamp.
3. Implement a decorator stack where `@wraps` correctly preserves identity even with multiple decorators.

### Related Topics

- `functools.update_wrapper` (the lower-level function called by `wraps`)
- `functools.partial` (used internally by `wraps`)
- `inspect.signature` (for actual signature preservation)
- Decorator composition (stacking multiple `@wraps`-aware decorators)

## Decorators with Arguments

### What It Is

Decorators with arguments, also called decorator factories, are functions that accept parameters and return a decorator. This allows parameterizing the behaviour of the decorator at decoration time, such as specifying log levels, retry counts, or permission levels.

### Why It Is Important

Parameterized decorators eliminate code duplication by allowing the same decorator pattern to be reused with different configurations. They enable declarative configuration at the definition site, making the code more expressive and maintainable.

### How It Works Internally

A decorator factory is a function that returns a decorator function. The `@decorator(args)` syntax evaluates the factory call first, then applies the returned decorator to the function. This is equivalent to `func = decorator_factory(args)(func)`.

```python
def factory(arg):
    def decorator(func):
        def wrapper(*args, **kwargs):
            # arg is available here via closure
            return func(*args, **kwargs)
        return wrapper
    return decorator
```

### Syntax

```python
def decorator_factory(parameter):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            # Use parameter here
            return func(*args, **kwargs)
        return wrapper
    return decorator

@decorator_factory(value)
def my_func():
    pass
```

### Beginner Examples

```python
from functools import wraps

def repeat(times):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(times=3)
def greet(name):
    print(f"Hello, {name}!")

greet("Alice")
# Hello, Alice!
# Hello, Alice!
# Hello, Alice!
```

### Intermediate Examples

```python
from functools import wraps

def log(level="INFO"):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            print(f"[{level}] Calling {func.__name__}")
            return func(*args, **kwargs)
        return wrapper
    return decorator

@log(level="DEBUG")
def process_data(data):
    return data * 2

@log()
def simple_task():
    return "done"

process_data(5)
simple_task()
```

### Advanced Examples

```python
from functools import wraps
import time

def throttle(rate_per_second):
    min_interval = 1.0 / rate_per_second
    last_called = [0.0]

    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            elapsed = time.time() - last_called[0]
            if elapsed < min_interval:
                time.sleep(min_interval - elapsed)
            last_called[0] = time.time()
            return func(*args, **kwargs)
        return wrapper
    return decorator

@throttle(rate_per_second=2)
def api_call(endpoint):
    print(f"Calling {endpoint} at {time.time():.2f}")

for _ in range(5):
    api_call("/users")
```

### Real-World Use Cases

- **Flask/Django route decorators**: `@app.route('/path', methods=['GET'])`.
- **Permission decorators**: `@requires_role('admin')` or `@has_permission('write')`.
- **Caching**: `@cache(ttl=300)` to cache with a time-to-live.
- **Rate limiting**: `@rate_limit(calls=100, period=60)` to limit API usage.
- **Retry logic**: `@retry(max_attempts=3, backoff=2)` for resilient services.

### Common Mistakes

- Forgetting the extra nesting level for the factory function.
- Confusing the decorator with the decorator factory (calling `@decorator` when `@decorator()` is needed).
- Not using `@wraps` on the innermost wrapper.
- Mutable default arguments in the factory being shared across decorations.

### Best Practices

- Always use keyword arguments for clarity in decorator factories.
- Provide sensible defaults so `@decorator()` works without arguments.
- Document the parameters clearly for users of your decorator.
- Consider supporting both `@decorator` and `@decorator(args)` using a partial or by inspecting the first argument.

### Performance Considerations

Each level of nesting adds function call overhead during decoration (not during execution). The outer factory is called once at definition time. The closure captures factory arguments, keeping them in memory for the lifetime of the decorated function. Complex decorator factories with heavy initialization should be optimized or executed lazily.

### Interview Questions

**Q: How do you write a decorator that works both with and without arguments?**

A: Use a single-argument decorator that checks if the argument is the function itself or a configuration parameter, or use `functools.partial` to detect the calling pattern.

**Q: What is the closure mechanism in decorator factories?**

A: Each nested function captures the variables from its enclosing scope. The factory's parameters are captured by the decorator function's closure, and the original function is captured by the wrapper's closure.

### Coding Challenges

1. Write a `@timeout(seconds=5)` decorator that raises an exception if the function takes too long.
2. Create a `@cached(ttl=60)` decorator that caches results for a configurable duration.
3. Implement a `@validate(schema=...)` decorator that validates inputs against a JSON-like schema.
4. Build a `@circuit_breaker(failure_threshold=5, recovery_timeout=30)` decorator.

### Related Topics

- Functools `partial` (for partial application in decorator factories)
- Closures (the underlying mechanism for parameter capture)
- First-class functions (decorators are an application of higher-order functions)
- `functools.singledispatch` (a specialized decorator for generic functions)
