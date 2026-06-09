# Modules - import, from...import, __name__, sys.path

## Introduction

Modules are Python files containing definitions and statements that can be imported and used in other Python files. They help organize code into logical units, promote reusability, and manage namespaces. Python comes with a extensive standard library of modules, and you can create your own modules or install third-party ones from PyPI.

## Why It Is Important

Modules are crucial because:
- They organize code into manageable, reusable files
- They prevent naming conflicts through namespaces
- They enable code sharing across projects
- They give access to Python's extensive standard library
- They support the `if __name__ == "__main__"` pattern for dual-purpose files
- They allow third-party package installation via pip

## Syntax

```python
# Import entire module
import module_name

# Import specific items
from module_name import function_name, ClassName

# Import with alias
import module_name as alias

# Import all (generally avoid)
from module_name import *

# Import submodule
import package.submodule

# Dynamic import
import importlib
module = importlib.import_module("module_name")
```

## Examples

### Importing Modules

```python
# Import entire module
import math
print(f"Pi: {math.pi}")
print(f"Factorial 5: {math.factorial(5)}")

# Import specific functions
from random import randint, choice
print(f"Random number 1-10: {randint(1, 10)}")
print(f"Random choice: {choice(['apple', 'banana', 'cherry'])}")

# Import with alias
import datetime as dt
now = dt.datetime.now()
print(f"Current time: {now}")

# Import submodule
import os.path
print(f"File exists: {os.path.exists('test.txt')}")

# Import all (use sparingly)
from math import *
print(sin(0))   # 0.0
print(cos(0))   # 1.0
```

### Creating and Using Modules

```python
# Save this as mymodule.py
"""
My custom module with utility functions.
"""

def greet(name):
    """Return a greeting."""
    return f"Hello, {name}!"

def add(a, b):
    """Add two numbers."""
    return a + b

PI = 3.14159

class Calculator:
    """Simple calculator class."""
    @staticmethod
    def multiply(a, b):
        return a * b
```

## Beginner Examples

### Using Standard Library Modules

```python
# os module - operating system interface
import os

print(f"Current directory: {os.getcwd()}")
print(f"All files: {os.listdir('.')}")
# os.mkdir("new_folder")  # Create directory

# sys module - system-specific parameters
import sys
print(f"Python version: {sys.version}")
print(f"Platform: {sys.platform}")
print(f"Arguments: {sys.argv}")

# datetime module
from datetime import datetime, date, timedelta

today = date.today()
print(f"Today: {today}")
print(f"Year: {today.year}, Month: {today.month}, Day: {today.day}")

# Date arithmetic
tomorrow = today + timedelta(days=1)
print(f"Tomorrow: {tomorrow}")

# random module
import random

print(f"Random float 0-1: {random.random()}")
print(f"Random int 1-100: {random.randint(1, 100)}")
print(f"Random choice: {random.choice(['heads', 'tails'])}")

# Shuffle
cards = list(range(1, 11))
random.shuffle(cards)
print(f"Shuffled: {cards}")

# math module
import math

print(f"Pi: {math.pi}")
print(f"Square root 16: {math.sqrt(16)}")
print(f"Ceil 3.14: {math.ceil(3.14)}")
print(f"Floor 3.14: {math.floor(3.14)}")
print(f"Sin(90°): {math.sin(math.radians(90))}")
```

### The if __name__ == "__main__" Pattern

```python
# Save as example_module.py
"""
This module demonstrates the __name__ == "__main__" pattern.
"""

def main():
    """Main function when run as script."""
    print("Running as script!")
    print(f"__name__ is: {__name__}")
    result = add_numbers(5, 3)
    print(f"5 + 3 = {result}")

def add_numbers(a, b):
    """Add two numbers."""
    return a + b

def helper_function():
    """Helper function for internal use."""
    return "Helper"

if __name__ == "__main__":
    # This code only runs when the file is executed directly
    # Not when imported as a module
    main()

# When imported, only functions and classes are available
# The main() block doesn't execute
```

## Intermediate Examples

### Module Search Path

```python
# sys.path shows where Python looks for modules
import sys
print("Python module search paths:")
for path in sys.path:
    print(f"  {path}")

# Adding a custom path
sys.path.append("/my/custom/path")

# Now you can import modules from that path
# import my_custom_module

# PYTHONPATH environment variable also adds paths
# set PYTHONPATH=/my/path; python script.py

# Inspecting a module's location
import os
print(f"os module location: {os.__file__}")

# Reloading a module (useful during development)
import importlib
# import mymodule
# importlib.reload(mymodule)
```

### Module Attributes

```python
# Useful module attributes
import math

# Get all names in a module
print("Names in math module:")
for name in dir(math):
    if not name.startswith("_"):
        print(f"  {name}")

# Module docstring
print(f"math docstring: {math.__doc__[:50]}...")

# Module file location
print(f"math file: {math.__file__}")

# Module name
print(f"math name: {math.__name__}")

# Gethelp on module
# help(math)

# Creating a module namespace
import math as mathematics
print(f"math is: {math.__name__}")
print(f"mathematics is: {mathematics.__name__}")
print(f"Same module: {math is mathematics}")
```

### Dynamic Imports

```python
# Dynamic import using importlib
import importlib

def dynamic_import(module_name):
    """Import a module dynamically by name."""
    try:
        module = importlib.import_module(module_name)
        return module
    except ImportError:
        print(f"Could not import {module_name}")
        return None

# Import modules dynamically
math_module = dynamic_import("math")
if math_module:
    print(f"Pi from dynamic import: {math_module.pi}")

json_module = dynamic_import("json")
if json_module:
    data = '{"name": "Alice"}'
    print(f"Parsed JSON: {json_module.loads(data)}")

# Try importing optional dependencies
def try_import(name, package_name=None):
    """Try to import a module, installing if needed."""
    try:
        return importlib.import_module(name)
    except ImportError:
        if package_name:
            print(f"Package {package_name} is not installed.")
            print(f"Install with: pip install {package_name}")
        return None

# numpy = try_import('numpy', 'numpy')
# if numpy:
#     print("NumPy is available")
```

## Advanced Examples

### Creating a Package with Multiple Modules

```python
# File structure:
# mypackage/
#     __init__.py
#     module1.py
#     module2.py

# __init__.py can be empty or contain package initialization
# from .module1 import function1
# from .module2 import function2

# Absolute imports
from mypackage import function1, function2

# Relative imports (inside package)
# from . import module1
# from .. import parent_module

# Import strategy examples
# import mypackage                   # Imports package
# from mypackage import module1     # Imports specific module
# from mypackage.module1 import fn  # Imports specific function
```

### Module as Singleton

```python
# Modules are singletons in Python
# They are loaded only once, subsequent imports get the same object

# config.py
"""
Configuration module - loaded once, shared across all imports.
"""
DEBUG = False
DATABASE_URL = "sqlite:///default.db"

def setup():
    """Initialize configuration."""
    global DEBUG, DATABASE_URL
    DEBUG = True
    DATABASE_URL = "postgresql://localhost/mydb"
    print("Configuration initialized")

# Usage in app1.py:
# from config import DEBUG, DATABASE_URL, setup
# setup()
# print(f"Debug: {DEBUG}")

# Usage in app2.py:
# from config import DATABASE_URL
# print(f"DB URL: {DATABASE_URL}")  # Same as in app1
```

### Lazy Module Loading

```python
# Lazy loading for expensive imports
class LazyModule:
    """Lazy module loader - imports only when accessed."""
    def __init__(self, module_name):
        self.module_name = module_name
        self._module = None
    
    def __getattr__(self, name):
        if self._module is None:
            self._module = __import__(self.module_name)
        return getattr(self._module, name)

# Usage - expensive import only when first accessed
lazy_math = LazyModule("math")
# math module not loaded yet
print(lazy_math.pi)  # Module imported here

# Alternative using importlib
class LazyImport:
    def __init__(self, name):
        self._name = name
        self._mod = None
    
    def __load(self):
        if self._mod is None:
            self._mod = importlib.import_module(self._name)
        return self._mod
    
    def __getattr__(self, name):
        return getattr(self.__load(), name)

numpy = LazyImport("numpy")
# numpy not imported until first access
# arr = numpy.array([1, 2, 3])  # Would import now
```

## Real-World Use Cases

- **Project Organization**: Splitting code into logical modules (models, views, controllers)
- **Code Reuse**: Sharing utility functions across files
- **Testing**: Importing test utilities and fixtures
- **Plugin Systems**: Dynamically loading modules at runtime
- **Configuration**: Separating configuration into its own module
- **API Wrappers**: Creating module-level API client wrappers
- **Feature Toggles**: Conditional imports for optional features

## Common Mistakes

```python
# Mistake 1: Circular imports
# file_a.py: from file_b import func_b
# file_b.py: from file_a import func_a
# Solution: Restructure code, use late imports, or move to common module

# Mistake 2: Import * (pollutes namespace)
from math import *  # Imports all names, can override existing
# sin = "my variable"  # Now shadows math.sin
# Better:
# import math  # math.sin is clear

# Mistake 3: Module-level side effects
# bad_module.py:
# print("Module loaded!")  # Runs on import!
# Better to put in if __name__ == "__main__":

# Mistake 4: Assuming import order
# Don't rely on side effects of module import order

# Mistake 5: ImportError not handled
try:
    import numpy as np
except ImportError:
    print("NumPy not installed. Using fallback.")
    # Provide fallback implementation

# Mistake 6: Modifying another module's attributes
import math
# Don't do this:
# math.pi = 3  # Modifies module, affects all users!
```

## Best Practices

- Import at the top of the file, one per line (PEP 8)
- Order imports: standard library, third-party, local
- Use explicit imports: `from module import name` not `import *`
- Use `import module` when using many names from it
- Use `if __name__ == "__main__":` for script functionality
- Handle ImportError for optional dependencies gracefully
- Avoid circular imports (restructure to prevent)
- Keep modules focused on a single responsibility
- Use relative imports within packages: `from . import sibling`
- Use absolute imports for clarity in most cases
- Document modules with docstrings

## Interview Questions

1. What is a module in Python?
2. What is the difference between `import module` and `from module import name`?
3. What does `if __name__ == "__main__":` do?
4. How does Python find modules (sys.path)?
5. What is the `__init__.py` file used for?
6. What are circular imports and how do you avoid them?
7. How do you reload a module in Python?
8. What is the difference between a module and a package?
9. How do you import modules dynamically?
10. What happens when a module is imported multiple times?

## Coding Challenges

```python
# Challenge 1: Create a simple module with utilities
# Save as string_utils.py:
"""
String utility functions.
"""
def reverse(s):
    return s[::-1]

def count_vowels(s):
    vowels = set("aeiouAEIOU")
    return sum(1 for c in s if c in vowels)

def is_palindrome(s):
    cleaned = ''.join(c.lower() for c in s if c.isalnum())
    return cleaned == cleaned[::-1]

# Usage:
# from string_utils import reverse, count_vowels, is_palindrome
# print(reverse("hello"))  # olleh

# Challenge 2: Import and measure module performance
import time
import importlib

def benchmark_import(module_name, iterations=1000):
    """Benchmark how long it takes to import a module."""
    start = time.perf_counter()
    for _ in range(iterations):
        importlib.import_module(module_name)
    end = time.perf_counter()
    avg_time = (end - start) / iterations * 1000  # ms
    print(f"{module_name}: {avg_time:.4f}ms per import (avg of {iterations})")

benchmark_import("math")
benchmark_import("json")
benchmark_import("os")

# Challenge 3: Module dependency analyzer
def analyze_module(module_name):
    """Analyze a module's dependencies and attributes."""
    import importlib
    try:
        mod = importlib.import_module(module_name)
    except ImportError:
        return {"error": f"Cannot import {module_name}"}
    
    result = {
        "name": module_name,
        "file": getattr(mod, "__file__", "N/A"),
        "doc": mod.__doc__[:100] if mod.__doc__ else "No docstring",
        "public_attributes": [x for x in dir(mod) if not x.startswith("_")],
        "functions": [],
        "classes": [],
    }
    
    for name in result["public_attributes"]:
        obj = getattr(mod, name)
        if callable(obj):
            if isinstance(obj, type):
                result["classes"].append(name)
            else:
                result["functions"].append(name)
    
    return result

analysis = analyze_module("json")
print(f"Module: {analysis['name']}")
print(f"Functions: {analysis['functions'][:5]}")
print(f"Classes: {analysis['classes']}")

# Challenge 4: Safe import with fallback
def safe_import(module_name, fallback_func=None):
    """Import a module with optional fallback."""
    try:
        return __import__(module_name)
    except ImportError:
        if fallback_func:
            return fallback_func()
        print(f"Warning: {module_name} not available")
        return None

# Challenge 5: Plugin loader
def load_plugins(plugin_names):
    """Dynamically load plugins from a list of module names."""
    plugins = {}
    for name in plugin_names:
        try:
            module = importlib.import_module(f"plugins.{name}")
            plugins[name] = module
            print(f"Loaded plugin: {name}")
        except ImportError as e:
            print(f"Failed to load plugin {name}: {e}")
    return plugins

# Example plugin module structure:
# plugins/
#   __init__.py
#   plugin1.py  (should have a run() function)
#   plugin2.py
```

## Summary

Modules are Python files that organize code, prevent naming conflicts, and enable code reuse. Python searches for modules in `sys.path` directories. The `if __name__ == "__main__":` pattern allows files to be both used as modules and run as scripts. Dynamic imports via `importlib` enable flexible, runtime module loading. Proper module organization is essential for maintainable Python projects.

## Related Topics

- Packages
- Import System
- sys.path
- __name__ == "__main__"
- importlib
- Namespace and Scope
- PIP and Package Management
- Virtual Environments
