# Packages - __init__.py, subpackages, relative imports

## Introduction

Packages are a way of organizing related modules into a directory hierarchy. A package is simply a directory containing a special `__init__.py` file (can be empty) and one or more module files. Packages can be nested (subpackages) and allow for a hierarchical structure of modules. Python also supports namespace packages (without `__init__.py`) and has package management through pip and PyPI.

## Why It Is Important

Packages are important because:
- They organize related modules into logical groups
- They prevent naming conflicts between modules
- They enable large project organization
- They support subpackages for deep hierarchies
- They allow distribution of reusable code via PyPI
- They enable relative imports within package hierarchies

## Syntax

```python
# Import from package
import package.module
from package import module
from package.module import function

# Import subpackage
import package.subpackage.module
from package.subpackage import module

# Relative imports (inside package)
from . import sibling_module
from .. import parent_module
from .subpackage import module

# Package structure:
# mypackage/
#     __init__.py
#     module1.py
#     module2.py
#     subpackage/
#         __init__.py
#         module3.py
```

## Examples

### Creating a Package

```python
# Directory structure:
# mypackage/
#     __init__.py
#     math_ops.py
#     string_ops.py
#     data_ops.py
```

```python
# __init__.py - can be empty or contain package initialization
"""
MyPackage - A collection of utility functions.
"""

__version__ = "1.0.0"
__author__ = "Your Name"

# Import key functions for easy access
from .math_ops import add, subtract, multiply, divide
from .string_ops import reverse, capitalize_words
from .data_ops import filter_none, unique

# Define what gets imported with "from package import *"
__all__ = ["add", "subtract", "multiply", "divide", 
           "reverse", "capitalize_words", "filter_none", "unique"]
```

```python
# math_ops.py
"""Mathematical operations."""

def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def multiply(a, b):
    return a * b

def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b
```

```python
# string_ops.py
"""String operations."""

def reverse(s):
    return s[::-1]

def capitalize_words(s):
    return " ".join(word.capitalize() for word in s.split())

def count_words(s):
    return len(s.split())
```

### Using a Package

```python
# Various ways to import from a package

# Method 1: Import the package, access modules
import mypackage
result = mypackage.math_ops.add(5, 3)
print(f"5 + 3 = {result}")

# Method 2: Import specific module
from mypackage import math_ops
print(f"10 * 5 = {math_ops.multiply(10, 5)}")

# Method 3: Import specific function
from mypackage.math_ops import divide
print(f"100 / 7 = {divide(100, 7):.2f}")

# Method 4: Using __init__.py imports
from mypackage import add, reverse
print(f"add: {add(10, 20)}")
print(f"reverse: {reverse('hello')}")

# Accessing package metadata
import mypackage
print(f"Package version: {mypackage.__version__}")
print(f"Package author: {mypackage.__author__}")
```

## Beginner Examples

### Package Structure Examples

```python
# Simple package structure:
# project/
#     main.py
#     utils/
#         __init__.py
#         file_utils.py
#         string_utils.py
#         math_utils.py
```

```python
# utils/__init__.py
"""Utility package initialization."""
print("Initializing utils package")

# Can do package-level setup here
# Example: load configuration
import os
DEBUG = os.environ.get("DEBUG", "False").lower() == "true"

# Selective export
from .string_utils import truncate, format_name
```

```python
# utils/string_utils.py
"""String utility functions."""

def truncate(text, max_length, suffix="..."):
    if len(text) <= max_length:
        return text
    return text[:max_length - len(suffix)] + suffix

def format_name(first, last):
    return f"{first.capitalize()} {last.capitalize()}"

def to_slug(text):
    import re
    text = text.lower().strip()
    return re.sub(r'[^a-z0-9]+', '-', text)
```

```python
# main.py using the package
from utils import truncate, format_name
from utils.file_utils import read_file

long_text = "This is a very long sentence that needs truncation"
print(truncate(long_text, 20))  # This is a very lon...

print(format_name("alice", "smith"))  # Alice Smith
```

### Subpackages

```python
# Deep package structure:
# app/
#     __init__.py
#     models/
#         __init__.py
#         user.py
#         product.py
#     views/
#         __init__.py
#         user_views.py
#         product_views.py
#     services/
#         __init__.py
#         auth_service.py
#         payment_service.py
```

```python
# Importing from subpackages
from app.models.user import User
from app.views.user_views import UserView
from app.services.auth_service import authenticate

# Or import the module
from app.models import user
new_user = user.User("Alice", "alice@example.com")
```

## Intermediate Examples

### Relative Imports

```python
# Inside a package, use relative imports
# Structure:
# mypackage/
#     __init__.py
#     utils.py
#     helpers.py
#     subpackage/
#         __init__.py
#         module.py
```

```python
# mypackage/utils.py
from .helpers import format_output  # Relative import (sibling)

def process_data(data):
    return format_output(data)
```

```python
# mypackage/subpackage/module.py
from ..utils import process_data  # Relative import (parent)
from . import sibling_module       # Relative import (same package)

def run():
    data = process_data("test")
    print(data)
```

### Package Initialization

```python
# Advanced __init__.py patterns

# __init__.py with lazy loading
class LazyLoader:
    """Lazy load modules when first accessed."""
    def __init__(self):
        self._loaded = {}
    
    def __getattr__(self, name):
        if name not in self._loaded:
            import importlib
            self._loaded[name] = importlib.import_module(f".{name}", __package__)
        return self._loaded[name]

import sys
sys.modules[__name__] = LazyLoader()
```

### Namespace Packages

```python
# Namespace packages (no __init__.py)
# Directory structure:
# project/
#     namespace_pkg/
#         module_a.py
#     another_location/
#         namespace_pkg/
#             module_b.py

# Both directories contribute to the same namespace_pkg
# import namespace_pkg.module_a
# import namespace_pkg.module_b

# Requires both directories to be in sys.path
```

## Advanced Examples

### Package as Executable

```python
# Making a package runnable with python -m
# Add __main__.py to the package:
```

```python
# mypackage/__main__.py
"""
Allows running the package with: python -m mypackage
"""
import sys
from . import __version__

def main():
    print(f"MyPackage v{__version__}")
    print("Running package as script...")
    # Package logic here

if __name__ == "__main__":
    main()
```

```python
# Now you can run:
# python -m mypackage
# Output: MyPackage v1.0.0
# Output: Running package as script
```

### Package Data and Resources

```python
# Including non-code files in packages
# Structure:
# mypackage/
#     __init__.py
#     data/
#         config.json
#         template.txt

import os
import json

def load_config():
    """Load package data file."""
    module_dir = os.path.dirname(__file__)
    config_path = os.path.join(module_dir, "data", "config.json")
    with open(config_path, "r") as f:
        return json.load(f)

# Better: use importlib.resources (Python 3.7+)
try:
    import importlib.resources as resources
    def load_template():
        return resources.read_text(__package__, "data/template.txt")
except ImportError:
    # Fallback for older Python
    pass
```

### Conditional Imports in Packages

```python
# Package with optional dependencies
# database/__init__.py

try:
    import pymongo
    HAS_MONGO = True
except ImportError:
    HAS_MONGO = False

try:
    import sqlalchemy
    HAS_SQL = True
except ImportError:
    HAS_SQL = False

class Database:
    """Database abstraction with optional backends."""
    
    @classmethod
    def create_mongo(cls, connection_string):
        if not HAS_MONGO:
            raise ImportError("pymongo is required for MongoDB")
        return MongoDatabase(connection_string)
    
    @classmethod
    def create_sql(cls, connection_string):
        if not HAS_SQL:
            raise ImportError("sqlalchemy is required for SQL")
        return SQLDatabase(connection_string)
```

## Real-World Use Cases

- **Web Frameworks**: Django, Flask, FastAPI organize code in packages
- **Data Science Libraries**: NumPy, Pandas, scikit-learn as packages
- **Project Structure**: Organizing large codebases with MVC pattern
- **Plugin Systems**: Dynamic discovery of plugins via packages
- **Shared Libraries**: Distributing reusable code via PyPI
- **Microservices**: Organizing service code as packages
- **Testing**: Test packages mirroring source structure

## Common Mistakes

```python
# Mistake 1: Missing __init__.py in older Python
# Python 3.3+ has namespace packages (works without __init__.py)
# But __init__.py is still recommended for regular packages

# Mistake 2: Circular imports between packages
# package_a/__init__.py: from package_b import something
# package_b/__init__.py: from package_a import something
# Solution: move shared code to a common module

# Mistake 3: Incorrect relative imports
# from ..module import func  # Only works inside a package
# Running script directly: can't use relative imports
# Use: python -m package.module instead

# Mistake 4: Name conflicts
# Don't name your package the same as a standard library module
# e.g., "email", "json", "math" - will shadow standard library!

# Mistake 5: Forgetting __all__ in __init__.py
# Without __all__, "from package import *" imports all non-private names
# Set __all__ to control what gets exported

# Mistake 6: Side effects in __init__.py
# Don't put code with side effects in package initialization
# It runs on every import!
```

## Best Practices

- Use `__init__.py` to expose a clean public API
- Use `__all__` to control what "from package import *" exports
- Use relative imports within packages
- Keep `__init__.py` minimal (no heavy imports)
- Use meaningful package names (lowercase, no underscores)
- Follow the principle "one package = one responsibility"
- Use subpackages for large, complex packages
- Document the package with docstrings in `__init__.py`
- Specify version in `__version__` variable
- Use `setup.py` or `pyproject.toml` for distributable packages
- Avoid circular dependencies between packages

## Interview Questions

1. What is the difference between a module and a package?
2. What is the purpose of `__init__.py`?
3. What are namespace packages?
4. How do relative imports work in packages?
5. What is `__all__` and how is it used?
6. How do you make a package executable with `python -m`?
7. How do you include data files in a package?
8. What is `__main__.py`?
9. How do you organize a large project using packages?
10. What is the difference between `from package import module` and `from package.module import function`?

## Coding Challenges

```python
# Challenge 1: Create a calculator package
# calculator/
#     __init__.py
#     basic.py
#     advanced.py
#     __main__.py

# calculator/__init__.py
from .basic import add, subtract, multiply, divide
from .advanced import power, sqrt, factorial

__all__ = ["add", "subtract", "multiply", "divide", "power", "sqrt", "factorial"]

# calculator/basic.py
def add(a, b): return a + b
def subtract(a, b): return a - b
def multiply(a, b): return a * b
def divide(a, b):
    if b == 0:
        raise ValueError("Division by zero")
    return a / b

# calculator/advanced.py
import math
def power(a, b): return a ** b
def sqrt(a): return math.sqrt(a)
def factorial(n): return math.factorial(n)

# calculator/__main__.py
from . import add, subtract
import sys
def main():
    if len(sys.argv) == 3:
        a, b = map(float, sys.argv[1:])
        print(f"Add: {add(a, b)}")
        print(f"Subtract: {subtract(a, b)}")
if __name__ == "__main__":
    main()

# Challenge 2: Package introspection
def package_info(package_name):
    """Get detailed information about a package."""
    import importlib
    try:
        mod = importlib.import_module(package_name)
    except ImportError:
        return {"error": f"Package {package_name} not found"}
    
    info = {
        "name": package_name,
        "file": getattr(mod, "__file__", "N/A"),
        "version": getattr(mod, "__version__", "Unknown"),
        "doc": (mod.__doc__ or "No docstring")[:200],
        "submodules": [],
    }
    
    import os
    package_dir = os.path.dirname(mod.__file__)
    for item in os.listdir(package_dir):
        if item.endswith(".py") and not item.startswith("_"):
            info["submodules"].append(item[:-3])
        elif os.path.isdir(os.path.join(package_dir, item)) and not item.startswith("_"):
            if os.path.exists(os.path.join(package_dir, item, "__init__.py")):
                info["submodules"].append(f"{item}/")
    
    return info

info = package_info("mypackage")
for key, value in info.items():
    print(f"{key}: {value}")

# Challenge 3: Plugin discovery system
def discover_plugins(package_name):
    """Discover all plugins in a package."""
    import importlib
    import pkgutil
    
    plugins = {}
    package = importlib.import_module(package_name)
    package_path = package.__path__
    
    for importer, modname, ispkg in pkgutil.iter_modules(package_path):
        if modname.startswith("plugin_"):
            module = importlib.import_module(f"{package_name}.{modname}")
            if hasattr(module, "run"):
                plugins[modname] = module.run
    
    return plugins

# Assuming plugins package structure:
# plugins/
#     __init__.py
#     plugin_email.py  (has run() function)
#     plugin_sms.py    (has run() function)

# Challenge 4: Package dependency tree
def dependency_tree(package_name, depth=0, max_depth=3):
    """Build a dependency tree for a package."""
    if depth > max_depth:
        return
    
    try:
        module = __import__(package_name)
        print("  " * depth + f"├── {package_name}")
        
        if hasattr(module, "__path__"):
            import pkgutil
            for importer, name, ispkg in pkgutil.iter_modules(module.__path__):
                if not name.startswith("_"):
                    full_name = f"{package_name}.{name}"
                    if ispkg:
                        dependency_tree(full_name, depth + 1, max_depth)
                    else:
                        print("  " * (depth + 1) + f"├── {name}")
    except ImportError:
        pass

# Challenge 5: Dynamic package reloader
def reload_package(package_name):
    """Reload a package and all its submodules."""
    import importlib
    import sys
    
    def _reload_recursive(module):
        importlib.reload(module)
        if hasattr(module, "__path__"):
            import pkgutil
            for importer, name, ispkg in pkgutil.iter_modules(module.__path__):
                full_name = f"{module.__name__}.{name}"
                if full_name in sys.modules:
                    _reload_recursive(sys.modules[full_name])
    
    if package_name in sys.modules:
        _reload_recursive(sys.modules[package_name])
        print(f"Reloaded {package_name} and all submodules")
    else:
        print(f"{package_name} is not imported")
```

## Summary

Packages organize related modules into directory hierarchies, with `__init__.py` (can be empty) marking a directory as a package. They support subpackages, relative imports, and namespace packages. The PyPI ecosystem with pip enables distribution and installation of third-party packages. Well-structured packages improve code organization, maintainability, and reusability.

## Related Topics

- Modules
- Import System
- __init__.py
- pip and PyPI
- setup.py / pyproject.toml
- Virtual Environments
- Relative vs Absolute Imports
- Application Structure and Organization
