# pip - Package installer, requirements.txt, PyPI

## Introduction

pip is the standard package manager for Python, used to install, update, and manage third-party packages from the Python Package Index (PyPI) and other package indexes. It is included by default with Python 3.4+ and is essential for managing project dependencies.

## Why It Is Important

pip enables code reuse by providing access to over 500,000 packages on PyPI. It automates dependency resolution, handles version conflicts, and integrates with virtual environments to keep projects isolated. Pip is the foundation of the Python packaging ecosystem.

## Syntax

```bash
pip install package_name
pip install package_name==1.2.3
pip install package_name>=1.0,<2.0
pip uninstall package_name
pip freeze
pip list
pip show package_name
pip install -r requirements.txt
```

## Examples

### Basic Installation and Uninstallation

```python
import subprocess
import sys

def run_pip(args):
    cmd = [sys.executable, "-m", "pip"] + args
    result = subprocess.run(cmd, capture_output=True, text=True)
    return result

# Install a package
result = run_pip(["install", "requests"])
print(f"Install success: {result.returncode == 0}")

# Show package info
result = run_pip(["show", "requests"])
print(result.stdout[:500] if result.stdout else "Package not found")

# Uninstall
result = run_pip(["uninstall", "requests", "-y"])
print(f"Uninstall success: {result.returncode == 0}")
```

### Working with requirements.txt

```python
import subprocess
import sys

def pip_freeze():
    result = subprocess.run(
        [sys.executable, "-m", "pip", "freeze"],
        capture_output=True, text=True
    )
    return result.stdout.strip().splitlines()

# Generate requirements.txt content
deps = pip_freeze()
sample_reqs = """requests==2.31.0
flask==3.0.0
numpy>=1.24.0,<2.0.0
pandas~=2.1.0
click>=8.0
"""

# Write requirements.txt
with open("requirements.txt", "w") as f:
    f.write(sample_reqs)
print("Created requirements.txt")

# Read back
with open("requirements.txt") as f:
    content = f.read()
print(f"Requirements file:\n{content}")

import os
os.remove("requirements.txt")
```

### Version Specifiers

```python
from packaging.specifiers import SpecifierSet
from packaging.version import Version

spec = SpecifierSet(">=1.0,<3.0")
versions = ["0.9", "1.0", "1.5", "2.0", "2.9", "3.0", "3.1"]

print(f"Versions matching {spec}:")
for v_str in versions:
    v = Version(v_str)
    matches = v in spec
    print(f"  {v_str}: {'✓' if matches else '✗'}")

# Compatible release (~=)
spec_tilde = SpecifierSet("~=1.2.3")
print(f"\nVersions matching ~=1.2.3:")
for v in ["1.2.3", "1.2.4", "1.3.0", "1.3.1", "2.0.0"]:
    print(f"  {v}: {Version(v) in spec_tilde}")

# Arbitrary equality
spec_arb = SpecifierSet("===1.2.3")
print(f"\n===1.2.3 matches 1.2.3: {Version('1.2.3') in spec_arb}")
```

### pip list and pip show

```python
import subprocess
import sys
import json

def pip_list():
    result = subprocess.run(
        [sys.executable, "-m", "pip", "list", "--format=json"],
        capture_output=True, text=True
    )
    return json.loads(result.stdout)

def pip_show(package):
    result = subprocess.run(
        [sys.executable, "-m", "pip", "show", package],
        capture_output=True, text=True
    )
    if result.returncode == 0:
        info = {}
        for line in result.stdout.strip().splitlines():
            if ": " in line:
                key, value = line.split(": ", 1)
                info[key.lower()] = value
        return info
    return None

packages = pip_list()
print(f"Installed packages: {len(packages)}")
for pkg in packages[:5]:
    print(f"  {pkg['name']} == {pkg['version']}")

pip_info = pip_show("pip")
if pip_info:
    print(f"\npip version: {pip_info.get('version', 'N/A')}")
    print(f"Requires: {pip_info.get('requires', 'N/A')}")
```

### Installing from different sources

```python
import subprocess
import sys

def pip_install(args):
    cmd = [sys.executable, "-m", "pip", "install"] + args
    result = subprocess.run(cmd, capture_output=True, text=True)
    return result

# From a requirements file
result = pip_install(["-r", "requirements.txt"])
print(f"Requirements install: {result.returncode == 0}")

# From a URL (VCS)
result = pip_install([
    "git+https://github.com/psf/requests.git@v2.31.0"
])
print(f"VCS install: {result.returncode == 0}")

# From a local wheel
result = pip_install(["--no-index", "package.whl"])
print(f"Local wheel install: {result.returncode == 0}")

# With constraints file
result = pip_install(["-c", "constraints.txt", "requests"])
print(f"Constrained install: {result.returncode == 0}")

# Editable mode for development
result = pip_install(["-e", "."])
print(f"Editable install: {result.returncode == 0}")
```

## Beginner Examples

```python
# Creating and using a requirements file
import subprocess
import sys
import os

def create_requirements():
    """Generate a requirements.txt from currently installed packages."""
    result = subprocess.run(
        [sys.executable, "-m", "pip", "freeze"],
        capture_output=True, text=True
    )
    packages = result.stdout.strip().splitlines()

    with open("requirements.txt", "w") as f:
        for pkg in packages:
            if "@" not in pkg:  # Skip editable packages
                f.write(pkg + "\n")
    print(f"Created requirements.txt with {len(packages)} packages")

def install_from_requirements():
    """Install packages from requirements.txt."""
    if not os.path.exists("requirements.txt"):
        print("No requirements.txt found")
        return

    result = subprocess.run(
        [sys.executable, "-m", "pip", "install", "-r", "requirements.txt"],
        capture_output=True, text=True
    )
    if result.returncode == 0:
        print("All packages installed successfully")
    else:
        print(f"Installation failed:\n{result.stderr}")

create_requirements()
install_from_requirements()
os.remove("requirements.txt")

# Check if a package is installed
def is_package_installed(package_name):
    result = subprocess.run(
        [sys.executable, "-m", "pip", "show", package_name],
        capture_output=True, text=True
    )
    return result.returncode == 0

for pkg in ["pip", "nonexistent_package_xyz"]:
    print(f"{pkg} installed: {is_package_installed(pkg)}")
```

## Intermediate Examples

```python
# Dependency tree analyzer
import subprocess
import sys
import json
from collections import defaultdict

def get_package_dependencies(package_name, depth=0, max_depth=3, visited=None):
    if visited is None:
        visited = set()

    if depth > max_depth or package_name in visited:
        return {}

    visited.add(package_name)
    info = pip_show(package_name)

    if not info:
        return {}

    requires = info.get("requires", "")
    dependencies = {}
    if requires:
        for dep in requires.split(", "):
            dep_name = dep.split(">")[0].split("<")[0].split("=")[0].split("[")[0].strip()
            if dep_name:
                dependencies[dep_name] = get_package_dependencies(
                    dep_name, depth + 1, max_depth, visited
                )

    return dependencies

def pip_show(package):
    result = subprocess.run(
        [sys.executable, "-m", "pip", "show", package],
        capture_output=True, text=True
    )
    if result.returncode == 0:
        info = {}
        for line in result.stdout.strip().splitlines():
            if ": " in line:
                key, value = line.split(": ", 1)
                info[key.lower()] = value
        return info
    return None

def print_tree(deps, indent=0):
    for pkg, subdeps in deps.items():
        print("  " * indent + f"├── {pkg}")
        if subdeps:
            print_tree(subdeps, indent + 1)

# Find outdated packages
def find_outdated():
    result = subprocess.run(
        [sys.executable, "-m", "pip", "list", "--outdated", "--format=json"],
        capture_output=True, text=True
    )
    if result.returncode == 0:
        outdated = json.loads(result.stdout)
        print(f"Outdated packages: {len(outdated)}")
        for pkg in outdated:
            print(f"  {pkg['name']}: {pkg['version']} -> {pkg['latest_version']}")
        return outdated
    return []

outdated = find_outdated()

# Generate dependency graph for pip itself
deps = get_package_dependencies("pip", max_depth=2)
print("\nDependency tree for pip:")
print_tree(deps)
```

## Advanced Examples

```python
# Custom pip mirror/registry management tool
import subprocess
import sys
import json
import os
from pathlib import Path
import hashlib
import urllib.request
import urllib.error
import time

class PipManager:
    def __init__(self, python_executable=None):
        self.python = python_executable or sys.executable

    def _run_pip(self, args):
        cmd = [self.python, "-m", "pip"] + args
        result = subprocess.run(cmd, capture_output=True, text=True)
        return result

    def install(self, package, version=None, extra_args=None):
        args = ["install"]
        if version:
            package = f"{package}=={version}"
        args.append(package)
        if extra_args:
            args.extend(extra_args)
        return self._run_pip(args)

    def install_from_index(self, package, index_url):
        return self.install(package, extra_args=["-i", index_url])

    def freeze(self):
        result = self._run_pip(["freeze"])
        if result.returncode == 0:
            return result.stdout.strip().splitlines()
        return []

    def list_packages(self, format="json"):
        result = self._run_pip(["list", f"--format={format}"])
        return result

    def download_wheels(self, packages, dest_dir="wheels"):
        dest = Path(dest_dir)
        dest.mkdir(exist_ok=True)
        args = ["download", "-d", str(dest)] + packages
        result = self._run_pip(args)
        downloaded = list(dest.iterdir()) if result.returncode == 0 else []
        return downloaded

    def install_from_wheels(self, wheel_dir="wheels"):
        wheel_path = Path(wheel_dir)
        if not wheel_path.exists():
            return None
        wheels = list(wheel_path.glob("*.whl"))
        if not wheels:
            return None
        args = ["install", "--no-index", f"--find-links={wheel_path}"] + [str(w) for w in wheels]
        return self._run_pip(args)

    def verify_integrity(self, package):
        result = self._run_pip(["download", "--no-deps", package, "--no-binary", ":all:"])
        return result.returncode == 0

    def batch_upgrade(self, packages=None):
        if packages is None:
            result = self._run_pip(["list", "--outdated", "--format=json"])
            if result.returncode != 0:
                return []
            outdated = json.loads(result.stdout)
            packages = [p["name"] for p in outdated]

        results = {}
        for pkg in packages:
            r = self._run_pip(["install", "--upgrade", pkg])
            results[pkg] = r.returncode == 0
        return results

    def get_package_hash(self, package, version=None):
        spec = f"{package}=={version}" if version else package
        result = self._run_pip(["hash", spec])
        if result.returncode == 0:
            return result.stdout.strip()
        return None

    def create_constraints_file(self, packages, filename="constraints.txt"):
        constraints = []
        for pkg in packages:
            result = self._run_pip(["show", pkg])
            if result.returncode == 0:
                for line in result.stdout.splitlines():
                    if line.startswith("Version:"):
                        ver = line.split(":")[1].strip()
                        constraints.append(f"{pkg}=={ver}")
                        break
        with open(filename, "w") as f:
            f.write("\n".join(constraints))
        return filename

    def install_with_constraints(self, package, constraints_file="constraints.txt"):
        return self._run_pip(["install", "-c", constraints_file, package])

    def export_environment(self, output="environment.yml", include_editable=False):
        packages = self.freeze()
        filtered = []
        for pkg in packages:
            if not include_editable and "-e " in pkg:
                continue
            filtered.append(pkg)
        with open(output, "w") as f:
            f.write("\n".join(filtered) + "\n")
        return output

manager = PipManager()
print(f"Python: {manager.python}")

frozen = manager.freeze()
print(f"Frozen packages: {len(frozen)}")

result = manager.list_packages()
if result.returncode == 0:
    packages = json.loads(result.stdout)
    print(f"Installed: {len(packages)} packages")

exported = manager.export_environment("test_export.txt")
print(f"Environment exported to {exported}")
os.remove("test_export.txt")

# Resolve conflict simulation
print("\nSimulating dependency resolution...")
deps_to_check = ["requests", "flask", "numpy"]
for dep in deps_to_check:
    result = manager._run_pip(["show", dep])
    if result.returncode == 0:
        for line in result.stdout.splitlines():
            if "Requires:" in line:
                requires = line.split(":", 1)[1].strip()
                if requires:
                    print(f"  {dep} requires: {requires}")
                else:
                    print(f"  {dep} requires: nothing")
```

## Real-World Use Cases

- **CI/CD pipelines**: Using `pip install -r requirements.txt` and `pip install -e .` in Docker builds
- **Microservices**: Pinning exact versions with `pip freeze > requirements.txt` for reproducible deployments
- **Monorepos**: Using `pip install -e ./packages/*` for local package development
- **Air-gapped environments**: Downloading wheels with `pip download` and transferring offline
- **Security audits**: Using `pip audit` or `pip list --outdated` to find vulnerable dependencies
- **Platform-specific builds**: Using `--platform` and `--python-version` flags for cross-compilation

## Common Mistakes

- Forgetting to activate a virtual environment before running pip install
- Using `pip freeze` without first activating the correct virtual environment
- Pinning dependencies too loosely (`>=`) causing unexpected breakage on upgrades
- Pinning dependencies too tightly (`==`) preventing security updates
- Committing `.pyc` files and `__pycache__` to version control
- Installing packages system-wide with `sudo pip install` instead of using a venv
- Omitting `-r` when installing from a requirements file
- Using `pip` instead of `python -m pip` in environments with multiple Python versions
- Not using `--no-deps` when you want precise control over dependency versions
- Confusing `constraints.txt` (optional version limits) with `requirements.txt` (required installs)

## Best Practices

- Always use a virtual environment; never install packages system-wide
- Pin exact versions in requirements files for reproducible builds
- Use `python -m pip` instead of bare `pip` to ensure the correct interpreter
- Separate dev/test dependencies from production: `requirements-dev.txt` alongside `requirements.txt`
- Use `pip-compile` (pip-tools) or `poetry` for robust dependency resolution
- Regularly run `pip list --outdated` and plan upgrades
- Use hashes for security: `pip install --require-hashes -r requirements.txt`
- Prefer wheels (`.whl`) over source distributions for faster, more reliable installs
- Use `pip cache purge` periodically to reclaim disk space in CI

## Interview Questions

1. **Q**: What is the difference between `requirements.txt` and `constraints.txt`?
   **A**: `requirements.txt` lists packages to install (with `pip install -r`), while `constraints.txt` only restricts versions without forcing installation, applied via `pip install -c`.

2. **Q**: How does pip resolve dependency conflicts?
   **A**: pip uses a backtracking resolver (introduced in pip 20.3) that explores dependency trees and backtracks on conflicts to find a compatible set of versions.

3. **Q**: What is the difference between `pip install` and `pip download`?
   **A**: `pip install` downloads and installs packages. `pip download` only downloads the distribution files (wheels/sdists) without installing them, useful for offline transfers.

4. **Q**: Explain the wheel format and its advantages over sdists.
   **A**: Wheels (`.whl`) are pre-built distribution packages that install faster because they skip the build step. They also include metadata for faster dependency resolution.

5. **Q**: How does `pip install -e .` work?
   **A**: The `-e` (editable) flag creates a symlink-like link to the source directory, so changes to the source code are immediately reflected without reinstalling.

## Coding Challenges

1. **Dependency Inspector**: Write a script that takes a package name and prints its complete dependency tree (transitive dependencies) using `pip show` and recursion.

2. **Version Conflict Detector**: Given multiple requirements.txt files, write a tool that identifies version conflicts across them and suggests compatible ranges.

3. **Offline Package Bundler**: Create a tool that takes a list of packages and their dependencies, downloads all wheels, and generates an install script for air-gapped environments.

4. **Outdated Package Reporter**: Build a CLI tool that lists all outdated packages, groups them by major/minor/patch, and estimates the risk level of each upgrade.

5. **Custom Package Index Mirror**: Implement a simple PyPI mirror that proxies requests, caches packages locally, and provides a fallback index URL.

## Summary

pip is the cornerstone of Python's package management ecosystem. It enables access to hundreds of thousands of community packages, handles complex dependency resolution, and integrates seamlessly with virtual environments. Mastering pip—including requirements files, version specifiers, constraints, and wheel management—is essential for professional Python development.

## Related Topics

Virtual Environments (56.x), Creating Packages (58.x), Standard Library (54.x), Import System (57.x)
