# Packages - __init__.py, subpackages, relative imports

## __init__.py

### What It Is
`__init__.py` is a special Python file that marks a directory as a Python package. When Python imports a package, it executes the `__init__.py` file inside that directory. This file can be empty or contain initialization code, import statements, and package metadata. Without this file (or a namespace package mechanism), Python will not treat a directory as a regular package.

### Why It Is Important
`__init__.py` is the entry point for a package. It controls what gets exposed when a user imports the package, allows package-level initialization (e.g., setting up configuration, logging, or database connections), defines `__all__` to restrict star imports, and stores package metadata like `__version__` and `__author__`. Properly designed `__init__.py` files create clean public APIs that hide internal module structure from consumers.

### How It Works Internally
When Python encounters `import mypackage`, it searches `sys.path` for a directory named `mypackage`. If found, Python looks for `mypackage/__init__.py`. If the file exists, Python executes it in the context of the new package namespace. The file's global namespace becomes the package's namespace. Python also sets `__path__` on the package to `['path/to/mypackage']`, which is used for subpackage and submodule resolution. If `__init__.py` is absent (Python 3.3+), a namespace package is created instead, where multiple directories can contribute to the same package.

### Syntax
```python
# Minimal __init__.py (empty file)

# Package metadata
__version__ = "2.1.0"
__author__ = "Alice Smith"

# Selective imports from submodules
from .math_ops import add, multiply
from .string_ops import capitalize

# Control star imports
__all__ = ["add", "multiply", "capitalize"]

# Package-level initialization
import logging
logging.getLogger(__name__).addHandler(logging.NullHandler())
```

### Beginner Examples
```python
# Directory: mypackage/__init__.py
"""MyPackage - mathematical and string utilities."""

__version__ = "1.0.0"

# Import key functions to top-level
from .basic import add, subtract
from .strings import reverse
```

```python
# mypackage/basic.py
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b
```

```python
# main.py
import mypackage

print(mypackage.add(5, 3))        # 8
print(mypackage.__version__)      # 1.0.0
```

### Intermediate Examples
```python
# __init__.py with conditional imports
import sys

if sys.platform == "win32":
    from ._win_impl import FileHandler
else:
    from ._posix_impl import FileHandler

__all__ = ["FileHandler"]
```

```python
# Lazy loading in __init__.py
class _LazyImporter:
    def __init__(self, module_name, package):
        self._module_name = module_name
        self._package = package
        self._module = None

    def __getattr__(self, name):
        if self._module is None:
            import importlib
            self._module = importlib.import_module(
                f".{self._module_name}", self._package
            )
        return getattr(self._module, name)

# Usage: package.users.get_user(1) auto-imports .users
import sys
sys.modules[__name__] = _LazyImporter(__name__, __package__ or __name__)
```

### Advanced Examples
```python
# Dynamic __all__ generation
import pkgutil
import importlib

_all_modules = []
__all__ = []

for importer, modname, ispkg in pkgutil.iter_modules(__path__):
    if not modname.startswith("_"):
        module = importlib.import_module(f".{modname}", __package__)
        public_names = [n for n in dir(module) if not n.startswith("_")]
        globals().update({n: getattr(module, n) for n in public_names})
        __all__.extend(public_names)
```

```python
# __init__.py with version from single source
from pathlib import Path

def _read_version():
    version_file = Path(__file__).parent / "VERSION"
    return version_file.read_text().strip()

__version__ = _read_version()
```

### Real-World Use Cases
- **Exposing public API**: Libraries like NumPy use `__init__.py` to expose key functions at the top level (`numpy.array`, `numpy.zeros`).
- **Plugin registration**: Web frameworks scan `__init__.py` to discover installed plugins.
- **Configuration loading**: Django's `apps.py` equivalent in `__init__.py` for app configuration.
- **Backward compatibility**: Deprecated functions are re-exported from `__init__.py` before removal.

### Common Mistakes
```python
# Mistake: Importing everything in __init__.py
from .module1 import *
from .module2 import *
# Can cause name collisions and slow imports

# Mistake: Side effects in __init__.py
print("Loading package...")  # Runs on every import
import os
os.chdir("/some/path")       # Side effect changes global state

# Mistake: Circular imports through __init__.py
# a_pkg/__init__.py: from . import b_module  # if b_module imports a_pkg
# b_module.py:       from a_pkg import something

# Mistake: Forgetting __all__ with star imports
# Without __all__, all non-underscore names are exported
```

### Best Practices
- Keep `__init__.py` minimal; defer heavy imports to submodules.
- Use `__all__` to explicitly define the public API.
- Store version in a single source (`__version__`).
- Avoid side effects at import time.
- Use relative imports within `__init__.py`.
- Import lazily for expensive dependencies.

### Performance Considerations
- Every function imported in `__init__.py` increases package import time.
- Use lazy imports for heavy dependencies like Pandas or TensorFlow.
- Star imports (`from .x import *`) can slow startup; prefer explicit imports.
- `__init__.py` executes once per interpreter session and is cached in `sys.modules`.

### Interview Questions
1. What happens if you remove `__init__.py` from a package directory?
2. How do you prevent `from package import *` from exporting private names?
3. What is the difference between `__all__` in `__init__.py` vs a module?
4. How do you implement lazy loading in `__init__.py`?
5. What is a namespace package and when would you use it?

### Coding Challenges
```python
# Challenge: Build an __init__.py that auto-discovers and registers plugins
# plugins/__init__.py
import pkgutil
import importlib

PLUGIN_REGISTRY = {}

for importer, modname, ispkg in pkgutil.iter_modules(__path__):
    if modname.startswith("plugin_"):
        module = importlib.import_module(f".{modname}", __package__)
        if hasattr(module, "register"):
            PLUGIN_REGISTRY[modname] = module.register()

def get_plugin(name):
    return PLUGIN_REGISTRY.get(name)

__all__ = ["PLUGIN_REGISTRY", "get_plugin"]
```

### Related Topics
- Modules
- Namespace Packages
- Import System
- `__path__` attribute

---

## Subpackages

### What It Is
Subpackages are packages nested inside another package. They allow organizing code into hierarchical namespaces. A subpackage is simply a subdirectory containing its own `__init__.py` and module files. The parent package's namespace scopes the subpackage, accessed via dot notation (e.g., `package.subpackage.module`).

### Why It Is Important
Subpackages enable logical separation of concerns within large projects. A web framework might have subpackages for models, views, templates, and middleware. Subpackages prevent namespace pollution, allow independent versioning of subsystems, and mirror domain hierarchies directly in code structure. They are essential for any non-trivial Python project.

### How It Works Internally
When Python encounters `import package.subpackage`, it first imports the top-level `package` (executing `package/__init__.py`), then looks for `subpackage` in `package.__path__`. Python checks for `package/subpackage/__init__.py`. If found, it executes that file and assigns the resulting module to `package.subpackage` in `sys.modules`. The parent package's `__path__` attribute determines where subpackages are searched, which can be extended to support distributed packages.

### Syntax
```python
# Package structure:
# myapp/
#     __init__.py
#     models/
#         __init__.py
#         user.py
#     views/
#         __init__.py
#         user_view.py

# Import subpackage module
from myapp.models.user import User
import myapp.views.user_view

# Import subpackage itself
from myapp import models
models.user.User("Alice")
```

### Beginner Examples
```python
# Directory layout:
# ecommerce/
#     __init__.py          # version, main API
#     products/
#         __init__.py       # Product, Category
#         product.py
#         category.py
#     orders/
#         __init__.py       # Order, LineItem
#         order.py
#     payments/
#         __init__.py       # PaymentProcessor
#         stripe.py
```

```python
# ecommerce/products/product.py
class Product:
    def __init__(self, sku, name, price):
        self.sku = sku
        self.name = name
        self.price = price
```

```python
# main.py
from ecommerce.products.product import Product
from ecommerce.orders.order import Order

p = Product("ABC-123", "Widget", 9.99)
order = Order()
order.add_item(p)
```

### Intermediate Examples
```python
# Subpackage initialization pattern
# ecommerce/products/__init__.py
from .product import Product
from .category import Category

__all__ = ["Product", "Category"]
```

```python
# Cross-subpackage references
# ecommerce/orders/order.py
from ecommerce.products.product import Product
from ecommerce.payments.stripe import StripeProcessor

class Order:
    def __init__(self):
        self.items = []
        self.processor = StripeProcessor()

    def add_item(self, product: Product):
        self.items.append(product)

    def checkout(self, token):
        total = sum(item.price for item in self.items)
        return self.processor.charge(total, token)
```

### Advanced Examples
```python
# Dynamic subpackage discovery
import pkgutil
import importlib

def discover_services(base_package):
    """Discover all service subpackages."""
    package = importlib.import_module(base_package)
    services = {}
    for importer, modname, ispkg in pkgutil.walk_packages(
        package.__path__, prefix=f"{base_package}."
    ):
        if ispkg:
            module = importlib.import_module(modname)
            services[modname] = module
    return services

# Usage: discover_services("myapp.services")
```

```python
# Subpackage with shared configuration
# config/__init__.py
import os

class Config:
    DEBUG = os.getenv("DEBUG", "false").lower() == "true"
    DATABASE_URL = os.getenv("DATABASE_URL", "sqlite:///dev.db")
    SECRET_KEY = os.getenv("SECRET_KEY", "dev-secret")

# config/dev.py
from . import Config
Config.DEBUG = True
Config.DATABASE_URL = "sqlite:///dev.db"

# config/prod.py
from . import Config
Config.DEBUG = False
```

### Real-World Use Cases
- **Django apps**: Each Django app is a subpackage under the project with `models/`, `views/`, `templates/`, `migrations/` subpackages.
- **Flask blueprints**: Blueprints are organized as subpackages with templates and static files.
- **Machine learning pipelines**: `pipeline/` with `data/`, `features/`, `models/`, `evaluation/` subpackages.
- **CLI tools**: `cli/` with `commands/`, `parsers/`, `formatters/` subpackages.

### Common Mistakes
```python
# Mistake: Deep nesting (more than 3-4 levels)
# package/sub_pkg/sub_sub_pkg/deep/too_deep.py

# Mistake: Circular imports between subpackages
# products/__init__.py: from orders import Order
# orders/__init__.py:   from products import Product

# Mistake: Missing __init__.py in subpackage (pre-3.3)
# Python 3.3+ works, but tools like mypy may not find it

# Mistake: Importing with incorrect dot notation
# import subpackage.module  # Wrong - need full path from top-level
```

### Best Practices
- Limit nesting to 3 levels max; refactor deep hierarchies.
- Use explicit relative imports within the same top-level package.
- Keep subpackage `__init__.py` files minimal.
- Use subpackages for distinct functional domains.
- Avoid circular dependencies between sibling subpackages; extract shared code to a common subpackage.
- Document subpackage boundaries clearly.

### Performance Considerations
- Deeply nested subpackages increase import time due to multiple `__init__.py` executions.
- Each dot in an import path triggers additional filesystem lookups.
- Cache frequently used subpackage references locally to avoid repeated attribute lookups.
- Consider `import myapp.models.user` vs `from myapp.models import user` — the latter saves one attribute access.

### Interview Questions
1. How do subpackages differ from modules in terms of import resolution?
2. What is the maximum recommended depth for package nesting?
3. How does Python locate a subpackage at the filesystem level?
4. How can two sibling subpackages reference each other without circular imports?
5. What role does `__path__` play in subpackage resolution?

### Coding Challenges
```python
# Challenge: Build a subpackage-based plugin architecture
# plugins/__init__.py
import pkgutil
import importlib

PLUGIN_REGISTRY = {}

def load_plugins():
    for importer, name, ispkg in pkgutil.iter_modules(__path__):
        if ispkg and not name.startswith("_"):
            module = importlib.import_module(f".{name}", __package__)
            if hasattr(module, "Plugin"):
                PLUGIN_REGISTRY[name] = module.Plugin()

# Now any subpackage under plugins/ with a Plugin class is auto-discovered
```

### Related Topics
- `__init__.py`
- Modules
- Namespace Packages
- Package Distribution

---

## Relative Imports

### What It Is
Relative imports allow modules within a package to import other modules using relative paths based on the current module's position in the package hierarchy. They use dot notation: a single dot (`.`) refers to the current package, two dots (`..`) refers to the parent package, and so on. Relative imports are evaluated based on the `__package__` and `__name__` attributes of the importing module.

### Why It Is Important
Relative imports make intra-package references resilient to package renaming. If you use absolute imports like `from mypackage.utils import helper`, renaming `mypackage` to `myapp` requires changes everywhere. Relative imports like `from .utils import helper` work regardless of the top-level package name. They also make code more self-documenting by showing the relationship between modules within the package.

### How It Works Internally
When Python encounters `from .submodule import func`, it resolves the relative import using the `__package__` attribute of the importing module. `__package__` is set to the package name (e.g., `mypackage.subpkg`). Python splits this, removes dots based on the level (`.` removes 0, `..` removes 1), constructs the absolute module name, and looks it up in `sys.modules` or imports it. Relative imports only work inside packages; running a module directly (`python module.py`) sets `__package__` to `None`, causing relative imports to raise `ImportError`.

### Syntax
```python
# Package structure:
# mypackage/
#     __init__.py
#     utils.py
#     module.py
#     subpkg/
#         __init__.py
#         helper.py
#         worker.py

# mypackage/module.py
from .utils import helper_func        # sibling module
from .subpkg.helper import run        # nested subpackage

# mypackage/subpkg/worker.py
from .helper import run               # same subpackage
from ..utils import helper_func       # parent package
from ..subpkg.helper import run       # sibling subpackage (via parent)
```

### Beginner Examples
```python
# mypackage/operations.py
def add(a, b):
    return a + b
```

```python
# mypackage/calculator.py
from .operations import add  # Relative import of sibling

def calculate_total(items):
    return sum(add(item.tax, item.price) for item in items)
```

```python
# mypackage/__init__.py
from .calculator import calculate_total
from .operations import add

__all__ = ["calculate_total", "add"]
```

### Intermediate Examples
```python
# mypackage/models/__init__.py
from .user import User
from .product import Product
```

```python
# mypackage/views/user_view.py
from ..models.user import User  # Go up one level, then into models

def format_user(user: User) -> dict:
    return {"id": user.id, "name": user.name}
```

```python
# mypackage/services/auth.py
from ..models.user import User           # Parent -> sibling subpackage
from .base import BaseService            # Same subpackage
from .. import __version__               # Parent package's __init__
```

### Advanced Examples
```python
# Complex relative imports with shared utilities
# myapp/
#     __init__.py
#     utils/
#         __init__.py
#         db.py
#         cache.py
#     services/
#         __init__.py
#         user_service.py
#         order_service.py
#     api/
#         __init__.py
#         v1/
#             __init__.py
#             users.py
#             orders.py

# myapp/api/v1/users.py
from ...services.user_service import UserService  # Up 3 levels
from ...utils.db import get_session               # Up 3 levels
from .. import api_config                         # Up 2 levels (api/__init__)
from . import schema                              # Same package
```

```python
# Dynamic relative import
import importlib

def import_sibling(name):
    """Import a sibling module using the caller's package."""
    import sys
    caller_frame = sys._getframe(1)
    package = caller_frame.f_globals.get("__package__")
    if not package:
        raise ImportError("Not inside a package")
    module_name = f".{name}"
    return importlib.import_module(module_name, package)
```

### Real-World Use Cases
- **Framework internals**: Django, Flask, and SQLAlchemy use relative imports extensively within their own codebases.
- **Microservice projects**: Services organized in subpackages use relative imports to share models and utilities.
- **Testing**: Test files mirror source structure and use relative imports for their corresponding modules.
- **Library development**: Library authors use relative imports so consumers can install under any package name.

### Common Mistakes
```python
# Mistake: Using relative imports in scripts run directly
# main.py (at project root)
# from .utils import helper  # ImportError: attempted relative import with no known parent package

# Correct: Run with -m flag
# python -m mypackage.main

# Mistake: Going too many levels up
# from ....module import x  # Fragile, confusing

# Mistake: Mixing relative and absolute in the same package
# from .utils import a    # relative
# from mypackage import b # absolute (breaks if package renamed)

# Mistake: Forgetting dot before module name
# from utils import helper  # absolute import, not relative!
```

### Best Practices
- Use relative imports for all intra-package references.
- Limit to at most 2-3 levels up (`..` or `...`); deeper suggests poor structure.
- Prefer `from . import sibling` over `from .sibling import name` for clarity.
- Never mix relative and absolute imports for the same package.
- Run package code with `python -m` to ensure relative imports resolve correctly.
- Use explicit relative imports in `__init__.py` (`from .submodule import name`).

### Performance Considerations
- Relative imports have negligible overhead compared to absolute imports.
- Python resolves relative imports at module load time, so there is no runtime penalty.
- Deeply nested relative imports (4+ levels) hint at overly deep package hierarchies that slow overall import times.
- Use `from . import mod` (one attribute lookup) vs `from .mod import func` (two attribute lookups) when importing many names.

### Interview Questions
1. What is the difference between `from .module import func` and `from module import func`?
2. Why do relative imports fail when running a script directly with `python script.py`?
3. What does the `__package__` attribute do in relation to relative imports?
4. How many dots would you use to import from a grandparent package?
5. What happens with relative imports inside `__main__.py`?

### Coding Challenges
```python
# Challenge: Write a utility that resolves any relative import path to its absolute form
def resolve_relative_import(importer_module, relative_path):
    """Convert relative import path to absolute module name.
    
    resolve_relative_import("package.sub.module", "..sibling.child")
    -> "package.sibling.child"
    """
    package = sys.modules[importer_module].__package__
    if not package:
        raise ValueError(f"No package context for {importer_module}")
    
    parts = package.split(".")
    relative_parts = relative_path.split(".")
    
    level = 0
    for part in relative_parts:
        if part == "":
            level += 1
        else:
            break
    
    if level > len(parts):
        raise ValueError(f"Relative import goes beyond top-level package")
    
    base = parts[:len(parts) - level]
    base.extend(p for p in relative_parts[level:] if p)
    return ".".join(base)

# Test
import sys
sys.modules["package.sub.module"] = type(sys)("package.sub.module")
sys.modules["package.sub.module"].__package__ = "package.sub"
print(resolve_relative_import("package.sub.module", "..sibling.child"))
# Output: "package.sibling.child"
```

### Related Topics
- `__package__` Attribute
- `__name__` Attribute
- Absolute Imports
- Import System
- `python -m` Flag
