# File Operations - open(), read(), write(), with statement

## Introduction

File operations are fundamental to programming, enabling data persistence, inter-program communication, and configuration management. Python's file I/O system provides a clean, portable interface for reading and writing files across all platforms. The core function is `open()`, which returns a file object supporting various reading, writing, and positioning methods.

## open() function

### What It Is

The `open()` function opens a file and returns a file object. It takes the file path and mode as primary arguments, with optional parameters for buffering, encoding, errors handling, and newline behavior. The file object serves as a handle to interact with the underlying operating system file resource.

### Why It Is Important

`open()` is the gateway to all file I/O in Python. Understanding its parameters and modes is essential for correctly reading and writing files, handling different file formats, and managing encoding issues across platforms.

### How It Works Internally

`open()` is a wrapper around the operating system's file descriptor system. It calls the OS kernel to open a file, obtains a file descriptor (an integer), and wraps it in a Python file object. The file object buffers data in memory and uses the descriptor for actual I/O operations. When the file is closed, the descriptor is released back to the OS.

### Syntax

```python
file_object = open(file, mode='r', buffering=-1, encoding=None,
                   errors=None, newline=None, closefd=True, opener=None)

# Common modes:
# 'r'   - Read (default)
# 'w'   - Write (truncates/creates)
# 'a'   - Append (creates if not exists)
# 'x'   - Exclusive creation (fails if exists)
# 'b'   - Binary mode (append to mode: 'rb', 'wb')
# 't'   - Text mode (default: 'rt' same as 'r')
# '+'   - Read and write (append to mode: 'r+', 'w+')
```

```python
# Examples
f = open("file.txt", "r", encoding="utf-8")
f = open("file.txt", "w")
f = open("file.txt", "a")
f = open("image.jpg", "rb")
f = open("data.bin", "wb")
f = open("file.txt", "x")  # Creates new file, fails if exists
f = open("file.txt", "r+")  # Reading and writing
```

### Beginner Examples

```python
# Basic file opening
file = open("example.txt", "r")
content = file.read()
file.close()

# Different modes
# 'r': file must exist
file = open("existing.txt", "r")

# 'w': creates or truncates
file = open("new.txt", "w")  # Creates new or overwrites existing

# 'a': creates or appends
file = open("log.txt", "a")  # Adds to end or creates new

# 'x': exclusive creation
try:
    file = open("unique.txt", "x")
except FileExistsError:
    print("File already exists")

# Binary mode
file = open("image.jpg", "rb")  # Read bytes
file = open("output.bin", "wb")  # Write bytes

# Specifying encoding
file = open("data.txt", "r", encoding="utf-8")
file = open("data.txt", "r", encoding="latin-1")

# Opening for both reading and writing
file = open("data.txt", "r+")  # Must exist, starts at beginning
file = open("data.txt", "w+")  # Creates or truncates, starts at beginning
```

## read() method

### What It Is

The `read()` method reads data from a file. Without arguments, it reads the entire file content as a single string (text mode) or bytes object (binary mode). With a `size` argument, it reads up to that many characters (text mode) or bytes (binary mode).

### Why It Is Important

`read()` is the simplest way to consume file content. Understanding its variants (`read()`, `readline()`, `readlines()`) and when to use each is crucial for efficient file processing, especially with large files where reading the entire content into memory is impractical.

### How It Works Internally

`read()` calls the OS `read()` system call through the file descriptor. The file object's internal buffer fills with data from disk, and Python copies data from the buffer to a new Python string/bytes object. For large files, Python may perform multiple underlying system calls. The file position cursor advances by the number of bytes read.

### Syntax

```python
content = file.read()          # Read entire file as string/bytes
content = file.read(size)      # Read up to size chars/bytes
line = file.readline()         # Read one line (including trailing \n)
lines = file.readlines()       # Read all lines into a list
```

```python
# Reading entire file
with open("data.txt", "r") as f:
    content = f.read()
    print(f"Read {len(content)} characters")

# Reading in chunks
with open("large.txt", "r") as f:
    while True:
        chunk = f.read(1024)  # Read 1KB at a time
        if not chunk:
            break
        process(chunk)

# Reading line by line
with open("data.txt", "r") as f:
    for line in f:  # Most efficient for line-by-line processing
        print(line.strip())

# Using readline
with open("data.txt", "r") as f:
    first_line = f.readline()
    second_line = f.readline()

# Using readlines
with open("data.txt", "r") as f:
    lines = f.readlines()  # All lines into memory
```

### Beginner Examples

```python
# read() - entire file
with open("poem.txt", "r") as f:
    poem = f.read()
    print(poem)

# read(size) - partial read
with open("large.txt", "r") as f:
    first_100 = f.read(100)
    print(f"First 100 chars: {first_100}")

# readline() - single line
with open("names.txt", "r") as f:
    name = f.readline().strip()
    while name:
        print(f"Processing: {name}")
        name = f.readline().strip()

# Iterating over file object (best for line-by-line)
with open("data.csv", "r") as f:
    for line in f:
        fields = line.strip().split(",")
        print(fields)

# readlines() - all lines
with open("items.txt", "r") as f:
    items = [line.strip() for line in f.readlines()]
```

## write() method

### What It Is

The `write()` method writes a string (text mode) or bytes object (binary mode) to a file. It returns the number of characters/bytes written. The `writelines()` method writes an iterable of strings to the file (note: it does NOT add newlines automatically).

### Why It Is Important

`write()` is the primary way to output data to files. Understanding buffering, flushing, and the difference between `write()` and `writelines()` is essential for correct and efficient file writing.

### How It Works Internally

`write()` copies data from Python's string/bytes buffer to the file object's internal buffer. The buffer is flushed to the OS when it fills up, when `flush()` is called, or when the file is closed. The `os` write system call then transfers data from the page cache to disk. For text mode, Python also handles newline translation (`\n` to `\r\n` on Windows).

### Syntax

```python
n = file.write(string)           # Write string, return chars written
file.writelines(iterable)        # Write list of strings (no newlines added)
file.flush()                     # Force write buffer to disk
```

```python
# Basic write
with open("output.txt", "w") as f:
    n = f.write("Hello, World!")
    print(f"Wrote {n} characters")

# Writing multiple lines
with open("lines.txt", "w") as f:
    f.write("First line\n")
    f.write("Second line\n")

# Using writelines
lines = ["Line 1\n", "Line 2\n", "Line 3\n"]
with open("output.txt", "w") as f:
    f.writelines(lines)

# Flushing explicitly
with open("log.txt", "w") as f:
    f.write("Important entry")
    f.flush()  # Ensure it's written immediately

# Binary write
with open("data.bin", "wb") as f:
    f.write(b"\x00\x01\x02\x03")
```

### Beginner Examples

```python
# Basic text writing
with open("hello.txt", "w") as f:
    f.write("Hello, World!\n")
    f.write("This is a file.\n")

# Appending to file
with open("log.txt", "a") as f:
    f.write("New log entry\n")

# Writing user input
name = input("Enter name: ")
with open("user.txt", "w") as f:
    f.write(f"Name: {name}\n")

# Writing numbers
numbers = [1, 2, 3, 4, 5]
with open("numbers.txt", "w") as f:
    for n in numbers:
        f.write(f"{n}\n")

# writelines example
items = ["apple", "banana", "cherry"]
with open("fruits.txt", "w") as f:
    f.writelines(f"{item}\n" for item in items)

# Writing binary data
with open("binary.bin", "wb") as f:
    f.write(bytes(range(256)))

# Writing in chunks
data = "x" * 100000
with open("large_output.txt", "w") as f:
    for i in range(0, len(data), 1000):
        f.write(data[i:i+1000])
```

## with statement

### What It Is

The `with` statement is a context manager that ensures a resource is properly acquired and released. For file objects, it guarantees the file is closed when the block exits, even if an exception occurs. It is the recommended way to handle file operations in Python.

### Why It Is Important

Without `with`, files must be explicitly closed with `file.close()`. If an exception occurs before `close()`, the file remains open, potentially causing resource leaks, file corruption, or running out of file descriptors. `with` eliminates this class of bugs by ensuring cleanup always happens.

### How It Works Internally

When `with` is entered, the file object's `__enter__` method is called, which returns the file object. When the block exits (normally or via exception), `__exit__` is called, which calls `file.close()`. The `__exit__` method receives exception information, allowing it to handle cleanup correctly regardless of how the block terminates.

### Syntax

```python
with open("file.txt", "r") as f:
    content = f.read()
# File is automatically closed here

# Multiple files
with open("input.txt", "r") as src, open("output.txt", "w") as dst:
    dst.write(src.read())

# Parentheses for grouping (Python 3.10+)
with (
    open("input.txt", "r") as src,
    open("output.txt", "w") as dst,
):
    dst.write(src.read())

# Custom context manager
class ManagedFile:
    def __init__(self, filename, mode):
        self.filename = filename
        self.mode = mode

    def __enter__(self):
        self.file = open(self.filename, self.mode)
        return self.file

    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.file:
            self.file.close()
        return False  # Don't suppress exceptions
```

### Beginner Examples

```python
# Simple with statement
with open("data.txt", "r") as f:
    content = f.read()
# File is closed, even if read() raised an exception

# Writing with with
with open("output.txt", "w") as f:
    f.write("Written inside a context manager")
# File is closed

# Multiple files
with open("source.txt", "r") as src, open("dest.txt", "w") as dst:
    dst.write(src.read())

# Equivalent without with (more error-prone):
f = open("data.txt", "r")
try:
    content = f.read()
finally:
    f.close()

# The with statement handles exceptions correctly:
with open("config.txt", "r") as f:
    config = parse(f.read())  # If parse() fails, file is still closed
```

### Intermediate Examples

```python
# Custom context manager for database connections
class DatabaseConnection:
    def __init__(self, conn_string):
        self.conn_string = conn_string
        self.connection = None

    def __enter__(self):
        self.connection = self._connect()
        return self.connection

    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.connection:
            self.connection.close()
        if exc_type is not None:
            print(f"Exception occurred: {exc_type.__name__}")
        return False

    def _connect(self):
        return {"connected": True, "conn_string": self.conn_string}

with DatabaseConnection("db://localhost") as conn:
    print(conn)

# Suppressing exceptions in context manager
class SuppressErrors:
    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type is not None:
            print(f"Suppressed: {exc_type.__name__}: {exc_val}")
            return True  # Suppress the exception
        return False

with SuppressErrors():
    raise ValueError("This error is suppressed")
print("Execution continues")
```

### Advanced Examples

```python
# Atomic file write (write to temp, then rename)
import os
import tempfile

class AtomicWrite:
    def __init__(self, filepath, mode="w", encoding="utf-8"):
        self.filepath = filepath
        self.mode = mode
        self.encoding = encoding

    def __enter__(self):
        dir_name = os.path.dirname(self.filepath) or "."
        fd, self.temp_path = tempfile.mkstemp(dir=dir_name)
        if "b" in self.mode:
            self.file = os.fdopen(fd, self.mode)
        else:
            self.file = os.fdopen(fd, self.mode, encoding=self.encoding)
        return self.file

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.file.close()
        if exc_type is None:
            os.replace(self.temp_path, self.filepath)
        else:
            os.unlink(self.temp_path)
        return False

# Usage
with AtomicWrite("config.json") as f:
    import json
    json.dump({"key": "value"}, f)

# File lock with context manager
import threading

class FileLock:
    def __init__(self, lock_file):
        self.lock_file = lock_file
        self.lock = threading.Lock()

    def __enter__(self):
        self.lock.acquire()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.lock.release()
        return False

# Measuring file operation time
import time

class TimingContext:
    def __init__(self, name="operation"):
        self.name = name

    def __enter__(self):
        self.start = time.perf_counter()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        elapsed = time.perf_counter() - self.start
        print(f"{self.name} took {elapsed:.3f}s")
        return False

with TimingContext("file copy"):
    with open("source.bin", "rb") as src, open("dest.bin", "wb") as dst:
        dst.write(src.read())
```

### Real-World Use Cases

```python
# File copier with progress
import os

def copy_with_progress(src, dst, chunk_size=8192):
    size = os.path.getsize(src)
    copied = 0
    with open(src, "rb") as s, open(dst, "wb") as d:
        while True:
            chunk = s.read(chunk_size)
            if not chunk:
                break
            d.write(chunk)
            copied += len(chunk)
            print(f"\r{copied/size*100:.1f}%", end="")
    print()

# Log file parser
def parse_log(logfile):
    entries = []
    with open(logfile, "r") as f:
        for line in f:
            if "ERROR" in line:
                entries.append(line.strip())
    return entries

# Config updater
def update_config(path, key, value):
    with open(path, "r") as f:
        lines = f.readlines()
    with open(path, "w") as f:
        for line in lines:
            if line.startswith(key + "="):
                f.write(f"{key}={value}\n")
            else:
                f.write(line)

# File splitter
def split_file(filename, lines_per_file=100):
    with open(filename, "r") as f:
        part = 1
        while True:
            lines = []
            for _ in range(lines_per_file):
                line = f.readline()
                if not line:
                    break
                lines.append(line)
            if not lines:
                break
            with open(f"{filename}.part{part}", "w") as out:
                out.writelines(lines)
            part += 1
```

### Common Mistakes

```python
# Mistake 1: Forgetting to close files
f = open("data.txt", "r")
content = f.read()
# File never closed! Use 'with' instead.

# Mistake 2: Reading entire large file
with open("huge.txt", "r") as f:
    lines = f.readlines()  # Loads everything into memory!
    for line in lines:
        process(line)

# Correct:
with open("huge.txt", "r") as f:
    for line in f:  # Streams line by line
        process(line)

# Mistake 3: Not specifying encoding
with open("data.txt", "r") as f:  # Platform-dependent encoding!
    content = f.read()

# Correct:
with open("data.txt", "r", encoding="utf-8") as f:
    content = f.read()

# Mistake 4: Using 'w' when you meant 'a'
with open("log.txt", "w") as f:  # Wipes existing content!
    f.write("New entry")

# Mistake 5: Forgetting 'b' for binary files
with open("image.jpg", "r") as f:  # Wrong! Use 'rb'
    data = f.read()

# Mistake 6: Not handling file-not-found
with open("may_not_exist.txt", "r") as f:  # FileNotFoundError!
    content = f.read()

# Correct:
try:
    with open("may_not_exist.txt", "r") as f:
        content = f.read()
except FileNotFoundError:
    content = ""

# Mistake 7: Assuming write() adds newlines
with open("names.txt", "w") as f:
    f.write("Alice")  # No newline added!
    f.write("Bob")    # Results in "AliceBob"
```

### Best Practices

- Always use `with` statements for file operations
- Specify `encoding="utf-8"` explicitly for text files
- Process large files line by line, not reading entire file
- Use `os.path.join()` or `pathlib.Path` for cross-platform paths
- Catch specific exceptions (`FileNotFoundError`, `PermissionError`)
- Use `tempfile` module for intermediate/temporary data
- Close files as soon as they're no longer needed
- Use binary mode (`rb`, `wb`) for non-text files
- Flush important writes with `file.flush()` before crash-prone operations
- Check if file exists before opening in 'r' mode

### Performance Considerations

File I/O is orders of magnitude slower than memory operations. Key optimizations:
- Use buffered I/O (default `buffering=8192`) for good throughput
- For large file processing: iterate over the file object directly
- For mixed reads/writes, use `r+` mode to avoid reopening
- Consider `mmap` for random access to large files
- Use `shutil.copyfileobj()` for efficient copying
- Write buffers to disk with `flush()` strategically, not after every write
- The `with` statement has negligible overhead compared to manual try/finally

### Interview Questions

1. What is the difference between `read()`, `readline()`, and `readlines()`?

   `read(size)` reads entire file or up to size bytes. `readline()` reads one line including newline. `readlines()` reads all lines into a list. For large files, iterate over the file object directly.

2. Explain the difference between 'w', 'a', and 'x' modes.

   `w`: write, truncates existing or creates new. `a`: append, preserves content, adds to end. `x`: exclusive creation, fails if file exists.

3. How does the `with` statement work for files?

   The file object's `__enter__` returns the file, and `__exit__` calls `file.close()` automatically, even if an exception occurs.

4. What is the purpose of `seek()` and `tell()`?

   `tell()` returns current file position. `seek(offset, whence)` moves the position. `whence=0` (start), 1 (current), 2 (end).

5. How do you handle binary files?

   Open with 'b' in mode ('rb', 'wb'). Read/write `bytes` objects. No encoding or newline translation.

6. What happens if you open a non-existent file in 'w' mode?

   It creates the file. In 'r' mode it raises `FileNotFoundError`.

### Coding Challenges

1. Implement a file splitter that divides a large file into multiple smaller files of N lines each.

2. Write a file difference finder that compares two files and reports additions, deletions, and changes.

3. Create a keyword search utility that searches all files in a directory tree recursively.

4. Implement a log rotation system that automatically archives and compresses log files when they reach a size threshold.

5. Build a hex dump viewer that displays binary files in hexadecimal with ASCII side-by-side.

### Related Topics

- `49_json.md` - JSON file serialization
- `50_csv.md` - CSV file handling
- `51_pickle.md` - Binary serialization
- `52_pathlib.md` - Object-oriented path management
- `53_temp_files.md` - Temporary file management
- `shutil` module - High-level file operations
- `os` module - Low-level OS file interface
- `io` module - Stream-based I/O
