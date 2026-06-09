# Try / Except / Else / Finally - Handling exceptions gracefully

## Introduction
Python provides `try/except/else/finally` blocks to catch and handle exceptions at specific points in your code.

## Why It Is Important
- Centralizes error handling logic
- Allows program recovery from predictable failures
- Ensures cleanup code runs (finally)
- Distinguishes successful execution (else) from error recovery (except)

## Syntax
```python
try:
    risky_operation()
except SomeError as e:
    handle_error(e)
else:
    no_error_occurred()
finally:
    always_run_this()
```

## Examples

### Beginner Examples

**Basic try/except:**
```python
try:
    number = int(input("Enter a number: "))
    print(f"You entered {number}")
except ValueError:
    print("That was not a valid number!")
```

**Multiple except blocks:**
```python
try:
    items = [1, 2, 3]
    index = int(input("Index: "))
    print(items[index])
except ValueError:
    print("Please enter a valid integer")
except IndexError:
    print("Index out of range")
```

### Intermediate Examples

**Else and finally:**
```python
def read_file(filename):
    try:
        file = open(filename, "r")
        content = file.read()
    except FileNotFoundError:
        print(f"{filename} not found")
        return None
    except PermissionError:
        print(f"No permission to read {filename}")
        return None
    else:
        print("File read successfully")
        return content
    finally:
        try:
            file.close()
        except NameError:
            pass
```

### Advanced Examples

**Context manager pattern with try/except/finally:**
```python
class DatabaseConnection:
    def __init__(self, conn_string):
        self.conn_string = conn_string
        self.connection = None

    def __enter__(self):
        try:
            self.connection = self._connect()
            return self.connection
        except ConnectionError as e:
            print(f"Connection failed: {e}")
            raise

    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.connection:
            self.connection.close()
        if exc_type is not None:
            print(f"Exception {exc_type} was suppressed")
        return True  # Suppress exceptions

    def _connect(self):
        if "invalid" in self.conn_string:
            raise ConnectionError("Invalid connection string")
        return {"connected": True}

with DatabaseConnection("valid_db") as conn:
    print(conn)
```

## Real-World Use Cases
- Database transactions: rollback on error, commit on success
- File processing: ensure files are closed after reading/writing
- API calls: retry on network timeout, cache on success
- Resource cleanup (sockets, locks, file handles)

## Common Mistakes
- Putting too much code in try block (catch unexpected errors)
- Forgetting that `else` runs only if no exception occurred (not after except)
- Using `return` in finally (overrides previous return value)
- Catching and ignoring exceptions without logging

## Best Practices
- Keep try blocks minimal — only the code that might fail
- Use specific exception types rather than `Exception` or bare `except:`
- Use `else` for code that should run only on success
- Use `finally` for cleanup that must always run
- Never use bare `except:` unless re-raising immediately

## Interview Questions
1. When does `else` run in try/except/else/finally?
2. What happens if both except and finally have return statements?
3. Can you nest try/except blocks? Is it recommended?

## Coding Challenges
1. Implement a safe division function that handles ZeroDivisionError and TypeError.
2. Write a retry mechanism that tries an operation up to 3 times before failing.

## Summary
The try/except/else/finally structure is Python's primary tool for exception handling. Each clause serves a distinct purpose, enabling clean separation of normal code from error handling and cleanup.

## Related Topics
- 43_exceptions.md — what exceptions are and the hierarchy
- 45_custom_exceptions.md — creating application-specific exceptions
