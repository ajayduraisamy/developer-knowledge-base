# Virtual Environments - venv, virtualenv, poetry, uv

## Introduction

A virtual environment is an isolated Python environment that maintains its own interpreter, site-packages, and executables, separate from the system-wide Python installation. This allows different projects to use different versions of libraries without conflicts.

## Why It Is Important

Virtual environments solve the "dependency hell" problem by ensuring each project has its own set of dependencies, regardless of what other projects or the system require. They enable reproducible builds, prevent version conflicts, and allow testing across multiple Python versions. They are considered an essential best practice for Python development.

## Syntax

```bash
# Built-in venv (Python 3.3+)
python -m venv myenv
myenv\Scripts\activate      # Windows
source myenv/bin/activate   # macOS/Linux
deactivate

# virtualenv (older, more features)
pip install virtualenv
virtualenv myenv

# pyenv (Python version manager)
pyenv install 3.12.0
pyenv local 3.12.0

# poetry
poetry new myproject
poetry add requests

# uv (fast Rust-based tool)
uv venv myenv
uv pip install requests
```

## Examples

### Creating and Using a Virtual Environment with venv

```python
import subprocess
import sys
import os
from pathlib import Path

def create_venv(env_path="myenv"):
    result = subprocess.run(
        [sys.executable, "-m", "venv", env_path],
        capture_output=True, text=True
    )
    if result.returncode == 0:
        print(f"Virtual environment created at {env_path}")
    else:
        print(f"Error: {result.stderr}")
    return result.returncode == 0

def get_venv_python(env_path="myenv"):
    if sys.platform == "win32":
        python_path = Path(env_path) / "Scripts" / "python.exe"
    else:
        python_path = Path(env_path) / "bin" / "python"
    return str(python_path) if python_path.exists() else None

def install_in_venv(env_path, package):
    python = get_venv_python(env_path)
    if not python:
        print(f"Venv not found at {env_path}")
        return False
    result = subprocess.run(
        [python, "-m", "pip", "install", package],
        capture_output=True, text=True
    )
    return result.returncode == 0

def list_in_venv(env_path):
    python = get_venv_python(env_path)
    if not python:
        return []
    result = subprocess.run(
        [python, "-m", "pip", "list", "--format=json"],
        capture_output=True, text=True
    )
    if result.returncode == 0:
        import json
        return json.loads(result.stdout)
    return []

env = "test_venv"
create_venv(env)
print(f"Venv Python: {get_venv_python(env)}")
if install_in_venv(env, "requests"):
    print("Installed requests in venv")
packages = list_in_venv(env)
print(f"Packages in venv: {len(packages)}")
for pkg in packages[:5]:
    print(f"  {pkg['name']}=={pkg['version']}")

import shutil
shutil.rmtree(env, ignore_errors=True)
```

### Activating and Deactivating

```python
import subprocess
import sys
import os
from pathlib import Path

def activate_and_run(env_path, commands):
    """Simulate activation by running commands with the venv's Python."""
    if sys.platform == "win32":
        python = Path(env_path) / "Scripts" / "python.exe"
        activate = Path(env_path) / "Scripts" / "activate.bat"
    else:
        python = Path(env_path) / "bin" / "python"
        activate = Path(env_path) / "bin" / "activate"

    results = []
    for cmd in commands:
        result = subprocess.run(
            [str(python), "-c", cmd],
            capture_output=True, text=True
        )
        results.append((cmd, result.stdout.strip(), result.stderr.strip()))
    return results

def check_activation_status():
    """Check if we're inside a virtual environment."""
    in_venv = sys.prefix != sys.base_prefix
    print(f"In virtual environment: {in_venv}")
    if in_venv:
        print(f"Virtual env path: {sys.prefix}")
        print(f"Base Python path: {sys.base_prefix}")
    return in_venv

check_activation_status()

venv_path = "test_activate_venv"
subprocess.run([sys.executable, "-m", "venv", venv_path])

if sys.platform == "win32":
    activate_cmd = f"{venv_path}\\Scripts\\activate.bat"
    deactivate_cmd = "deactivate"
else:
    activate_cmd = f"source {venv_path}/bin/activate"
    deactivate_cmd = "deactivate"

print(f"\nActivate command: {activate_cmd}")
print(f"Deactivate command: {deactivate_cmd}")

results = activate_and_run(venv_path, [
    "import sys; print(sys.prefix != sys.base_prefix)",
    "import os; print(os.environ.get('VIRTUAL_ENV', 'Not set'))"
])
for cmd, out, err in results:
    print(f"  $ python -c \"{cmd}\"")
    print(f"  => {out}")

import shutil
shutil.rmtree(venv_path, ignore_errors=True)
```

### Using virtualenv (Legacy Tool)

```python
import subprocess
import sys

def install_virtualenv():
    result = subprocess.run(
        [sys.executable, "-m", "pip", "install", "virtualenv"],
        capture_output=True, text=True
    )
    return result.returncode == 0

def create_with_virtualenv(env_path, python_version=None):
    cmd = [sys.executable, "-m", "virtualenv", env_path]
    if python_version:
        cmd.extend(["--python", python_version])
    result = subprocess.run(cmd, capture_output=True, text=True)
    return result.returncode == 0

def compare_venv_vs_virtualenv():
    info = {
        "feature": ["Built-in", "Faster", "Pip support", "Python 2", "--system-site-packages", "Seed packages config"],
        "venv": ["Yes (3.3+)", "Yes", "Bundled", "No", "Yes", "Limited"],
        "virtualenv": ["No", "No (slower)", "Yes", "Yes", "Yes", "Full"]
    }
    for i in range(len(info["feature"])):
        print(f"{info['feature'][i]:30s} | venv: {info['venv'][i]:12s} | virtualenv: {info['virtualenv'][i]:12s}")

if install_virtualenv():
    print("virtualenv installed")
    create_with_virtualenv("test_venv_2")
    print("virtualenv venv created")
    import shutil
    shutil.rmtree("test_venv_2", ignore_errors=True)

compare_venv_vs_virtualenv()
```

## Beginner Examples

```python
# Project setup script with venv
import subprocess
import sys
import os
from pathlib import Path

def setup_project(project_name, dependencies=None):
    project_dir = Path(project_name)
    project_dir.mkdir(exist_ok=True)
    venv_dir = project_dir / ".venv"

    print(f"Setting up project: {project_name}")

    # Create virtual environment
    result = subprocess.run(
        [sys.executable, "-m", "venv", str(venv_dir)],
        capture_output=True, text=True
    )
    if result.returncode != 0:
        print(f"Failed to create venv: {result.stderr}")
        return False

    # Determine the Python executable inside the venv
    if sys.platform == "win32":
        python_path = venv_dir / "Scripts" / "python.exe"
    else:
        python_path = venv_dir / "bin" / "python"

    # Upgrade pip inside venv
    subprocess.run(
        [str(python_path), "-m", "pip", "install", "--upgrade", "pip"],
        capture_output=True
    )

    # Install dependencies
    if dependencies:
        subprocess.run(
            [str(python_path), "-m", "pip", "install"] + dependencies,
            capture_output=True
        )
        print(f"Installed: {', '.join(dependencies)}")

    # Create requirements.txt
    result = subprocess.run(
        [str(python_path), "-m", "pip", "freeze"],
        capture_output=True, text=True
    )
    if result.returncode == 0:
        req_file = project_dir / "requirements.txt"
        with open(req_file, "w") as f:
            f.write(result.stdout)
        print(f"Created {req_file}")

    # Create a simple main.py
    main_file = project_dir / "main.py"
    main_file.write_text(f'''"""Main entry point for {project_name}."""
import sys

def main():
    print(f"Python: {{sys.version}}")
    print(f"Virtual env: {{sys.prefix}}")

if __name__ == "__main__":
    main()
''')
    print(f"Created {main_file}")

    print(f"\nProject {project_name} is ready!")
    print(f"Activate: .venv\\Scripts\\activate  (Windows)")
    print(f"          source .venv/bin/activate  (macOS/Linux)")
    return True

setup_project("example_project", ["requests"])
import shutil
shutil.rmtree("example_project", ignore_errors=True)

# Listing all venvs created in a directory
def find_virtual_environments(base_dir="."):
    venvs = []
    for item in Path(base_dir).iterdir():
        if item.is_dir():
            venv_python = item / "Scripts" / "python.exe" if sys.platform == "win32" else item / "bin" / "python"
            if venv_python.exists():
                result = subprocess.run(
                    [str(venv_python), "--version"],
                    capture_output=True, text=True
                )
                version = result.stdout.strip() if result.returncode == 0 else "Unknown"
                venvs.append((item.name, version))
    return venvs

venvs = find_virtual_environments(".")
if venvs:
    print("\nFound virtual environments:")
    for name, version in venvs:
        print(f"  {name}: {version}")
```

## Intermediate Examples

```python
# Multi-environment testing system
import subprocess
import sys
import os
from pathlib import Path
import json
import time

class MultiEnvTester:
    def __init__(self, base_dir="test_envs"):
        self.base_dir = Path(base_dir)
        self.base_dir.mkdir(exist_ok=True)
        self.results = {}

    def create_env(self, name, python_path=None):
        env_dir = self.base_dir / name
        if env_dir.exists():
            print(f"Environment {name} already exists")
            return False

        cmd = [python_path or sys.executable, "-m", "venv", str(env_dir)]
        result = subprocess.run(cmd, capture_output=True, text=True)
        if result.returncode == 0:
            print(f"Created env: {name}")
            return True
        print(f"Failed: {result.stderr}")
        return False

    def get_python(self, name):
        env_dir = self.base_dir / name
        if sys.platform == "win32":
            python = env_dir / "Scripts" / "python.exe"
        else:
            python = env_dir / "bin" / "python"
        return str(python) if python.exists() else None

    def install(self, name, packages):
        python = self.get_python(name)
        if not python:
            return False
        result = subprocess.run(
            [python, "-m", "pip", "install"] + packages,
            capture_output=True, text=True
        )
        return result.returncode == 0

    def run_test(self, name, test_code):
        python = self.get_python(name)
        if not python:
            return None
        start = time.time()
        result = subprocess.run(
            [python, "-c", test_code],
            capture_output=True, text=True
        )
        elapsed = time.time() - start
        return {
            "stdout": result.stdout.strip(),
            "stderr": result.stderr.strip(),
            "returncode": result.returncode,
            "elapsed": round(elapsed, 3)
        }

    def test_across_envs(self, test_code, env_names=None):
        if env_names is None:
            env_names = [d.name for d in self.base_dir.iterdir() if d.is_dir()]

        for env in env_names:
            result = self.run_test(env, test_code)
            self.results[env] = result
            if result:
                status = "PASS" if result["returncode"] == 0 else "FAIL"
                print(f"{env}: {status} ({result['elapsed']}s)")
                if result["stdout"]:
                    print(f"  Output: {result['stdout'][:100]}")
        return self.results

    def compare_packages(self, env_names=None):
        if env_names is None:
            env_names = [d.name for d in self.base_dir.iterdir() if d.is_dir()]

        all_packages = {}
        for env in env_names:
            python = self.get_python(env)
            if not python:
                continue
            result = subprocess.run(
                [python, "-m", "pip", "list", "--format=json"],
                capture_output=True, text=True
            )
            if result.returncode == 0:
                pkgs = json.loads(result.stdout)
                all_packages[env] = {p["name"]: p["version"] for p in pkgs}

        if len(all_packages) >= 2:
            env_list = list(all_packages.keys())
            base_pkgs = set(all_packages[env_list[0]].keys())
            for env in env_list[1:]:
                diff = base_pkgs.symmetric_difference(set(all_packages[env].keys()))
                if diff:
                    print(f"Differences between {env_list[0]} and {env}:")
                    for pkg in sorted(diff):
                        v1 = all_packages[env_list[0]].get(pkg, "MISSING")
                        v2 = all_packages[env].get(pkg, "MISSING")
                        print(f"  {pkg}: {v1} vs {v2}")

tester = MultiEnvTester("test_envs_multi")
tester.create_env("env_py311")
tester.create_env("env_py312") if False else None

import shutil
if Path("test_envs_multi").exists():
    shutil.rmtree("test_envs_multi")
```

## Advanced Examples

```python
# Advanced environment manager with pyenv, poetry, and uv integration
import subprocess
import sys
import os
from pathlib import Path
import json
import re
import shutil

class AdvancedEnvManager:
    def __init__(self, base_dir=".environments"):
        self.base_dir = Path(base_dir)
        self.base_dir.mkdir(exist_ok=True)

    def check_tools(self):
        tools = {
            "venv": self._check_cmd([sys.executable, "-m", "venv", "--help"]),
            "pyenv": self._check_cmd(["pyenv", "--version"]),
            "poetry": self._check_cmd(["poetry", "--version"]),
            "pipx": self._check_cmd(["pipx", "--version"]),
            "uv": self._check_cmd(["uv", "--version"]),
        }
        for name, available in tools.items():
            print(f"{name}: {'✓' if available else '✗'}")
        return tools

    def _check_cmd(self, cmd):
        try:
            subprocess.run(cmd, capture_output=True, timeout=5)
            return True
        except (subprocess.SubprocessError, FileNotFoundError):
            return False

    def create_pyenv_virtualenv(self, python_version, env_name):
        result = subprocess.run(
            ["pyenv", "virtualenv", python_version, env_name],
            capture_output=True, text=True
        )
        if result.returncode == 0:
            print(f"Created pyenv venv: {env_name} (Python {python_version})")
        else:
            print(f"Error: {result.stderr}")
        return result.returncode == 0

    def create_poetry_project(self, project_dir, python_version=None):
        path = Path(project_dir)
        path.mkdir(exist_ok=True)
        cmd = ["poetry", "init", "--no-interaction"]
        if python_version:
            cmd.extend(["--python", python_version])
        result = subprocess.run(cmd, cwd=str(path), capture_output=True, text=True)
        if result.returncode == 0:
            print(f"Initialized Poetry project in {project_dir}")
            subprocess.run(["poetry", "install"], cwd=str(path), capture_output=True)
        return result.returncode == 0

    def create_uv_venv(self, env_name):
        result = subprocess.run(
            ["uv", "venv", env_name],
            capture_output=True, text=True
        )
        if result.returncode == 0:
            print(f"Created uv venv: {env_name}")
        return result.returncode == 0

    def uv_install(self, env_name, packages):
        uv_python = Path(env_name) / "Scripts" / "python.exe" if sys.platform == "win32" else Path(env_name) / "bin" / "python"
        result = subprocess.run(
            ["uv", "pip", "install"] + packages,
            capture_output=True, text=True,
            env={**os.environ, "UV_SYSTEM_PYTHON": "0"}
        )
        return result.returncode == 0

    def replicate_environment(self, source_venv, target_venv):
        python = Path(source_venv) / "Scripts" / "python.exe" if sys.platform == "win32" else Path(source_venv) / "bin" / "python"
        if not python.exists():
            print(f"Source venv not found: {source_venv}")
            return False

        freeze_result = subprocess.run(
            [str(python), "-m", "pip", "freeze"],
            capture_output=True, text=True
        )

        target_python = Path(target_venv) / "Scripts" / "python.exe" if sys.platform == "win32" else Path(target_venv) / "bin" / "python"
        if not target_python.exists():
            print(f"Target venv not found: {target_venv}")
            return False

        req_file = self.base_dir / "temp_reqs.txt"
        with open(req_file, "w") as f:
            f.write(freeze_result.stdout)

        result = subprocess.run(
            [str(target_python), "-m", "pip", "install", "-r", str(req_file)],
            capture_output=True, text=True
        )
        req_file.unlink(missing_ok=True)

        if result.returncode == 0:
            print(f"Replicated {source_venv} -> {target_venv}")
        return result.returncode == 0

    def export_environment_spec(self, env_name):
        python = Path(env_name) / "Scripts" / "python.exe" if sys.platform == "win32" else Path(env_name) / "bin" / "python"
        if not python.exists():
            return None

        info = {
            "name": env_name,
            "python_version": subprocess.run([str(python), "--version"], capture_output=True, text=True).stdout.strip(),
            "pip_version": subprocess.run([str(python), "-m", "pip", "--version"], capture_output=True, text=True).stdout.strip(),
        }

        freeze = subprocess.run([str(python), "-m", "pip", "freeze"], capture_output=True, text=True)
        info["packages"] = freeze.stdout.strip().splitlines() if freeze.returncode == 0 else []

        return info

    def benchmark_creation(self, tool_commands, count=3):
        results = {}
        import time
        for name, cmd in tool_commands.items():
            times = []
            for i in range(count):
                start = time.time()
                subprocess.run(cmd, capture_output=True, timeout=30)
                elapsed = time.time() - start
                times.append(elapsed)
                cleanup = cmd[cmd.index("venv") + 1] if "venv" in cmd else None
            results[name] = {
                "min": min(times),
                "max": max(times),
                "avg": sum(times) / len(times)
            }
            print(f"{name}: avg={results[name]['avg']:.3f}s")
        return results

manager = AdvancedEnvManager()
tools = manager.check_tools()

spec = manager.export_environment_spec(".")
if spec:
    print(f"Python: {spec['python_version']}")
    print(f"Packages: {len(spec['packages'])}")

shutil.rmtree(".environments", ignore_errors=True)
```

## Real-World Use Cases

- **CI/CD pipelines**: Creating fresh venvs per build to ensure clean, reproducible test environments
- **Microservices**: Each service gets its own venv with pinned dependencies for independent deployment
- **Data science**: Using conda environments for complex numerical library dependencies (numpy, scipy, tensorflow)
- **Legacy code maintenance**: Maintaining Python 2.7 environments alongside Python 3.x projects
- **Cross-platform development**: Testing code across Windows, macOS, and Linux using platform-specific venvs
- **Security research**: Isolating potentially malicious packages in throwaway virtual environments

## Common Mistakes

- Activating the wrong virtual environment (check prompt or `which python`)
- Forgetting to activate the venv before running `pip install`
- Committing `.venv` directories to version control (add to `.gitignore`)
- Creating a venv inside a Git repository but not using `.gitignore` for it
- Using `pip freeze` outside the venv and getting system packages
- Running out of disk space from multiple large venvs (use `pip cache purge`)
- Mixing conda environments with pip/venv environments in the same project
- Not recreating the venv when switching between Python versions

## Best Practices

- Always use a virtual environment for every project, including small scripts
- Name the venv directory `.venv` (hidden) consistently across projects
- Add `.venv/` to `.gitignore` immediately when starting a project
- Use `python -m venv` instead of `virtualenv` for Python 3.3+ projects
- Use `pip freeze > requirements.txt` to capture exact versions for reproducibility
- Regularly recreate venvs when upgrading major dependency versions
- Consider `pipenv` or `poetry` for projects with complex dependency graphs
- Use `.envrc` (direnv) to auto-activate venvs when entering project directories

## Interview Questions

1. **Q**: What is the difference between `sys.prefix` and `sys.base_prefix`?
   **A**: `sys.prefix` points to the current environment root (venv or system). `sys.base_prefix` always points to the system Python that created the environment. They differ when a virtual environment is active.

2. **Q**: How does venv work internally?
   **A**: venv creates a directory with its own `bin/` (or `Scripts/` on Windows) containing copies/symlinks of the Python executable. It modifies `sys.path` to prioritize the venv's site-packages over the system's. The activate script sets `VIRTUAL_ENV` and adjusts `PATH`.

3. **Q**: What are the differences between venv, virtualenv, and conda?
   **A**: venv is built-in (Python 3.3+), lightweight, and sufficient for most needs. virtualenv supports Python 2 and has more configuration options. conda is a cross-language package manager that handles non-Python dependencies and manages binary packages.

4. **Q**: How do you programmatically check if a virtual environment is active?
   **A**: Check `sys.prefix != sys.base_prefix` in Python, or check if `VIRTUAL_ENV` is set in the environment.

5. **Q**: Why should you avoid using `sudo pip install`?
   **A**: It installs packages system-wide, polluting the system Python, and potentially breaking OS tools that depend on specific package versions. It also poses security risks and makes it harder to reproduce environments.

## Coding Challenges

1. **Venv Auditor**: Write a tool that scans a directory tree, finds all `.venv` directories, and reports their Python version, package count, and total disk usage.

2. **Dependency Sync**: Create a script that takes two venv directories as input and reports which packages differ in version, including transitive dependencies.

3. **Venv Creation Benchmark**: Write a benchmark that measures creation time, install time for a standard set of packages, and total size for venv, virtualenv, and uv.

4. **Activation Script Generator**: Generate cross-platform activation scripts (PowerShell, bash, fish) for a given venv path.

5. **Dependency Upgrade Simulator**: Given a venv and a list of packages, simulate upgrading each to its latest version and report which would break existing constraints.

## Summary

Virtual environments are essential for isolating project dependencies and preventing version conflicts. Python's built-in `venv` module provides a lightweight solution, while tools like pipenv, poetry, pyenv, and uv offer enhanced workflows. Proper venv usage—creating one per project, pinning dependencies, and never installing system-wide—is a hallmark of professional Python development.

## Related Topics

pip (55.x), Creating Packages (58.x), Import System (57.x), Standard Library (54.x)
