# Exceptions - raise, exception hierarchy, built-in exceptions

## Introduction

Exceptions are Python's mechanism for handling errors and other exceptional events that occur during program execution. When an error occurs, Python creates an exception object and raises it. If not caught, the exception propagates up the call stack until it is handled or the program terminates. Python's exception system is built around a hierarchy of exception classes, starting with `BaseException` at the root. Understanding how to raise, catch, and leverage the exception hierarchy is fundamental to writing robust, maintainable Python code.

## raise statement

### What It Is

The `raise` statement explicitly triggers an exception in Python. It can be used to raise built-in exceptions, custom exceptions, or re-raise caught exceptions. When `raise` is executed, normal program flow stops and the exception propagates up the call stack.

### Why It Is Important

`raise` gives developers the ability to signal error conditions explicitly. Instead of returning error codes or `None` values that must be checked by callers, `raise` produces an immediate, unambiguous signal that something went wrong. This leads to cleaner code because error handling is separated from normal logic, and callers cannot accidentally ignore errors.

### How It Works Internally

When `raise` executes, Python creates an instance of an exception class and begins unwinding the call stack. Each frame in the stack is examined for an exception handler (`except` block). If none is found, the program terminates with a traceback. The interpreter maintains a chain of exceptions through `__cause__` (set by `raise ... from`) and `__context__` (set automatically when an exception is raised while handling another). This chain is displayed in the traceback and aids debugging.

### Syntax

```python
raise SomeException("message")       # Raise a new exception
raise                                 # Re-raise the current exception
raise SomeException from cause        # Chain exceptions with explicit cause
raise SomeException from None         # Suppress exception context
```

```python
# Basic raise
raise ValueError("invalid value")

# Re-raise in except block
try:
    risky_operation()
except SomeError:
    log_error()
    raise

# Exception chaining
try:
    parse_data(raw)
except ParseError as e:
    raise DataError("Failed to parse") from e

# Suppress context
try:
    do_something()
except Exception:
    raise ValueError("Something went wrong") from None
```

### Beginner Examples

```python
# Example 1: Simple raise with built-in exception
def divide(a, b):
    if b == 0:
        raise ZeroDivisionError("Cannot divide by zero")
    return a / b

# Example 2: Raise different exceptions based on condition
def get_item(items, index):
    if not isinstance(index, int):
        raise TypeError(f"Index must be int, got {type(index).__name__}")
    if index < 0 or index >= len(items):
        raise IndexError(f"Index {index} out of range for list of length {len(items)}")
    return items[index]

# Example 3: Using raise to signal invalid state
class BankAccount:
    def __init__(self, balance):
        if balance < 0:
            raise ValueError(f"Initial balance cannot be negative: {balance}")
        self.balance = balance

    def withdraw(self, amount):
        if amount <= 0:
            raise ValueError(f"Withdrawal amount must be positive: {amount}")
        if amount > self.balance:
            raise ValueError(f"Insufficient funds: have {self.balance}, need {amount}")
        self.balance -= amount
        return self.balance
```

### Intermediate Examples

```python
# Example 1: Exception chaining with from
import json

def load_config(path):
    try:
        with open(path) as f:
            return json.load(f)
    except FileNotFoundError as e:
        raise RuntimeError(f"Config file not found: {path}") from e
    except json.JSONDecodeError as e:
        raise RuntimeError(f"Invalid JSON in config: {path}") from e

# Example 2: Re-raising with additional context
def process_transaction(txn):
    try:
        validate(txn)
        execute(txn)
    except ValidationError:
        raise  # Re-raise as-is
    except Exception as e:
        raise RuntimeError(f"Transaction {txn.id} failed") from e

# Example 3: raise from None to hide internal details
def api_call():
    try:
        return internal_api()
    except InternalError:
        raise ApiError("Service unavailable") from None

# Example 4: Conditional re-raise
def safe_operation():
    try:
        do_risky_thing()
    except TemporaryError:
        time.sleep(1)
        return retry()
    except PermanentError:
        raise  # Don't retry permanent errors
```

### Advanced Examples

```python
# Example 1: Exception group with raise* (Python 3.11+)
try:
    raise ExceptionGroup("validation errors", [
        ValueError("email is invalid"),
        TypeError("age must be int"),
        KeyError("missing field: name"),
    ])
except* ValueError as e:
    print(f"ValueErrors: {e.exceptions}")
except* TypeError as e:
    print(f"TypeErrors: {e.exceptions}")
except* Exception as e:
    print(f"Other: {e.exceptions}")

# Example 2: Context manager that transforms exceptions
from contextlib import contextmanager

@contextmanager
def api_error_wrapper():
    try:
        yield
    except ConnectionError as e:
        raise ApiError("API connection failed") from e
    except TimeoutError as e:
        raise ApiError("API timed out") from e

with api_error_wrapper():
    call_external_api()

# Example 3: Dynamic exception creation
def raise_with_extra(type_, message, **extra):
    exc = type_(message)
    for k, v in extra.items():
        setattr(exc, k, v)
    raise exc

try:
    raise_with_extra(ValueError, "bad input", code=42, field="username")
except ValueError as e:
    print(e.code, e.field)  # 42 username

# Example 4: Exception suppression pattern
import sys

class Suppressor:
    def __init__(self, *exc_types):
        self.exc_types = exc_types

    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type is not None and issubclass(exc_type, self.exc_types):
            return True  # Suppress
        return False

with Suppressor(ValueError, TypeError):
    int("not a number")  # Silently ignored
```

### Real-World Use Cases

```python
# Retry decorator
import time
import functools

def retry(max_attempts=3, delay=1, retry_on=(ConnectionError, TimeoutError)):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            last_exception = None
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except retry_on as e:
                    last_exception = e
                    if attempt < max_attempts - 1:
                        time.sleep(delay * (attempt + 1))
            raise last_exception
        return wrapper
    return decorator

# Validation pattern
def validate_user(data):
    errors = []
    if not data.get("email"):
        errors.append(ValueError("Email is required"))
    if not data.get("age"):
        errors.append(ValueError("Age is required"))
    elif data["age"] < 0:
        errors.append(ValueError("Age cannot be negative"))
    if errors:
        raise ExceptionGroup("Validation failed", errors)
    return data
```

### Common Mistakes

```python
# Mistake 1: Raising Exception instead of specific type
raise Exception("something broke")  # Too vague

# Mistake 2: Swallowing exceptions silently
try:
    do_something()
except Exception:
    pass  # Never do this

# Mistake 3: Losing traceback information
try:
    do_something()
except Exception as e:
    raise RuntimeError("Failed")  # Loses original traceback
# Correct:
    raise RuntimeError("Failed") from e

# Mistake 4: Using raise with wrong syntax
raise "error"  # TypeError: exceptions must derive from BaseException
raise ValueError  # Missing parentheses - raises with no message

# Mistake 5: Raising in finally block (overrides prior exception)
try:
    raise ValueError("original")
finally:
    raise RuntimeError("override")  # ValueError is lost
```

### Best Practices

- Raise specific exception types, never bare `Exception` or `BaseException`
- Always provide descriptive error messages with relevant context
- Use `raise ... from e` to chain exceptions and preserve traceback
- Use `raise ... from None` to hide implementation details from API consumers
- Prefer raising exceptions over returning error codes or `None`
- Document which exceptions your functions can raise
- Use `raise` (no arguments) inside `except` blocks to re-raise the current exception
- Avoid raising exceptions in `finally` blocks unless you intend to override

### Performance Considerations

Raising exceptions is relatively expensive compared to normal control flow because Python must build the traceback and unwind the stack. In performance-critical code, use exceptions for exceptional conditions, not for regular control flow. The overhead includes memory allocation for the exception object, traceback construction, and stack unwinding. Python's exception handling is optimized so that `try` blocks have zero cost when no exception is raised, but the `raise` itself has significant cost.

## Exception hierarchy

### What It Is

Python's exception hierarchy is a tree of classes rooted at `BaseException`. All built-in exceptions inherit from `BaseException`, which has two main branches: `Exception` (for program errors) and `SystemExit`, `KeyboardInterrupt`, `GeneratorExit` (for system events). Most user-defined exceptions should inherit from `Exception`.

### Why It Is Important

The hierarchy enables catch specificity: handlers can catch a broad category (like `OSError`) or a specific type (like `FileNotFoundError`). This granularity allows code to handle different error conditions differently while still being able to catch groups of related errors. Understanding where your exception types fit in the hierarchy ensures that `except` clauses behave predictably.

### How It Works Internally

`except` clauses use `isinstance()` checking against the exception hierarchy. A clause `except OSError` catches anything that is an instance of `OSError` or any of its subclasses. The interpreter walks `except` clauses in order and uses the first matching one. This means more specific exceptions must be caught before more general ones.

### Syntax

```
BaseException
 +-- SystemExit
 +-- KeyboardInterrupt
 +-- GeneratorExit
 +-- Exception
      +-- StopIteration
      +-- StopAsyncIteration
      +-- ArithmeticError
      |    +-- FloatingPointError
      |    +-- OverflowError
      |    +-- ZeroDivisionError
      +-- AssertionError
      +-- AttributeError
      +-- BufferError
      +-- EOFError
      +-- ImportError
      |    +-- ModuleNotFoundError
      +-- LookupError
      |    +-- IndexError
      |    +-- KeyError
      +-- MemoryError
      +-- NameError
      |    +-- UnboundLocalError
      +-- OSError
      |    +-- BlockingIOError
      |    +-- ChildProcessError
      |    +-- ConnectionError
      |    |    +-- BrokenPipeError
      |    |    +-- ConnectionAbortedError
      |    |    +-- ConnectionRefusedError
      |    |    +-- ConnectionResetError
      |    +-- FileExistsError
      |    +-- FileNotFoundError
      |    +-- InterruptedError
      |    +-- IsADirectoryError
      |    +-- NotADirectoryError
      |    +-- PermissionError
      |    +-- ProcessLookupError
      |    +-- TimeoutError
      +-- ReferenceError
      +-- RuntimeError
      |    +-- NotImplementedError
      |    +-- RecursionError
      +-- SyntaxError
      |    +-- IndentationError
      |         +-- TabError
      +-- SystemError
      +-- TypeError
      +-- ValueError
      |    +-- UnicodeError
      |         +-- UnicodeDecodeError
      |         +-- UnicodeEncodeError
      |         +-- UnicodeTranslateError
      +-- Warning
           +-- DeprecationWarning
           +-- PendingDeprecationWarning
           +-- RuntimeWarning
           +-- SyntaxWarning
           +-- UserWarning
           +-- FutureWarning
           +-- ImportWarning
           +-- UnicodeWarning
           +-- BytesWarning
           +-- ResourceWarning
```

```python
# Understanding the hierarchy
print(issubclass(FileNotFoundError, OSError))  # True
print(issubclass(FileNotFoundError, Exception))  # True
print(issubclass(FileNotFoundError, BaseException))  # True
print(issubclass(FileNotFoundError, LookupError))  # False

# Catching at different levels
try:
    open("nonexistent.txt")
except FileNotFoundError:
    print("Specific: file not found")
except OSError:
    print("Broader: OS error")
except Exception:
    print("Broadest: any exception")
```

### Best Practices

- Never catch `BaseException` unless you have an extremely specific reason (it catches `SystemExit` and `KeyboardInterrupt`)
- Catch the most specific exception type that makes sense
- Order `except` clauses from most specific to most general
- Create custom exceptions that inherit from specific branches when appropriate
- Use `Exception` as a catch-all for logging only, not for recovery

## Built-in exceptions

### What It Is

Python provides dozens of built-in exception classes covering common error categories. These are organized in the hierarchy described above and are available in the built-in namespace without any imports. They cover I/O errors, arithmetic errors, lookup errors, type errors, and many more.

### Why It Is Important

Built-in exceptions provide a standardized vocabulary for error conditions. When a function raises `ValueError`, every Python programmer immediately understands that the argument has the right type but an inappropriate value. This shared vocabulary makes code more readable and maintainable. Using the appropriate built-in exception also ensures that standard library and third-party code can handle your errors appropriately.

### How It Works Internally

Built-in exceptions are implemented in C in CPython, making them fast. They are simple subclasses of the hierarchy classes and most carry just an error message (`args[0]`). Some have additional attributes: `OSError` has `errno`, `strerror`, `filename`; `StopIteration` has `value`; `UnicodeError` has `encoding`, `reason`, `start`, `end`. These extra attributes provide rich diagnostic information.

### Syntax

```python
# Common built-in exceptions
raise ValueError("bad value")            # Argument has wrong value
raise TypeError("expected int")          # Argument has wrong type
raise IndexError("list index out of range")  # Sequence index out of bounds
raise KeyError("key not found")          # Dictionary key not found
raise FileNotFoundError("no such file")  # File not found (OSError subclass)
raise ZeroDivisionError("division by zero")  # Arithmetic: division by zero
raise AttributeError("no such attribute")    # Object has no such attribute
raise ImportError("module not found")    # Import failed
raise StopIteration                      # Iterator exhausted
raise RuntimeError("something broke")    # Generic runtime error
raise NotImplementedError("subclass must implement")  # Abstract method
```

### Beginner Examples

```python
# ValueError: function argument has correct type but invalid value
int("hello")           # ValueError: invalid literal for int()
float("abc")           # ValueError: could not convert string to float
"hello".index("z")     # ValueError: substring not found

# TypeError: operation applied to wrong type
"hello" + 5            # TypeError: can only concatenate str (not "int") to str
len(42)                # TypeError: object of type 'int' has no len()
[1, 2][1.5]            # TypeError: list indices must be integers or slices, not float

# IndexError: sequence index out of range
[1, 2, 3][10]          # IndexError: list index out of range
"hello"[10]            # IndexError: string index out of range

# KeyError: dictionary key not found
{"a": 1}["b"]          # KeyError: 'b'

# ZeroDivisionError: division or modulo by zero
10 / 0                 # ZeroDivisionError: division by zero
10 % 0                 # ZeroDivisionError: integer modulo by zero

# FileNotFoundError: file does not exist
open("nonexistent.txt")  # FileNotFoundError

# AttributeError: object has no attribute
None.len()             # AttributeError: 'NoneType' object has no attribute 'len'

# ImportError: module not found
import nonexistent_module  # ModuleNotFoundError (subclass of ImportError)
```

### Intermediate Examples

```python
# Handling multiple exception types
def safe_divide(a, b):
    try:
        return a / b
    except ZeroDivisionError:
        return float("inf")
    except TypeError:
        return None

# Using exception attributes
try:
    open("/root/secret.txt")
except PermissionError as e:
    print(f"No permission: {e.errno} - {e.strerror}")
except FileNotFoundError as e:
    print(f"Not found: {e.filename}")

# OSError with rich attributes
import errno
try:
    os.mkdir("/existing/path")
except OSError as e:
    if e.errno == errno.EEXIST:
        print("Directory already exists")
    elif e.errno == errno.EACCES:
        print("Permission denied")
```

### Advanced Examples

```python
# Using StopIteration in generator
def take(n, iterable):
    it = iter(iterable)
    result = []
    for _ in range(n):
        try:
            result.append(next(it))
        except StopIteration:
            break
    return result

# NotImplementedError for abstract classes
class BaseProcessor:
    def process(self, data):
        raise NotImplementedError("Subclasses must implement process()")

class CSVProcessor(BaseProcessor):
    def process(self, data):
        return data.split(",")

# UnicodeError handling
try:
    value = b"\xff\xfe".decode("utf-8")
except UnicodeDecodeError as e:
    print(f"Encoding: {e.encoding}, position: {e.start}-{e.end}")
    print(f"Bad bytes: {e.object[e.start:e.end]}")

# RuntimeError and RecursionError
import sys
sys.setrecursionlimit(100)
def recurse(n):
    return recurse(n + 1)
try:
    recurse(0)
except RecursionError:
    print("Hit recursion limit")

# MemoryError (rare in practice - Python usually OOMs)
try:
    data = bytearray(10**12)
except MemoryError:
    print("Not enough memory")
```

### Real-World Use Cases

```python
# Web scraping with HTTPError handling
import urllib.request
import urllib.error

def fetch_url(url):
    try:
        with urllib.request.urlopen(url) as response:
            return response.read()
    except urllib.error.HTTPError as e:
        if e.code == 404:
            return None
        elif e.code == 403:
            raise PermissionError(f"Access denied to {url}")
        else:
            raise
    except urllib.error.URLError as e:
        raise ConnectionError(f"Failed to reach {url}") from e

# Database operations with exception mapping
class DatabaseError(Exception): pass
class ConnectionError(DatabaseError): pass
class QueryError(DatabaseError): pass

def execute_query(conn, sql):
    try:
        return conn.execute(sql)
    except conn.OperationalError as e:
        if "connection" in str(e).lower():
            raise ConnectionError("Database connection lost") from e
        raise QueryError(f"Query failed: {sql}") from e
```

### Common Mistakes

```python
# Mistake 1: Using wrong exception type
raise Exception("User not found")  # Too generic, should be KeyError or LookupError
raise RuntimeError("File not found")  # Should be FileNotFoundError

# Mistake 2: Catching too broad
try:
    do_something()
except Exception:  # Catches everything including KeyboardInterrupt
    log()

# Mistake 3: Not using exception subclasses properly
try:
    open(filename)
except OSError:  # Catches all OS errors
    pass
# Better: catch specific subclass
try:
    open(filename)
except FileNotFoundError:
    pass

# Mistake 4: Silent catch with bare except
try:
    process()
except:
    pass  # Catches SystemExit, KeyboardInterrupt, everything
```

### Best Practices

- Match the built-in exception to the error semantics precisely
- Use `ValueError` for invalid argument values, `TypeError` for wrong types
- Use `KeyError` for missing dictionary keys, `IndexError` for sequence bounds
- Use `OSError` subclasses for system call failures
- Use `StopIteration` for generator exhaustion
- Use `NotImplementedError` for abstract methods
- Prefer built-in exceptions over custom ones when they fit
- Document what exceptions your functions may raise

### Performance Considerations

Built-in exceptions are implemented in C and are the fastest exceptions to raise. Custom Python exceptions add a small overhead for class creation and method resolution but this is negligible in practice. The main cost is always the stack unwinding and traceback construction, which is the same regardless of exception type.

### Interview Questions

1. What is the root of the Python exception hierarchy?
   `BaseException`. Everything inherits from it. You should catch `Exception` (not `BaseException`) to avoid catching `SystemExit` and `KeyboardInterrupt`.

2. What is the difference between `ValueError` and `TypeError`?
   `TypeError` means the argument has the wrong type (e.g., passing a string where an int is expected). `ValueError` means the argument has the right type but an invalid value (e.g., `int("hello")`).

3. What does `raise` without arguments do?
   It re-raises the current exception in an `except` block. It preserves the original traceback.

4. How does exception chaining with `raise ... from` work?
   `raise X from Y` sets the `__cause__` attribute of X to Y, creating an explicit chaining that Python displays in the traceback as "The above exception was the direct cause of the following exception".

5. What is the difference between `raise X from Y` and just `raise X` inside an `except` block?
   Inside an `except` block, `raise X` implicitly sets `__context__` to the original exception (implicit chaining). `raise X from Y` sets `__cause__` explicitly. `from None` suppresses the context.

6. What is `ExceptionGroup` and when was it introduced?
   `ExceptionGroup` was introduced in Python 3.11. It allows raising and catching multiple unrelated exceptions simultaneously using `except*` syntax.

### Coding Challenges

1. Implement a `safe_execute` function that takes a function and arguments, catches any exception, and returns either the result or an exception object.

2. Write a decorator that catches specific exceptions and converts them to API-friendly error responses with status codes.

3. Implement a retry mechanism with exponential backoff that handles `ConnectionError`, `TimeoutError`, and `RateLimitError` differently.

4. Create a `ExceptionCollector` context manager that collects all exceptions raised inside the block and raises them as an `ExceptionGroup` at the end.

5. Implement a validation function that takes a schema and data, collects all validation errors, and raises them together using exception groups.

### Related Topics

- `44_try_except.md` - Catching and handling exceptions with try/except/else/finally
- `45_custom_exceptions.md` - Creating custom exception hierarchies
- `46_logging.md` - Logging exceptions for debugging and monitoring
- `47_assertions.md` - Assertions for debugging and contract checking
