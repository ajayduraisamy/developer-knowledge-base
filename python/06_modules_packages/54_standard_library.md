# Standard Library - os, sys, re, math, datetime, random, subprocess

## Introduction

Python's standard library is a vast collection of modules and packages that come bundled with every Python installation. It provides ready-to-use solutions for common programming tasks, from file I/O and string processing to networking, data serialization, and system interaction.

## Why It Is Important

The standard library eliminates the need to write everything from scratch or immediately reach for third-party packages. It is tested, documented, and guaranteed to be available wherever Python runs, making code more portable, maintainable, and secure.

## Syntax

```python
import module_name
from module_name import function_name
from module_name import function_one, function_two
from module_name import name as alias
import module_name.submodule
```

## Examples

### os — Operating System Interface

```python
import os

cwd = os.getcwd()
print(f"Current working directory: {cwd}")

os.makedirs("temp_dir", exist_ok=True)
print(f"Created temp_dir: {os.path.exists('temp_dir')}")

os.rmdir("temp_dir")
print(f"Removed temp_dir: {os.path.exists('temp_dir')}")

all_files = os.listdir(".")
print(f"Number of items in CWD: {len(all_files)}")

env_path = os.environ.get("PATH", "")
print(f"PATH variable length: {len(env_path)} chars")
```

### sys — System-Specific Parameters

```python
import sys

print(f"Python version: {sys.version}")
print(f"Platform: {sys.platform}")
print(f"Command line args: {sys.argv}")

sys.path.insert(0, "/tmp/custom_packages")
print(f"sys.path entries: {len(sys.path)}")

print(f"Recursion limit: {sys.getrecursionlimit()}")
sys.setrecursionlimit(5000)
print(f"New recursion limit: {sys.getrecursionlimit()}")

print(f"Default encoding: {sys.getdefaultencoding()}")
print(f"File system encoding: {sys.getfilesystemencoding()}")
```

### re — Regular Expressions

```python
import re

text = "Contact us at support@example.com or sales@company.org"

email_pattern = r"[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}"
emails = re.findall(email_pattern, text)
print(f"Emails found: {emails}")

match = re.search(r"(\w+)@(\w+)", text)
if match:
    print(f"Full match: {match.group()}")
    print(f"Username: {match.group(1)}, Domain: {match.group(2)}")

replaced = re.sub(r"@\w+\.\w+", "@redacted.com", text)
print(f"Redacted: {replaced}")

phone = "123-456-7890"
if re.fullmatch(r"\d{3}-\d{3}-\d{4}", phone):
    print(f"{phone} is a valid phone number")

parts = re.split(r"[,;]\s*", "apple, banana; cherry, date")
print(f"Split: {parts}")
```

### math — Mathematical Functions

```python
import math

print(f"Pi: {math.pi:.6f}")
print(f"Euler's number: {math.e:.6f}")
print(f"Infinity: {math.inf}")
print(f"Not a number: {math.nan}")

print(f"Square root of 144: {math.sqrt(144)}")
print(f"Ceil of 3.14: {math.ceil(3.14)}")
print(f"Floor of 3.14: {math.floor(3.14)}")
print(f"Factorial of 5: {math.factorial(5)}")
print(f"GCD of 48 and 18: {math.gcd(48, 18)}")

print(f"sin(pi/2): {math.sin(math.pi / 2)}")
print(f"cos(0): {math.cos(0)}")
print(f"tan(pi/4): {math.tan(math.pi / 4):.1f}")

print(f"log10(1000): {math.log10(1000)}")
print(f"log(e^2): {math.log(math.e ** 2)}")
print(f"2^10: {math.pow(2, 10)}")
```

### datetime — Dates and Times

```python
from datetime import datetime, date, time, timedelta, timezone

now = datetime.now()
print(f"Current datetime: {now}")
print(f"Today's date: {date.today()}")

utc_now = datetime.now(timezone.utc)
print(f"UTC time: {utc_now}")

parsed = datetime.strptime("2024-12-25 10:30:00", "%Y-%m-%d %H:%M:%S")
print(f"Parsed datetime: {parsed}")

formatted = parsed.strftime("%A, %B %d, %Y at %I:%M %p")
print(f"Formatted: {formatted}")

delta = timedelta(days=7, hours=3)
future = now + delta
print(f"One week + 3 hours from now: {future}")

diff = future - now
print(f"Difference in seconds: {diff.total_seconds():.0f}")
```

### random — Generate Random Numbers

```python
import random

print(f"Random float [0,1): {random.random():.4f}")
print(f"Random int 1-100: {random.randint(1, 100)}")
print(f"Random choice: {random.uniform(10.0, 20.0):.2f}")

fruits = ["apple", "banana", "cherry", "date", "elderberry"]
print(f"Random item: {random.choice(fruits)}")
print(f"Random sample (3): {random.sample(fruits, 3)}")

random.shuffle(fruits)
print(f"Shuffled: {fruits}")

print(f"Gaussian(0,1): {random.gauss(0, 1):.4f}")

random.seed(42)
print(f"Seeded choice: {random.choice(fruits)}")
random.seed(42)
print(f"Same choice: {random.choice(fruits)}")
```

### json — JSON Encoding and Decoding

```python
import json

data = {
    "name": "Alice",
    "age": 30,
    "skills": ["Python", "Data Science", "Machine Learning"],
    "active": True,
    "metadata": None,
    "scores": {"math": 95, "science": 88}
}

json_string = json.dumps(data, indent=2)
print(f"JSON string:\n{json_string}")

decoded = json.loads(json_string)
print(f"Decoded name: {decoded['name']}")
print(f"Decoded skills: {decoded['skills']}")

with open("sample.json", "w") as f:
    json.dump(data, f, indent=2)

with open("sample.json", "r") as f:
    loaded = json.load(f)

print(f"Round-trip equality: {data == loaded}")

import os
os.remove("sample.json")
```

### collections — Container Data Types

```python
from collections import Counter, defaultdict, OrderedDict, deque, namedtuple

text = "mississippi"
counter = Counter(text)
print(f"Letter counts: {dict(counter)}")
print(f"Most common (3): {counter.most_common(3)}")

dd = defaultdict(list)
dd["fruits"].append("apple")
dd["fruits"].append("banana")
dd["veggies"].append("carrot")
print(f"Default dict: {dict(dd)}")

Point = namedtuple("Point", ["x", "y"])
p1 = Point(10, 20)
p2 = Point(3, 4)
print(f"Point p1: x={p1.x}, y={p1.y}")
print(f"Distance from origin: {((p1.x)**2 + (p1.y)**2)**0.5:.1f}")

dq = deque(maxlen=5)
for i in range(10):
    dq.append(i)
print(f"Deque (maxlen=5): {list(dq)}")

dq.appendleft(99)
print(f"After appendleft: {list(dq)}")

od = OrderedDict()
od["first"] = 1
od["second"] = 2
od["third"] = 3
print(f"Ordered dict keys: {list(od.keys())}")
```

### itertools — Iterator Tools

```python
import itertools

print("Count (first 5):", list(itertools.islice(itertools.count(10, 2), 5)))
print("Cycle (first 6):", list(itertools.islice(itertools.cycle("AB"), 6)))
print("Repeat (5 times):", list(itertools.repeat("x", 5)))

print("Chain:", list(itertools.chain([1, 2], [3, 4], [5])))
print("Compress:", list(itertools.compress("ABCDEF", [1, 0, 1, 0, 1, 0])))

print("Permutations of 'AB':", list(itertools.permutations("AB", 2)))
print("Combinations of 'ABC' (2):", list(itertools.combinations("ABC", 2)))
print("Product:", list(itertools.product("AB", repeat=2)))

data = [1, 2, 3, 4, 5]
print("Accumulate:", list(itertools.accumulate(data)))
print("Accumulate (mul):", list(itertools.accumulate(data, lambda a, b: a * b)))

grouped = itertools.groupby("AAABBBCCAAA", lambda c: c)
print("Groupby:", {k: len(list(g)) for k, g in grouped})
```

### pathlib — Object-Oriented File System Paths

```python
from pathlib import Path

p = Path(".")
print(f"Current dir absolute: {p.absolute()}")
print(f"Current dir name: {p.name}")

new_dir = Path("test_dir")
new_dir.mkdir(exist_ok=True)
print(f"Directory exists: {new_dir.exists()}")
print(f"Is directory: {new_dir.is_dir()}")

file_path = new_dir / "hello.txt"
file_path.write_text("Hello, World!")
print(f"File content: {file_path.read_text()}")
print(f"File size: {file_path.stat().st_size} bytes")

renamed = new_dir / "greeting.txt"
file_path.rename(renamed)
print(f"Renamed exists: {renamed.exists()}")

for item in Path(".").iterdir():
    print(f"  {'[DIR]' if item.is_dir() else '[FILE]'} {item.name}")

renamed.unlink()
new_dir.rmdir()
```

### subprocess — Spawning Subprocesses

```python
import subprocess
import sys

result = subprocess.run(
    [sys.executable, "-c", "print('Hello from subprocess')"],
    capture_output=True,
    text=True
)
print(f"stdout: {result.stdout.strip()}")
print(f"stderr: {result.stderr}")
print(f"Return code: {result.returncode}")

result_check = subprocess.run(
    [sys.executable, "-c", "print('OK')"],
    capture_output=True,
    text=True,
    check=True
)
print(f"Checked call: {result_check.stdout.strip()}")

with subprocess.Popen(
    [sys.executable, "-c", """
import sys
for i in range(5):
    print(f"line {i}")
"""],
    stdout=subprocess.PIPE,
    text=True
) as proc:
    for line in proc.stdout:
        print(f"Pipe: {line.strip()}")

print("Subprocess module complete")
```

### argparse — Command-Line Argument Parsing

```python
import argparse

parser = argparse.ArgumentParser(description="Sample argument parser")
parser.add_argument("input", help="Input file path")
parser.add_argument("-o", "--output", help="Output file path")
parser.add_argument("-v", "--verbose", action="store_true", help="Verbose mode")
parser.add_argument("--count", type=int, default=1, help="Number of times")
parser.add_argument("--mode", choices=["read", "write", "append"], default="read")

args = parser.parse_args(["--verbose", "--count", "3", "in.txt"])
print(f"Input: {args.input}")
print(f"Output: {args.output}")
print(f"Verbose: {args.verbose}")
print(f"Count: {args.count}")
print(f"Mode: {args.mode}")
```

## Beginner Examples

```python
# File copy utility using standard library modules
import shutil
import os
from pathlib import Path

def backup_file(source_path, backup_dir="backups"):
    source = Path(source_path)
    if not source.exists():
        print(f"Error: {source_path} does not exist")
        return False

    backup_path = Path(backup_dir)
    backup_path.mkdir(exist_ok=True)

    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    dest_name = f"{source.stem}_{timestamp}{source.suffix}"
    dest = backup_path / dest_name

    shutil.copy2(source, dest)
    print(f"Backed up {source.name} -> {dest}")
    return True

from datetime import datetime
backup_file(__file__)

# Count lines of code in a directory
def count_lines(directory=".", extensions=(".py",)):
    total = 0
    files = 0
    for path in Path(directory).rglob("*"):
        if path.suffix in extensions and path.is_file():
            lines = len(path.read_text().splitlines())
            total += lines
            files += 1
    print(f"Found {files} files with {total} total lines")
    return total

count_lines()
```

## Intermediate Examples

```python
# JSON configuration manager with schema validation
import json
import os
from pathlib import Path
import re

class ConfigManager:
    def __init__(self, config_path="config.json"):
        self.config_path = Path(config_path)
        self.data = {}
        self.load()

    def load(self):
        if self.config_path.exists():
            with open(self.config_path) as f:
                self.data = json.load(f)
        else:
            self.data = {"version": "1.0", "settings": {}}
            self.save()

    def save(self):
        self.config_path.parent.mkdir(parents=True, exist_ok=True)
        with open(self.config_path, "w") as f:
            json.dump(self.data, f, indent=2)

    def get(self, key, default=None):
        keys = key.split(".")
        value = self.data
        for k in keys:
            if isinstance(value, dict):
                value = value.get(k)
            else:
                return default
        return value if value is not None else default

    def set(self, key, value):
        keys = key.split(".")
        target = self.data
        for k in keys[:-1]:
            if k not in target:
                target[k] = {}
            target = target[k]
        target[keys[-1]] = value
        self.save()

    def validate_email_config(self):
        email = self.get("settings.email")
        if email and re.match(r"[^@]+@[^@]+\.[^@]+", email):
            return True
        print("Invalid email configuration")
        return False

cfg = ConfigManager("test_config.json")
cfg.set("settings.email", "user@example.com")
cfg.set("settings.retries", 3)
cfg.set("database.host", "localhost")
cfg.set("database.port", 5432)
print(f"Email: {cfg.get('settings.email')}")
print(f"Host: {cfg.get('database.host')}")
cfg.validate_email_config()
os.remove("test_config.json")

# Log file analyzer using collections and re
def analyze_log_file(log_content):
    from collections import Counter, defaultdict
    import re

    ip_pattern = r"\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}"
    ips = re.findall(ip_pattern, log_content)
    ip_counts = Counter(ips)
    print(f"Unique IPs: {len(ip_counts)}")
    print(f"Top IPs: {ip_counts.most_common(3)}")

    error_pattern = r"(ERROR|WARNING|INFO|DEBUG)"
    levels = re.findall(error_pattern, log_content)
    level_counts = Counter(levels)
    print(f"Log levels: {dict(level_counts)}")

    return {"ips": ip_counts, "levels": level_counts}

sample_log = """
2024-01-15 10:30:00 INFO 192.168.1.1 - Server started
2024-01-15 10:30:05 ERROR 192.168.1.2 - Connection timeout
2024-01-15 10:30:10 WARNING 192.168.1.1 - High memory usage
2024-01-15 10:30:15 ERROR 192.168.1.3 - Disk full
2024-01-15 10:30:20 INFO 192.168.1.1 - Request completed
"""
analyze_log_file(sample_log)
```

## Advanced Examples

```python
# Concurrent file processor using multiprocessing, pathlib, and argparse
import multiprocessing as mp
from pathlib import Path
import argparse
import json
import hashlib
import time

def process_file(file_path):
    path = Path(file_path)
    if not path.is_file():
        return None

    content = path.read_bytes()
    result = {
        "path": str(path),
        "size": len(content),
        "md5": hashlib.md5(content).hexdigest(),
        "lines": len(content.splitlines()) if content else 0,
        "extension": path.suffix
    }
    return result

def process_batch(file_paths):
    results = []
    for fp in file_paths:
        result = process_file(fp)
        if result:
            results.append(result)
    return results

def scan_and_process(directory=".", recursive=True, output=None, workers=4):
    base = Path(directory)
    if not base.exists():
        print(f"Directory {directory} does not exist")
        return

    pattern = "**/*" if recursive else "*"
    files = [str(p) for p in base.glob(pattern) if p.is_file()]
    print(f"Found {len(files)} files to process")

    if not files:
        return

    chunk_size = max(1, len(files) // workers)
    chunks = [files[i:i + chunk_size] for i in range(0, len(files), chunk_size)]

    start = time.time()
    with mp.Pool(workers) as pool:
        chunk_results = pool.map(process_batch, chunks)

    all_results = []
    for cr in chunk_results:
        all_results.extend(cr)

    elapsed = time.time() - start
    total_size = sum(r["size"] for r in all_results)
    total_lines = sum(r["lines"] for r in all_results)

    summary = {
        "files_processed": len(all_results),
        "total_size_bytes": total_size,
        "total_lines": total_lines,
        "time_seconds": round(elapsed, 2),
        "files_per_second": round(len(all_results) / elapsed, 1) if elapsed > 0 else 0
    }

    print(f"Processed {summary['files_processed']} files in {summary['time_seconds']}s")
    print(f"Total size: {total_size}, Total lines: {total_lines}")

    if output:
        with open(output, "w") as f:
            json.dump({"summary": summary, "files": all_results}, f, indent=2)
        print(f"Results saved to {output}")

    return summary

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Advanced concurrent file processor")
    parser.add_argument("directory", nargs="?", default=".", help="Directory to scan")
    parser.add_argument("-o", "--output", help="Output JSON file")
    parser.add_argument("-w", "--workers", type=int, default=4, help="Number of workers")
    parser.add_argument("--no-recursive", action="store_false", dest="recursive")
    args = parser.parse_args()

    summary = scan_and_process(
        directory=args.directory,
        recursive=args.recursive,
        output=args.output,
        workers=args.workers
    )

    if summary:
        print(json.dumps(summary, indent=2))

# Custom itertools-like combinatorics library
class Combinatorics:
    @staticmethod
    def permutations(items, r=None):
        n = len(items)
        r = r or n
        if r > n:
            return
        indices = list(range(n))
        cycles = list(range(n, n - r, -1))
        yield tuple(items[i] for i in indices[:r])
        while True:
            for i in reversed(range(r)):
                cycles[i] -= 1
                if cycles[i] == 0:
                    indices[i:] = indices[i + 1:] + indices[i:i + 1]
                    cycles[i] = n - i
                else:
                    j = cycles[i]
                    indices[i], indices[-j] = indices[-j], indices[i]
                    yield tuple(items[i] for i in indices[:r])
                    break
            else:
                return

    @staticmethod
    def combinations(items, r):
        n = len(items)
        if r > n:
            return
        indices = list(range(r))
        yield tuple(items[i] for i in indices)
        while True:
            for i in reversed(range(r)):
                if indices[i] != i + n - r:
                    break
            else:
                return
            indices[i] += 1
            for j in range(i + 1, r):
                indices[j] = indices[j - 1] + 1
            yield tuple(items[i] for i in indices)

c = Combinatorics()
print("Permutations of ABC:", list(c.permutations("ABC", 2)))
print("Combinations of ABCD (3):", list(c.combinations("ABCD", 3)))
```

## Real-World Use Cases

- **os/sys**: Environment configuration, path manipulation, cross-platform utilities
- **re**: Log parsing, input validation (emails, phone numbers, URLs), code analysis
- **math/datetime/random**: Scientific computing, timestamp handling, Monte Carlo simulations
- **json**: REST API responses, configuration files, data interchange format
- **collections/itertools**: Efficient data processing, memory-efficient iteration, frequency analysis
- **pathlib**: Cross-platform file system operations, replacing os.path
- **subprocess/argparse**: CLI tools, build scripts, system administration
- **shutil**: File backups, directory archival (make_archive), disk usage statistics

## Common Mistakes

- Using `os.path` when `pathlib.Path` is more readable and cross-platform
- Forgetting that `random` is not suitable for cryptographic purposes (use `secrets` instead)
- Neglecting to handle `json.JSONDecodeError` when parsing untrusted JSON
- Modifying a `defaultdict` or `Counter` while iterating over it
- Using `subprocess.run` with `shell=True` without sanitizing input (security risk)
- Overlooking that `itertools` functions return iterators, not lists (lazy evaluation)
- Assuming `datetime.now()` returns timezone-aware objects (it returns naive objects by default)
- Using mutable default arguments in combination with `defaultdict`

## Best Practices

- Prefer `import module` over `from module import *` to avoid namespace pollution
- Use `pathlib.Path` for all file path operations in new code
- Leverage `itertools` and `collections` for memory-efficient data processing
- Always specify `encoding="utf-8"` explicitly when opening text files
- Use `argparse` over `sys.argv` for robust CLI argument parsing
- Prefer `subprocess.run` with `capture_output=True` over `os.system`
- Use `json.dumps(..., ensure_ascii=False)` when handling non-ASCII text
- Use `datetime.timezone.utc` instead of the deprecated `pytz` library in Python 3.11+

## Interview Questions

1. **Q**: What is the difference between `os.path.join` and `pathlib.Path /` operator?
   **A**: Both join path components, but pathlib is object-oriented, more readable, and cross-platform. The `/` operator creates a `Path` object while `os.path.join` returns a string.

2. **Q**: How does `Counter.most_common()` work internally?
   **A**: It uses `heapq.nlargest` to find the n most frequent elements in O(n log k) time where k is the number of requested items.

3. **Q**: Explain the difference between `re.match` and `re.search`.
   **A**: `re.match` checks for a match only at the beginning of the string, while `re.search` checks anywhere in the string.

4. **Q**: How do you make `datetime.now()` timezone-aware?
   **A**: Use `datetime.now(timezone.utc)` or `datetime.now().astimezone()` to get a timezone-aware datetime.

5. **Q**: When would you use `defaultdict` over a regular `dict`?
   **A**: When you want to avoid KeyError checks for missing keys, especially when building collections (lists of grouped items, nested dictionaries).

## Coding Challenges

1. **Log Parser**: Write a function that parses an Apache-style log file and returns hourly request counts, top IPs, and error rates using `re`, `collections.Counter`, and `datetime`.

2. **File Synchronizer**: Using `pathlib`, `os`, and `hashlib`, write a tool that compares two directories and prints files that are new, modified, or deleted.

3. **JSON Schema Validator**: Using `json` and `collections.abc`, implement a simple JSON schema validator that checks required fields, types, and nested structures.

4. **Pipeline Combinator**: Using `itertools`, implement a function that takes multiple iterables and yields elements in a round-robin fashion, stopping when all iterables are exhausted.

5. **Configuration Merge**: Using `collections.ChainMap` or nested `defaultdict`, implement a configuration system that merges defaults, user config, and environment variables with proper precedence.

## Summary

Python's standard library is a comprehensive toolkit covering virtually every common programming need. Mastering modules like `os`, `sys`, `re`, `json`, `collections`, `itertools`, `pathlib`, and `subprocess` is essential for writing idiomatic, efficient, and portable Python code. The standard library reduces external dependencies, improves code consistency, and is rigorously tested across all supported platforms.

## Related Topics

Packages and Modules (54.x series), Creating Packages (58.x), Virtual Environments (56.x), pip (55.x), Import System (57.x)
