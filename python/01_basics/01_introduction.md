# What is Python? - The python interpreter, REPL, and running scripts

## Introduction

Python is a high-level, interpreted, general-purpose programming language created by Guido van Rossum and first released in 1991. It emphasizes code readability through significant indentation and a clean syntax. Python supports multiple programming paradigms including procedural, object-oriented, and functional programming. It is dynamically typed, garbage-collected, and has a large standard library. The language is used across web development, data science, AI/ML, automation, scientific computing, and more.

## What is Python?

### What It Is
Python is an interpreted, bytecode-compiled, dynamically-typed programming language. Code is compiled to platform-independent bytecode which runs on the Python Virtual Machine (PVM). Python is often described as a "batteries included" language due to its comprehensive standard library.

### Why It Is Important
Python's design philosophy (The Zen of Python) prioritizes readability and simplicity. It consistently ranks among the top programming languages in the TIOBE and Stack Overflow surveys. Major organizations including Google, Netflix, NASA, and Spotify rely on Python for critical systems.

### How It Works Internally
Python source code (.py files) is compiled to bytecode (.pyc files) by the compiler. This bytecode is executed by the Python Virtual Machine (PVM). The CPython interpreter (the reference implementation) uses a stack-based VM. The Global Interpreter Lock (GIL) allows only one thread to execute bytecode at a time, simplifying memory management but limiting CPU-bound parallelism.

### Syntax
```python
# Source code (.py) -> Bytecode (.pyc) -> Execution on PVM

# Hello World
print("Hello, World!")

# Python emphasizes readability with significant whitespace
def greet(name: str) -> str:
    return f"Hello, {name}!"

# Dynamic typing
x = 10        # int
x = "hello"   # str — type changes at runtime
```

### Beginner Examples
```python
# First program
print("Welcome to Python!")

# Simple arithmetic
result = (3 + 5) * 2
print(f"Result: {result}")

# User input
name = input("What is your name? ")
print(f"Hello, {name}!")

# Basic conditional
age = int(input("Enter age: "))
if age >= 18:
    print("Adult")
else:
    print("Minor")
```

### Intermediate Examples
```python
# Working with the standard library
import json
import os
from datetime import datetime

# JSON processing
data = {"name": "Alice", "scores": [85, 92, 78]}
json_str = json.dumps(data, indent=2)
print(json_str)

# File operations
with open("example.txt", "w") as f:
    f.write("Hello, file!\n")

# Environment variables
print(f"Home: {os.environ.get('HOME', 'Not set')}")

# Date/time
now = datetime.now()
print(f"Current time: {now.isoformat()}")
```

### Advanced Examples
```python
# Metaclass example
class SingletonMeta(type):
    _instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Database(metaclass=SingletonMeta):
    def __init__(self):
        self.connected = False

db1 = Database()
db2 = Database()
print(f"Same instance: {db1 is db2}")  # True

# Introspection
import inspect

def example_func(a: int, b: str = "default") -> bool:
    return True

sig = inspect.signature(example_func)
for name, param in sig.parameters.items():
    print(f"{name}: {param.annotation} = {param.default}")

# Customizing import behavior with __import__
import sys

class ImportLogger:
    def find_spec(self, fullname, path, target=None):
        print(f"Importing: {fullname}")
        return None  # Fall through to default

sys.meta_path.insert(0, ImportLogger())
import math  # Will log "Importing: math"
```

### Real-World Use Cases
- **Web Development**: Django, FastAPI, Flask for building APIs and websites
- **Data Science**: Pandas, NumPy for analysis and manipulation
- **Machine Learning**: TensorFlow, PyTorch, scikit-learn
- **DevOps**: Ansible, SaltStack automation scripts
- **Scripting**: Automating file processing, data migration, system administration

### Common Mistakes
```python
# Mistake 1: Indentation inconsistency
def foo():
    print("hello")
    print("world")  # Tab vs spaces — IndentationError

# Mistake 2: Mutable default arguments
def append_to(element, target=[]):  # Shared across calls!
    target.append(element)
    return target

print(append_to(1))  # [1]
print(append_to(2))  # [1, 2] — not [2]!

# Correct:
def append_to(element, target=None):
    if target is None:
        target = []
    target.append(element)
    return target

# Mistake 3: Late binding in closures
funcs = [lambda: i for i in range(5)]
print([f() for f in funcs])  # [4, 4, 4, 4, 4], not [0, 1, 2, 3, 4]

# Correct:
funcs = [lambda i=i: i for i in range(5)]
print([f() for f in funcs])  # [0, 1, 2, 3, 4]
```

### Best Practices
- Follow PEP 8 style guide
- Use type hints for public APIs (Python 3.5+)
- Write docstrings for modules, classes, and functions
- Use virtual environments for dependency isolation
- Keep functions small and focused (Single Responsibility Principle)
- Prefer `pathlib` over `os.path` for path manipulation (Python 3.6+)

### Performance Considerations
- Python 3.11+ includes significant performance improvements (up to 60% faster than 3.10)
- The GIL limits CPU-bound multi-threading; use `multiprocessing` for parallel CPU work
- Bytecode compilation adds ~1-2 seconds startup overhead
- JIT compilation via PyPy can be 4-10x faster for long-running pure-Python programs
- Use `__slots__` in classes to reduce memory overhead

### Interview Questions
1. How does Python execute code (source to execution)?
2. What is the difference between CPython, PyPy, and Jython?
3. Explain the Global Interpreter Lock (GIL) and its impact.
4. What is bytecode in Python and where is it stored?
5. How does Python's memory management work (reference counting + garbage collection)?
6. What are Python's key design philosophies?
7. Why is Python called a "batteries included" language?
8. Explain the difference between an interpreted and compiled language. How does Python fit?
9. What is the difference between `__pycache__` and `.pyc` files?
10. How does Python's import system work?

### Coding Challenges
```python
# Challenge 1: Bytecode Inspector
import dis

def factorial(n):
    if n <= 1:
        return 1
    return n * factorial(n - 1)

dis.dis(factorial)

# Challenge 2: Python Version Info
import sys
import platform

def system_info():
    info = {
        "implementation": sys.implementation.name,
        "version": sys.version,
        "platform": platform.platform(),
        "architecture": platform.machine(),
        "byteorder": sys.byteorder,
        "max_int": sys.maxsize,
        "path_separator": os.sep,
    }
    return info

import os
for k, v in system_info().items():
    print(f"{k}: {v}")
```

### Related Topics
- Python Installation and Setup
- Variables and Data Types
- Control Flow
- Functions and Modules
- OOP in Python
- The Python Standard Library

## Python Interpreter

### What It Is
The Python interpreter is the program that reads and executes Python code. The reference implementation is CPython, written in C. Alternative interpreters include PyPy (JIT-compiled), Jython (Java-based), and IronPython (.NET-based).

### Why It Is Important
The interpreter is the bridge between Python source code and machine execution. Understanding how the interpreter works helps diagnose performance issues, debug complex problems, and write more efficient code.

### How It Works Internally
CPython operates in four stages: (1) Lexical analysis — source code is tokenized, (2) Parsing — tokens are built into an Abstract Syntax Tree (AST), (3) Compilation — AST is compiled into bytecode, (4) Execution — bytecode runs on the evaluation loop in `ceval.c`. The interpreter uses a stack-based architecture with frames for scope management.

### Syntax
```bash
# Running the interpreter
python          # Opens interactive mode
python script.py  # Executes a script
python -m module  # Runs a module as __main__
python -c "code"  # Executes a code string

# Version-specific
python -m ensurepip --upgrade  # Ensure pip is installed
python -X dev script.py  # Development mode (Python 3.7+)
python -X utf8 script.py  # UTF-8 mode (Python 3.7+)
```

### Beginner Examples
```python
# Check which interpreter is running
import sys
print(f"Interpreter: {sys.implementation.name}")
print(f"Version: {sys.version_info.major}.{sys.version_info.minor}")
print(f"Executable: {sys.executable}")

# Command-line flags (run with -v to see imports)
# python -v script.py  # Verbose import logging
# python -O script.py   # Optimized mode (removes asserts)
```

### Intermediate Examples
```python
# Modifying interpreter behavior
import sys

# Add a directory to the module search path
sys.path.insert(0, "/custom/path")

# Set recursion limit
sys.setrecursionlimit(10000)

# Get interpreter flags
print(f"Optimized mode: {sys.flags.optimize}")
print(f"Debug mode: {sys.flags.debug}")
print(f"Verbose mode: {sys.flags.verbose}")

# Site-packages location
import site
print(f"Site packages: {site.getsitepackages()}")
```

### Advanced Examples
```python
# Customizing the interpreter with sitecustomize.py
# Place in site-packages/sitecustomize.py
import sys

def enable_dev_mode():
    if not sys.flags.dev_mode:
        print("Warning: Run with -X dev for development mode")

# Overriding builtins
import builtins

original_print = builtins.print
def debug_print(*args, **kwargs):
    import inspect
    frame = inspect.currentframe().f_back
    original_print(f"[{frame.f_lineno}]", *args, **kwargs)

builtins.print = debug_print

print("This shows line number")  # [5] This shows line number
```

### Real-World Use Cases
- **CI/CD**: Running tests with different interpreter versions
- **Containerization**: Multi-stage Docker builds with different Python versions
- **Performance Tuning**: Choosing between CPython and PyPy based on workload
- **Embedded Systems**: MicroPython for microcontrollers
- **Education**: Using interpreter flags for debugging student code

### Common Mistakes
```python
# Mistake 1: Assuming python == python3
# On Unix systems, python may point to Python 2
# Always use python3 or check: python --version

# Mistake 2: Mixing interpreter versions
# The shebang line determines which interpreter runs:
#!/usr/bin/env python3  # Correct
#!/usr/bin/python       # Ambiguous

# Mistake 3: Not refreshing after reinstall
# After installing a new Python version, terminal sessions
# may still cache the old path. Restart or use hash -r.
```

### Best Practices
- Always use `python3` (or `python3.x` explicitly) on Unix systems
- Use shebang `#!/usr/bin/env python3` for scripts (portable)
- Run with `-X dev` during development for extra warnings
- Use `-m` flag for running modules as scripts
- Check `sys.implementation.name` when writing cross-interpreter code
- Set `PYTHONUTF8=1` on Windows for UTF-8 mode (Python 3.7+)

### Performance Considerations
- CPython 3.12+ uses zero-cost exception handlers for `try` blocks without `except`
- Python 3.11 introduced specialized adaptive interpreter for faster bytecode
- PyPy can be 4-10x faster but uses more memory
- `python -O` removes assert statements but gives minimal speedup
- Cold start overhead: ~30-50ms for simple scripts

### Interview Questions
1. What is the difference between CPython and PyPy?
2. How does the Python interpreter handle memory management?
3. What happens when you type `python script.py`?
4. What is the purpose of `sys.path`?
5. How does the `-m` flag work in Python?
6. What are the different Python implementations and their use cases?
7. How does the interpreter handle circular imports?
8. What is the difference between `python -c` and `python -m`?
9. How does the Garbage Collector interact with the interpreter?
10. What is the purpose of `__pycache__` and how does it improve performance?

### Coding Challenges
```python
# Challenge: Cross-version compatibility checker
import sys

def check_compatibility(target_version=(3, 10)):
    current = sys.version_info[:2]
    if current >= target_version:
        print(f"✓ Compatible with Python {current}")
    else:
        raise RuntimeError(
            f"Need Python {'.'.join(map(str, target_version))}+, "
            f"have {'.'.join(map(str, current))}"
        )

check_compatibility((3, 10))

# Challenge: Interpreter info script
def interpreter_debug():
    import platform
    info = {
        "Implementation": sys.implementation.name,
        "Version": sys.version.split()[0],
        "Build": sys.version.split()[2] if len(sys.version.split()) > 2 else "unknown",
        "Compiler": platform.python_compiler(),
        "Build Date": platform.python_build()[1],
        "ABI Tags": sys.implementation.cache_tag,
    }
    for k, v in info.items():
        print(f"{k:20}: {v}")

interpreter_debug()
```

### Related Topics
- Bytecode and Disassembly
- Python Implementations (CPython, PyPy, Jython)
- The `sys` Module
- Module Import System
- Performance Optimization

## REPL

### What It Is
The Read-Eval-Print Loop (REPL) is an interactive shell that reads Python expressions, evaluates them, prints results, and loops for more input. It is started by running `python` without arguments.

### Why It Is Important
The REPL enables rapid experimentation, debugging, and learning. It allows testing small code snippets without creating files, exploring module APIs via tab completion, and inspecting objects interactively.

### How It Works Internally
The REPL uses the `code` module internally. Each line is read by the `sys.stdin` reader, compiled to bytecode via `compile()`, executed in the `__main__` namespace with `exec()`, and results (non-None expressions) are converted to strings with `repr()` and printed via `sys.stdout`. The `sys.ps1` and `sys.ps2` variables control the prompt strings.

### Syntax
```python
# The REPL shows >>> for primary prompt, ... for continuation
>>> 2 + 2
4
>>> def add(a, b):
...     return a + b
...
>>> add(3, 5)
8

# Special REPL commands
>>> help(print)     # Opens pager with documentation
>>> dir(list)       # Shows all list methods
>>> type(42)        # <class 'int'>
```

### Beginner Examples
```python
# REPL as a calculator
>>> 10 * 3.14
31.4
>>> 2 ** 10
1024

# Exploring objects
>>> s = "hello"
>>> s.upper()
'HELLO'
>>> s.__class__
<class 'str'>

# Checking what's available
>>> import math
>>> [x for x in dir(math) if not x.startswith('_')]
['acos', 'acosh', 'asin', ...]
```

### Intermediate Examples
```python
# Using _ for the last result
>>> 42 * 2
84
>>> _ * 2
168

# Multi-line editing with history
>>> for i in range(3):
...     print(f"Line {i}")
...
Line 0
Line 1
Line 2

# Import and explore modules
>>> import json
>>> help(json.dumps)

# Using the REPL as a debug tool
>>> import sys
>>> sys.path
['', '/usr/lib/python3.12', ...]
```

### Advanced Examples
```python
# Customizing the REPL
>>> import sys
>>> sys.ps1 = ">>> "  # Default
>>> sys.ps2 = "... "  # Default

# Creating a custom REPL
>>> import code
>>> variables = {'x': 42, 'y': 'hello'}
>>> console = code.InteractiveConsole(variables)
>>> console.interact(banner="Custom REPL", exitmsg="Goodbye!")

# Remote debugging REPL
>>> import code
>>> def start_debug_shell(globals_dict):
...     code.interact(local=globals_dict)
...
# Call from breakpoint to get interactive shell at that point
```

### Real-World Use Cases
- **Exploratory Data Analysis**: Testing Pandas operations interactively
- **API Exploration**: Testing HTTP requests and parsing responses
- **Debugging**: Dropping into a REPL at breakpoints with `code.interact()`
- **Education**: Teaching Python concepts interactively
- **System Administration**: Managing servers with interactive Python

### Common Mistakes
```python
# Mistake 1: Confusing REPL output with return value
>>> print("hello")  # Output: hello
'hello'  # Wait, why? print() returns None!

# Mistake 2: Forgetting to assign variables
>>> 42  # Prints but not stored
>>> _   # Can access via _

# Mistake 3: Tab completion not working
# Fix: Install readline (Unix) or pyreadline (Windows)
```

### Best Practices
- Use `dir()` and `help()` extensively for module exploration
- Access the last result with `_` (single underscore)
- Use `%history` in IPython (Jupyter) for session persistence
- Keep REPL sessions organized with `reset()` or new instances
- Use `Ctrl+D` (Unix) or `Ctrl+Z` (Windows) to exit
- Use `IPython` for a richer REPL experience with syntax highlighting

### Performance Considerations
- The REPL is single-threaded within the main thread
- Large computations block the REPL until complete
- Memory accumulates during long sessions; use `del` for large objects
- Import time is paid once per session; subsequent imports are cached
- IPython uses more memory but provides better introspection

### Interview Questions
1. What does REPL stand for and how does it work?
2. What is the `_` variable in the Python REPL?
3. How do you create a custom REPL?
4. What is the difference between `exec()` and `eval()` in the context of the REPL?
5. How does IPython differ from the standard Python REPL?
6. How can you use the REPL for debugging?
7. What is `code.interact()` and when would you use it?
8. How does the REPL handle multi-line statements?
9. What happens when you type an expression in the REPL?
10. How can you customize the REPL prompts?

### Coding Challenges
```python
# Challenge: Build a minimal REPL
def mini_repl():
    import sys
    namespace = {}
    print("Mini Python REPL (type 'exit()' to quit)")
    while True:
        try:
            code = input(">>> ")
            if code.strip() == "exit()":
                break
            result = eval(code, namespace)
            if result is not None:
                print(repr(result))
        except SyntaxError:
            try:
                exec(code, namespace)
            except Exception as e:
                print(f"Error: {e}")
        except Exception as e:
            print(f"Error: {e}")

mini_repl()

# Challenge: IPython-style history
from collections import deque
history = deque(maxlen=100)

class HistoryREPL:
    def __init__(self):
        self.namespace = {}
        self.counter = 0
    
    def run(self):
        while True:
            try:
                code = input(f"In [{self.counter}]: ")
                if code.strip() == "exit()":
                    break
                self.counter += 1
                history.append(code)
                result = eval(code, self.namespace)
                if result is not None:
                    print(f"Out[{self.counter-1}]: {result!r}")
            except SyntaxError:
                exec(code, self.namespace)
            except Exception as e:
                print(f"Error: {e}")
```

### Related Topics
- The `code` Module
- IPython and Jupyter
- `exec()` and `eval()` Functions
- Interactive Debugging
- Readline and History

## Running Scripts

### What It Is
Running scripts is the process of executing Python code stored in `.py` files. Scripts can be run directly (`python script.py`), as modules (`python -m module`), or as executable files (with shebang on Unix).

### Why It Is Important
Scripts are the primary way to package and distribute Python programs. Understanding script execution enables proper structuring of programs, command-line argument handling, and cross-platform compatibility.

### How It Works Internally
When `python script.py` runs, the interpreter opens the file, reads it as UTF-8 by default (Python 3+), compiles it to bytecode, and executes it in the `__main__` namespace. The `__name__` variable is set to `"__main__"` for the entry-point script. The exit code of the process is the return value of `sys.exit()`.

### Syntax
```bash
# Direct script execution
python script.py arg1 arg2 arg3

# Module execution (runs module's __main__.py or the file)
python -m http.server 8000
python -m unittest discover

# Executable script (Unix shebang)
#!/usr/bin/env python3
# chmod +x script.py
./script.py

# One-liners
python -c "print(sum(range(100)))"

# Optimized mode (removes __debug__ blocks)
python -O script.py
```

### Beginner Examples
```python
# script.py — Basic script structure
#!/usr/bin/env python3
"""
Simple script that processes command-line arguments.
"""
import sys

def main():
    print(f"Script: {sys.argv[0]}")
    print(f"Arguments: {sys.argv[1:]}")
    print(f"Arg count: {len(sys.argv) - 1}")

if __name__ == "__main__":
    main()
```

### Intermediate Examples
```python
#!/usr/bin/env python3
"""Script with argument parsing and error handling."""
import argparse
import sys
from pathlib import Path

def parse_args():
    parser = argparse.ArgumentParser(
        description="Process files with options",
        formatter_class=argparse.RawDescriptionHelpFormatter,
    )
    parser.add_argument("input", type=Path, help="Input file path")
    parser.add_argument("-o", "--output", type=Path, help="Output file path")
    parser.add_argument("-v", "--verbose", action="store_true", help="Verbose output")
    parser.add_argument("--limit", type=int, default=0, help="Line limit")
    return parser.parse_args()

def process_file(input_path: Path, output_path: Path, limit: int, verbose: bool):
    if verbose:
        print(f"Processing {input_path} -> {output_path}")
    
    with open(input_path, "r", encoding="utf-8") as f:
        lines = f.readlines()
    
    if limit > 0:
        lines = lines[:limit]
    
    with open(output_path, "w", encoding="utf-8") as f:
        f.writelines(lines)
    
    print(f"Wrote {len(lines)} lines to {output_path}")

def main():
    args = parse_args()
    
    if not args.input.exists():
        print(f"Error: {args.input} not found", file=sys.stderr)
        sys.exit(1)
    
    output = args.output or args.input.with_suffix(".out.txt")
    
    try:
        process_file(args.input, output, args.limit, args.verbose)
    except Exception as e:
        print(f"Error: {e}", file=sys.stderr)
        sys.exit(2)

if __name__ == "__main__":
    main()
```

### Advanced Examples
```python
#!/usr/bin/env python3
"""Production-grade script with logging, config, and signal handling."""
import argparse
import logging
import signal
import sys
from pathlib import Path
from typing import Optional

logger = logging.getLogger(__name__)

class GracefulShutdown:
    def __init__(self):
        self.shutdown_requested = False
        signal.signal(signal.SIGINT, self._handler)
        signal.signal(signal.SIGTERM, self._handler)
    
    def _handler(self, signum, frame):
        logger.warning(f"Received signal {signum}, shutting down...")
        self.shutdown_requested = True
    
    def is_shutdown_requested(self) -> bool:
        return self.shutdown_requested

def setup_logging(verbose: bool):
    level = logging.DEBUG if verbose else logging.INFO
    logging.basicConfig(
        level=level,
        format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
        datefmt="%Y-%m-%d %H:%M:%S",
    )

def parse_args(argv: Optional[list] = None):
    parser = argparse.ArgumentParser(
        description="Production-grade Python script",
        formatter_class=argparse.ArgumentDefaultsHelpFormatter,
    )
    parser.add_argument("--config", type=Path, default=Path("config.yaml"))
    parser.add_argument("--verbose", "-v", action="store_true")
    parser.add_argument("--workers", type=int, default=4)
    return parser.parse_args(argv)

def main():
    args = parse_args()
    setup_logging(args.verbose)
    shutdown = GracefulShutdown()
    
    logger.info(f"Starting with config: {args.config}")
    logger.info(f"Workers: {args.workers}")
    
    try:
        # Main work loop
        while not shutdown.is_shutdown_requested():
            # ... do work ...
            import time
            time.sleep(1)
            logger.debug("Processing tick...")
    except Exception as e:
        logger.exception(f"Fatal error: {e}")
        sys.exit(1)
    finally:
        logger.info("Shutdown complete")

if __name__ == "__main__":
    main()
```

### Real-World Use Cases
- **CLI Tools**: Git, pip, pytest are all Python scripts
- **Web Servers**: Gunicorn, uWSGI run WSGI apps
- **Data Pipelines**: ETL scripts processing data batch jobs
- **DevOps**: Deployment scripts, monitoring agents
- **System Tools**: File watchers, backup scripts, log analyzers

### Common Mistakes
```python
# Mistake 1: Not using if __name__ == "__main__"
# Code runs on import, not just execution
def main():
    print("Hello")

main()  # Wrong! Runs when imported

# Mistake 2: Hardcoding paths
data_file = "data.txt"  # Relative to CWD, not script location

# Correct: use __file__
from pathlib import Path
data_file = Path(__file__).parent / "data.txt"

# Mistake 3: Ignoring return codes
import sys
# Always exit with meaningful codes:
sys.exit(0)  # Success
sys.exit(1)  # General error
```

### Best Practices
- Always use `if __name__ == "__main__":` guard
- Use `argparse` (not `sys.argv` directly) for complex CLIs
- Return meaningful exit codes (0 for success, non-zero for errors)
- Handle `SIGINT` and `SIGTERM` for graceful shutdown
- Use logging instead of print for production scripts
- Make scripts importable (not just executable)
- Add shebang for Unix compatibility
- Use `pathlib` for cross-platform path handling

### Performance Considerations
- Script startup time is dominated by imports (`import` overhead)
- `python -O` provides minimal speedup; use it for assert removal
- For scripts that run frequently, consider `__pycache__` precompilation
- For long-running scripts, memory leaks accumulate; use `weakref` for caches
- Profile before optimizing: `python -m cProfile script.py`

### Interview Questions
1. What is the purpose of `if __name__ == "__main__":`?
2. How do you handle command-line arguments in Python?
3. What exit codes should a script return and why?
4. How do you make a Python script executable on Unix?
5. What is the difference between `python script.py` and `python -m module`?
6. How do you handle Ctrl+C in a Python script?
7. What is the `argparse` module and why use it over `sys.argv`?
8. How do you ensure Python scripts work cross-platform?
9. What is the role of `__main__.py` in packages?
10. How do you debug a Python script that crashes on startup?

### Coding Challenges
```python
# Challenge: Build a CLI file search tool
#!/usr/bin/env python3
import argparse
import sys
from pathlib import Path

def search_files(root: Path, pattern: str, recursive: bool = False):
    method = root.rglob if recursive else root.glob
    for path in method(pattern):
        yield path

def main():
    parser = argparse.ArgumentParser(description="File search utility")
    parser.add_argument("pattern", help="Glob pattern to search")
    parser.add_argument("root", nargs="?", default=".", type=Path)
    parser.add_argument("-r", "--recursive", action="store_true")
    parser.add_argument("-v", "--verbose", action="store_true")
    
    args = parser.parse_args()
    
    if not args.root.exists():
        print(f"Error: {args.root} not found", file=sys.stderr)
        sys.exit(1)
    
    for path in search_files(args.root, args.pattern, args.recursive):
        print(path)

if __name__ == "__main__":
    main()
```

### Related Topics
- The `argparse` Module
- The `sys` Module (argv, exit, path)
- The `pathlib` Module
- Logging Best Practices
- Signal Handling
- `if __name__ == "__main__"` Idiom
