# pip - Package installer, requirements.txt, PyPI
## Introduction
pip (Pip Installs Packages) is the standard package manager for Python. It downloads and installs packages from PyPI (Python Package Index) and other indexes, manages dependencies, and handles package versions. Since Python 3.4, pip is bundled with Python itself. This file covers pip as a package installer, the `requirements.txt` standard, and the process of publishing packages to PyPI.

## Package installer
### What It Is
pip is a command-line tool for installing, upgrading, and removing Python packages. It resolves dependencies, downloads wheels or source distributions, and places them into the appropriate site-packages directory.

### Why It Is Important
pip is the backbone of Python's ecosystem. It enables developers to reuse millions of open-source packages with a single command. Without pip, every project would need to bundle its dependencies manually.

### How It Works Internally
pip parses package metadata from PyPI (a JSON API at `https://pypi.org/pypi/<package>/json`). It fetches PEP 508 requirement specifiers, builds a dependency graph, and uses a SAT solver (or the simpler backtracking resolver in pip 20.3+) to select compatible versions. It then downloads wheels (`.whl`, pre-built archives) or source distributions (`.tar.gz`/`.zip`) and extracts them into `site-packages`. Installation is logged in `pip`s cache (`~/.cache/pip`).

### Syntax
```bash
pip install <package>
pip install <package>==<version>
pip install <package>>=<version>,<<version>
pip install -r requirements.txt
pip uninstall <package>
pip list                     # installed packages
pip show <package>           # package details
pip freeze                   # list installed with versions
pip check                    # verify dependencies
pip search <query>           # search PyPI (limited)
pip cache list               # show cached downloads
```

### Beginner Examples
```bash
# Install a package
pip install requests

# Install specific version
pip install django==4.2.10

# Install with version range
pip install "numpy>=1.24,<2.0"

# Install multiple packages
pip install flask gunicorn psycopg2-binary

# Upgrade a package
pip install --upgrade pip
pip install --upgrade requests

# Uninstall
pip uninstall requests -y

# List installed packages
pip list

# Show package info
pip show requests
```

### Intermediate Examples
```bash
# Install from requirements
pip install -r requirements.txt

# Install from a different index
pip install --index-url https://private-pypi.example.com/simple/ my-private-pkg

# Install with extras
pip install "fastapi[all]"
pip install "click[testing]"

# Install editable (development) mode
pip install -e ./my-package

# Constraints file
pip install -c constraints.txt

# Dry run
pip install --dry-run requests

# Install with platform specifier
pip install --platform win_amd64 --only-binary=:all: numpy

# User install (no sudo)
pip install --user jupyter

# Download without installing
pip download --dest ./wheels requests
```

### Advanced Examples
```bash
# Build wheels from source
pip wheel --no-deps -w ./dist my-package

# Install from VCS
pip install git+https://github.com/psf/requests.git
pip install git+https://github.com/psf/requests.git@v2.31.0
pip install git+https://github.com/psf/requests.git@main#egg=requests

# Install from local wheel
pip install ./dist/my_package-1.0.0-py3-none-any.whl

# Install with hash checking (Hashin mode)
pip install --require-hashes -r requirements.txt

# Package from a subdirectory
pip install "https://github.com/org/repo/archive/main.zip#subdirectory=packages/foo"

# Use pip as a Python module (when pip itself is broken)
python -m pip install --upgrade pip

# Install with no index (offline)
pip install --no-index --find-links=./wheels requests

# Reliable freeze with normalized names
pip freeze --exclude-editable --all
```

### Real-World Use Cases
- **Project setup** — `pip install -r requirements.txt` in CI/CD
- **Environment bootstrap** — installing pip itself in Docker images
- **Offline deployment** — downloading wheels on a build machine, transferring to air-gapped servers
- **CI/CD optimization** — caching `pip` cache to speed up builds
- **Private packages** — hosting internal packages on private PyPI servers (DevPI, Gemfury, AWS CodeArtifact)

### Common Mistakes
- Forgetting to activate a virtual environment before installing
- Using `pip install` with `sudo` instead of `--user` or a virtualenv
- Committing `pip freeze` output without filtering editable packages
- Not pinning transitive dependencies in requirements.txt
- Using `pip install -e .` without `setup.py` or `pyproject.toml`

### Best Practices
- Always use virtual environments (venv) per project
- Pin exact versions in requirements.txt for reproducible builds
- Use `pip freeze > requirements.txt` after verifying compatibility
- Separate dev and production dependencies (`requirements-dev.txt`)
- Use `pip install --upgrade pip` regularly
- Prefer `python -m pip` over bare `pip` in scripts
- Use hashes for supply-chain security (`--require-hashes`)
- Use pip-compile (pip-tools) for dependency resolution

### Performance Considerations
- `--only-binary=:all:` avoids building from source (faster, but may miss platform wheels)
- Using a wheel mirror (like `--find-links=./wheels`) avoids network
- `pip install` with no cache is slower; use `--no-cache-dir` only for space
- Using `Pipfile`/`Pipfile.lock` with pipenv can be slower than plain `requirements.txt`

### Interview Questions
1. What is the difference between `install` and `download` in pip?
2. How does pip resolve dependency conflicts?
3. What is a wheel and how is it different from a source distribution?
4. How would you install a package from a private Git repository?
5. What does `--editable` do in `pip install -e .`?

### Coding Challenges
- Write a script that compares two `requirements.txt` files and reports version differences
- Implement a minimal dependency resolver that handles version conflicts
- Build a tool that audits installed packages for known vulnerabilities (like `pip-audit`)

### Related Topics
- venv, pyproject.toml, setuptools, twine, conda, pip-tools

## requirements.txt
### What It Is
`requirements.txt` is a plain-text file listing all Python packages (with optional version specifiers) needed for a project. It is the standard way to declare dependencies for pip-based projects.

### Why It Is Important
`requirements.txt` enables reproducible environments — anyone can run `pip install -r requirements.txt` and get the exact same dependencies. It serves as documentation of the project's dependencies and simplifies CI/CD and deployment workflows.

### How It Works Internally
pip reads each line of `requirements.txt`, parses PEP 508 requirement specifiers, resolves dependencies against the configured index, and installs matching packages. Lines starting with `#` are comments. Option lines starting with `--` set pip flags. `-r` includes other requirements files. `-c` references constraints files.

### Syntax
```txt
package_name
package_name==1.2.3
package_name>=1.0,<2.0
package_name~=1.2      # compatible release (>=1.2,<2.0)
package_name>=1.0       # minimum version
-r base.txt             # include another file
-c constraints.txt      # constraints file
--index-url https://...
--extra-index-url https://...
-e ./local-package      # editable install
package_name[extra]     # extras
git+https://...#egg=package_name  # VCS
```

### Beginner Examples
```txt
# requirements.txt
flask==3.0.0
requests==2.31.0
gunicorn==21.2.0
psycopg2-binary==2.9.9
python-dotenv==1.0.0
```

```bash
pip install -r requirements.txt
```

### Intermediate Examples
```txt
# requirements.in (source for pip-compile)
flask
requests
pandas>=2.0,<3.0

# requirements-dev.txt
-r requirements.txt
pytest==7.4.3
pytest-cov==4.1.0
black==23.12.1
flake8==6.1.0
pre-commit==3.5.0

# constraints.txt (pins transitive deps)
urllib3<2.0
certifi>=2023.0.0
```

### Advanced Examples
```txt
# Full pinned requirements.txt with hashes
flask==3.0.0 \
    --hash=sha256:abc123... \
    --hash=sha256:def456...
requests==2.31.0 \
    --hash=sha256:789abc...

# Multi-platform conditional
platform_system=="Windows"  # comment
pywin32==306; sys_platform == "win32"

# VCS and local deps
-e git+https://github.com/myorg/my-lib.git@v1.0#egg=my-lib
./packages/local-helper

# Extras
celery[redis,gevent]==5.3.4
```

### Real-World Use Cases
- **Docker builds** — `COPY requirements.txt . && pip install -r requirements.txt`
- **CI pipelines** — caching based on requirements.txt hash
- **Deployments** — reproducible installations across environments
- **Onboarding** — new developers run a single command to set up

### Common Mistakes
- Using `>=` in requirements.txt (pulls latest on each install)
- Committing editable (`-e .`) installs in final requirements.txt
- Mixing `requirements.txt` with `Pipfile`/`Pipfile.lock` conventions
- Forgetting to regenerate after adding/removing dependencies
- Using `pip freeze > requirements.txt` in a global (non-virtual) environment

### Best Practices
- Pin exact versions for all dependencies including transitive ones
- Separate production vs. development dependencies
- Use pip-compile (pip-tools) to generate locked requirements from loose specs
- Regenerate requirements.txt when adding/updating dependencies
- Use `--require-hashes` for supply-chain integrity
- Include a `requirements.in` (source) and `requirements.txt` (locked)
- Keep comments explaining why specific pins/versions are used
- Run `pip check` after installation to verify dependency consistency

### Performance Considerations
- A large requirements.txt with many transitive deps increases resolution time
- Using constraints files can speed up resolution by narrowing the search space
- Frozen (`==`) requirements resolve faster than ranged (`>=`) specifiers
- Hash verification adds overhead but is negligible for normal use

### Interview Questions
1. What is the difference between a requirements file and a constraints file?
2. How would you manage dev-only dependencies with requirements.txt?
3. What are the security implications of hash-locked requirements?
4. How do environment markers work in requirements.txt?

### Coding Challenges
- Write a script that validates a requirements.txt file (syntax, resolvability)
- Build a diff tool that shows dependency changes between two requirements.txt files
- Create a generator that converts `pyproject.toml` dependencies to requirements.txt

### Related Topics
- pip-tools, pip-compile, pip-sync, constraints files, Pipfile

## PyPI publishing
### What It Is
PyPI (Python Package Index) is the official third-party software repository for Python. Publishing to PyPI means uploading a Python package so that anyone can install it with `pip install <package>`.

### Why It Is Important
Publishing to PyPI makes your package accessible to the entire Python community. It is the standard distribution channel for open-source Python libraries, tools, and frameworks.

### How It Works Internally
Publishing involves three steps: building (creating a distribution format — wheel and/or source distribution), registering (creating the package on PyPI), and uploading (sending files via HTTP to PyPI's API at `https://upload.pypi.org/legacy/`). PyPI validates metadata, checks for duplicates, and archives the files. The package then becomes available on the PyPI web interface and the JSON API.

### Syntax
```bash
# Build
python -m build

# Upload to Test PyPI
twine upload --repository-url https://test.pypi.org/legacy/ dist/*

# Upload to PyPI
twine upload dist/*

# With API token
twine upload --username __token__ --password pypi-XXXXXXXX dist/*
```

### Beginner Examples
```python
# pyproject.toml (Python 3.12+)
[build-system]
requires = ["setuptools>=68.0"]
build-backend = "setuptools.build_meta"

[project]
name = "my-package"
version = "0.1.0"
authors = [
    {name = "Your Name", email = "you@example.com"}
]
description = "A short description of my package"
readme = "README.md"
requires-python = ">=3.9"
license = {text = "MIT"}
classifiers = [
    "Programming Language :: Python :: 3",
    "License :: OSI Approved :: MIT License",
]
dependencies = [
    "requests>=2.28",
]

[project.urls]
Homepage = "https://github.com/you/my-package"
```

```bash
# Step 1: Install build tools
pip install build twine

# Step 2: Build distributions
python -m build

# Step 3: Upload to Test PyPI first
twine upload --repository-url https://test.pypi.org/legacy/ dist/*

# Step 4: Install from Test PyPI to verify
pip install --index-url https://test.pypi.org/simple/ my-package

# Step 5: Upload to real PyPI
twine upload dist/*
```

### Intermediate Examples
```python
# pyproject.toml with optional dependencies
[project]
name = "my-lib"
version = "0.2.0"
dependencies = ["click>=8.0"]
optional-dependencies = {dev = ["pytest", "black"], web = ["fastapi"]}

[project.scripts]
my-cli = "my_lib.cli:main"

[project.entry-points."myplugin"]
my_plugin = "my_lib.plugin"

[tool.setuptools.packages.find]
where = ["src"]
exclude = ["tests.*"]
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
# Configuration in ~/.pypirc
[distutils]
index-servers =
    pypi
    testpypi

[pypi]
username = __token__
password = pypi-XXXXXXXX

[testpypi]
repository = https://test.pypi.org/legacy/
username = __token__
password = pypi-XXXXXXXX
```

```python
# setup.py (traditional, still supported)
from setuptools import setup, find_packages

setup(
    name="my-package",
    version="0.1.0",
    packages=find_packages(where="src"),
    package_dir={"": "src"},
    install_requires=["requests>=2.28"],
    extras_require={
        "dev": ["pytest>=7.0", "black"],
        "web": ["flask>=2.0"],
    },
    entry_points={
        "console_scripts": [
            "my-cmd=my_package.cli:main",
        ],
    },
    python_requires=">=3.9",
)
```

### Real-World Use Cases
- **Open-source libraries** — publishing reusable libraries to the community
- **Internal tools** — hosting on private PyPI for company-internal packages
- **CLI tools** — distributing command-line applications via pip
- **Plugins** — publishing plugin packages for frameworks (e.g., pytest plugins)

### Common Mistakes
- Uploading without building (uploading source instead of wheel)
- Forgetting to bump version before publishing (PyPI rejects duplicate versions)
- Including `__pycache__` or `.pyc` files in distributions
- Not testing on Test PyPI before publishing to production PyPI
- Exposing API tokens in code or committing to version control
- Publishing with an overly broad license or missing license file

### Best Practices
- Always publish to Test PyPI first (`test.pypi.org`)
- Use Trusted Publishing (OIDC) for CI/CD instead of API tokens
- Use `twine check dist/*` to validate distributions before uploading
- Follow semantic versioning (semver) for release numbering
- Include a `README.md`, `LICENSE`, `CHANGELOG.md`
- Add classifiers for Python versions, license, and topic
- Use `python -m build` (not `setup.py sdist bdist_wheel` directly)
- Configure `.gitignore` to exclude `dist/`, `*.egg-info/`, `__pycache__/`
- Enable 2FA on your PyPI account
- Use Trusted Publishing with GitHub Actions for secure automated releases

### Performance Considerations
- Wheels are faster to install than source distributions (no compilation step)
- Universal wheels (`py3-none-any`) work everywhere — worth the effort
- Platform-specific wheels (for C extensions) require CI for each platform
- Binary wheels reduce user installation time significantly

### Interview Questions
1. What is the difference between `sdist` and `bdist_wheel`?
2. How do you specify package dependencies in pyproject.toml?
3. What is a "universal wheel" and when should you use it?
4. How does Trusted Publishing (OIDC) improve security for PyPI uploads?
5. What steps would you take to publish a package that contains C extensions?

### Coding Challenges
- Write a CI script that auto-increments version and publishes on tag push
- Build a tool that checks a package's metadata for common issues before release
- Create a minimal PyPI index server for internal package hosting

### Related Topics
- pyproject.toml, setuptools, twine, build, hatchling, flit, poetry
