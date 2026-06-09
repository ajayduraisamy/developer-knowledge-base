# Logging - logging module, levels, handlers, formatters

## Introduction

The `logging` module is Python's built-in framework for emitting log messages from applications. It provides a flexible, configurable system for recording events, errors, and diagnostic information at different severity levels. Unlike `print()` statements, the logging module supports hierarchical loggers, multiple output destinations (handlers), message formatting, runtime level control, and production-grade configuration. It is the standard approach for observability in Python applications.

## logging module

### What It Is

The `logging` module provides a hierarchical logger system where loggers are organized by name (typically using dot-separated module paths). It separates the concerns of producing log messages (loggers), routing them (handlers), and formatting them (formatters). This separation allows fine-grained control over which messages go where.

### Why It Is Important

Logging is essential for understanding application behavior in production where interactive debugging is impossible. It provides an audit trail of events, helps diagnose failures, tracks performance issues, and integrates with centralized log management systems. Proper logging separates observability from business logic, making both more maintainable.

### How It Works Internally

When a logger emits a message, it checks its own level threshold first. If the message passes, it creates a `LogRecord` object and passes it to all attached handlers. Each handler independently checks its own level threshold and formatting. The record can also be passed to parent loggers if `propagate` is enabled. This two-level filtering (logger + handler) enables complex routing configurations.

### Syntax

```python
import logging

# Basic configuration (root logger)
logging.basicConfig(level=logging.INFO)

# Module-level logger (best practice)
logger = logging.getLogger(__name__)

# Logging methods
logger.debug("Detailed diagnostic info")
logger.info("General operational message")
logger.warning("Something unexpected")
logger.error("A failure occurred")
logger.critical("System cannot continue")

# Exception logging
try:
    1 / 0
except ZeroDivisionError:
    logger.exception("An error occurred while dividing")

# Lazy formatting
logger.info("User %s logged in from %s", username, ip_address)
```

### Beginner Examples

```python
import logging

# Simple setup
logging.basicConfig(level=logging.INFO)
logging.info("Application started")

# Logging to file
logging.basicConfig(
    filename="app.log",
    level=logging.DEBUG,
    format="%(asctime)s - %(levelname)s - %(message)s"
)
logging.info("This goes to app.log")

# Different severity levels
logging.basicConfig(level=logging.DEBUG)
logging.debug("Detailed diagnostic info")
logging.info("General operational messages")
logging.warning("Something unexpected but not an error")
logging.error("A failure occurred")
logging.critical("System cannot continue")

# Logging exceptions
try:
    result = 10 / 0
except ZeroDivisionError:
    logging.error("Caught an exception", exc_info=True)
    # Or use the shortcut:
    logging.exception("Caught an exception")

# String formatting in log messages
name = "Alice"
age = 30
logging.info("User %s is %d years old", name, age)  # Preferred
logging.info(f"User {name} is {age} years old")  # Works but less efficient

# Disabling logging
logging.disable(logging.CRITICAL)  # Suppresses all below CRITICAL
logging.warning("This won't appear")
logging.disable(logging.NOTSET)  # Re-enable
```

### Intermediate Examples

```python
import logging
import sys

# Multiple handlers with different levels
logger = logging.getLogger("multi_handler")
logger.setLevel(logging.DEBUG)

console = logging.StreamHandler(sys.stdout)
console.setLevel(logging.WARNING)
console.setFormatter(logging.Formatter("%(levelname)-8s %(message)s"))
logger.addHandler(console)

file_handler = logging.FileHandler("debug.log")
file_handler.setLevel(logging.DEBUG)
file_handler.setFormatter(logging.Formatter(
    "%(asctime)s | %(levelname)-8s | %(name)s | %(message)s"
))
logger.addHandler(file_handler)

error_handler = logging.FileHandler("errors.log")
error_handler.setLevel(logging.ERROR)
error_handler.setFormatter(logging.Formatter(
    "%(asctime)s | %(levelname)-8s | %(module)s:%(lineno)d | %(message)s"
))
logger.addHandler(error_handler)

logger.debug("Debug: only in debug.log")
logger.info("Info: only in debug.log")
logger.warning("Warning: console + debug.log")
logger.error("Error: console + debug.log + errors.log")

# Logger hierarchy and propagation
logging.basicConfig(level=logging.WARNING)

parent = logging.getLogger("parent")
child = logging.getLogger("parent.child")
grandchild = logging.getLogger("parent.child.grandchild")

parent.setLevel(logging.INFO)
child.setLevel(logging.WARNING)
# grandchild inherits from child

parent.info("Parent info (shows)")
child.info("Child info (doesn't show, level is WARNING)")
child.warning("Child warning (shows)")

# Filtering log records
class SensitiveDataFilter(logging.Filter):
    def filter(self, record):
        msg = record.getMessage()
        sensitive = ["password", "ssn", "credit_card"]
        return not any(s in msg.lower() for s in sensitive)

logger = logging.getLogger("filtered")
logger.setLevel(logging.DEBUG)
logger.addFilter(SensitiveDataFilter())
console = logging.StreamHandler()
console.setFormatter(logging.Formatter("%(message)s"))
logger.addHandler(console)

logger.info("User logged in successfully")  # Shows
logger.info("User password is secret123")  # Filtered out

# Custom log level
CUSTOM_LEVEL = 25
logging.addLevelName(CUSTOM_LEVEL, "IMPORTANT")

def important(self, message, *args, **kwargs):
    if self.isEnabledFor(CUSTOM_LEVEL):
        self._log(CUSTOM_LEVEL, message, args, **kwargs)

logging.Logger.important = important

logger = logging.getLogger("custom_level")
logger.setLevel(logging.DEBUG)
logger.important("This is a custom level message")
```

## Log levels

### What It Is

Log levels define the severity of log messages. Python defines five standard levels (in increasing order): `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`. Each level has a numeric value and a standard meaning. Levels enable filtering: you can set a threshold and only messages at or above that level are processed.

### Why It Is Important

Levels separate diagnostic detail from actionable alerts. During development, `DEBUG` provides rich detail. In production, only `WARNING` and above typically appear, keeping log volume manageable while still capturing issues. Levels enable configuration without code changes—raising the threshold in production reduces noise and I/O overhead.

### How It Works Internally

Each level is an integer constant: `DEBUG=10`, `INFO=20`, `WARNING=30`, `ERROR=40`, `CRITICAL=50`. The logger compares the message level against its configured level using `>=`. If the message level passes, it's processed; otherwise, it's immediately discarded. This short-circuit is important for performance—`logger.debug(...)` with lazy formatting is very cheap when `DEBUG` is disabled.

### Syntax

```python
logging.debug("msg")     # Level 10
logging.info("msg")      # Level 20
logging.warning("msg")   # Level 30
logging.error("msg")     # Level 40
logging.critical("msg")  # Level 50

# Getting the level
print(logger.level)              # Numeric value
print(logger.getEffectiveLevel())  # Resolved level (considering parents)
print(logging.getLevelName(20))  # "INFO"

# Checking if level is enabled
if logger.isEnabledFor(logging.DEBUG):
    logger.debug("Expensive computation: %s", compute())
```

### Beginner Examples

```python
import logging

logging.basicConfig(level=logging.WARNING)

# These produce no output (below threshold)
logging.debug("Debug message")
logging.info("Info message")

# These produce output (at or above threshold)
logging.warning("Warning message")
logging.error("Error message")
logging.critical("Critical message")

# Changing levels at runtime
logging.getLogger().setLevel(logging.DEBUG)
logging.debug("Now this appears")  # Shows after level change
```

## Handlers

### What It Is

Handlers determine where log messages go. Python provides built-in handlers for console output, files, rotating files, syslog, email, HTTP, and more. Each handler independently controls its own level threshold and formatting. Multiple handlers can be attached to a single logger.

### Why It Is Important

Handlers enable flexible log routing: `DEBUG` to a file for developers, `WARNING` to a file for operations, `ERROR` to an email alert, `CRITICAL` to a pager system. This separation means the code that logs doesn't need to know about destinations—handlers handle that.

### How It Works Internally

When a `LogRecord` is emitted, the logger passes it to each registered handler. Each handler checks its own level threshold, applies its formatter, and emits the record to its destination. If the logger's `propagate` attribute is `True` (default), the record is also passed to parent loggers' handlers. This continues up to the root logger.

### Syntax

```python
import logging

logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

# StreamHandler: writes to console
console = logging.StreamHandler()
console.setLevel(logging.INFO)
console.setFormatter(logging.Formatter("%(levelname)s: %(message)s"))
logger.addHandler(console)

# FileHandler: writes to file
file_handler = logging.FileHandler("app.log", encoding="utf-8")
file_handler.setLevel(logging.DEBUG)
file_handler.setFormatter(logging.Formatter(
    "%(asctime)s - %(levelname)s - %(message)s"
))
logger.addHandler(file_handler)

# RotatingFileHandler: file rotation by size
rotating = logging.handlers.RotatingFileHandler(
    "app.log", maxBytes=1048576, backupCount=5
)
logger.addHandler(rotating)

# TimedRotatingFileHandler: rotation by time
timed = logging.handlers.TimedRotatingFileHandler(
    "app.log", when="midnight", interval=1, backupCount=30
)
logger.addHandler(timed)

# QueueHandler: non-blocking logging
import queue
from logging.handlers import QueueHandler, QueueListener

log_queue = queue.Queue()
queue_handler = QueueHandler(log_queue)
logger.addHandler(queue_handler)

listener = QueueListener(log_queue, file_handler)
listener.start()
# Messages are now processed in a background thread
```

### Beginner Examples

```python
import logging

# Console handler
console = logging.StreamHandler()
console.setLevel(logging.WARNING)
formatter = logging.Formatter("%(levelname)-8s: %(message)s")
console.setFormatter(formatter)
logging.getLogger("").addHandler(console)
logging.warning("This goes to console")

# File handler
file_handler = logging.FileHandler("example.log")
file_handler.setLevel(logging.DEBUG)
file_handler.setFormatter(logging.Formatter(
    "%(asctime)s - %(levelname)s - %(message)s"
))
logging.getLogger("").addHandler(file_handler)
logging.debug("This goes to the file")

# Removing handlers
logger = logging.getLogger("temp")
logger.addHandler(logging.StreamHandler())
logger.handlers.clear()  # Remove all handlers
```

## Formatters

### What It Is

Formatters define the layout of log messages. They use `%`-style formatting with `LogRecord` attributes like `%(asctime)s`, `%(name)s`, `%(levelname)s`, `%(message)s`, `%(module)s`, `%(lineno)d`, `%(funcName)s`. Custom formatters can produce JSON, XML, or any other structured format.

### Why It Is Important

Formats determine how useful logs are for debugging and analysis. A good format includes timestamp, level, source location, and message. Structured formats (JSON) enable automated parsing and analysis in log aggregation systems. Different handlers can use different formats—machine-readable for files, human-readable for console.

### How It Works Internally

The `Formatter.format()` method takes a `LogRecord` and returns a string. It uses `%` formatting to interpolate the record's attributes. Custom formatters can override `format()` to add extra processing like JSON serialization, sensitive data masking, or contextual information injection.

### Syntax

```python
import logging

# Basic formatter
formatter = logging.Formatter(
    "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
)

# With date format
formatter = logging.Formatter(
    "%(asctime)s [%(levelname)s] %(name)s: %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S"
)

# Commonly used attributes
# %(name)s       - Logger name
# %(levelname)s  - Log level text
# %(levelno)s    - Log level number
# %(pathname)s   - Full pathname of source file
# %(filename)s   - Filename part
# %(module)s     - Module name
# %(funcName)s   - Function name
# %(lineno)d     - Line number
# %(asctime)s    - Human-readable time
# %(created)f    - Time when created (time.time())
# %(msecs)d      - Millisecond part
# %(relativeCreated)d - Time relative to logging module load
# %(thread)d     - Thread ID
# %(threadName)s - Thread name
# %(process)d    - Process ID
# %(message)s    - The logged message
```

### Beginner Examples

```python
import logging

# Simple console formatter
logging.basicConfig(
    level=logging.INFO,
    format="%(levelname)s: %(message)s"
)
logging.info("Clean and simple")

# Timestamped formatter
logging.basicConfig(
    level=logging.DEBUG,
    format="%(asctime)s | %(levelname)-8s | %(name)s | %(message)s",
    datefmt="%H:%M:%S"
)

# Verbose formatter for debugging
logging.basicConfig(
    level=logging.DEBUG,
    format="%(asctime)s [%(levelname)-8s] %(name)s (%(module)s:%(lineno)d): %(message)s"
)
```

### Advanced Examples

```python
import logging
import json
import sys

# JSON formatter for structured logging
class JsonFormatter(logging.Formatter):
    def format(self, record):
        log_entry = {
            "timestamp": self.formatTime(record, self.datefmt),
            "level": record.levelname,
            "logger": record.name,
            "module": record.module,
            "function": record.funcName,
            "line": record.lineno,
            "message": record.getMessage(),
        }
        if record.exc_info and record.exc_info[0] is not None:
            log_entry["exception"] = {
                "type": record.exc_info[0].__name__,
                "message": str(record.exc_info[1]),
            }
        if hasattr(record, "extra_fields"):
            log_entry.update(record.extra_fields)
        return json.dumps(log_entry)

class StructuredLogger(logging.Logger):
    def _log(self, level, msg, args, exc_info=None, extra=None, **kwargs):
        if extra is None:
            extra = {}
        extra.setdefault("extra_fields", kwargs.pop("extra_fields", {}))
        super()._log(level, msg, args, exc_info, extra)

logging.setLoggerClass(StructuredLogger)
logger = logging.getLogger("structured")
handler = logging.StreamHandler(sys.stdout)
handler.setFormatter(JsonFormatter())
logger.addHandler(handler)
logger.setLevel(logging.DEBUG)

logger.info("User action", extra_fields={"user_id": 42, "action": "login"})

# Contextual logging with contextvars (Python 3.7+)
import contextvars

request_id_var = contextvars.ContextVar("request_id", default="N/A")

class ContextFilter(logging.Filter):
    def filter(self, record):
        record.request_id = request_id_var.get()
        return True

logger = logging.getLogger("contextual")
handler = logging.StreamHandler()
handler.setFormatter(logging.Formatter(
    "[%(request_id)s] %(levelname)s: %(message)s"
))
logger.addHandler(handler)
logger.addFilter(ContextFilter())

def handle_request(request_id):
    token = request_id_var.set(request_id)
    try:
        logger.info("Processing")
    finally:
        request_id_var.reset(token)

handle_request("REQ-001")
handle_request("REQ-002")
```

### Real-World Use Cases

```python
import logging
import logging.config
import sys
from pathlib import Path

# Production logging configuration
LOGGING_CONFIG = {
    "version": 1,
    "disable_existing_loggers": False,
    "formatters": {
        "standard": {
            "format": "%(asctime)s [%(levelname)s] %(name)s: %(message)s",
        },
        "json": {
            "format": '{"time": "%(asctime)s", "level": "%(levelname)s", '
                      '"logger": "%(name)s", "message": "%(message)s"}',
        },
    },
    "handlers": {
        "console": {
            "class": "logging.StreamHandler",
            "level": "INFO",
            "formatter": "standard",
        },
        "file": {
            "class": "logging.handlers.RotatingFileHandler",
            "level": "DEBUG",
            "formatter": "standard",
            "filename": "app.log",
            "maxBytes": 10485760,
            "backupCount": 5,
        },
        "error_file": {
            "class": "logging.handlers.RotatingFileHandler",
            "level": "ERROR",
            "formatter": "json",
            "filename": "errors.json",
            "maxBytes": 10485760,
            "backupCount": 10,
        },
    },
    "loggers": {
        "myapp": {
            "handlers": ["console", "file", "error_file"],
            "level": "DEBUG",
            "propagate": False,
        },
    },
    "root": {
        "handlers": ["console"],
        "level": "WARNING",
    },
}

logging.config.dictConfig(LOGGING_CONFIG)
logger = logging.getLogger("myapp")
logger.info("Application started")

# Service-level structured logging
class ServiceLogger:
    def __init__(self, service_name, version="1.0.0"):
        self.logger = logging.getLogger(service_name)
        handler = logging.StreamHandler(sys.stdout)
        handler.setFormatter(logging.Formatter(
            "%(asctime)s %(levelname)s [%(service)s] %(message)s"
        ))
        self.logger.addHandler(handler)
        self.logger = logging.LoggerAdapter(self.logger, {
            "service": service_name,
        })
        self.logger.setLevel(logging.INFO)

    def info(self, msg, **context):
        self.logger.info(f"{msg} | {context}")

svc_logger = ServiceLogger("payment-service")
svc_logger.info("Transaction started", txn_id="TXN-123", amount=99.95)
```

### Common Mistakes

```python
import logging

# Mistake 1: Using print() instead of logging
print("User logged in")  # No level, no format, no routing

# Mistake 2: Not using getLogger(__name__)
logging.info("message")  # Uses root logger, hard to configure per-module

# Mistake 3: Calling basicConfig() multiple times
logging.basicConfig(level=logging.DEBUG)
logging.basicConfig(level=logging.INFO)  # Second call is ignored!

# Mistake 4: Setting level on handler but not logger
logger = logging.getLogger("mistake")
handler = logging.StreamHandler()
handler.setLevel(logging.DEBUG)  # This alone does nothing!
logger.addHandler(handler)
logger.setLevel(logging.INFO)  # Logger level blocks DEBUG messages
logger.debug("Will this show?")  # No, blocked by logger

# Mistake 5: Using f-strings (eager evaluation)
logging.debug(f"Expensive: {compute()}")  # Always runs compute()
# Correct: lazy evaluation
logging.debug("Expensive: %s", compute())

# Mistake 6: Logging sensitive data
password = "super_secret_123"
logging.warning(f"Login with password: {password}")  # BAD

# Mistake 7: Not using logger.exception()
try:
    1 / 0
except ZeroDivisionError:
    logging.error("Error occurred")  # No traceback!
    logging.exception("Error occurred")  # Has traceback

# Mistake 8: Duplicate handlers
def setup_logger():
    logger = logging.getLogger("myapp")
    logger.addHandler(logging.StreamHandler())  # Adds new one every call
    return logger
```

### Best Practices

- Always use `getLogger(__name__)` for module-level loggers
- Configure logging once at application entry point
- Use lazy formatting with `%s` placeholders, not f-strings
- Rotate log files with `RotatingFileHandler` or `TimedRotatingFileHandler`
- Use appropriate levels: DEBUG for diagnostics, INFO for events, WARNING for issues, ERROR for failures, CRITICAL for system-down
- Log exceptions with `logger.exception()` to include traceback
- Sanitize sensitive data before logging
- Use structured logging (JSON) for machine parsing
- Separate ERROR+ logs for quick incident triage
- Use `logging.config.dictConfig()` for complex configurations
- Libraries should never configure logging, only emit messages
- Use `isEnabledFor()` to guard expensive debug computation

### Performance Considerations

Logging can be expensive if done in hot paths. Key optimizations:
- Lazy formatting: `logger.debug("x=%s", x)` avoids string formatting if `DEBUG` is disabled
- Level checks are fast integer comparisons
- Use `isEnabledFor()` for expensive log message construction
- Use `QueueHandler`/`QueueListener` for non-blocking logging in high-throughput systems
- Async logging decouples I/O from the main thread
- Avoid logging in tight loops—evaluate whether each iteration needs logging

### Interview Questions

1. What is the difference between `logging` and `print()`?
   Logging provides levels, flexible routing, formatting, runtime configuration, and can be disabled without code changes.

2. How does logger hierarchy work?
   Loggers are dot-separated (e.g., "app.module"). Children inherit settings from parents. Messages propagate upward unless `propagate=False`.

3. What is `getLogger(__name__)`?
   It creates a logger named after the module, enabling per-module log level and handler configuration.

4. What is the difference between `logging.error()` and `logging.exception()`?
   `exception()` is identical to `error()` but automatically includes the current exception's traceback.

5. How do you ensure lazy log message evaluation?
   Use `logger.debug("value is %s", value)` with `%s` placeholders. The string is only formatted if the level is enabled.

6. What are the five standard log levels and their numeric values?
   DEBUG(10), INFO(20), WARNING(30), ERROR(40), CRITICAL(50).

7. How does `basicConfig()` work and what are its limitations?
   It configures the root logger with a StreamHandler. It only works once—subsequent calls are ignored. Not suitable for complex configurations.

8. What is the difference between logger level and handler level?
   The logger level is checked first and can discard a message before handlers see it. Handler levels filter independently after passing the logger.

### Coding Challenges

1. Create a logger that sends DEBUG and INFO to a file, WARNING to both file and console, and ERROR/CRITICAL to a separate error file with full traceback.

2. Build a JSON formatter that outputs timestamp, level, logger, module, function, line number, message, and extra context fields.

3. Write a logging Filter that masks passwords, API keys, SSNs, and credit card numbers in log messages.

4. Implement a handler that counts ERROR/CRITICAL messages per minute and triggers an alert callback above a threshold.

5. Create a contextual logging system using `contextvars` that automatically includes request_id, user_id, and session_id in every log message.

### Related Topics

- `43_exceptions.md` - Exception handling basics
- `44_try_except.md` - Exception handling patterns
- `45_custom_exceptions.md` - Custom exception classes
- `47_assertions.md` - Assertions for debugging
- Python's `logging.config` module
- `logging.handlers` (SysLogHandler, SMTPHandler, HTTPHandler)
- `structlog` third-party library for advanced structured logging
- OpenTelemetry for distributed tracing
