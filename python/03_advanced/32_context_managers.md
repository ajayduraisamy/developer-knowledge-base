# Context Managers - with statement, __enter__, __exit__

## Introduction

A context manager is a Python object that defines a temporary runtime context for a block of code, ensuring that setup and teardown operations are performed automatically. They are used with the `with` statement, which guarantees that resources are acquired and released properly, even if an exception occurs. The classic use case is file handling — `with open(...) as f:` ensures the file is closed when the block exits, regardless of how it exits.

## Why It Is Important

Context managers enforce deterministic resource management, eliminating common bugs like resource leaks (unclosed files, database connections, network sockets, locks). They encapsulate the "before/after" pattern in a reusable, readable way, replacing verbose `try/finally` blocks. They are invaluable for managing transactional operations, temporary state changes, timing code blocks, locking mechanisms, and database connections. Python's standard library includes many context managers, and the `contextlib` module provides tools to create them easily.

## Syntax

```python
# Using a context manager
with context_manager as resource:
    # use resource
    pass

# Multiple context managers (Python 3.1+)
with cm1 as r1, cm2 as r2:
    pass

# Nested context managers
with cm1 as r1:
    with cm2 as r2:
        pass

# Class-based context manager
class MyContextManager:
    def __enter__(self):
        # setup
        return resource
    def __exit__(self, exc_type, exc_val, exc_tb):
        # teardown
        return False  # Don't suppress exception

# Generator-based context manager (contextlib.contextmanager)
from contextlib import contextmanager

@contextmanager
def my_context():
    # setup
    try:
        yield resource
    finally:
        # teardown
        pass
```

## Examples

```python
from contextlib import contextmanager, suppress, redirect_stdout, redirect_stderr, ExitStack
import sys
import io
import time
from typing import Optional, Type, Generator, Any
```

### Using open() as Context Manager

```python
with open('example.txt', 'w') as f:
    f.write('Hello, world!')

# File is automatically closed after the block
```

### Class-based Context Manager

```python
class ManagedFile:
    def __init__(self, filename: str, mode: str = 'r') -> None:
        self.filename = filename
        self.mode = mode

    def __enter__(self) -> 'ManagedFile':
        self.file = open(self.filename, self.mode, encoding='utf-8')
        return self.file

    def __exit__(self, exc_type: Optional[Type[BaseException]],
                 exc_val: Optional[BaseException],
                 exc_tb: Optional[object]) -> Optional[bool]:
        self.file.close()
        if exc_type is not None:
            print(f"Exception: {exc_val}")
        return False  # Don't suppress

with ManagedFile('example.txt', 'r') as f:
    content = f.read()
    print(content)
```

### Generator-based Context Manager

```python
@contextmanager
def managed_file(filename: str, mode: str = 'r') -> Generator[Any, None, None]:
    f = open(filename, mode, encoding='utf-8')
    try:
        yield f
    finally:
        f.close()

with managed_file('example.txt', 'r') as f:
    print(f.read())
```

## Beginner Examples

### Timer Context Manager

```python
@contextmanager
def timer():
    """Measure execution time of a block."""
    start = time.perf_counter()
    try:
        yield
    finally:
        elapsed = time.perf_counter() - start
        print(f"Block took {elapsed:.6f}s")

with timer():
    sum(range(10_000_000))
```

### Handling Exceptions in Context Manager

```python
class SuppressAndLog:
    def __enter__(self) -> 'SuppressAndLog':
        return self

    def __exit__(self, exc_type: Optional[Type[BaseException]],
                 exc_val: Optional[BaseException],
                 exc_tb: Optional[object]) -> Optional[bool]:
        if exc_type is not None:
            print(f"Suppressed: {exc_type.__name__}: {exc_val}")
            return True  # Suppress the exception
        return False

with SuppressAndLog():
    x = 1 / 0
    print("This won't run")

print("Continues after suppressed exception")
```

### redirect_stdout Example

```python
buffer = io.StringIO()
with redirect_stdout(buffer):
    print("This goes to buffer")
    print("This also goes to buffer")
output = buffer.getvalue()
print(f"Captured: {output.strip()}")
```

## Intermediate Examples

### contextlib.suppress

```python
# Instead of:
try:
    os.remove("somefile.txt")
except FileNotFoundError:
    pass

# Use:
from contextlib import suppress
import os

with suppress(FileNotFoundError):
    os.remove("somefile.txt")
```

### Database Connection Context Manager

```python
import sqlite3

class DatabaseConnection:
    def __init__(self, db_path: str) -> None:
        self.db_path = db_path
        self.connection: Optional[sqlite3.Connection] = None

    def __enter__(self) -> sqlite3.Connection:
        self.connection = sqlite3.connect(self.db_path)
        return self.connection

    def __exit__(self, exc_type: Optional[Type[BaseException]],
                 exc_val: Optional[BaseException],
                 exc_tb: Optional[object]) -> Optional[bool]:
        if self.connection:
            if exc_type is None:
                self.connection.commit()
            else:
                self.connection.rollback()
            self.connection.close()
        return False

# with DatabaseConnection('test.db') as conn:
#     conn.execute("CREATE TABLE IF NOT EXISTS test (id INT)")
#     conn.execute("INSERT INTO test VALUES (1)")
```

### Nested Context Managers

```python
@contextmanager
def indent(level: int = 1) -> Generator[None, None, None]:
    print("  " * level + "Enter", end=" -> ")
    try:
        yield
    finally:
        print("  " * level + "Exit")

with indent(0):
    print("Outer")
    with indent(1):
        print("Inner")
        with indent(2):
            print("Innermost")
```

### ExitStack for Dynamic Context Managers

```python
@contextmanager
def named_context(name: str) -> Generator[str, None, None]:
    print(f"Entering {name}")
    try:
        yield name
    finally:
        print(f"Exiting {name}")

names = ["A", "B", "C"]
with ExitStack() as stack:
    resources = [stack.enter_context(named_context(n)) for n in names]
    print("Inside with all:", resources)

print("After ExitStack")
```

## Advanced Examples

### Transaction Context Manager (Commit/Rollback)

```python
class Transaction:
    """Simulates a database transaction."""

    def __init__(self) -> None:
        self.operations: list[str] = []
        self.committed = False

    def add(self, operation: str) -> None:
        self.operations.append(operation)
        print(f"  Added: {operation}")

    def __enter__(self) -> 'Transaction':
        print("BEGIN TRANSACTION")
        return self

    def __exit__(self, exc_type: Optional[Type[BaseException]],
                 exc_val: Optional[BaseException],
                 exc_tb: Optional[object]) -> Optional[bool]:
        if exc_type is None:
            self.committed = True
            print(f"COMMIT ({len(self.operations)} operations)")
        else:
            print(f"ROLLBACK due to {exc_type.__name__}: {exc_val}")
        return False

trans = Transaction()
try:
    with trans:
        trans.add("INSERT INTO users ...")
        trans.add("UPDATE accounts ...")
        raise ValueError("Something went wrong")
except ValueError:
    pass
print(f"Committed: {trans.committed}")
```

### Timeout Context Manager

```python
import signal
from contextlib import contextmanager

class TimeoutError(Exception):
    pass

@contextmanager
def timeout(seconds: int):
    """Raise TimeoutError if block runs longer than seconds."""

    def handler(signum, frame):
        raise TimeoutError(f"Timed out after {seconds}s")

    original_handler = signal.signal(signal.SIGALRM, handler)
    signal.alarm(seconds)
    try:
        yield
    finally:
        signal.alarm(0)
        signal.signal(signal.SIGALRM, original_handler)

# with timeout(5):
#     time.sleep(10)  # Would raise TimeoutError
```

### Temporary Directory Context Manager

```python
import tempfile
import shutil
from pathlib import Path

@contextmanager
def temporary_directory() -> Generator[Path, None, None]:
    """Create a temporary directory that is cleaned up on exit."""
    tmp_dir = tempfile.mkdtemp()
    try:
        yield Path(tmp_dir)
    finally:
        shutil.rmtree(tmp_dir, ignore_errors=True)

with temporary_directory() as tmp:
    test_file = tmp / "test.txt"
    test_file.write_text("Temporary content")
    print(f"File exists inside: {test_file.exists()}")

print(f"File exists outside: {test_file.exists()}")
```

### Redirecting Multiple Streams

```python
@contextmanager
def redirect_both(out_file: str, err_file: str) -> Generator[None, None, None]:
    """Redirect both stdout and stderr to files."""
    with open(out_file, 'w') as out, open(err_file, 'w') as err:
        with redirect_stdout(out), redirect_stderr(err):
            yield

# with redirect_both('out.log', 'err.log'):
#     print("This goes to out.log")
#     print("This goes to err.log", file=sys.stderr)
```

### Context Manager that Returns Different Resources Based on Conditions

```python
@contextmanager
def get_connection(database: str) -> Generator[str, None, None]:
    """Mock: returns different connections based on database name."""
    if database == "sqlite":
        conn = f"SQLite connection to {database}"
    elif database == "postgres":
        conn = f"PostgreSQL connection to {database}"
    else:
        raise ValueError(f"Unknown database: {database}")
    print(f"  Opening {conn}")
    try:
        yield conn
    finally:
        print(f"  Closing {conn}")

with get_connection("sqlite") as conn:
    print(f"  Using: {conn}")
```

### Reentrant Context Manager

```python
class Reentrant:
    """Context manager that can be entered multiple times."""

    def __init__(self) -> None:
        self.level = 0

    def __enter__(self) -> 'Reentrant':
        self.level += 1
        print(f"Enter (level {self.level})")
        return self

    def __exit__(self, *args: Any) -> None:
        print(f"Exit (level {self.level})")
        self.level -= 1

r = Reentrant()
with r:
    with r:
        with r:
            print("Deeply nested")
```

### Measuring with Context Manager

```python
@contextmanager
def profile(name: str = "block") -> Generator[None, None, None]:
    """Profile CPU and wall time of a block."""
    import cProfile, pstats, io
    profiler = cProfile.Profile()
    profiler.enable()
    start = time.perf_counter()
    try:
        yield
    finally:
        elapsed = time.perf_counter() - start
        profiler.disable()
        s = io.StringIO()
        ps = pstats.Stats(profiler, stream=s).sort_stats('cumtime')
        ps.print_stats(5)
        print(f"--- {name}: {elapsed:.4f}s ---")
        print(s.getvalue())

# with profile("test"):
#     sum(range(10_000_000))
```

## Real-World Use Cases

- **File I/O**: `with open(...) as f:` — automatic close.
- **Threading locks**: `with lock:` — acquire/release.
- **Database connections**: `with connection:` — commit/rollback.
- **Subprocess management**: `with subprocess.Popen(...)` — wait/cleanup.
- **Decimal arithmetic context**: `with decimal.localcontext():` — temporary precision.
- **Numpy errstate**: `with np.errstate(divide='raise'):` — temporary error handling.
- **Mock objects in tests**: `with unittest.mock.patch(...):` — temporary patching.
- **HTML generation**: `with html_element("div"):` — tag nesting.
- **Network connections**: `with socket.create_connection(...):` — connection cleanup.

## Common Mistakes

- Not returning a value from `__enter__` — the `as` variable will be `None`.
- Returning `False` from `__exit__` when you want to suppress exceptions (should return `True`).
- Forgetting to re-raise exceptions in `__exit__` when you don't want to suppress them.
- Using `@contextmanager` on a generator that doesn't wrap `yield` in `try/finally`.
- Using multiple context managers in one `with` statement when error handling differs.
- Forgetting that `__exit__` arguments will be `None` when no exception occurs.
- Modifying resources inside the with block that invalidate the cleanup logic.

## Best Practices

- Use `@contextmanager` for simple cases; use class-based for complex state management.
- Always wrap `yield` in `try/finally` when using `@contextmanager`.
- Return `True` from `__exit__` only when you intentionally suppress exceptions.
- Use `contextlib.suppress` instead of bare `try/except: pass` for ignoring errors.
- Use `contextlib.ExitStack` for dynamic or unknown numbers of context managers.
- Document what resource the context manager provides (or `None` if nothing).
- Use `contextlib.redirect_stdout`/`redirect_stderr` for capturing output in tests.
- For async operations, use `async with` and `__aenter__`/`__aexit__`.

## Interview Questions

1. How does the `with` statement work in Python?
2. What methods must a context manager implement?
3. What is the purpose of the `__exit__` method's parameters?
4. What does it mean to "suppress an exception" in a context manager?
5. How do you create a context manager using a generator?
6. What is `contextlib.ExitStack` and when would you use it?
7. How does `contextlib.suppress` differ from `try/except: pass`?
8. Can you use multiple context managers in a single `with` statement?
9. How do you create a context manager for database transactions?
10. What is the difference between class-based and generator-based context managers?

## Coding Challenges

1. **File Open with Retry**: Implement a context manager that retries opening a file on failure.
2. **Atomic File Write**: Create a context manager that writes to a temporary file, then renames it on success.
3. **Context Timer**: Measure and report the execution time of a block.
4. **Logging Context**: Create a context manager that temporarily changes the logging level.
5. **Environment Variable**: Temporarily set an environment variable, restoring on exit.
6. **Directory Changer**: Implement `cd` as a context manager that changes and restores the working directory.
7. **Performance Counter**: Count how many times a value is accessed inside the context.
8. **Lock with Timeout**: Implement a context manager that acquires a threading lock with a timeout.

## Summary

Context managers provide a clean, Pythonic way to manage resources and encapsulate setup/teardown logic using the `with` statement. They can be implemented as classes with `__enter__`/`__exit__` or as generator functions with `@contextlib.contextmanager`. The `contextlib` module offers utilities like `suppress`, `redirect_stdout`, `ExitStack`, and `closing` to simplify common patterns.

## Related Topics

- with statement
- try/finally blocks
- contextlib module
- Resource management
- File I/O
- Database transactions
- Threading and locks
- Async context managers (__aenter__, __aexit__)