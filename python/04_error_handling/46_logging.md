# Logging - logging module, levels, handlers, formatters

## Introduction

The `logging` module is Python's built-in framework for emitting log messages from applications. It provides a flexible, configurable system for recording events, errors, and diagnostic information at different severity levels. Unlike simple `print()` statements, the logging module supports hierarchical loggers, multiple output destinations (handlers), message formatting, runtime level control, and both development and production configurations. It is the standard approach for instrumentation in Python applications of any size.

## Why It Is Important

Logging is essential for understanding application behavior in production where interactive debugging is impossible. It provides an audit trail of events, helps diagnose failures, tracks performance issues, and monitors security-related events. Proper logging can mean the difference between spending hours to reproduce a bug and fixing it in minutes. Well-structured logs are critical for incident response, compliance requirements, and integration with centralized log management systems (ELK stack, Splunk, Datadog, etc.). Unlike exceptions, which handle error flow control, logging is the primary mechanism for observability.

## Syntax

Basic logging setup:

```python
import logging

logging.basicConfig(level=logging.INFO)
logging.debug("This won't show")
logging.info("Application started")
logging.warning("Low disk space")
logging.error("Failed to connect to database")
logging.critical("System is shutting down")
```

Creating a named logger (best practice):

```python
import logging

logger = logging.getLogger(__name__)
logger.info("Using module-level logger")
```

Configuration with format and handlers:

```python
logging.basicConfig(
    level=logging.DEBUG,
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s",
    filename="app.log",
    filemode="a",
)
```

Using handlers:

```python
import logging

logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

console = logging.StreamHandler()
console.setLevel(logging.INFO)
console.setFormatter(logging.Formatter("%(levelname)s: %(message)s"))
logger.addHandler(console)

file_handler = logging.FileHandler("app.log")
file_handler.setLevel(logging.DEBUG)
file_handler.setFormatter(logging.Formatter(
    "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
))
logger.addHandler(file_handler)
```

## Examples

```python
import logging
import sys

# Example 1: Basic configuration
logging.basicConfig(
    level=logging.DEBUG,
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
)

logging.debug("This is a debug message")
logging.info("This is an info message")
logging.warning("This is a warning message")
logging.error("This is an error message")
logging.critical("This is a critical message")


# Example 2: Module-level logger
logger = logging.getLogger(__name__)
logger.info("Hello from the module-level logger")


# Example 3: Logging with extra context
logger = logging.getLogger("context_demo")
extra = {"user_id": 42, "ip": "192.168.1.1"}
logger.info("User login", extra=extra)


# Example 4: Logging variable values
x = 42
y = "hello"
logger.debug("x=%d, y=%s", x, y)


# Example 5: Exception logging
try:
    1 / 0
except ZeroDivisionError:
    logger.exception("An error occurred while dividing")


# Example 6: Timed rotation
import logging.handlers

rotating_handler = logging.handlers.TimedRotatingFileHandler(
    "daily.log", when="midnight", interval=1, backupCount=7
)
rotating_handler.setFormatter(logging.Formatter(
    "%(asctime)s - %(levelname)s - %(message)s"
))
logger.addHandler(rotating_handler)
logger.info("This will go to the rotating file")
```

## Beginner Examples

```python
import logging

# Beginner Example 1: Simple message logging
logging.basicConfig(level=logging.INFO)
logging.info("Program started")


# Beginner Example 2: Logging to a file
logging.basicConfig(
    filename="app.log",
    level=logging.DEBUG,
    format="%(asctime)s - %(levelname)s - %(message)s"
)
# Run the script twice to see appending behavior
logging.info("This message goes to app.log")


# Beginner Example 3: Different severity levels
logging.basicConfig(level=logging.DEBUG)
logging.debug("Detailed diagnostic info")
logging.info("General operational messages")
logging.warning("Something unexpected but not an error")
logging.error("A failure occurred")
logging.critical("System cannot continue")


# Beginner Example 4: Capturing exceptions
try:
    result = 10 / 0
except ZeroDivisionError:
    logging.error("Caught an exception", exc_info=True)


# Beginner Example 5: Simple string formatting in log messages
name = "Alice"
age = 30
logging.basicConfig(level=logging.INFO)
logging.info(f"User {name} is {age} years old")  # f-string (works but lazy evaluation is better)
logging.info("User %s is %d years old", name, age)  # preferred style


# Beginner Example 6: Disabling logging
logging.disable(logging.CRITICAL)  # Suppresses all messages below CRITICAL
logging.warning("This won't appear")
logging.disable(logging.NOTSET)  # Re-enable logging
logging.warning("This will appear again")


# Beginner Example 7: Simple console handler
console = logging.StreamHandler()
console.setLevel(logging.WARNING)
formatter = logging.Formatter("%(levelname)-8s: %(message)s")
console.setFormatter(formatter)
logging.getLogger("").addHandler(console)
logging.warning("This goes to console")
```

## Intermediate Examples

```python
import logging
import sys
from pathlib import Path

# Intermediate Example 1: Multiple handlers with different levels
logger = logging.getLogger("multi_handler")
logger.setLevel(logging.DEBUG)

# Console handler: only warnings and above
console = logging.StreamHandler(sys.stdout)
console.setLevel(logging.WARNING)
console.setFormatter(logging.Formatter(
    "%(levelname)-8s %(message)s"
))
logger.addHandler(console)

# File handler: everything
file_handler = logging.FileHandler("debug.log")
file_handler.setLevel(logging.DEBUG)
file_handler.setFormatter(logging.Formatter(
    "%(asctime)s | %(levelname)-8s | %(name)s | %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S"
))
logger.addHandler(file_handler)

# Error file handler: errors only
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


# Intermediate Example 2: Logger hierarchy and propagation
logging.basicConfig(level=logging.WARNING)

parent = logging.getLogger("parent")
child = logging.getLogger("parent.child")
grandchild = logging.getLogger("parent.child.grandchild")

parent.setLevel(logging.INFO)
child.setLevel(logging.WARNING)
# grandchild inherits from child

print(f"parent level: {parent.level}")
print(f"child level: {child.level}")
print(f"grandchild level (inherited): {grandchild.level}")

parent.info("Parent info (shows)")
child.info("Child info (doesn't show, level is WARNING)")
child.warning("Child warning (shows)")


# Intermediate Example 3: Filtering log records
class SensitiveDataFilter(logging.Filter):
    def filter(self, record):
        # Block messages containing sensitive data
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


# Intermediate Example 4: Custom log level
CUSTOM_LEVEL = 25  # Between INFO (20) and WARNING (30)
logging.addLevelName(CUSTOM_LEVEL, "IMPORTANT")


def important(self, message, *args, **kwargs):
    if self.isEnabledFor(CUSTOM_LEVEL):
        self._log(CUSTOM_LEVEL, message, args, **kwargs)


logging.Logger.important = important

logger = logging.getLogger("custom_level")
logger.setLevel(logging.DEBUG)
logging.basicConfig(level=logging.DEBUG, format="%(levelname)s: %(message)s")  # Use root

logger.important("This is a custom level message")


# Intermediate Example 5: Logging from multiple modules
# Simulate two modules
module_a_logger = logging.getLogger("app.module_a")
module_b_logger = logging.getLogger("app.module_b")

module_a_logger.info("Module A processing started")
module_b_logger.info("Module B processing started")
module_a_logger.warning("Module A: deprecated config option used")
module_b_logger.error("Module B: failed to connect")


# Intermediate Example 6: QueueHandler for non-blocking logging
import queue
from logging.handlers import QueueHandler, QueueListener

log_queue = queue.Queue()
queue_handler = QueueHandler(log_queue)
listener_logger = logging.getLogger("listener")
listener_logger.addHandler(queue_handler)

# Real handler processes messages in background
file_handler = logging.FileHandler("queue.log")
file_handler.setFormatter(logging.Formatter("%(asctime)s %(message)s"))
listener = QueueListener(log_queue, file_handler)
listener.start()

listener_logger.info("This message is queued (non-blocking)")
listener.stop()
```

## Advanced Examples

```python
import logging
import json
import sys
from pathlib import Path

# Advanced Example 1: JSON formatter for structured logging
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
logger.error("Database failure", extra_fields={"db": "primary", "query_time_ms": 2500})

try:
    1 / 0
except ZeroDivisionError:
    logger.exception("Math error", extra_fields={"operation": "division"})


# Advanced Example 2: Contextual logging with contextvars (Python 3.7+)
import contextvars

request_id_var = contextvars.ContextVar("request_id", default="N/A")
user_id_var = contextvars.ContextVar("user_id", default="N/A")


class ContextFilter(logging.Filter):
    def filter(self, record):
        record.request_id = request_id_var.get()
        record.user_id = user_id_var.get()
        return True


logger = logging.getLogger("contextual")
logger.setLevel(logging.DEBUG)
handler = logging.StreamHandler(sys.stdout)
handler.setFormatter(logging.Formatter(
    "[%(request_id)s] [user=%(user_id)s] %(levelname)s: %(message)s"
))
logger.addHandler(handler)
logger.addFilter(ContextFilter())


def handle_request(request_id, user_id):
    token_r = request_id_var.set(request_id)
    token_u = user_id_var.set(user_id)
    try:
        logger.info("Processing request")
        do_work()
    finally:
        request_id_var.reset(token_r)
        user_id_var.reset(token_u)


def do_work():
    logger.info("Doing work")
    logger.warning("Something looks off")


# Simulate concurrent requests
handle_request("REQ-001", "user_42")
handle_request("REQ-002", "user_99")


# Advanced Example 3: Dynamic log level change at runtime
class DynamicLevelHandler(logging.Handler):
    def emit(self, record):
        if record.levelno >= self.level:
            print(f"DYNAMIC [{record.levelname}] {record.getMessage()}")


logger = logging.getLogger("dynamic")
handler = DynamicLevelHandler()
handler.setLevel(logging.INFO)
logger.addHandler(handler)
logger.setLevel(logging.DEBUG)

logger.info("This shows (INFO >= INFO)")
logger.debug("This does not show (DEBUG < INFO)")

handler.setLevel(logging.DEBUG)
logger.debug("Now this shows (DEBUG >= DEBUG)")


# Advanced Example 4: Log rotation with compression
import gzip
import shutil
from logging.handlers import RotatingFileHandler


class CompressedRotatingFileHandler(RotatingFileHandler):
    def doRollover(self):
        super().doRollover()
        if self.backupCount > 0:
            for i in range(self.backupCount, 0, -1):
                sfn = f"{self.baseFilename}.{i}"
                gzfn = f"{sfn}.gz"
                if Path(sfn).exists() and not Path(gzfn).exists():
                    with open(sfn, "rb") as f_in:
                        with gzip.open(gzfn, "wb") as f_out:
                            shutil.copyfileobj(f_in, f_out)
                    Path(sfn).unlink()


logger = logging.getLogger("compressed_rotating")
handler = CompressedRotatingFileHandler(
    "rotating.log", maxBytes=100, backupCount=3
)
handler.setFormatter(logging.Formatter("%(message)s"))
logger.addHandler(handler)
logger.setLevel(logging.DEBUG)

for i in range(20):
    logger.info(f"Line {i}: " + "x" * 50)


# Advanced Example 5: Watchdog pattern with logging alerts
class AlertHandler(logging.Handler):
    def __init__(self, threshold=logging.ERROR, alert_callback=None):
        super().__init__(level=threshold)
        self.alert_callback = alert_callback

    def emit(self, record):
        if self.alert_callback:
            self.alert_callback(record)


def send_alert(record):
    print(f"ALERT: {record.levelname} - {record.getMessage()}")


logger = logging.getLogger("watchdog")
logger.setLevel(logging.DEBUG)
logger.addHandler(AlertHandler(logging.ERROR, send_alert))
logger.addHandler(logging.StreamHandler(sys.stdout))

logger.info("Normal operation")
logger.error("Critical failure detected")  # Triggers alert callback
```

## Real-World Use Cases

```python
import logging
import sys
from pathlib import Path

# Real-world: Typical production logging configuration
def setup_production_logging(log_dir="logs", app_name="myapp"):
    log_dir = Path(log_dir)
    log_dir.mkdir(exist_ok=True)

    # Root logger
    root_logger = logging.getLogger()
    root_logger.setLevel(logging.INFO)

    # Format
    formatter = logging.Formatter(
        "%(asctime)s | %(levelname)-8s | %(name)s | %(module)s:%(lineno)d | "
        "%(message)s"
    )

    # Console: WARNING+
    console = logging.StreamHandler(sys.stdout)
    console.setLevel(logging.WARNING)
    console.setFormatter(formatter)
    root_logger.addHandler(console)

    # Main log file: INFO+
    main_handler = logging.handlers.TimedRotatingFileHandler(
        log_dir / f"{app_name}.log",
        when="midnight",
        interval=1,
        backupCount=30,
        encoding="utf-8",
    )
    main_handler.setLevel(logging.INFO)
    main_handler.setFormatter(formatter)
    root_logger.addHandler(main_handler)

    # Error log file: ERROR+
    error_handler = logging.handlers.RotatingFileHandler(
        log_dir / f"{app_name}_error.log",
        maxBytes=10 * 1024 * 1024,
        backupCount=10,
        encoding="utf-8",
    )
    error_handler.setLevel(logging.ERROR)
    error_handler.setFormatter(formatter)
    root_logger.addHandler(error_handler)

    return root_logger


# Real-world: Structured logging for microservices
class ServiceLogger:
    def __init__(self, service_name, version="1.0.0"):
        self.logger = logging.getLogger(service_name)
        self.service_name = service_name
        self.version = version
        self._setup()

    def _setup(self):
        handler = logging.StreamHandler(sys.stdout)
        handler.setFormatter(logging.Formatter(
            "%(asctime)s %(levelname)s [%(service)s/%(version)s] %(message)s"
        ))
        self.logger.addHandler(handler)
        self.logger = logging.LoggerAdapter(self.logger, {
            "service": self.service_name,
            "version": self.version,
        })
        self.logger.setLevel(logging.INFO)

    def info(self, msg, **context):
        self.logger.info(f"{msg} | context={context}")

    def error(self, msg, exc_info=None, **context):
        self.logger.error(f"{msg} | context={context}", exc_info=exc_info)


svc_logger = ServiceLogger("payment-service", "2.1.0")
svc_logger.info("Transaction started", txn_id="TXN-123", amount=99.95)
svc_logger.info("Payment processed", txn_id="TXN-123", gateway="stripe")
svc_logger.error("Payment failed", txn_id="TXN-123", error_code="card_declined")


# Real-world: Web request logging middleware pattern
import time


class RequestLogger:
    def __init__(self, logger_name="web.requests"):
        self.logger = logging.getLogger(logger_name)

    def log_request(self, method, path, status_code, duration_ms, **extra):
        self.logger.info(
            "%s %s -> %d (%dms)",
            method, path, status_code, duration_ms,
            extra=extra,
        )


req_logger = RequestLogger()
req_logger.log_request("GET", "/api/users", 200, 45, user_agent="Mozilla/5.0")
req_logger.log_request("POST", "/api/login", 401, 12, reason="invalid_password")
```

## Common Mistakes

1. **Calling `logging.basicConfig()` multiple times** — it only works the first time; subsequent calls are ignored.
2. **Using `print()` instead of `logging`** — no levels, formatting, or routing control.
3. **Not using `getLogger(__name__)`** — using root logger directly creates noisy, hard-to-configure output.
4. **Setting level on handlers but not the logger** — the logger level is checked first; if it blocks, the handler never sees the record.
5. **Leaking sensitive data** — logging passwords, API keys, PII, or credit card numbers.
6. **String interpolation at call site** — using f-strings forces string building even if the log level discards the message; use `%s` formatting with lazy evaluation.
7. **Not handling encoding** — `FileHandler` defaults to system encoding; specify `encoding="utf-8"` explicitly.
8. **Over-logging in hot paths** — logging every iteration of a tight loop kills performance.
9. **Ignoring log rotation** — single files grow unboundedly, filling disks.
10. **Using `logging.basicConfig()` in library code** — libraries should never configure logging, only emit messages.

```python
import logging

# Mistake 1: Logging sensitive data
password = "super_secret_123"
logging.warning(f"Login attempt with password: {password}")  # BAD
logging.warning("Login attempt from IP %s", "192.168.1.1")  # GOOD


# Mistake 2: String formatting at call site (f-strings)
for i in range(1000):
    # BAD: evaluates str(i) even if DEBUG is disabled
    logging.debug(f"Processing item {i}")
    # GOOD: lazy evaluation, only formats if DEBUG is enabled
    logging.debug("Processing item %d", i)


# Mistake 3: Setting level on handler but not logger
logger = logging.getLogger("mistake")
handler = logging.StreamHandler()
handler.setLevel(logging.DEBUG)  # This alone does nothing!
logger.addHandler(handler)
logger.setLevel(logging.INFO)  # Logger level blocks DEBUG messages
logger.debug("Will this show?")  # No, blocked by logger level


# Mistake 4: Incorrect handler cleanup creates duplicate handlers
def setup_logger():
    logger = logging.getLogger("duplicate")
    # BAD: adds a new handler every time setup_logger is called
    logger.addHandler(logging.StreamHandler())
    return logger


# Mistake 5: Propagate not disabled for root logger children
logger = logging.getLogger("myapp")
logger.propagate = True  # Messages go to root logger too (usually desirable)
```

## Best Practices

1. **Always use `getLogger(__name__)`** — enables hierarchical configuration per-module.
2. **Configure once at application entry point** — use `basicConfig()` or a config dict; libraries should never configure.
3. **Use lazy formatting** — `logger.info("user %s logged in", user.name)` instead of f-strings.
4. **Rotate log files** — `RotatingFileHandler` or `TimedRotatingFileHandler` with sensible limits.
5. **Use appropriate levels** — DEBUG for diagnostics, INFO for normal events, WARNING for issues, ERROR for failures, CRITICAL for system-down.
6. **Log exceptions with `logger.exception()`** — automatically includes traceback at ERROR level.
7. **Sanitize sensitive data** — filter or mask passwords, tokens, and PII before logging.
8. **Use structured logging** — JSON format for machine parsing (log aggregation systems).
9. **Separate error logs** — send ERROR+ to a separate file for quick incident triage.
10. **Use `logging.config.dictConfig()`** for complex configurations instead of programmatic setup.

```python
import logging.config

# Best practice: dictConfig for complex setups
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
            "level": "WARNING",
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
    },
    "loggers": {
        "myapp": {
            "handlers": ["console", "file"],
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
logger.info("Application configured with dictConfig")
```

## Interview Questions

**Q1: What is the difference between `logging` and `print()` for debugging?**
A: Logging provides severity levels, flexible output destinations, formatting, runtime configuration, and can be turned on/off without code changes. `print()` goes only to stdout and cannot be filtered.

**Q2: How does the logger hierarchy work in Python?**
A: Loggers are organized hierarchically by name (dot-separated). A child logger (e.g., "app.module") inherits settings from its parent ("app"). Messages propagate upward unless `propagate=False` is set. The root logger sits at the top.

**Q3: What is the purpose of `logging.getLogger(__name__)`?**
A: It creates a logger named after the module, enabling fine-grained control over logging per module. Configuration can then target specific package/module trees.

**Q4: How do you ensure lazy evaluation of log messages?**
A: Use `logger.debug("value is %s", value)` with `%s` placeholders and arguments. The string is only formatted if the level is enabled. With f-strings `logger.debug(f"value is {value}")`, the formatting always happens.

**Q5: What is the difference between `logging.error()` and `logging.exception()`?**
A: `logging.exception()` is identical to `logging.error()` but automatically includes the current exception's traceback in the log output. It should only be used inside an `except` block.

**Q6: How do you prevent logging from being a performance bottleneck?**
A: Use lazy formatting, avoid logging in hot loops, use asynchronous handlers like `QueueHandler`/`QueueListener`, and ensure appropriate log levels are set so the logger short-circuits before formatting.

## Coding Challenges

**Challenge 1: Multi-level Logger Setup**
Configure a logger that sends DEBUG and INFO to a file, WARNING to both file and console, and ERROR/CRITICAL to a separate error file with full traceback and timestamps.

**Challenge 2: Structured JSON Logger**
Create a custom LogRecord subclass or Formatter that outputs all log entries as JSON, including timestamp, level, logger name, module, function, line number, message, and any extra context passed via `extra=`.

**Challenge 3: Sensitive Data Masker**
Write a logging Filter that scans log messages for patterns resembling passwords, API keys, SSNs, and credit card numbers and replaces them with `[REDACTED]`.

**Challenge 4: Log Monitor**
Implement a handler that counts ERROR and CRITICAL messages per minute and triggers an alert callback when the count exceeds a configurable threshold.

## Summary

Logging is Python's premier observability framework, offering hierarchical loggers, multiple severity levels, flexible handlers, and customizable formatting. It separates the concerns of producing diagnostic messages from their routing, formatting, and storage. Proper logging practices—using module-level loggers, lazy formatting, log rotation, and structured output—are essential for building production-ready applications. The distinction between configuring logging at application entry points and simply emitting messages from libraries is critical for reusable code. When combined with monitoring and alerting systems, logging provides the foundation for understanding and debugging complex distributed systems.

## Related Topics

- Python's `logging.config` module (file-based and dict-based configuration)
- `logging.handlers` (SysLogHandler, SMTPHandler, HTTPHandler, QueueHandler)
- The `contextvars` module for contextual logging in async applications
- The `structlog` third-party library for advanced structured logging
- Log aggregation systems (ELK Stack, Graylog, Splunk, Datadog)
- Python's `traceback` module for stack trace extraction
- The `warnings` module for non-critical diagnostic messages
- Observability: metrics (prometheus) and tracing (opentelemetry) alongside logging
