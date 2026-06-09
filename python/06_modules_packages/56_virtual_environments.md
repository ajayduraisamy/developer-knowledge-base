# Virtual Environments - venv, virtualenv, poetry, uv
## Introduction
Virtual environments create isolated Python environments per project, preventing dependency conflicts between projects. Each environment has its own Python interpreter, site-packages directory, and scripts. Python provides the built-in `venv` module; third-party tools like `virtualenv`, `poetry`, and `uv` offer additional features.

## venv
### What It Is
`venv` is Python's built-in module (since Python 3.3) for creating lightweight virtual environments. It is included in the standard library and requires no additional installation.

### Why It Is Important
`venv` is zero-dependency — available on every Python 3 installation. It is the simplest, most portable way to create isolated environments. Tools like `pip` and `setuptools` are automatically installed in each environment.

### How It Works Internally
`python -m venv <dir>` creates a directory containing:
- A `pyvenv.cfg` configuration file pointing to the base Python installation
- A `Scripts/` (Windows) or `bin/` (Unix) directory with `python`, `pip`, and activation scripts
- A `Lib/site-packages/` directory for installed packages
- Symlinks/copies of the Python executable

When activated, the `PATH` environment variable is modified to put the environment's `Scripts`/`bin` directory first, ensuring `python` and `pip` resolve to the environment's versions. Deactivation restores the original `PATH`.

### Syntax
```bash
# Create
python -m venv .venv
python -m venv --system-site-packages .venv  # include base Python's packages
python -m venv --prompt myproject .venv       # custom prompt

# Activate (Windows)
.venv\Scripts\activate

# Activate (Unix/Mac)
source .venv/bin/activate

# Deactivate
deactivate

# Delete
rm -rf .venv   # Unix
Remove-Item -Recurse -Force .venv  # Windows PowerShell
```

### Beginner Examples
```bash
# Create and activate
python -m venv myenv
# Windows:
myenv\Scripts\activate
# Linux/Mac:
source myenv/bin/activate

# Verify activation
which python   # shows myenv/bin/python (Unix)
where python   # shows myenv\Scripts\python.exe (Windows)
python -c "import sys; print(sys.executable)"

# Install packages inside environment
pip install requests flask

# Deactivate
deactivate
```

### Intermediate Examples
```bash
# Create environment with specific Python version
py -3.11 -m venv .venv       # Windows with Python launcher
python3.11 -m venv .venv     # Unix

# Check what's installed
.venv\Scripts\python -m pip list
python -m pip list  # after activation

# Run a script directly without activation
.venv\Scripts\python -c "import sys; print(sys.path)"

# Export and recreate
pip freeze > requirements.txt
python -m venv newenv
newenv\Scripts\python -m pip install -r requirements.txt
```

### Advanced Examples
```python
# Creating venv programmatically
import venv
import subprocess

def create_venv(path, system_site_packages=False):
    builder = venv.EnvBuilder(
        system_site_packages=system_site_packages,
        clear=True,
        symlinks=True,
        upgrade=True,
    )
    builder.create(path)

    # Install packages after creation
    pip_path = f"{path}/Scripts/pip" if sys.platform == "win32" else f"{path}/bin/pip"
    subprocess.check_call([pip_path, "install", "requests", "flask"])

create_venv("./project-venv")
```

```python
# Custom venv with post-setup hooks
class CustomEnvBuilder(venv.EnvBuilder):
    def post_setup(self, context):
        """Install dev packages after environment creation."""
        pip_cmd = [
            context.env_exe, "-m", "pip", "install",
            "pytest", "black", "mypy",
        ]
        subprocess.check_call(pip_cmd)

builder = CustomEnvBuilder(with_pip=True)
builder.create("./dev-venv")
```

### Real-World Use Cases
- **Every Python project** — isolating dependencies per project
- **CI/CD** — creating fresh environments for each build
- **Testing** — testing against multiple Python versions
- **Teaching** — providing clean environments for students

### Common Mistakes
- Committing the `.venv` directory to version control
- Forgetting to activate the environment before installing
- Using `pip install` with `--user` inside an activated venv
- Renaming or moving the `.venv` directory (paths are hardcoded in scripts)
- Creating the venv in a location with spaces in the path (Windows issues)

### Best Practices
- Always create venv inside the project directory (e.g., `.venv`)
- Add `.venv/` to `.gitignore`
- Activate before running `pip install` or `python` commands
- Use `pip freeze > requirements.txt` to document dependencies
- Use `--prompt` to make terminal prompts meaningful
- Never use `sudo pip install` — always use a virtual environment
- Prefer `.venv` as the directory name (standard convention)

### Performance Considerations
- `venv` creates symlinks on Unix (fast) and copies on Windows (slower)
- Packages installed inside venv consume disk space per environment
- CI systems should cache the venv directory between runs
- Creating many venvs with large packages (torch, tensorflow) can use significant disk space

### Interview Questions
1. How does `venv` isolate packages from the system Python?
2. What is the difference between `venv` and `--system-site-packages`?
3. How does activation work at the shell level?
4. Can you move or rename a virtual environment after creation?

### Coding Challenges
- Write a script that finds all `.venv` directories in a project tree
- Build a command that recreates a virtual environment from a requirements file
- Implement a Python script that determines whether it is running inside a virtual environment

### Related Topics
- pip, virtualenv, pyenv, poetry, pipenv, conda

## virtualenv
### What It Is
`virtualenv` is a third-party tool predating `venv` that creates isolated Python environments. It supports Python 2.7+ and offers more features than `venv`, including faster creation, seed packages, and extended configuration.

### Why It Is Important
`virtualenv` works with Python 2 (legacy systems), supports creating environments for other Python versions than the one running `virtualenv`, and provides `--system-site-packages` control, pip bootstrapping, and `app-data` seeding for speed.

### How It Works Internally
`virtualenv` (version 20+) uses an `app-data` seed mechanism: it stores base packages (`pip`, `setuptools`, `wheel`) in a user-level cache directory (`~/.local/virtualenv/`). When creating an environment, it copies (or symlinks) these seeds rather than downloading them fresh. This makes subsequent environment creation much faster than `venv`. It also uses `importlib.resources` for pip installation.

### Syntax
```bash
pip install virtualenv

virtualenv .venv
virtualenv -p python3.11 .venv     # specific Python version
virtualenv --system-site-packages .venv
virtualenv --no-pip .venv          # exclude pip
virtualenv --seeder pip           # seed method
virtualenv --clear .venv          # clear existing
```

### Beginner Examples
```bash
pip install virtualenv

# Create a virtual environment for a specific Python version
virtualenv -p python3.10 myenv

# Activate (same as venv)
myenv\Scripts\activate    # Windows
source myenv/bin/activate # Unix

# Check Python version
python --version
# Should show 3.10.x

# Install packages
pip install django

# Deactivate
deactivate
```

### Intermediate Examples
```bash
# Create with custom seed packages
virtualenv --seed setuptools pip myenv

# Create without pip (for minimal environments)
virtualenv --no-pip --no-setuptools --no-wheel minimal-env

# Create environment for a different Python (must be installed)
virtualenv -p /usr/local/bin/python3.8 py38-env

# Upgrade an existing environment
virtualenv --upgrade .venv

# Relocatable (experimental)
virtualenv --relocatable .venv

# Verbose output
virtualenv -vvv .venv
```

### Advanced Examples
```python
# Programmatic use of virtualenv
import virtualenv

# Create environment programmatically
virtualenv.cli_run([".venv", "--python", "python3.11", "--verbose"])

# Or use the session API
from virtualenv import session_via_cli
from virtualenv.run import run_virtualenv

result = run_virtualenv({
    "dest": "./custom-env",
    "python": "python3.11",
    "seeders": ["pip"],
    "activators": ["bash", "batch"],
    "clear": True,
})
```

### Real-World Use Cases
- **Legacy Python 2 projects** — `virtualenv` supports Python 2.7
- **Cross-version development** — testing against multiple Python versions
- **CI/CD acceleration** — `app-data` caching makes environment creation faster
- **Tools like tox** — tox uses virtualenv under the hood

### Common Mistakes
- Installing `virtualenv` globally instead of in a user or project environment
- Using `virtualenv` when `venv` suffices (adding an unnecessary dependency)
- Confusing `virtualenv` Python version requirements with project requirements

### Best Practices
- Use `virtualenv` only when you need features missing from `venv` (multi-version, Python 2)
- Use `virtualenv --help` to see all configuration options
- Consider `virtualenv` for CI where speed matters
- Keep `virtualenv` itself up to date (`pip install --upgrade virtualenv`)

### Performance Considerations
- `virtualenv` 20+ creates environments 2-5x faster than `venv` due to `app-data` seeding
- The `app-data` cache grows over time; can be cleared via `virtualenv --clear-app-data`
- Network download is avoided for seed packages on subsequent creations

### Interview Questions
1. What advantages does `virtualenv` have over the built-in `venv`?
2. How does the `app-data` seed mechanism work?
3. Can `virtualenv` create environments for Python versions different from the host?

### Coding Challenges
- Write a script that benchmarks `venv` vs `virtualenv` creation time over 10 runs
- Build a tool that lists all virtual environments on a system by scanning directories

### Related Topics
- venv, tox, nox, pip, pyenv

## poetry
### What It Is
Poetry is a modern dependency management and packaging tool for Python. It combines virtual environment management, dependency resolution, build system, and PyPI publishing in a single tool, using `pyproject.toml` as the single configuration file.

### Why It Is Important
Poetry solves the split between `pip`/`setuptools`/`twine`/`pipenv` by providing a unified workflow. It uses a deterministic resolver, generates a `poetry.lock` file for reproducible installs, and handles virtual environments automatically.

### How It Works Internally
Poetry's resolver uses the pubgrub algorithm (same as Dart's pub) for dependency resolution — it handles complex dependency graphs better than pip's backtracking resolver. Poetry creates and manages virtual environments inside its cache directory (`~/AppData/Local/pypoetry` on Windows, `~/.cache/pypoetry` on Linux). It reads dependencies from `pyproject.toml` under `[tool.poetry.dependencies]`, resolves them, and writes `poetry.lock` with exact versions and hashes.

### Syntax
```bash
poetry new my-project          # create new project
poetry init                    # interactive init in existing dir
poetry add requests            # add dependency
poetry add --dev pytest        # dev dependency
poetry remove requests         # remove dependency
poetry install                 # install from lock file
poetry update                  # update deps, rewrite lock
poetry shell                   # spawn shell in venv
poetry run python script.py    # run in venv
poetry build                   # build sdist and wheel
poetry publish                 # publish to PyPI
poetry env info                # show venv info
```

### Beginner Examples
```bash
# Install poetry
pipx install poetry

# Create a new project
poetry new my-project
cd my-project

# Add dependencies
poetry add requests
poetry add --dev pytest pytest-cov

# Install from existing project
poetry install

# Run a script in the environment
poetry run python -c "import requests; print('ok')"

# Activate shell
poetry shell
# Now you're inside the venv
```

```toml
# pyproject.toml (generated by poetry)
[tool.poetry]
name = "my-project"
version = "0.1.0"
description = ""
authors = ["Your Name <you@example.com>"]
readme = "README.md"

[tool.poetry.dependencies]
python = "^3.10"
requests = "^2.31.0"

[tool.poetry.group.dev.dependencies]
pytest = "^7.4.0"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

### Intermediate Examples
```bash
# Add with version constraints
poetry add "flask>=2.3,<3.0"
poetry add "numpy@latest"
poetry add "click@^8.0"

# Add from Git
poetry add "git+https://github.com/psf/requests.git"
poetry add "git+https://github.com/psf/requests.git#v2.31.0"

# Update specific package
poetry update requests

# Export to requirements.txt
poetry export --without-hashes --output requirements.txt
poetry export --with dev --output requirements-dev.txt

# Manage environments
poetry env list
poetry env info --path
poetry env remove 3.10
poetry env use python3.11

# Build and publish
poetry build
poetry publish --dry-run
poetry publish
```

```toml
# Dependency groups and optional dependencies
[tool.poetry.dependencies]
python = "^3.10"

[tool.poetry.group.web.dependencies]
flask = "^2.3"
redis = "^5.0"

[tool.poetry.group.data.dependencies]
pandas = "^2.0"
numpy = "^1.25"

[tool.poetry.group.dev.dependencies]
pytest = "^7.4"
black = "^23.0"
mypy = "^1.6"

[tool.poetry.scripts]
my-cli = "my_project.cli:main"

[tool.poetry.urls]
"Bug Tracker" = "https://github.com/user/project/issues"
```

### Advanced Examples
```toml
# pyproject.toml with advanced features
[tool.poetry]
name = "advanced-project"
version = "0.3.0"
description = "Advanced project with extras"
authors = ["Dev <dev@example.com>"]
license = "MIT"
readme = "README.md"
homepage = "https://github.com/user/project"
repository = "https://github.com/user/project"
documentation = "https://docs.example.com"
keywords = ["python", "cli"]
classifiers = [
    "Development Status :: 4 - Beta",
    "Intended Audience :: Developers",
]

[tool.poetry.dependencies]
python = ">=3.10,<3.13"
# Extras
# Install with: poetry add my-package[extra1,extra2]
[tool.poetry.extras]
extras = ["extra1", "extra2"]

[tool.poetry.group.dev.dependencies]
pytest = "^7.4"

[tool.poetry.group.docs.dependencies]
sphinx = "^7.2"

[[tool.poetry.source]]
name = "private-pypi"
url = "https://private-pypi.example.com/simple/"
default = false  # only use for packages not found on PyPI
```

```yaml
# .github/workflows/ci.yml with poetry
name: CI
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - run: pipx install poetry
      - run: poetry install --with dev
      - run: poetry run pytest
```

### Real-World Use Cases
- **Open-source packages** — unified build, publish, and dependency management
- **Projects with complex dependency resolution** — Poetry's resolver is more reliable than pip's
- **Team projects** — `poetry.lock` ensures reproducible environments
- **CI/CD** — fast installs from lock file, caching support

### Common Mistakes
- Running `poetry add` outside a project directory
- Committing both `poetry.lock` and `Pipfile.lock` or `requirements.txt`
- Using `pip install` inside a Poetry-managed virtual environment
- Forgetting to run `poetry lock` after editing `pyproject.toml` directly
- Mixing `[tool.poetry.dependencies]` with `[project.dependencies]` (different standards)

### Best Practices
- Use `pipx install poetry` to install poetry globally
- Always commit `poetry.lock` to version control (for applications)
- Use `poetry export` to generate `requirements.txt` for Docker builds
- Use dependency groups (`[tool.poetry.group]`) for dev, docs, test deps
- Run `poetry update` regularly to keep dependencies current
- Use `poetry check` to validate `pyproject.toml`
- For libraries, define version ranges (not exact pins) in dependencies

### Performance Considerations
- Poetry's resolver is slower than pip's for complex dependency graphs but more accurate
- `poetry.lock` makes subsequent `poetry install` very fast
- Poetry caches downloaded packages, speeding up repeated operations
- Using `--no-root` skips installing the project itself (useful in CI for cache)

### Interview Questions
1. How does Poetry differ from pip + virtualenv?
2. What is the purpose of `poetry.lock`?
3. How does Poetry handle dev dependencies?
4. What algorithm does Poetry use for dependency resolution?
5. How do you publish a package using Poetry?

### Coding Challenges
- Write a CI script that caches Poetry's virtual environment between runs
- Build a tool that converts a `requirements.txt` to a `pyproject.toml` for Poetry
- Create a GitHub Action that publishes to PyPI using Poetry on tagged releases

### Related Topics
- pyproject.toml, pip, pipenv, flit, hatch, pdm

## uv
### What It Is
`uv` is an extremely fast Python package manager written in Rust. It serves as a drop-in replacement for pip, pip-tools, and virtualenv, with built-in support for dependency resolution, environment management, and script running.

### Why It Is Important
`uv` is 10-100x faster than pip due to its Rust implementation and advanced caching. It reduces CI times, improves developer experience, and supports drop-in compatibility (`uv pip install` works exactly like `pip install`). Created by Astral (the Ruff authors), it is rapidly gaining adoption.

### How It Works Internally
uv uses:
- A global content-addressable cache (similar to Docker layers) for downloaded packages
- Parallel HTTP downloads via reqwest
- A SAT-based resolver written in Rust
- Symlinks/hardlinks from cache to virtual environments (no copying)
- Lock file (uv.lock) with universal resolution (supports multiple platforms in one lock)

It implements the PEP standards directly without delegating to pip's internals.

### Syntax
```bash
uv venv                # create virtual environment
uv pip install flask   # drop-in pip replacement
uv pip sync            # sync environment to requirements
uv lock                # generate uv.lock
uv sync                # sync environment from lock
uv run script.py       # run in environment
uv add requests        # add dependency
uv remove requests     # remove dependency
uv tool install ruff   # install CLI tool (like pipx)
uv build               # build package
uv publish             # publish to PyPI
```

### Beginner Examples
```bash
# Install uv
pip install uv
# Or standalone
curl -LsSf https://astral.sh/uv/install.sh | sh
# Windows (PowerShell)
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"

# Create a virtual environment
uv venv
# Activate (standard activation)
source .venv/bin/activate

# Install packages (same as pip)
uv pip install flask
uv pip install -r requirements.txt

# Freeze
uv pip freeze

# All-in-one: create env + sync + run
uv run script.py
```

### Intermediate Examples
```bash
# uv.lock based workflow (modern)
uv init my-project
cd my-project
echo 'print("hello")' > main.py
uv add requests
# This creates pyproject.toml, uv.lock, .venv
uv run main.py

# Platform-specific resolution
uv pip install --python-platform linux --only-binary :all: numpy

# Export uv.lock to requirements format
uv export --format requirements-txt > requirements.txt

# Use as pip replacement in existing workflows
uv pip compile requirements.in -o requirements.txt
uv pip sync requirements.txt

# Workspace support
uv add --dev my-dep

# Tool management (like pipx)
uv tool install black
uv tool run black --help
uv tool upgrade --all
```

### Advanced Examples
```yaml
# Dockerfile with uv for optimized builds
FROM python:3.11-slim

# Install uv
COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/uv

# Copy dependency files
COPY pyproject.toml uv.lock ./

# Sync dependencies (uses Docker layer caching)
RUN uv sync --frozen --no-dev --no-install-project

# Copy source
COPY . .

# Sync with project install
RUN uv sync --frozen --no-dev
```

```bash
# Advanced uv workflows

# Multiple Python versions
uv python list               # list available Python versions
uv python install 3.12       # install a Python version
uv venv --python 3.12        # create env with specific version

# Workspace (monorepo) support
# pyproject.toml
# [tool.uv.workspace]
# members = ["packages/*"]

uv add --workspace some-dep  # add to workspace root
uv add packages/foo --editable  # workspace member dependency

# Caching
uv cache dir                 # show cache location
uv cache clean               # clear cache
uv cache prune               # remove stale entries

# Lock file verification
uv lock --check              # verify lock is up-to-date
uv sync --frozen             # fail if lock outdated

# Dependency tree
uv tree                      # show dependency tree
uv tree --invert             # show reverse dependency view

# Custom indexes
uv pip install --index-url https://private.example.com/simple/ my-pkg
```

### Real-World Use Cases
- **CI/CD optimization** — reducing build times from minutes to seconds
- **Large monorepos** — fast dependency resolution for hundreds of packages
- **Docker builds** — multi-stage builds with uv cache mounts
- **Migration targets** — teams migrating from pip/poetry for performance
- **Script runners** — `uv run script.py` with automatic dependency handling

### Common Mistakes
- Expecting `uv pip install` to understand pip arguments that modify environment state
- Not using `uv.lock` and relying only on `requirements.txt` (missing uv's benefits)
- Confusing `uv add` (which modifies pyproject.toml) with `uv pip install` (which doesn't)
- Running `uv sync` without a lock file when one exists

### Best Practices
- Use `uv.lock` for reproducible builds across environments
- Use `uv sync` instead of `uv pip install` for project-managed workflows
- Use `uv export` to generate requirements.txt for Docker or tools that don't support uv
- Use `uv python install` to manage multiple Python versions
- Use uv in CI with cache mounts for maximum speed
- Adopt uv's script runner (`uv run`) for one-off scripts with dependencies declared in a header comment
- For Docker builds, copy `pyproject.toml` and `uv.lock` before source code for layer caching

### Performance Considerations
- uv is 10-100x faster than pip for cold installs, 2-10x for hot installs
- The global cache avoids re-downloading and re-building packages
- Parallel downloads improve speed for packages with many dependencies
- uv's resolver is significantly faster than Poetry's for large dependency graphs
- Content-addressable storage means packages are stored only once on disk, even across projects

### Interview Questions
1. How does uv achieve its performance advantage over pip?
2. What is the difference between `uv pip install` and `uv add`?
3. How does uv handle caching differently from pip?
4. What is a "universal lock file" and why is it useful?
5. How would you migrate a project from pip+venv to uv?

### Coding Challenges
- Create a Dockerfile that uses uv to install dependencies with optimal caching
- Write a script that benchmarks `pip install` vs `uv pip install` for a large requirement set
- Build a GitHub Action workflow that uses uv for Python project CI

### Related Topics
- pip, pip-tools, virtualenv, poetry, ruff (same author), cargo (inspiration)
