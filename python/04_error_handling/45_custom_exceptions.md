# Custom Exceptions - class MyError(Exception), exception hierarchies

## Introduction

Custom exceptions are user-defined exception classes that inherit from Python's built-in `Exception` class (or any of its subclasses). While Python provides a rich set of built-in exceptions (ValueError, TypeError, KeyError, etc.), custom exceptions allow developers to create domain-specific error types that precisely describe problems occurring in their applications. They form the backbone of robust error handling strategies in large-scale Python projects by making error handling more expressive, granular, and maintainable.

## Why It Is Important

Custom exceptions are critical for building maintainable and self-documenting code. They enable precise error catching—instead of catching a generic `Exception` and inspecting its message, you can catch a specific custom type. This improves code clarity, allows hierarchical error handling, and makes APIs more predictable for consumers. In library and framework development, custom exceptions are the standard way to communicate error conditions to users without exposing internal implementation details. They also facilitate better debugging by attaching extra context (attributes) that help diagnose root causes faster.

## Syntax

Creating a custom exception is as simple as defining a class that inherits from `Exception`:

```python
class MyCustomError(Exception):
    pass
```

You can also inherit from more specific built-in exceptions:

```python
class MyValueError(ValueError):
    pass
```

For richer exceptions with extra attributes:

```python
class DetailedError(Exception):
    def __init__(self, message, code, payload=None):
        self.code = code
        self.payload = payload
        super().__init__(message)
```

Custom exceptions are raised and caught just like built-in exceptions:

```python
raise MyCustomError("Something went wrong")

try:
    raise DetailedError("Invalid state", 42, {"key": "value"})
except DetailedError as e:
    print(e.code)
```

## Examples

```python
# Example 1: Minimal custom exception
class ValidationError(Exception):
    pass


def validate_age(age):
    if age < 0:
        raise ValidationError(f"Age cannot be negative: {age}")
    if age > 150:
        raise ValidationError(f"Age seems unrealistic: {age}")
    return True


try:
    validate_age(-5)
except ValidationError as e:
    print(f"Validation failed: {e}")


# Example 2: Custom exception with attributes
class ApiError(Exception):
    def __init__(self, message, status_code, response_body=None):
        self.status_code = status_code
        self.response_body = response_body
        super().__init__(message)


def call_api(url):
    raise ApiError("Not Found", 404, {"error": "resource_missing"})


try:
    call_api("/users/999")
except ApiError as e:
    print(f"API call failed with status {e.status_code}: {e}")


# Example 3: Custom exception hierarchy
class DatabaseError(Exception):
    pass


class ConnectionError(DatabaseError):
    pass


class QueryError(DatabaseError):
    pass


class TimeoutError(QueryError):
    pass


def execute_query(query, timeout=30):
    if timeout > 60:
        raise TimeoutError(f"Query timed out after {timeout}s", query=query)
    if not query.startswith("SELECT"):
        raise QueryError("Only SELECT queries are allowed")
    return ["result1", "result2"]


try:
    execute_query("DELETE FROM users", timeout=90)
except TimeoutError as e:
    print(f"Timeout: {e}")
except QueryError as e:
    print(f"Query problem: {e}")
except DatabaseError as e:
    print(f"General database error: {e}")
```

## Beginner Examples

```python
# Beginner Example 1: Simple custom exception for input validation
class NegativeNumberError(Exception):
    pass


def square_root(value):
    if value < 0:
        raise NegativeNumberError("Cannot compute square root of a negative number")
    return value ** 0.5


try:
    result = square_root(-9)
except NegativeNumberError as e:
    print(f"Error: {e}")


# Beginner Example 2: Custom exception with a message
class EmptyListError(Exception):
    pass


def average(numbers):
    if not numbers:
        raise EmptyListError("Cannot compute average of an empty list")
    return sum(numbers) / len(numbers)


try:
    avg = average([])
except EmptyListError as e:
    print(f"Error: {e}")


# Beginner Example 3: Using pass for a minimal custom exception
class InsufficientFundsError(Exception):
    pass


class BankAccount:
    def __init__(self, balance=0):
        self.balance = balance

    def withdraw(self, amount):
        if amount > self.balance:
            raise InsufficientFundsError(
                f"Cannot withdraw {amount}, balance is {self.balance}"
            )
        self.balance -= amount
        return self.balance


account = BankAccount(100)
try:
    account.withdraw(200)
except InsufficientFundsError as e:
    print(f"Withdrawal failed: {e}")


# Beginner Example 4: Catching multiple custom exceptions
class TooHotError(Exception):
    pass


class TooColdError(Exception):
    pass


def check_temperature(temp):
    if temp > 40:
        raise TooHotError(f"Temperature {temp}°C is too hot")
    if temp < 0:
        raise TooColdError(f"Temperature {temp}°C is too cold")
    return "Temperature is acceptable"


try:
    check_temperature(50)
except TooHotError as e:
    print(f"Hot: {e}")
except TooColdError as e:
    print(f"Cold: {e}")
```

## Intermediate Examples

```python
# Intermediate Example 1: Exception with multiple attributes and __str__
class HttpError(Exception):
    def __init__(self, message, status_code, headers=None):
        self.status_code = status_code
        self.headers = headers or {}
        super().__init__(message)

    def __str__(self):
        return f"[{self.status_code}] {self.args[0]}"


class NotFoundError(HttpError):
    def __init__(self, resource, resource_id):
        self.resource = resource
        self.resource_id = resource_id
        super().__init__(
            f"{resource} with id {resource_id} not found",
            404
        )


try:
    raise NotFoundError("User", 42)
except NotFoundError as e:
    print(f"Status: {e.status_code}, Resource: {e.resource}, ID: {e.resource_id}")
    print(f"String: {e}")


# Intermediate Example 2: Exception hierarchy with try/except grouping
class PaymentError(Exception):
    pass


class InsufficientBalanceError(PaymentError):
    def __init__(self, balance, required, currency="USD"):
        self.balance = balance
        self.required = required
        self.currency = currency
        self.shortfall = required - balance
        super().__init__(
            f"Insufficient balance: have {balance} {currency}, "
            f"need {required} {currency}"
        )


class CardDeclinedError(PaymentError):
    def __init__(self, card_last_four, reason):
        self.card_last_four = card_last_four
        self.reason = reason
        super().__init__(f"Card ending in {card_last_four} declined: {reason}")


class FraudSuspicionError(PaymentError):
    def __init__(self, transaction_id):
        self.transaction_id = transaction_id
        super().__init__(f"Transaction {transaction_id} flagged for fraud review")


def process_payment(amount, balance, card_last_four):
    if amount > balance:
        raise InsufficientBalanceError(balance, amount)
    if card_last_four == "0000":
        raise CardDeclinedError(card_last_four, "card expired")
    if amount > 10000:
        raise FraudSuspicionError("TXN-99999")
    return "Payment successful"


try:
    process_payment(500, 100, "1234")
except InsufficientBalanceError as e:
    print(f"Need more money: short by {e.shortfall} {e.currency}")
except CardDeclinedError as e:
    print(f"Card issue: {e.reason}")
except FraudSuspicionError as e:
    print(f"Fraud check needed for {e.transaction_id}")
except PaymentError as e:
    print(f"Other payment problem: {e}")


# Intermediate Example 3: Adding context with exception chaining
class DataProcessingError(Exception):
    pass


def load_data(filepath):
    try:
        with open(filepath, "r") as f:
            return f.read()
    except FileNotFoundError as e:
        raise DataProcessingError(f"Cannot load data from {filepath}") from e


def parse_data(content):
    if not content.strip():
        raise DataProcessingError("Empty data content")
    try:
        return int(content)
    except ValueError as e:
        raise DataProcessingError("Failed to parse content as integer") from e


try:
    result = parse_data(load_data("nonexistent.txt"))
except DataProcessingError as e:
    print(f"Data processing failed: {e}")
    print(f"Caused by: {e.__cause__}")


# Intermediate Example 4: Using __init_subclass__ for registry pattern
class RegisteredError(Exception):
    _registry = {}

    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        RegisteredError._registry[cls.__name__] = cls

    @classmethod
    def from_name(cls, name, message=""):
        if name in cls._registry:
            return cls._registry[name](message)
        return UnknownError(f"Unknown error type: {name}")


class NetworkError(RegisteredError):
    pass


class ConfigError(RegisteredError):
    pass


class UnknownError(Exception):
    pass


print(RegisteredError._registry)
err = RegisteredError.from_name("NetworkError", "Connection refused")
print(f"Created: {type(err).__name__}: {err}")
```

## Advanced Examples

```python
# Advanced Example 1: Context manager for capturing and re-raising exceptions
from contextlib import contextmanager
import traceback
import sys


class ExceptionCapture:
    def __init__(self):
        self.exceptions = []

    @contextmanager
    def capture(self, error_types=Exception):
        try:
            yield
        except error_types as e:
            self.exceptions.append({
                "type": type(e).__name__,
                "message": str(e),
                "traceback": traceback.format_exc()
            })


capture = ExceptionCapture()
with capture.capture((ValueError, TypeError)):
    int("not_a_number")

with capture.capture((ValueError, TypeError)):
    "hello" + 42

for exc in capture.exceptions:
    print(f"Caught {exc['type']}: {exc['message']}")


# Advanced Example 2: Reusable exception with automatic traceback trimming
import linecache


class SanitizedError(Exception):
    """Strips internal stack frames from the traceback."""

    def __init__(self, message, hide_frames=1):
        self.hide_frames = hide_frames
        super().__init__(message)

    def with_traceback(self, tb):
        for _ in range(self.hide_frames):
            if tb is not None:
                tb = tb.tb_next
        return super().with_traceback(tb)


def internal_helper():
    raise SanitizedError("Something broke", hide_frames=1)


def public_api():
    internal_helper()


try:
    public_api()
except SanitizedError as e:
    traceback.print_exception(type(e), e, e.__traceback__)


# Advanced Example 3: Exception with automatic logging
import logging

logger = logging.getLogger(__name__)


class LoggedException(Exception):
    def __init__(self, message, log_level=logging.ERROR, **context):
        self.context = context
        super().__init__(message)
        logger.log(log_level, "%s | context=%s", message, context)


class DatabaseTimeout(LoggedException):
    pass


try:
    raise DatabaseTimeout("Query exceeded time limit", timeout=60, query="SELECT *")
except DatabaseTimeout:
    print("Exception was automatically logged")


# Advanced Example 4: Exception with retry metadata
class RetryableError(Exception):
    def __init__(self, message, retry_after=1.0, max_retries=3):
        self.retry_after = retry_after
        self.max_retries = max_retries
        self.attempts = 0
        super().__init__(message)

    def should_retry(self):
        self.attempts += 1
        return self.attempts < self.max_retries


class RateLimitError(RetryableError):
    pass


class ServiceUnavailableError(RetryableError):
    pass


import time


def call_with_retry(func, *args, **kwargs):
    last_exception = None
    while True:
        try:
            return func(*args, **kwargs)
        except RetryableError as e:
            last_exception = e
            if not e.should_retry():
                raise
            print(f"Retrying after {e.retry_after}s (attempt {e.attempts})")
            time.sleep(e.retry_after)


def unreliable_service():
    import random
    if random.random() < 0.7:
        raise ServiceUnavailableError("Service is down", retry_after=0.1, max_retries=3)
    return "Success"


try:
    result = call_with_retry(unreliable_service)
    print(f"Got result: {result}")
except RetryableError as e:
    print(f"All retries exhausted after {e.attempts} attempts")
```

## Real-World Use Cases

Custom exceptions are extensively used in web frameworks (e.g., Django's `Http404`, `PermissionDenied`), database libraries (e.g., SQLAlchemy's `IntegrityError`, `OperationalError`), API clients, and financial systems. They allow libraries to provide a clean error contract to consumers, enable middleware to handle errors centrally, and make audit logging more precise. In microservice architectures, custom exceptions are often serialized to JSON error responses with structured error codes for client consumption.

```python
# Real-world: Flask-style HTTP exception framework
class HTTPException(Exception):
    def __init__(self, description, status_code=500):
        self.description = description
        self.status_code = status_code
        super().__init__(description)

    def to_response(self):
        return {
            "error": type(self).__name__,
            "message": self.description,
            "status_code": self.status_code,
        }, self.status_code


class BadRequest(HTTPException):
    def __init__(self, description="Bad request"):
        super().__init__(description, 400)


class Unauthorized(HTTPException):
    def __init__(self, description="Unauthorized"):
        super().__init__(description, 401)


class Forbidden(HTTPException):
    def __init__(self, description="Forbidden"):
        super().__init__(description, 403)


class NotFound(HTTPException):
    def __init__(self, description="Resource not found"):
        super().__init__(description, 404)


class ConflictError(HTTPException):
    def __init__(self, description="Resource conflict"):
        super().__init__(description, 409)


class InternalServerError(HTTPException):
    def __init__(self, description="Internal server error"):
        super().__init__(description, 500)


def handle_request(method, path, body=None):
    if method not in ("GET", "POST", "PUT", "DELETE"):
        raise BadRequest(f"Unsupported method: {method}")
    if path == "/secret" and not body:
        raise Unauthorized("Authentication required")
    if path == "/admin" and body.get("role") != "admin":
        raise Forbidden("Admin access required")
    if path == "/users/999":
        raise NotFound("User not found")
    if path == "/conflict":
        raise ConflictError("Resource already exists")
    return {"status": "ok"}


def app(method, path, body=None):
    try:
        return handle_request(method, path, body)
    except HTTPException as e:
        return e.to_response()


print(app("GET", "/users/999"))
print(app("DELETE", "/", None))
print(app("GET", "/secret"))
```

## Common Mistakes

1. **Overusing custom exceptions** — creating a new exception type for every possible error leads to explosion of types; use judiciously with hierarchy grouping.
2. **Catching the wrong level** — catching `Exception` instead of a specific custom exception, silently swallowing unexpected errors.
3. **Inheriting from BaseException** — should always inherit from `Exception` (not `BaseException`) to avoid catching `SystemExit`/`KeyboardInterrupt`.
4. **Forgetting to call `super().__init__()`** — results in the exception message being empty.
5. **Using mutable default arguments** — in the `__init__` signature (e.g., `headers=None` instead of `headers={}`).
6. **Not using `__cause__` or `from`** — swallowing the original traceback when re-raising a custom exception.
7. **Making exceptions too heavy** — including database connections or file handles in exception objects that may persist across long stack unwinds.
8. **Inconsistent naming** — not following the `Error`/`Exception` suffix convention.

```python
# Common mistakes demonstration

# Mistake 1: Inheriting from BaseException (wrong)
class BadMistake(BaseException):  # Catches KeyboardInterrupt, SystemExit
    pass


# Mistake 2: Forgetting super().__init__
class ForgetfulError(Exception):
    def __init__(self, message, code):
        self.code = code
        # Missing: super().__init__(message)


try:
    raise ForgetfulError("Oops", 123)
except ForgetfulError as e:
    print(f"Message is empty: '{e}'")  # Empty message!


# Mistake 3: Mutable default argument
class BuggyError(Exception):
    def __init__(self, message, extra=None):
        self.extra = extra or {}
        super().__init__(message)


# Mistake 4: Swallowing the chain
class WrapperError(Exception):
    pass


def read_file(path):
    try:
        with open(path) as f:
            return f.read()
    except FileNotFoundError:
        raise WrapperError("File not found")  # Loses original traceback!


# Correct way:
def read_file_fixed(path):
    try:
        with open(path) as f:
            return f.read()
    except FileNotFoundError as e:
        raise WrapperError("File not found") from e
```

## Best Practices

1. **Inherit from `Exception`**, never from `BaseException` directly.
2. **Name your exceptions with an `Error` or `Exception` suffix** (e.g., `ValidationError`, `ConnectionException`).
3. **Build hierarchies** that mirror your domain (e.g., `DatabaseError` -> `ConnectionError`, `QueryError`).
4. **Add meaningful attributes** (status codes, error IDs, payload) to aid debugging and error reporting.
5. **Implement `__str__` and/or `__repr__`** for informative error messages.
6. **Use exception chaining (`raise ... from e`)** to preserve the original traceback.
7. **Keep exceptions lightweight** — avoid heavy objects or I/O in constructors.
8. **Document exceptions in docstrings** so consumers know what to catch.
9. **Provide factory class methods** for common error patterns.
10. **Use dataclasses for simple exceptions** in modern Python (3.7+).

```python
# Best practices in action
from dataclasses import dataclass


@dataclass
class ApiError(Exception):
    message: str
    status_code: int = 500
    error_code: str = "UNKNOWN"
    details: dict = None

    def __post_init__(self):
        if self.details is None:
            self.details = {}
        super().__init__(self.message)

    def to_dict(self):
        return {
            "error": self.error_code,
            "message": self.message,
            "status": self.status_code,
            "details": self.details,
        }


class RateLimitError(ApiError):
    def __init__(self, retry_after, message="Rate limit exceeded"):
        super().__init__(
            message=message,
            status_code=429,
            error_code="RATE_LIMIT",
            details={"retry_after_seconds": retry_after},
        )


# Usage
try:
    raise RateLimitError(retry_after=30)
except ApiError as e:
    print(e.to_dict())
```

## Interview Questions

**Q1: How do you create a custom exception in Python?**
```python
class MyError(Exception):
    pass
```

**Q2: Why would you use a hierarchy of custom exceptions?**
A: To allow callers to catch specific errors or broad categories. For example, `DatabaseError` -> `ConnectionError`, `TimeoutError` lets users either handle specific issues or catch all database problems.

**Q3: What is the difference between `raise` and `raise from`?**
```python
# Without from: loses original context
try:
    int("abc")
except ValueError:
    raise RuntimeError("Bad conversion")

# With from: preserves chain
try:
    int("abc")
except ValueError as e:
    raise RuntimeError("Bad conversion") from e
```

**Q4: When should you NOT use a custom exception?**
A: When a built-in exception (`ValueError`, `TypeError`, `KeyError` etc.) already communicates the problem clearly. Over-customizing leads to exception class explosion.

**Q5: How can you make custom exceptions serializable?**
A: Implement `__reduce__` or use dataclasses with `asdict()` from the `dataclasses` module, or add a `to_dict()` method.

## Coding Challenges

**Challenge 1: HTTP Error Framework**
Create a set of custom exceptions for an HTTP client library with status codes, headers, and response body attributes. Include: `HttpClientError` (4xx), `HttpServerError` (5xx), and specific subclasses like `NotFoundError`, `UnauthorizedError`, `InternalServerError`.

**Challenge 2: Retryable Operation Runner**
Build an exception hierarchy for a task queue: `TaskError` -> `RetryableTaskError` (with `max_retries` and `delay`) and `FatalTaskError`. Write a runner that retries `RetryableTaskError` up to `max_retries` times before giving up.

**Challenge 3: Validation Rule Engine**
Create exceptions for a validation framework: `ValidationError` (base), `RequiredFieldError`, `TypeMismatchError`, `RangeError`, `PatternMismatchError`. Each should store the field name, value, and a user-readable message. Build a simple validator that uses these.

## Summary

Custom exceptions are a fundamental tool for building robust, maintainable Python applications. They provide semantic clarity, enable precise error handling, and form the foundation of a library's error contract. By creating well-structured exception hierarchies, adding meaningful context, using exception chaining, and following established naming conventions, developers can dramatically improve debugging experiences and API usability. The key is to balance granularity with simplicity—create enough exception types to be useful, but not so many that they become a burden to maintain.

## Related Topics

- Python's built-in exception hierarchy (`Exception`, `BaseException`, `ArithmeticError`, `OSError`)
- Exception chaining (`raise ... from e`)
- Context managers and exception handling (`__enter__`, `__exit__`)
- The `warnings` module for non-fatal issues
- `traceback` module for traceback inspection
- Dataclasses for lightweight exception definitions
- Logging module integration with exceptions
