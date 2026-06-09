# Custom Exceptions - class MyError(Exception), exception hierarchies

## Introduction

Custom exceptions are user-defined exception classes that inherit from Python's built-in `Exception` class. While Python provides dozens of built-in exceptions for common error conditions, real-world applications often need domain-specific errors that precisely describe problems in their context. Custom exceptions form the backbone of robust error handling in large-scale Python projects, making error handling more expressive, granular, and maintainable.

## class MyError(Exception)

### What It Is

Creating a custom exception is as simple as defining a class that inherits from `Exception` (or any of its subclasses). The new class inherits all the standard exception behavior and can add custom attributes, methods, and formatting. The convention is to name custom exceptions with an "Error" or "Exception" suffix.

### Why It Is Important

Custom exceptions allow precise error catching. Instead of catching a generic `Exception` and inspecting its message string, you catch a specific custom type. This makes code self-documenting, prevents string-matching fragility, and enables hierarchical error handling. They also improve API clarity by giving consumers a clear contract of what errors to expect.

### How It Works Internally

A custom exception class is a regular Python class. When raised, Python creates an instance and stores `args` (the positional arguments passed to the constructor). The `__init__` method can store additional attributes. The `__str__` method controls string representation. Exception objects, like all Python objects, are garbage-collected when no longer referenced. The inheritance chain determines which `except` clauses will catch the exception.

### Syntax

```python
class MyError(Exception):
    pass

class MyError(Exception):
    def __init__(self, message, code=None):
        self.code = code
        super().__init__(message)
```

### Beginner Examples

```python
# Minimal custom exception
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

# Custom exception with message
class EmptyListError(Exception):
    pass

def average(numbers):
    if not numbers:
        raise EmptyListError("Cannot compute average of empty list")
    return sum(numbers) / len(numbers)

try:
    avg = average([])
except EmptyListError as e:
    print(f"Error: {e}")

# Simple domain exception
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

# Multiple custom exceptions
class TooHotError(Exception): pass
class TooColdError(Exception): pass

def check_temperature(temp):
    if temp > 40:
        raise TooHotError(f"Temperature {temp} is too hot")
    if temp < 0:
        raise TooColdError(f"Temperature {temp} is too cold")
    return "Temperature is acceptable"

try:
    check_temperature(50)
except TooHotError as e:
    print(f"Hot: {e}")
except TooColdError as e:
    print(f"Cold: {e}")
```

### Intermediate Examples

```python
# Custom exception with attributes and __str__
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

# Adding context with exception chaining
class DataProcessingError(Exception):
    pass

def load_data(filepath):
    try:
        with open(filepath) as f:
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
    print(f"Failed: {e}")
    print(f"Caused by: {e.__cause__}")

# Exception with factory methods
class ApiError(Exception):
    def __init__(self, message, status_code=500, error_code="UNKNOWN"):
        self.status_code = status_code
        self.error_code = error_code
        super().__init__(message)

    @classmethod
    def not_found(cls, resource):
        return cls(f"{resource} not found", 404, "NOT_FOUND")

    @classmethod
    def bad_request(cls, detail):
        return cls(f"Bad request: {detail}", 400, "BAD_REQUEST")

    @classmethod
    def unauthorized(cls, reason="Authentication required"):
        return cls(reason, 401, "UNAUTHORIZED")

raise ApiError.not_found("User")
```

### Advanced Examples

```python
# Dataclass-based exception (Python 3.7+)
from dataclasses import dataclass

@dataclass
class DatabaseError(Exception):
    message: str
    query: str = ""
    error_code: int = 0
    connection_id: str = ""

    def __post_init__(self):
        super().__init__(self.message)

    def to_dict(self):
        return {
            "error": "DATABASE_ERROR",
            "message": self.message,
            "code": self.error_code,
        }

class ConnectionTimeout(DatabaseError):
    def __init__(self, host, port, timeout):
        super().__init__(
            message=f"Connection to {host}:{port} timed out after {timeout}s",
            error_code=1001,
        )

# Exception with automatic logging
import logging
logger = logging.getLogger(__name__)

class LoggedException(Exception):
    def __init__(self, message, log_level=logging.ERROR, **context):
        self.context = context
        super().__init__(message)
        logger.log(log_level, "%s | context=%s", message, context)

class ServiceUnavailable(LoggedException):
    pass

# Exception with retry metadata
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

import time

def call_with_retry(func, *args, **kwargs):
    while True:
        try:
            return func(*args, **kwargs)
        except RetryableError as e:
            if not e.should_retry():
                raise
            time.sleep(e.retry_after)
```

## Exception hierarchies

### What It Is

Exception hierarchies organize custom exceptions into parent-child relationships, mirroring the structure of Python's built-in exception hierarchy. A base exception class defines a category, and subclasses define specific error types within that category.

### Why It Is Important

Hierarchies enable flexible error handling. Callers can catch a broad category (like `PaymentError`) to handle all payment failures, or catch a specific type (like `CardDeclinedError`) for specialized handling. This follows the principle of catching what you can handle and letting the rest propagate. Hierarchies also make APIs more maintainable—adding a new exception subclass doesn't break existing catch clauses for parent types.

### How It Works Internally

The `except` clause uses `isinstance()` matching against the hierarchy. `except PaymentError` catches any instance of `PaymentError` or any of its subclasses. This polymorphic behavior is the same mechanism used for all Python inheritance. The `except` clauses are checked in order, so more specific exceptions should be listed before broader ones.

### Syntax

```python
class PaymentError(Exception): pass
class InsufficientBalanceError(PaymentError): pass
class CardDeclinedError(PaymentError): pass
class FraudSuspicionError(PaymentError): pass
class NetworkError(PaymentError): pass
```

### Beginner Examples

```python
# Simple hierarchy
class DatabaseError(Exception): pass
class ConnectionError(DatabaseError): pass
class QueryError(DatabaseError): pass
class TimeoutError(QueryError): pass

def execute_query(query, timeout=30):
    if timeout > 60:
        raise TimeoutError(f"Query timed out after {timeout}s")
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

### Intermediate Examples

```python
# Hierarchical HTTP errors
class HTTPError(Exception):
    def __init__(self, description, status_code=500):
        self.description = description
        self.status_code = status_code
        super().__init__(description)

class BadRequest(HTTPError):
    def __init__(self, description="Bad request"):
        super().__init__(description, 400)

class Unauthorized(HTTPError):
    def __init__(self, description="Unauthorized"):
        super().__init__(description, 401)

class Forbidden(HTTPError):
    def __init__(self, description="Forbidden"):
        super().__init__(description, 403)

class NotFound(HTTPError):
    def __init__(self, description="Not found"):
        super().__init__(description, 404)

class ConflictError(HTTPError):
    def __init__(self, description="Conflict"):
        super().__init__(description, 409)

class InternalServerError(HTTPError):
    def __init__(self, description="Internal error"):
        super().__init__(description, 500)

# Handler that catches hierarchy
def handle_request(method, path):
    if method not in ("GET", "POST"):
        raise BadRequest(f"Unsupported method: {method}")
    if path == "/secret":
        raise Unauthorized("Authentication required")
    if path == "/users/999":
        raise NotFound("User not found")
    return {"status": "ok"}

def app(method, path):
    try:
        return handle_request(method, path)
    except NotFound:
        return {"error": "not_found"}, 404
    except HTTPError as e:
        return {"error": e.description}, e.status_code
```

### Advanced Examples

```python
# Rich hierarchy with shared behavior
class ServiceError(Exception):
    def __init__(self, message, service=None, request_id=None):
        self.service = service
        self.request_id = request_id
        super().__init__(message)

    def to_dict(self):
        return {
            "error": type(self).__name__,
            "message": str(self),
            "service": self.service,
            "request_id": self.request_id,
        }

class ValidationError(ServiceError): pass
class RateLimit(ServiceError):
    def __init__(self, message, retry_after, **kwargs):
        self.retry_after = retry_after
        super().__init__(message, **kwargs)

class DependencyError(ServiceError): pass
class TimeoutError(DependencyError): pass
class CircuitBreakerOpen(DependencyError): pass

# Middleware-style handler
def error_middleware(func):
    def wrapper(*args, **kwargs):
        try:
            return func(*args, **kwargs)
        except ServiceError as e:
            return e.to_dict(), e.status_code if hasattr(e, "status_code") else 500
        except Exception as e:
            return {"error": "INTERNAL", "message": str(e)}, 500
    return wrapper

# __init_subclass__ for registry
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

class NetworkError(RegisteredError): pass
class ConfigError(RegisteredError): pass
class UnknownError(Exception): pass

err = RegisteredError.from_name("NetworkError", "Connection refused")
print(type(err).__name__)  # NetworkError
```

### Real-World Use Cases

```python
# Payment processing hierarchy
class PaymentError(Exception):
    def __init__(self, message, transaction_id=None):
        self.transaction_id = transaction_id
        super().__init__(message)

class InsufficientBalanceError(PaymentError):
    def __init__(self, balance, amount, currency="USD"):
        self.balance = balance
        self.required = amount
        self.shortfall = amount - balance
        super().__init__(
            f"Insufficient balance: have {balance} {currency}, need {amount} {currency}",
        )

class CardDeclinedError(PaymentError):
    def __init__(self, card_last_four, reason):
        self.card_last_four = card_last_four
        self.reason = reason
        super().__init__(f"Card ending in {card_last_four} declined: {reason}")

class FraudSuspicionError(PaymentError):
    pass

class GatewayTimeoutError(PaymentError):
    def __init__(self, gateway, timeout):
        self.gateway = gateway
        self.timeout = timeout
        super().__init__(f"Gateway {gateway} timed out after {timeout}s")

def process_payment(amount, balance, card_last_four):
    if amount > balance:
        raise InsufficientBalanceError(balance, amount)
    if card_last_four == "0000":
        raise CardDeclinedError(card_last_four, "card expired")
    if amount > 10000:
        raise FraudSuspicionError("Transaction flagged for review")
    return "Payment successful"

try:
    process_payment(500, 100, "1234")
except InsufficientBalanceError as e:
    print(f"Need more: short by {e.shortfall}")
except CardDeclinedError as e:
    print(f"Card issue: {e.reason}")
except PaymentError as e:
    print(f"Other payment error: {e}")
```

### Common Mistakes

```python
# Mistake 1: Inheriting from BaseException
class BadError(BaseException):  # Catches KeyboardInterrupt, SystemExit!
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

# Mistake 3: Mutable default arguments
class BuggyError(Exception):
    def __init__(self, message, extra={}):  # BAD: shared mutable default
        self.extra = extra
        super().__init__(message)

# Correct:
class FixedError(Exception):
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

# Correct:
def read_file_fixed(path):
    try:
        with open(path) as f:
            return f.read()
    except FileNotFoundError as e:
        raise WrapperError("File not found") from e

# Mistake 5: Not including useful context
class ApiError(Exception):
    pass  # No status_code, no error_code, no details

# Mistake 6: Over-customizing (too many exception types)
class ErrorLevel1(Exception): pass
class ErrorLevel2(ErrorLevel1): pass
class ErrorLevel3(ErrorLevel2): pass  # Pointlessly specific
```

### Best Practices

- Always inherit from `Exception`, never from `BaseException`
- Name with `Error` or `Exception` suffix (e.g., `ValidationError`)
- Build meaningful hierarchies that mirror your domain
- Add informative attributes (status codes, error codes, context)
- Implement `__str__` for readable error messages
- Use `raise ... from e` to preserve original traceback
- Keep exceptions lightweight—no heavy objects or I/O in constructors
- Document exceptions in docstrings for API consumers
- Use dataclasses for simple exceptions (Python 3.7+)
- Provide factory class methods for common error creation patterns
- Create a base exception for your library/application
- Use `__init_subclass__` for registry patterns when needed

### Performance Considerations

Custom exceptions have the same performance characteristics as built-in exceptions. The overhead is entirely in the raising (traceback construction, stack unwinding), not in the exception class itself. Creating many custom exception classes is cheap. However, avoid expensive operations in exception constructors, since exceptions may be constructed even when they won't be caught (e.g., during traceback display).

### Interview Questions

1. How do you create a custom exception in Python?

   ```python
   class MyError(Exception):
       pass
   ```

2. Why use a hierarchy of custom exceptions?

   To allow callers to catch specific errors or broad categories. `DatabaseError` -> `ConnectionError`, `TimeoutError` lets users handle specific issues or catch all database problems with one `except DatabaseError` clause.

3. What is the difference between `raise` and `raise from`?

   `raise X` inside an `except` block implicitly chains via `__context__`. `raise X from Y` sets `__cause__` explicitly with a clearer traceback message. `raise X from None` suppresses the chain.

4. When should you NOT use a custom exception?

   When a built-in exception (`ValueError`, `TypeError`, `KeyError`) already communicates the problem clearly. Over-customizing leads to exception class explosion.

5. How can you make custom exceptions serializable?

   Implement `__reduce__`, use dataclasses with `asdict()`, or add a `to_dict()` method.

6. Why should exceptions inherit from `Exception` not `BaseException`?

   `BaseException` includes `SystemExit`, `KeyboardInterrupt`, and `GeneratorExit`. Inheriting from it means your exception could be caught by `except BaseException` which also catches system-level exceptions that should normally terminate the program.

### Coding Challenges

1. Create an HTTP error hierarchy with `ClientError` (4xx), `ServerError` (5xx), and specific subclasses like `NotFoundError`, `BadRequestError`, `InternalServerError`. Each should carry status code, message, and response body.

2. Build an exception hierarchy for a task queue: `TaskError` base, `RetryableTaskError` (with `max_retries` and `delay`), and `FatalTaskError`. Write a runner that retries retryable errors.

3. Create validation exceptions: `ValidationError` (base), `RequiredFieldError`, `TypeMismatchError`, `RangeError`, `PatternMismatchError`. Each stores field name, value, and message.

4. Implement a `retry` decorator that checks if an exception is a subclass of `RetryableError` and retries accordingly, with different backoff strategies per exception type.

5. Design a circuit breaker that uses exception types to determine which failures should open the circuit.

### Related Topics

- `43_exceptions.md` - The raise statement and exception hierarchy overview
- `44_try_except.md` - Catching exceptions with try/except
- `46_logging.md` - Logging exceptions
- `47_assertions.md` - Assertions vs exceptions for debugging
