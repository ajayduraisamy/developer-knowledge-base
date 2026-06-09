# Installing Python - Downloading, PATH setup, venv, pip

## Introduction

Installing Python correctly is the foundation of a productive development environment. The process involves downloading the appropriate installer for your operating system, configuring the PATH environment variable to run Python from any terminal, and understanding package management with pip and virtual environments with venv. Python 3.x is the current standard; Python 2 reached end-of-life on January 1, 2020.

## Downloading

### What It Is
Downloading Python means obtaining the official installer or source distribution from python.org or using platform-specific package managers. Python releases follow a annual cadence (e.g., 3.12, 3.13) with each version receiving approximately 5 years of security support.

### Why It Is Important
Using the correct installer ensures you get a stable, secure Python build. The official installer includes the standard library, pip, and IDLE. Third-party distributions (Anaconda, ActivePython) include additional packages but may lag behind official releases.

### How It Works Internally
The CPython source code is compiled into a platform-specific binary. Windows installers (.exe, .msi) package the Python DLL, standard library (.py and .pyc files), and the Python executable. macOS installers use .pkg format. Linux distributions compile from source or use pre-built .deb/.rpm packages. Each build embeds the Python version, build date, and compiler information accessible via `sys.version`.

### Syntax
```bash
# Windows — download from python.org and run the installer
# IMPORTANT: Check "Add Python to PATH" during installation

# macOS — using Homebrew
brew install python@3.12

# Linux — Ubuntu/Debian
sudo apt update
sudo apt install python3 python3-pip python3-venv

# Linux — Fedora/RHEL
sudo dnf install python3 python3-pip

# Verify installation
python3 --version
python3 -c "import sys; print(sys.version)"
```

### Beginner Examples
```python
# Verify installation by running a test script
import sys
import platform

def check_installation():
    print(f"Python {sys.version_info.major}.{sys.version_info.minor}.{sys.version_info.micro}")
    print(f"Running on: {platform.system()} {platform.release()}")
    print(f"Executable: {sys.executable}")
    
    # Check pip
    try:
        import pip
        print(f"pip version: {pip.__version__}")
    except ImportError:
        print("pip not available")

if __name__ == "__main__":
    check_installation()
```

### Intermediate Examples
```python
import subprocess
import sys

# Programmatically find Python installation
def find_python_executables():
    """Find all Python executables on the system."""
    import os
    paths = []
    
    if sys.platform == "win32":
        # Check common Windows locations
        for drive in ["C:", "D:"]:
            for ver in ["3.12", "3.11", "3.10", "3.9"]:
                path = f"{drive}\\Python{ver.replace('.', '')}\\python.exe"
                if os.path.exists(path):
                    paths.append(path)
    else:
        # Unix: use which
        for name in ["python3", "python3.12", "python3.11", "python3.10"]:
            result = subprocess.run(["which", name], capture_output=True, text=True)
            if result.returncode == 0:
                paths.append(result.stdout.strip())
    
    return paths

print(f"Python executables found: {find_python_executables()}")
```

### Advanced Examples
```python
#!/usr/bin/env python3
"""Multi-version Python download and setup script (conceptual)."""
import os
import platform
import subprocess
import sys
from pathlib import Path
from urllib.request import urlopen
import json

class PythonInstaller:
    def __init__(self, version: str = "3.12.0"):
        self.version = version
        self.system = platform.system()
        self.download_url = (
            f"https://www.python.org/ftp/python/{version}/"
        )
    
    def get_latest_stable(self) -> str:
        """Fetch latest Python version from API."""
        url = "https://endoflife.date/api/python.json"
        with urlopen(url) as resp:
            releases = json.loads(resp.read())
            return releases[0]["latest"]
    
    def get_download_url(self) -> str:
        """Get platform-specific download URL."""
        arch = platform.machine()
        if self.system == "Windows":
            suffix = "amd64.exe" if "64" in arch else "win32.exe"
        elif self.system == "Darwin":
            suffix = "macos11.pkg"
        else:
            suffix = "linux-x86_64.tar.xz"
        return f"{self.download_url}python-{self.version}-{suffix}"
    
    def install(self):
        """Run the installer (platform-specific)."""
        url = self.get_download_url()
        print(f"Download from: {url}")
        # In production, would download and run the installer
        return True

if __name__ == "__main__":
    installer = PythonInstaller()
    print(f"Latest stable: {installer.get_latest_stable()}")
    print(f"Platform: {installer.system}")
```

### Real-World Use Cases
- **Containers**: Multi-stage Docker builds with base Python images
- **CI/CD**: Matrix testing across Python 3.9, 3.10, 3.11, 3.12
- **Education**: Standardized installation across classroom machines
- **Enterprise**: Managed Python deployments via SCCM, Jamf, or Ansible
- **Embedded**: MicroPython for IoT devices

### Common Mistakes
```python
# Mistake 1: Not adding Python to PATH on Windows
# Error: 'python' is not recognized as an internal or external command
# Fix: Re-run installer and select "Add Python to PATH"

# Mistake 2: Using python instead of python3 on Unix
# python -> Python 2.7 (deprecated)
# python3 -> Python 3.x (correct)

# Mistake 3: Installing 32-bit Python on 64-bit Windows
# Causes memory limit of ~2GB for data processing
# Fix: Download the 64-bit (x86-64) installer

# Mistake 4: Using the Windows Store Python (limited file system access)
# Better: Use python.org installer for full capabilities
```

### Best Practices
- Always download from python.org (not third-party sites)
- Install the latest stable minor version (e.g., 3.12.x, not 3.13.0a1)
- On Windows, select "Add Python to PATH" and "Install for all users"
- Use 64-bit Python unless you have a specific reason for 32-bit
- Verify the installer signature with GPG for security
- Document Python version requirements in your project README

### Performance Considerations
- Python 3.11+ is 10-60% faster than Python 3.10
- Each minor version typically improves performance (e.g., 3.12 is faster than 3.11)
- Downloading and installing Python takes 1-5 minutes
- The standard library adds ~30-50MB to installation size
- Embedded distributions (Embeddable Python for Windows) are ~10MB

### Interview Questions
1. How do you install Python on Windows/macOS/Linux?
2. What is the difference between Python 2 and Python 3?
3. How do you verify a Python installation?
4. Why should you add Python to PATH?
5. What is the difference between 32-bit and 64-bit Python?
6. How do you install multiple Python versions on the same machine?
7. What is pyenv and why would you use it?
8. How do you check if pip is installed?
9. What are the system requirements for Python?
10. How do you uninstall Python cleanly?

### Coding Challenges
```python
# Challenge: Installation verification script
import sys

def verify_python_install():
    checks = [
        ("Python 3.x", sys.version_info.major >= 3),
        ("Has importlib", hasattr(sys, "meta_path")),
        ("Has pip", True),  # Would need subprocess check
        ("Has venv", True),  # importlib.util.find_spec("venv")
    ]
    for name, passed in checks:
        status = "✓" if passed else "✗"
        print(f"{status} {name}")

verify_python_install()
```

### Related Topics
- Python Version Management (pyenv)
- PATH Environment Variables
- Package Management with pip
- Virtual Environments
- Docker for Python

## PATH Setup

### What It Is
PATH is an environment variable that tells the operating system where to find executable programs. Adding Python to PATH allows you to run `python`, `pip`, and other Python tools from any terminal without specifying the full installation path.

### Why It Is Important
Without PATH setup, you must type the full path (`C:\Python312\python.exe` or `/usr/local/bin/python3`) every time you run Python. Proper PATH configuration ensures scripts with shebangs work, py launcher works on Windows, and IDEs can detect Python installations.

### How It Works Internally
When you type a command in the terminal, the shell searches each directory listed in the PATH variable (in order) for an executable with that name. On Windows, PATH is a semicolon-separated list; on Unix, it's colon-separated. The `which` command (Unix) or `where` command (Windows) shows which executable will be used. Python installers typically add entries like `C:\Python312\` and `C:\Python312\Scripts\` (Windows) or `/usr/local/bin` (Unix).

### Syntax
```bash
# View current PATH
# Windows:
echo %PATH%

# Unix:
echo $PATH

# Add Python to PATH temporarily
# Windows:
set PATH=C:\Python312;%PATH%

# Unix:
export PATH="/usr/local/bin:$PATH"

# Make PATH permanent
# Windows: System Properties > Environment Variables
# Unix: Add to ~/.bashrc, ~/.zshrc, or ~/.profile
```

### Beginner Examples
```python
import os
import sys

def check_path():
    paths = os.environ.get("PATH", "").split(os.pathsep)
    print(f"PATH has {len(paths)} entries")
    
    # Check if Python directory is in PATH
    python_dir = os.path.dirname(sys.executable)
    in_path = any(python_dir in p for p in paths)
    print(f"Python directory in PATH: {in_path}")
    
    # Show Python-related path entries
    for p in paths:
        if "python" in p.lower():
            print(f"  Python PATH entry: {p}")

check_path()
```

### Intermediate Examples
```python
import os
import sys
import subprocess
from pathlib import Path

def resolve_command(command: str) -> str | None:
    """Find the full path of a command using PATH."""
    paths = os.environ.get("PATH", "").split(os.pathsep)
    for path in paths:
        full = Path(path) / command
        if full.exists():
            return str(full)
    return None

def which_python():
    """Find which python executable will run."""
    python_path = resolve_command("python3" if sys.platform != "win32" else "python.exe")
    return python_path

print(f"Python will run from: {which_python()}")

# Test python resolution
result = subprocess.run(
    ["python3", "-c", "import sys; print(sys.executable)"],
    capture_output=True, text=True
)
print(f"Actual executable: {result.stdout.strip()}")
```

### Advanced Examples
```python
"""Cross-platform PATH management utility."""
import os
import sys
from pathlib import Path
from typing import Optional

class PathManager:
    @staticmethod
    def get_system_path() -> list[str]:
        return os.environ.get("PATH", "").split(os.pathsep)
    
    @staticmethod
    def normalize_path(path: str) -> str:
        return str(Path(path).resolve())
    
    @staticmethod
    def is_in_path(path: str) -> bool:
        normalized = PathManager.normalize_path(path)
        return any(
            PathManager.normalize_path(p) == normalized
            for p in PathManager.get_system_path()
        )
    
    @staticmethod
    def find_executable(name: str) -> Optional[str]:
        """Find an executable in PATH, similar to `which`."""
        paths = PathManager.get_system_path()
        extensions = [""]
        if sys.platform == "win32":
            extensions = [".exe", ".bat", ".cmd", ".ps1"]
        
        for p in paths:
            for ext in extensions:
                full = Path(p) / f"{name}{ext}"
                if full.exists():
                    return str(full)
        return None
    
    @staticmethod
    def python_versions_in_path() -> list[tuple[int, int, str]]:
        """Find all Python executables in PATH."""
        versions = []
        for name in ["python", "python3", "python3.12", "python3.11", "python3.10"]:
            path = PathManager.find_executable(name)
            if path:
                import subprocess
                result = subprocess.run(
                    [path, "-c", "import sys; print(f'{sys.version_info.major}.{sys.version_info.minor}.{sys.version_info.micro}')"],
                    capture_output=True, text=True, timeout=5
                )
                if result.returncode == 0:
                    ver = result.stdout.strip().split(".")
                    key = (int(ver[0]), int(ver[1]))
                    versions.append((key[0], key[1], path))
        return versions

if __name__ == "__main__":
    pm = PathManager()
    print(f"Python in PATH: {pm.find_executable('python3') or pm.find_executable('python')}")
    print(f"All Python versions in PATH: {pm.python_versions_in_path()}")
```

### Real-World Use Cases
- **CI/CD**: GitHub Actions use `setup-python` action to add Python to PATH
- **Docker**: ENV PATH modifications in Dockerfiles
- **Development**: Multiple Python versions managed via PATH ordering
- **Deployment**: Shell scripts that rely on Python being in PATH

### Common Mistakes
```python
# Mistake 1: Conflicting Python versions in PATH
# If C:\Python312\ and C:\Python311\ are both in PATH, the first one wins.
# Fix: Ensure the desired version appears first

# Mistake 2: Forgetting to restart terminal after PATH changes
# Environment variables are loaded at terminal startup
# Fix: Open a new terminal window

# Mistake 3: Modifying PATH incorrectly on Windows
# set PATH = %PATH%;C:\Python  # Wrong! (spaces around =)
# set PATH=%PATH%;C:\Python    # Correct
```

### Best Practices
- Add Python and Scripts directories to PATH, not individual files
- On Windows, let the installer manage PATH (check the box)
- Use `py` launcher on Windows for multiple Python versions
- Document PATH requirements in project setup instructions
- Use `export PATH` in shell config files (not command line)
- Avoid duplicate PATH entries

### Performance Considerations
- PATH length affects command resolution time (usually negligible)
- Each PATH entry is checked in order until the command is found
- Long PATH values (>1024 chars) can cause issues on Windows
- Shell startup time increases slightly with more PATH entries
- `hash -r` (Unix) clears the command hash table after PATH changes

### Interview Questions
1. What is the PATH environment variable?
2. How do you add Python to PATH on Windows?
3. What happens when multiple Python versions are in PATH?
4. How does the `which` command work on Unix?
5. How do you permanently modify PATH?
6. What is the `py` launcher on Windows?
7. How do you check if Python is in PATH?
8. What is the difference between user PATH and system PATH?
9. How do you remove a Python installation from PATH?
10. What are PATH security considerations?

### Coding Challenges
```python
# Challenge: PATH diagnostic tool
import os

def diagnose_path():
    path = os.environ.get("PATH", "")
    entries = path.split(os.pathsep)
    
    python_related = []
    missing = []
    
    for entry in entries:
        if not os.path.exists(entry):
            missing.append(entry)
        elif "python" in entry.lower():
            python_related.append(entry)
    
    print(f"Total PATH entries: {len(entries)}")
    print(f"Python-related entries: {len(python_related)}")
    for e in python_related:
        print(f"  {e}")
    print(f"Missing entries: {len(missing)}")
    for e in missing[:5]:
        print(f"  (missing) {e}")

diagnose_path()
```

### Related Topics
- Environment Variables
- Shell Configuration (.bashrc, .zshrc)
- The `py` Launcher
- Command Resolution (`which`, `where`)
- Cross-Platform Development

## venv

### What It Is
venv is Python's built-in module for creating lightweight virtual environments — isolated Python environments with their own site-packages directories. It has been included in the standard library since Python 3.3.

### Why It Is Important
Virtual environments solve dependency conflicts by isolating project dependencies. Each project gets its own copy of pip, Python interpreter symlink, and site-packages directory. This prevents version conflicts between projects (e.g., Project A needs Django 3.2, Project B needs Django 5.0).

### How It Works Internally
`python -m venv <directory>` creates a directory containing:
- `pyvenv.cfg` — configuration file pointing to the base Python installation
- A symlink or copy of the Python executable
- `site-packages/` — initially empty, where installed packages go
- Activation scripts for various shells (`activate`, `activate.bat`, `deactivate`)
- Modified `sys.path` to prioritize the venv's site-packages over the global one

When activated, the shell's PATH is modified to put the venv's `bin/` or `Scripts/` directory first, so `python` and `pip` resolve to the venv versions.

### Syntax
```bash
# Create a virtual environment
python -m venv .venv
python -m venv --system-site-packages .venv  # Include global packages
python -m venv --prompt myproject .venv      # Custom prompt name

# Activate
# Windows (cmd):
.venv\Scripts\activate
# Windows (PowerShell):
.venv\Scripts\Activate.ps1
# Unix/macOS:
source .venv/bin/activate

# Deactivate
deactivate

# Check if inside a venv
python -c "import sys; print(sys.prefix != sys.base_prefix)"
```

### Beginner Examples
```python
# Check if running inside a virtual environment
import sys

def check_venv():
    in_venv = sys.prefix != sys.base_prefix
    if in_venv:
        print(f"Virtual environment: {sys.prefix}")
        print(f"Base Python: {sys.base_prefix}")
    else:
        print("Not in a virtual environment")
    
    print(f"Python executable: {sys.executable}")
    print(f"sys.path[0]: {sys.path[0]}")

check_venv()
```

### Intermediate Examples
```python
"""Programmatic venv creation and management."""
import os
import sys
import subprocess
from pathlib import Path

class VenvManager:
    def __init__(self, path: str | Path):
        self.path = Path(path).resolve()
        self.bin_dir = self.path / ("Scripts" if os.name == "nt" else "bin")
        self.python = self.bin_dir / ("python.exe" if os.name == "nt" else "python3")
        self.pip = self.bin_dir / ("pip.exe" if os.name == "nt" else "pip")
    
    def create(self, system_site_packages: bool = False):
        cmd = [sys.executable, "-m", "venv", str(self.path)]
        if system_site_packages:
            cmd.append("--system-site-packages")
        subprocess.run(cmd, check=True)
        print(f"Created venv at {self.path}")
    
    def install(self, *packages: str):
        subprocess.run([str(self.pip), "install", *packages], check=True)
    
    def run(self, script: str):
        subprocess.run([str(self.python), script], check=True)
    
    def freeze(self) -> str:
        result = subprocess.run(
            [str(self.pip), "freeze"],
            capture_output=True, text=True, check=True
        )
        return result.stdout

# Usage
manager = VenvManager(".venv")
# manager.create()
# manager.install("requests", "numpy")
# print(manager.freeze())
```

### Advanced Examples
```python
"""Cross-project dependency management with venv."""
import json
import os
import subprocess
import sys
from pathlib import Path
from typing import Optional

class ProjectVenv:
    CONFIG_FILE = "pyproject.toml"
    
    def __init__(self, project_dir: str | Path):
        self.project_dir = Path(project_dir).resolve()
        self.venv_dir = self.project_dir / ".venv"
        
    def exists(self) -> bool:
        return self.venv_dir.exists()
    
    def create(self, python: Optional[str] = None):
        python = python or sys.executable
        subprocess.run(
            [python, "-m", "venv", str(self.venv_dir)],
            check=True
        )
    
    @property
    def python_path(self) -> Path:
        if os.name == "nt":
            return self.venv_dir / "Scripts" / "python.exe"
        return self.venv_dir / "bin" / "python"
    
    def pip_install(self, *packages, upgrade_pip: bool = True):
        if upgrade_pip:
            subprocess.run(
                [str(self.python_path), "-m", "pip", "install", "--upgrade", "pip"],
                check=True
            )
        if packages:
            subprocess.run(
                [str(self.python_path), "-m", "pip", "install", *packages],
                check=True
            )
    
    def export_requirements(self, path: str | Path = "requirements.txt"):
        result = subprocess.run(
            [str(self.python_path), "-m", "pip", "freeze"],
            capture_output=True, text=True, check=True
        )
        Path(path).write_text(result.stdout)
    
    def import_requirements(self, path: str | Path = "requirements.txt"):
        p = Path(path)
        if p.exists():
            self.pip_install("-r", str(p))
    
    def get_info(self) -> dict:
        import json
        result = subprocess.run(
            [str(self.python_path), "-c", """
import sys, json
print(json.dumps({
    'version': sys.version,
    'prefix': sys.prefix,
    'base_prefix': sys.base_prefix,
    'path': sys.path[:3]
}))
            """],
            capture_output=True, text=True, check=True
        )
        return json.loads(result.stdout)

# Usage
# v = ProjectVenv(".")
# if not v.exists():
#     v.create()
# print(v.get_info())
```

### Real-World Use Cases
- **Project Isolation**: Each microservice in a monorepo gets its own venv
- **CI/CD**: Creating venvs in CI pipelines for test isolation
- **Deployment**: Using venv inside Docker images (lighter than system Python)
- **Education**: Giving students pre-configured venvs
- **Testing**: Running tests against multiple dependency versions

### Common Mistakes
```python
# Mistake 1: Committing venv to version control
# .venv/ should be in .gitignore

# Mistake 2: Forgetting to activate venv
# Install packages globally when meant to be local
# Always check: which python (Unix) or where python (Windows)

# Mistake 3: Using `--system-site-packages` unnecessarily
# This links global packages and defeats isolation

# Mistake 4: Deleting the venv directory without deactivating
# The activation script references the deleted directory
# Always run `deactivate` first

# Mistake 5: Using different Python version for venv than project requires
# Create venv with specific version: python3.12 -m venv .venv
```

### Best Practices
- Always use venv for every project (even small scripts)
- Name the directory `.venv` (convention, automatically hidden)
- Add `.venv/` to `.gitignore`
- Generate `requirements.txt` with `pip freeze > requirements.txt`
- Use `--prompt` to set a meaningful shell prompt
- Create venv in the project root directory
- Document activation commands in project README

### Performance Considerations
- Creating a venv takes ~1-2 seconds
- Each venv uses ~10-50MB of disk space (symlinks on Unix are smaller)
- Package installs inside venv are slightly slower than global (extra PATH resolution)
- venv symlinks to the base Python — no duplicate Python binary
- Disk usage grows with installed packages; `.venv` can easily reach 100-500MB

### Interview Questions
1. What is a virtual environment and why use it?
2. How does venv differ from virtualenv?
3. What files/directories does venv create?
4. How does venv isolation work internally?
5. What is the difference between `sys.prefix` and `sys.base_prefix`?
6. Should you commit venv to version control? Why or why not?
7. How do you recreate a venv from requirements?
8. What does the activation script actually do?
9. How do you use a different Python version for a venv?
10. What is `--system-site-packages`?

### Coding Challenges
```python
# Challenge: Auto-venv activator (shell function concept)
import os
import sys
from pathlib import Path

def find_venv(start_path: str = ".") -> Path | None:
    """Walk up directories looking for .venv."""
    path = Path(start_path).resolve()
    for parent in [path] + list(path.parents):
        venv = parent / ".venv"
        bin_dir = venv / ("Scripts" if os.name == "nt" else "bin")
        python = bin_dir / ("python.exe" if os.name == "nt" else "python3")
        if venv.exists() and python.exists():
            return venv
    return None

def get_venv_info(path: Path) -> dict:
    """Get details about a virtual environment."""
    cfg = path / "pyvenv.cfg"
    info = {}
    if cfg.exists():
        for line in cfg.read_text().splitlines():
            if "=" in line:
                k, v = line.split("=", 1)
                info[k.strip()] = v.strip()
    return info

venv_path = find_venv()
if venv_path:
    print(f"Found venv: {venv_path}")
    print(f"Info: {get_venv_info(venv_path)}")
else:
    print("No venv found")
```

### Related Topics
- virtualenv (third-party, more features)
- pipenv and Poetry (higher-level tools)
- Dependency Management
- `requirements.txt` Format
- Python Packaging

## pip

### What It Is
pip is Python's package installer, included by default since Python 3.4. It downloads and installs packages from the Python Package Index (PyPI, pypi.org) and other indexes. pip handles dependencies, resolves version conflicts, and manages package installations in the current Python environment.

### Why It Is Important
pip provides access to the Python ecosystem's 400,000+ packages. It enables reproducible installations via `requirements.txt`, supports version pinning, and integrates with virtual environments. pip is the standard tool for installing third-party libraries.

### How It Works Internally
pip downloads package metadata and source distributions or wheels from PyPI. It resolves dependencies using a SAT solver (since pip 10.0+), downloads packages, and installs them into the current environment's `site-packages` directory. Wheels (`.whl` files) are pre-built distributions that install faster than source distributions. pip maintains a local cache (`~/.cache/pip`) to avoid re-downloading.

### Syntax
```bash
# Basic commands
pip install requests               # Install a package
pip install requests==2.31.0       # Specific version
pip install requests>=2.30,<3.0    # Version range
pip install numpy pandas matplotlib # Multiple packages
pip install -r requirements.txt    # From file

# Uninstall
pip uninstall requests

# List installed
pip list
pip list --outdated                # Show outdated packages

# Show package info
pip show requests
pip show -f requests               # With files

# Freeze dependencies
pip freeze > requirements.txt

# Check compatibility
pip check                          # Verify installed packages
pip install --upgrade pip          # Upgrade pip itself
```

### Beginner Examples
```python
# Programmatic pip usage (not recommended, but possible)
import subprocess
import sys

def install_package(package_name: str):
    """Install a pip package via subprocess."""
    result = subprocess.run(
        [sys.executable, "-m", "pip", "install", package_name],
        capture_output=True, text=True
    )
    if result.returncode == 0:
        print(f"Installed {package_name}")
    else:
        print(f"Error: {result.stderr}")

def list_installed():
    result = subprocess.run(
        [sys.executable, "-m", "pip", "list", "--format=columns"],
        capture_output=True, text=True
    )
    print(result.stdout)

# Usage
install_package("requests")
list_installed()
```

### Intermediate Examples
```python
"""Dependency auditing and requirements management."""
import subprocess
import sys
from typing import Optional

class PipManager:
    def __init__(self, python_executable: Optional[str] = None):
        self.python = python_executable or sys.executable
    
    def _run_pip(self, *args) -> subprocess.CompletedProcess:
        return subprocess.run(
            [self.python, "-m", "pip", *args],
            capture_output=True, text=True, check=True
        )
    
    def freeze(self) -> dict[str, str]:
        """Returns dict of package_name -> version."""
        result = self._run_pip("freeze")
        deps = {}
        for line in result.stdout.strip().splitlines():
            if "==" in line:
                name, version = line.split("==", 1)
                deps[name.lower()] = version
        return deps
    
    def audit_outdated(self) -> list[dict]:
        """Find outdated packages."""
        result = self._run_pip("list", "--outdated", "--format=json")
        import json
        return json.loads(result.stdout)
    
    def check_dependencies(self) -> list[str]:
        """Check for broken dependencies."""
        result = subprocess.run(
            [self.python, "-m", "pip", "check"],
            capture_output=True, text=True
        )
        if result.returncode == 0:
            return []
        return result.stderr.strip().splitlines()
    
    def install(self, *packages: str, upgrade: bool = False):
        cmd = ["install"]
        if upgrade:
            cmd.append("--upgrade")
        cmd.extend(packages)
        self._run_pip(*cmd)

# Usage
pm = PipManager()
print(f"Installed packages: {len(pm.freeze())}")
outdated = pm.audit_outdated()
print(f"Outdated: {len(outdated)} packages")
```

### Advanced Examples
```python
"""Custom pip index and dependency resolution."""
import json
import subprocess
import sys
from pathlib import Path
from typing import Optional

class AdvancedPipManager:
    def __init__(self, python: Optional[str] = None):
        self.python = python or sys.executable
    
    def download_with_deps(self, package: str, output_dir: str | Path = "vendor"):
        """Download a package and all dependencies."""
        output_dir = Path(output_dir)
        output_dir.mkdir(exist_ok=True)
        
        subprocess.run([
            self.python, "-m", "pip", "download",
            "--dest", str(output_dir),
            package
        ], check=True)
        
        print(f"Downloaded {package} and deps to {output_dir}")
    
    def install_from_requirements(self, req_file: str | Path, dry_run: bool = False):
        """Install packages from requirements.txt with options."""
        cmd = [
            self.python, "-m", "pip", "install",
            "-r", str(req_file),
            "--no-compile",  # Faster install
        ]
        if dry_run:
            cmd.append("--dry-run")
        
        result = subprocess.run(cmd, capture_output=True, text=True)
        if dry_run:
            # Parse what would be installed
            for line in result.stdout.splitlines():
                if line.startswith("Would install"):
                    print(line)
    
    def get_dependency_tree(self, package: str) -> dict:
        """Get dependency tree for a package."""
        import pip._vendor.resolvelib as resolvelib
        
        result = subprocess.run([
            self.python, "-m", "pip", "install",
            package, "--dry-run", "--report", "-"
        ], capture_output=True, text=True)
        
        if result.stdout:
            report = json.loads(result.stdout)
            return {
                item["metadata"]["name"]: {
                    "version": item["metadata"]["version"],
                    "dependencies": item["metadata"].get("requires_dist", [])
                }
                for item in report.get("install", [])
            }
        return {}
    
    def create_constraints_file(
        self, output: str | Path = "constraints.txt", *packages: str
    ):
        """Create a constraints file that pins transitive dependencies."""
        # Install packages first to get resolved versions
        subprocess.run([
            self.python, "-m", "pip", "install", *packages
        ], check=True)
        
        # Get all installed packages
        result = subprocess.run([
            self.python, "-m", "pip", "freeze"
        ], capture_output=True, text=True, check=True)
        
        Path(output).write_text(result.stdout)
        print(f"Written {len(result.stdout.splitlines())} constraints to {output}")

# Usage
manager = AdvancedPipManager()
tree = manager.get_dependency_tree("fastapi")
if tree:
    for name, info in tree.items():
        print(f"{name}=={info['version']}")
```

### Real-World Use Cases
- **Dependency Management**: Installing project dependencies via requirements.txt
- **CI/CD**: Automating package installation in build pipelines
- **Package Publishing**: Using twine to upload packages to PyPI
- **Offline Installs**: Downloading packages for air-gapped environments
- **Auditing**: Checking for outdated or vulnerable packages

### Common Mistakes
```bash
# Mistake 1: Installing packages without venv (global install)
pip install requests  # Installs globally — pollutes system Python

# Correct:
python -m venv .venv
source .venv/bin/activate
pip install requests

# Mistake 2: Not pinning versions
pip install requests  # Gets latest, may break later

# Correct:
pip install requests==2.31.0

# Mistake 3: Using pip outside a venv to list packages
pip list  # Shows all global packages, very long

# Mistake 4: Installing with sudo (Unix)
sudo pip install requests  # Overrides system packages

# Correct:
pip install --user requests  # User installation

# Mistake 5: Forgetting --upgrade for pip itself
pip install --upgrade pip  # Always do this first in a new venv
```

### Best Practices
- Always use pip inside a virtual environment
- Pin package versions in requirements.txt: `requests==2.31.0`
- Use `pip install --upgrade pip` at the start of each venv
- Run `pip check` to verify dependency compatibility
- Use `pip list --outdated` periodically for updates
- Use `python -m pip` instead of bare `pip` (ensures correct Python)
- Use `requirements.in` (loose) and `requirements.txt` (frozen) for layered deps
- Consider using `pip-tools` with `pip-compile` for deterministic builds
- Use `--no-cache-dir` in Docker builds to reduce image size

### Performance Considerations
- pip install uses HTTP(S) to download from PyPI (CDN-backed, ~50-200ms latency)
- Installing from wheels (.whl) is 5-10x faster than source distributions
- The pip cache (`~/.cache/pip`) saves re-downloading; can grow to several GB
- Dependency resolution (pip 20.3+) uses backtracking and can be slow for complex deps
- `--no-deps` skips dependency resolution for faster single-package installs
- `--no-compile` skips `.pyc` compilation, reducing install time by ~30%

### Interview Questions
1. What is pip and what problem does it solve?
2. How do you pin package versions in pip?
3. What is the difference between `requirements.txt` and `setup.py`?
4. How does pip resolve dependencies?
5. What is a wheel and how is it different from a source distribution?
6. How do you create a requirements.txt file?
7. What does `pip check` do?
8. How do you install packages from a private index?
9. What is the pip cache and how do you manage it?
10. How do you handle transitive dependency conflicts?

### Coding Challenges
```python
# Challenge: Requirements.txt validator
import re
from pathlib import Path

def validate_requirements(path: str | Path = "requirements.txt"):
    """Validate a requirements.txt file."""
    path = Path(path)
    if not path.exists():
        return {"valid": False, "error": "File not found"}
    
    lines = path.read_text().splitlines()
    errors = []
    packages = []
    
    spec_pattern = re.compile(
        r'^[a-zA-Z0-9_.-]+'  # Package name
        r'(([><=!~]=?[a-zA-Z0-9.*]+)(,?[><=!~]=?[a-zA-Z0-9.*]+)*)?'  # Version spec
        r'(\s*#.*)?$'  # Optional comment
    )
    
    for i, line in enumerate(lines, 1):
        line = line.strip()
        if not line or line.startswith("#") or line.startswith("-r"):
            continue
        if line.startswith("git+") or line.startswith("http"):
            continue
        
        if not spec_pattern.match(line):
            errors.append(f"Line {i}: Invalid format '{line}'")
        else:
            name = line.split("=")[0].split("[")[0].split("!")[0]
            packages.append(name)
    
    return {
        "valid": len(errors) == 0,
        "packages": packages,
        "count": len(packages),
        "errors": errors
    }

result = validate_requirements()
print(f"Valid: {result['valid']}, Packages: {result['count']}")
```

### Related Topics
- `requirements.txt` Format
- Python Packaging (setuptools, wheel, twine)
- PyPI (Python Package Index)
- Dependency Resolution
- Private Package Repositories
- pip-tools, pipenv, Poetry
