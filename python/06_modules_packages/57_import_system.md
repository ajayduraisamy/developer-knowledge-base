# Import System - sys.path, import hooks, importlib, lazy loading
## Introduction
Python's import system is one of the most sophisticated and extensible module-loading mechanisms among programming languages. It governs how modules and packages are found, loaded, cached, and executed. Understanding the import system is crucial for debugging import errors, creating plugins, building frameworks, and optimizing startup time.

## sys.path
### What It Is
`sys.path` is a list of strings specifying the search path for modules. When you write `import foo`, Python iterates through `sys.path` directories (and zip archives) looking for `foo.py`, `foo/__init__.py`, or a compiled extension `foo.pyd`/`foo.so`.

### Why It Is Important
Almost all import errors (`ModuleNotFoundError`) are caused by `sys.path` not containing the right directory. Knowing how `sys.path` is constructed and how to modify it is essential for managing module discovery.

### How It Works Internally
`sys.path` is initialized at interpreter startup from (in order):
1. The script's directory (or current directory if running interactively)
2. The `PYTHONPATH` environment variable
3. The site-packages directories (determined at Python installation/build time)
4. `.pth` files in site-packages (for adding extra paths)

The import machinery calls `importlib.machinery.PathFinder.find_module()` (or `find_spec()`) which iterates `sys.path`, trying each entry.

### Syntax
```python
import sys

# View
print(sys.path)

# Append
sys.path.append("/path/to/my/modules")

# Insert at beginning (highest priority)
sys.path.insert(0, "/path/to/custom")

# Remove
sys.path.remove("/unwanted/path")

# Environment variable (bash)
# export PYTHONPATH=/custom/path:$PYTHONPATH
```

### Beginner Examples
```python
# Understanding sys.path
import sys

print("Module search paths:")
for i, path in enumerate(sys.path, 1):
    print(f"{i}. {path}")

# This shows the script directory, standard library, site-packages, etc.

# Adding a path temporarily
sys.path.insert(0, "/home/user/mymodules")
import my_custom_module  # Now it can be found
```

### Intermediate Examples
```python
import sys

# Check where a module will be loaded from
def find_module_location(module_name):
    if module_name in sys.modules:
        mod = sys.modules[module_name]
        return getattr(mod, "__file__", "built-in module")
    
    # Simulate what Python will search
    for path in sys.path:
        import os
        py_file = os.path.join(path, f"{module_name}.py")
        init_file = os.path.join(path, module_name, "__init__.py")
        pkg_dir = os.path.join(path, module_name)
        
        if os.path.isfile(py_file):
            return py_file
        if os.path.isfile(init_file):
            return init_file
    
    return None

print(find_module_location("json"))
print(find_module_location("numpy"))

# Debug import issues
try:
    import non_existent_module
except ImportError as e:
    print(f"Import failed: {e}")
    print(f"Current sys.path: {sys.path[:3]}...")
```

### Advanced Examples
```python
import sys
import os

# Compute site-packages path at runtime
def get_site_packages():
    for path in sys.path:
        if path.endswith("site-packages"):
            return path
    return None

print(f"Site-packages: {get_site_packages()}")

# Add a path relative to the current script
script_dir = os.path.dirname(os.path.abspath(__file__))
project_root = os.path.dirname(script_dir)
sys.path.insert(0, project_root)

# Using __init__.py to manipulate path
# In a package's __init__.py:
# sys.path.insert(0, os.path.dirname(__file__))

# Portable approach for development
def setup_dev_path():
    """Add src/ to sys.path if running from project root."""
    import pathlib
    src = pathlib.Path(__file__).resolve().parent / "src"
    if src.is_dir() and str(src) not in sys.path:
        sys.path.insert(0, str(src))
```

### Real-World Use Cases
- **Development setups** — adding `src/` to sys.path without installing the package
- **Plugin systems** — adding plugin directories to sys.path dynamically
- **Site customization** — using `.pth` files to add paths without modifying PYTHONPATH
- **Embedded Python** — configuring sys.path for embedded interpreters

### Common Mistakes
- Modifying `sys.path` after imports have already failed (too late)
- Using relative paths in sys.path that depend on the current working directory
- Forgetting that `sys.path[0]` is the script directory, not the project root
- Appending instead of inserting when you want priority over other paths

### Best Practices
- Prefer `PYTHONPATH` environment variable over modifying `sys.path` in code
- Use absolute paths when modifying `sys.path` programmatically
- Check if a path already exists in `sys.path` before adding it (avoid duplicates)
- Use context managers for temporary sys.path modifications
- Consider using `.pth` files in site-packages for permanent additions
- For development, install packages in editable mode (`pip install -e .`) instead of sys.path hacks

### Performance Considerations
- `sys.path` is searched sequentially — longer paths mean slower first imports
- Adding many paths slows down module search
- Network-mounted paths in `sys.path` can cause import delays
- `sys.path` modifications do not affect already-loaded modules (in `sys.modules`)

### Interview Questions
1. What is the order of directories searched by `sys.path`?
2. How can you add a directory to `sys.path` permanently?
3. What is the difference between `sys.path` and `sys.meta_path`?
4. How does `PYTHONPATH` affect module imports?

### Coding Challenges
- Write a function that traces which path entry a module was loaded from
- Build a context manager that temporarily adds a path to `sys.path`
- Create a script that visualizes the `sys.path` search order

### Related Topics
- importlib, PYTHONPATH, .pth files, site module, path hooks

## Import hooks
### What It Is
Import hooks are a mechanism in Python's import system that allows customizing or intercepting the module-loading process. Python supports two types: *meta hooks* (registered in `sys.meta_path`) that can intercept any import, and *path hooks* (registered in `sys.path_hooks`) that handle specific `sys.path` entries.

### Why It Is Important
Import hooks enable powerful customization: loading modules from databases, URLs, compressed archives, or custom formats; implementing import-time code transformations (e.g., automatic transpilation); and restricting or auditing imports (security sandboxes).

### How It Works Internally
When `import foo` is executed:
1. Python checks `sys.modules` cache
2. If not found, iterates `sys.meta_path` finders (normally: `BuiltinImporter`, `FrozenImporter`, `PathFinder`)
3. The first finder that returns a module spec wins
4. For `PathFinder`, it iterates `sys.path` entries; for each entry, it checks `sys.path_hooks` for a suitable loader

A finder is any object with `find_spec(fullname, path, target=None)` method (Python 3.4+) returning a `ModuleSpec` or `None`.

### Syntax
```python
import sys
from importlib.abc import MetaPathFinder, Loader
from importlib.util import spec_from_loader

# Meta path hook
class MyMetaFinder(MetaPathFinder):
    def find_spec(self, fullname, path, target=None):
        # Return a ModuleSpec or None
        pass

sys.meta_path.insert(0, MyMetaFinder())

# Path hook
class MyPathFinder:
    @classmethod
    def find_spec(cls, fullname, path, target=None):
        pass
```

### Beginner Examples
```python
# Trace all imports
import sys

original_import = __builtins__.__import__

def trace_import(name, *args, **kwargs):
    print(f"Importing: {name}")
    return original_import(name, *args, **kwargs)

__builtins__.__import__ = trace_import

# Now every import will print its name
import os  # prints "Importing: os"
```

### Intermediate Examples
```python
import sys
from importlib.abc import MetaPathFinder, Loader
from importlib.util import spec_from_loader

# A meta path finder that logs all module lookups
class LoggingMetaFinder(MetaPathFinder):
    def find_spec(self, fullname, path, target=None):
        if path is not None:
            print(f"[ImportHook] Looking for {fullname} in path={path}")
        else:
            print(f"[ImportHook] Looking for {fullname} (top-level)")
        return None  # Delegate to other finders

sys.meta_path.insert(0, LoggingMetaFinder())

# Now use it
import json
```

### Advanced Examples
```python
import sys
from importlib.abc import MetaPathFinder, Loader

# Virtual module provider — modules generated on-the-fly
class VirtualModule:
    def __init__(self, name, code):
        self.name = name
        self.code = code

class VirtualLoader(Loader):
    def __init__(self, module):
        self.module = module
    
    def create_module(self, spec):
        return None  # Use default semantics
    
    def exec_module(self, module):
        exec(self.module.code, module.__dict__)

class VirtualMetaFinder(MetaPathFinder):
    def __init__(self):
        self.virtual_modules = {}
    
    def register(self, name, code):
        self.virtual_modules[name] = VirtualModule(name, code)
    
    def find_spec(self, fullname, path, target=None):
        if fullname in self.virtual_modules:
            module = self.virtual_modules[fullname]
            loader = VirtualLoader(module)
            from importlib.util import spec_from_loader
            spec = spec_from_loader(fullname, loader)
            return spec
        return None

# Register the finder
vfinder = VirtualMetaFinder()
sys.meta_path.insert(0, vfinder)

# Create a virtual module
vfinder.register(
    "greeter",
    """
def hello(name):
    return f"Hello, {name}!"
"""
)

# Use it
import greeter
print(greeter.hello("World"))  # Hello, World!
```

```python
# Path hook for JSON modules
import json
import sys
from importlib.abc import PathEntryFinder, Loader

class JsonLoader(Loader):
    def __init__(self, path):
        self.path = path
    
    def create_module(self, spec):
        return None
    
    def exec_module(self, module):
        with open(self.path) as f:
            data = json.load(f)
        module.__dict__.update(data)

class JsonPathFinder(PathEntryFinder):
    @classmethod
    def find_spec(cls, name, path, target=None):
        import os
        for entry in (path or sys.path):
            json_path = os.path.join(entry, f"{name}.json")
            if os.path.isfile(json_path):
                from importlib.util import spec_from_loader
                loader = JsonLoader(json_path)
                return spec_from_loader(name, loader)
        return None

# Register the .json path hook
sys.path_hooks.append(JsonPathFinder)

# Now create config.json and import it
# {"host": "localhost", "port": 8080}
import config
print(config.host)  # localhost
```

### Real-World Use Cases
- **Code hot-reloading** — custom import hooks that reload modules from disk on change
- **Plugin systems** — scanning directories for plugin modules dynamically
- **Lazy imports** — delaying module loading until first attribute access
- **Transpilation** — importing `.pyx` (Cython) or `.ts` (TypeScript) files directly
- **Sandboxing** — restricting which modules can be imported (deny lists)
- **Zip imports** — Python itself uses path hooks for importing from `.zip` files

### Common Mistakes
- Not returning `None` from `find_spec` when not handling a module (breaks the chain)
- Forgetting to insert at position 0 in `sys.meta_path` (your finder gets called last)
- Creating infinite loops by importing inside a finder/loader
- Not implementing `create_module` or `exec_module` correctly in custom loaders

### Best Practices
- Always return `None` from `find_spec` if you don't handle the module
- Use `importlib.abc` base classes for proper interface compliance
- Keep meta finders lightweight — they're called for every import
- Prefer path hooks over meta hooks when the hook is path-specific
- Test hooks edge cases: nested packages, relative imports, namespace packages
- Handle errors gracefully — a broken finder can break all imports

### Performance Considerations
- Meta path finders are called for *every* import — they must be fast
- Path hooks are only evaluated for each `sys.path` entry (less frequent)
- Custom loaders that read from slow sources (network, database) should cache
- Consider using `importlib.invalidate_caches()` when changing hooks at runtime

### Interview Questions
1. What is the difference between a meta path finder and a path hook?
2. How would you implement a module that loads from a database?
3. What does `find_spec` return and how is it used?
4. How does Python resolve imports from `.zip` files?
5. What is the order of finder invocation during import?

### Coding Challenges
- Implement an import hook that logs all imported modules with timestamps
- Build a simple plugin system that discovers and loads plugins from a directory
- Create a meta finder that prevents specific modules from being imported
- Implement a loader that reads modules from a SQLite database

### Related Topics
- importlib, sys.meta_path, sys.path_hooks, importlib.abc, importlib.machinery

## importlib module
### What It Is
`importlib` is the Python module that implements the import statement. It provides the `import_module()` convenience function, the `abc` submodule (abstract base classes for finders/loaders), the `machinery` submodule (concrete implementations), and `resources` for accessing package data files.

### Why It Is Important
`importlib` exposes the import machinery as a public API. This allows programmatic imports (with strings), dynamic module loading, custom importers, resource access, and introspection — all without `exec` or `__import__` hacks.

### How It Works Internally
`importlib` implements the full import protocol described in PEP 302 and PEP 451. At its core is `_bootstrap.py` and `_bootstrap_external.py` — these are frozen into the Python interpreter. The public `importlib` module re-exports key functions and provides building blocks for customization.

### Syntax
```python
import importlib

# Import by string name
module = importlib.import_module("os.path")

# Module utilities
importlib.util.find_spec("json")
importlib.util.module_from_spec(spec)
importlib.reload(module)

# Resources (package data)
importlib.resources.read_text(package, resource_name)
importlib.resources.files(package) / "data.txt"

# ABCs
from importlib.abc import MetaPathFinder, PathEntryFinder, Loader
```

### Beginner Examples
```python
import importlib

# Dynamic import by string
module_name = "json"
json_module = importlib.import_module(module_name)
print(json_module.dumps({"key": "value"}))

# Import submodule
os_path = importlib.import_module("os.path")
print(os_path.join("a", "b"))

# Reload a module after changes
importlib.reload(json_module)

# Check if a module can be imported
spec = importlib.util.find_spec("math")
print(f"Found: {spec.name}, origin: {spec.origin}")
```

### Intermediate Examples
```python
import importlib

# Import all modules in a package
import pkgutil

def import_submodules(package_name):
    package = importlib.import_module(package_name)
    results = {package_name: package}
    
    for importer, name, is_pkg in pkgutil.walk_packages(
        package.__path__, package.__name__ + "."
    ):
        try:
            results[name] = importlib.import_module(name)
        except ImportError as e:
            print(f"Failed to import {name}: {e}")
    
    return results

submodules = import_submodules("os")
print(f"Loaded {len(submodules)} os submodules")

# Dynamic import with fallback
def import_optional(module_name, fallback_value=None):
    try:
        return importlib.import_module(module_name)
    except ImportError:
        return fallback_value

orjson = import_optional("orjson", json)
print(f"Using: {orjson.__name__}")
```

### Advanced Examples
```python
import importlib
from importlib import util
import sys

# Create a module programmatically
def create_virtual_module(name, namespace, package=None):
    spec = util.spec_from_loader(name, None, origin="virtual")
    module = util.module_from_spec(spec)
    module.__dict__.update(namespace)
    if package:
        module.__package__ = package
        module.__path__ = [package]
    sys.modules[name] = module
    return module

# Create a module on the fly
create_virtual_module("config", {
    "DEBUG": True,
    "DATABASE_URL": "sqlite:///app.db",
    "VERSION": "1.0.0",
})

import config
print(config.DEBUG)  # True

# Reload with custom logic
def reload_with_tracking(name):
    if name in sys.modules:
        del sys.modules[name]
    
    import time
    start = time.perf_counter()
    module = importlib.import_module(name)
    elapsed = time.perf_counter() - start
    
    module.__load_time__ = elapsed
    return module

# Subclassing for custom import behavior
class RecordedLoader(importlib.abc.Loader):
    def __init__(self, delegate):
        self.delegate = delegate
        self.load_count = 0
    
    def create_module(self, spec):
        return self.delegate.create_module(spec)
    
    def exec_module(self, module):
        self.load_count += 1
        print(f"[Loader] Loading {module.__name__} (load #{self.load_count})")
        self.delegate.exec_module(module)
```

```python
# importlib.resources (Python 3.7+) — reading package data
from importlib import resources

# Assuming a package structure:
# my_package/
#   __init__.py
#   data/
#     config.json
#     template.html

# Read text resource
try:
    config_text = resources.read_text("my_package", "config.json")
    # Or using files() API (Python 3.9+)
    data = resources.files("my_package") / "data" / "config.json"
    config_text = data.read_text()
except Exception:
    pass

# Binary resources
# resources.read_binary("my_package", "icon.png")
```

### Real-World Use Cases
- **Plugin systems** — lazy-loading plugins by name from configuration
- **Configuration modules** — dynamically importing config files per environment
- **Lazy imports** — using `importlib.import_module` inside functions instead of top-level
- **Hot-reloading** — calling `importlib.reload` during development
- **Package data** — accessing non-Python files bundled with a package
- **Testing/mocking** — swapping `sys.modules` entries to replace real modules with mocks

### Common Mistakes
- Using `__import__` directly instead of `importlib.import_module` (which is higher-level)
- Forgetting that `import_module` returns the *top-level* package for dotted names
- Reloading modules without considering that existing references still point to the old module
- Calling `importlib.reload` on C extension modules (raises `ImportError`)
- Not understanding that `reload` re-executes the module code but doesn't update `from X import Y` style imports in other modules

### Best Practices
- Use `importlib.import_module` for programmatic imports
- Use `importlib.util.find_spec` to check importability without loading
- Use `importlib.reload` sparingly and only in development/debugging
- Use `importlib.resources` for accessing package data files
- Prefer `pkgutil` for iterating package contents
- For Python 3.9+, use `importlib.resources.files()` API (modern)
- Clear `sys.modules` cache carefully when implementing hot-reload

### Performance Considerations
- `importlib.import_module` has slight overhead over `import` statements
- `importlib.util.find_spec` is cheaper than importing (doesn't execute module code)
- `importlib.reload` re-executes the entire module — expensive for large modules
- Every import via importlib still goes through the full finder chain

### Interview Questions
1. What is the difference between `importlib.import_module` and `__import__`?
2. How does `importlib.reload` work and what are its limitations?
3. How can you create a module without a corresponding file?
4. What is `ModuleSpec` and what information does it contain?
5. How do you access data files bundled with a package using importlib?

### Coding Challenges
- Write a function that dynamically reloads all modules from a specific package
- Build a simple plugin loader that discovers and imports plugins from a directory
- Create a module that provides `importlib.resources` style API for file access
- Implement a module tracker that logs every imported module with its source

### Related Topics
- pkgutil, import hooks, sys.modules, module_spec, packaging resources

## Lazy loading
### What It Is
Lazy loading delays module import until the module is actually used (first attribute access), rather than at the `import` statement execution time. This can significantly reduce startup time for applications with many imports.

### Why It Is Important
Many Python applications import dozens or hundreds of modules at startup, most of which aren't used immediately. Lazy loading shifts this cost to first use, improving perceived startup performance. It is particularly valuable for CLI tools, web frameworks, and large applications.

### How It Works Internally
Lazy loading works by replacing the real module in `sys.modules` with a proxy object (or by using a special loader). When the proxy is first accessed (attribute get, method call, etc.), it triggers the actual import, then delegates all further operations to the real module. Python 3.12+ introduced `importlib.util.LazyLoader` as a built-in lazy loader.

### Syntax
```python
# Python 3.12+ built-in lazy loader
from importlib.util import LazyLoader

# Manual lazy module
class LazyModule:
    def __init__(self, name):
        self.__name__ = name
        self.__lazy_module = None
    
    def _load(self):
        if self.__lazy_module is None:
            import importlib
            self.__lazy_module = importlib.import_module(self.__name__)
        return self.__lazy_module
    
    def __getattr__(self, name):
        return getattr(self._load(), name)

# Using __getattr__ on a module (Python 3.7+)
# In __init__.py:
# def __getattr__(name):
#     return importlib.import_module(f".{name}", __name__)
```

### Beginner Examples
```python
# Manual lazy import pattern
def lazy_import(name):
    """Return a proxy that imports the module on first use."""
    import importlib
    
    class _LazyModule:
        def __init__(self, name):
            self._name = name
            self._mod = None
        
        def _get_mod(self):
            if self._mod is None:
                self._mod = importlib.import_module(self._name)
            return self._mod
        
        def __getattr__(self, attr):
            return getattr(self._get_mod(), attr)
    
    return _LazyModule(name)

# Usage — no import happens until .dumps() is accessed
json = lazy_import("json")
print("Module loaded lazily...")
print(json.dumps({"key": "value"}))  # Import happens here
```

### Intermediate Examples
```python
# Using __getattr__ for lazy submodule loading (Python 3.7+)
# In mypackage/__init__.py:
"""
Lazy submodule loading example.
"""
import importlib

__all__ = ["sub1", "sub2"]

def __getattr__(name):
    """Lazily load submodules on attribute access."""
    if name in __all__:
        return importlib.import_module(f".{name}", __name__)
    raise AttributeError(f"module {__name__!r} has no attribute {name!r}")

def __dir__():
    return __all__ + list(globals().keys())
```

```python
# Using importlib.util.LazyLoader (Python 3.12+)
# This requires a custom finder that uses LazyLoader

# Example of lazy importing submodules
from importlib import util

class LazyFinder:
    def __init__(self, modules):
        self.modules = modules
    
    def find_spec(self, fullname, path, target=None):
        if fullname in self.modules:
            spec = util.spec_from_loader(fullname, None)
            spec.loader = util.LazyLoader(spec.loader)
            return spec
        return None
```

### Advanced Examples
```python
# Full lazy loading framework using proxy pattern
import importlib
import sys

class LazyProxy:
    def __init__(self, module_name):
        self._module_name = module_name
        self._module = None
    
    def _resolve(self):
        if self._module is None:
            self._module = importlib.import_module(self._module_name)
        return self._module
    
    def __getattr__(self, name):
        return getattr(self._resolve(), name)
    
    def __call__(self, *args, **kwargs):
        return self._resolve()(*args, **kwargs)
    
    def __repr__(self):
        return f"<LazyProxy for {self._module_name}>"

# Lazy import context manager
class LazyImportContext:
    """Context that lazily imports modules within its scope."""
    def __init__(self):
        self.proxies = {}
    
    def __getattr__(self, name):
        if name not in self.proxies:
            self.proxies[name] = LazyProxy(name)
        return self.proxies[name]

# Usage
lazy = LazyImportContext()
# No import yet
print("No imports yet...")
# Use it — triggers import
print(lazy.json.dumps({"a": 1}))

# Lazy import for optimization
import time

def load_data(use_heavy_deps=False):
    # Heavy imports only on demand
    if use_heavy_deps:
        from pandas import DataFrame
        from numpy import array
        return DataFrame({"col": array([1, 2, 3])})
    return {"col": [1, 2, 3]}

# Module-level __getattr__ for lazy submodule access
# In a package __init__.py
import importlib

def __getattr__(name):
    # Map short names to full module paths
    lazy_mapping = {
        "cli": ".cli",
        "models": ".models",
        "utils": ".utils",
    }
    if name in lazy_mapping:
        return importlib.import_module(lazy_mapping[name], __name__)
    raise AttributeError(f"module {__name__!r} has no attribute {name!r}")
```

### Real-World Use Cases
- **CLI tools** — importing heavy dependencies only when specific subcommands need them
- **Web frameworks** — Django, FastAPI, Flask use lazy loading for heavyweight components
- **IDE tools** — autocompletion servers load only what's needed
- **Large applications** — splitting import cost across user interaction points
- **Plugin systems** — delay loading plugins until they're activated

### Common Mistakes
- Thinking lazy loading hides import errors (they just happen later)
- Using lazy loading for frequently accessed modules (adds overhead)
- Creating circular references with lazy proxies
- Forgetting that `LazyProxy` doesn't support `isinstance` checks against the real module types
- Not handling the case where the module fails to import on first access

### Best Practices
- Use lazy loading for heavy, rarely-used modules (pandas, numpy, torch, PIL)
- Do NOT lazy load modules that are always used (adds unnecessary overhead)
- Use Python 3.12+ `importlib.util.LazyLoader` where possible
- Implement lazy loading with `__getattr__` at the module level (Python 3.7+)
- Profile startup time before implementing lazy loading — target the actual bottlenecks
- Document which modules are lazily loaded for clarity
- Use `ModuleNotFoundError` handling in lazy loads for graceful degradation
- Consider using PEP 562 (`__getattr__` on modules) for clean lazy submodule access

### Performance Considerations
- Lazy loading trades startup time for first-use latency
- The proxy layer adds a small overhead per attribute access (usually negligible)
- For modules accessed many times, lazy loading may hurt overall performance
- Module `__getattr__` is only called once per attribute (result is cached)
- Using `LazyLoader` built-in avoids the Python-level proxy overhead
- Measure before and after with `time` module or profiling tools

### Interview Questions
1. What is the purpose of lazy loading in Python?
2. How does the `__getattr__` method work at the module level (Python 3.7+)?
3. What are the trade-offs between eager and lazy imports?
4. How would you implement a lazy import proxy class?
5. What is `importlib.util.LazyLoader` and when was it introduced?

### Coding Challenges
- Implement a lazy_module function that returns a proxy that delays import
- Profile a script's startup time with eager vs. lazy imports for a heavy library
- Build a simple CLI framework that only imports heavy libraries when needed
- Write a context manager that tracks all lazy import resolution times

### Related Topics
- PEP 562 (Module __getattr__), PEP 690 (Lazy Imports), importlib, import hooks
