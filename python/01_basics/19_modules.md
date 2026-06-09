# Modules - import, from...import, __name__, sys.path

## Introduction

Modules are Python files (`.py`) containing definitions and statements that can be imported and used in other Python files. They organize code into logical units, prevent naming conflicts via namespaces, and promote reusability across projects. Python ships with a rich standard library (200+ modules), and the ecosystem provides hundreds of thousands of third-party modules via PyPI. Understanding Python's import system, module search path, and the `__name__` variable is essential for structuring any non-trivial project.

## import statement

### What It Is

The `import` statement loads a module by name, executing its top-level code and making its contents available under the module's namespace. Once loaded, the module object is cached in `sys.modules` so subsequent imports of the same module are instantaneous.

### Why It Is Important

`import` is the primary mechanism for code reuse in Python. It provides access to the standard library, third-party packages, and your own project modules. Understanding import semantics, caching, and execution order prevents circular-import bugs and initialization issues.

### How It Works Internally

When `import foo` executes, CPython follows these steps:
1. Check `sys.modules` — if found, return the cached module (O(1) dict lookup)
2. Find the module via `sys.meta_path` finders: `importlib.machinery.PathFinder` for file-based modules, `BuiltinImporter` for C builtins, and `FrozenImporter` for frozen modules
3. If a spec is found (`ModuleSpec`), create a new module object and execute its code in the module's namespace (`__dict__`)
4. Insert the module into `sys.modules` (atomic operation)
5. Bind the name `foo` in the current scope to the module object

Bytecode: `import foo` compiles to `IMPORT_NAME foo` followed by `STORE_NAME foo`.

### Syntax

```python
# Import entire module
import math

# Import multiple modules
import os, sys, json

# Import with alias
import numpy as np
import pandas as pd

# Import submodule
import os.path

# Dynamic import
import importlib
module = importlib.import_module("module_name")
```

### Beginner Examples

```python
# Basic import
import math
print(math.pi)           # 3.141592653589793
print(math.sqrt(16))     # 4.0
print(math.factorial(5)) # 120

# Multiple imports
import os, sys
print(os.getcwd())       # Current working directory
print(sys.version)       # Python version info

# Import with alias
import datetime as dt
now = dt.datetime.now()
print(now)

# Submodule import
import os.path
print(os.path.exists("."))  # True
```

### Intermediate Examples

```python
# Import side effects
# When a module is imported, its top-level code runs once
# bad_module.py:
# print("Module initialized!")  # Runs on import
# DATABASE_URL = "postgres://..."

# Module singleton behavior
import math as m1
import math as m2
print(m1 is m2)  # True (same cached module)

# Import inspection
import json
print(type(json))          # <class 'module'>
print(json.__name__)       # json
print(json.__file__)       # /usr/lib/python3.x/json/__init__.py
print(json.__doc__[:50])   # JSON encoder and decoder...

# Import order matters in some cases
# Circular imports fail silently only if modules are fully loaded
# file_a.py:
# import file_b   # At this point file_b is partially loaded
# def func_a(): return file_b.func_b()

# Conditional imports
try:
    import numpy as np
    HAS_NUMPY = True
except ImportError:
    HAS_NUMPY = False
    np = None

# Dynamic import
import importlib
module_name = "json"
mod = importlib.import_module(module_name)
print(mod.loads('{"a": 1}'))  # {'a': 1}
```

### Advanced Examples

```python
# Lazy module import
class LazyModule:
    def __init__(self, name):
        self._name = name
        self._mod = None
    
    def __getattr__(self, name):
        if self._mod is None:
            import importlib
            self._mod = importlib.import_module(self._name)
        return getattr(self._mod, name)

lazy_json = LazyModule("json")
# json module not imported yet
data = lazy_json.loads('{"key": "value"}')
# Import happens on first attribute access

# Module reloading (development)
import importlib
# import mymodule
# importlib.reload(mymodule)  # Re-executes the module

# Import from arbitrary file path
import importlib.util
import sys

def import_from_path(path, name=None):
    if name is None:
        name = path.replace("/", ".").replace("\\", ".").rstrip(".py")
    spec = importlib.util.spec_from_file_location(name, path)
    module = importlib.util.module_from_spec(spec)
    sys.modules[name] = module
    spec.loader.exec_module(module)
    return module

# mod = import_from_path("/path/to/custom/module.py")

# Import all submodules of a package
def import_submodules(package):
    import pkgutil
    results = {}
    for importer, modname, ispkg in pkgutil.iter_modules(package.__path__):
        full_name = f"{package.__name__}.{modname}"
        results[modname] = importlib.import_module(full_name)
    return results

# import os
# submodules = import_submodules(os)
# print(list(submodules.keys()))
```

### Real-World Use Cases

- **Standard library access**: `import os`, `sys`, `json`, `re`, `datetime`
- **Project structure**: Splitting code across multiple files
- **Third-party libraries**: `import requests`, `import flask`, `import pandas`
- **Plugin systems**: Dynamic module loading for extensions
- **Conditional feature support**: Try-import optional dependencies
- **Lazy loading**: Defer expensive imports until needed
- **Configuration**: Importing settings modules

### Common Mistakes

```python
# Mistake 1: Circular imports
# a.py: from b import func_b
# b.py: from a import func_a  # Error: cannot import partially initialized module
# Fix: import module-level (import b), restructure, or use late imports

# Mistake 2: Import * (namespace pollution)
from math import *  # Imports ~50 names, may shadow your variables
# sin = 42  # Now shadows math.sin
# Fix: import math / from math import sin

# Mistake 3: Module-level side effects
# config.py:
print("Loading config...")  # Runs on every import of this module!
# Fix: Put print in if __name__ == "__main__":

# Mistake 4: ImportError not handled
try:
    import numpy as np
except ImportError:
    print("numpy not available")
    # Provide fallback

# Mistake 5: Overwriting module reference
import math
math = "string"  # Now you can't access math module!
```

### Best Practices

- Place imports at the top of the file (PEP 8)
- Order imports: standard library, third-party, local (blank line between)
- Use explicit imports: `from module import name` not `import *`
- Use `import module` when using many names from it
- Use `import module as alias` for shortening long names
- Handle ImportError for optional dependencies gracefully
- Avoid circular imports (restructure to prevent)
- Keep modules focused on a single responsibility
- Document modules with docstrings

### Performance Considerations

First import executes the entire module (can be slow for heavy modules). Subsequent imports are O(1) dict lookup in `sys.modules`. Use lazy imports for heavy dependencies that aren't always needed. `import module` is slightly faster than `from module import name` because it avoids binding specific names. However, `from module import name` can be faster for repeated attribute access (avoids module dict lookup each time).

### Interview Questions

1. What is the difference between `import module` and `from module import name`?
2. How does Python avoid re-importing the same module multiple times?
3. What is the import sequence (sys.modules, finders, loaders)?
4. How do you handle circular imports?
5. What is the difference between `import module` and `import module as m`?
6. How do you dynamically import a module?
7. How do you reload a module?
8. What happens when a module is imported?
9. How does `import *` work and why is it discouraged?

### Coding Challenges

```python
# Challenge 1: Import time benchmark
import time
import importlib

def time_import(module_name, iterations=100):
    start = time.perf_counter()
    for _ in range(iterations):
        importlib.invalidate_caches()
        importlib.import_module(module_name)
    avg = (time.perf_counter() - start) / iterations * 1000
    print(f"{module_name}: {avg:.4f}ms")

# time_import("json", 10)

# Challenge 2: Module dependency analyzer
def get_dependencies(module):
    import inspect
    deps = set()
    for name, obj in inspect.getmembers(module):
        if isinstance(obj, type(module)):
            deps.add(obj.__name__)
    return deps

import json
print(get_dependencies(json))

# Challenge 3: Custom importer
class OnlyAllowList(list):
    pass

import sys

class SafeImporter:
    allowed = {"math", "json", "os", "sys"}
    
    @classmethod
    def find_spec(cls, fullname, path, target=None):
        if fullname not in cls.allowed:
            raise ImportError(f"{fullname} is not in the allowlist")
        return None  # Delegate to default finder

# sys.meta_path.insert(0, SafeImporter)
# import math  # Works
# import flask  # ImportError

# Challenge 4: Module as singleton config
# config.py:
# _settings = {}
# 
# def get(key):
#     return _settings.get(key)
# 
# def set(key, value):
#     global _settings
#     _settings[key] = value

# Usage in multiple files, settings are shared across all imports
```

### Related Topics

- Packages (__init__.py)
- from import statement
- __name__ variable
- sys.path module search
- importlib module
- Circular imports

## from import statement

### What It Is

`from module import name` imports a specific name from a module into the current namespace, allowing direct access without the module prefix. It can also import all names with `from module import *` (discouraged in production code).

### Why It Is Important

Selective imports reduce verbosity by eliminating the module prefix, improve code readability for frequently-used names, and make dependencies explicit. They also reduce memory usage by only importing needed names.

### How It Works Internally

`from foo import bar` compiles to:
1. `IMPORT_NAME foo` — loads the module (or retrieves from cache)
2. `IMPORT_FROM bar` — retrieves `bar` from the module's dict
3. `STORE_NAME bar` — binds `bar` in the current scope

The module is still fully loaded and cached — the difference is only which names are bound locally. `from foo import *` uses `STAR_IMPORT` bytecode, which copies all public names (those not starting with `_`, or those in `__all__`) into the current namespace.

### Syntax

```python
# Import specific name
from math import pi, sqrt

# Import with alias
from math import factorial as fact

# Import all (avoid)
from math import *

# Import from submodule
from os.path import join, exists

# Relative import (inside packages)
from . import sibling_module
from ..parent_module import func
```

### Beginner Examples

```python
# Import specific names
from math import pi, sqrt
print(pi)         # 3.14159
print(sqrt(25))   # 5.0

# Import with alias
from datetime import datetime as dt
now = dt.now()
print(now)

# Multiple from same module
from os import getcwd, listdir
print(getcwd())
print(listdir(".")[:5])

# Import from submodule
from os.path import join, exists
path = join("folder", "file.txt")
print(exists(path))
```

### Intermediate Examples

```python
# __all__ controls import *
# math.__all__ doesn't exist, so * imports all non-underscore names
# Custom modules can define:
# __all__ = ["func1", "func2", "ClassName"]

# Import with rename for conflict resolution
from statistics import mean as avg
from numpy import mean as np_mean

# Selective import vs module reference
# Module reference:
import json
data = json.loads('{"a": 1}')

# Selective import:
from json import loads
data = loads('{"a": 1}')  # Slightly faster for repeated use

# Import with error handling
try:
    from collections import Counter
except ImportError:
    # Fallback for older Python
    class Counter(dict):
        pass

# Hidden names (_ prefix are not imported by *)
from os import _exists  # Can still import explicitly
```

### Advanced Examples

```python
# Relative imports in packages
# Structure:
# mypackage/
#   __init__.py
#   utils.py
#   core.py

# utils.py:
# from . import config  # Import sibling module
# from ..parent import helper  # Import from parent package

# Re-export pattern
# __init__.py:
from .utils import format_data
from .core import process
__all__ = ["format_data", "process"]

# Conditional import with fallback
try:
    import tomllib  # Python 3.11+
except ImportError:
    import tomli as tomllib

# Dynamic from-import
module_name = "os"
name = "getcwd"
mod = __import__(module_name, fromlist=[name])
func = getattr(mod, name)
print(func())  # Calls os.getcwd()

# Import side effects -- module runs once
# Even with from import, the module fully executes
from math import pi  # math module still fully loaded
import sys
print("math" in sys.modules)  # True
```

### Real-World Use Cases

- **Frequently used functions**: `from os.path import join`, `from re import search`
- **Typing**: `from typing import List, Dict, Optional`
- **Framework imports**: `from flask import Flask, request, jsonify`
- **Testing**: `from unittest.mock import patch, Mock`
- **Data science**: `from pandas import DataFrame`, `from numpy import array`
- **Aliasing for clarity**: `from decimal import Decimal as D`

### Common Mistakes

```python
# Mistake 1: Circular import with from-import
# a.py: from b import func
# b.py: from a import func  # ImportError: cannot import name 'func'
# Fix: Use import module (not from) in circular scenarios

# Mistake 2: Shadowing built-ins
from math import sum  # Hides built-in sum()
from os import id     # Hides built-in id()
# Be cautious about name conflicts

# Mistake 3: import * overwriting
x = 42
from math import *  # If math has x defined, it overwrites!
# Avoid import * in production

# Mistake 4: Forgetting relative import scope
# Relative imports (.parent) only work inside packages
# Running a module directly: can't use relative imports

# Mistake 5: Importing submodule vs function
from os import path  # path is a submodule (works)
from os import path.join  # SyntaxError! Can't chain in from-import
# Fix: from os.path import join
```

### Best Practices

- Favor `from module import name` for frequently used names
- Favor `import module` when using many names or avoiding name conflicts
- Never use `from module import *` in library or production code
- Use explicit `__all__` in your modules to control exported names
- Use relative imports within packages (PEP 328)
- Keep import lines explicit (one name per line or group by module)
- Use `from module import name as alias` for name disambiguation

### Performance Considerations

`from module import name` is slightly faster than `module.name` for repeated access (avoids module-attr lookup). But the import itself is the same speed. The performance benefit is only significant in tight loops. Use local binding for hot paths:

```python
# Slow in loop:
import math
for x in data:
    y = math.sqrt(x)  # Module lookup each iteration

# Fast:
from math import sqrt
for x in data:
    y = sqrt(x)  # Direct local reference
```

### Interview Questions

1. What is the difference between `import module` and `from module import name`?
2. Does `from module import name` prevent the rest of the module from executing?
3. What is `__all__` and how does it affect `from module import *`?
4. How do relative imports work?
5. What happens with circular imports when using `from x import y`?
6. How do you import a name that conflicts with a built-in?
7. Why is `from module import *` discouraged?

### Coding Challenges

```python
# Challenge 1: Safe dynamic from-import
def safe_import(module_name, name, fallback=None):
    try:
        mod = __import__(module_name, fromlist=[name])
        return getattr(mod, name)
    except (ImportError, AttributeError):
        return fallback

sqrt_func = safe_import("math", "sqrt", lambda x: x**0.5)
print(sqrt_func(25))  # 5.0

# Challenge 2: Import dependency graph
def get_from_imports(filepath):
    import ast
    with open(filepath) as f:
        tree = ast.parse(f.read())
    imports = []
    for node in ast.walk(tree):
        if isinstance(node, ast.ImportFrom):
            for alias in node.names:
                imports.append((node.module, alias.name, alias.asname))
    return imports

# print(get_from_imports("my_module.py"))

# Challenge 3: Selective re-exporter
def selective_export(module, names):
    result = {}
    for name in names:
        if hasattr(module, name):
            result[name] = getattr(module, name)
    return result

import math
exported = selective_export(math, ["pi", "sqrt", "nonexistent"])
print(list(exported.keys()))  # ['pi', 'sqrt']
```

### Related Topics

- import statement
- __all__ variable
- Relative imports
- Circular imports
- importlib module

## __name__ variable

### What It Is

`__name__` is a special variable set by Python in every module. When a module is run directly (e.g., `python script.py`), `__name__` is set to `"__main__"`. When imported, `__name__` is set to the module's name (e.g., `"math"`, `"os.path"`).

### Why It Is Important

The `__name__` variable enables the dual-purpose module pattern — a file can be both a reusable module (when imported) and an executable script (when run directly). This is essential for testing, examples, and CLI entry points packaged with libraries.

### How It Works Internally

When Python runs a script with `python script.py`, it creates a `__main__` module and sets `__name__ = "__main__"`. The script's code is compiled and executed as the body of this module. When imported, a new module object is created with `__name__` set to the import path. The check `if __name__ == "__main__":` compiles to a simple comparison that resolves at runtime based on how the file was executed.

### Syntax

```python
# Dual-purpose module pattern
if __name__ == "__main__":
    # Code here only runs when executed directly
    # Not when imported
    main()
```

### Beginner Examples

```python
# Save as calculator.py
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

if __name__ == "__main__":
    # Test code -- runs only when executed directly
    print("Testing calculator module:")
    print(f"add(5, 3) = {add(5, 3)}")
    print(f"subtract(5, 3) = {subtract(5, 3)}")

# When imported: from calculator import add
# The test code does NOT run
```

### Intermediate Examples

```python
# CLI tool pattern
import sys

def process_data(filename):
    with open(filename) as f:
        return len(f.readlines())

def main():
    if len(sys.argv) != 2:
        print("Usage: python script.py <filename>")
        sys.exit(1)
    count = process_data(sys.argv[1])
    print(f"Lines: {count}")

if __name__ == "__main__":
    main()

# Test/demo pattern
def factorial(n):
    if n <= 1:
        return 1
    return n * factorial(n - 1)

if __name__ == "__main__":
    import time
    for n in [5, 10, 15]:
        start = time.perf_counter()
        result = factorial(n)
        elapsed = time.perf_counter() - start
        print(f"factorial({n}) = {result} ({elapsed:.6f}s)")
```

### Advanced Examples

```python
# Running module-level benchmarks
def fibonacci(n):
    if n < 2:
        return n
    a, b = 0, 1
    for _ in range(n):
        a, b = b, a + b
    return a

if __name__ == "__main__":
    import timeit
    n = 1000
    result = timeit.timeit(lambda: fibonacci(100), number=n)
    print(f"fibonacci(100) averaged {result/n*1e6:.2f}us")

# Interactive exploration
if __name__ == "__main__":
    import code
    code.interact(local=locals())

# Unit test integration
if __name__ == "__main__":
    import unittest
    unittest.main()

# Running doctests
if __name__ == "__main__":
    import doctest
    doctest.testmod()
```

### Real-World Use Cases

- **CLI entry points**: Scripts that provide command-line functionality
- **Testing**: Running module-level tests when executed directly
- **Examples**: Demo code that users can run after installing a library
- **Benchmarks**: Performance tests included in the module file
- **Exploration**: Interactive shell for module exploration
- **Debugging**: Module-level debug output that doesn't run on import

### Common Mistakes

```python
# Mistake 1: Forgetting __name__ == "__main__" check
# If you have code that runs on import that shouldn't:
# print("Module loaded!")  # DON'T -- runs on import!
if __name__ == "__main__":
    print("Module loaded!")

# Mistake 2: Running production code at module level
DATABASE_URL = "postgres://localhost/db"

# Don't do this:
# try:
#     connection = connect(DATABASE_URL)  # Runs on import!
# except:
#     pass

# Instead, lazy initialize:
def get_connection():
    return connect(DATABASE_URL)

# Mistake 3: Name collision with __name__
# __name__ = "custom"  # Don't reassign __name__!

# Mistake 4: Assuming __name__ in imported modules
# __name__ in imported modules is the module name, not "__main__"
```

### Best Practices

- Always guard script code with `if __name__ == "__main__":`
- Define a `main()` function and call it from the guard
- Keep module-level code to definitions only (imports, constants, functions, classes)
- Use the guard for tests, examples, and demos
- Include usage help in the guard (`argparse`, `sys.argv` handling)
- Use the guard for running doctests/unittests

### Performance Considerations

The `if __name__ == "__main__":` check is O(1) string comparison. There is no measurable overhead. Python even peephole-optimizes the comparison in some cases.

### Interview Questions

1. What is the value of `__name__` when a script is run directly?
2. What is the value when a module is imported?
3. What is the `if __name__ == "__main__":` pattern used for?
4. Can you have multiple `if __name__ == "__main__":` blocks?
5. How do you make a module both importable and executable?
6. What happens if you don't use the guard?
7. How does pytest handle the `__name__` guard?

### Coding Challenges

```python
# Challenge 1: Self-testing module
def add(a, b): return a + b
def subtract(a, b): return a - b

if __name__ == "__main__":
    assert add(2, 3) == 5
    assert subtract(5, 3) == 2
    print("All tests passed!")

# Challenge 2: CLI with argparse
if __name__ == "__main__":
    import argparse
    parser = argparse.ArgumentParser()
    parser.add_argument("--name", default="World")
    parser.add_argument("--greeting", default="Hello")
    args = parser.parse_args()
    print(f"{args.greeting}, {args.name}!")

# Challenge 3: Module that detects its own execution mode
def execution_mode():
    return "script" if __name__ == "__main__" else "import"

print(f"Module is being {execution_mode()}ed")
```

### Related Topics

- Module execution model
- sys.modules
- __main__ module
- __file__ variable
- Entry points (console_scripts)
- argparse

## sys.path

### What It Is

`sys.path` is a list of directory paths where Python searches for modules. The first element is typically the script's directory (or the current directory when running interactively). The rest includes `PYTHONPATH` environment variable entries and Python's installation-dependent defaults.

### Why It Is Important

Understanding `sys.path` is crucial for resolving import errors, managing virtual environments, organizing project code into importable packages, and installing third-party libraries in discoverable locations.

### How It Works Internally

When a module needs to be imported, Python iterates through:
1. `sys.modules` cache (first, fastest)
2. Each finder in `sys.meta_path` (the `PathFinder` for `sys.path`)
3. The `PathFinder` searches each directory in `sys.path` in order
4. For each directory, it looks for the module file (`module.py`, `module/__init__.py`, or compiled extensions like `.pyd`/`.so`)
5. The first match wins

`sys.path` is initialized at startup from:
- The directory of the script being run (or current directory for interactive mode)
- `PYTHONPATH` environment variable
- Site-packages directories (from `site` module)
- Standard library directory

### Syntax

```python
import sys

# Inspect search paths
print(sys.path)

# Add a custom path
sys.path.append("/my/custom/path")

# Insert at beginning (higher priority)
sys.path.insert(0, "/my/custom/path")

# Remove a path
sys.path.remove("/unwanted/path")
```

### Beginner Examples

```python
import sys

# View all module search paths
print("Python module search paths:")
for i, path in enumerate(sys.path):
    print(f"  {i}: {path}")

# The first entry is usually the script directory
import os
print(f"Script directory: {os.path.dirname(os.path.abspath(__file__))}")
print(f"Matches sys.path[0]: {sys.path[0]}")

# Add a custom directory
sys.path.insert(0, r"C:\MyPythonModules")

# Now you can import from that directory
# import my_custom_module
```

### Intermediate Examples

```python
# PYTHONPATH environment variable
import os
# Can be set before running:
# export PYTHONPATH=/my/project:/other/project  (Linux)
# $env:PYTHONPATH = "C:\my\project;C:\other\project"  (Windows)
# It gets merged into sys.path automatically

# Site-packages location
site_packages = [p for p in sys.path if "site-packages" in p]
print(f"Third-party packages: {site_packages}")

# Find where a module lives
import json
print(json.__file__)  # Shows the path

# Check if a path exists before adding
custom_path = r"D:\my_libs"
if os.path.isdir(custom_path) and custom_path not in sys.path:
    sys.path.append(custom_path)

# Temporary path modification using context manager
class PathContext:
    def __init__(self, *paths):
        self.paths = paths
    
    def __enter__(self):
        for p in self.paths:
            if p not in sys.path:
                sys.path.insert(0, p)
        return self
    
    def __exit__(self, *args):
        for p in self.paths:
            if p in sys.path:
                sys.path.remove(p)

# with PathContext(r"D:\my_libs"):
#     import my_lib
```

### Advanced Examples

```python
# Custom path finder
import sys
import importlib.abc

class VersionedPathFinder(importlib.abc.MetaPathFinder):
    def __init__(self, version_map):
        self.version_map = version_map  # {module_name: path}
    
    def find_spec(self, fullname, path, target=None):
        if fullname in self.version_map:
            path = self.version_map[fullname]
            return importlib.machinery.PathFinder.find_spec(fullname, [path])
        return None

# sys.meta_path.insert(0, VersionedPathFinder({"my_module": "/special/path"}))

# PTH file processing (site module)
# Python processes .pth files in site-packages to add paths
# my_paths.pth file:
# D:\my\library
# import my_side_effect_script

# .pth files can contain:
# - Directory paths (one per line)
# - import statements (prefixed with "import ")
# This is how many packages add themselves to sys.path

# Virtual environment path isolation
import site
def in_virtualenv():
    return hasattr(sys, "real_prefix") or (
        hasattr(sys, "base_prefix") and sys.base_prefix != sys.prefix
    )

print(f"Virtual env: {in_virtualenv()}")

# Disabling user site-packages
# python -s script.py  # Skip user site
# site.ENABLE_USER_SITE = False
```

### Real-World Use Cases

- **Development modules**: Adding source directories during development
- **Virtual environments**: Isolated site-packages
- **Custom library locations**: Non-standard install paths
- **Plugin discovery**: Adding plugin directories at runtime
- **Mock/stub injection**: Replacing modules with test doubles
- **Version management**: Using different module versions
- **Embedded Python**: Configuring paths for embedded use cases

### Common Mistakes

```python
# Mistake 1: sys.path modifications after imports
# If you add a path after importing, modules there won't be found
import sys
sys.path.append("/path/to/modules")
import my_module  # Works fine -- actually this is OK in Python

# But if you need to override an already-imported module:
# import json
# sys.path.insert(0, "/custom/json")
# import json  # Still the cached version!
# del sys.modules["json"]
# import json  # Now loads from custom path

# Mistake 2: Hardcoding absolute paths
sys.path.append("C:/Users/me/project/libs")  # Not portable!
# Use __file__-relative paths:
import os
sys.path.append(os.path.join(os.path.dirname(__file__), "libs"))

# Mistake 3: Modifying sys.path in library code
# Libraries should NOT modify sys.path (unexpected side effects)
# Use relative imports or request users to set PYTHONPATH

# Mistake 4: Assuming path order
# sys.path[0] is the script directory, but this may not be what you expect
# when running from different directories

# Mistake 5: Forgetting to check if path exists
sys.path.append("/nonexistent/path")  # Works but useless
```

### Best Practices

- Modify `sys.path` at the start of the script, before any imports
- Use `sys.path.insert(0, ...)` for highest priority
- Use relative paths based on `__file__`, not absolute paths
- Use `PYTHONPATH` environment variable for permanent additions
- Use virtual environments for project isolation instead of path manipulation
- Never modify `sys.path` in library code (only in scripts/applications)
- Check if a path exists before adding it
- Use `sys.path.append()` for lower priority, `insert(0)` for higher

### Performance Considerations

`sys.path` is searched sequentially until a module is found. Larger `sys.path` means slower first-time imports (especially over network drives). Python caches `PathFinder` directory listings, so repeated imports are fast. `sys.path.insert(0, ...)` at import time is negligible overhead.

### Interview Questions

1. What is `sys.path` and how is it initialized?
2. How do you add a directory to `sys.path`?
3. What is the difference between `sys.path.append` and `sys.path.insert(0, ...)`?
4. How does `PYTHONPATH` relate to `sys.path`?
5. How do you find where a module is located on disk?
6. What are `.pth` files?
7. How do virtual environments affect `sys.path`?
8. Why should libraries avoid modifying `sys.path`?

### Coding Challenges

```python
# Challenge 1: Module path inspector
def where_is(module_name):
    import importlib
    try:
        mod = importlib.import_module(module_name)
        return mod.__file__
    except ImportError:
        for path in sys.path:
            for ext in [".py", ".pyd", ".so"]:
                full = os.path.join(path, module_name + ext)
                if os.path.exists(full):
                    return full
    return None

print(where_is("json"))   # /usr/lib/python3.x/json/__init__.py
print(where_is("os.path"))

# Challenge 2: Safe path adder
def ensure_on_path(directory):
    import os, sys
    directory = os.path.abspath(directory)
    if os.path.isdir(directory) and directory not in sys.path:
        sys.path.insert(0, directory)
        return True
    return False

# ensure_on_path("./lib")

# Challenge 3: Path resolver for project
def get_project_modules():
    import sys, os
    script_dir = os.path.dirname(os.path.abspath(__file__))
    project_root = os.path.dirname(script_dir)
    paths_to_add = [
        os.path.join(project_root, "src"),
        os.path.join(project_root, "lib"),
    ]
    for p in paths_to_add:
        if os.path.isdir(p) and p not in sys.path:
            sys.path.insert(0, p)
    return paths_to_add
```

### Related Topics

- Module import system
- PYTHONPATH environment variable
- Virtual environments
- site module
- .pth files
- importlib.machinery
- sys.meta_path
