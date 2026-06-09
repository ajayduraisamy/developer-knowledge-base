# Creating Packages - pyproject.toml, setuptools, publishing to PyPI
## Introduction
Creating a Python package involves structuring your code for distribution, declaring metadata and dependencies, building distributable artifacts (wheels and source distributions), and uploading them to a package index like PyPI. Modern Python packaging centers on the `pyproject.toml` standard (PEP 517/518/621), with `setuptools` as the most common build backend.

## pyproject.toml
### What It Is
`pyproject.toml` is a configuration file format specified in PEP 518 and PEP 621. It centralizes project metadata, build system requirements, and tool configurations in a single TOML file, replacing the historical proliferation of `setup.py`, `setup.cfg`, `MANIFEST.in`, and various tool-specific config files.

### Why It Is Important
`pyproject.toml` standardizes Python project configuration across different build backends (setuptools, flit, hatch, poetry, pdm). It separates build system requirements from project metadata, makes configuration declarative (not code), and integrates tool settings (linters, formatters, type checkers) in one place.

### How It Works Internally
When a build frontend (like `pip` or `build`) encounters a project with `pyproject.toml`, it reads the `[build-system]` table to determine:
- `requires` — packages needed to build the project (e.g., `setuptools`, `wheel`)
- `build-backend` — the Python object implementing the build API (`setuptools.build_meta:__legacy__`, `hatchling.build`, etc.)

The frontend creates an isolated build environment, installs `requires`, then calls the backend's `build_wheel()` or `build_sdist()` function. PEP 621 defines `[project]` for standardized metadata.

### Syntax
```toml
[build-system]
requires = ["setuptools>=68.0"]
build-backend = "setuptools.build_meta"

[project]
name = "my-package"
version = "0.1.0"
requires-python = ">=3.9"
dependencies = ["requests>=2.28"]
```

### Beginner Examples
```toml
[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"

[project]
name = "my-first-package"
version = "0.1.0"
description = "A simple example package"
readme = "README.md"
authors = [{name = "Your Name", email = "you@example.com"}]
license = {text = "MIT"}
requires-python = ">=3.9"
dependencies = ["click>=8.0"]
```

### Intermediate Examples
```toml
[build-system]
requires = ["setuptools>=68.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "advanced-pkg"
dynamic = ["version"]
description = "Advanced package with extras"
readme = "README.md"
authors = [{name = "Developer", email = "dev@example.com"}]
license = {text = "MIT"}
requires-python = ">=3.10"
dependencies = ["click>=8.1", "rich>=13.0"]

[project.optional-dependencies]
dev = ["pytest>=7.4", "black>=23.0", "mypy>=1.6", "ruff>=0.1.0"]
web = ["fastapi>=0.100", "uvicorn[standard]>=0.23"]

[project.scripts]
my-cli = "advanced_pkg.cli:main"

[tool.setuptools.packages.find]
where = ["src"]
exclude = ["tests*", "docs*"]

[tool.setuptools.dynamic]
version = {attr = "advanced_pkg.__version__"}
```

### Advanced Examples
```toml
[build-system]
requires = ["hatchling>=1.18"]
build-backend = "hatchling.build"

[project]
name = "monorepo-pkg"
version = "1.0.0"
description = "Monorepo structured package"
requires-python = ">=3.11"
dependencies = ["tomli>=2.0"]

[tool.hatch.build.targets.wheel]
packages = ["src/monorepo_pkg"]

[tool.hatch.envs.default]
dependencies = ["pytest>=7.4", "coverage>=7.3"]

[tool.hatch.envs.default.scripts]
test = "pytest tests/"
lint = "ruff check src/ tests/"
```

```toml
# Dynamic version from VCS
[build-system]
requires = ["setuptools>=68.0", "setuptools-scm>=8.0"]
build-backend = "setuptools.build_meta"

[project]
name = "vcs-versioned-pkg"
dynamic = ["version"]

[tool.setuptools_scm]
version_scheme = "no-guess-dev"
local_scheme = "no-local-version"
```

### Real-World Use Cases
- Open-source libraries declaring metadata, dependencies, and build config
- Internal company packages specifying private dependencies and indexes
- CLI tools using [project.scripts] for console entry points
- Plugin systems using [project.entry-points] for discoverable plugins
- Monorepos configuring subpackage discovery and build isolation

### Common Mistakes
- Mixing [project] (PEP 621) with [tool.poetry] backend-specific sections
- Not including wheel in [build-system] requires when using older setuptools
- Forgetting readme = "README.md" — PyPI requires this
- Setting dynamic fields without providing corresponding tool configuration

### Best Practices
- Always use pyproject.toml over legacy setup.py for metadata
- Use src/ layout (src/mypackage/) to prevent import confusion
- Separate production and development dependencies
- Use dynamic = ["version"] with setuptools-scm for VCS-based versioning
- Include classifiers for Python versions, license, and OS support

### Performance Considerations
- Build backend choice affects build speed: hatchling and flit_core are faster
- Including unnecessary files increases wheel size
- Dynamic version resolution (git commands) adds overhead to every build

### Interview Questions
1. What is the purpose of [build-system] in pyproject.toml?
2. How does PEP 621 change package metadata declaration?
3. What is the difference between project.dependencies and project.optional-dependencies?
4. How do you specify console scripts in pyproject.toml?
5. What is the src/ layout and why is it recommended?

### Coding Challenges
- Create a minimal pyproject.toml for a package with one dependency and a CLI script
- Convert a legacy setup.py-based project to pyproject.toml format
- Write a script that validates a pyproject.toml against the PEP 621 schema

### Related Topics
- setuptools, hatchling, flit, poetry, PEP 517, PEP 518, PEP 621, TOML

## setuptools
### What It Is
Setuptools is the most widely used Python build backend and package development library. It builds on top of distutils and provides package discovery, dependency management, namespace packages, C extension building, and the setup.py command-line interface.

### Why It Is Important
Despite the rise of alternatives (hatch, flit, poetry), setuptools remains the default backend for the majority of Python packages on PyPI. It has the most extensive ecosystem support, C extension compilation, and compatibility with legacy workflows.

### How It Works Internally
Setuptools extends distutils by adding find_packages() for automatic package discovery, entry points (console scripts, plugin hooks), dependency resolution and extras, MANIFEST.in for file inclusion/exclusion, and build commands (build, sdist, bdist_wheel, develop, install).

### Syntax
```python
from setuptools import setup, find_packages

setup(
    name="my-package",
    version="0.1.0",
    packages=find_packages(),
    install_requires=["requests>=2.28"],
)
```

### Beginner Examples
```toml
[build-system]
requires = ["setuptools>=68.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "hello-world-pkg"
version = "0.1.0"
description = "A hello world package"
requires-python = ">=3.9"
dependencies = []

[tool.setuptools.packages.find]
include = ["hello_world*"]
```

```python
# hello_world/__init__.py
def greet(name):
    return f"Hello, {name}!"
```

### Intermediate Examples
```python
from setuptools import setup, find_packages

with open("README.md") as f:
    long_description = f.read()

setup(
    name="dynamic-pkg",
    use_scm_version=True,
    setup_requires=["setuptools-scm"],
    description="Package with dynamic version",
    long_description=long_description,
    long_description_content_type="text/markdown",
    packages=find_packages(where="src"),
    package_dir={"": "src"},
    include_package_data=True,
    install_requires=["click>=8.0", "rich>=12.0"],
    extras_require={"dev": ["pytest", "black", "mypy"]},
    entry_points={"console_scripts": ["dynamic-cli=dynamic_pkg.cli:main"]},
    python_requires=">=3.9",
)
```

### Advanced Examples
```python
# C extension building
from setuptools import setup, Extension

module = Extension(
    "mypkg._core",
    sources=["src/mypkg/_core.c"],
    include_dirs=["/usr/local/include"],
    libraries=["ssl", "crypto"],
    library_dirs=["/usr/local/lib"],
)

setup(
    name="mypkg-with-c",
    version="0.1.0",
    ext_modules=[module],
    packages=["mypkg"],
)

# Custom build commands
from setuptools import Command

class LintCommand(Command):
    description = "run linter"
    user_options = []
    def initialize_options(self): pass
    def finalize_options(self): pass
    def run(self):
        import subprocess
        subprocess.check_call(["ruff", "check", "src/"])

setup(
    name="custom-commands",
    version="0.1.0",
    cmdclass={"lint": LintCommand},
)
```

### Real-World Use Cases
- Building and distributing Python packages with C extensions
- Creating packages with data files and scripts
- Managing namespace packages for large projects
- Legacy project maintenance for existing setup.py-based code

### Common Mistakes
- Using setup.py for metadata when pyproject.toml should be used
- Not including data files via MANIFEST.in or include_package_data=True
- Hardcoding paths instead of using package_data
- Forgetting to exclude test files from the distribution

### Best Practices
- Prefer pyproject.toml + setup.cfg over setup.py for static metadata
- Use setup.py only for dynamic configuration or C extensions
- Always use find_packages() or explicit package list
- Include a README, LICENSE, and other standard files via MANIFEST.in
- Use pip install -e . for development installation

### Performance Considerations
- setuptools is slower than hatchling and flit for build operations
- C extension compilation adds significant build time
- Larger packages with many data files increase install time

### Interview Questions
1. What is the difference between install_requires and extras_require?
2. How do you include non-Python files in a setuptools package?
3. What is the purpose of setup_requires?
4. How do you build C extensions with setuptools?

### Coding Challenges
- Create a package with both Python modules and a C extension
- Write a setuptools Command subclass that runs tests before building
- Build a setup.py that dynamically discovers subpackages

### Related Topics
- pyproject.toml, distutils, wheel, MANIFEST.in, Cython

## Publishing to PyPI
### What It Is
Publishing to PyPI means uploading a Python package so that anyone can install it with pip install <package>. PyPI (Python Package Index) is the official third-party software repository for Python.

### Why It Is Important
Publishing to PyPI makes your package accessible to the entire Python community. It is the standard distribution channel for open-source Python libraries, tools, and frameworks.

### How It Works Internally
Publishing involves three steps: building (creating a wheel and/or source distribution), registering (creating the package on PyPI), and uploading (sending files via HTTP to PyPI's API). PyPI validates metadata, checks for duplicates, and archives the files. The package then becomes available via the PyPI web interface and the JSON API.

### Syntax
```bash
python -m build                     # build sdist and wheel
twine upload dist/*                 # upload to PyPI
twine upload --repository-url https://test.pypi.org/legacy/ dist/*
twine upload --username __token__ --password pypi-XXXXXXXX dist/*
```

### Beginner Examples
```bash
# 1. Install build tools
pip install build twine

# 2. Build distributions
python -m build

# 3. Upload to Test PyPI first
twine upload --repository-url https://test.pypi.org/legacy/ dist/*

# 4. Test install from Test PyPI
pip install --index-url https://test.pypi.org/simple/ my-package

# 5. Upload to real PyPI
twine upload dist/*
```

### Intermediate Examples
```python
# pyproject.toml with scripts and extras
[project]
name = "my-lib"
version = "0.2.0"
dependencies = ["click>=8.0"]
optional-dependencies = {dev = ["pytest", "black"], web = ["fastapi"]}

[project.scripts]
my-cli = "my_lib.cli:main"

[project.entry-points."myplugin"]
my_plugin = "my_lib.plugin"
```

```yaml
# .github/workflows/publish.yml
name: Publish to PyPI
on:
  release:
    types: [published]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
      - run: pip install build twine
      - run: python -m build
      - run: twine upload dist/*
        env:
          TWINE_USERNAME: __token__
          TWINE_PASSWORD: ${{ secrets.PYPI_API_TOKEN }}
```

### Advanced Examples
```python
# Programmatic PyPI upload
import os
import subprocess

def build_and_publish(api_token):
    subprocess.check_call(["python", "-m", "build"])
    subprocess.check_call([
        "twine", "upload", "dist/*",
        "--username", "__token__",
        "--password", api_token,
    ])

# Verify package before upload
def check_before_publish():
    import importlib.metadata
    dist = importlib.metadata.distribution("my-package")
    print(f"Version: {dist.version}")
    print(f"Requires: {list(dist.requires or [])}")
```

### Real-World Use Cases
- Open-source libraries distributed to the community
- Internal tools hosted on private PyPI servers (DevPI, AWS CodeArtifact)
- CLI tools distributed via pip install

### Common Mistakes
- Uploading without building (source instead of wheel)
- Forgetting to bump version before publishing (PyPI rejects duplicates)
- Including __pycache__ or .pyc files in distributions
- Not testing on Test PyPI before production upload
- Exposing API tokens in code or committing to version control

### Best Practices
- Always publish to Test PyPI first (test.pypi.org)
- Use Trusted Publishing (OIDC) for CI/CD instead of API tokens
- Use twine check dist/* to validate distributions before uploading
- Follow semantic versioning (semver) for release numbering
- Include README.md, LICENSE, CHANGELOG.md in the distribution
- Add classifiers for Python versions, license, and topic
- Use python -m build (not setup.py sdist bdist_wheel directly)
- Enable 2FA on your PyPI account

### Performance Considerations
- Wheels install faster than source distributions (no compilation)
- Universal wheels (py3-none-any) work everywhere
- Platform-specific wheels for C extensions require CI for each platform
- Binary wheels reduce user installation time significantly

### Interview Questions
1. What is the difference between sdist and bdist_wheel?
2. How do you specify package dependencies in pyproject.toml?
3. What is a "universal wheel" and when should you use it?
4. How does Trusted Publishing (OIDC) improve security for PyPI uploads?
5. What steps would you take to publish a package that contains C extensions?

### Coding Challenges
- Write a CI script that auto-increments version and publishes on tag push
- Build a tool that checks package metadata for common issues before release
- Create a minimal PyPI index server for internal package hosting

### Related Topics
- pyproject.toml, setuptools, twine, build, hatchling, flit, poetry, pip
