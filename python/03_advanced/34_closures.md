# Closures - Nested functions, __closure__, nonlocal variables

## Introduction

A closure is a function that retains access to variables from its enclosing lexical scope even after that scope has finished executing. In Python, when a nested function references variables from its containing function, those variables are stored in the function's `__closure__` attribute, creating a closure. This allows the inner function to "remember" the environment in which it was created.

## Why It Is Important

Closures enable powerful programming patterns: data hiding (encapsulation without classes), factory functions, memoization, decorators, callback functions with state, and partial application. They are fundamental to Python's decorator mechanism and are widely used in event-driven programming, GUI toolkits, and functional-style code. Closures provide a clean way to associate state with a function without resorting to global variables or class instances.

## Syntax

```python
def outer_function(outer_var):
    def inner_function(inner_var):
        # Can access outer_var
        return outer_var + inner_var
    return inner_function  # Returning the inner function

# Create a closure
closure = outer_function(10)
print(closure(5))  # 15

# Closure attributes
print(closure.__closure__)    # tuple of cell objects
print(closure.__code__.co_freevars)  # names of free variables
```

## Examples

```python
from typing import Callable, Any, Optional
import time
```

### Basic Closure

```python
def multiplier(factor: float) -> Callable[[float], float]:
    def multiply(number: float) -> float:
        return number * factor
    return multiply

double = multiplier(2)
triple = multiplier(3)

print(double(5))   # 10
print(triple(5))   # 15
print(double(10))  # 20
```

### Inspecting Closure Attributes

```python
def outer(x: int) -> Callable[[], int]:
    y = 20
    def inner() -> int:
        return x + y
    return inner

closure = outer(10)
print(closure.__closure__)   # (<cell at ...>, <cell at ...>)
print(closure.__code__.co_freevars)  # ('x', 'y')
print(closure.__code__.co_varnames)  # ()
```

### Modifying Closure State (Early Python Quirk)

```python
def counter() -> Callable[[], int]:
    count = 0
    def increment() -> int:
        nonlocal count
        count += 1
        return count
    return increment

c = counter()
print(c())  # 1
print(c())  # 2
print(c())  # 3
```

## Beginner Examples

### Counter Using Closure

```python
def make_counter() -> Callable[[], int]:
    count = 0
    def counter() -> int:
        nonlocal count
        count += 1
        return count
    return counter

counter_a = make_counter()
counter_b = make_counter()

print(counter_a())  # 1
print(counter_a())  # 2
print(counter_b())  # 1  (independent state)
print(counter_a())  # 3
```

### Greeting Factory

```python
def greeter(greeting: str) -> Callable[[str], str]:
    def greet(name: str) -> str:
        return f"{greeting}, {name}!"
    return greet

say_hello = greeter("Hello")
say_hi = greeter("Hi")
say_goodbye = greeter("Goodbye")

print(say_hello("Alice"))
print(say_hi("Bob"))
print(say_goodbye("Charlie"))
```

### Power Function Factory

```python
def power_of(exponent: int) -> Callable[[int], int]:
    def power(base: int) -> int:
        return base ** exponent
    return power

square = power_of(2)
cube = power_of(3)
fourth = power_of(4)

print(square(5))   # 25
print(cube(5))     # 125
print(fourth(5))   # 625
```

### Averager (Accumulator)

```python
def running_average() -> Callable[[float], float]:
    total = 0.0
    count = 0
    def average(value: float) -> float:
        nonlocal total, count
        total += value
        count += 1
        return total / count
    return average

avg = running_average()
print(avg(10))  # 10.0
print(avg(20))  # 15.0
print(avg(30))  # 20.0
```

## Intermediate Examples

### Memoization with Closure

```python
def memoize(func: Callable) -> Callable:
    cache: dict[Any, Any] = {}
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        key = (args, tuple(sorted(kwargs.items())))
        if key not in cache:
            cache[key] = func(*args, **kwargs)
        return cache[key]
    return wrapper

@memoize
def expensive(n: int) -> int:
    """Simulate expensive computation."""
    print(f"Computing {n}...")
    time.sleep(0.1)
    return n * n

print(expensive(4))  # Computes
print(expensive(4))  # Uses cache
print(expensive(5))  # Computes
```

### Access Control with Closure

```python
def make_secure(secret_password: str) -> Callable[[str], bool]:
    def check_password(password: str) -> bool:
        return password == secret_password
    return check_password

check_admin = make_secure("admin123")
print(check_admin("wrong"))     # False
print(check_admin("admin123"))  # True
```

### Limited Call Counter

```python
def limited_calls(max_calls: int) -> Callable[[Callable], Callable]:
    def decorator(func: Callable) -> Callable:
        calls = 0
        def wrapper(*args: Any, **kwargs: Any) -> Any:
            nonlocal calls
            if calls >= max_calls:
                raise RuntimeError(f"Max calls ({max_calls}) exceeded")
            calls += 1
            return func(*args, **kwargs)
        return wrapper
    return decorator

@limited_calls(3)
def greet(name: str) -> str:
    return f"Hello, {name}!"

print(greet("Alice"))
print(greet("Bob"))
print(greet("Charlie"))
# print(greet("Diana"))  # Raises RuntimeError
```

### Closure as Partial Function

```python
def partial(func: Callable, *args: Any, **kwargs: Any) -> Callable:
    """Simple implementation of functools.partial using closure."""
    def wrapper(*more_args: Any, **more_kwargs: Any) -> Any:
        combined_kwargs = {**kwargs, **more_kwargs}
        return func(*args, *more_args, **combined_kwargs)
    return wrapper

def power(base: int, exponent: int) -> int:
    return base ** exponent

square = partial(power, exponent=2)
cube = partial(power, exponent=3)

print(square(5))  # 25
print(cube(3))    # 27
```

## Advanced Examples

### Closure for OOP-style Encapsulation

```python
def create_account(initial_balance: float = 0) -> dict[str, Callable]:
    """Create an account object using closures instead of a class."""
    balance: float = initial_balance
    transactions: list[str] = []

    def deposit(amount: float) -> float:
        nonlocal balance
        if amount <= 0:
            raise ValueError("Amount must be positive")
        balance += amount
        transactions.append(f"Deposit: +{amount}")
        return balance

    def withdraw(amount: float) -> float:
        nonlocal balance
        if amount > balance:
            raise ValueError("Insufficient funds")
        balance -= amount
        transactions.append(f"Withdraw: -{amount}")
        return balance

    def get_balance() -> float:
        return balance

    def get_statement() -> list[str]:
        return list(transactions)

    return {
        "deposit": deposit,
        "withdraw": withdraw,
        "balance": get_balance,
        "statement": get_statement,
    }

account = create_account(100)
print(account["balance"]())
account["deposit"](50)
account["withdraw"](30)
print(account["balance"]())
print(account["statement"]())
```

### Closure for Lazy Property Evaluation

```python
def lazy_property(func: Callable) -> property:
    """Implement a lazy property using closure."""
    name = f"_lazy_{func.__name__}"
    @property
    def wrapper(self: Any) -> Any:
        if not hasattr(self, name):
            setattr(self, name, func(self))
        return getattr(self, name)
    return wrapper

class Circle:
    def __init__(self, radius: float) -> None:
        self.radius = radius

    @lazy_property
    def area(self) -> float:
        print("Computing area (once)...")
        return 3.14159 * self.radius ** 2

c = Circle(10)
print(c.area)
print(c.area)  # Cached, no recomputation
```

### Closure for Currying

```python
def curry(func: Callable) -> Callable:
    """Curry a function using closures."""
    arity = func.__code__.co_argcount
    def curried(*args: Any) -> Callable:
        if len(args) >= arity:
            return func(*args)
        def more(*more_args: Any) -> Any:
            return curried(*args, *more_args)
        return more
    return curried

@curry
def add(a: int, b: int, c: int) -> int:
    return a + b + c

add5 = add(5)
add5_and_3 = add5(3)
print(add5_and_3(2))  # 10
print(add(1, 2, 3))   # 6
```

### Closure for Retry Logic

```python
def with_retry(max_attempts: int = 3, delay: float = 1.0) -> Callable:
    def decorator(func: Callable) -> Callable:
        def wrapper(*args: Any, **kwargs: Any) -> Any:
            last_exception: Optional[Exception] = None
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    last_exception = e
                    print(f"Attempt {attempt}/{max_attempts} failed: {e}")
                    if attempt < max_attempts:
                        time.sleep(delay)
            raise RuntimeError(f"All {max_attempts} attempts failed") from last_exception
        return wrapper
    return decorator

@with_retry(max_attempts=2, delay=0.1)
def flaky_call() -> str:
    import random
    if random.random() < 0.6:
        raise ConnectionError("Network timeout")
    return "Success"

import random
random.seed(42)
print(flaky_call())
```

### Closure for Environment Context

```python
def create_env(initial: dict[str, str] | None = None) -> dict[str, Callable]:
    """Simple environment/binding context using closures."""
    bindings: dict[str, Any] = dict(initial or {})

    def set_var(name: str, value: Any) -> None:
        bindings[name] = value

    def get_var(name: str) -> Any:
        if name not in bindings:
            raise NameError(f"Name '{name}' is not defined")
        return bindings[name]

    def has_var(name: str) -> bool:
        return name in bindings

    def eval_in_env(expr: str) -> Any:
        return eval(expr, {"__builtins__": {}}, bindings)

    return {
        "set": set_var,
        "get": get_var,
        "has": has_var,
        "eval": eval_in_env,
    }

env = create_env({"x": 10, "y": 20})
env["set"]("z", 30)
print(env["get"]("z"))
# print(env["eval"]("x + y + z"))  # 60
```

### Closure with Timed Expiry

```python
def make_cached_with_ttl(ttl_seconds: float = 60) -> Callable:
    """A caching closure that expires entries after TTL."""
    cache: dict[str, tuple[Any, float]] = {}

    def cached_func(func: Callable) -> Callable:
        def wrapper(*args: Any, **kwargs: Any) -> Any:
            key = str((args, tuple(sorted(kwargs.items()))))
            now = time.time()
            if key in cache:
                value, timestamp = cache[key]
                if now - timestamp < ttl_seconds:
                    return value
            value = func(*args, **kwargs)
            cache[key] = (value, now)
            return value
        return wrapper
    return cached_func

@make_cached_with_ttl(ttl_seconds=2)
def expensive_square(n: int) -> int:
    print(f"Computing {n}^2...")
    return n * n

print(expensive_square(5))
print(expensive_square(5))  # Cached
time.sleep(3)
print(expensive_square(5))  # Re-computed after TTL expiry
```

## Real-World Use Cases

- **Decorators**: Every decorator is a closure (the wrapper function closes over the original function).
- **Callback handlers**: In GUI programming, closures capture state for event handlers.
- **Factory functions**: Creating specialized versions of a function (e.g., `multiplier(2)`, `multiplier(3)`).
- **Partial application**: Fixing some arguments of a function (like `functools.partial`).
- **Memoization**: Caching results of expensive function calls.
- **Encapsulation**: Creating data-hiding patterns without classes (module pattern).
- **Configuration**: Freezing configuration values in functions passed to APIs.
- **State machines**: Holding state in a function without global variables.

## Common Mistakes

- Trying to modify a variable from the enclosing scope without `nonlocal` — creates a new local variable instead.
- Creating closures in a loop that reference the loop variable — all closures end up sharing the same final value (late binding).
- Assuming closures give write access to enclosing scope variables by default.
- Overusing closures when a class would be more appropriate and readable.
- Forgetting that `__closure__` is `None` for functions that don't reference their enclosing scope.
- Expecting closures to be serializable (most are not, unless designed for it).

### Late Binding in Loops

```python
# Problem: all closures reference the same i
funcs = [lambda: i for i in range(5)]
print([f() for f in funcs])  # [4, 4, 4, 4, 4]

# Fix: capture the value as default argument
funcs = [lambda i=i: i for i in range(5)]
print([f() for f in funcs])  # [0, 1, 2, 3, 4]
```

## Best Practices

- Use `nonlocal` explicitly when you need to reassign variables from enclosing scopes.
- Be aware of late-binding behavior when creating closures in loops; use default arguments to capture values.
- Prefer classes for complex stateful objects; use closures for simple function factories.
- Document what the closure captures for clarity.
- Use `__closure__` only for debugging or introspection, never for production logic.
- Use `functools.wraps` when creating closures that wrap other functions (decorators).
- Keep the enclosed scope small; avoid capturing large data structures unnecessarily.

## Interview Questions

1. What is a closure in Python?
2. How does Python implement closures internally (`__closure__`, `__code__.co_freevars`)?
3. What is the difference between a closure and a regular nested function?
4. How does `nonlocal` work and why is it needed?
5. What is the late-binding problem with closures in loops?
6. How do closures relate to decorators?
7. What is the difference between a closure and a class with `__call__`?
8. Can closures modify variables from the enclosing scope?
9. How can you inspect the variables a closure has captured?
10. What are some real-world use cases for closures?

## Coding Challenges

1. **Counter**: Create a closure factory that generates independent counters.
2. **Power Factory**: Write a function `power(n)` that returns a function computing `x^n`.
3. **Memoization**: Implement a generic `memoize` decorator using closures.
4. **Once**: Create a closure that ensures a function is only called once (subsequent calls return the cached result).
5. **Rate Limiter**: Write a closure that limits how often a function can be called.
6. **Compose**: Use closures to implement function composition.
7. **Partial**: Implement `functools.partial` using closures.
8. **Throttle**: Create a closure that delays function calls to run at most once per N seconds.

## Summary

A closure is a function that retains access to its enclosing scope's variables after that scope has exited. Created whenever a nested function references variables from its containing function, closures enable data hiding, function factories, decorators, memoization, and partial application. They are a fundamental building block of Python's functional programming toolkit.

## Related Topics

- Decorators
- Nested functions
- nonlocal statement
- Function scopes and LEGB rule
- functools module
- Higher-order functions
- Lambda functions
- Callable objects