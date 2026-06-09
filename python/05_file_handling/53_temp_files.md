# Temporary Files - tempfile.TemporaryFile, TemporaryDirectory

## Introduction

The `tempfile` module in Python provides a robust and secure way to create temporary files and directories. These temporary resources are automatically managed, typically created in system-specific temporary directories, and can be configured to be automatically deleted when no longer needed.

Temporary files are essential for programs that need to store intermediate data, cache computation results, or handle data that exceeds available memory. The `tempfile` module handles important details like unique naming, proper permissions, and cleanup, reducing the risk of security vulnerabilities and resource leaks.

The module offers several levels of abstraction, from high-level objects like `TemporaryFile` and `TemporaryDirectory` that handle all cleanup automatically, to lower-level functions like `mkstemp()` and `mkdtemp()` that give more control over the lifecycle.

## Why It Is Important

Temporary file management is crucial for many programming scenarios:

- **Intermediate Data Storage**: Store data that's too large for memory during processing
- **Caching**: Cache downloaded content or computation results temporarily
- **Unit Testing**: Create isolated test environments with temporary files and directories
- **Secure File Handling**: Create files with restricted permissions to prevent unauthorized access
- **Atomic Writes**: Write to a temporary file and rename atomically to prevent partial writes
- **External Tool Integration**: Pass temporary files to external programs or libraries
- **Automatic Cleanup**: Ensure temporary resources are cleaned up even when exceptions occur
- **Cross-Platform Portability**: Automatically use the correct temporary directory for each platform

Understanding `tempfile` is important for writing robust, secure applications that handle intermediate data correctly.

## Syntax

```python
import tempfile

# High-level: Automatic cleanup
with tempfile.TemporaryFile(mode='w+b', buffering=-1, encoding=None,
                            newline=None, suffix=None, prefix=None,
                            dir=None) as f:
    f.write(b'data')

with tempfile.NamedTemporaryFile(mode='w+b', buffering=-1, encoding=None,
                                 newline=None, suffix=None, prefix=None,
                                 dir=None, delete=True) as f:
    f.write(b'data')
    print(f.name)  # Has a visible name

with tempfile.TemporaryDirectory(suffix=None, prefix=None, dir=None) as tmpdir:
    print(tmpdir)

# Low-level: Manual cleanup
fd, path = tempfile.mkstemp(suffix=None, prefix=None, dir=None, text=False)
try:
    with os.fdopen(fd, 'w') as f:
        f.write('data')
finally:
    os.unlink(path)

dir_path = tempfile.mkdtemp(suffix=None, prefix=None, dir=None)
try:
    # Use directory
    pass
finally:
    shutil.rmtree(dir_path)

# Get temporary directory
tmp_dir = tempfile.gettempdir()
```

## Examples

### Basic TemporaryFile Usage

```python
import tempfile

# Create and use a temporary file (automatically deleted)
with tempfile.TemporaryFile() as f:
    f.write(b'Hello, temporary world!')
    f.seek(0)
    content = f.read()
    print(f"Content: {content}")
    print(f"File object type: {type(f)}")
# File is automatically deleted here

# TemporaryFile with text mode
with tempfile.TemporaryFile(mode='w+', encoding='utf-8') as f:
    f.write('Text mode works too')
    f.seek(0)
    print(f.read())
```

### NamedTemporaryFile

```python
import tempfile
import os

# Create a named temporary file
with tempfile.NamedTemporaryFile(delete=True) as f:
    print(f"Temp file name: {f.name}")
    print(f"Temp file exists: {os.path.exists(f.name)}")
    f.write(b'Named temporary file content')
    f.flush()
    print(f"File size: {os.path.getsize(f.name)} bytes")
# File is automatically deleted

# Keep the file after closing
with tempfile.NamedTemporaryFile(delete=False) as f:
    f.write(b'This file will persist')
    temp_name = f.name

print(f"File still exists: {os.path.exists(temp_name)}")
os.unlink(temp_name)  # Clean up manually
```

### TemporaryDirectory

```python
import tempfile
import os
from pathlib import Path

with tempfile.TemporaryDirectory() as tmp_dir:
    print(f"Temp directory: {tmp_dir}")
    
    # Create files in the temporary directory
    file1 = Path(tmp_dir) / 'test1.txt'
    file1.write_text('Hello from temp dir!')
    
    file2 = Path(tmp_dir) / 'sub' / 'test2.txt'
    file2.parent.mkdir()
    file2.write_text('Nested file')
    
    print(f"Files created: {list(Path(tmp_dir).rglob('*'))}")
# Directory and all contents are automatically deleted
```

## Beginner Examples

### Example 1: Simple Data Processor

```python
import tempfile
import os

def process_large_data(data_chunks):
    """Process data that might be too large for memory."""
    with tempfile.TemporaryFile(mode='w+') as tmp:
        for chunk in data_chunks:
            tmp.write(chunk + '\n')
        
        tmp.seek(0)
        
        # Process line by line
        line_count = 0
        total_chars = 0
        
        for line in tmp:
            line = line.strip()
            if line:
                line_count += 1
                total_chars += len(line)
        
        return {
            'lines': line_count,
            'characters': total_chars,
            'source': 'temporary file'
        }

data = [f"Line {i}: Sample data item {i}" for i in range(1000)]
result = process_large_data(data)
print(f"Processed {result['lines']} lines, {result['characters']} characters")
```

### Example 2: Secure File Downloader

```python
import tempfile
import os
import hashlib

def download_to_temp(url, verify_hash=None):
    """Download content to a temporary file (simulated)."""
    tmp = tempfile.NamedTemporaryFile(delete=False)
    print(f"Downloading to: {tmp.name}")
    
    try:
        # Simulate download
        content = f"Simulated content from {url}"
        tmp.write(content.encode())
        tmp.close()
        
        if verify_hash:
            with open(tmp.name, 'rb') as f:
                file_hash = hashlib.sha256(f.read()).hexdigest()
            if file_hash != verify_hash:
                raise ValueError("Hash verification failed!")
        
        return tmp.name
    except:
        os.unlink(tmp.name)
        raise

def process_temp_file(filepath):
    """Process the downloaded temporary file."""
    try:
        with open(filepath, 'r') as f:
            content = f.read()
        print(f"Processing: {content[:50]}...")
        return content
    finally:
        os.unlink(filepath)
        print(f"Cleaned up: {filepath}")

tmp_file = download_to_temp('https://example.com/data.txt')
result = process_temp_file(tmp_file)
```

### Example 3: Temporary Test Environment

```python
import tempfile
import os
import json
from pathlib import Path

class TestEnvironment:
    def __init__(self):
        self.temp_dir = tempfile.mkdtemp()
        self.files = {}
        print(f"Test environment created at: {self.temp_dir}")
    
    def create_file(self, name, content=''):
        file_path = Path(self.temp_dir) / name
        file_path.parent.mkdir(parents=True, exist_ok=True)
        file_path.write_text(content)
        self.files[name] = str(file_path)
        return str(file_path)
    
    def create_config(self, config_dict):
        config_path = os.path.join(self.temp_dir, 'config.json')
        with open(config_path, 'w') as f:
            json.dump(config_dict, f, indent=2)
        self.files['config'] = config_path
        return config_path
    
    def read_file(self, name):
        if name in self.files:
            with open(self.files[name], 'r') as f:
                return f.read()
        raise FileNotFoundError(f"File '{name}' not found in test environment")
    
    def cleanup(self):
        import shutil
        shutil.rmtree(self.temp_dir)
        print(f"Test environment cleaned up: {self.temp_dir}")
    
    def __enter__(self):
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.cleanup()
        return False

with TestEnvironment() as env:
    env.create_file('input.txt', 'Hello, World!')
    env.create_file('output/log.txt', 'Processing started')
    env.create_config({
        'debug': True,
        'max_retries': 3
    })
    
    print(f"Input file content: {env.read_file('input.txt')}")
    print(f"Config: {env.read_file('config')}")
```

## Intermediate Examples

### Example 4: Temporary Directory for Image Processing

```python
import tempfile
import os
from pathlib import Path
import shutil

class ImageProcessor:
    def __init__(self):
        self.temp_dir = None
    
    def process_images(self, input_files, operations):
        """Process images using temporary directory for intermediate files."""
        self.temp_dir = tempfile.mkdtemp(prefix='imgproc_')
        results = []
        
        try:
            for i, input_file in enumerate(input_files):
                if not os.path.exists(input_file):
                    print(f"File not found: {input_file}")
                    continue
                
                print(f"Processing {input_file}...")
                
                # Create intermediate file in temp directory
                intermediate = os.path.join(self.temp_dir, f'intermediate_{i}_step1.png')
                self._apply_operation(input_file, intermediate, operations.get('step1', 'resize'))
                
                result_file = os.path.join(self.temp_dir, f'result_{i}.png')
                self._apply_operation(intermediate, result_file, operations.get('step2', 'filter'))
                
                results.append(result_file)
            
            final_outputs = []
            for i, result in enumerate(results):
                output_path = f"output_image_{i}.png"
                shutil.copy2(result, output_path)
                final_outputs.append(output_path)
            
            return final_outputs
        
        finally:
            shutil.rmtree(self.temp_dir)
            print(f"Temporary files cleaned up: {self.temp_dir}")
    
    def _apply_operation(self, source, dest, operation):
        # Simulate image processing
        with open(source, 'w' if not os.path.exists(source) else 'a') as f:
            pass
        with open(dest, 'w') as f:
            f.write(f"Processed by {operation} from {source}")

# Create dummy input files
for i in range(3):
    Path(f'input_{i}.png').write_text(f'fake image {i}')

processor = ImageProcessor()
outputs = processor.process_images(
    ['input_0.png', 'input_1.png', 'input_2.png'],
    {'step1': 'resize', 'step2': 'sharpen'}
)
print(f"Output files: {outputs}")

for f in outputs + ['input_0.png', 'input_1.png', 'input_2.png']:
    if os.path.exists(f):
        os.unlink(f)
```

### Example 5: Atomic File Writer

```python
import tempfile
import os
import shutil

class AtomicWriter:
    """Write to a file atomically using a temporary file."""
    
    def __init__(self, filepath, mode='w', encoding='utf-8'):
        self.filepath = filepath
        self.mode = mode
        self.encoding = encoding
        self.temp_path = None
        self.temp_file = None
    
    def __enter__(self):
        dir_name = os.path.dirname(self.filepath) or '.'
        prefix = os.path.basename(self.filepath) + '.tmp.'
        
        fd, self.temp_path = tempfile.mkstemp(
            suffix='.tmp',
            prefix=prefix,
            dir=dir_name
        )
        
        if 'b' in self.mode:
            self.temp_file = os.fdopen(fd, self.mode)
        else:
            self.temp_file = os.fdopen(fd, self.mode, encoding=self.encoding)
        
        return self.temp_file
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.temp_file.close()
        
        if exc_type is None:
            os.replace(self.temp_path, self.filepath)
            print(f"Atomic write: {self.filepath}")
        else:
            os.unlink(self.temp_path)
            print(f"Write failed, temp file deleted: {self.temp_path}")
        
        return False

def safe_config_update(config_path, updates):
    """Update a config file atomically."""
    import json
    
    if os.path.exists(config_path):
        with open(config_path, 'r') as f:
            config = json.load(f)
    else:
        config = {}
    
    config.update(updates)
    
    with AtomicWriter(config_path) as f:
        json.dump(config, f, indent=2)

import json
safe_config_update('config_atomic.json', {'version': 2, 'debug': True})
print(json.loads(Path('config_atomic.json').read_text()))

# Test failure scenario
try:
    with AtomicWriter('config_atomic.json') as f:
        f.write('{"valid": true}')
        raise ValueError("Simulated error")
except ValueError:
    pass

Path('config_atomic.json').unlink(missing_ok=True)
```

### Example 6: Temporary Database for Testing

```python
import tempfile
import sqlite3
import os
from pathlib import Path

class TempDatabase:
    def __init__(self, schema=None):
        self.temp_dir = tempfile.TemporaryDirectory()
        self.db_path = os.path.join(self.temp_dir.name, 'test.db')
        self.connection = sqlite3.connect(self.db_path)
        self.cursor = self.connection.cursor()
        
        if schema:
            self.cursor.executescript(schema)
            self.connection.commit()
    
    def execute(self, query, params=None):
        if params:
            self.cursor.execute(query, params)
        else:
            self.cursor.execute(query)
        self.connection.commit()
        return self.cursor
    
    def fetchall(self):
        return self.cursor.fetchall()
    
    def fetchone(self):
        return self.cursor.fetchone()
    
    def close(self):
        self.connection.close()
        self.temp_dir.cleanup()

# Create schema
SCHEMA = """
CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE
);

CREATE TABLE IF NOT EXISTS posts (
    id INTEGER PRIMARY KEY,
    user_id INTEGER,
    title TEXT,
    content TEXT,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
"""

db = TempDatabase(SCHEMA)

# Insert test data
db.execute("INSERT INTO users (name, email) VALUES (?, ?)", ('Alice', 'alice@test.com'))
db.execute("INSERT INTO users (name, email) VALUES (?, ?)", ('Bob', 'bob@test.com'))
db.execute("INSERT INTO posts (user_id, title, content) VALUES (?, ?, ?)",
           (1, 'First Post', 'Hello World!'))

# Query
db.execute("SELECT u.name, p.title FROM users u JOIN posts p ON u.id = p.user_id")
print(f"User posts: {db.fetchall()}")

# Simulate application test
def test_user_creation():
    test_db = TempDatabase(SCHEMA)
    test_db.execute("INSERT INTO users (name, email) VALUES (?, ?)", ('Test', 'test@test.com'))
    test_db.execute("SELECT * FROM users WHERE email = ?", ('test@test.com',))
    result = test_db.fetchone()
    assert result is not None
    assert result[1] == 'Test'
    test_db.close()
    print("User creation test passed!")

test_user_creation()
db.close()
```

## Advanced Examples

### Example 7: Temporary File Pool Manager

```python
import tempfile
import os
import threading
from contextlib import contextmanager
from pathlib import Path

class TempFilePool:
    """A pool of reusable temporary files."""
    
    def __init__(self, max_files=10, prefix='pool_', suffix='.tmp'):
        self.max_files = max_files
        self.prefix = prefix
        self.suffix = suffix
        self.temp_dir = tempfile.mkdtemp(prefix='filepool_')
        self.available = []
        self.in_use = set()
        self.lock = threading.Lock()
        self._preallocate()
    
    def _preallocate(self):
        for _ in range(self.max_files):
            fd, path = tempfile.mkstemp(
                prefix=self.prefix,
                suffix=self.suffix,
                dir=self.temp_dir
            )
            os.close(fd)
            os.unlink(path)
            self.available.append(Path(path))
    
    def acquire(self):
        with self.lock:
            if not self.available:
                fd, path = tempfile.mkstemp(
                    prefix=self.prefix,
                    suffix=self.suffix,
                    dir=self.temp_dir
                )
                os.close(fd)
                path = Path(path)
            else:
                path = self.available.pop()
            
            self.in_use.add(path)
            path.write_text('')
            return path
    
    def release(self, path):
        with self.lock:
            if path in self.in_use:
                self.in_use.remove(path)
                path.write_text('')
                self.available.append(path)
    
    @contextmanager
    def get_temp_file(self):
        path = self.acquire()
        try:
            yield path
        finally:
            self.release(path)
    
    def cleanup(self):
        import shutil
        shutil.rmtree(self.temp_dir)
    
    def __enter__(self):
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.cleanup()
        return False

def parallel_processing_example():
    import concurrent.futures
    
    with TempFilePool(max_files=5) as pool:
        def process_item(item_id):
            with pool.get_temp_file() as tmp_file:
                tmp_file.write_text(f"Processing {item_id}")
                content = tmp_file.read_text()
                return f"Item {item_id}: {content}"
        
        with concurrent.futures.ThreadPoolExecutor(max_workers=10) as executor:
            results = list(executor.map(process_item, range(20)))
        
        for r in results:
            print(r)

parallel_processing_example()
```

### Example 8: Secure Temporary File with Auto-Encryption

```python
import tempfile
import os
import hashlib
import shutil
from cryptography.fernet import Fernet

class SecureTempFile:
    """A temporary file that encrypts its contents."""
    
    def __init__(self, suffix=None, prefix=None, dir=None):
        self.key = Fernet.generate_key()
        self.cipher = Fernet(self.key)
        
        fd, self.path = tempfile.mkstemp(suffix=suffix, prefix=prefix, dir=dir)
        os.close(fd)
    
    def write(self, data):
        if isinstance(data, str):
            data = data.encode()
        encrypted = self.cipher.encrypt(data)
        with open(self.path, 'wb') as f:
            f.write(encrypted)
    
    def read(self):
        with open(self.path, 'rb') as f:
            encrypted = f.read()
        decrypted = self.cipher.decrypt(encrypted)
        return decrypted
    
    def read_text(self):
        return self.read().decode()
    
    def get_key(self):
        return self.key
    
    def cleanup(self):
        if os.path.exists(self.path):
            # Overwrite with random data before deleting
            size = os.path.getsize(self.path)
            with open(self.path, 'wb') as f:
                f.write(os.urandom(size))
            os.unlink(self.path)
            print(f"Secure cleanup performed on: {self.path}")
    
    def __enter__(self):
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.cleanup()
        return False

with SecureTempFile(prefix='secure_', suffix='.enc') as stf:
    stf.write("This is highly sensitive data")
    content = stf.read_text()
    print(f"Decrypted content: {content}")
    
    with open(stf.path, 'rb') as f:
        encrypted_data = f.read()
    print(f"Encrypted bytes (first 50): {encrypted_data[:50]}")
    
    print(f"Temp file existed: {os.path.exists(stf.path)}")

print(f"Temp file cleaned up: {not os.path.exists(stf.path)}")
```

## Real-World Use Cases

1. **Web File Uploads**: Save uploaded files to temporary storage for processing before moving to permanent storage or performing validation.

2. **Batch Data Processing**: Store intermediate results during ETL operations where data exceeds available memory.

3. **Unit Testing**: Create isolated test environments with temporary files, directories, and databases that are automatically cleaned up after tests.

4. **Compiler/Interpreter Caches**: Store compilation artifacts or cached bytecode in temporary directories.

5. **Email Attachments**: Save email attachments temporarily for virus scanning before delivering to users.

6. **Document Conversion**: Store intermediate file formats during document conversion (e.g., DOCX to PDF).

7. **Database Backups**: Create temporary backup files during database dump and restore operations.

8. **Package Installation**: Extract package archives to temporary directories during installation.

## Common Mistakes

1. **Forgetting to Close Temporary Files**: Always use context managers (`with` statement) to ensure automatic cleanup.

2. **Leaking File Paths with delete=False**: When using `delete=False`, remember to manually clean up the file.

3. **Assuming Temporary Directory Location**: Don't assume `/tmp` or `C:\\Temp`. Use `tempfile.gettempdir()` to get the system's temporary directory.

4. **Not Handling Permission Errors**: Temporary directories may have restrictive permissions. Handle `PermissionError` appropriately.

5. **Creating Temporary Files in the Current Directory**: Using `dir='.'` pollutes the working directory. Use the default system temporary directory instead.

6. **Not Using Secure Permissions**: On multi-user systems, temporary files with default permissions may be readable by other users.

## Best Practices

1. **Always Use Context Managers**: Use `with` statements for `TemporaryFile()`, `NamedTemporaryFile()`, and `TemporaryDirectory()` to ensure automatic cleanup.

2. **Specify Prefix/Suffix for Debugging**: Use meaningful prefixes and suffixes to identify temporary files during debugging.

3. **Handle Cleanup on Errors**: Always clean up temporary resources in `finally` blocks or use context managers.

4. **Use TemporaryDirectory for Multiple Files**: When working with multiple temporary files, create a temporary directory to keep them organized.

5. **Don't Rely on Automatic Cleanup**: For critical applications, implement explicit cleanup in addition to relying on context managers.

6. **Be Aware of Disk Space**: Monitor and limit temporary file usage to avoid filling up the disk.

## Interview Questions

**Q1: What is the difference between TemporaryFile and NamedTemporaryFile?**

A: `TemporaryFile` creates an anonymous temporary file with no visible name in the filesystem (on Unix, the file is unlinked immediately after creation). `NamedTemporaryFile` creates a temporary file with a visible name in the filesystem that can be passed to other programs or examined.

**Q2: How does TemporaryDirectory work?**

A: `TemporaryDirectory` creates a temporary directory using `mkdtemp()` and returns its path. When used as a context manager, it automatically deletes the directory and all its contents when the context exits.

**Q3: What is the purpose of tempfile.gettempdir()?**

A: It returns the system's temporary directory (e.g., `/tmp` on Linux, `C:\\Temp` on Windows, `/var/tmp` on macOS). This is the default location where temporary files are created.

**Q4: When would you use mkstemp() instead of TemporaryFile()?**

A: `mkstemp()` gives you more control over the temporary file lifecycle. It returns a file descriptor and path, allowing you to manage the file yourself. Use it when you need to pass the file path to external programs or when you need custom cleanup logic.

## Coding Challenges

**Challenge 1: Temp File Cleanup Utility** - Create a tool that finds and cleans up orphaned temporary files older than a configurable age.

**Challenge 2: Temporary File Manager** - Implement a context manager that tracks all temporary files created within its scope and ensures cleanup.

**Challenge 3: Disk-Backed Cache** - Build a disk-backed cache using temporary files that automatically evicts least-recently-used entries.

**Challenge 4: Temp Directory Quota Manager** - Create a temporary directory manager that enforces disk usage quotas.

**Challenge 5: Encrypted Temp File Context Manager** - Implement a context manager that creates encrypted temporary files and securely wipes them on cleanup.

## Summary

Python's `tempfile` module provides secure, cross-platform temporary file and directory management. High-level classes like `TemporaryFile`, `NamedTemporaryFile`, and `TemporaryDirectory` offer automatic cleanup via context managers. Lower-level functions like `mkstemp()` and `mkdtemp()` give more control for advanced use cases. The module ensures unique filenames, appropriate permissions, and proper cleanup. Best practices include always using context managers, specifying meaningful prefixes/suffixes, and being mindful of disk space and security implications.

## Related Topics

- `48_file_operations.md` - File I/O operations with temporary files
- `49_json.md` - JSON processing with temporary files
- `50_csv.md` - CSV processing with temporary files
- `51_pickle.md` - Pickle serialization using temporary storage
- `52_pathlib.md` - Path handling for temporary file paths
- os module - Low-level file descriptor operations
- shutil module - High-level file operations for cleanup
- contextlib - Custom context managers for resource management
