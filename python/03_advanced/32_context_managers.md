# Context Managers - with statement, __enter__, __exit__

## Introduction

Context managers in Python provide a clean and robust way to manage resources that require setup and teardown operations. The `with` statement automates the acquisition and release of resources, ensuring that cleanup code always runs even if exceptions occur. Common use cases include file handling, database connections, locks, and network connections.

## with Statement

### What It Is

The `with` statement in Python simplifies exception-safe resource management. It wraps the execution of a block of code with methods defined by a context manager object. This ensures that resources are properly acquired before the block runs and released after it completes, regardless of whether the block exits normally or via an exception.

### Why It Is Important

Before `with`, resource cleanup required explicit `try/finally` blocks, which were verbose and error-prone. The `with` statement reduces boilerplate, eliminates resource leaks, and makes code more readable. It's especially important for ensuring that files are closed, locks are released, and database connections are returned to pools automatically.

### How It Works Internally

When Python executes `with expr as var:`:

1. It evaluates `expr` to get a context manager object.
2. It calls the context manager's `__enter__()` method and assigns the result to `var` (if `as` is used).
3. It executes the body of the `with` block.
4. Regardless of how the body exits (normally, by exception, `return`, `break`, or `continue`), it calls the context manager's `__exit__()` method.
5. If `__exit__()` receives an exception and returns `True`, the exception is suppressed. If it returns `False`, the exception propagates.

### Syntax

```python
with context_manager as variable:
    # Body
    pass

# Multiple context managers
with cm1 as v1, cm2 as v2:
    pass

# Nested (Python 3.1+)
with cm1 as v1:
    with cm2 as v2:
        pass
```

### Beginner Examples

```python
# File handling (the classic example)
with open("file.txt", "r") as f:
    content = f.read()
# File is automatically closed, even if an error occurs

# Without 'with', you'd need:
f = open("file.txt", "r")
try:
    content = f.read()
finally:
    f.close()
```

### Intermediate Examples

```python
# Multiple context managers
with open("input.txt") as infile, open("output.txt", "w") as outfile:
    for line in infile:
        outfile.write(line.upper())

# Lock management
import threading
lock = threading.Lock()
with lock:
    # Critical section - lock is automatically released
    shared_counter += 1

# Database transaction
import sqlite3
conn = sqlite3.connect("data.db")
with conn as c:
    c.execute("UPDATE users SET name = ? WHERE id = ?", ("Alice", 1))
# conn.commit() is called on success, conn.rollback() on exception
```

### Advanced Examples

```python
import contextlib
import tempfile
import shutil
import os

@contextlib.contextmanager
def temporary_directory():
    """Create and clean up a temporary directory."""
    dirpath = tempfile.mkdtemp()
    try:
        yield dirpath
    finally:
        shutil.rmtree(dirpath)

with temporary_directory() as tmpdir:
    filepath = os.path.join(tmpdir, "test.txt")
    with open(filepath, "w") as f:
        f.write("Hello")
    print(os.listdir(tmpdir))
# tmpdir is cleaned up automatically

# Nested context managers for complex setup
class ManagedDatabase:
    def __init__(self, db_name):
        self.db_name = db_name
        self.conn = None

    def __enter__(self):
        self.conn = sqlite3.connect(self.db_name)
        return self.conn.cursor()

    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type is None:
            self.conn.commit()
        else:
            self.conn.rollback()
        self.conn.close()
        return False  # Don't suppress exceptions
```

### Real-World Use Cases

- **File I/O**: `with open(...) as f` for automatic file closing.
- **Threading locks**: `with lock` for safe lock acquisition and release.
- **Database connections**: Automatically commit/rollback transactions.
- **Network connections**: Ensure sockets are properly closed.
- **Subprocess management**: `subprocess.Popen` as context manager.
- **Redirection**: Temporarily redirect stdout/stderr via `contextlib.redirect_stdout`.
- **Testing**: `unittest.mock.patch` as context manager.

### Common Mistakes

- Suppressing exceptions unintentionally by returning `True` from `__exit__`.
- Using `with` with objects that don't implement the context manager protocol.
- Assuming `as var` is required (it's optional — use when you need the return value of `__enter__`).
- Forgetting that `__exit__` receives three arguments even if you don't use them.
- Not handling cleanup in `__exit__` for partially acquired resources.

### Best Practices

- Use `with` whenever you acquire resources that need cleanup.
- Keep the `with` block as short as possible to release resources promptly.
- Use `contextlib.contextmanager` for simple context managers instead of writing a class.
- Return `False` from `__exit__` to let exceptions propagate naturally.
- Handle partial setup in `__enter__` by wrapping initialization in a try/finally.

### Performance Considerations

Context managers add minimal overhead — typically two method calls per `with` block. The real performance benefit is avoiding resource leaks and ensuring prompt cleanup. For long-running `with` blocks, remember that resources (like file handles) are held until the block exits.

### Interview Questions

**Q: What happens if an exception occurs inside a `with` block?**

A: The `__exit__` method is called with the exception type, value, and traceback. If `__exit__` returns `False` (default), the exception propagates. If it returns `True`, the exception is suppressed.

**Q: Can a context manager suppress an exception?**

A: Yes. If `__exit__` returns `True`, the exception is suppressed. This is useful for cases like `contextlib.suppress` or when you want to handle specific exceptions gracefully.

### Coding Challenges

1. Write a `@contextmanager` that measures the execution time of a block.
2. Implement a `change_directory(path)` context manager that temporarily changes the working directory.
3. Create a context manager that acquires and releases a semaphore with a timeout.
4. Build a context manager that redirects `sys.stdout` to a file temporarily.

### Related Topics

- `contextlib` module (`contextmanager`, `closing`, `suppress`, `redirect_stdout`)
- `try/finally` blocks (manual resource management)
- `__del__` method (destructor-based cleanup, less reliable)
- Async context managers (`__aenter__`, `__aexit__`, `async with`)

## __enter__ and __exit__

### What It Is

`__enter__` and `__exit__` are the magic methods that define a class as a context manager. `__enter__` is called when entering the `with` block and should return the resource to be bound to the `as` variable. `__exit__` is called when leaving the `with` block and handles cleanup logic.

### Why It Is Important

These methods enable any class to participate in the `with` statement protocol, making resource management a first-class concept in Python. By implementing these methods, you create objects that guarantee proper setup and teardown, which is critical for robust resource management in production code.

### How It Works Internally

When `with obj:` executes:

1. `obj.__enter__()` is called immediately. Its return value is assigned to the `as` variable.
2. The body runs. If an exception occurs, `sys.exc_info()` captures it.
3. `obj.__exit__(exc_type, exc_val, exc_tb)` is called:
   - If no exception: all three args are `None`.
   - If exception: args are the exception type, value, and traceback.
4. If `__exit__` returns a truthy value and an exception occurred, the exception is swallowed.

### Syntax

```python
class MyContextManager:
    def __enter__(self):
        # Setup code
        self.resource = acquire_resource()
        return self.resource  # Bound to 'as' variable

    def __exit__(self, exc_type, exc_val, exc_tb):
        # Cleanup code
        release_resource(self.resource)
        # Return False to propagate exceptions, True to suppress
        return False
```

### Beginner Examples

```python
class ManagedFile:
    def __init__(self, filename, mode="r"):
        self.filename = filename
        self.mode = mode
        self.file = None

    def __enter__(self):
        self.file = open(self.filename, self.mode)
        return self.file

    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.file:
            self.file.close()
        # Don't suppress exceptions
        return False

# Usage
with ManagedFile("hello.txt", "w") as f:
    f.write("Hello, World!")
```

### Intermediate Examples

```python
class Connection:
    def __init__(self, host, port):
        self.host = host
        self.port = port
        self.connected = False

    def __enter__(self):
        print(f"Connecting to {self.host}:{self.port}")
        self.connected = True
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("Disconnecting")
        self.connected = False
        if exc_type is not None:
            print(f"Error during connection: {exc_val}")
        # Return True to suppress ConnectionError only
        return exc_type is ConnectionError

    def send(self, data):
        if not self.connected:
            raise RuntimeError("Not connected")
        print(f"Sending: {data}")

with Connection("localhost", 8080) as conn:
    conn.send("Hello")
```

### Advanced Examples

```python
class ResourcePool:
    def __init__(self, resource_factory, max_size=5):
        self.factory = resource_factory
        self.max_size = max_size
        self.pool = []
        self.in_use = 0

    def __enter__(self):
        if self.pool:
            resource = self.pool.pop()
        elif self.in_use < self.max_size:
            resource = self.factory()
            self.in_use += 1
        else:
            raise RuntimeError("Pool exhausted")
        self.current = resource
        return resource

    def __exit__(self, exc_type, exc_val, exc_tb):
        if hasattr(self, 'current'):
            self.pool.append(self.current)
            del self.current
        return False  # Don't suppress exceptions

class TransactionContext:
    def __init__(self, db_connection):
        self.db = db_connection
        self.nested = []

    def __enter__(self):
        if self.nested:
            savepoint = f"sp_{len(self.nested)}"
            self.db.execute(f"SAVEPOINT {savepoint}")
            self.nested.append(savepoint)
        else:
            self.db.execute("BEGIN")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type is None:
            if self.nested:
                savepoint = self.nested.pop()
                self.db.execute(f"RELEASE SAVEPOINT {savepoint}")
            else:
                self.db.execute("COMMIT")
        else:
            if self.nested:
                savepoint = self.nested.pop()
                self.db.execute(f"ROLLBACK TO SAVEPOINT {savepoint}")
            else:
                self.db.execute("ROLLBACK")
        return False
```

### Real-World Use Cases

- **Database connection pools**: Acquire and return connections to a pool.
- **Timer/Profiler**: Measure execution time of code blocks.
- **Transaction management**: Handle nested transactions with savepoints.
- **Thread pool executors**: `concurrent.futures.ThreadPoolExecutor` as context manager.
- **Change tracking**: Record all state changes in a block and rollback on error.

### Common Mistakes

- Accessing `exc_type`, `exc_val`, `exc_tb` without checking if they're `None`.
- Returning `True` from `__exit__` without explicitly meaning to suppress exceptions.
- Forgetting to handle partial initialization (what if `__enter__` partially succeeds?).
- Assuming `__exit__` won't be called if `__enter__` raises an exception.
- Not calling `super().__exit__()` in subclasses of context managers.

### Best Practices

- Use `__enter__` only for setup that needs matching teardown in `__exit__`.
- In `__exit__`, always clean up resources regardless of whether an exception occurred.
- Return `False` from `__exit__` unless you specifically want to suppress exceptions.
- Handle the case where `__enter__` might fail after partially acquiring resources.
- Keep `__exit__` simple and avoid raising exceptions (they mask the original exception).

### Performance Considerations

The overhead of `__enter__`/`__exit__` is two method calls. For resource-intensive operations (like network connections), the overhead is negligible. For very hot code paths, consider inlining resource management or using `try/finally` with manually managed resources.

### Interview Questions

**Q: When is `__exit__` called with all three arguments as `None`?**

A: When the `with` block exits without an exception. If the block raises an exception, `__exit__` receives the exception type, value, and traceback.

**Q: Can `__exit__` raise an exception?**

A: Yes, but it's strongly discouraged. If `__exit__` raises, it replaces the original exception (if any), making debugging very difficult. Always handle cleanup errors gracefully inside `__exit__`.

### Coding Challenges

1. Implement a `Timer` context manager that prints elapsed time on exit.
2. Create a `atomic_write(filepath)` context manager that writes to a temp file and renames on success.
3. Implement a `try_except(*exception_types)` context manager that catches specific exceptions.
4. Build a `redirect_stdin(filepath)` context manager that temporarily redirects stdin.

### Related Topics

- `contextlib.contextmanager` (generator-based alternative)
- `with` statement semantics
- `try/finally` pattern
- `contextlib.closing` (for objects with `close()` but no context manager)
- Async context managers (`async with`)

## contextlib Module

### What It Is

The `contextlib` module provides utilities for working with context managers. It includes a decorator for creating context managers from generator functions (`@contextmanager`), a context manager for closing objects (`closing`), a way to suppress exceptions (`suppress`), and tools for composing and redirecting.

### Why It Is Important

`contextlib` reduces boilerplate by providing pre-built context managers for common patterns and a convenient way to create custom ones without writing a full class. The `@contextmanager` decorator is especially useful for simple resource management where a class would be overkill.

### How It Works Internally

`@contextmanager` wraps a generator function in a context manager class. The `__enter__` method creates the generator and calls `next()` to reach the `yield`. The `__exit__` method either calls `next()` again (on normal exit) or `throw()` (on exception). The value after `yield` is the return value of `__enter__`.

### Syntax

```python
from contextlib import contextmanager, closing, suppress, redirect_stdout

@contextmanager
def my_context():
    # Setup (runs on __enter__)
    resource = acquire()
    try:
        yield resource  # Value for 'as' variable
    finally:
        # Cleanup (runs on __exit__)
        release(resource)
```

### Beginner Examples

```python
from contextlib import contextmanager

@contextmanager
def open_file(filename, mode="r"):
    file = open(filename, mode)
    try:
        yield file
    finally:
        file.close()

with open_file("test.txt", "w") as f:
    f.write("Hello from contextmanager!")

# Built-in closing
from contextlib import closing
import urllib.request

with closing(urllib.request.urlopen("https://python.org")) as page:
    for line in page:
        if line.strip():
            print(line[:50])
```

### Intermediate Examples

```python
from contextlib import contextmanager, suppress
import os

@contextmanager
def change_directory(path):
    """Temporarily change working directory."""
    old_cwd = os.getcwd()
    os.chdir(path)
    try:
        yield
    finally:
        os.chdir(old_cwd)

with change_directory("/tmp"):
    print("Inside:", os.getcwd())
print("Outside:", os.getcwd())

# Suppress specific exceptions
with suppress(FileNotFoundError):
    os.remove("does_not_exist.txt")
print("No error raised")

# Redirect stdout
from contextlib import redirect_stdout
import io

buffer = io.StringIO()
with redirect_stdout(buffer):
    print("This goes to buffer")
print("Captured:", buffer.getvalue())
```

### Advanced Examples

```python
from contextlib import contextmanager, ExitStack
import os
import tempfile
import shutil

@contextmanager
def temporary_environment():
    """Create and clean up a temporary working environment."""
    temp_dir = tempfile.mkdtemp()
    old_cwd = os.getcwd()
    try:
        os.chdir(temp_dir)
        yield temp_dir
    finally:
        os.chdir(old_cwd)
        shutil.rmtree(temp_dir)

# ExitStack for dynamic context management
def process_files(filenames):
    with ExitStack() as stack:
        files = [stack.enter_context(open(fname)) for fname in filenames]
        # Files will be closed in reverse order on exit
        for lines in zip(*files):
            yield lines

# Async context manager support
from contextlib import asynccontextmanager

@asynccontextmanager
async def async_resource():
    print("Acquiring async resource")
    try:
        yield "resource"
    finally:
        print("Releasing async resource")

# async with async_resource() as res:
#     print(f"Using {res}")
```

### Real-World Use Cases

- **Testing**: `unittest.mock.patch` as context manager via `contextlib`.
- **Timing**: Custom `@contextmanager` to measure and log execution time.
- **Configuration**: Temporarily override configuration values.
- **Resource pooling**: Acquire resources from a pool and return them automatically.
- **Error isolation**: `suppress` to ignore expected, harmless exceptions.
- **Deferred cleanup**: `ExitStack` to manage dynamic collections of context managers.

### Common Mistakes

- Forgetting the `try/finally` inside a `@contextmanager` generator (cleanup may not run).
- Using `@contextmanager` on a generator that yields more than once (raises `RuntimeError`).
- Catching exceptions inside `@contextmanager` without re-raising to let them propagate.
- Not wrapping the `yield` in `try/finally` when using `@contextmanager`.
- Using `ExitStack.enter_context` in the wrong order (resources are closed in reverse order).

### Best Practices

- Use `@contextmanager` for simple context managers that don't need a class.
- Always wrap `yield` in `try/finally` with `@contextmanager` for reliable cleanup.
- Use `suppress` instead of empty `try/except` blocks.
- Use `ExitStack` when the number of context managers is dynamic or unknown at definition time.
- Use `closing` for objects with a `close()` method but no context manager protocol.

### Performance Considerations

`@contextmanager` has slightly more overhead than a custom class because each `__enter__`/`__exit__` involves creating and advancing a generator. For performance-critical code, implement a class directly. `ExitStack` is efficient for managing collections but has per-context-manager overhead.

### Interview Questions

**Q: What is the difference between `contextmanager` and writing a class with `__enter__`/`__exit__`?**

A: `@contextmanager` is more concise but slightly slower. Class-based context managers offer more control (e.g., multiple methods, state inspection) and can be more efficient. Use `@contextmanager` for simple cases, classes for complex ones.

**Q: What is `ExitStack` and when would you use it?**

A: `ExitStack` is a context manager that allows dynamic management of multiple context managers. It's useful when you don't know at definition time how many context managers you need, such as processing a variable list of files or database connections.

### Coding Challenges

1. Write a `@contextmanager` that temporarily adds a path to `sys.path`.
2. Use `ExitStack` to create a function that opens N files and ensures all are closed.
3. Implement a `@contextmanager` that acquires a lock and logs the acquisition time.
4. Create a `timeout(seconds)` context manager using `signal.SIGALRM` or threading.

### Related Topics

- `with` statement
- Generator functions (used by `@contextmanager`)
- `try/finally` blocks
- `contextvars` (context variables for async context managers)
- `asyncio` (async context managers with `@asynccontextmanager`)
