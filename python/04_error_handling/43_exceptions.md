# Exceptions - raise, exception hierarchy, built-in exceptions

## Introduction
An exception is an event that interrupts the normal flow of a program. When Python encounters an error it cannot handle, it raises an exception. If not caught, the program crashes.

## Why It Is Important
- Prevents programs from crashing unexpectedly
- Separates error-handling code from main logic
- Enables graceful degradation and recovery
- Essential for robust production systems

## Syntax
```python
raise ValueError("invalid value")
```

## Examples

### Beginner Examples

**Built-in exceptions:**
```python
# ValueError
int("abc")  # Raises ValueError

# TypeError
"hello" + 5  # Raises TypeError

# IndexError
items = [1, 2]
items[5]  # Raises IndexError

# KeyError
data = {"a": 1}
data["b"]  # Raises KeyError

# ZeroDivisionError
10 / 0  # Raises ZeroDivisionError
```

### Intermediate Examples

**Raising exceptions with custom messages:**
```python
def withdraw(balance, amount):
    if amount > balance:
        raise ValueError(
            f"Insufficient funds: have ${balance}, need ${amount}"
        )
    return balance - amount

try:
    withdraw(100, 200)
except ValueError as e:
    print(e)
```

### Advanced Examples

**Exception hierarchy and chaining:**
```python
# Python 3.11+ exception groups
try:
    raise ExceptionGroup("group",
        [ValueError("bad value"),
         TypeError("bad type")])
except* ValueError as e:
    print(f"ValueErrors: {e.exceptions}")
except* TypeError as e:
    print(f"TypeErrors: {e.exceptions}")
```

## Real-World Use Cases
- Validating user input and raising appropriate errors
- Retry logic when API calls fail
- Graceful shutdown on file I/O errors
- Custom domain exceptions in business logic

## Common Mistakes
- Catching too broad an exception (bare `except:`)
- Swallowing exceptions silently
- Raising `Exception` instead of a specific type

## Best Practices
- Catch specific exception types, never bare `except:`
- Use `finally` for cleanup operations
- Use `raise ... from e` for exception chaining
- Create custom exception hierarchies for your application

## Interview Questions
1. What is the difference between `BaseException` and `Exception`?
2. How does exception chaining work with `raise ... from`?
3. What happens if a `finally` block raises an exception?

## Coding Challenges
1. Implement a retry decorator that catches exceptions and retries n times.
2. Write a safe division function with comprehensive error handling.

## Summary
Exceptions are Python's mechanism for error handling. Understanding the exception hierarchy and proper handling patterns is critical for writing robust code.

## Related Topics
- 44_try_except.md — the try/except/else/finally blocks
- 45_custom_exceptions.md — creating your own exception types
