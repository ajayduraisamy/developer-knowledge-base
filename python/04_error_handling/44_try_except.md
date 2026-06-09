# Try / Except / Else / Finally - Handling exceptions gracefully

## Introduction

Python's `try` statement is the primary mechanism for catching and handling exceptions. It provides four distinct clauses—`try`, `except`, `else`, and `finally`—each serving a specific purpose in the error-handling workflow. Together they enable robust error recovery, resource cleanup, and separation of normal logic from error-handling code.

## try block

### What It Is

The `try` block contains code that might raise an exception. Python monitors execution of this block and, if an exception occurs, immediately jumps to the matching `except` block (if any). Code after the point of failure in the `try` block does NOT execute.

### Why It Is Important

The `try` block defines the scope of exception monitoring. It tells Python and other developers which operations are expected to potentially fail. Keeping `try` blocks minimal—only wrapping the code that might raise—is a best practice because it prevents catching exceptions from unrelated code.

### How It Works Internally

When Python executes a `try` block, it sets up a special frame on the interpreter stack that records the exception handler location. If no exception occurs, the frame is popped and execution continues normally. If an exception is raised, the interpreter walks the stack looking for this frame, then jumps to the matching `except` clause. This setup has near-zero performance cost when no exception is raised (a key Python design choice).

### Syntax

```python
try:
    risky_code()
```

### Beginner Examples

```python
# Minimal try block
try:
    number = int(input("Enter a number: "))
    print(f"You entered {number}")
except ValueError:
    print("That was not a valid number!")

# Keep try block focused
# GOOD: only wrap the risky part
try:
    result = risky_operation()
except ValueError:
    result = None

# BAD: wrapping too much code
try:
    result = risky_operation()
    data = process(result)
    save(data)  # If this raises, the except might handle it incorrectly
except ValueError:
    print("Ignoring ValueError from any of the above lines")
```

## except block

### What It Is

The `except` block catches exceptions raised in the `try` block. You can specify which exception types to catch, access the exception object, and define recovery logic. Multiple `except` blocks can be chained to handle different exception types differently.

### Why It Is Important

`except` blocks enable program recovery from predictable failures. Without them, any exception would crash the program. They allow you to degrade gracefully, retry operations, log errors, and provide meaningful feedback to users.

### How It Works Internally

When an exception propagates to a `try` statement, the interpreter examines each `except` clause in order using `isinstance()` checking. The first matching clause executes. If no match is found, the exception continues propagating. Once a matching clause is found, the exception is considered handled and execution continues after the try/except block (or in `finally`/`else`).

### Syntax

```python
try:
    ...
except SomeError:                          # Catch specific exception
    ...
except (ErrorA, ErrorB):                   # Catch multiple types
    ...
except SomeError as e:                     # Bind exception to variable
    ...
except:                                    # Bare except (avoid!)
    ...
```

### Beginner Examples

```python
# Single except
try:
    value = int(input("Age: "))
except ValueError:
    print("Must enter a number")

# Multiple except blocks
try:
    items = [1, 2, 3]
    index = int(input("Index: "))
    print(items[index])
except ValueError:
    print("Invalid index format")
except IndexError:
    print("Index out of range")

# Accessing the exception object
try:
    10 / 0
except ZeroDivisionError as e:
    print(f"Error: {e}")

# Catching multiple exception types
try:
    data = {"key": "value"}
    print(data["nonexistent"])
    print(10 / 0)
except (KeyError, ZeroDivisionError) as e:
    print(f"Caught: {type(e).__name__}: {e}")
```

### Intermediate Examples

```python
# Exception handling with logging
import logging

logger = logging.getLogger(__name__)

try:
    result = process_data(raw_input)
except ValueError as e:
    logger.warning("Bad input data", exc_info=True)
    result = None
except RuntimeError as e:
    logger.error("Processing failed", exc_info=True)
    raise  # Re-raise after logging
except Exception as e:
    logger.critical("Unexpected error", exc_info=True)
    result = None

# Conditional exception handling
try:
    result = api_call()
except HTTPError as e:
    if e.status_code == 429:  # Rate limited
        time.sleep(60)
        result = api_call()
    elif e.status_code >= 500:  # Server error
        result = None
    else:
        raise  # Re-raise unexpected errors
```

## else block

### What It Is

The `else` block executes only if the `try` block completed WITHOUT raising an exception. It runs after the `try` block but before `finally`. This separates "success path" code from the "error path" code in `except`.

### Why It Is Important

`else` makes the code structure clearer by placing success-only logic in its own clause. Without `else`, you would need to put success code at the end of the `try` block, where it could accidentally catch exceptions that should propagate. The `else` block is not protected by the preceding `except` clauses, so exceptions raised in `else` are NOT caught by them.

### How It Works Internally

After the `try` block executes without raising, the interpreter jumps to the `else` clause (if present). If `else` raises an exception, it is not caught by the preceding `except` blocks; it either propagates to an outer handler or crashes the program.

### Syntax

```python
try:
    risky_code()
except SomeError:
    handle_error()
else:
    success_code()  # Only if no exception
```

### Beginner Examples

```python
# else block for success-only logic
try:
    file = open("data.txt", "r")
except FileNotFoundError:
    print("File not found")
else:
    content = file.read()
    print(f"Read {len(content)} bytes")
    file.close()

# Without else (less clear):
try:
    file = open("data.txt", "r")
    content = file.read()  # This could also fail
    file.close()
except FileNotFoundError:
    print("File not found")
```

### Intermediate Examples

```python
# else in database operations
try:
    connection = db.connect()
except ConnectionError:
    print("Could not connect")
else:
    # This only runs on successful connection
    results = connection.query("SELECT * FROM users")
    connection.close()

# else vs putting code in try
try:
    division = a / b
except ZeroDivisionError:
    result = None
else:
    result = division * 2  # Not protected by the except above
```

## finally block

### What It Is

The `finally` block executes ALWAYS, regardless of whether an exception occurred, was caught, or propagated. It is used for cleanup actions: closing files, releasing locks, closing network connections, freeing resources.

### Why It Is Important

`finally` guarantees cleanup code runs even if an exception is unhandled, if a `return`, `break`, or `continue` is executed, or if the program is about to exit. This makes it essential for resource management and preventing resource leaks.

### How It Works Internally

The interpreter establishes the `finally` clause as a cleanup handler. Even if execution is interrupted by an exception, `return`, `break`, or `continue`, the `finally` block executes. If the `finally` block itself raises an exception, it replaces any exception that was being propagated (this is usually undesirable).

### Syntax

```python
try:
    risky_code()
finally:
    cleanup_code()  # Always runs

try:
    risky_code()
except SomeError:
    handle_error()
finally:
    cleanup_code()  # Always runs

try:
    risky_code()
else:
    success_code()
finally:
    cleanup_code()  # Always runs
```

### Beginner Examples

```python
# Basic finally for cleanup
try:
    file = open("data.txt", "r")
    content = file.read()
except FileNotFoundError:
    print("File not found")
finally:
    file.close()  # Always close the file

# finally with return
def example():
    try:
        return "from try"
    finally:
        print("Finally still runs")  # Prints even though try returns

print(example())  # Prints "Finally still runs" then "from try"

# finally without except
def read_with_cleanup(filename):
    file = open(filename)
    try:
        return file.read()
    finally:
        file.close()  # Runs even if read() fails
```

### Intermediate Examples

```python
# finally for resource cleanup
import threading

lock = threading.Lock()

def critical_section():
    lock.acquire()
    try:
        # Critical code that might fail
        process_data()
    finally:
        lock.release()  # Always release the lock

# finally with exception in the block
def example():
    try:
        raise ValueError("original error")
    finally:
        print("Cleanup runs")
        # The ValueError still propagates after finally

# finally overriding return
def confusing():
    try:
        return "try"
    finally:
        return "finally"  # Overrides the try return!

print(confusing())  # "finally"
```

### Advanced Examples

```python
# Complete try/except/else/finally
def load_config(path):
    file = None
    try:
        file = open(path, "r")
        raw = file.read()
    except FileNotFoundError:
        print(f"Config {path} not found, using defaults")
        return DEFAULT_CONFIG
    except PermissionError:
        print(f"No permission to read {path}")
        return DEFAULT_CONFIG
    else:
        # Only runs if no exception occurred in try
        config = parse_config(raw)
        return config
    finally:
        # Always runs - cleanup
        if file:
            file.close()

# Nested try/except patterns
def complex_operation():
    conn = None
    try:
        conn = db_connect()
        try:
            cursor = conn.cursor()
            cursor.execute("UPDATE users SET active=1")
        except DatabaseError:
            conn.rollback()
            raise
        else:
            conn.commit()
    finally:
        if conn:
            conn.close()

# Context manager using try/finally
class ManagedResource:
    def __enter__(self):
        print("Acquiring resource")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("Releasing resource")
        if exc_type is not None:
            print(f"Exception was: {exc_type.__name__}: {exc_val}")
        return False  # Don't suppress exceptions
```

### Real-World Use Cases

```python
# Transaction rollback pattern
def transfer_funds(from_acct, to_acct, amount):
    try:
        from_acct.withdraw(amount)
        to_acct.deposit(amount)
    except InsufficientFunds:
        raise
    except Exception as e:
        from_acct.rollback()
        to_acct.rollback()
        raise TransactionError("Transfer failed") from e
    else:
        from_acct.commit()
        to_acct.commit()

# Graceful shutdown
def run_server():
    server = start_server()
    try:
        server.serve_forever()
    except KeyboardInterrupt:
        print("Shutting down...")
    finally:
        server.stop()
        server.cleanup()

# Retry with cleanup
def fetch_with_retry(url, retries=3):
    for attempt in range(retries):
        conn = None
        try:
            conn = urlopen(url)
            return conn.read()
        except (ConnectionError, TimeoutError) as e:
            if attempt == retries - 1:
                raise
            time.sleep(2 ** attempt)
        finally:
            if conn:
                conn.close()
```

### Common Mistakes

```python
# Mistake 1: Putting too much code in try
try:
    data = load_data()
    result = process(data)  # Should NOT be here - not the risky part
    save(result)
except FileNotFoundError:
    handle_missing_file()

# Mistake 2: return in finally overrides other returns
def bad():
    try:
        return 1
    finally:
        return 2  # Returns 2, never 1!

# Mistake 3: Swallowing exceptions
try:
    do_something()
except Exception:
    pass  # This hides every error!

# Mistake 4: Not specifying exception types
try:
    do_something()
except:  # Bare except catches SystemExit, KeyboardInterrupt
    pass

# Mistake 5: Catching too broad after specific
try:
    process()
except Exception:  # This catches everything
    pass
except ValueError:  # Dead code - never reached
    pass

# Mistake 6: Assuming code after except doesn't run
try:
    x = 1 / 0
except ZeroDivisionError:
    x = 0
print(x)  # Works fine - exception was handled

# Mistake 7: Forgetting that else is NOT protected
try:
    x = int("5")
except ValueError:
    pass
else:
    y = x / 0  # This ZeroDivisionError is NOT caught!
```

### Best Practices

- Keep `try` blocks minimal—only wrap code expected to raise
- Catch specific exception types, never bare `except:`
- Order `except` clauses from most specific to most general
- Use `else` for success-only code to avoid accidentally catching unrelated exceptions
- Use `finally` for resource cleanup that must always run
- Never use `return` in `finally` unless you intend to override
- Prefer context managers (`with` statement) over try/finally for resource management
- Log exceptions with `logger.exception()` or `exc_info=True` before handling
- Re-raise exceptions you cannot handle (`raise` without arguments)
- Use `raise ... from e` for exception chaining when translating exceptions

### Performance Considerations

The `try` statement has negligible overhead when no exception occurs—Python only sets up a frame on the stack, which is very cheap. The cost comes when an exception IS raised: traceback construction, stack unwinding, and exception handler lookup. For this reason, exceptions should not be used for regular control flow in performance-critical code. In Python, the common idiom "It's easier to ask for forgiveness than permission" (EAFP) using try/except is often faster than "Look before you leap" (LBYL) with if/else, because the common case (no exception) is fast, and the if/else approach might double the number of operations.

### Interview Questions

1. What is the difference between `else` and `finally`?

   `else` runs only if no exception occurred. `finally` always runs, regardless of exceptions. `else` is for success-only code; `finally` is for cleanup.

2. Can you have multiple `except` blocks?

   Yes. They are checked in order and the first matching one executes. Order specific exceptions first, general ones last.

3. What happens if an exception is raised in `finally`?

   It replaces any exception that was being propagated. The original exception is lost unless you capture it and chain it.

4. What is the purpose of the `else` clause?

   To separate success-path code from error-handling code. Code in `else` is not protected by the preceding `except` blocks, so if it raises a different exception, that exception propagates normally.

5. How do you create a context manager using try/finally?

   Implement `__enter__` and `__exit__` methods. In `__exit__`, use try/finally to ensure cleanup happens even if the `__exit__` method itself encounters an error.

6. What happens if both the `try` and `finally` blocks have `return` statements?

   The `finally` return wins. The `try` block's return value is discarded. This is almost always a bug.

### Coding Challenges

1. Implement a `retry` decorator using try/except that retries an operation up to N times with exponential backoff.

2. Write a context manager `DatabaseTransaction` that uses try/except to commit on success and rollback on failure.

3. Create a safe file reader that tries multiple encodings (utf-8, latin-1, cp1252) using nested try/except blocks.

4. Implement a `suppress` context manager that selectively suppresses specified exception types (like `contextlib.suppress`).

5. Write a function that processes a list of items, catching exceptions per-item so one failure doesn't stop the entire batch.

### Related Topics

- `43_exceptions.md` - The raise statement and exception hierarchy
- `45_custom_exceptions.md` - Creating custom exception types
- `46_logging.md` - Logging exceptions during handling
- `47_assertions.md` - Assertions vs exceptions for debugging
