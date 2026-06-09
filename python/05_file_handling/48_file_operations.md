# File Operations - open(), read(), write(), with statement

## Introduction

File operations are a fundamental aspect of programming that allow you to interact with the file system. Python provides built-in functions and methods to create, read, update, and delete files. The primary function for file handling is `open()`, which returns a file object that serves as the interface to the underlying file resource.

File operations in Python are designed to be simple and intuitive while providing the flexibility needed for complex scenarios. Python abstracts away the underlying operating system differences, providing a unified interface for file handling across Windows, macOS, and Linux.

Understanding file operations is crucial because almost every application needs to persist data, read configuration files, process data files, or generate output files. Python's file handling capabilities range from simple text file reading to complex binary file manipulation.

## Why It Is Important

File operations are the bridge between your program's in-memory data and persistent storage on disk. Without file operations, data would be lost when a program terminates. File operations enable:

- **Data Persistence**: Saving program state, user data, and application settings between sessions
- **Data Processing**: Reading and processing large datasets from files (logs, CSVs, configuration files)
- **Inter-Program Communication**: Sharing data between different programs through files
- **Logging and Auditing**: Writing logs for debugging, monitoring, and compliance
- **Configuration Management**: Reading configuration files to control application behavior without code changes
- **Backup and Recovery**: Creating backups of critical data and restoring from backups
- **Import/Export Functionality**: Handling data interchange between different systems

Mastering file operations is essential for any Python developer because file I/O is involved in virtually every real-world application, from simple scripts to large-scale data processing systems.

## Syntax

```python
# Basic open() syntax
file_object = open(file, mode='r', buffering=-1, encoding=None, errors=None, newline=None, closefd=True, opener=None)

# Common modes:
# 'r'  - Read (default)
# 'w'  - Write (overwrites existing content)
# 'a'  - Append (adds to end of file)
# 'x'  - Exclusive creation (fails if file exists)
# 'b'  - Binary mode (append to mode, e.g., 'rb', 'wb')
# '+'  - Read and write (append to mode, e.g., 'r+', 'w+')
# 't'  - Text mode (default, e.g., 'rt' is same as 'r')

# Reading methods
content = file.read()            # Read entire file
content = file.read(size)        # Read specified number of bytes/characters
line = file.readline()           # Read one line
lines = file.readlines()         # Read all lines into a list

# Writing methods
bytes_written = file.write(string)     # Write string
file.writelines(list_of_strings)       # Write list of strings (no newlines added)

# File position methods
position = file.tell()           # Get current position
file.seek(offset, whence=0)      # Move to position
# whence: 0=start, 1=current, 2=end

# The with statement (context manager)
with open('file.txt', 'r') as f:
    content = f.read()
# File is automatically closed after the block
```

## Examples

### Basic File Reading and Writing

```python
# Writing to a file
with open('example.txt', 'w') as f:
    f.write('Hello, World!\n')
    f.write('This is a second line.\n')
    f.write('This is a third line.\n')

# Reading the entire file
with open('example.txt', 'r') as f:
    content = f.read()
print(content)
# Output:
# Hello, World!
# This is a second line.
# This is a third line.

# Reading line by line
with open('example.txt', 'r') as f:
    for line in f:
        print(repr(line))
# Output:
# 'Hello, World!\n'
# 'This is a second line.\n'
# 'This is a third line.\n'
```

### Appending to a File

```python
# Appending to an existing file
with open('example.txt', 'a') as f:
    f.write('This line is appended.\n')

# Verify the append
with open('example.txt', 'r') as f:
    print(f.read())
# Output:
# Hello, World!
# This is a second line.
# This is a third line.
# This line is appended.
```

### Using seek() and tell()

```python
with open('example.txt', 'w') as f:
    f.write('0123456789')

with open('example.txt', 'r') as f:
    print(f'tell(): {f.tell()}')    # 0
    print(f.read(5))                # 01234
    print(f'tell(): {f.tell()}')    # 5
    f.seek(0)                       # Go back to beginning
    print(f.read(3))                # 012
    f.seek(7)
    print(f.read())                 # 789
```

### Binary File Operations

```python
# Writing binary data
data = bytes([0, 1, 2, 3, 4, 5, 255])
with open('binary.bin', 'wb') as f:
    f.write(data)

# Reading binary data
with open('binary.bin', 'rb') as f:
    content = f.read()
    print(content)
    print(list(content))
# Output: b'\x00\x01\x02\x03\x04\x05\xff'
# Output: [0, 1, 2, 3, 4, 5, 255]
```

## Beginner Examples

### Example 1: Writing User Input to a File

```python
# Collect user information and save to file
name = input("Enter your name: ")
age = input("Enter your age: ")
email = input("Enter your email: ")

with open('user_info.txt', 'w') as f:
    f.write(f"Name: {name}\n")
    f.write(f"Age: {age}\n")
    f.write(f"Email: {email}\n")

print("User information saved to user_info.txt")

# Read and display the saved information
with open('user_info.txt', 'r') as f:
    print("\nSaved Information:")
    print(f.read())
```

### Example 2: Reading a File and Counting Lines

```python
# Count lines, words, and characters in a file
filename = input("Enter filename to analyze: ")

try:
    with open(filename, 'r') as f:
        content = f.read()
        lines = content.count('\n') + 1 if content else 0
        words = len(content.split())
        chars = len(content)
    
    print(f"\nFile Analysis: {filename}")
    print(f"Lines: {lines}")
    print(f"Words: {words}")
    print(f"Characters: {chars}")
except FileNotFoundError:
    print(f"Error: File '{filename}' not found.")
except PermissionError:
    print(f"Error: Permission denied to read '{filename}'.")
```

### Example 3: Creating a Simple To-Do List Manager

```python
# Simple to-do list manager using file operations
import sys

TODO_FILE = 'todo.txt'

def show_tasks():
    try:
        with open(TODO_FILE, 'r') as f:
            tasks = f.readlines()
            if tasks:
                print("\nYour To-Do List:")
                for i, task in enumerate(tasks, 1):
                    print(f"{i}. {task.strip()}")
            else:
                print("\nYour to-do list is empty.")
    except FileNotFoundError:
        print("\nNo to-do list found. Add some tasks!")

def add_task(task):
    with open(TODO_FILE, 'a') as f:
        f.write(task + '\n')
    print(f"Added: {task}")

def remove_task(task_num):
    try:
        with open(TODO_FILE, 'r') as f:
            tasks = f.readlines()
        
        if 1 <= task_num <= len(tasks):
            removed = tasks.pop(task_num - 1)
            with open(TODO_FILE, 'w') as f:
                f.writelines(tasks)
            print(f"Removed: {removed.strip()}")
        else:
            print(f"Invalid task number: {task_num}")
    except FileNotFoundError:
        print("No tasks to remove.")

# Main loop
while True:
    print("\n--- To-Do List Manager ---")
    print("1. Show tasks")
    print("2. Add task")
    print("3. Remove task")
    print("4. Exit")
    
    choice = input("Choose an option: ")
    
    if choice == '1':
        show_tasks()
    elif choice == '2':
        task = input("Enter task: ")
        add_task(task)
    elif choice == '3':
        show_tasks()
        try:
            num = int(input("Enter task number to remove: "))
            remove_task(num)
        except ValueError:
            print("Invalid number.")
    elif choice == '4':
        print("Goodbye!")
        break
    else:
        print("Invalid choice.")
```

## Intermediate Examples

### Example 4: File Copier with Progress Indicator

```python
import os

def copy_file_with_progress(source, destination, chunk_size=8192):
    """Copy a file from source to destination with progress indicator."""
    source_size = os.path.getsize(source)
    copied = 0
    
    with open(source, 'rb') as src, open(destination, 'wb') as dst:
        while True:
            chunk = src.read(chunk_size)
            if not chunk:
                break
            dst.write(chunk)
            copied += len(chunk)
            progress = (copied / source_size) * 100
            print(f"\rProgress: {progress:.1f}% ({copied}/{source_size} bytes)", end='')
    
    print("\nCopy complete!")

# Usage
# copy_file_with_progress('large_file.zip', 'large_file_backup.zip')
```

### Example 5: Log File Parser with Date Filtering

```python
from datetime import datetime

def parse_log_file(logfile, start_date=None, end_date=None):
    """Parse a log file and filter entries by date range.
    
    Expected log format: [2024-01-15 10:30:45] LEVEL: message
    """
    entries = []
    
    with open(logfile, 'r') as f:
        for line in f:
            try:
                # Extract timestamp from log line
                timestamp_str = line[1:21]  # Extract [2024-01-15 10:30:45]
                timestamp = datetime.strptime(timestamp_str, '%Y-%m-%d %H:%M:%S')
                
                if start_date and timestamp < start_date:
                    continue
                if end_date and timestamp > end_date:
                    continue
                
                entries.append((timestamp, line.strip()))
            except (ValueError, IndexError):
                continue
    
    return entries

def generate_log_summary(entries):
    """Generate summary statistics from log entries."""
    levels = {'INFO': 0, 'WARNING': 0, 'ERROR': 0, 'DEBUG': 0}
    error_messages = []
    
    for timestamp, entry in entries:
        for level in levels:
            if level in entry:
                levels[level] += 1
                if level == 'ERROR':
                    error_messages.append(entry)
                break
    
    return levels, error_messages

# Example usage
def create_sample_log():
    with open('sample.log', 'w') as f:
        f.write('[2024-01-15 10:30:45] INFO: Server started\n')
        f.write('[2024-01-15 10:31:00] DEBUG: Loading configuration\n')
        f.write('[2024-01-15 10:31:15] INFO: Database connected\n')
        f.write('[2024-01-15 10:32:00] WARNING: High memory usage detected\n')
        f.write('[2024-01-15 10:33:00] ERROR: Connection timeout to external API\n')
        f.write('[2024-01-15 10:34:00] INFO: Request processed successfully\n')
        f.write('[2024-01-15 10:35:00] ERROR: Disk space low\n')

if __name__ == '__main__':
    create_sample_log()
    
    entries = parse_log_file('sample.log')
    levels, errors = generate_log_summary(entries)
    
    print("Log Summary:")
    for level, count in levels.items():
        print(f"  {level}: {count}")
    
    if errors:
        print("\nErrors Found:")
        for error in errors:
            print(f"  {error}")
```

### Example 6: Read File in Reverse Order

```python
def read_file_reverse(filename, chunk_size=1024):
    """Read a file line by line in reverse order."""
    with open(filename, 'rb') as f:
        f.seek(0, 2)  # Seek to end
        file_size = f.tell()
        
        buffer = b''
        position = file_size
        
        lines = []
        while position > 0:
            read_size = min(chunk_size, position)
            position -= read_size
            f.seek(position)
            chunk = f.read(read_size)
            buffer = chunk + buffer
            
            # Process lines from buffer
            while b'\n' in buffer:
                line_bytes, buffer = buffer.rsplit(b'\n', 1)
                lines.append(line_bytes.decode('utf-8', errors='replace'))
        
        if buffer:
            lines.append(buffer.decode('utf-8', errors='replace'))
        
        return lines

# Create test file
def create_test_file():
    with open('reverse_test.txt', 'w') as f:
        for i in range(1, 11):
            f.write(f"This is line {i}\n")

if __name__ == '__main__':
    create_test_file()
    reversed_lines = read_file_reverse('reverse_test.txt')
    
    print("File read in reverse order:")
    for line in reversed_lines:
        print(line)
```

## Advanced Examples

### Example 7: Memory-Mapped File Processing

```python
import mmap
import os

def process_large_file_with_mmap(filename):
    """Process a large file using memory mapping for efficiency."""
    with open(filename, 'r+b') as f:
        # Memory-map the file
        with mmap.mmap(f.fileno(), 0) as mm:
            # Count occurrences of a pattern
            pattern = b'important'
            count = 0
            start = 0
            
            while True:
                pos = mm.find(pattern, start)
                if pos == -1:
                    break
                count += 1
                start = pos + 1
                # Read context around the match
                context_start = max(0, pos - 30)
                context_end = min(len(mm), pos + len(pattern) + 30)
                context = mm[context_start:context_end]
                print(f"Match at {pos}: ...{context.decode('utf-8', errors='replace')}...")
            
            print(f"Total matches found: {count}")
            
            # Modify content in-place
            mm[0:10] = b'MODIFIED '

# Create sample data
def create_large_sample():
    with open('large_sample.txt', 'w') as f:
        for i in range(100000):
            f.write(f"This is line {i} with some important data.\n")
            if i % 10000 == 0:
                f.write(f"CRITICAL: important flag at line {i}\n")

if __name__ == '__main__':
    create_large_sample()
    process_large_file_with_mmap('large_sample.txt')
```

### Example 8: Custom Buffered File Reader

```python
class BufferedFileReader:
    """A custom buffered file reader with line caching."""
    
    def __init__(self, filename, buffer_size=8192):
        self.filename = filename
        self.buffer_size = buffer_size
        self._file = open(filename, 'rb')
        self._buffer = b''
        self._buffer_position = 0
        self._file_position = 0
    
    def readline(self):
        """Read a line using the internal buffer."""
        while b'\n' not in self._buffer:
            chunk = self._file.read(self.buffer_size)
            if not chunk:
                break
            self._buffer += chunk
        
        if b'\n' in self._buffer:
            line, self._buffer = self._buffer.split(b'\n', 1)
            self._buffer_position += len(line) + 1
            return line.decode('utf-8', errors='replace') + '\n'
        elif self._buffer:
            line = self._buffer
            self._buffer = b''
            return line.decode('utf-8', errors='replace')
        else:
            return ''
    
    def readlines(self):
        """Read all lines into a list."""
        lines = []
        while True:
            line = self.readline()
            if not line:
                break
            lines.append(line)
        return lines
    
    def seek(self, position, whence=0):
        """Seek to a position in the file."""
        if whence == 0:
            self._file.seek(position)
        elif whence == 1:
            self._file.seek(position, 1)
        elif whence == 2:
            self._file.seek(position, 2)
        
        self._buffer = b''
        self._buffer_position = self._file.tell()
        self._file_position = self._file.tell()
    
    def tell(self):
        """Return current position."""
        return self._file_position
    
    def close(self):
        self._file.close()
    
    def __enter__(self):
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.close()
        return False

# Usage
if __name__ == '__main__':
    with open('test_buffer.txt', 'w') as f:
        for i in range(100):
            f.write(f"Line {i}: Python file operations are powerful.\n")
    
    with BufferedFileReader('test_buffer.txt', buffer_size=50) as reader:
        for i in range(5):
            line = reader.readline()
            print(f"Read: {line.strip()}")
            print(f"Position: {reader.tell()}")
```

### Example 9: Concurrent File Processing with Threading

```python
import threading
import os
import queue

def process_file_chunk(filename, start, end, result_queue):
    """Process a chunk of a file."""
    results = {
        'lines': 0,
        'words': 0,
        'chars': 0,
        'start': start,
        'end': end
    }
    
    with open(filename, 'r') as f:
        f.seek(start)
        # If not at start of file, read partial first line
        if start > 0:
            f.readline()
        
        while f.tell() < end:
            line = f.readline()
            if not line:
                break
            results['lines'] += 1
            results['words'] += len(line.split())
            results['chars'] += len(line)
    
    result_queue.put(results)

def parallel_file_analysis(filename, num_threads=4):
    """Analyze a file using multiple threads."""
    file_size = os.path.getsize(filename)
    chunk_size = file_size // num_threads
    
    threads = []
    result_queue = queue.Queue()
    
    for i in range(num_threads):
        start = i * chunk_size
        end = start + chunk_size if i < num_threads - 1 else file_size
        thread = threading.Thread(
            target=process_file_chunk,
            args=(filename, start, end, result_queue)
        )
        threads.append(thread)
        thread.start()
    
    for thread in threads:
        thread.join()
    
    # Aggregate results
    total_lines = 0
    total_words = 0
    total_chars = 0
    
    while not result_queue.empty():
        result = result_queue.get()
        total_lines += result['lines']
        total_words += result['words']
        total_chars += result['chars']
    
    return total_lines, total_words, total_chars

# Create large test file
def create_large_test_file():
    with open('large_analysis.txt', 'w') as f:
        import random
        import string
        for _ in range(100000):
            words = [' '.join(''.join(random.choices(string.ascii_lowercase, k=random.randint(3, 10))) 
                            for _ in range(random.randint(5, 15)))]
            f.write(words[0] + '\n')

if __name__ == '__main__':
    print("Creating test file...")
    create_large_test_file()
    
    print("Analyzing file...")
    lines, words, chars = parallel_file_analysis('large_analysis.txt', num_threads=4)
    
    print(f"\nResults:")
    print(f"Lines: {lines}")
    print(f"Words: {words}")
    print(f"Characters: {chars}")
```

## Real-World Use Cases

1. **Configuration File Management**: Reading YAML, JSON, or INI configuration files to configure application behavior without recompiling or modifying source code.

2. **Data ETL Pipelines**: Extracting data from source files (CSV, JSON, XML), transforming it (cleaning, validation, enrichment), and loading it into databases or data warehouses.

3. **Log Aggregation and Analysis**: Collecting and parsing log files from multiple services, extracting metrics, error patterns, and usage statistics for monitoring and debugging.

4. **File Synchronization Tools**: Implementing file sync utilities that compare source and destination directories, copying only changed files, and handling conflicts.

5. **Backup and Restore Systems**: Creating incremental and full backups of important data, with compression and encryption, and implementing restore functionality.

6. **Report Generation**: Reading template files, populating them with data from databases or other sources, and generating output reports in various formats (PDF, HTML, Excel).

7. **Web Scraping and Data Collection**: Saving scraped web content to files, managing crawl state in files, and processing collected data in batch.

8. **Incremental Data Processing**: Tracking processed records using file markers, resuming interrupted processing from where it left off.

## Common Mistakes

1. **Forgetting to Close Files**: Not using the `with` statement leads to file handles remaining open, potentially causing resource leaks and file corruption.

   ```python
   # Bad
   f = open('file.txt', 'r')
   content = f.read()
   # f never closed
   
   # Good
   with open('file.txt', 'r') as f:
       content = f.read()
   ```

2. **Reading Entire Large Files into Memory**: Using `read()` or `readlines()` on huge files can consume all available RAM.

   ```python
   # Bad - loads entire file into memory
   with open('huge_file.txt', 'r') as f:
       all_lines = f.readlines()
   
   # Good - processes line by line
   with open('huge_file.txt', 'r') as f:
       for line in f:
           process(line)
   ```

3. **Ignoring File Encoding**: Not specifying encoding can cause UnicodeDecodeError on files with non-ASCII characters.

   ```python
   # Bad
   with open('file.txt', 'r') as f:
       content = f.read()
   
   # Good
   with open('file.txt', 'r', encoding='utf-8') as f:
       content = f.read()
   ```

4. **Assuming File Always Exists**: Not handling FileNotFoundError leads to program crashes.

5. **Confusing Read/Write Modes**: Using 'w' instead of 'a' accidentally erases file contents. Using 'r' on a non-existent file raises an error.

6. **Not Flushing Buffers**: When writing critical data, buffering can cause data loss if the program crashes before the buffer is flushed.

7. **Hardcoding File Paths**: Hardcoded paths break when the program moves to a different environment.

8. **Not Handling Binary Mode Correctly**: Forgetting 'b' when working with binary files can corrupt data due to newline translation.

## Best Practices

1. **Always Use Context Managers**: Use the `with` statement to ensure files are properly closed, even if exceptions occur.

2. **Specify Encoding Explicitly**: Always specify `encoding='utf-8'` (or the appropriate encoding) when working with text files to avoid cross-platform issues.

3. **Process Files Line by Line**: For large files, iterate over the file object directly instead of reading the entire file into memory.

4. **Use os.path or pathlib for Path Operations**: Use `os.path.join()` or `pathlib.Path` for cross-platform path manipulation instead of string concatenation.

5. **Handle Exceptions Properly**: Catch specific exceptions like `FileNotFoundError`, `PermissionError`, and `IOError`.

6. **Use Temporary Files for Intermediate Data**: Use the `tempfile` module for temporary data instead of creating files in the current directory.

7. **Buffer Size Optimization**: For large file operations, experiment with different buffer sizes to find the optimal balance between memory usage and I/O performance.

8. **Use Binary Mode for Non-Text Files**: Always use binary mode ('rb', 'wb') for images, videos, archives, and other non-text files.

9. **Close Files Explicitly in Long-Running Apps**: In long-running applications, close files as soon as they're no longer needed to free system resources.

## Interview Questions

**Q1: What is the difference between `read()`, `readline()`, and `readlines()`?**

A: `read(size)` reads the entire file or up to `size` bytes/characters as a single string. `readline()` reads one line including the newline character at the end. `readlines()` reads all lines into a list of strings. For large files, iterating directly over the file object (`for line in file:`) is recommended over `readlines()`.

**Q2: Explain the difference between 'w', 'a', and 'x' modes.**

A: 'w' opens the file for writing, truncating (overwriting) the file if it exists or creating it if it doesn't. 'a' opens for appending, preserving existing content and adding new data at the end. 'x' opens for exclusive creation, failing if the file already exists (useful for avoiding accidental overwrites).

**Q3: How does the `with` statement work in file operations?**

A: The `with` statement uses context manager protocol. The file object's `__enter__` method returns the file object, and `__exit__` method (which calls `f.close()`) is automatically invoked when the block exits, even if an exception occurs. This ensures proper resource cleanup.

**Q4: What is the purpose of `seek()` and `tell()`?**

A: `tell()` returns the current position of the file pointer (in bytes from the beginning of the file). `seek(offset, whence)` moves the file pointer to a new position. `whence=0` (default) means relative to beginning, `whence=1` means relative to current position, `whence=2` means relative to end.

**Q5: How do you handle binary files in Python?**

A: Binary files are opened with 'b' in the mode string (e.g., 'rb', 'wb', 'ab'). Binary mode reads and writes `bytes` objects instead of strings. No encoding/decoding is performed, and no newline translation occurs. Binary mode is essential for images, audio, video, archives, and any non-text data.

## Coding Challenges

**Challenge 1: File Splitter**
Write a program that splits a large text file into multiple smaller files, each with a specified number of lines.

**Challenge 2: File Difference Finder**
Implement a simplified version of the `diff` command that compares two files line by line and reports additions, deletions, and modifications.

**Challenge 3: Keyword Search Across Files**
Write a recursive file search utility that searches for a keyword across all files in a directory tree and reports matching files with line numbers and context.

**Challenge 4: Log File Rotator**
Implement a log rotation system that automatically archives and compresses log files when they reach a certain size, keeping a configurable number of backups.

**Challenge 5: Binary File Hex Viewer**
Create a hex dump utility that displays binary files in hexadecimal format with an ASCII representation column, similar to the `xxd` command.

## Summary

File operations are a core competency for Python developers. The `open()` function provides the foundation, supporting various modes for reading, writing, appending, and binary access. The `with` statement simplifies resource management by automatically closing files. Understanding file positioning with `seek()` and `tell()` enables random access patterns. Python's file operations abstract away OS-specific details, providing a consistent interface across platforms. Best practices include always using context managers, specifying encoding explicitly, processing large files line by line, and handling exceptions appropriately. Mastery of file operations enables everything from simple data persistence to complex data processing pipelines.

## Related Topics

- `49_json.md` - JSON serialization and deserialization
- `50_csv.md` - CSV file reading and writing
- `51_pickle.md` - Python object serialization with pickle
- `52_pathlib.md` - Object-oriented file system paths
- `53_temp_files.md` - Temporary file and directory creation
- 05 Exception Handling - try/except blocks for robust file operations
- 06 Context Managers - understanding the context manager protocol
- os module - operating system interface for file operations
- shutil module - high-level file operations (copy, move, delete)
- gzip/bz2/lzma modules - compressed file handling
