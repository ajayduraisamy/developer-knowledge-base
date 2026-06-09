# Functions - def, return, parameters, *args, **kwargs

## Introduction

Functions are reusable, organized blocks of code that perform a specific task. They are defined with the `def` keyword, can accept parameters, return values, and support flexible argument passing through `*args` and `**kwargs`. Functions in Python are first-class objects — they can be assigned to variables, passed as arguments, and returned from other functions. Understanding parameters, argument passing, and the varied call signatures is crucial for writing flexible, reusable Python code.

## def Statement

### What It Is
The `def` statement defines a function — a named block of code that can be called later. It creates a function object bound to the given name in the current namespace. Functions can have parameters, a docstring, a body, and a return value.

### Why It Is Important
The `def` statement is the primary mechanism for defining reusable logic. It enables code organization, the DRY (Don't Repeat Yourself) principle, abstraction, and modular design. Understanding how `def` works is foundational to all Python programming.

### How It Works Internally
When Python encounters `def`, it compiles the function body into a code object (containing bytecode, constants, and variable names) and creates a function object wrapping that code object. The function object stores `__defaults__` (default argument values), `__code__` (compiled bytecode), `__globals__` (reference to global namespace), `__closure__` (for closures), and annotations. The function is not executed until called — `def` only creates the object.

### Syntax
```python
# Basic function
def function_name(parameters):
    """Docstring."""
    body
    return value

# Function with type hints
def add(a: int, b: int) -> int:
    return a + b

# Lambda (anonymous function)
lambda params: expression

# Inner/nested function
def outer():
    def inner():
        pass
    return inner
```

### Beginner Examples
```python
# Simple function
def greet():
    print("Hello, World!")

greet()  # Hello, World!

# Function with parameter
def greet_person(name):
    print(f"Hello, {name}!")

greet_person("Alice")  # Hello, Alice!

# Function with return
def square(x):
    return x * x

result = square(5)
print(result)  # 25

# Multiple returns
def min_max(numbers):
    return min(numbers), max(numbers)

minimum, maximum = min_max([3, 1, 7, 2, 9])
print(f"Min: {minimum}, Max: {maximum}")
```

### Intermediate Examples
```python
# Functions as first-class objects
def apply(func, value):
    return func(value)

def double(x):
    return x * 2

print(apply(double, 5))  # 10
print(apply(lambda x: x ** 2, 5))  # 25

# Function attributes
def func():
    """Example function."""
    func.calls += 1
    return func.calls

func.calls = 0
print(func())  # 1
print(func())  # 2

# Inner functions
def make_multiplier(factor):
    def multiply(x):
        return x * factor
    return multiply

double = make_multiplier(2)
triple = make_multiplier(3)
print(double(5))  # 10
print(triple(5))  # 15

# Functions with type hints
from typing import List, Optional

def process_items(items: List[str], prefix: Optional[str] = None) -> List[str]:
    if prefix:
        return [f"{prefix}: {item}" for item in items]
    return items

print(process_items(["a", "b"], "Item"))
```

### Advanced Examples
```python
# Decorators — functions modifying functions
from functools import wraps
import time

def timer(func):
    """Decorator that measures execution time."""
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__} took {elapsed:.6f}s")
        return result
    return wrapper

@timer
def slow_sum(n: int) -> int:
    return sum(range(n))

print(slow_sum(10_000_000))

# Decorator with arguments
def retry(max_attempts: int = 3, delay: float = 0.1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            import time
            last_exception = None
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    last_exception = e
                    time.sleep(delay * (attempt + 1))
            raise last_exception
        return wrapper
    return decorator

@retry(max_attempts=3)
def unstable_operation():
    import random
    if random.random() < 0.7:
        raise ConnectionError("Temporary failure")
    return "Success"

# Partial function application
from functools import partial

def power(base, exponent):
    return base ** exponent

square = partial(power, exponent=2)
cube = partial(power, exponent=3)
print(square(5))  # 25
print(cube(3))    # 27

# Function introspection
import inspect

def example(a: int, b: str = "default") -> bool:
    """Example function with annotations."""
    return True

sig = inspect.signature(example)
for name, param in sig.parameters.items():
    print(f"{name}: kind={param.kind.name}, default={param.default}")

# Callable class (alternative to function)
class Counter:
    def __init__(self):
        self.count = 0
    
    def __call__(self):
        self.count += 1
        return self.count

counter = Counter()
print(counter())  # 1
print(counter())  # 2
print(counter())  # 3
```

### Real-World Use Cases
- **Decorators**: Logging, caching, authentication, rate limiting
- **Utility Libraries**: Reusable helper functions across projects
- **API Handlers**: Flask/FastAPI route functions
- **Data Pipelines**: ETL functions for data transformation
- **Callbacks**: Event handlers, sort keys, map/reduce functions
- **Factory Functions**: Creating configured objects

### Common Mistakes
```python
# Mistake 1: Mutable default arguments
def append_to(item, target=[]):
    target.append(item)
    return target

print(append_to(1))  # [1]
print(append_to(2))  # [1, 2] — not [2]!

# Correct:
def append_to(item, target=None):
    if target is None:
        target = []
    target.append(item)
    return target

# Mistake 2: Forgetting return
def square(n):
    result = n * n  # No return!

print(square(5))  # None

# Mistake 3: Modifying mutable arguments
def process(data):
    data.clear()  # Modifies the caller's list!
    return data

original = [1, 2, 3]
process(original)
print(original)  # []

# Mistake 4: Function name shadowing builtins
def list():  # Shadows built-in list()
    return "my list"

# Mistake 5: Late binding in closures
funcs = [lambda: i for i in range(5)]
print([f() for f in funcs])  # [4, 4, 4, 4, 4]

# Correct:
funcs = [lambda i=i: i for i in range(5)]
print([f() for f in funcs])  # [0, 1, 2, 3, 4]
```

### Best Practices
- Functions should do one thing (Single Responsibility Principle)
- Keep functions short (ideally under 30-40 lines)
- Use descriptive names (verbs for functions): `calculate_average()`
- Include docstrings for public functions
- Use type annotations for clarity and tooling support
- Avoid mutable default arguments
- Prefer returning values over printing or modifying globals
- Use `@wraps` when writing decorators
- Use early returns to reduce nesting
- Write pure functions (no side effects) when possible

### Performance Considerations
- Function call overhead is ~50-100ns in CPython
- Local variable access is faster than global or closure access
- Python 3.12+ has faster function calls (specialized bytecode)
- Built-in functions (`sum`, `max`, `min`) are faster than manual loops
- `functools.lru_cache` can memoize expensive function calls
- Decorators add call overhead; use `@wraps` to preserve metadata

### Interview Questions
1. How does Python define functions internally?
2. What is the difference between a function and a method?
3. How do decorators work in Python?
4. What are closures and how do they capture variables?
5. What is the difference between `@staticmethod` and `@classmethod`?
6. How does function argument passing work (pass-by-object-reference)?
7. What is the `functools.wraps` decorator and why use it?
8. How do you create a function with an arbitrary number of arguments?
9. What are docstrings and how are they accessed?
10. How do Python lambda functions differ from regular functions?

### Coding Challenges
```python
# Challenge 1: Memoization decorator
from functools import wraps

def memoize(func):
    cache = {}
    @wraps(func)
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    return wrapper

@memoize
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

print(fibonacci(100))  # 354224848179261915075

# Challenge 2: Function composition
def compose(*funcs):
    """Compose functions right-to-left: compose(f, g)(x) = f(g(x))"""
    from functools import reduce
    def composed(x):
        return reduce(lambda v, f: f(v), reversed(funcs), x)
    return composed

add_one = lambda x: x + 1
double = lambda x: x * 2
f = compose(add_one, double)
print(f(5))  # 11 (double(5) = 10, add_one(10) = 11)
```

### Related Topics
- Lambda Functions
- Decorators
- Closures and Scope
- Recursion
- `functools` Module
- Type Annotations
- Generators (`yield`)

## return Statement

### What It Is
The `return` statement exits a function and optionally passes a value back to the caller. If no expression is specified, `return` (or reaching the end of a function) returns `None`.

### Why It Is Important
`return` is how functions communicate results to calling code. It enables function composition, pipeline processing, and the separation of computation from side effects. Understanding early returns helps write cleaner, less-nested code.

### How It Works Internally
`return` compiles to `RETURN_VALUE` bytecode, which pops the top-of-stack value (or `None` if no value) and stores it in the call frame's return value slot. The function's frame is then deallocated (or frozen for generators). For functions with complex cleanup, `try/finally` blocks are executed before the return.

### Syntax
```python
# Return a value
def add(a, b):
    return a + b

# Return nothing (implicitly returns None)
def log(message):
    print(message)
    # No return — returns None

# Early return
def divide(a, b):
    if b == 0:
        return None  # Early return
    return a / b

# Return multiple values (as tuple)
def get_stats(numbers):
    return min(numbers), max(numbers), sum(numbers) / len(numbers)
```

### Beginner Examples
```python
# Basic return
def square(n):
    return n * n

result = square(4)
print(result)  # 16

# Returning different types
def describe(n):
    if n > 0:
        return "positive"
    elif n < 0:
        return "negative"
    return "zero"

print(describe(-5))  # negative

# Early return pattern
def validate_age(age):
    if age < 0:
        return "Invalid: negative age"
    if age < 18:
        return "Minor"
    if age < 65:
        return "Adult"
    return "Senior"

# Multiple return values
def min_max_avg(numbers):
    return min(numbers), max(numbers), sum(numbers) / len(numbers)

lo, hi, avg = min_max_avg([1, 2, 3, 4, 5])
print(f"lo={lo}, hi={hi}, avg={avg}")
```

### Intermediate Examples
```python
# Return value unpacking
def get_user():
    return "Alice", 30, "alice@example.com"

name, age, email = get_user()
print(f"{name} ({age}): {email}")

# Sentinel return pattern
_MISSING = object()

def find_item(items, target, default=_MISSING):
    for item in items:
        if item == target:
            return item
    if default is _MISSING:
        raise ValueError(f"{target} not found")
    return default

# Return from nested functions
def get_config_value(path: list[str]):
    def deep_get(d, keys):
        if not keys:
            return d
        if isinstance(d, dict) and keys[0] in d:
            return deep_get(d[keys[0]], keys[1:])
        return None
    return deep_get(config, path)

config = {"database": {"host": "localhost", "port": 5432}}
print(get_config_value(["database", "host"]))  # localhost

# Conditional return simplification
def classify(n):
    if n > 0: return "positive"
    if n < 0: return "negative"
    return "zero"
```

### Advanced Examples
```python
# The return is handled by try/finally for cleanup
def read_file_with_cleanup(path: str) -> str | None:
    """Return file contents, ensuring file is closed."""
    f = None
    try:
        f = open(path, "r")
        return f.read()  # finally runs BEFORE returning!
    except FileNotFoundError:
        return None
    finally:
        if f:
            f.close()  # Always executes, even after return

# Context manager version (better)
def read_file(path: str) -> str | None:
    try:
        with open(path, "r") as f:
            return f.read()
    except FileNotFoundError:
        return None

# Return with teardown
import contextlib

@contextlib.contextmanager
def managed_resource():
    print("Acquiring resource")
    resource = {"data": []}
    try:
        yield resource
    finally:
        print("Releasing resource")

def process_data():
    with managed_resource() as res:
        res["data"].append(42)
        return res  # Finally runs before return completes

result = process_data()
# Acquiring resource
# Releasing resource
print(result)  # {'data': [42]}

# Generator return (Python 3.3+)
def range_with_return(stop: int):
    i = 0
    while i < stop:
        yield i
        i += 1
    return "done"  # Available in StopIteration.value

gen = range_with_return(3)
try:
    while True:
        print(next(gen))
except StopIteration as e:
    print(f"Returned: {e.value}")  # done
```

### Real-World Use Cases
- **API Endpoints**: Returning JSON responses
- **Validation**: Returning error messages or success indicators
- **Data Processing**: Returning transformed data
- **Configuration**: Returning computed or merged settings
- **Caching**: Returning cached values or None for misses

### Common Mistakes
```python
# Mistake 1: Forgetting return — returns None
def square(n):
    n * n  # No return!

print(square(5))  # None

# Mistake 2: Unreachable code after return
def example():
    return 42
    print("Never runs")  # Unreachable

# Mistake 3: Returning too many values without unpacking
def get_data():
    return 1, 2, 3, 4, 5

data = get_data()  # Tuple
print(data)  # (1, 2, 3, 4, 5)

# Mistake 4: Implicit return of None in comparison
def is_even(n):
    if n % 2 == 0:
        return True
    # Missing: return False

print(is_even(3))  # None (not False!)

# Mistake 5: Returning from finally overrides other returns
def test():
    try:
        return 42
    finally:
        return 0  # This return overrides the 42!

print(test())  # 0
```

### Best Practices
- Always return a consistent type from a function (not sometimes str, sometimes None)
- Use early returns to avoid deep nesting
- Return `None` to indicate "no result" only if the caller expects it
- Use `Optional[Type]` in type hints when returning None is possible
- Prefer returning multiple values as a named tuple or dataclass for clarity
- Avoid complex expressions in return statements; assign to a variable first
- Use `return` without value to exit a void function early

### Performance Considerations
- `return` is a single bytecode instruction (very fast)
- Returning large objects doesn't copy them (just returns a reference)
- Return values are passed by reference; no copying occurs
- Python 3.12+ optimizes common return patterns
- Returning from deeply nested loops may be slower than using a local variable

### Interview Questions
1. What does a function return if there's no `return` statement?
2. Can you have multiple return statements in a function?
3. How does `return` interact with `try`/`finally`?
4. What does returning multiple values actually return?
5. What is the difference between `return` and `yield`?
6. How do you return early from a function?
7. What happens when you return from a `with` block?
8. Can you return a function from another function?
9. What is the difference between `return` and `raise`?
10. How do you return a value from a generator (Python 3.3+)?

### Coding Challenges
```python
# Challenge 1: Early return for validation
def process_order(order: dict) -> dict:
    if not order.get("items"):
        return {"error": "No items in order", "code": 400}
    if not order.get("customer_id"):
        return {"error": "Missing customer", "code": 400}
    if order.get("total", 0) <= 0:
        return {"error": "Invalid total", "code": 400}
    return {"status": "success", "order_id": 123}

# Challenge 2: @dataclass return for multiple values
from dataclasses import dataclass

@dataclass
class Analysis:
    min: float
    max: float
    mean: float
    median: float
    std: float

def analyze(numbers: list[float]) -> Analysis:
    import statistics
    return Analysis(
        min=min(numbers),
        max=max(numbers),
        mean=statistics.mean(numbers),
        median=statistics.median(numbers),
        std=statistics.stdev(numbers) if len(numbers) > 1 else 0.0
    )

result = analyze([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
print(f"Mean: {result.mean:.2f}, Median: {result.median}")
```

### Related Topics
- The `return` Statement
- `yield` and Generators
- Function Return Types
- `try`/`finally` with Return
- Sentinel Values

## Parameters

### What It Is
Parameters are the variables listed in a function's definition that receive values when the function is called. Python supports positional parameters, keyword parameters, default parameters, and variable-length parameters.

### Why It Is Important
Understanding parameter types and their ordering rules is essential for writing flexible, clear functions. Python's parameter system supports positional-only (Python 3.8+), keyword-only, and variadic parameters, enabling precise API design.

### How It Works Internally
When a function is called, CPython creates a frame with a local variables array. Positional arguments are packed into a tuple; keyword arguments into a dictionary. The `__code__.co_varnames` stores parameter names, `co_flags` indicates if `*args` or `**kwargs` are used. Default values are stored in `__defaults__` and `__kwdefaults__`. Python 3.8+ supports positional-only parameters with `/` in the signature.

### Syntax
```python
# Parameter types
def func(pos1, pos2, /, pos_or_kwd, *, kwd1, kwd2):
    """Positional-only | Positional-or-keyword | Keyword-only."""
    pass

# Default parameters
def func(a, b=10, c="default"):
    pass

# Variable-length parameters
def func(*args, **kwargs):
    pass
```

### Beginner Examples
```python
# Positional parameters
def greet(name, greeting):
    print(f"{greeting}, {name}!")

greet("Alice", "Hello")  # Hello, Alice!

# Keyword arguments (order doesn't matter)
def create_profile(name, age, city="Unknown"):
    return f"{name}, {age}, from {city}"

print(create_profile(age=30, name="Bob"))
print(create_profile("Charlie", 25, city="London"))

# Default parameters
def power(base, exponent=2):
    return base ** exponent

print(power(5))     # 25 (5^2)
print(power(5, 3))  # 125 (5^3)
```

### Intermediate Examples
```python
# Positional-only parameters (Python 3.8+)
def greet(name, /, greeting="Hello"):
    """name is positional-only, greeting is positional-or-keyword."""
    return f"{greeting}, {name}!"

print(greet("Alice"))           # OK
print(greet("Bob", "Hi"))       # OK
# print(greet(name="Charlie"))  # TypeError!

# Keyword-only parameters
def create_user(name, *, age=None, email=None):
    """After *, all parameters are keyword-only."""
    return {"name": name, "age": age, "email": email}

print(create_user("Alice", age=30, email="a@b.com"))
# print(create_user("Alice", 30))  # TypeError!

# Combined parameter types
def format_date(year, month, day, /, *, separator="-"):
    """year, month, day are positional-only; separator is keyword-only."""
    return f"{year:04d}{separator}{month:02d}{separator}{day:02d}"

print(format_date(2024, 3, 15))           # 2024-03-15
print(format_date(2024, 3, 15, separator="/"))  # 2024/03/15

# Default parameter evaluation
import time

def log_message(message, timestamp=None):
    if timestamp is None:
        timestamp = time.time()
    print(f"[{timestamp:.3f}] {message}")
```

### Advanced Examples
```python
# Parameter annotations and introspection
import inspect

def api_handler(
    request,  # type: Request
    /,        # positional-only
    db_session=None,
    *,
    timeout: float = 30.0,
    retry: int = 3,
    verbose: bool = False,
) -> dict:
    """Handle API request with configurable options."""
    return {"status": "ok"}

sig = inspect.signature(api_handler)
for name, param in sig.parameters.items():
    print(f"{name}: {param.kind.name} (default={param.default})")

# Partial application with parameter binding
from functools import partial

def send_email(to: str, subject: str, body: str, cc: str | None = None, bcc: str | None = None):
    print(f"To: {to}, Subject: {subject}")

# Pre-configure some parameters
send_admin_email = partial(send_email, to="admin@example.com")
send_admin_email("System Alert", "Server is down")

# Parameter forwarding with * and **
def wrapper(*args, **kwargs):
    print(f"args={args}, kwargs={kwargs}")
    return original_function(*args, **kwargs)

# Dynamic parameter binding
import dataclasses

@dataclasses.dataclass
class Config:
    host: str = "localhost"
    port: int = 8080
    debug: bool = False
    
    def to_params(self) -> dict:
        return dataclasses.asdict(self)

def start_server(**params):
    print(f"Starting server with: {params}")

config = Config(host="0.0.0.0", port=3000)
start_server(**config.to_params())
```

### Real-World Use Cases
- **API Design**: Positional-only for required params, keyword-only for options
- **Configuration**: Functions with many optional keyword parameters
- **Wrapper/Decorator Functions**: Using `*args, **kwargs` for forwarding
- **Data Processing**: Functions with flexible input formats
- **CLI Commands**: Parameters mapping to command-line flags

### Common Mistakes
```python
# Mistake 1: Mutable default parameters
def add_item(item, items=[]):
    items.append(item)
    return items
# Default list is shared across calls!

# Mistake 2: Wrong parameter order
def func(a, b=10, c):  # SyntaxError! non-default after default
    pass

# Must be: positional, default, *args, keyword-only

# Mistake 3: Forgetting / or * syntax
# To make all params keyword-only, put * first:
def func(*, a, b): pass  # Both keyword-only
# func(1, 2)  # TypeError

# Mistake 4: Parameter shadowing
def func(list):  # Shadows built-in list
    return list([1, 2, 3])

# Mistake 5: Using mutable types as default for caching
def fib(n, cache={}):  # Shared cache!
    if n in cache:
        return cache[n]
    # ...
```

### Best Practices
- Use positional-only parameters (`/`) for parameters that are order-dependent
- Use keyword-only parameters (`*`) for options and flags
- Default parameter values should be immutable (never `[]` or `{}`)
- Limit the number of parameters (3-5 is ideal; more suggests refactoring)
- Use `*args` and `**kwargs` sparingly (they hide the signature)
- Provide defaults for optional parameters
- Use type annotations for all parameters
- Document complex parameter interactions

### Performance Considerations
- Positional arguments are faster than keyword arguments (no dict lookup)
- Default argument values are evaluated once at def-time, not at call-time
- `*args` packs remaining positional args into a tuple (small overhead)
- `**kwargs` packs keyword args into a dict (more overhead)
- Python 3.12+ optimizes parameter passing with specialized bytecode
- Accessing `*args` inside the function is a tuple iteration (fast)

### Interview Questions
1. What is the difference between positional and keyword arguments?
2. How do you define positional-only parameters (Python 3.8+)?
3. How do you define keyword-only parameters?
4. Why are mutable default arguments dangerous?
5. When are default argument values evaluated?
6. What is the parameter ordering rule in Python?
7. How do you forward all arguments to another function?
8. What is the difference between `*args` and `**kwargs`?
9. How do you give a function an unlimited number of parameters?
10. What is parameter annotation and how is it used?

### Coding Challenges
```python
# Challenge 1: Parameter-validating decorator
from functools import wraps

def validate_params(**validators):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            import inspect
            bound = inspect.signature(func).bind(*args, **kwargs)
            bound.apply_defaults()
            for name, value in bound.arguments.items():
                if name in validators:
                    if not validators[name](value):
                        raise ValueError(f"Invalid {name}: {value}")
            return func(*args, **kwargs)
        return wrapper
    return decorator

@validate_params(age=lambda x: 0 <= x <= 150)
def set_age(age: int):
    return f"Age set to {age}"

print(set_age(30))   # OK
# set_age(200)  # ValueError

# Challenge 2: Flexible CSV parser
def parse_csv(data: str, /, *, delimiter=",", has_header=True, skip_empty=True):
    lines = data.strip().splitlines()
    if skip_empty:
        lines = [l for l in lines if l.strip()]
    if has_header:
        header = lines[0].split(delimiter)
        rows = [dict(zip(header, line.split(delimiter))) for line in lines[1:]]
    else:
        rows = [line.split(delimiter) for line in lines]
    return rows
```

### Related Topics
- `*args` and `**kwargs`
- Argument Unpacking
- Function Signatures
- `inspect.signature()`
- `functools.partial()`

## *args and **kwargs

### What It Is
`*args` captures additional positional arguments as a tuple; `**kwargs` captures additional keyword arguments as a dict. They allow functions to accept variable numbers of arguments, enabling flexible APIs and argument forwarding.

### Why It Is Important
`*args` and `**kwargs` are essential for decorators, wrapper functions, subclassing (super() calls), and APIs that need flexibility. They enable the open/closed principle — extending function behavior without changing existing callers.

### How It Works Internally
At the bytecode level, `*args` collects the positional arguments beyond the named parameters into a tuple object. `**kwargs` collects keyword arguments beyond the named parameters into a dict. When used in function calls (`func(*args, **kwargs)`), the tuple and dict are unpacked — this is implemented via `BUILD_TUPLE_UNPACK_WITH_CALL` and `BUILD_MAP_UNPACK_WITH_CALL` bytecodes.

### Syntax
```python
# Definition: capturing
def func(*args, **kwargs):
    print(args)    # tuple of positional args
    print(kwargs)  # dict of keyword args

# Call: unpacking
func(*iterable, **mapping)

# Mixed
def func(a, b, *args, **kwargs):
    pass
```

### Beginner Examples
```python
# *args — variable positional arguments
def sum_all(*numbers):
    """Sum any number of arguments."""
    print(f"Received: {numbers}")  # tuple
    return sum(numbers)

print(sum_all(1, 2, 3))        # 6
print(sum_all(10, 20, 30, 40)) # 100
print(sum_all())               # 0

# **kwargs — variable keyword arguments
def print_info(**info):
    """Print key-value pairs."""
    for key, value in info.items():
        print(f"  {key}: {value}")

print_info(name="Alice", age=30, city="NYC")

# Mixing *args and **kwargs
def log(level, *messages, **metadata):
    print(f"[{level}]", *messages)
    for key, value in metadata.items():
        print(f"  {key}={value}")

log("INFO", "Server started", "Port 8080", user="admin")
```

### Intermediate Examples
```python
# Unpacking with * and **
def greet(greeting, name, punctuation="!"):
    return f"{greeting}, {name}{punctuation}"

# Unpack list/tuple
args = ["Hello", "Alice"]
print(greet(*args))  # Hello, Alice!

# Unpack dict
kwargs = {"name": "Bob", "greeting": "Hi", "punctuation": "."}
print(greet(**kwargs))  # Hi, Bob.

# Combined unpacking
def configure(host, port, *, debug=False, ssl=False):
    return f"{host}:{port} (debug={debug}, ssl={ssl})"

config = {"host": "localhost", "port": 8080}
options = {"debug": True}
print(configure(**config, **options))

# Forwarding arguments
def wrapper(*args, **kwargs):
    print("Before")
    result = actual_function(*args, **kwargs)
    print("After")
    return result

# Accepting only **kwargs (keyword-only args)
def create_user(**fields):
    return {"created": fields}

print(create_user(name="Alice", age=30, role="admin"))
```

### Advanced Examples
```python
# Decorator with *args and **kwargs
from functools import wraps
import time

def log_calls(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        args_repr = [repr(a) for a in args]
        kwargs_repr = [f"{k}={v!r}" for k, v in kwargs.items()]
        signature = ", ".join(args_repr + kwargs_repr)
        print(f"Calling {func.__name__}({signature})")
        return func(*args, **kwargs)
    return wrapper

@log_calls
def multiply(a, b, c=1):
    return a * b * c

multiply(2, 3)          # Calling multiply(2, 3)
multiply(2, 3, c=4)     # Calling multiply(2, 3, c=4)

# Super() with *args, **kwargs
class Base:
    def __init__(self, name, **kwargs):
        self.name = name
        for key, value in kwargs.items():
            setattr(self, key, value)

class Extended(Base):
    def __init__(self, *args, **kwargs):
        print(f"Extended.__init__: args={args}, kwargs={kwargs}")
        super().__init__(*args, **kwargs)

obj = Extended("Test", age=30, city="NYC")
print(obj.name)  # Test
print(obj.age)   # 30

# *args unpacking in function calls
def create_menu(*items):
    return {"menu": items, "count": len(items)}

items = ["Pizza", "Pasta", "Salad"]
print(create_menu(*items))  # Unpack list into *args

# **kwargs with defaults merging
def connect(**options):
    defaults = {"host": "localhost", "port": 5432, "dbname": "test"}
    defaults.update(options)
    return defaults

print(connect(host="prod.example.com", dbname="production"))
# {'host': 'prod.example.com', 'port': 5432, 'dbname': 'production'}

# Context variable forwarding
def transaction(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("BEGIN TRANSACTION")
        try:
            result = func(*args, **kwargs)
            print("COMMIT")
            return result
        except Exception:
            print("ROLLBACK")
            raise
    return wrapper

@transaction
def transfer(from_acct, to_acct, amount):
    print(f"Transfer ${amount}: {from_acct} -> {to_acct}")

transfer("A001", "B002", 100.0)
```

### Real-World Use Cases
- **Decorators**: Logging, timing, caching, retry, authentication
- **Subclassing**: Forwarding unknown arguments to parent class
- **Configuration**: Merging default config with user-supplied overrides
- **API Clients**: Flexible HTTP request parameters
- **Plugin Systems**: Dispatching arbitrary arguments to plugins
- **Mocking/Testing**: Capturing all arguments for verification

### Common Mistakes
```python
# Mistake 1: Using *args and **kwargs when explicit parameters are better
def process(*args):  # What should args be?
    pass
# Better:
def process(items: list, verbose: bool = False):
    pass

# Mistake 2: Forgetting that **kwargs captures all keyword args
def func(a, **kwargs):
    pass
# func(a=1, b=2)  # OK — b goes to kwargs
# func(1, 2)      # TypeError — 2 not expected

# Mistake 3: Using *args when you mean to accept a list
def total(*args):
    return sum(args)

# total([1, 2, 3])  # TypeError! (list is one argument)
# Correct:
def total_list(items):
    return sum(items)

# Mistake 4: Mutating **kwargs
def process(**options):
    options.clear()  # Modifies the caller's dict if shared!
    return options

# Mistake 5: Not unpacking *args when calling another function
def caller(*args):
    target(args)  # Passes a single tuple!
    
def target(a, b, c):
    pass

# Correct:
def caller(*args):
    target(*args)  # Unpacks tuple
```

### Best Practices
- Use `*args` for variable-length positional arguments (like `range()`)
- Use `**kwargs` for optional configuration options
- Prefer explicit parameters for required or commonly-used arguments
- Always use `*args, **kwargs` in decorator wrappers
- Document what `*args` and `**kwargs` accept
- Merge `**kwargs` with defaults using `.copy()` or `dict(defaults, **kwargs)`
- Use `**kwargs` sparingly in public APIs (hides the function signature)

### Performance Considerations
- `*args` creates a tuple (small overhead for many arguments)
- `**kwargs` creates a dict (more overhead)
- Unpacking at call site (`func(*args)`) is fast (C-level)
- Using `*args`/`**kwargs` prevents some Python compiler optimizations
- For performance-critical code, use explicit parameters
- Python 3.12+ has optimized `CALL_FUNCTION_EX` for unpacking calls

### Interview Questions
1. What is the difference between `*args` and `**kwargs`?
2. How do you forward arguments from one function to another?
3. Can you use `*args` and `**kwargs` together?
4. What is the naming convention for these special parameters?
5. How does `*` and `**` work in function calls (unpacking)?
6. How do you make all function parameters keyword-only?
7. When should you NOT use `*args` and `**kwargs`?
8. How do you merge `**kwargs` with default values?
9. How do decorators use `*args` and `**kwargs`?
10. Can you use `*args` in the middle of the parameter list?

### Coding Challenges
```python
# Challenge 1: Flexible event emitter
class EventEmitter:
    def __init__(self):
        self._handlers = {}
    
    def on(self, event: str, handler):
        self._handlers.setdefault(event, []).append(handler)
    
    def emit(self, event: str, *args, **kwargs):
        for handler in self._handlers.get(event, []):
            handler(*args, **kwargs)

emitter = EventEmitter()
emitter.on("data", lambda x, y: print(f"Data: {x}, {y}"))
emitter.on("data", lambda *a, **kw: print(f"Received: {a}, {kw}"))
emitter.emit("data", 42, 100, extra="info")

# Challenge 2: Configurable retry decorator
from functools import wraps
import time

def retry(max_attempts: int = 3, **retry_options):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            delay = retry_options.get("delay", 0.1)
            backoff = retry_options.get("backoff", 1.0)
            exceptions = retry_options.get("exceptions", (Exception,))
            
            last_exception = None
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    last_exception = e
                    if attempt < max_attempts - 1:
                        time.sleep(delay * (backoff ** attempt))
            raise last_exception
        return wrapper
    return decorator

@retry(max_attempts=3, delay=0.5, backoff=2.0, exceptions=(ConnectionError,))
def unreliable_call():
    import random
    if random.random() < 0.7:
        raise ConnectionError("Failed")
    return "Success"
```

### Related Topics
- Parameter Types
- Argument Unpacking
- Decorators
- `functools.partial()`
- Function Introspection
- Super() and Cooperative Multiple Inheritance
