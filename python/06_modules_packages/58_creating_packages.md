# Creating Packages - pyproject.toml, setuptools, publishing to PyPI

## Introduction

A Python package is a collection of modules organized in a directory hierarchy that can be distributed, installed, and imported as a single unit. Creating a proper package involves structuring your code, writing metadata, and optionally publishing to PyPI for worldwide distribution.

## Why It Is Important

Packages enable code reuse, modular design, and easy distribution. Properly packaging your code allows other developers (and your future self) to install it with a simple `pip install your-package`, manage dependencies, and version releases. It is the standard way to share Python code with the community.

## Syntax

```
mypackage/
  __init__.py
  module_a.py
  module_b.py
  subpackage/
    __init__.py
    module_c.py
  tests/
    test_module_a.py
pyproject.toml          # Modern (PEP 621)
setup.py                # Traditional
setup.cfg               # Declarative (legacy)
README.md
LICENSE
```

## Examples

### Basic Package Structure

```python
import os
from pathlib import Path

def create_package_structure(pkg_name):
    base = Path(pkg_name)
    base.mkdir(exist_ok=True)

    # Create __init__.py
    init_file = base / "__init__.py"
    init_file.write_text(f'''"""
{pkg_name} - A sample Python package.
"""

__version__ = "0.1.0"
__author__ = "Developer"

from .module_a import *
from .module_b import *
''')

    # Create module_a.py
    module_a = base / "module_a.py"
    module_a.write_text('''"""Module A provides basic math utilities."""

def add(a, b):
    """Add two numbers together."""
    return a + b

def subtract(a, b):
    """Subtract b from a."""
    return a - b

def multiply(a, b):
    """Multiply two numbers."""
    return a * b

def divide(a, b):
    """Divide a by b. Raises ZeroDivisionError if b is 0."""
    if b == 0:
        raise ZeroDivisionError("Cannot divide by zero")
    return a / b

def power(base, exp):
    """Raise base to the power of exp."""
    return base ** exp

__all__ = ["add", "subtract", "multiply", "divide", "power"]
''')

    # Create module_b.py
    module_b = base / "module_b.py"
    module_b.write_text('''"""Module B provides string utilities."""

def reverse_string(text):
    """Reverse a string."""
    return text[::-1]

def count_vowels(text):
    """Count vowels in a string."""
    vowels = "aeiouAEIOU"
    return sum(1 for c in text if c in vowels)

def is_palindrome(text):
    """Check if a string is a palindrome."""
    cleaned = text.lower().replace(" ", "")
    return cleaned == cleaned[::-1]

def capitalize_words(text):
    """Capitalize each word in a string."""
    return " ".join(word.capitalize() for word in text.split())

__all__ = ["reverse_string", "count_vowels", "is_palindrome", "capitalize_words"]
''')

    # Create subpackage
    subpkg = base / "subpackage"
    subpkg.mkdir(exist_ok=True)
    (subpkg / "__init__.py").write_text('''"""Subpackage initialization."""
from .module_c import *
''')

    module_c = subpkg / "module_c.py"
    module_c.write_text('''"""Module C provides advanced utilities."""

from ..module_a import add, multiply

def compute_stats(numbers):
    """Compute basic statistics for a list of numbers.

    >>> compute_stats([1, 2, 3, 4, 5])
    {'sum': 15, 'mean': 3.0, 'min': 1, 'max': 5, 'count': 5}
    """
    if not numbers:
        return {}
    total = sum(numbers)
    count = len(numbers)
    return {
        "sum": total,
        "mean": total / count,
        "min": min(numbers),
        "max": max(numbers),
        "count": count
    }

def matrix_multiply(A, B):
    """Multiply two matrices."""
    n = len(A)
    m = len(B[0])
    p = len(B)
    result = [[0] * m for _ in range(n)]
    for i in range(n):
        for j in range(m):
            for k in range(p):
                result[i][j] += A[i][k] * B[k][j]
    return result

__all__ = ["compute_stats", "matrix_multiply"]
''')

    # Create tests directory
    tests = Path("tests")
    tests.mkdir(exist_ok=True)
    test_file = tests / f"test_{pkg_name}.py"
    test_file.write_text(f'''"""Tests for {pkg_name} package."""

from {pkg_name} import add, subtract, reverse_string, is_palindrome

def test_add():
    assert add(2, 3) == 5
    assert add(-1, 1) == 0

def test_subtract():
    assert subtract(10, 5) == 5
    assert subtract(0, 5) == -5

def test_reverse_string():
    assert reverse_string("hello") == "olleh"
    assert reverse_string("") == ""

def test_is_palindrome():
    assert is_palindrome("racecar") == True
    assert is_palindrome("hello") == False

if __name__ == "__main__":
    test_add()
    test_subtract()
    test_reverse_string()
    test_is_palindrome()
    print("All tests passed!")
''')

    print(f"Package structure created at {base}/")
    for p in base.rglob("*"):
        if p.is_file():
            print(f"  {p.relative_to(Path('.'))}")

    return base

pkg_dir = create_package_structure("mypackage")
```

### setup.py (Traditional)

```python
from setuptools import setup, find_packages

setup(
    name="mypackage",
    version="0.1.0",
    description="A sample Python package",
    long_description=open("README.md").read() if Path("README.md").exists() else "",
    long_description_content_type="text/markdown",
    author="Developer",
    author_email="dev@example.com",
    url="https://github.com/example/mypackage",
    packages=find_packages(),
    include_package_data=True,
    install_requires=[
        "requests>=2.28.0",
    ],
    extras_require={
        "dev": ["pytest>=7.0", "black", "flake8"],
        "docs": ["sphinx>=5.0"],
    },
    python_requires=">=3.8",
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
    ],
    entry_points={
        "console_scripts": [
            "mypackage-cli=mypackage.cli:main",
        ],
    },
)
```

### pyproject.toml (Modern PEP 621)

```python
from pathlib import Path

pyproject_content = """[build-system]
requires = ["setuptools>=68.0", "wheel"]
build-backend = "setuptools.backends._legacy:_Backend"

[project]
name = "mypackage"
version = "0.1.0"
description = "A sample Python package"
readme = "README.md"
authors = [
    {name = "Developer", email = "dev@example.com"}
]
license = {text = "MIT"}
classifiers = [
    "Programming Language :: Python :: 3",
    "License :: OSI Approved :: MIT License",
]
requires-python = ">=3.8"
dependencies = [
    "requests>=2.28.0",
]

[project.optional-dependencies]
dev = ["pytest>=7.0", "black", "flake8"]
docs = ["sphinx>=5.0"]

[project.urls]
Homepage = "https://github.com/example/mypackage"

[project.scripts]
mypackage-cli = "mypackage.cli:main"

[tool.setuptools.packages.find]
include = ["mypackage*"]
"""

Path("pyproject.toml").write_text(pyproject_content)
print("Created pyproject.toml (PEP 621)")
```

### setup.cfg (Declarative)

```python
from pathlib import Path

setup_cfg_content = """[metadata]
name = mypackage
version = 0.1.0
description = A sample Python package
author = Developer
author_email = dev@example.com

[options]
packages = find:
install_requires =
    requests>=2.28.0
python_requires = >=3.8

[options.extras_require]
dev = pytest>=7.0; black; flake8
docs = sphinx>=5.0

[options.entry_points]
console_scripts =
    mypackage-cli = mypackage.cli:main

[options.packages.find]
include = mypackage*
"""

Path("setup.cfg").write_text(setup_cfg_content)
print("Created setup.cfg")
```

### Installing the Package in Editable Mode

```python
import subprocess
import sys

def install_editable(package_dir="."):
    result = subprocess.run(
        [sys.executable, "-m", "pip", "install", "-e", package_dir],
        capture_output=True, text=True
    )
    if result.returncode == 0:
        print(f"Installed {package_dir} in editable mode")
    else:
        print(f"Error: {result.stderr}")
    return result.returncode == 0

pip_result = subprocess.run(
    [sys.executable, "-m", "pip", "install", "-e", "."],
    capture_output=True, text=True
)
print(pip_result.stdout[:300] if pip_result.returncode == 0 else pip_result.stderr[:300])
```

## Beginner Examples

```python
# Creating a simple single-module package
from pathlib import Path

def create_simple_package():
    pkg_dir = Path("simple_pkg")
    pkg_dir.mkdir(exist_ok=True)

    init_file = pkg_dir / "__init__.py"
    init_file.write_text("""from .greeter import greet, farewell
from .calculator import add, subtract, multiply, divide

__version__ = "0.1.0"
""")

    greeter = pkg_dir / "greeter.py"
    greeter.write_text("""def greet(name):
    return f"Hello, {name}!"

def farewell(name):
    return f"Goodbye, {name}!"

def welcome_message():
    return "Welcome to simple_pkg!"
""")

    calculator = pkg_dir / "calculator.py"
    calculator.write_text("""from functools import reduce

def add(*args):
    return sum(args)

def subtract(a, b):
    return a - b

def multiply(*args):
    return reduce(lambda x, y: x * y, args) if args else 0

def divide(a, b):
    if b == 0:
        raise ZeroDivisionError("Cannot divide by zero")
    return a / b

def average(numbers):
    return sum(numbers) / len(numbers) if numbers else 0
""")

    # Create pyproject.toml
    pyproject = Path("pyproject.toml")
    pyproject.write_text("""[build-system]
requires = ["setuptools>=68.0"]
build-backend = "setuptools.backends._legacy:_Backend"

[project]
name = "simple-pkg"
version = "0.1.0"
description = "A very simple package example"
requires-python = ">=3.8"
""")

    print("Simple package created!")
    return pkg_dir

simple_pkg = create_simple_package()

# Import and use the package
import sys
sys.path.insert(0, ".")
from simple_pkg import greet, add
print(greet("World"))
print(f"add(1, 2, 3): {add(1, 2, 3)}")

import shutil
# shutil.rmtree("simple_pkg")
# Path("pyproject.toml").unlink()
```

## Intermediate Examples

```python
# Package versioning with setuptools_scm
from pathlib import Path

scm_example = """# If using setuptools-scm for automatic versioning from git tags:
# pyproject.toml:
# [build-system]
# requires = ["setuptools>=68.0", "setuptools-scm>=8.0"]
# build-backend = "setuptools.backends._legacy:_Backend"
#
# [tool.setuptools_scm]
# version_scheme = "post-release"

def get_version_from_scm():
    try:
        from setuptools_scm import get_version
        return get_version()
    except (ImportError, LookupError):
        return "0.0.0"
"""

print("setuptools-scm is useful for automatic versioning from git tags")

# Package with compiled extensions (C extensions)
ext_example = """
from setuptools import setup, Extension

module = Extension(
    "my_package._fastmath",
    sources=["src/_fastmath.c"],
    include_dirs=["include"],
)

setup(
    name="my-package",
    version="0.1.0",
    ext_modules=[module],
)
"""

print("Cython/C extensions can be included via Extension class")

# Testing package distribution with build module
def build_package():
    import subprocess
    import sys

    result = subprocess.run(
        [sys.executable, "-m", "build", "--wheel"],
        capture_output=True, text=True
    )
    if result.returncode == 0:
        dist_dir = Path("dist")
        wheels = list(dist_dir.glob("*.whl"))
        print(f"Built wheels: {[w.name for w in wheels]}")
        return wheels
    else:
        print(f"Build failed: {result.stderr}")
        return []

def create_entry_points():
    """Create a CLI entry point for the package."""
    cli_file = Path("mypackage/cli.py")
    cli_file.write_text('''"""Command-line interface for mypackage."""

import argparse
import sys

def main():
    parser = argparse.ArgumentParser(description="mypackage CLI")
    parser.add_argument("command", choices=["add", "greet"], help="Command to run")
    parser.add_argument("--name", default="World", help="Name for greeting")
    parser.add_argument("--numbers", nargs="+", type=float, help="Numbers to add")

    args = parser.parse_args()

    if args.command == "add":
        if args.numbers:
            from mypackage import add
            result = add(*args.numbers)
            print(f"Result: {result}")
        else:
            print("Please provide --numbers")
    elif args.command == "greet":
        from mypackage import greet
        print(greet(args.name))

if __name__ == "__main__":
    main()
''')
    print("CLI entry point created")

create_entry_points()
```

## Advanced Examples

```python
# Complete package with pyproject.toml, CI config, documentation, and testing
from pathlib import Path
import textwrap

class PackageScaffolder:
    def __init__(self, name, author="Developer", email="dev@example.com"):
        self.name = name
        self.author = author
        self.email = email
        self.base_dir = Path(name.replace("-", "_"))

    def create_structure(self):
        # Create package directory
        pkg_dir = self.base_dir
        pkg_dir.mkdir(parents=True, exist_ok=True)
        (pkg_dir / "subpackage").mkdir(exist_ok=True)

        # Create __init__.py
        self._write_file(pkg_dir / "__init__.py", f'''"""
{self.name} - A professional Python package.
"""

__version__ = "0.1.0"
__author__ = "{self.author}"

from .public_api import *
''')

        # Create main module
        self._write_file(pkg_dir / "public_api.py", '''"""Public API for the package."""

from .core import Processor, Config
from .utils import format_output, validate_input

__all__ = ["Processor", "Config", "format_output", "validate_input"]
''')

        # Create core module
        self._write_file(pkg_dir / "core.py", '''"""Core module containing main business logic."""

import json
from pathlib import Path
from typing import Any, Dict, Optional

class Config:
    """Configuration manager for the package."""

    def __init__(self, config_path: Optional[str] = None):
        self.config_path = Path(config_path) if config_path else None
        self._data: Dict[str, Any] = {}
        if self.config_path and self.config_path.exists():
            self.load()

    def load(self):
        if self.config_path:
            with open(self.config_path) as f:
                self._data = json.load(f)

    def save(self):
        if self.config_path:
            self.config_path.parent.mkdir(parents=True, exist_ok=True)
            with open(self.config_path, "w") as f:
                json.dump(self._data, f, indent=2)

    def get(self, key: str, default: Any = None) -> Any:
        return self._data.get(key, default)

    def set(self, key: str, value: Any):
        self._data[key] = value

    def __repr__(self):
        return f"Config(data={self._data})"

class Processor:
    """Main processor class."""

    def __init__(self, config: Optional[Config] = None):
        self.config = config or Config()

    def process(self, data: Dict[str, Any]) -> Dict[str, Any]:
        result = {}
        for key, value in data.items():
            if isinstance(value, (int, float)):
                result[key] = self._transform_numeric(value)
            elif isinstance(value, str):
                result[key] = self._transform_string(value)
            else:
                result[key] = value
        return result

    def _transform_numeric(self, value):
        multiplier = self.config.get("multiplier", 1)
        return value * multiplier

    def _transform_string(self, value):
        prefix = self.config.get("prefix", "")
        suffix = self.config.get("suffix", "")
        return f"{prefix}{value}{suffix}"

    def batch_process(self, items):
        return [self.process(item) for item in items]

    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.config.save()
''')

        # Create utils module
        self._write_file(pkg_dir / "utils.py", '''"""Utility functions."""

from typing import Any

def format_output(data: Any, indent: int = 2) -> str:
    """Format data as pretty-printed JSON string."""
    import json
    return json.dumps(data, indent=indent, default=str)

def validate_input(value: Any, expected_type: type) -> bool:
    """Validate that a value matches the expected type."""
    return isinstance(value, expected_type)

def merge_dicts(base: dict, override: dict) -> dict:
    """Deep merge two dictionaries."""
    result = base.copy()
    for key, value in override.items():
        if key in result and isinstance(result[key], dict) and isinstance(value, dict):
            result[key] = merge_dicts(result[key], value)
        else:
            result[key] = value
    return result

def chunk_list(items, chunk_size):
    """Split a list into chunks of the specified size."""
    for i in range(0, len(items), chunk_size):
        yield items[i:i + chunk_size]
''')

        # Create subpackage __init__
        self._write_file(pkg_dir / "subpackage" / "__init__.py", '''"""Subpackage initialization."""

from .specialized import SpecializedProcessor

__all__ = ["SpecializedProcessor"]
''')

        # Create subpackage module
        self._write_file(pkg_dir / "subpackage" / "specialized.py", '''"""Specialized processors."""

from ..core import Processor

class SpecializedProcessor(Processor):
    """Extended processor with additional functionality."""

    def __init__(self, config=None, mode="strict"):
        super().__init__(config)
        self.mode = mode

    def process(self, data):
        result = super().process(data)
        if self.mode == "strict":
            result["_validated"] = True
        return result

    def analyze(self, data):
        """Analyze data and return statistics."""
        numeric_values = [v for v in data.values() if isinstance(v, (int, float))]
        return {
            "total": len(data),
            "numeric_count": len(numeric_values),
            "numeric_sum": sum(numeric_values),
            "mode": self.mode
        }
''')

        # Create tests
        test_dir = Path("tests")
        test_dir.mkdir(exist_ok=True)
        self._write_file(test_dir / f"test_{self.base_dir.name}_core.py", f'''"""Tests for core module."""

import pytest
import json
from pathlib import Path
from {self.base_dir.name} import Config, Processor

@pytest.fixture
def sample_config():
    return Config()

@pytest.fixture
def temp_config(tmp_path):
    config_file = tmp_path / "config.json"
    config_file.write_text(json.dumps({{"multiplier": 2, "prefix": "!!"}}))
    return Config(str(config_file))

class TestConfig:
    def test_default_values(self, sample_config):
        assert sample_config.get("nonexistent") is None
        assert sample_config.get("nonexistent", "default") == "default"

    def test_set_and_get(self, sample_config):
        sample_config.set("key1", "value1")
        assert sample_config.get("key1") == "value1"

    def test_config_persistence(self, temp_config):
        assert temp_config.get("multiplier") == 2
        assert temp_config.get("prefix") == "!!"

class TestProcessor:
    def test_process_numeric(self):
        proc = Processor()
        result = proc.process({{"value": 10}})
        assert result["value"] == 10

    def test_process_with_config(self, temp_config):
        proc = Processor(temp_config)
        result = proc.process({{"value": 10, "text": "hello"}})
        assert result["value"] == 20
        assert result["text"] == "!!hello"

    def test_batch_process(self):
        proc = Processor()
        items = [{{"a": 1}}, {{"a": 2}}, {{"a": 3}}]
        results = proc.batch_process(items)
        assert len(results) == 3
        assert results[0]["a"] == 1

    def test_context_manager(self, tmp_path):
        config_file = tmp_path / "test_config.json"
        config = Config(str(config_file))
        config.set("key", "value")
        config.save()
        assert config_file.exists()
''')

        # Create pyproject.toml
        self._write_file("pyproject.toml", f"""[build-system]
requires = ["setuptools>=68.0", "wheel"]
build-backend = "setuptools.backends._legacy:_Backend"

[project]
name = "{self.name}"
version = "0.1.0"
description = "A professional Python package"
readme = "README.md"
authors = [
    {{name = "{self.author}", email = "{self.email}"}}
]
license = {{text = "MIT"}}
classifiers = [
    "Programming Language :: Python :: 3",
    "Programming Language :: Python :: 3.8",
    "Programming Language :: Python :: 3.9",
    "Programming Language :: Python :: 3.10",
    "Programming Language :: Python :: 3.11",
    "Programming Language :: Python :: 3.12",
    "License :: OSI Approved :: MIT License",
    "Operating System :: OS Independent",
]
requires-python = ">=3.8"
keywords = ["example", "template", "package"]

[project.urls]
Homepage = "https://github.com/{self.author}/{self.name}"
Repository = "https://github.com/{self.author}/{self.name}.git"

[tool.setuptools.packages.find]
include = ["{self.base_dir.name}*"]

[tool.pytest.ini_options]
testpaths = ["tests"]

[tool.black]
line-length = 100
target-version = ["py38"]

[tool.isort]
profile = "black"
line_length = 100
""")

        # Create README
        self._write_file("README.md", f"# {self.name}\n\nA professional Python package.\n\n## Installation\n\n```bash\npip install {self.name}\n```\n\n## Usage\n\n```python\nfrom {self.base_dir.name} import Processor, Config\n\nconfig = Config()\nconfig.set('multiplier', 2)\nprocessor = Processor(config)\nresult = processor.process({{'value': 10}})\nprint(result)\n```\n")

        # Create LICENSE
        self._write_file("LICENSE", "MIT License\n\nCopyright (c) 2024 Developer\n\nPermission is hereby granted...")

        # Create .gitignore
        self._write_file(".gitignore", """# Python
__pycache__/
*.py[cod]
*.egg-info/
dist/
build/
.eggs/
*.egg
.venv/
venv/
env/

# IDE
.vscode/
.idea/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Testing
.pytest_cache/
coverage/
htmlcov/
.tox/

# Distribution
*.whl
*.tar.gz
""")

        print(f"Complete package scaffold created for '{self.name}'")
        print(f"\nStructure:")
        for p in sorted(Path(".").rglob("*")):
            if p.is_file() and ".pyc" not in str(p) and "__pycache__" not in str(p):
                rel = p.relative_to(Path("."))
                print(f"  {rel}")

        return self.base_dir

    def _write_file(self, path, content):
        content = textwrap.dedent(content)
        path_obj = Path(path)
        path_obj.parent.mkdir(parents=True, exist_ok=True)
        path_obj.write_text(content)

scaffolder = PackageScaffolder("my-advanced-pkg")
scaffolder.create_structure()

# Demonstrate building and testing
def verify_package_integrity():
    import importlib
    try:
        from my_advanced_pkg import Processor, Config
        config = Config()
        config.set("test", True)
        processor = Processor(config)
        result = processor.process({"value": 42})
        print(f"Package works! Result: {result}")
        return True
    except ImportError as e:
        print(f"Cannot verify package (expected if not installed): {e}")
        return False

verify_package_integrity()

import shutil
import os

for p in ["my_advanced_pkg", "my-advanced-pkg", "tests"]:
    path = Path(p)
    if path.exists() and path.is_dir():
        shutil.rmtree(path, ignore_errors=True)

for f in ["pyproject.toml", "README.md", "LICENSE", ".gitignore"]:
    Path(f).unlink(missing_ok=True)

for d in ["simple_pkg", "mypackage"]:
    Path(d).unlink(missing_ok=True) if Path(d).is_file() else shutil.rmtree(d, ignore_errors=True) if Path(d).exists() else None
```

### Publishing to PyPI (Simulation)

```python
def simulate_pypi_publish():
    import subprocess
    import sys

    steps = [
        "pip install build twine",
        "python -m build",
        "twine check dist/*",
        "twine upload dist/*  # Prompts for PyPI credentials",
        "pip install your-package"
    ]

    print("Steps to publish to PyPI:")
    for i, step in enumerate(steps, 1):
        print(f"  {i}. {step}")

    commands = [
        [sys.executable, "-m", "pip", "install", "build", "twine"],
    ]

    print("\nWould run:")
    for cmd in commands:
        print(f"  {' '.join(cmd)}")

simulate_pypi_publish()
```

## Real-World Use Cases

- **Open-source libraries**: Publishing utility libraries, frameworks, and tools to PyPI
- **Internal corporate packages**: Sharing code across teams via private PyPI servers (e.g., devpi, AWS CodeArtifact)
- **Microservices**: Packaging service logic with its own dependencies for containerized deployment
- **CLI tools**: Distributing command-line applications via `pip install` with console_scripts entry points
- **Plugin systems**: Creating namespace packages (e.g., `flask_plugin_*`) for extensible applications
- **Data science**: Packaging models, preprocessing pipelines, and analysis code with versioned dependencies

## Common Mistakes

- Forgetting to include a `__init__.py` in subpackages (pre-3.3) or leaving it empty when initialization is needed
- Not using `find_packages()` in setup.py and manually listing packages (easy to forget new ones)
- Including `.pyc` files, `__pycache__`, or environment files in the distribution
- Hardcoding version strings instead of reading from a single source (use `importlib.metadata` or `__version__`)
- Publishing sensitive information (API keys, passwords) to PyPI
- Using overly broad `install_requires` version specifiers that conflict with other packages
- Not pinning build-system requirements in `pyproject.toml`
- Forgetting to include `README.md` as `long_description` for a nice PyPI page

## Best Practices

- Use `pyproject.toml` (PEP 621) for modern, declarative package configuration
- Define a clear, single source of truth for version strings (e.g., `__version__` in `__init__.py` read by `importlib.metadata`)
- Use `find_packages(include=["mypackage*"])` to auto-discover packages
- Separate dev dependencies into `[project.optional-dependencies] dev = [...]`
- Include tests in the source distribution but not in the installed package
- Use `python -m build` to create reproducible distributions
- Add `console_scripts` entry points for CLI tools
- Always specify `python_requires` to prevent installation on unsupported versions
- Use `include_package_data=True` with `MANIFEST.in` or use `tool.setuptools.package-data`

## Interview Questions

1. **Q**: What is the difference between a package and a module?
   **A**: A module is a single `.py` file. A package is a directory containing `__init__.py` and submodules/subpackages. Packages can be distributed as a single unit.

2. **Q**: What is the purpose of `__init__.py`?
   **A**: It marks a directory as a Python package, can import submodules for convenience, define `__all__`, and execute initialization code when the package is first imported.

3. **Q**: How does `setuptools.find_packages()` work?
   **A**: It recursively searches for directories containing `__init__.py` files and returns their names as a list of packages to include in the distribution.

4. **Q**: What are namespace packages and when would you use them?
   **A**: Namespace packages allow splitting a single package across multiple distribution packages (e.g., `namespace_package.subpackage_a` in one dist and `namespace_package.subpackage_b` in another). They use implicit namespace packages (PEP 420) with no `__init__.py`.

5. **Q**: How do you handle conditional dependencies for different platforms?
   **A**: Use environment markers in `install_requires` (e.g., `pywin32; sys_platform == "win32"`) or `extras_require` with platform-specific extras.

## Coding Challenges

1. **Package Dependency Visualizer**: Write a tool that reads a package's metadata and generates a dependency graph as a DOT file or ASCII tree.

2. **Custom Package Index**: Implement a minimal PyPI-compatible package index server that can serve wheels and handle `pip install`.

3. **Version Bumper**: Create a CLI tool that reads `__version__` from an `__init__.py`, increments it (major/minor/patch), and updates all relevant files (pyproject.toml, CHANGELOG, etc.).

4. **License Compatibility Checker**: Build a tool that reads a package's dependencies' licenses and checks compatibility with the package's own license.

5. **Package Size Optimizer**: Analyze a package's distribution and suggest optimizations: removing unnecessary files, compressing assets, tree-shaking unused submodules.

## Summary

Creating Python packages involves structuring code with `__init__.py`, writing metadata (pyproject.toml), and optionally publishing to PyPI. Modern tooling uses PEP 621 for declarative configuration, setuptools for building, and twine for uploading. Proper packaging practices—versioning, dependency management, entry points, and testing—are essential for sharing and maintaining Python code professionally.

## Related Topics

Import System (57.x), pip (55.x), Virtual Environments (56.x), Standard Library (54.x)
