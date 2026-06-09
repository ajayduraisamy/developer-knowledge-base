# Pathlib - Path(), PurePath, path operations, glob (Python 3.4+)

## Introduction

`pathlib` is a Python module introduced in Python 3.4 that provides an object-oriented approach to filesystem paths. Instead of representing paths as strings, `pathlib` uses `Path` objects with methods and properties that make path manipulation intuitive and less error-prone. It addresses the pain points of string-based path handling: cross-platform separator differences, complex path joining, and error-prone string manipulation.

## Path() class

### What It Is

`Path()` is the main class in pathlib. It represents a filesystem path and provides methods for both path manipulation and filesystem I/O. `Path` objects can be created from strings, joined with the `/` operator, and provide properties for accessing path components.

### Why It Is Important

`Path()` replaces most `os.path` functions with a cleaner, object-oriented API. It handles cross-platform path separators automatically, provides method chaining, and combines path manipulation with direct filesystem access methods (read, write, mkdir, etc.). It's the recommended way to handle paths in Python 3.6+.

### How It Works Internally

`Path()` creates either a `PosixPath` or `WindowsPath` instance depending on the operating system. These classes inherit from `PurePosixPath`/`PureWindowsPath` (for string operations) and mix in filesystem methods. The `/` operator is overloaded to perform path joining using the correct separator for the platform.

### Syntax

```python
from pathlib import Path

# Creating paths
p = Path("/usr/local/bin")         # POSIX absolute path
p = Path("C:\\Users\\Alice")       # Windows absolute path
p = Path("relative/path")          # Relative path
p = Path()                          # Current directory (.)
p = Path.home()                     # User home directory
p = Path.cwd()                      # Current working directory

# Path properties
p.parts          # ('usr', 'local', 'bin') - tuple of components
p.parent         # /usr/local - parent directory
p.parents        # [PurePosixPath('/usr/local'), PurePosixPath('/usr'), PurePosixPath('/')]
p.name           # 'bin' - final component
p.stem           # 'file' - filename without extension
p.suffix         # '.txt' - file extension
p.suffixes       # ['.tar', '.gz'] - all extensions
p.root           # '/' - root of path
p.anchor         # '/' or 'C:\\' - drive + root

# Path joining
p = Path("/usr") / "local" / "bin"

# Existence checks
p.exists()
p.is_file()
p.is_dir()
p.is_absolute()
p.is_symlink()
```

### Beginner Examples

```python
from pathlib import Path

# Creating paths
home = Path.home()
cwd = Path.cwd()
print(f"Home: {home}")
print(f"CWD: {cwd}")

# Path components
p = Path("/usr/local/bin/python3")
print(f"Parts: {p.parts}")
print(f"Parent: {p.parent}")
print(f"Name: {p.name}")
print(f"Stem: {p.stem}")
print(f"Suffix: {p.suffix}")

# Path joining with /
config = Path("/etc") / "app" / "config.ini"
print(f"Config: {config}")

# Check existence
p = Path("test.txt")
if p.exists():
    print(f"{p} exists")
    print(f"Is file: {p.is_file()}")
    print(f"Is dir: {p.is_dir()}")

# Simple I/O
file_path = Path("hello.txt")
file_path.write_text("Hello, pathlib!")
content = file_path.read_text()
print(content)
file_path.unlink()  # Delete file

# Directory creation
new_dir = Path("new_directory")
new_dir.mkdir(exist_ok=True)
new_dir.rmdir()  # Remove empty directory
```

## PurePath

### What It Is

`PurePath` is the base class for path objects that provide purely computational path operations without filesystem access. It has two subclasses: `PurePosixPath` (for POSIX-style paths) and `PureWindowsPath` (for Windows-style paths). `Path` inherits from `PurePath` and adds filesystem I/O.

### Why It Is Important

`PurePath` is useful when you only need path manipulation (splitting, joining, resolving) without touching the filesystem. It's also platform-independent: on Windows you can use `PurePosixPath` to manipulate POSIX-style paths (e.g., for remote server paths).

### How It Works Internally

`PurePath` stores the path string internally and provides properties/methods that compute results from the string without any filesystem calls. Methods like `.parent`, `.name`, `.stem` are string operations. `.resolve()` (on `Path`, not `PurePath`) requires filesystem access to resolve symlinks and normalize.

### Syntax

```python
from pathlib import PurePath, PurePosixPath, PureWindowsPath

# Platform-specific
pure_posix = PurePosixPath("/usr/local/bin")
pure_win = PureWindowsPath("C:\\Users\\Alice")

# Cross-platform use on any OS
posix = PurePosixPath("linux/path/file.txt")
print(posix.parent)  # PurePosixPath('linux/path')

win = PureWindowsPath("windows\\path\\file.txt")
print(win.parent)    # PureWindowsPath('windows\\path')

# Common operations
p = PurePath("/usr/local/bin")
print(p.parts)          # ('/', 'usr', 'local', 'bin')
print(p.drive)          # ''
print(p.root)           # '/'
print(p.anchor)         # '/'

# Comparison (order-independent of case on Windows)
p1 = PurePosixPath("/usr/bin")
p2 = PurePosixPath("/usr/bin")
print(p1 == p2)  # True

# No filesystem methods
p = PurePath("/etc/hosts")
# p.exists()  # AttributeError: 'PurePath' object has no attribute 'exists'
```

## Path operations

### What It Is

Path operations encompass all the methods available on `Path` objects for manipulating and querying paths, as well as performing filesystem I/O. These include reading/writing files, creating/deleting directories, resolving paths, and querying file metadata.

### Why It Is Important

`Path` provides a unified API for common filesystem tasks that previously required multiple modules (`os.path`, `os`, `glob`, `shutil`). This reduces the number of imports needed and makes code more readable.

### How It Works Internally

Filesystem methods on `Path` delegate to the corresponding `os` module functions internally. For example, `path.exists()` calls `os.path.exists(path)`, `path.mkdir()` calls `os.mkdir()` or `os.makedirs()`. The `Path` object simply provides a cleaner interface.

### Syntax

```python
from pathlib import Path

p = Path("some/path")

# Querying
p.exists()                      # Whether path exists
p.is_file()                     # Whether it's a file
p.is_dir()                      # Whether it's a directory
p.is_absolute()                 # Whether path is absolute
p.is_symlink()                  # Whether it's a symbolic link
p.stat()                        # File stat info
p.lstat()                       # Stat without following symlinks
p.owner()                       # File owner name
p.group()                       # File group name

# Reading/Writing
p.read_text(encoding="utf-8")     # Read text content
p.read_bytes()                     # Read binary content
p.write_text("content")           # Write text
p.write_bytes(b"content")         # Write binary

# Directory operations
p.mkdir(mode=0o777, parents=False, exist_ok=False)  # Create directory
p.rmdir()                        # Remove empty directory
p.rename(target)                 # Rename/move
p.replace(target)                # Rename/move, overwrite target
p.iterdir()                      # Iterate over directory contents

# Path manipulation
p.resolve()                      # Resolve symlinks, make absolute
p.absolute()                     # Make absolute (doesn't resolve symlinks)
p.relative_to(other)             # Get relative path to other
p.with_name(name)                # Replace final component
p.with_suffix(suffix)            # Replace extension
p.with_stem(stem)                # Replace stem without extension (3.9+)

# Utilities
p.samefile(other)                # Whether paths point to same file
p.expanduser()                   # Expand ~ to home directory
p.home()                         # Home directory
p.cwd()                          # Current working directory
```

### Beginner Examples

```python
from pathlib import Path

# Creating directories
Path("output/data").mkdir(parents=True, exist_ok=True)

# Writing and reading files
p = Path("data.txt")
p.write_text("Hello, World!")
print(p.read_text())

# Iterating directory
for item in Path(".").iterdir():
    if item.is_file():
        print(f"File: {item.name}")
    elif item.is_dir():
        print(f"Dir: {item.name}")

# Getting file info
p = Path("data.txt")
if p.exists():
    stat = p.stat()
    print(f"Size: {stat.st_size} bytes")
    print(f"Modified: {stat.st_mtime}")

# Renaming files
p = Path("old_name.txt")
if p.exists():
    p.rename("new_name.txt")

# Resolving paths
print(Path(".").resolve())  # Full absolute path

# Changing extension
p = Path("data.txt")
print(p.with_suffix(".csv"))  # data.csv

# Relative paths
p1 = Path("/usr/local/bin")
p2 = Path("/usr/local/lib/file.txt")
print(p2.relative_to(p1))  # ../lib/file.txt
```

### Intermediate Examples

```python
from pathlib import Path
import shutil

# File copy using Path
src = Path("source.txt")
dst = Path("backup/source.txt")
dst.parent.mkdir(parents=True, exist_ok=True)
shutil.copy2(src, dst)

# Finding files by properties
def find_large_files(directory, min_size_mb=100):
    path = Path(directory)
    for file in path.rglob("*"):
        if file.is_file() and file.stat().st_size > min_size_mb * 1024 * 1024:
            yield file

# Batch rename
def rename_files(directory, pattern, replacement):
    path = Path(directory)
    for item in path.iterdir():
        if item.is_file():
            new_name = item.name.replace(pattern, replacement)
            if new_name != item.name:
                item.rename(item.parent / new_name)

# Safe file deletion with confirmation
def safe_delete(path, confirm=True):
    p = Path(path)
    if not p.exists():
        return
    if p.is_dir():
        if confirm:
            response = input(f"Delete directory '{p}' and all contents? (y/N): ")
            if response.lower() != "y":
                return
        shutil.rmtree(p)
    else:
        p.unlink()

# Directory tree walking
def tree(directory, prefix=""):
    path = Path(directory)
    contents = list(path.iterdir())
    for i, item in enumerate(contents):
        is_last = i == len(contents) - 1
        print(f"{prefix}{'└── ' if is_last else '├── '}{item.name}")
        if item.is_dir():
            extension = "    " if is_last else "│   "
            tree(item, prefix + extension)
```

## glob (Python 3.4+)

### What It Is

`Path.glob()` and `Path.rglob()` provide pattern-based file matching within a directory tree. `glob()` matches files in the current directory, while `rglob()` recursively searches all subdirectories. They support the standard glob patterns (`*`, `?`, `[abc]`, `**`).

### Why It Is Important

Globbing is essential for finding files by pattern without manually traversing directory trees. `pathlib`'s glob methods return `Path` objects (not strings), integrate naturally with the Path API, and are generally faster than the standalone `glob` module for simple patterns.

### How It Works Internally

`glob()` and `rglob()` use Python's `pathlib._NormalAccessor.scandir()` for efficient directory iteration. They compile the glob pattern and match filenames against it. `rglob()` is equivalent to `glob("**/pattern")` and recursively enters subdirectories. The methods are generators that yield results lazily as they're found.

### Syntax

```python
from pathlib import Path

p = Path(".")

# Basic glob
for py_file in p.glob("*.py"):
    print(py_file)

# Recursive glob
for all_py in p.rglob("*.py"):
    print(all_py)

# Equivalent forms
p.rglob("*.py")         # Recursive
p.glob("**/*.py")       # Same thing (Python 3.5+)

# Single character wildcard
p.glob("file_?.txt")    # file_1.txt, file_a.txt, etc.

# Character set
p.glob("[abc]*.py")     # Starts with a, b, or c

# Multiple patterns
for ext in ["*.py", "*.md", "*.txt"]:
    for file in p.glob(ext):
        print(file)

# Check if any files match
has_python = any(p.glob("*.py"))
```

### Beginner Examples

```python
from pathlib import Path

# Find all Python files
py_files = list(Path("src").glob("*.py"))
print(f"Found {len(py_files)} Python files")

# Find all text files recursively
txt_files = Path(".").rglob("*.txt")
for file in txt_files:
    print(file)

# Count files by extension
from collections import Counter
counter = Counter()
for file in Path(".").rglob("*"):
    if file.is_file():
        counter[file.suffix] += 1
print(counter)

# Find directories
dirs = [p for p in Path(".").glob("*") if p.is_dir()]
print(f"Subdirectories: {len(dirs)}")

# Find files matching multiple extensions
extensions = {".py", ".js", ".ts"}
matching = [
    f for f in Path(".").rglob("*")
    if f.suffix in extensions
]

# Check for specific file
if Path(".").glob("config.yaml"):
    print("Config found")

# Find files by size pattern
large_files = [
    f for f in Path(".").rglob("*")
    if f.is_file() and f.stat().st_size > 1024 * 1024
]
```

### Intermediate Examples

```python
from pathlib import Path

# Recursive file search with pattern exclusion
def find_files(directory, pattern, exclude_dirs=None):
    exclude_dirs = exclude_dirs or {".git", "__pycache__", "venv"}
    path = Path(directory)
    for item in path.rglob(pattern):
        if not any(part in exclude_dirs for part in item.parts):
            yield item

# Group files by directory
def group_by_directory(root, pattern):
    groups = {}
    for file in Path(root).rglob(pattern):
        parent = file.parent
        if parent not in groups:
            groups[parent] = []
        groups[parent].append(file)
    return groups

# Find most recently modified file
def latest_file(directory, pattern="*"):
    files = list(Path(directory).rglob(pattern))
    if not files:
        return None
    return max(files, key=lambda f: f.stat().st_mtime)

# Glob with filtering
config_files = [
    f for f in Path(".").rglob("*.json")
    if "test" not in f.name.lower()
]

# Check for duplicate filenames
from collections import Counter
filenames = [f.name for f in Path(".").rglob("*") if f.is_file()]
duplicates = {name: count for name, count in Counter(filenames).items() if count > 1}
```

### Advanced Examples

```python
from pathlib import Path
import hashlib
import json
from datetime import datetime

# File integrity checker
def scan_files(directory):
    state = {}
    for file in Path(directory).rglob("*"):
        if file.is_file() and file.name != ".integrity.json":
            relative = str(file.relative_to(directory))
            state[relative] = {
                "hash": hashlib.sha256(file.read_bytes()).hexdigest(),
                "size": file.stat().st_size,
                "modified": datetime.fromtimestamp(file.stat().st_mtime).isoformat(),
            }
    Path(directory, ".integrity.json").write_text(json.dumps(state, indent=2))
    return state

# Project scaffolder
def scaffold_project(name):
    base = Path.cwd() / name
    base.mkdir(parents=True, exist_ok=True)
    (base / "src" / name).mkdir(parents=True, exist_ok=True)
    (base / "tests").mkdir()
    (base / "README.md").write_text(f"# {name}\n")
    (base / "src" / name / "__init__.py").write_text("")
    (base / "requirements.txt").write_text("# dependencies\n")
    return base

# File watcher
class FileWatcher:
    def __init__(self, directory):
        self.directory = Path(directory)
        self.snapshot = self._take_snapshot()

    def _take_snapshot(self):
        snapshot = {}
        for f in self.directory.rglob("*"):
            if f.is_file():
                stat = f.stat()
                snapshot[f] = (stat.st_size, stat.st_mtime)
        return snapshot

    def check_changes(self):
        current = self._take_snapshot()
        changes = {"added": [], "removed": [], "modified": []}
        for path, state in current.items():
            if path not in self.snapshot:
                changes["added"].append(path)
            elif state != self.snapshot[path]:
                changes["modified"].append(path)
        for path in self.snapshot:
            if path not in current:
                changes["removed"].append(path)
        self.snapshot = current
        return changes

# Config locator
def find_config(app_name, filename="config.ini"):
    candidates = [
        Path.home() / ".config" / app_name / filename,
        Path.home() / f".{app_name}" / filename,
        Path("/etc") / app_name / filename,
        Path.cwd() / "config" / filename,
    ]
    for path in candidates:
        if path.exists():
            return path
    return None
```

### Real-World Use Cases

```python
from pathlib import Path

# Build system: find all source files
source_files = list(Path("src").rglob("*.py"))
test_files = list(Path("tests").rglob("*.py"))

# Data pipeline: organize output structure
output_dir = Path("output") / datetime.now().strftime("%Y-%m-%d")
output_dir.mkdir(parents=True, exist_ok=True)
(output_dir / "processed").mkdir()
(output_dir / "errors").mkdir()

# Log rotation: archive old logs
log_dir = Path("logs")
for log_file in log_dir.glob("*.log"):
    if log_file.stat().st_mtime < time.time() - 86400 * 30:  # 30 days
        archive_name = log_dir / "archive" / f"{log_file.stem}_{datetime.now():%Y%m%d}{log_file.suffix}"
        archive_name.parent.mkdir(exist_ok=True)
        log_file.rename(archive_name)

# Backup: find new/modified files
backup_dir = Path("backup")
source_dir = Path("data")
for file in source_dir.rglob("*"):
    if file.is_file():
        rel = file.relative_to(source_dir)
        dest = backup_dir / rel
        if not dest.exists() or file.stat().st_mtime > dest.stat().st_mtime:
            shutil.copy2(file, dest)
```

### Common Mistakes

```python
# Mistake 1: Not using parents=True with mkdir()
p = Path("a/b/c")
p.mkdir()  # FileNotFoundError if a/b doesn't exist!
p.mkdir(parents=True, exist_ok=True)  # Correct

# Mistake 2: Converting to string unnecessarily
path = Path("data.txt")
# Most libraries (numpy, pandas, etc.) accept Path objects
# Avoid str(path) unless the API requires it

# Mistake 3: Hardcoding separators
# Wrong:
path = Path("usr/local/bin")
# This creates a relative path "usr/local/bin" on all platforms
# If you mean absolute:
path = Path("/usr/local/bin")

# Mistake 4: is_file()/is_dir() on non-existent paths
p = Path("nonexistent.txt")
print(p.is_file())  # Returns False (not an error)
print(p.is_dir())   # Returns False

# Always check exists() first:
if p.exists():
    if p.is_file():
        process_file(p)

# Mistake 5: Modifying paths as strings
path = Path("data.txt")
# Wrong:
new_path = Path(str(path).replace(".txt", ".csv"))
# Correct:
new_path = path.with_suffix(".csv")

# Mistake 6: Using rglob without limiting depth
files = list(Path(".").rglob("*"))  # Could be millions of files
```

### Best Practices

- Use `Path` over `PurePath` unless you only need string operations
- Use `/` operator for path joining (more readable than `os.path.join`)
- Use `rglob()` for recursive searches (`**/*.py` is less clear)
- Use `parents=True, exist_ok=True` with `mkdir()` to avoid errors
- Let libraries accept `Path` objects directly (no need for `str()`)
- Use `with_name()`, `with_suffix()`, `with_stem()` instead of string manipulation
- Use `open()` as a method on Path: `path.open()` (same as built-in `open()`)
- Prefer Path methods over `os.path` functions
- Use `path.resolve()` for absolute canonical paths
- Use `path.relative_to()` for computing relative paths between two directories

### Performance Considerations

- `Path` objects have minimal overhead—they store a single string
- `iterdir()` is efficient for listing directory contents
- `rglob()` is optimized but can be slow on deep directory trees
- `Path.stat()` makes a system call—cache results if used repeatedly
- `read_bytes()`/`read_text()` load entire file into memory
- For large directory trees, consider using `os.scandir()` directly
- `glob()`/`rglob()` are lazily evaluated generators

### Interview Questions

1. What is the difference between `PurePath` and `Path`?

   `PurePath` provides path manipulation without filesystem access. `Path` inherits from `PurePath` and adds filesystem operations like `exists()`, `mkdir()`, `read_text()`.

2. How do you join paths in pathlib?

   Use the `/` operator: `Path("/usr") / "local" / "bin"` produces `Path("/usr/local/bin")`.

3. What does `Path.rglob("*.py")` do?

   It recursively searches for all Python files matching `*.py` starting from the path's directory and all subdirectories.

4. How do you get the filename without extension?

   Use `.stem` property. For `Path("file.txt")`, `.stem` returns `"file"` and `.suffix` returns `".txt"`.

5. What is the difference between `path.resolve()` and `path.absolute()`?

   `resolve()` makes the path absolute AND resolves symlinks/normalizes. `absolute()` just prepends the current working directory without resolving symlinks.

### Coding Challenges

1. Duplicate file finder: recursively find duplicate files by content hash.

2. Directory size calculator: calculate total size of all files in a tree with per-directory breakdowns.

3. File history tracker: track file modifications over time with state snapshots.

4. Template-based project generator: scaffold projects from template files.

5. Log archiver: archive/compress log files older than N days.

### Related Topics

- `48_file_operations.md` - File I/O operations
- `49_json.md` - JSON file reading with Path
- `50_csv.md` - CSV file path management
- `51_pickle.md` - Pickle file handling
- `53_temp_files.md` - Temporary file management
- `os.path` module - Legacy string-based path API
- `glob` module - Legacy file pattern matching
- `shutil` module - High-level file operations
