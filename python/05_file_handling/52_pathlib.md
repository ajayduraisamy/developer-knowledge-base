# Pathlib - Path(), PurePath, path operations, glob (Python 3.4+)

## Introduction

`pathlib` is a Python module introduced in Python 3.4 that provides an object-oriented approach to working with filesystem paths. Instead of representing paths as strings, `pathlib` represents them as `Path` objects with methods and properties that make path manipulation intuitive and less error-prone.

The module offers two main types of paths: `PurePath` (for purely computational operations without I/O) and `Path` (for actual filesystem operations). Both come in Windows and POSIX variants: `PureWindowsPath`, `PurePosixPath`, `WindowsPath`, and `PosixPath`.

`pathlib` addresses the pain points of string-based path handling: cross-platform separator differences, complex path joining, and error-prone string manipulation for tasks like getting parent directories, file extensions, and path components.

## Why It Is Important

Path handling is a fundamental requirement in almost every Python program. `pathlib` is important because:

- **Cross-Platform Compatibility**: Automatically uses the correct path separator (/ on POSIX, \\ on Windows)
- **Readable Code**: Method chaining and property access make path operations more readable than os.path functions
- **Comprehensive API**: Combines os.path, glob, os.listdir, shutil, and more in a single consistent interface
- **Type Safety**: Path objects are distinct from strings, preventing accidental misuse
- **Modern Python**: It's the recommended way to handle paths in Python 3.6+
- **Rich Comparison**: Path objects support equality, ordering, and hashing
- **Operator Overloading**: The `/` operator provides intuitive path joining

Mastering `pathlib` is essential for writing clean, portable Python code that interacts with the filesystem.

## Syntax

```python
from pathlib import Path, PurePath, PureWindowsPath, PurePosixPath

# Creating paths
p = Path('/usr/local/bin')       # POSIX path
p = Path('C:\\Users\\Alice')     # Windows path
p = Path()                        # Current directory
p = Path.home()                   # User home directory
p = Path.cwd()                    # Current working directory

# Path properties
p.parts                          # Tuple of path components
p.parent                         # Parent directory
p.parents                        # Sequence of all ancestors
p.name                           # Final component (file or directory name)
p.stem                           # Filename without extension
p.suffix                         # File extension
p.suffixes                       # List of all extensions
p.root                           # Root of the path
p.anchor                         # Drive letter + root

# Path joining with /
new_path = Path('/usr') / 'local' / 'bin' / 'app'

# Checking existence/type
p.exists()
p.is_file()
p.is_dir()
p.is_absolute()
p.is_symlink()

# Directory operations
p.mkdir(parents=True, exist_ok=True)
p.rmdir()                        # Remove empty directory
p.rename(target)
p.replace(target)

# File operations
p.read_text(encoding='utf-8')
p.write_text('content', encoding='utf-8')
p.read_bytes()
p.write_bytes(b'content')

# Globbing
list(p.glob('*.py'))             # Match in current directory
list(p.rglob('*.py'))            # Recursive match
list(p.glob('**/*.py'))          # Recursive match (Python 3.5+)
```

## Examples

### Basic Path Operations

```python
from pathlib import Path

# Creating paths
home = Path.home()
cwd = Path.cwd()
print(f"Home: {home}")
print(f"CWD: {cwd}")

# Path components
p = Path('/usr/local/bin/python3')
print(f"Parts: {p.parts}")
print(f"Parent: {p.parent}")
print(f"Name: {p.name}")
print(f"Stem: {p.stem}")
print(f"Suffix: {p.suffix}")
print(f"Root: {p.root}")

# Reading and writing files
file_path = Path('hello.txt')
file_path.write_text('Hello, pathlib!')
content = file_path.read_text()
print(f"File content: {content}")

# Path joining with /
config_dir = Path('/etc') / 'app' / 'config'
ini_file = config_dir / 'settings.ini'
print(f"Config file: {ini_file}")
```

### Checking File Properties

```python
from pathlib import Path
import os

path = Path('.')

print(f"Current directory exists: {path.exists()}")
print(f"Is directory: {path.is_dir()}")
print(f"Is file: {path.is_file()}")

for item in path.iterdir():
    item_type = 'DIR' if item.is_dir() else 'FILE' if item.is_file() else 'OTHER'
    size = item.stat().st_size if item.is_file() else 0
    print(f"[{item_type}] {item.name} ({size} bytes)")
```

### File Globbing

```python
from pathlib import Path

# Find all Python files
base = Path('.')
py_files = list(base.glob('*.py'))
print(f"Python files: {[f.name for f in py_files]}")

# Recursive search for all .md files
md_files = list(base.rglob('*.md'))
print(f"Markdown files: {len(md_files)} found")

# Search with multiple patterns
for ext in ['*.py', '*.md', '*.txt']:
    files = list(base.glob(ext))
    print(f"{ext}: {len(files)} files")
```

## Beginner Examples

### Example 1: File Organizer by Extension

```python
from pathlib import Path

def organize_files(directory):
    base = Path(directory)
    if not base.exists() or not base.is_dir():
        print(f"Directory not found: {directory}")
        return
    
    for item in base.iterdir():
        if item.is_file():
            ext = item.suffix.lower().lstrip('.')
            if not ext:
                ext = 'no_extension'
            
            ext_dir = base / ext
            if not ext_dir.exists():
                ext_dir.mkdir()
                print(f"Created directory: {ext_dir.name}")
            
            destination = ext_dir / item.name
            item.rename(destination)
            print(f"Moved: {item.name} -> {ext_dir.name}/")
    
    print("Organization complete!")

def create_test_files():
    test_dir = Path('test_files')
    test_dir.mkdir(exist_ok=True)
    
    for info in [('txt', 3), ('jpg', 2), ('py', 2), ('pdf', 1), ('docx', 1)]:
        ext, count = info
        for i in range(count):
            file = test_dir / f"file_{i+1}.{ext}"
            file.write_text(f"This is file {i+1}.{ext}")
    
    print(f"Created test files in {test_dir}")

if __name__ == '__main__':
    create_test_files()
    organize_files('test_files')
```

### Example 2: Directory Tree Viewer

```python
from pathlib import Path

def show_directory_tree(start_path, indent='', max_depth=3, current_depth=0):
    if current_depth > max_depth:
        return
    
    path = Path(start_path)
    if not path.is_dir():
        print(f"{indent}📄 {path.name}")
        return
    
    print(f"{indent}📁 {path.name}/")
    indent += '    '
    
    try:
        items = sorted(path.iterdir(), key=lambda x: (not x.is_dir(), x.name.lower()))
        for item in items:
            if item.is_dir():
                show_directory_tree(item, indent, max_depth, current_depth + 1)
            else:
                size = item.stat().st_size
                print(f"{indent}📄 {item.name} ({size:,} bytes)")
    except PermissionError:
        print(f"{indent}[Permission Denied]")

def count_files_by_type(directory):
    path = Path(directory)
    extensions = {}
    
    for file in path.rglob('*'):
        if file.is_file():
            ext = file.suffix.lower() or 'no_ext'
            extensions[ext] = extensions.get(ext, 0) + 1
    
    print(f"\nFile count by type in '{directory}':")
    for ext, count in sorted(extensions.items(), key=lambda x: -x[1]):
        print(f"  {ext or 'no extension':<10}: {count}")

show_directory_tree('.', max_depth=2)
count_files_by_type('.')
```

### Example 3: Batch File Renamer

```python
from pathlib import Path
import re

def batch_rename(directory, pattern, replacement, dry_run=True):
    """Rename files matching a pattern.
    
    Args:
        directory: Target directory
        pattern: Regex pattern to match
        replacement: Replacement string (can use \\1, \\2 etc.)
        dry_run: If True, only print what would be renamed
    """
    path = Path(directory)
    if not path.is_dir():
        print(f"Not a directory: {directory}")
        return
    
    renamed = 0
    for item in path.iterdir():
        if item.is_file():
            new_name = re.sub(pattern, replacement, item.name)
            if new_name != item.name:
                new_path = item.parent / new_name
                if dry_run:
                    print(f"Would rename: {item.name} -> {new_name}")
                else:
                    item.rename(new_path)
                    print(f"Renamed: {item.name} -> {new_name}")
                renamed += 1
    
    print(f"\n{'Would rename' if dry_run else 'Renamed'} {renamed} file(s)")

def add_date_prefix(directory, date_str):
    """Add date prefix to all files in directory."""
    path = Path(directory)
    for item in path.iterdir():
        if item.is_file():
            new_name = f"{date_str}_{item.name}"
            item.rename(item.parent / new_name)
            print(f"Renamed: {item.name} -> {new_name}")

def make_sequential(directory, prefix='file', start=1, digits=3):
    """Rename all files in directory to sequential names."""
    path = Path(directory)
    for i, item in enumerate(sorted(path.iterdir()), start):
        if item.is_file():
            ext = item.suffix
            new_name = f"{prefix}_{str(i).zfill(digits)}{ext}"
            item.rename(item.parent / new_name)
            print(f"Renamed: {item.name} -> {new_name}")

if __name__ == '__main__':
    test_dir = Path('rename_test')
    test_dir.mkdir(exist_ok=True)
    for name in ['report_2024.txt', 'report_2023.txt', 'summary_2024.txt', 'notes.txt']:
        (test_dir / name).write_text('test')
    
    print("Dry run:")
    batch_rename('rename_test', r'report_(\d{4})', r'document_\1', dry_run=True)
```

## Intermediate Examples

### Example 4: File Synchronization Tool

```python
from pathlib import Path
import hashlib
import shutil
from datetime import datetime

class FileSync:
    def __init__(self, source, destination):
        self.source = Path(source)
        self.destination = Path(destination)
    
    def sync(self):
        """Synchronize source to destination."""
        if not self.source.exists():
            print(f"Source does not exist: {self.source}")
            return
        
        self.destination.mkdir(parents=True, exist_ok=True)
        
        synced = 0
        skipped = 0
        
        for src_path in self.source.rglob('*'):
            rel_path = src_path.relative_to(self.source)
            dest_path = self.destination / rel_path
            
            if src_path.is_dir():
                dest_path.mkdir(exist_ok=True)
                continue
            
            if dest_path.exists():
                if self._files_equal(src_path, dest_path):
                    skipped += 1
                    continue
            
            shutil.copy2(src_path, dest_path)
            synced += 1
            print(f"Synced: {rel_path}")
        
        print(f"\nSync complete: {synced} copied, {skipped} skipped")
    
    def _files_equal(self, path1, path2):
        if path1.stat().st_size != path2.stat().st_size:
            return False
        
        hash1 = hashlib.md5(path1.read_bytes()).hexdigest()
        hash2 = hashlib.md5(path2.read_bytes()).hexdigest()
        return hash1 == hash2
    
    def show_diff(self):
        """Show files that differ between source and destination."""
        print("\nDifferences:")
        
        for src_path in self.source.rglob('*'):
            rel_path = src_path.relative_to(self.source)
            dest_path = self.destination / rel_path
            
            if src_path.is_file():
                if not dest_path.exists():
                    print(f"  Only in source: {rel_path}")
                elif not self._files_equal(src_path, dest_path):
                    print(f"  Modified: {rel_path}")
        
        for dest_path in self.destination.rglob('*'):
            rel_path = dest_path.relative_to(self.destination)
            src_path = self.source / rel_path
            
            if dest_path.is_file() and not src_path.exists():
                print(f"  Only in destination: {rel_path}")

source = Path('source_dir')
dest = Path('dest_dir')

source.mkdir(exist_ok=True)
(source / 'file1.txt').write_text('Hello')
(source / 'sub').mkdir(exist_ok=True)
(source / 'sub' / 'file2.txt').write_text('World')

syncer = FileSync(source, dest)
syncer.sync()

(source / 'file1.txt').write_text('Modified!')
syncer.show_diff()
syncer.sync()
```

### Example 5: Config File Locator and Parser

```python
from pathlib import Path
import configparser

class ConfigManager:
    def __init__(self, app_name):
        self.app_name = app_name
        self.config_dirs = self._get_config_dirs()
        self.config = configparser.ConfigParser()
    
    def _get_config_dirs(self):
        """Get platform-specific config directory candidates."""
        home = Path.home()
        candidates = []
        
        if home:
            candidates.extend([
                home / '.config' / self.app_name,
                home / f'.{self.app_name}',
                Path('/etc') / self.app_name,
                Path.cwd() / 'config',
            ])
        
        return candidates
    
    def find_config(self, filename='config.ini'):
        """Find a config file in the search path."""
        for config_dir in self.config_dirs:
            config_file = config_dir / filename
            if config_file.exists():
                print(f"Found config: {config_file}")
                return config_file
        return None
    
    def load_config(self, filename='config.ini'):
        """Load and parse configuration."""
        config_file = self.find_config(filename)
        if config_file:
            self.config.read(str(config_file))
            print(f"Loaded config from: {config_file}")
            return True
        print("No config file found, using defaults")
        return False
    
    def get(self, section, key, fallback=None):
        return self.config.get(section, key, fallback=fallback)
    
    def create_default_config(self, directory=None):
        """Create a default config file."""
        if directory is None:
            directory = self.config_dirs[0]
        
        dir_path = Path(directory)
        dir_path.mkdir(parents=True, exist_ok=True)
        
        config_file = dir_path / 'config.ini'
        if config_file.exists():
            print(f"Config already exists: {config_file}")
            return
        
        self.config['General'] = {
            'debug': 'false',
            'language': 'en',
            'theme': 'light'
        }
        self.config['Database'] = {
            'host': 'localhost',
            'port': '5432',
            'database': self.app_name,
            'user': 'admin'
        }
        
        with open(config_file, 'w') as f:
            self.config.write(f)
        
        print(f"Created default config: {config_file}")

app = ConfigManager('myapp')
app.create_default_config()
app.load_config()
print(f"Debug mode: {app.get('General', 'debug')}")
print(f"DB host: {app.get('Database', 'host')}")
```

### Example 6: Temporary Project Scaffolder

```python
from pathlib import Path
from datetime import datetime

class ProjectScaffolder:
    TEMPLATES = {
        'python': {
            'src/{project_name}/__init__.py': '# {project_name}\n',
            'src/{project_name}/main.py': 'def main():\n    pass\n\nif __name__ == "__main__":\n    main()\n',
            'tests/__init__.py': '',
            'tests/test_main.py': 'def test_main():\n    assert True\n',
            'README.md': '# {project_name}\n\n## Description\n\n## Installation\n\n## Usage\n',
            'requirements.txt': '# dependencies\n',
            '.gitignore': '__pycache__/\n*.pyc\n.env\nvenv/\n',
            'setup.py': 'from setuptools import setup, find_packages\n\nsetup(\n    name="{project_name}",\n    version="0.1.0",\n    packages=find_packages("src"),\n    package_dir={"": "src"},\n)\n',
        },
        'web': {
            'public/index.html': '<!DOCTYPE html>\n<html>\n<head>\n    <title>{project_name}</title>\n</head>\n<body>\n    <h1>{project_name}</h1>\n</body>\n</html>\n',
            'public/css/style.css': '/* {project_name} styles */\n',
            'public/js/app.js': '// {project_name} application\n',
            'package.json': '{{\n    "name": "{project_name}",\n    "version": "1.0.0"\n}}\n',
        }
    }
    
    def __init__(self, project_name, project_type='python'):
        self.project_name = project_name
        self.project_type = project_type
        self.base_path = Path.cwd() / project_name
    
    def scaffold(self):
        if self.base_path.exists():
            print(f"Project directory already exists: {self.base_path}")
            return False
        
        template = self.TEMPLATES.get(self.project_type)
        if not template:
            print(f"Unknown project type: {self.project_type}")
            return False
        
        self.base_path.mkdir(parents=True)
        print(f"Creating project: {self.project_name}")
        
        for relative_path, content in template.items():
            file_path = self.base_path / relative_path.format(project_name=self.project_name)
            file_path.parent.mkdir(parents=True, exist_ok=True)
            
            formatted_content = content.format(
                project_name=self.project_name,
                year=datetime.now().year
            )
            
            file_path.write_text(formatted_content)
            print(f"  Created: {relative_path}")
        
        print(f"\nProject scaffolded at: {self.base_path}")
        return True

scaffolder = ProjectScaffolder('my_awesome_project', 'python')
scaffolder.scaffold()

print("\nProject structure:")
for item in sorted(scaffolder.base_path.rglob('*')):
    rel = item.relative_to(scaffolder.base_path)
    prefix = '📁' if item.is_dir() else '📄'
    print(f"  {prefix} {rel}")
```

## Advanced Examples

### Example 7: File Integrity Monitor

```python
from pathlib import Path
import hashlib
import json
from datetime import datetime

class FileIntegrityMonitor:
    def __init__(self, base_dir, state_file='.integrity.json'):
        self.base_dir = Path(base_dir)
        self.state_file = self.base_dir / state_file
        self.state = self._load_state()
    
    def _load_state(self):
        if self.state_file.exists():
            return json.loads(self.state_file.read_text())
        return {}
    
    def _save_state(self):
        self.state_file.write_text(json.dumps(self.state, indent=2))
    
    def _hash_file(self, file_path):
        hasher = hashlib.sha256()
        for chunk in iter(lambda: file_path.read_bytes(), b''):
            hasher.update(chunk)
        return hasher.hexdigest()
    
    def scan(self):
        """Scan all files and record their hashes."""
        self.state = {}
        for file_path in self.base_dir.rglob('*'):
            if file_path.is_file() and file_path.name != self.state_file.name:
                relative = str(file_path.relative_to(self.base_dir))
                self.state[relative] = {
                    'hash': self._hash_file(file_path),
                    'size': file_path.stat().st_size,
                    'modified': datetime.fromtimestamp(file_path.stat().st_mtime).isoformat()
                }
        self._save_state()
        print(f"Scanned {len(self.state)} files")
    
    def check(self):
        """Check current files against recorded state."""
        if not self.state:
            print("No state recorded. Run scan() first.")
            return
        
        changes = []
        
        current_files = set()
        for file_path in self.base_dir.rglob('*'):
            if file_path.is_file() and file_path.name != self.state_file.name:
                relative = str(file_path.relative_to(self.base_dir))
                current_files.add(relative)
                
                if relative in self.state:
                    current_hash = self._hash_file(file_path)
                    if current_hash != self.state[relative]['hash']:
                        changes.append(('MODIFIED', relative))
                else:
                    changes.append(('ADDED', relative))
        
        for relative in self.state:
            if relative not in current_files:
                changes.append(('DELETED', relative))
        
        if changes:
            print(f"\nFound {len(changes)} change(s):")
            for change_type, path in sorted(changes):
                print(f"  [{change_type}] {path}")
        else:
            print("No changes detected.")
        
        return changes

monitor = FileIntegrityMonitor('monitored_dir')
Path('monitored_dir').mkdir(exist_ok=True)
Path('monitored_dir/test.txt').write_text('Hello World')
Path('monitored_dir/sub').mkdir(exist_ok=True)
Path('monitored_dir/sub/data.txt').write_text('Some data')

monitor.scan()

Path('monitored_dir/test.txt').write_text('Modified content!')
Path('monitored_dir/new_file.txt').write_text('New file!')
Path('monitored_dir/sub/data.txt').unlink()

monitor.check()
```

### Example 8: Event-Driven File Watcher

```python
from pathlib import Path
import time
from datetime import datetime

class FileWatcher:
    def __init__(self, directory, callback=None):
        self.directory = Path(directory)
        self.callback = callback
        self.files_state = {}
        self.running = False
    
    def _get_file_state(self):
        state = {}
        for file_path in self.directory.rglob('*'):
            if file_path.is_file():
                stat = file_path.stat()
                state[str(file_path.relative_to(self.directory))] = {
                    'size': stat.st_size,
                    'mtime': stat.st_mtime
                }
        return state
    
    def _handle_change(self, event_type, file_path, details=None):
        message = f"[{datetime.now().isoformat()}] {event_type}: {file_path}"
        if details:
            message += f" ({details})"
        print(message)
        
        if self.callback:
            self.callback(event_type, file_path, details)
    
    def start(self, interval=1.0):
        print(f"Watching directory: {self.directory}")
        self.files_state = self._get_file_state()
        self.running = True
        
        try:
            while self.running:
                time.sleep(interval)
                current_state = self._get_file_state()
                
                for filename, state in current_state.items():
                    if filename not in self.files_state:
                        self._handle_change('CREATED', filename)
                    elif state['mtime'] != self.files_state[filename]['mtime']:
                        self._handle_change('MODIFIED', filename)
                    elif state['size'] != self.files_state[filename]['size']:
                        self._handle_change('SIZE_CHANGED', filename)
                
                for filename in self.files_state:
                    if filename not in current_state:
                        self._handle_change('DELETED', filename)
                
                self.files_state = current_state
        except KeyboardInterrupt:
            print("\nWatcher stopped.")
    
    def stop(self):
        self.running = False

def on_file_change(event, filename, details):
    if event == 'CREATED':
        print(f"  -> New file detected!")
    elif event == 'MODIFIED':
        file_path = Path('watched_dir') / filename
        if file_path.suffix == '.py':
            print(f"  -> Python file changed, re-running...")

watcher_dir = Path('watched_dir')
watcher_dir.mkdir(exist_ok=True)

print("Creating test files (in another terminal or this script):")
test_file = watcher_dir / 'test.py'
test_file.write_text('# initial version\nprint("hello")\n')

watcher = FileWatcher('watched_dir', callback=on_file_change)
print("\nStarting watcher (Ctrl+C to stop)...")
print("Modify files in 'watched_dir' to see events")
watcher.start(interval=1.0)
```

## Real-World Use Cases

1. **Build Systems**: Finding source files, managing build artifacts, and cleaning build directories using pathlib's intuitive API.

2. **Data Processing Pipelines**: Traversing directory trees to find input files, creating output directory structures, and managing intermediate files.

3. **Configuration Management**: Locating configuration files across platform-specific paths, reading/writing config, and creating default configurations.

4. **Log Rotation**: Monitoring log file sizes, archiving old logs, and managing log directory structures.

5. **File Backup Systems**: Comparing source and backup directories, identifying new/modified files, and creating incremental backups.

6. **Development Tools**: Project scaffolding, code generation, and file watchers for automatic recompilation.

## Common Mistakes

1. **Ignoring Cross-Platform Differences**: Using hardcoded forward slashes on Windows or backslashes on POSIX. Use pathlib's `/` operator instead.

2. **Not Using parents=True with mkdir()**: `Path.mkdir()` fails if parent directories don't exist unless `parents=True` is specified.

3. **Calling is_file()/is_dir() on non-existent paths**: These return False for non-existent paths, which can hide bugs. Check `exists()` first.

4. **Converting to String Unnecessarily**: Many libraries now accept Path objects. Avoid `str(path)` unless required by an API that only accepts strings.

5. **Modifying Paths as Strings**: Don't use string operations on path strings. Use pathlib methods like `with_name()`, `with_suffix()`, or `with_stem()`.

## Best Practices

1. **Use Path Over PurePath**: Unless you're only doing string operations without filesystem access, use `Path` for the full API.

2. **Use / Operator for Joining**: The `/` operator is more readable than `os.path.join()` and automatically handles separators.

3. **Use rglob for Recursive Searches**: `path.rglob('*.py')` is cleaner than `path.glob('**/*.py')`.

4. **Use Path.open() Instead of open()**: `path.open()` is a method on Path objects and works identically to the built-in `open()`.

5. **Prefer Path Methods Over os.path**: Use `path.parent`, `path.name`, `path.stem`, `path.suffix` instead of `os.path.dirname()`, `os.path.basename()`, etc.

## Interview Questions

**Q1: What is the difference between PurePath and Path?**

A: `PurePath` provides path computation operations (joining, splitting, etc.) without filesystem access. `Path` inherits from PurePath and adds filesystem operations like `exists()`, `mkdir()`, `read_text()`, etc.

**Q2: How do you join paths in pathlib?**

A: Use the `/` operator: `Path('/usr') / 'local' / 'bin'` produces `Path('/usr/local/bin')`. This works with both Path objects and strings.

**Q3: What does `Path.rglob('*.py')` do?**

A: It recursively searches for all Python files matching the pattern `*.py` starting from the path's directory and all subdirectories. It's equivalent to `Path.glob('**/*.py')`.

**Q4: How do you get the filename without extension in pathlib?**

A: Use the `.stem` property. For `Path('/path/to/file.txt')`, `.stem` returns `'file'` and `.suffix` returns `'.txt'`.

## Coding Challenges

**Challenge 1: Duplicate File Finder** - Recursively find duplicate files in a directory tree using pathlib and hashlib.

**Challenge 2: Directory Size Calculator** - Calculate the total size of all files in a directory tree, showing per-directory breakdowns.

**Challenge 3: File History Tracker** - Track file modifications over time, storing snapshots of file states.

**Challenge 4: Template-Based Project Generator** - Create a more sophisticated project scaffolder that reads templates from a configuration file.

**Challenge 5: Log File Archiver** - Automatically archive and compress log files older than a configurable number of days.

## Summary

`pathlib` provides an object-oriented, cross-platform interface for filesystem path operations. The `Path` class combines path manipulation with filesystem access methods. Key features include the `/` operator for path joining, properties like `.parent`, `.name`, `.stem`, `.suffix`, recursive file matching with `rglob()`, and direct file I/O with `read_text()`/`write_text()`. `pathlib` replaces most `os.path` functions and is the recommended approach for path handling in modern Python.

## Related Topics

- `48_file_operations.md` - File I/O operations that work with Path objects
- `49_json.md` - JSON file reading/writing with pathlib
- `50_csv.md` - CSV file path management
- `51_pickle.md` - Pickle file path handling
- `53_temp_files.md` - Temporary file management with pathlib
- os.path module - The legacy string-based path API
- glob module - Legacy file pattern matching
- shutil module - High-level file operations using pathlib
