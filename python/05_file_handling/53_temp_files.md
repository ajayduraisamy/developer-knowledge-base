# Temporary Files - tempfile.TemporaryFile, TemporaryDirectory

## Introduction

The `tempfile` module provides secure, cross-platform functions for creating temporary files and directories. These temporary resources are created in system-specific temporary directories, have unique names to prevent collisions, and can be automatically cleaned up. The module handles important details like unique naming, proper permissions, and cleanup, reducing security vulnerabilities and resource leaks.

## TemporaryFile

### What It Is

`TemporaryFile()` creates an anonymous temporary file that has no visible name in the filesystem (on Unix, the file is unlinked immediately after creation). It returns a file-like object that can be read and written. When closed (or when the context manager exits), the file is automatically deleted.

### Why It Is Important

`TemporaryFile` is ideal for storing intermediate data that doesn't need to persist or be shared. It's automatically cleaned up, immune to name collisions, and has restricted permissions. The anonymous nature (no visible filename) provides additional security—other processes cannot open the file.

### How It Works Internally

`TemporaryFile()` calls `mkstemp()` to create a temporary file with a unique name in the system's temp directory. On Unix, it immediately unlinks the file, removing the directory entry but keeping the file descriptor open. This means the file exists in the filesystem (has inode, disk space) but has no name—it can only be accessed through the file descriptor. When the file is closed, the descriptor is released and the space is reclaimed.

### Syntax

```python
import tempfile

# Basic usage with context manager
with tempfile.TemporaryFile(mode="w+b", buffering=-1, encoding=None,
                            newline=None, suffix=None, prefix=None,
                            dir=None, errors=None) as f:
    f.write(b"data")
    f.seek(0)
    content = f.read()
# File is automatically deleted when block exits

# Text mode
with tempfile.TemporaryFile(mode="w+", encoding="utf-8") as f:
    f.write("Hello, World!")
    f.seek(0)
    print(f.read())
```

### Beginner Examples

```python
import tempfile

# Basic usage
with tempfile.TemporaryFile() as f:
    f.write(b"Hello, temporary world!")
    f.seek(0)
    content = f.read()
    print(content)
# File is automatically deleted here

# Text mode
with tempfile.TemporaryFile(mode="w+", encoding="utf-8") as f:
    f.write("Text mode works too")
    f.seek(0)
    print(f.read())

# Writing binary data
with tempfile.TemporaryFile(mode="w+b") as f:
    f.write(b"binary data")
    f.seek(0)
    print(f.read())  # b'binary data'

# Specifying prefix and suffix
with tempfile.TemporaryFile(prefix="myapp_", suffix=".tmp") as f:
    f.write(b"data")
    # The file has a name like myapp_XXXXX.tmp
```

### Intermediate Examples

```python
import tempfile
import os

# Using TemporaryFile for data processing
def process_large_data(items):
    """Process items using temporary file for intermediate storage."""
    with tempfile.TemporaryFile(mode="w+") as f:
        for item in items:
            f.write(f"{item}\n")

        f.seek(0)
        result = []
        for line in f:
            result.append(line.strip())
    return result

data = [f"Item {i}" for i in range(1000)]
result = process_large_data(data)
print(f"Processed {len(result)} items")

# NamedTemporaryFile (has a visible name)
with tempfile.NamedTemporaryFile(delete=True) as f:
    print(f"Temp file name: {f.name}")
    f.write(b"Named temporary file")
    f.flush()
    # The file exists and has a name
    print(f"File size: {os.path.getsize(f.name)} bytes")
# File is deleted when context exits

# NamedTemporaryFile with delete=False (manual cleanup)
with tempfile.NamedTemporaryFile(delete=False, prefix="keep_") as f:
    f.write(b"This file will persist")
    temp_name = f.name

print(f"File still exists: {os.path.exists(temp_name)}")
os.unlink(temp_name)  # Must clean up manually
```

## TemporaryDirectory

### What It Is

`TemporaryDirectory()` creates a temporary directory with a unique name. It can be used as a context manager, and when the context exits, the entire directory and all its contents are automatically deleted. It returns the directory path as a string.

### Why It Is Important

`TemporaryDirectory` is essential when you need to create multiple temporary files as a group (e.g., during testing, extraction of archives, multi-step processing). It provides atomic cleanup of all created files, which is much safer than attempting to clean up individual files.

### How It Works Internally

`TemporaryDirectory()` calls `mkdtemp()` to create a unique directory in the system temp directory. When used as a context manager, it registers cleanup via `atexit` module for process termination cases. On context exit, it uses `shutil.rmtree()` to remove the directory and all contents.

### Syntax

```python
import tempfile

with tempfile.TemporaryDirectory(suffix=None, prefix=None, dir=None) as tmpdir:
    print(f"Temp directory: {tmpdir}")
    # Create files inside tmpdir
    file_path = Path(tmpdir) / "data.txt"
    file_path.write_text("Hello!")
# Directory and all contents are automatically deleted
```

### Beginner Examples

```python
import tempfile
from pathlib import Path

# Basic TemporaryDirectory
with tempfile.TemporaryDirectory() as tmp_dir:
    print(f"Temp directory: {tmp_dir}")

    # Create files in the temp directory
    file1 = Path(tmp_dir) / "test1.txt"
    file1.write_text("Hello from temp dir!")

    file2 = Path(tmp_dir) / "sub" / "test2.txt"
    file2.parent.mkdir()
    file2.write_text("Nested file")

    print(f"Files created: {list(Path(tmp_dir).rglob('*'))}")
# Directory and all contents are automatically deleted

# Specifying prefix
with tempfile.TemporaryDirectory(prefix="myapp_") as tmp_dir:
    print(tmp_dir)  # /tmp/myapp_XXXXXXXX/

# Multiple temp directories
with tempfile.TemporaryDirectory() as input_dir, \
     tempfile.TemporaryDirectory() as output_dir:
    # Process files from input_dir to output_dir
    pass
```

### Intermediate Examples

```python
import tempfile
import os
import shutil
from pathlib import Path

# Temporary environment for testing
class TestEnvironment:
    def __init__(self):
        self.temp_dir = tempfile.mkdtemp()
        self.files = {}

    def create_file(self, name, content=""):
        file_path = Path(self.temp_dir) / name
        file_path.parent.mkdir(parents=True, exist_ok=True)
        file_path.write_text(content)
        self.files[name] = str(file_path)
        return str(file_path)

    def cleanup(self):
        shutil.rmtree(self.temp_dir)

    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.cleanup()
        return False

# Using context manager form
with tempfile.TemporaryDirectory() as tmpdir:
    # Simulate file processing pipeline
    input_file = Path(tmpdir) / "input.txt"
    input_file.write_text("raw data")

    processed = Path(tmpdir) / "processed.txt"
    processed.write_text(input_file.read_text().upper())

    output_file = Path(tmpdir) / "output.txt"
    output_file.write_text(processed.read_text())

    print(f"Pipeline complete in {tmpdir}")
    for f in Path(tmpdir).iterdir():
        print(f"  {f.name}: {f.read_text()}")
# Everything cleaned up

# Temporary database for testing
import sqlite3

with tempfile.TemporaryDirectory() as tmpdir:
    db_path = Path(tmpdir) / "test.db"
    conn = sqlite3.connect(str(db_path))
    conn.execute("CREATE TABLE users (id INT, name TEXT)")
    conn.execute("INSERT INTO users VALUES (1, 'Alice')")
    result = conn.execute("SELECT * FROM users").fetchall()
    print(f"DB result: {result}")
    conn.close()
# Database file cleaned up
```

### Advanced Examples

```python
import tempfile
import os
import threading
from contextlib import contextmanager
from pathlib import Path

# Temp file pool for concurrent processing
class TempFilePool:
    def __init__(self, max_files=10, prefix="pool_", suffix=".tmp"):
        self.max_files = max_files
        self.temp_dir = tempfile.mkdtemp(prefix="filepool_")
        self.available = []
        self.in_use = set()
        self.lock = threading.Lock()
        self._preallocate()

    def _preallocate(self):
        for _ in range(self.max_files):
            fd, path = tempfile.mkstemp(
                prefix="pool_", suffix=".tmp", dir=self.temp_dir
            )
            os.close(fd)
            os.unlink(path)
            self.available.append(Path(path))

    def acquire(self):
        with self.lock:
            if not self.available:
                fd, path = tempfile.mkstemp(
                    prefix="pool_", suffix=".tmp", dir=self.temp_dir
                )
                os.close(fd)
                path = Path(path)
            else:
                path = self.available.pop()
            self.in_use.add(path)
            return path

    def release(self, path):
        with self.lock:
            if path in self.in_use:
                self.in_use.remove(path)
                self.available.append(path)

    @contextmanager
    def get_temp_file(self):
        path = self.acquire()
        try:
            yield path
        finally:
            self.release(path)

    def cleanup(self):
        shutil.rmtree(self.temp_dir)

    def __enter__(self):
        return self

    def __exit__(self, *args):
        self.cleanup()

# Atomic file writer using tempfile
class AtomicWriter:
    def __init__(self, filepath, mode="w", encoding="utf-8"):
        self.filepath = filepath
        self.mode = mode
        self.encoding = encoding

    def __enter__(self):
        dir_name = os.path.dirname(self.filepath) or "."
        prefix = os.path.basename(self.filepath) + ".tmp."
        fd, self.temp_path = tempfile.mkstemp(
            suffix=".tmp", prefix=prefix, dir=dir_name
        )
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

# Secure temp file with encryption
class SecureTempFile:
    def __init__(self, suffix=None, prefix=None, dir=None):
        fd, self.path = tempfile.mkstemp(suffix=suffix, prefix=prefix, dir=dir)
        os.close(fd)

    def write(self, data):
        if isinstance(data, str):
            data = data.encode()
        # In real code, encrypt here
        encrypted = data  # Placeholder
        with open(self.path, "wb") as f:
            f.write(encrypted)

    def read(self):
        with open(self.path, "rb") as f:
            encrypted = f.read()
        return encrypted  # In real code, decrypt

    def cleanup(self):
        if os.path.exists(self.path):
            size = os.path.getsize(self.path)
            with open(self.path, "wb") as f:
                f.write(os.urandom(size))  # Overwrite with random data
            os.unlink(self.path)

    def __enter__(self):
        return self

    def __exit__(self, *args):
        self.cleanup()
```

## mkstemp

### What It Is

`mkstemp()` is a lower-level function that creates a temporary file and returns a file descriptor (integer) and the file path. Unlike `TemporaryFile`, it does NOT automatically delete the file—you must manage cleanup yourself. This gives you more control over the lifecycle.

### Why It Is Important

`mkstemp()` is useful when you need the temporary file path for use with other programs, when you need the raw file descriptor for low-level I/O, or when you need to keep the file open for an extended period. It's also the underlying mechanism used by `TemporaryFile` and `NamedTemporaryFile`.

### How It Works Internally

`mkstemp()` generates a unique filename using a combination of the prefix, random characters, and suffix. It creates the file with restricted permissions (0600 on Unix, only owner can read/write) and returns the OS-level file descriptor. The file is guaranteed to be newly created (not overwriting an existing file).

### Syntax

```python
import tempfile
import os

# Create temp file, returns (fd, path)
fd, path = tempfile.mkstemp(suffix=None, prefix=None, dir=None, text=False)

# Use the file descriptor
with os.fdopen(fd, "w") as f:
    f.write("data")

# Must clean up manually
os.unlink(path)

# Create temp directory
dir_path = tempfile.mkdtemp(suffix=None, prefix=None, dir=None)

# Must clean up manually
import shutil
shutil.rmtree(dir_path)
```

### Beginner Examples

```python
import tempfile
import os

# Basic mkstemp usage
fd, path = tempfile.mkstemp()
print(f"Temp file: {path}")
print(f"FD: {fd}")

# Write using the fd
with os.fdopen(fd, "w") as f:
    f.write("Hello from mkstemp")

# Read the file
with open(path, "r") as f:
    print(f.read())

# Must clean up!
os.unlink(path)

# mkdtemp
dir_path = tempfile.mkdtemp()
print(f"Temp dir: {dir_path}")

# Create files in it
with open(os.path.join(dir_path, "test.txt"), "w") as f:
    f.write("test")

# Clean up
import shutil
shutil.rmtree(dir_path)
```

### Intermediate Examples

```python
import tempfile
import os

# mkstemp with custom prefix/suffix
fd, path = tempfile.mkstemp(prefix="myapp_", suffix=".log")
os.close(fd)
print(f"Created: {path}")
os.unlink(path)

# mkstemp in specific directory
fd, path = tempfile.mkstemp(dir="/tmp")
os.close(fd)
os.unlink(path)

# mkstemp with text mode
fd, path = tempfile.mkstemp(text=True)
with os.fdopen(fd, "w") as f:
    f.write("Text mode temp file")
os.unlink(path)

# Safe temp file creation pattern
def create_temp_copy(source_path):
    """Create a temporary copy of a file."""
    fd, tmp_path = tempfile.mkstemp()
    try:
        with open(source_path, "rb") as src, os.fdopen(fd, "wb") as dst:
            dst.write(src.read())
        return tmp_path
    except:
        os.unlink(tmp_path)
        raise
```

### Real-World Use Cases

```python
import tempfile
import os
import subprocess

# External tool integration
def process_with_external_tool(data, tool_path):
    """Pass data to an external command via temp file."""
    fd, input_path = tempfile.mkstemp()
    fd2, output_path = tempfile.mkstemp()
    os.close(fd)
    os.close(fd2)

    try:
        with open(input_path, "w") as f:
            f.write(data)

        subprocess.run(
            [tool_path, input_path, output_path],
            check=True, capture_output=True
        )

        with open(output_path, "r") as f:
            result = f.read()
        return result
    finally:
        os.unlink(input_path)
        os.unlink(output_path)

# Safe config update with atomic write
def safe_update_config(config_path, updates):
    """Update a config file atomically."""
    import json

    if os.path.exists(config_path):
        with open(config_path, "r") as f:
            config = json.load(f)
    else:
        config = {}

    config.update(updates)

    fd, tmp_path = tempfile.mkstemp(
        dir=os.path.dirname(config_path) or ".",
        prefix=os.path.basename(config_path) + "."
    )
    try:
        with os.fdopen(fd, "w") as f:
            json.dump(config, f, indent=2)
        os.replace(tmp_path, config_path)
    except:
        os.unlink(tmp_path)
        raise

# Temporary credential file for CLI tools
def run_with_credentials(command, credentials):
    """Run a command using temporary credential file."""
    fd, cred_path = tempfile.mkstemp()
    try:
        with os.fdopen(fd, "w") as f:
            f.write(f"username={credentials['user']}\n")
            f.write(f"password={credentials['pass']}\n")

        env = os.environ.copy()
        env["CREDENTIAL_FILE"] = cred_path
        subprocess.run(command, env=env, check=True)
    finally:
        os.unlink(cred_path)
```

### Common Mistakes

```python
# Mistake 1: Not cleaning up mkstemp files
fd, path = tempfile.mkstemp()
# File is never deleted! Always clean up with os.unlink(path)

# Mistake 2: Forgetting to close the fd before unlink
fd, path = tempfile.mkstemp()
os.unlink(path)  # fd is still open!
os.close(fd)  # Correct order: close fd, then unlink

# Mistake 3: Not using context managers
f = tempfile.TemporaryFile()
f.write(b"data")
# If an exception occurs here, f is never closed!
# Always use 'with' for automatic cleanup

# Mistake 4: Assuming temp directory location
path = "/tmp/mytemp.txt"  # Wrong assumption!
# Use tempfile.gettempdir() or let tempfile choose

# Mistake 5: Not handling PermissionError
# Temp directories may have restrictive permissions

# Mistake 6: Creating temp files in the current directory
with tempfile.TemporaryFile(dir=".") as f:  # Pollutes CWD
    pass
# Better to use default:
with tempfile.TemporaryFile() as f:
    pass
```

### Best Practices

- Always use context managers (`with` statement) for automatic cleanup
- Use `TemporaryFile` for anonymous temp files (most secure)
- Use `TemporaryDirectory` when creating multiple temp files
- Use `NamedTemporaryFile(delete=False)` only when the path is needed externally
- Use `mkstemp()` only when you need full control over the lifecycle
- Always clean up `mkstemp`/`mkdtemp` in `finally` blocks
- Specify meaningful `prefix` and `suffix` for debugging
- Use `tempfile.gettempdir()` to check temp directory location
- Be aware of disk space—temp files consume real disk space
- On multi-user systems, temp files with default permissions are secure

### Performance Considerations

- Creating a temp file involves system calls (relatively cheap)
- `TemporaryFile` on Unix is slightly more efficient (no directory entry to clean)
- `TemporaryDirectory` cleanup via `shutil.rmtree` can be slow for many files
- Consider in-memory alternatives (`io.StringIO`, `io.BytesIO`) for small data
- Temp file I/O performance depends on the underlying filesystem (SSD vs HDD, tmpfs vs disk)
- On Linux, `/dev/shm` or `tmpfs` mounts provide RAM-backed temp storage

### Interview Questions

1. Difference between `TemporaryFile` and `NamedTemporaryFile`?

   `TemporaryFile` creates an anonymous file with no visible filesystem name. `NamedTemporaryFile` has a visible name that can be passed to other programs. On Unix, `TemporaryFile` immediately unlinks the file, so only the file descriptor remains.

2. How does `TemporaryDirectory` work?

   It creates a temp directory via `mkdtemp()` and returns its path. As a context manager, it deletes the directory and all contents on exit via `shutil.rmtree()`.

3. What is `tempfile.gettempdir()`?

   It returns the system's temporary directory path (e.g., `/tmp`, `C:\Temp`, `/var/tmp`). This is configurable via environment variables (`TMPDIR`, `TEMP`, `TMP`).

4. When would you use `mkstemp()` instead of `TemporaryFile()`?

   When you need the raw file descriptor for low-level I/O, when you need to pass the file path to another program/process, or when you need control over when the file is deleted.

5. Are temporary files secure?

   Yes when using `tempfile` properly. `mkstemp()` creates files with permissions 0600. `TemporaryFile` immediately unlinks on Unix. However, `NamedTemporaryFile` leaves the name visible briefly, creating a race condition window.

### Coding Challenges

1. Build a temp file cleanup utility that finds and removes orphaned temp files older than a configurable age.

2. Implement a context manager that tracks all temp files created within its scope and ensures cleanup.

3. Create a disk-backed cache using temp files with LRU eviction.

4. Build a temp directory manager with disk usage quotas.

5. Implement an encrypted temp file context manager that securely wipes data on cleanup.

### Related Topics

- `48_file_operations.md` - File I/O operations
- `49_json.md` - JSON processing with temp files
- `50_csv.md` - CSV processing with temp files
- `51_pickle.md` - Pickle serialization with temp storage
- `52_pathlib.md` - Path operations on temp files
- `os` module - Low-level file descriptor operations
- `shutil` module - File operations for cleanup
- `contextlib` - Custom context managers
