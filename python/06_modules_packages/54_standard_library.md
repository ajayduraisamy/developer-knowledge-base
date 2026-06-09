# Standard Library - os, sys, re, math, datetime, random, subprocess
## Introduction
Python's standard library is one of its greatest strengths — a vast collection of modules included with every Python installation that provides ready-to-use functionality for file I/O, system interaction, text processing, mathematics, date/time manipulation, random generation, and process management. This file covers seven essential modules that every Python developer should master.

## os module
### What It Is
The `os` module provides a portable interface to operating system functionality. It abstracts OS-specific differences (Windows, Linux, macOS) behind a unified Python API for tasks such as file/directory manipulation, environment variables, process management, and path handling.

### Why It Is Important
Using `os` instead of shell commands or platform-specific code makes Python scripts cross-platform. It is the foundational tool for interacting with the underlying operating system in a structured, programmatic way.

### How It Works Internally
The `os` module is implemented in C (`posixmodule.c` on Unix, `_winapi.c` on Windows). It calls native OS system calls (e.g., `CreateFileW` on Windows, `open/read/write` on POSIX) via Python's C API. The module detects the platform at import time and exposes the appropriate implementation — `os` is actually a loader that imports `posix` or `nt` depending on `sys.platform`.

### Syntax
```python
import os
os.getcwd()          # current working directory
os.listdir(".")      # list directory contents
os.mkdir("newdir")   # create directory
os.remove("file")    # delete file
os.rename("old", "new")
os.environ           # dict of environment variables
os.getenv("HOME")    # get env var or None
```

### Beginner Examples
```python
import os

cwd = os.getcwd()
print(f"Current directory: {cwd}")

items = os.listdir(".")
for item in items:
    print(item)

os.makedirs("data/logs/2025", exist_ok=True)
print("Directories created")
```

### Intermediate Examples
```python
import os

def walk_and_print(root_dir):
    for dirpath, dirnames, filenames in os.walk(root_dir):
        level = dirpath.replace(root_dir, "").count(os.sep)
        indent = " " * 2 * level
        print(f"{indent}{os.path.basename(dirpath)}/")
        sub_indent = " " * 2 * (level + 1)
        for f in filenames:
            full = os.path.join(dirpath, f)
            size = os.path.getsize(full)
            print(f"{sub_indent}{f} ({size} bytes)")

walk_and_print(".")

# Environment variables
db_url = os.getenv("DATABASE_URL", "sqlite:///default.db")
debug = os.getenv("DEBUG", "0") == "1"
os.environ["MY_APP_MODE"] = "production"
```

### Advanced Examples
```python
import os
import tempfile

# Low-level file descriptor operations
fd = os.open("test.bin", os.O_RDWR | os.O_CREAT, 0o644)
os.write(fd, b"hello world")
os.lseek(fd, 0, os.SEEK_SET)
data = os.read(fd, 5)
os.close(fd)

# Fork (Unix only)
pid = os.fork()
if pid == 0:
    print(f"Child process: {os.getpid()}")
    os._exit(0)
else:
    print(f"Parent process, child PID: {pid}")
    os.waitpid(pid, 0)

# Temporary file with automatic cleanup
with tempfile.TemporaryDirectory() as tmpdir:
    path = os.path.join(tmpdir, "tmp.txt")
    os.write(os.open(path, os.O_CREAT | os.O_WRONLY), b"data")
    print(os.listdir(tmpdir))

# Symbolic links
try:
    os.symlink("target.txt", "link.txt")
    print(f"Symlink target: {os.readlink('link.txt')}")
except OSError:
    print("Symlinks not supported on this platform")
```

### Real-World Use Cases
- **Build scripts** that must work on both Windows CI and Linux production
- **Log rotation** — renaming, compressing, deleting old log files by age
- **Configuration management** — reading environment variables for secrets
- **Directory watchers** — polling `os.listdir` or `os.scandir` for changes
- **Cross-platform installers** — creating directory structures and symlinks

### Common Mistakes
- Using `os.system("rm -rf ...")` instead of `os.remove/os.rmdir` — not cross-platform
- Forgetting `exist_ok=True` on `os.makedirs`, causing race conditions
- Assuming `os.listdir` returns full paths — it returns basenames
- Using `os.getcwd()` result after changing directory in another thread

### Best Practices
- Prefer `os.path.join` over string concatenation for paths
- Use `os.path.exists`, `os.path.isfile`, `os.path.isdir` for type checks
- For directory iteration, `os.scandir` is faster than `os.listdir` + `os.path.isdir`
- Use `pathlib.Path` for modern, object-oriented path handling (Python 3.4+)
- Protect file operations with `try/except OSError`

### Performance Considerations
- `os.scandir` reduces syscalls by retrieving file attributes in one call
- `os.walk` internally uses `scandir` for efficient traversal
- Repeated `os.stat` calls are expensive — cache results when possible
- `os.listdir` on very large directories can be memory-intensive

### Interview Questions
1. What is the difference between `os.mkdir` and `os.makedirs`?
2. How would you recursively calculate total directory size using `os.walk`?
3. Explain how `os.path.join` handles absolute path components.
4. What does `os.fork()` do and when is it used?

### Coding Challenges
- Write a function that finds the 5 largest files in a directory tree
- Implement a `rmtree` replacement that logs all deletions
- Build a cross-platform script that sets up a project skeleton

### Related Topics
- pathlib, shutil, tempfile, subprocess, sys

## sys module
### What It Is
The `sys` module provides access to Python interpreter variables and functions. It exposes command-line arguments, the Python path, standard streams, exit functions, and low-level interpreter configuration.

### Why It Is Important
Many Python programs need to read CLI arguments, modify the module search path, or interact with the interpreter itself. `sys` is the gateway for these capabilities.

### How It Works Internally
`sys` is a built-in module compiled into the Python interpreter (`PySys_Init` in `Python/sysmodule.c`). Its attributes are populated during interpreter initialization — `sys.path` is built from `PYTHONPATH`, site-packages, and the current working directory. `sys.modules` is a dict caching all imported modules.

### Syntax
```python
import sys
sys.argv          # command-line arguments list
sys.path          # module search path (list of strings)
sys.modules       # dict of imported modules
sys.exit(0)       # exit interpreter
sys.version       # Python version string
sys.platform      # platform identifier ('win32', 'linux', 'darwin')
sys.stdin/stdout/stderr
```

### Beginner Examples
```python
import sys

print(f"Python version: {sys.version}")
print(f"Platform: {sys.platform}")
print(f"Arguments: {sys.argv[1:]}")

sys.stdout.write("Hello stdout\n")
sys.stderr.write("Error message\n")

# Exit with status code
if len(sys.argv) < 2:
    print("Usage: script.py <name>")
    sys.exit(1)
```

### Intermediate Examples
```python
import sys

# Modify module search path
sys.path.insert(0, "/my/custom/path")

# Import a module by string name
import importlib
module = importlib.import_module(sys.argv[1])
print(f"Imported: {module.__name__}")

# Check loaded modules
mod_names = [k for k in sys.modules if "os" in k]
print(f"OS-related modules: {mod_names}")

# Recursion limit
print(f"Recursion limit: {sys.getrecursionlimit()}")
sys.setrecursionlimit(5000)

# Size of objects
data = [i for i in range(1000)]
print(f"List size: {sys.getsizeof(data)} bytes")
print(f"Item size: {sys.getsizeof(data[0])} bytes")
```

### Advanced Examples
```python
import sys
import traceback

# Custom excepthook
def global_exception_handler(exc_type, exc_value, exc_tb):
    with open("errors.log", "a") as f:
        traceback.print_exception(exc_type, exc_value, exc_tb, file=f)
    print(f"Unhandled exception: {exc_type.__name__}: {exc_value}", file=sys.stderr)

sys.excepthook = global_exception_handler

# Audit hooks (Python 3.8+)
def audit_hook(event, args):
    if event in ("import", "open"):
        print(f"Audit: {event} - {args}")

sys.addaudithook(audit_hook)

# Set tracing for debugger-like behavior
def trace_calls(frame, event, arg):
    if event == "call":
        print(f"Calling: {frame.f_code.co_name}")
    return trace_calls

sys.settrace(trace_calls)

# Control stdin/stdout encoding
sys.stdin.reconfigure(encoding="utf-8")
sys.stdout.reconfigure(encoding="utf-8")
```

### Real-World Use Cases
- **CLI tools** — parsing `sys.argv` for arguments
- **Plugin systems** — dynamically importing modules by path
- **Debug tools** — using `sys.settrace` for code coverage
- **Environment checks** — verifying Python version early in startup
- **Security** — audit hooks to detect malicious imports

### Common Mistakes
- Modifying `sys.path` permanently instead of using virtual environments
- Using `sys.argv[0]` as the script directory without `os.path.dirname`
- Forgetting that `sys.exit` raises `SystemExit` (caught by outer `except`)
- Assuming `sys.platform` returns `"linux"` or `"windows"` consistently

### Best Practices
- Use `argparse` instead of raw `sys.argv` for complex CLI
- Prefer `sys.exit(1)` over `os._exit(1)` for clean shutdown
- Use `sys.path` modifications sparingly — prefer `PYTHONPATH` env var
- Check `sys.version_info >= (3, 10)` for version gating

### Performance Considerations
- `sys.getsizeof` only returns shallow size (not nested objects)
- `sys.modules` lookups are O(1) — used internally for fast imports
- Setting `sys.settrace` significantly slows execution

### Interview Questions
1. What is the difference between `sys.exit` and `os._exit`?
2. How does `sys.path` get populated?
3. What is `sys.modules` and how can it be used?
4. Explain the purpose of `sys.version_info`.

### Coding Challenges
- Write a script that reports import times for all loaded modules
- Build a minimal debugger using `sys.settrace` that prints line numbers
- Create a module that refuses to import on Python versions < 3.10

### Related Topics
- os, argparse, importlib, traceback, platform

## re module
### What It Is
The `re` module provides regular expression (regex) matching operations. It allows pattern-based text search, extraction, replacement, and splitting using a regex syntax largely compatible with Perl.

### Why It Is Important
String methods (`str.find`, `str.replace`, etc.) handle only literal patterns. `re` enables complex pattern matching — validating email formats, extracting structured data from logs, syntax highlighting, and text normalization.

### How It Works Internally
Patterns are compiled into an internal bytecode representation (a mini-instruction set) that is executed by the `_sre` engine (C module). The engine uses backtracking with a stack — patterns with nested quantifiers can cause catastrophic backtracking (exponential time). Python 3.11+ includes an optimised regex engine with better caching.

### Syntax
```python
import re
re.search(pattern, string)    # first match anywhere
re.match(pattern, string)     # match at start only
re.fullmatch(pattern, string) # match entire string
re.findall(pattern, string)   # all non-overlapping matches
re.finditer(pattern, string)  # iterator of match objects
re.sub(pattern, repl, string) # replace
re.split(pattern, string)     # split by pattern
re.compile(pattern)           # compile for reuse
```

### Beginner Examples
```python
import re

text = "Contact: alice@example.com, bob@test.org"

# Search for first email
match = re.search(r"[\w.]+@[\w.]+", text)
if match:
    print(f"Found: {match.group()} at pos {match.start()}")

# Find all emails
emails = re.findall(r"[\w.]+@[\w.]+", text)
print(f"All emails: {emails}")

# Replace
censored = re.sub(r"[\w.]+@[\w.]+", "[REDACTED]", text)
print(censored)

# Split
parts = re.split(r"[,;]\s*", "apple, banana; cherry, date")
print(parts)
```

### Intermediate Examples
```python
import re

# Named groups
log = "2025-01-15 ERROR: Disk 98% full on /dev/sda1"
pattern = r"(?P<date>\d{4}-\d{2}-\d{2})\s+(?P<level>\w+):\s+(?P<message>.+)"
match = re.search(pattern, log)
if match:
    print(match.groupdict())
    # {'date': '2025-01-15', 'level': 'ERROR', 'message': 'Disk 98% full on /dev/sda1'}

# Compile for performance
email_re = re.compile(r"^[\w.+-]+@[\w-]+\.[\w.]+$")
valid_emails = [e for e in ["a@b.com", "bad", "c@d.org"] if email_re.match(e)]
print(valid_emails)

# Non-greedy vs greedy
text = "<b>bold</b> and <i>italic</i>"
greedy = re.findall(r"<.*>", text)    # ['<b>bold</b> and <i>italic</i>']
lazy = re.findall(r"<.*?>", text)      # ['<b>', '</b>', '<i>', '</i>']
print(f"Greedy: {greedy}, Lazy: {lazy}")

# Lookahead/lookbehind
prices = "Item A: $10, Item B: $20, $5 discount"
dollars = re.findall(r"(?<=\$)\d+", prices)
print(f"Prices: {dollars}")
```

### Advanced Examples
```python
import re

# Verbose pattern with comments
date_pattern = re.compile(r"""
    (?P<year>\d{4})     # four-digit year
    [-/]                # separator
    (?P<month>\d{2})    # two-digit month
    [-/]                # separator
    (?P<day>\d{2})      # two-digit day
""", re.VERBOSE | re.IGNORECASE)

# Callback in substitution
def uppercase_hex(match):
    value = int(match.group(1), 16)
    return f"0x{value:X}"

text = "colors: #ff0000, #00ff00"
result = re.sub(r"#([0-9a-f]{6})", uppercase_hex, text)
print(result)  # colors: #0xFF0000, #0x00FF00

# Scanner/tokenizer
master_re = re.compile("|".join([
    r"(?P<NUMBER>\d+)",
    r"(?P<PLUS>\+)",
    r"(?P<MINUS>\-)",
    r"(?P<MUL>\*)",
    r"(?P<WS>\s+)",
]))

def tokenize(expr):
    pos = 0
    while pos < len(expr):
        match = master_re.match(expr, pos)
        if not match:
            raise SyntaxError(f"Unexpected char at {pos}")
        tok_type = match.lastgroup
        if tok_type != "WS":
            yield (tok_type, match.group())
        pos = match.end()

print(list(tokenize("12 + 34 * 5")))
```

### Real-World Use Cases
- **Log parsing** — extracting timestamps, error codes, IP addresses
- **Data validation** — email, phone, URL format checking
- **Code linting** — finding TODO/FIXME comments, checking naming conventions
- **Text normalization** — collapsing whitespace, normalizing quotes
- **Web scraping** — extracting data from HTML (though BeautifulSoup is preferred)

### Common Mistakes
- Forgetting to escape backslashes — use raw strings `r"\d"` not `"\\d"`
- Overusing regex when string methods suffice — `str.startswith` is faster
- Catastrophic backtracking with nested `(.*)*` patterns
- Using `re.match` when `re.search` is needed (match anchors to start)
- Not compiling patterns used in loops

### Best Practices
- Always use raw strings `r"..."` for patterns
- Compile patterns with `re.compile` when reused
- Use named groups `(?P<name>...)` for clarity
- Add `re.VERBOSE` for complex patterns with comments
- Set `re.DOTALL` if `.` should match newlines
- Use a timeout or limit input size to prevent ReDoS attacks

### Performance Considerations
- `re.compile` caches patterns (default cache 512 entries)
- Character classes `[a-z]` are faster than alternation `(a|b|c|...)`
- Atomic groups `(?>...)` prevent backtracking
- `re.search` exits on first match; `re.findall` must scan entire input
- Use `re.DEBUG` flag to inspect compiled bytecode

### Interview Questions
1. What is the difference between `re.match` and `re.search`?
2. Explain greedy vs non-greedy quantifiers.
3. What is catastrophic backtracking and how do you prevent it?
4. How do lookahead and lookbehind assertions work?
5. What is a raw string and why is it important in regex?

### Coding Challenges
- Write a regex that validates IPv4 addresses
- Parse Apache/Nginx log lines into structured dicts
- Implement a simple template engine using `re.sub`
- Extract all URLs from a markdown document

### Related Topics
- str methods, textwrap, difflib, tokenize

## math module
### What It Is
The `math` module provides C-standard mathematical functions: trigonometric, logarithmic, exponential, power, special functions, and constants (pi, e, tau, inf, nan). It operates on floats and integers.

### Why It Is Important
Raw Python arithmetic lacks many common mathematical functions. `math` fills this gap with fast, IEEE-754 compliant implementations. It is essential for scientific computing, game physics, data analysis, and financial calculations.

### How It Works Internally
Most functions wrap C math library calls (`sin`, `cos`, `sqrt`, `log`, etc.) from `libm`. Integer functions like `gcd` and `comb` use pure-Python algorithms in CPython. Constants like `math.pi` are computed at compile time using `M_PI`.

### Syntax
```python
import math
math.sqrt(x)      # square root
math.log(x, base) # logarithm
math.sin(x)       # sine (radians)
math.cos(x)       # cosine
math.ceil(x)      # round up
math.floor(x)     # round down
math.factorial(x) # factorial
math.gcd(a, b)    # greatest common divisor
math.pi           # 3.14159...
math.e            # 2.71828...
math.inf          # infinity
math.nan          # not a number
```

### Beginner Examples
```python
import math

# Basic operations
print(f"sqrt(16) = {math.sqrt(16)}")
print(f"pi = {math.pi}")
print(f"e = {math.e}")

# Rounding
print(f"ceil(3.2) = {math.ceil(3.2)}")
print(f"floor(3.8) = {math.floor(3.8)}")

# GCD and LCM
print(f"gcd(48, 18) = {math.gcd(48, 18)}")
print(f"lcm via gcd: {48 * 18 // math.gcd(48, 18)}")

# Trigonometry
angle = math.radians(45)
print(f"sin(45°) = {math.sin(angle):.4f}")
print(f"cos(45°) = {math.cos(angle):.4f}")
```

### Intermediate Examples
```python
import math

# Distance between two points
def distance(x1, y1, x2, y2):
    return math.hypot(x2 - x1, y2 - y1)

print(f"Distance: {distance(0, 0, 3, 4)}")  # 5.0

# Factorial and combinations
print(f"5! = {math.factorial(5)}")
print(f"C(10, 3) = {math.comb(10, 3)}")      # 120
print(f"P(10, 3) = {math.perm(10, 3)}")      # 720

# Logarithmic scales
for x in [1, 10, 100, 1000]:
    print(f"log10({x}) = {math.log10(x)}")

# Degree/radian conversion
print(f"180° in radians = {math.radians(180)}")
print(f"π rad in degrees = {math.degrees(math.pi)}")

# Special functions
print(f"gamma(5) = {math.gamma(5)}")      # 24.0 (4!)
print(f"erf(1) = {math.erf(1):.4f}")       # error function
```

### Advanced Examples
```python
import math
import random

# Numerical integration (Simpson's rule)
def integrate(f, a, b, n=1000):
    h = (b - a) / n
    s = f(a) + f(b)
    for i in range(1, n, 2):
        s += 4 * f(a + i * h)
    for i in range(2, n - 1, 2):
        s += 2 * f(a + i * h)
    return s * h / 3

area = integrate(math.sin, 0, math.pi)
print(f"∫sin(x)dx from 0 to π ≈ {area:.6f}")  # 2.0

# Is close comparison (floating point safe)
a, b = 0.1 + 0.2, 0.3
print(f"Direct: {a == b}")           # False
print(f"isclose: {math.isclose(a, b)}")  # True

# Floating point classification
for v in [0.0, -0.0, math.inf, -math.inf, math.nan]:
    print(f"{v}: isfinite={math.isfinite(v)}, isnan={math.isnan(v)}, isinf={math.isinf(v)}")

# Modular arithmetic with large numbers
mod = 10**9 + 7
n = 10**6
fact = math.factorial(n) % mod
print(f"{n}! mod {mod} = {fact}")

# Custom sinc function
def sinc(x):
    return 1.0 if x == 0 else math.sin(x) / x

for x in [0, 0.5, 1.0]:
    print(f"sinc({x}) = {sinc(x):.4f}")
```

### Real-World Use Cases
- **Game physics** — velocity, acceleration, collision detection via vectors
- **Financial models** — compound interest, amortization, present value
- **Data normalization** — log transformation, z-score, min-max scaling
- **Geometry** — area/volume calculations, coordinate transformations
- **Statistics** — combinations, permutations, gamma for distributions

### Common Mistakes
- Using `math.sqrt(-1)` instead of `cmath.sqrt` for imaginary results
- Expecting `math.isclose` defaults to work for very small/large numbers
- Confusing `math.ceil(-0.5)` → `0` not `-1`
- Forgetting trig functions expect radians, not degrees

### Best Practices
- Use `math.isclose` for float equality checks with tolerance
- Prefer `math.hypot(x, y)` over `math.sqrt(x*x + y*y)` (avoids overflow)
- Use `math.fsum` for accurate summation of floats (higher precision)
- For complex numbers, use `cmath` (complex math) module
- For arbitrary-precision, use `decimal.Decimal`

### Performance Considerations
- `math` functions are implemented in C and very fast
- `math.sqrt` is significantly faster than `x**0.5`
- `math.hypot` avoids intermediate overflow but is slightly slower
- Precompute constants outside loops

### Interview Questions
1. Why should you use `math.isclose` instead of `==` for floats?
2. What is the difference between `math.floor` and `math.trunc` for negative numbers?
3. How does `math.hypot` improve numerical stability?
4. Explain the gamma function and its relationship to factorial.

### Coding Challenges
- Implement a prime sieve using `math.isqrt` for upper bound
- Write a `Point` class with distance, rotation, and translation using `math`
- Calculate the approximate value of pi using Monte Carlo integration
- Implement logistic regression hypothesis function using `math.exp`

### Related Topics
- cmath, decimal, fractions, random, statistics, numpy

## datetime module
### What It Is
The `datetime` module supplies classes for manipulating dates, times, and time intervals: `date`, `time`, `datetime`, `timedelta`, `timezone`, and `tzinfo`. It supports arithmetic, formatting, parsing, and time zone handling.

### Why It Is Important
Working with dates and times is ubiquitous in software — logging, scheduling, data analysis, user interfaces. `datetime` provides a standard, reliable way to handle these operations without external dependencies.

### How It Works Internally
`datetime` is implemented in C (`_datetimemodule.c`). Dates are stored as proleptic Gregorian ordinal integers (days since 0001-01-01). Times are stored as microseconds since midnight. `timedelta` stores days, seconds, and microseconds separately for exact representation. Timezone handling uses the `tzinfo` abstract base class with concrete `timezone` (fixed offset) in Python 3.2+.

### Syntax
```python
from datetime import date, time, datetime, timedelta, timezone

date.today()                     # current local date
datetime.now()                   # current local datetime
datetime.utcnow()                # UTC datetime (naive)
datetime.now(timezone.utc)       # timezone-aware UTC
dt.strftime("%Y-%m-%d")          # format to string
datetime.strptime("2025-01-15", "%Y-%m-%d")  # parse string
dt - other_dt                    # timedelta arithmetic
```

### Beginner Examples
```python
from datetime import date, datetime, timedelta

# Current date and time
today = date.today()
now = datetime.now()
print(f"Today: {today}")
print(f"Now: {now}")

# Date arithmetic
tomorrow = today + timedelta(days=1)
yesterday = today - timedelta(days=1)
print(f"Tomorrow: {tomorrow}")
print(f"Yesterday: {yesterday}")

# Formatting
formatted = now.strftime("%A, %B %d, %Y at %I:%M %p")
print(f"Formatted: {formatted}")

# Parsing
parsed = datetime.strptime("2025-12-25 10:30", "%Y-%m-%d %H:%M")
print(f"Parsed: {parsed}")

# Date components
print(f"Year: {today.year}, Month: {today.month}, Day: {today.day}")
```

### Intermediate Examples
```python
from datetime import date, datetime, timedelta, timezone

# Date range iteration
def date_range(start, end):
    for n in range(int((end - start).days)):
        yield start + timedelta(n)

start = date(2025, 1, 1)
end = date(2025, 1, 10)
for d in date_range(start, end):
    print(d.strftime("%a %Y-%m-%d"))

# Timezone handling
utc = timezone.utc
eastern = timezone(timedelta(hours=-5))
dt_utc = datetime.now(utc)
dt_est = dt_utc.astimezone(eastern)
print(f"UTC: {dt_utc.isoformat()}")
print(f"EST: {dt_est.isoformat()}")

# Age calculation
def age(birth_date):
    today = date.today()
    return today.year - birth_date.year - (
        (today.month, today.day) < (birth_date.month, birth_date.day)
    )

print(f"Age: {age(date(1990, 6, 15))}")

# ISO week number
iso = today.isocalendar()
print(f"ISO year: {iso[0]}, week: {iso[1]}, weekday: {iso[2]}")

# Quarter calculation
quarter = (today.month - 1) // 3 + 1
print(f"Q{quarter} {today.year}")
```

### Advanced Examples
```python
from datetime import datetime, timedelta, timezone
import calendar

# Last day of month
def last_day_of_month(year, month):
    return calendar.monthrange(year, month)[1]

print(f"Days in Feb 2025: {last_day_of_month(2025, 2)}")

# Business days between two dates
def business_days(start, end):
    days = []
    current = start
    while current < end:
        if current.weekday() < 5:
            days.append(current)
        current += timedelta(days=1)
    return days

start = date(2025, 1, 1)
end = date(2025, 1, 15)
print(f"Business days: {len(business_days(start, end))}")

# Timezone-aware scheduling
def next_alarm(hour=9, minute=0, tz=timezone.utc):
    now = datetime.now(tz)
    alarm = now.replace(hour=hour, minute=minute, second=0, microsecond=0)
    if alarm <= now:
        alarm += timedelta(days=1)
    return alarm

alarm = next_alarm(14, 30, timezone(timedelta(hours=2)))
print(f"Next alarm: {alarm}")

# RFC 3339 / ISO 8601 parsing (Python 3.11+)
# dt = datetime.fromisoformat("2025-01-15T10:30:00+00:00")

# Unix timestamp conversion
ts = 1705314000
dt_from_ts = datetime.fromtimestamp(ts, tz=timezone.utc)
print(f"From timestamp: {dt_from_ts}")

# High-precision timing
start = datetime.now()
for _ in range(1_000_000):
    pass
elapsed = datetime.now() - start
print(f"Elapsed: {elapsed.total_seconds():.6f}s")
```

### Real-World Use Cases
- **Logging** — timestamping events with timezone awareness
- **Scheduling** — computing next run times for cron-like jobs
- **Data pipelines** — partitioning data by date, time-windowed aggregations
- **E-commerce** — calculating delivery windows, order age
- **Finance** — maturity dates, coupon payment schedules, business day conventions

### Common Mistakes
- Performing arithmetic on naive datetimes with inconsistent local times
- Assuming `datetime.utcnow()` is timezone-aware (it returns a naive datetime)
- Using `timedelta(months=1)` — `timedelta` does not support months
- Confusing `datetime.time` with the `time` module
- Forgetting DST transitions when adding days

### Best Practices
- Always use timezone-aware datetimes in production systems
- Store datetimes in UTC, convert to local time only for display
- Use `datetime.now(timezone.utc)` instead of `datetime.utcnow()`
- Prefer `dateutil` library for complex timezone and date parsing
- Use `timedelta` for duration, `dateutil.relativedelta` for calendar months
- In Python 3.11+, use `fromisoformat` for standard parsing

### Performance Considerations
- `datetime` objects are immutable (creates new objects on operation)
- Parsing with `strptime` is relatively slow — cache format strings
- `datetime.now()` is fast but not monotonic — use `time.perf_counter` for benchmarks

### Interview Questions
1. What is the difference between naive and aware datetimes?
2. How do you handle DST transitions in Python?
3. What is the maximum and minimum `datetime` representable?
4. How does `timedelta` store its value internally?
5. Why is storing datetimes in UTC recommended?

### Coding Challenges
- Implement a function that lists all Mondays in a given year
- Build a cron expression parser using `datetime`
- Create a meeting scheduler that finds common free slots across time zones
- Write a function that calculates age in years, months, and days

### Related Topics
- time, calendar, dateutil, pytz, pendulum, timezone

## random module
### What It Is
The `random` module implements pseudo-random number generators for various distributions: uniform, normal, exponential, triangular, and more. It also provides functions for random selection, shuffling, and sampling.

### Why It Is Important
Randomness is needed for simulations, games, testing, cryptography (non-secure), statistical sampling, and randomized algorithms. `random` provides a simple, fast, reproducible interface to these operations.

### How It Works Internally
The default generator is the Mersenne Twister (MT19937) — a deterministic PRNG with a period of 2^19937-1. It maintains a 624-element internal state array. The algorithm is not cryptographically secure — for security use `secrets` or `os.urandom`. Python 3.9+ also provides `random.Random` subclassability and `SystemRandom`.

### Syntax
```python
import random
random.random()          # float in [0.0, 1.0)
random.randint(1, 10)    # int in [1, 10] inclusive
random.uniform(0, 5)     # float in [0.0, 5.0)
random.choice(seq)       # random element
random.choices(seq, k=3) # k elements with replacement
random.sample(seq, k=3)  # k unique elements
random.shuffle(lst)      # in-place shuffle
random.seed(42)          # fixed seed for reproducibility
random.gauss(mu, sigma)  # normal distribution
```

### Beginner Examples
```python
import random

# Basic random numbers
print(f"Random float: {random.random():.4f}")
print(f"Random int 1-10: {random.randint(1, 10)}")
print(f"Random float 0-5: {random.uniform(0, 5):.2f}")

# Selection
fruits = ["apple", "banana", "cherry", "date"]
print(f"Random fruit: {random.choice(fruits)}")
print(f"Two fruits: {random.sample(fruits, 2)}")

# Shuffle
deck = list(range(52))
random.shuffle(deck)
print(f"Shuffled deck (first 5): {deck[:5]}")

# Dice roll
def roll_dice(n=2):
    return sum(random.randint(1, 6) for _ in range(n))

print(f"Rolled: {roll_dice()}")
```

### Intermediate Examples
```python
import random

# Weighted choices
colors = ["red", "green", "blue", "yellow"]
weights = [0.5, 0.3, 0.15, 0.05]
print(f"Weighted choice: {random.choices(colors, weights=weights, k=10)}")

# Normal distribution
heights = [round(random.gauss(170, 10), 1) for _ in range(20)]
print(f"Mean height: {sum(heights) / len(heights):.1f}")

# Reproducible randomness
random.seed(42)
print(f"First: {random.randint(1, 100)}")  # 82
random.seed(42)
print(f"Second: {random.randint(1, 100)}")  # 82 (same)

# Random password generator (non-cryptographic)
def simple_password(length=12):
    chars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789!@#$%"
    return "".join(random.choice(chars) for _ in range(length))

print(f"Password: {simple_password()}")

# Shuffle without mutation
original = [1, 2, 3, 4, 5]
shuffled = original[:]
random.shuffle(shuffled)
print(f"Original: {original}, Shuffled: {shuffled}")
```

### Advanced Examples
```python
import random
import math
import statistics

# Monte Carlo estimation of pi
def estimate_pi(num_points=100_000):
    inside = 0
    for _ in range(num_points):
        x, y = random.random(), random.random()
        if x * x + y * y <= 1.0:
            inside += 1
    return 4 * inside / num_points

print(f"Estimated pi: {estimate_pi(200_000):.6f}")

# Custom distribution via inverse transform
def exponential_random(rate=1.0):
    return -math.log(1.0 - random.random()) / rate

samples = [exponential_random(0.5) for _ in range(1000)]
print(f"Mean (expected 2.0): {statistics.mean(samples):.3f}")

# Random walk simulation
def random_walk(steps=100):
    x, y = 0, 0
    path = [(x, y)]
    for _ in range(steps):
        dx, dy = random.choice([(1, 0), (-1, 0), (0, 1), (0, -1)])
        x += dx
        y += dy
        path.append((x, y))
    return path

path = random_walk(1000)
print(f"Final position: {path[-1]}, distance: {math.hypot(*path[-1]):.2f}")

# Shuffled deck with card class
suits = ["♠", "♥", "♦", "♣"]
ranks = ["2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K", "A"]
deck = [f"{r}{s}" for s in suits for r in ranks]
random.shuffle(deck)
hand = deck[:5]
print(f"Hand: {hand}")

# Reservoir sampling (streaming)
def reservoir_sample(stream, k):
    reservoir = []
    for i, item in enumerate(stream):
        if i < k:
            reservoir.append(item)
        else:
            j = random.randint(0, i)
            if j < k:
                reservoir[j] = item
    return reservoir

sample = reservoir_sample(range(1_000_000), 5)
print(f"Reservoir sample: {sample}")
```

### Real-World Use Cases
- **Games** — dice rolls, card shuffles, loot drops, procedural generation
- **Simulations** — Monte Carlo methods, queuing models, financial risk
- **Testing** — generating random test data, fuzzing
- **A/B testing** — random assignment to control/treatment groups
- **Machine learning** — train/test split, mini-batch sampling, weight initialization

### Common Mistakes
- Using `random` for security-sensitive operations (passwords, tokens, session IDs)
- Forgetting `random.seed()` makes randomness reproducible, not unpredictable
- Assuming `random.shuffle` returns a value (shuffles in-place, returns None)
- Using `random.sample` on a generator (requires a sequence)

### Best Practices
- Use `secrets` module for cryptographic randomness
- Call `random.seed()` once at program start for reproducibility in tests
- Use `random.SystemRandom` for non-blocking OS entropy
- For weighted selections, `random.choices` with `weights` parameter
- Use `random.getrandbits(k)` for fast random integers

### Performance Considerations
- `random.random()` is very fast (C implementation)
- `random.shuffle` is O(n) in-place
- `random.sample` uses partial Fisher-Yates shuffle, O(k) for k samples
- Mersenne Twister is not thread-safe; use separate `Random` instances per thread

### Interview Questions
1. What algorithm does Python's `random` module use?
2. What is the difference between `random.sample` and `random.choices`?
3. How do you make random numbers reproducible?
4. Why should you not use `random` for password generation?
5. Explain the Fisher-Yates shuffle algorithm.

### Coding Challenges
- Implement a deck of cards with shuffle, deal, and hand evaluation
- Write a simulate_dice function that returns the distribution of sums
- Implement reservoir sampling for an infinite stream
- Build a simple procedural terrain generator using Perlin-like noise
- Monte Carlo simulation for the birthday paradox

### Related Topics
- secrets, os.urandom, numpy.random, statistics, itertools

## subprocess module
### What It Is
The `subprocess` module allows spawning new processes, connecting to their input/output/error pipes, and obtaining their return codes. It replaces older modules like `os.system`, `os.spawn*`, and `os.popen*`.

### Why It Is Important
Python programs often need to run external commands, shell scripts, or other executables. `subprocess` provides a secure, flexible, and cross-platform API for process creation and management with full I/O control.

### How It Works Internally
On POSIX, `subprocess` uses `os.fork()` and `os.execve()` to create child processes. On Windows, it uses `CreateProcess`. Pipes are created via `os.pipe()` on POSIX or anonymous pipes on Windows. The `Popen` class manages process lifecycle — it spawns the subprocess in `__init__` and provides `wait`, `poll`, `communicate`, and `kill` methods.

### Syntax
```python
import subprocess
subprocess.run(args, ...)           # run command, wait, return CompletedProcess
subprocess.Popen(args, ...)         # low-level process management
subprocess.check_call(args)         # run, raise CalledProcessError on non-zero
subprocess.check_output(args)       # run, return stdout
subprocess.PIPE                     # send to pipe
subprocess.STDOUT                   # redirect stderr to stdout
```

### Beginner Examples
```python
import subprocess

# Simple command
result = subprocess.run(["echo", "Hello World"], capture_output=True, text=True, shell=True)
print(f"stdout: {result.stdout}")
print(f"returncode: {result.returncode}")

# Check output
output = subprocess.check_output(["echo", "Hello"], text=True, shell=True)
print(f"Output: {output.strip()}")

# Error handling
try:
    subprocess.check_call(["false"], shell=True)
except subprocess.CalledProcessError as e:
    print(f"Command failed with code {e.returncode}")

# Shell=True (use with caution)
subprocess.run("dir", shell=True)
```

### Intermediate Examples
```python
import subprocess

# Capture both stdout and stderr
result = subprocess.run(
    ["python", "-c", "import sys; print('out'); sys.stderr.write('err')"],
    capture_output=True,
    text=True,
)
print(f"stdout: {result.stdout.strip()}")
print(f"stderr: {result.stderr.strip()}")

# Pipe input
proc = subprocess.Popen(
    ["sort"],
    stdin=subprocess.PIPE,
    stdout=subprocess.PIPE,
    text=True,
)
stdout, _ = proc.communicate("banana\napple\ncherry\n")
print(f"Sorted: {stdout.strip()}")

# Timeout
try:
    subprocess.run(["sleep", "10"], timeout=3, shell=True)
except subprocess.TimeoutExpired:
    print("Command timed out")

# Working directory and environment
result = subprocess.run(
    ["pwd"],
    cwd="/tmp",
    capture_output=True,
    text=True,
    shell=True,
)
print(f"Working directory: {result.stdout.strip()}")

# Environment modification
import os
env = os.environ.copy()
env["MY_VAR"] = "hello"
result = subprocess.run(
    ["echo", "%MY_VAR%"],
    env=env,
    capture_output=True,
    text=True,
    shell=True,
)
print(f"Env var: {result.stdout.strip()}")
```

### Advanced Examples
```python
import subprocess
import threading
import queue

# Streaming output in real-time
def stream_output():
    proc = subprocess.Popen(
        ["ping", "localhost", "-n", "5"],
        stdout=subprocess.PIPE,
        stderr=subprocess.STDOUT,
        text=True,
        bufsize=1,
    )
    for line in iter(proc.stdout.readline, ""):
        print(f"[OUT] {line.strip()}")
    proc.stdout.close()
    proc.wait()

# Parallel subprocesses
def run_parallel(commands):
    processes = [subprocess.Popen(cmd, shell=True) for cmd in commands]
    for proc in processes:
        proc.wait()
    return [proc.returncode for proc in processes]

codes = run_parallel(["echo a", "echo b", "echo c"])
print(f"Return codes: {codes}")

# Interactive subprocess
proc = subprocess.Popen(
    ["python", "-i"],
    stdin=subprocess.PIPE,
    stdout=subprocess.PIPE,
    stderr=subprocess.STDOUT,
    text=True,
)
stdout, _ = proc.communicate("print(2 + 2)\nexit()\n")
print(stdout)

# Process group management
import signal
import os

def run_with_timeout(cmd, timeout):
    proc = subprocess.Popen(cmd, shell=True, preexec_fn=os.setsid if os.name != "nt" else None)
    try:
        proc.wait(timeout=timeout)
    except subprocess.TimeoutExpired:
        # Kill process group
        if os.name == "nt":
            proc.kill()
        else:
            os.killpg(os.getpgid(proc.pid), signal.SIGTERM)
        raise
    return proc.returncode
```

### Real-World Use Cases
- **Build automation** — running compilers, linters, test runners
- **Deployment** — SSH commands, Docker operations, package managers
- **Data processing** — piping data through external filters (jq, sed, awk)
- **System administration** — checking disk usage, restarting services
- **FFI alternative** — calling CLI tools when Python libraries don't exist

### Common Mistakes
- Using `shell=True` with user input (command injection risk)
- Not quoting arguments with `shell=True` — prefer list form
- Ignoring the return code (check `result.returncode` or use `check_call`)
- Deadlocking by reading `stdout` before closing `stdin` in `Popen`
- Using `subprocess.run` with `stdout=PIPE` but never reading the pipe

### Best Practices
- **Never use** `shell=True` with untrusted input or string concatenation
- Prefer list argument form (`["ls", "-l"]`) over shell string
- Use `capture_output=True` (Python 3.7+) instead of manually setting `stdout=PIPE`
- Set `text=True` to get strings instead of bytes
- Use `timeout` parameter to prevent hangs
- Handle `CalledProcessError` and `TimeoutExpired` explicitly
- Use `shlex.split()` for safe tokenization of command strings

### Performance Considerations
- Process creation is expensive — avoid spawning processes in tight loops
- Pipe buffering can cause delays — use `bufsize=1` for line-buffered output
- `subprocess.run` blocks until completion — use `Popen` with `asyncio` for concurrent I/O
- On Windows, process creation is slower than on POSIX

### Interview Questions
1. What is the difference between `subprocess.run` and `subprocess.Popen`?
2. Why is `shell=True` dangerous?
3. How do you prevent deadlocks when using `Popen` with pipes?
4. How would you run multiple subprocesses in parallel?
5. What does `capture_output=True` do in `subprocess.run`?

### Coding Challenges
- Write a Python function that runs a command with a timeout and kills the process tree
- Build a simple task runner that executes commands in parallel with output streaming
- Implement a Python REPL that runs code in a subprocess for isolation
- Create a wrapper around `ffmpeg` that converts video formats

### Related Topics
- os.system, os.exec*, shlex, asyncio.subprocess, signal, multiprocessing
