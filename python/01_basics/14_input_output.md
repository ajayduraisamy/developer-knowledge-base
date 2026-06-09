# Input and Output - input(), print(), formatted output

## Introduction

Input and output (I/O) operations enable programs to interact with users and external systems. Python provides `input()` for reading user input from stdin, `print()` for writing to stdout, and multiple string formatting systems (f-strings, `.format()`, `%`-formatting) for producing well-structured output. Handling input validation, encoding, and formatting correctly is essential for building robust, user-friendly CLI applications and data processing pipelines.

## input()

### What It Is

`input()` reads a line from standard input (stdin), returns it as a string (with the trailing newline removed), and optionally displays a prompt string. In Python 3, `input()` always returns a string — unlike Python 2's `input()` (which evaluated the input as Python code).

### Why It Is Important

Every interactive program depends on `input()` for user interaction — command-line tools, prompts, menus, and data-entry forms. Understanding its behavior, type conversion requirements, and error handling is fundamental to building usable software.

### How It Works Internally

`input(prompt)` is implemented as a C function in CPython that:
1. Writes the prompt string to stderr (not stdout — so it appears even if stdout is redirected)
2. Reads a line from buffered stdin using `fgets()` or equivalent
3. Strips the trailing newline character
4. Returns the resulting str object
5. Raises `EOFError` if stdin is closed (Ctrl+D / Ctrl+Z)
6. Raises `KeyboardInterrupt` if Ctrl+C is pressed

The function holds the GIL during the read, so no other Python threads execute during input.

### Syntax

```python
# Basic input
name = input("Enter name: ")

# Input without prompt
line = input()

# Type conversion (always returns str)
age = int(input("Enter age: "))
price = float(input("Enter price: "))
```

### Beginner Examples

```python
# Simple greeting
name = input("What is your name? ")
print(f"Hello, {name}!")

# Numeric input
age = int(input("How old are you? "))
print(f"Next year you'll be {age + 1}")

# Multiple values on one line
x, y = input("Enter two numbers: ").split()
x, y = int(x), int(y)
print(f"Sum: {x + y}")

# Multi-line input
lines = []
print("Enter 3 lines:")
for _ in range(3):
    lines.append(input())
print(f"You entered: {lines}")
```

### Intermediate Examples

```python
# Robust integer input with validation
def get_int(prompt="Enter an integer: ", min_val=None, max_val=None):
    while True:
        try:
            value = input(prompt).strip()
            if not value:
                print("Input cannot be empty.")
                continue
            num = int(value)
            if min_val is not None and num < min_val:
                print(f"Value must be >= {min_val}")
                continue
            if max_val is not None and num > max_val:
                print(f"Value must be <= {max_val}")
                continue
            return num
        except ValueError:
            print("Invalid integer. Please try again.")
        except KeyboardInterrupt:
            print("\nOperation cancelled.")
            return None

age = get_int("Enter age: ", min_val=0, max_val=150)
if age is not None:
    print(f"Age: {age}")

# Reading multiple typed values
def get_floats(prompt, count):
    while True:
        parts = input(prompt).split()
        if len(parts) != count:
            print(f"Please enter exactly {count} values.")
            continue
        try:
            return [float(p) for p in parts]
        except ValueError:
            print("All values must be numbers.")

vals = get_floats("Enter 3 numbers: ", 3)
print(f"Average: {sum(vals) / len(vals):.2f}")
```

### Advanced Examples

```python
# Password input with getpass
import getpass
password = getpass.getpass("Enter password: ")
print(f"Password length: {len(password)} chars")

# Non-blocking input attempt
import select
import sys
import tty
import termios

def get_key_nonblocking(timeout=0.1):
    if not select.select([sys.stdin], [], [], timeout)[0]:
        return None
    return sys.stdin.read(1)

# Custom input with history
class InputWithHistory:
    def __init__(self):
        self.history = []
    
    def get_input(self, prompt="> "):
        value = input(prompt)
        self.history.append(value)
        return value
    
    def get_history(self):
        return list(self.history)

cli = InputWithHistory()
# Commands would be called repeatedly in a REPL loop

# Reading input with pipe detection
import sys
def is_piped():
    return not sys.stdin.isatty()

if is_piped():
    data = sys.stdin.read()
    print(f"Read {len(data)} chars from pipe")
else:
    name = input("Enter name interactively: ")
    print(f"Hello, {name}!")
```

### Real-World Use Cases

- **CLI tools**: Interactive prompts for configuration
- **Scripts**: Reading piped data via stdin
- **Educational software**: Student input for exercises
- **Testing**: Simulating user input with `unittest.mock.patch`
- **Configuration wizards**: Step-by-step setup tools
- **Data entry**: CSV/JSON input collection
- **REPLs**: Input loop for interactive interpreters

### Common Mistakes

```python
# Mistake 1: Not converting input type
age = input("Enter age: ")
# print(age + 5)  # TypeError: str + int
age = int(age)  # Must convert

# Mistake 2: Not handling empty input
name = input("Name: ").strip()
if not name:
    # Handle empty input
    name = "Guest"

# Mistake 3: Assuming input() handles Ctrl+C gracefully
try:
    data = input("Enter data: ")
except KeyboardInterrupt:
    print("\nUser cancelled")
    # Cleanup and exit

# Mistake 4: Not stripping whitespace
city = input("City: ").strip()
# Without .strip(), "  NYC  " would have leading/trailing spaces

# Mistake 5: Using input() for sensitive data (visible on screen)
import getpass
pwd = getpass.getpass("Password: ")  # Hidden input

# Mistake 6: Not handling EOFError
try:
    data = input()
except EOFError:
    print("End of input reached")
```

### Best Practices

- Always convert `input()` return value to the desired type
- Use `try/except ValueError` for numeric input validation
- Use `.strip()` to remove extraneous whitespace
- Use `getpass.getpass()` for sensitive data
- Handle `KeyboardInterrupt` and `EOFError` gracefully
- Use `sys.stdin.isatty()` to detect pipe vs interactive mode
- Provide clear, informative prompts
- Validate and re-prompt on invalid input
- Use infinite loops with `break` for menu systems

### Performance Considerations

`input()` is blocking — it suspends execution until a newline is received. It's IO-bound, not CPU-bound. Reading large piped input with `input()` in a loop is slower than `sys.stdin.read()` or `sys.stdin.readline()` because `input()` processes escape sequences and strips newlines. For bulk piped input, use `sys.stdin.buffer.read()` and decode manually.

### Interview Questions

1. What is the return type of `input()`?
2. How does `input()` differ between Python 2 and Python 3?
3. How do you read sensitive data without echoing to screen?
4. What exceptions can `input()` raise?
5. How do you detect if input is from a pipe vs interactive terminal?
6. How do you read multi-line input efficiently?
7. What happens with `input()` on EOF?
8. How do you mock `input()` for unit testing?

### Coding Challenges

```python
# Challenge 1: Menu with input validation
def menu(options):
    while True:
        print("\nOptions:")
        for key, label in options.items():
            print(f"  {key}. {label}")
        choice = input("Select: ").strip()
        if choice in options:
            return choice
        print("Invalid choice. Try again.")

choice = menu({"1": "Start", "2": "Settings", "3": "Exit"})
print(f"Chose: {choice}")

# Challenge 2: Multi-input CSV parser
def read_csv_input():
    print("Enter CSV rows (blank line to finish):")
    rows = []
    while True:
        line = input().strip()
        if not line:
            break
        rows.append([cell.strip() for cell in line.split(",")])
    return rows

# Challenge 3: Type-aware input
def typed_input(prompt, type_fn=str):
    while True:
        try:
            return type_fn(input(prompt).strip())
        except (ValueError, TypeError):
            print(f"Please enter a valid {type_fn.__name__}")

val = typed_input("Enter a float: ", float)
print(f"{val} * 2 = {val * 2}")

# Challenge 4: Batch input with timeout
import signal

class TimeoutError(Exception):
    pass

def timeout_handler(signum, frame):
    raise TimeoutError()

def input_with_timeout(prompt, timeout=5):
    signal.signal(signal.SIGALRM, timeout_handler)
    signal.alarm(timeout)
    try:
        return input(prompt)
    except TimeoutError:
        print("\nInput timed out")
        return None
    finally:
        signal.alarm(0)
```

### Related Topics

- Type conversion (int(), float(), str())
- Exception handling (try/except)
- sys.stdin, sys.stdout, sys.stderr
- getpass module
- String formatting (f-strings)
- File I/O (open, read, write)

## print()

### What It Is

`print()` writes one or more objects to a text stream (default stdout), separated by a configurable separator and followed by a configurable end string. It can also write to any file-like object via the `file` parameter.

### Why It Is Important

`print()` is the primary mechanism for output in Python — debugging, logging, user-facing messages, and formatted reports. Its flexibility (sep, end, file, flush) adapts to diverse output needs.

### How It Works Internally

`print()` is implemented as a Python built-in function (C implementation in `bltinmodule.c`). It:
1. Converts each argument to string via `str(obj)` (or calls `repr()` if `__repr__` is requested)
2. Writes each argument separated by `sep`
3. Appends `end` (default `\n`)
4. Writes the combined string to `file` (default `sys.stdout`)
5. Calls `file.flush()` if `flush=True`

The function holds the GIL during write operations.

### Syntax

```python
# Basic
print("Hello, World!")

# Multiple arguments
print("a", "b", "c")

# Custom separator
print("a", "b", "c", sep=", ")

# Custom end
print("Loading", end="")
print("...done")

# Write to file
with open("out.txt", "w") as f:
    print("Data", file=f)

# Force flush
print("Progress: 50%", flush=True)
```

### Beginner Examples

```python
# Basic string output
print("Hello, World!")

# Multiple arguments
print("The answer is", 42)

# Custom separator
print("apple", "banana", "cherry", sep=" | ")
# apple | banana | cherry

# Custom end (suppress newline)
print("Countdown:", end=" ")
for i in range(5, 0, -1):
    print(i, end="...")
print("Go!")

# Printing special characters
print("Line1\nLine2")     # Newline
print("Column1\tColumn2")  # Tab
print("Path: C:\\Users")   # Backslash

# Printing to file
with open("log.txt", "w") as f:
    print("Error: connection failed", file=f)
```

### Intermediate Examples

```python
# Progress indicator
import time
def progress(current, total):
    percent = int(current / total * 100)
    bar = "#" * (percent // 5) + "-" * (20 - percent // 5)
    print(f"\r[{bar}] {percent}%", end="", flush=True)

for i in range(101):
    progress(i, 100)
    time.sleep(0.02)
print()

# Pretty table printing
def print_table(header, rows):
    widths = [len(h) for h in header]
    for row in rows:
        for i, cell in enumerate(row):
            widths[i] = max(widths[i], len(str(cell)))
    
    sep_line = "|" + "|".join("-" * (w + 2) for w in widths) + "|"
    header_line = "| " + " | ".join(h.ljust(w) for h, w in zip(header, widths)) + " |"
    
    print(sep_line)
    print(header_line)
    print(sep_line.replace("-", "="))
    for row in rows:
        print("| " + " | ".join(str(c).ljust(w) for c, w in zip(row, widths)) + " |")
    print(sep_line)

print_table(["Name", "Age", "City"],
    [["Alice", 30, "NYC"], ["Bob", 25, "LA"], ["Charlie", 35, "Chicago"]])

# Printing with different output streams
import sys
print("Standard output", file=sys.stdout)
print("Error message", file=sys.stderr)

# Logging with timestamp
from datetime import datetime
def log(message, level="INFO"):
    timestamp = datetime.now().strftime("%H:%M:%S")
    print(f"[{timestamp}] [{level}] {message}")

log("Application started")
log("Processing complete", "INFO")
log("Connection failed", "ERROR")
```

### Advanced Examples

```python
# Thread-safe print wrapper
import threading
class ThreadSafePrint:
    _lock = threading.Lock()
    
    @classmethod
    def print(cls, *args, **kwargs):
        with cls._lock:
            print(*args, **kwargs)

# Custom print class
class Logger:
    def __init__(self, prefix="", output=sys.stdout):
        self.prefix = prefix
        self.output = output
    
    def log(self, *args, **kwargs):
        print(f"[{self.prefix}]", *args, file=self.output, **kwargs)
    
    def error(self, *args, **kwargs):
        print(f"[ERROR]", *args, file=sys.stderr, **kwargs)

logger = Logger("APP")
logger.log("User logged in")
logger.error("Database timeout")

# Conditional flushing for real-time output
def stream_output(generator):
    for item in generator:
        print(item, flush=True)

def data_stream():
    import time
    for i in range(10):
        time.sleep(0.1)
        yield f"Data point {i}"

# stream_output(data_stream())  # Each line appears immediately

# Pretty JSON print
import json
data = {"users": [
    {"name": "Alice", "age": 30, "skills": ["python", "sql"]},
    {"name": "Bob", "age": 25, "skills": ["java", "aws"]},
]}
print(json.dumps(data, indent=2))
```

### Real-World Use Cases

- **Debugging**: Quick variable inspection during development
- **CLI tools**: User-facing output, progress indicators
- **Logging**: Timestamped diagnostic output
- **Data export**: Printing CSV/TSV formatted data
- **Report generation**: Tabular output for console reports
- **Monitoring**: Real-time status updates with flush=True
- **Testing**: Capturing output with `unittest.mock.patch('builtins.print')`

### Common Mistakes

```python
# Mistake 1: Concatenation confusion
# print("Result: " + 42)  # TypeError
print("Result:", 42)  # Correct (multiple args)
print(f"Result: {42}")  # Or f-string

# Mistake 2: Not suppressing newline when needed
print("Loading", end="")
print(".")  # Now prints on same line

# Mistake 3: Forgetting flush for real-time output
import time
for i in range(5):
    print(i, end=" ", flush=True)  # Without flush, buffered!
    time.sleep(0.5)

# Mistake 4: Modifying sep/end globally
# Don't reassign print's defaults unless intentional

# Mistake 5: Using print for production logging
# Use logging module instead:
import logging
logging.basicConfig(level=logging.INFO)
logging.info("This is logged properly")

# Mistake 6: print() with too many arguments (performance)
# For very large output, build string first:
big_string = "\n".join(f"Line {i}" for i in range(1000000))
# Then print once:
print(big_string)
```

### Best Practices

- Use f-strings inside `print()` for formatted output
- Use `file=sys.stderr` for error messages
- Use `flush=True` for real-time progress indicators
- Use `sep` and `end` parameters instead of string concatenation
- Use `logging` module instead of `print()` for production
- Avoid `print()` in library code (use logging or raise exceptions)
- Build large strings before printing once for performance
- Use `json.dumps` with `indent` for pretty-printing structured data

### Performance Considerations

Each `print()` call acquires the GIL, writes to stdout (buffered by default), and releases. For high-volume output, batch writes into a single string and print once. stdout is line-buffered by default when connected to a terminal, but block-buffered when piped — use `flush=True` or `sys.stdout.reconfigure(line_buffering=True)` for predictable behavior.

### Interview Questions

1. What are the `sep`, `end`, `file`, and `flush` parameters of `print()`?
2. How is `print()` different from `sys.stdout.write()`?
3. When would you use `flush=True`?
4. How do you print to stderr?
5. How does `print()` handle multiple arguments?
6. How do you suppress the newline in `print()`?
7. Is `print()` thread-safe?
8. How do you capture `print()` output in tests?

### Coding Challenges

```python
# Challenge 1: Logging wrapper
def create_logger(prefix, output=sys.stdout):
    def log(message):
        from datetime import datetime
        ts = datetime.now().isoformat()
        print(f"[{ts}] [{prefix}] {message}", file=output)
    return log

info = create_logger("INFO")
error = create_logger("ERROR", sys.stderr)
info("System ready")
error("Disk full")

# Challenge 2: Progress bar
def progress_bar(current, total, width=40):
    filled = int(width * current / total)
    bar = "█" * filled + "░" * (width - filled)
    pct = current / total * 100
    print(f"\r[{bar}] {pct:5.1f}% ({current}/{total})", end="", flush=True)

# Challenge 3: Silent print suppressor
import os
def suppress_print():
    sys.stdout = open(os.devnull, "w")
def restore_print():
    sys.stdout = sys.__stdout__

suppress_print()
print("This won't show")
restore_print()
print("This will show")

# Challenge 4: CSV printer
def print_csv(rows):
    for row in rows:
        print(",".join(str(c) for c in row))
```

### Related Topics

- sys.stdout, sys.stderr
- String formatting (f-strings, .format(), %)
- logging module
- File I/O
- Buffering and flushing
- unittest.mock.patch for testing output

## Formatted output

### What It Is

Formatted output controls how values are represented as strings — alignment, padding, precision, number format (binary, hex, decimal), and locale-aware formatting. Python offers three systems: f-strings (Python 3.6+, preferred), the `str.format()` method, and the legacy `%`-formatting.

### Why It Is Important

Proper formatting produces readable reports, aligned tables, locale-correct numbers, and professional-looking CLI output. It distinguishes production-quality tools from quick scripts and is essential for data presentation.

### How It Works Internally

F-strings are evaluated at compile time — the compiler transforms `f"Value: {x}"` into bytecode that builds a string using `FORMAT_VALUE` and `BUILD_STRING` operations. The format spec (e.g., `:.2f`) follows Python's Format Specification Mini-Language, which is parsed by `string.Formatter` or the C-level `PyOS_double_to_string()` for floats. `str.format()` is implemented in Python (the `string` module) and has more overhead than f-strings.

### Syntax

```python
# f-strings (Python 3.6+)
name = "Alice"
print(f"Name: {name}")
print(f"Pi: {3.14159:.2f}")
print(f"Hex: {255:#x}")
print(f"Align: {name:>10}")

# str.format()
print("Name: {}".format(name))
print("Pi: {:.2f}".format(3.14159))
print("{:<10} {:>5}".format("left", 42))

# %-formatting (legacy)
print("Name: %s, Age: %d" % (name, 30))
print("Pi: %.2f" % 3.14159)
```

### Beginner Examples

```python
# f-strings basics
name = "Alice"
age = 30
score = 95.5678

print(f"Name: {name}")
print(f"Age: {age}")
print(f"Score: {score:.1f}")   # 95.6
print(f"Score: {score:.2f}")   # 95.57
print(f"Score: {score:.0f}")   # 96

# Width and alignment
for i in range(1, 6):
    print(f"|{i:>5}|{i**2:<5}|{i**3:^5}|")

# Percentage
rate = 0.8567
print(f"Rate: {rate:.1%}")  # 85.7%

# Thousands separator
large = 1234567
print(f"Large: {large:,}")  # 1,234,567

# Multiple values
print(f"{name} is {age} years old and scored {score:.1f}")
```

### Intermediate Examples

```python
# Format specifiers
val = 42.6789
print(f"Default:      {val}")
print(f"2 decimals:   {val:.2f}")
print(f"6 width, 2 d: {val:8.2f}")
print(f"Left align:   {val:<8.2f}")
print(f"Center:       {val:^8.2f}")
print(f"Sign always:  {val:+}")
print(f"Sign space:   {val: }")

# Number bases
n = 255
print(f"Binary:  {n:b}")   # 11111111
print(f"Octal:   {n:o}")   # 377
print(f"Hex:     {n:x}")   # ff
print(f"Hex up:  {n:X}")   # FF
print(f"Hex #:   {n:#x}")  # 0xff

# str.format() positional and keyword
print("{0} {1} {0}".format("spam", "eggs"))
print("{name} is {age}".format(name="Bob", age=25))

# Date formatting
from datetime import datetime
now = datetime.now()
print(f"{now:%Y-%m-%d %H:%M:%S}")   # 2024-01-15 14:30:00
print(f"{now:%B %d, %Y}")            # January 15, 2024

# Dynamic format spec
width = 10
precision = 3
print(f"{3.14159:{width}.{precision}f}")

# Dict formatting
person = {"name": "Alice", "age": 30}
print(f"{person['name']} is {person['age']}")
```

### Advanced Examples

```python
# Custom __format__ for objects
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __format__(self, fmt):
        if fmt == "polar":
            import math
            r = math.sqrt(self.x**2 + self.y**2)
            theta = math.degrees(math.atan2(self.y, self.x))
            return f"({r:.2f}, {theta:.1f}°)"
        return f"({self.x}, {self.y})"

p = Point(3, 4)
print(f"Cartesian: {p}")         # (3, 4)
print(f"Polar: {p:polar}")       # (5.00, 53.1°)

# Conditional formatting
def format_status(status, start_time):
    elapsed = datetime.now() - start_time
    if status == "running":
        return f"Status: RUNNING ({elapsed.total_seconds():.0f}s)"
    elif status == "done":
        return f"Status: DONE ({elapsed.total_seconds():.0f}s)"
    return f"Status: {status.title()}"

# Template string with format
from string import Template
t = Template("$name has $$ $amount")
print(t.substitute(name="Alice", amount=100))
# Alice has $ 100

# Locale-aware formatting
import locale
locale.setlocale(locale.LC_ALL, "en_US.UTF-8")
print(f"{1234567.89:n}")  # 1,234,567.89 (locale-aware)

# Nested f-string expressions (Python 3.12+)
items = [1, 2, 3]
print(f"Sum: {sum(items)} Average: {sum(items)/len(items):.2f}")

# Rich formatted table
def format_table(header, rows):
    widths = [len(h) for h in header]
    for row in rows:
        for i, cell in enumerate(row):
            widths[i] = max(widths[i], len(str(cell)))
    
    lines = []
    sep = "+-" + "-+-".join("-" * w for w in widths) + "-+"
    hdr = "| " + " | ".join(h.center(w) for h, w in zip(header, widths)) + " |"
    lines.append(sep)
    lines.append(hdr)
    lines.append(sep.replace("-", "="))
    for row in rows:
        line = "| " + " | ".join(str(c).ljust(w) for c, w in zip(row, widths)) + " |"
        lines.append(line)
    lines.append(sep)
    return "\n".join(lines)

print(format_table(["ID", "Name", "Score"],
    [[1, "Alice", 95.5], [2, "Bob", 87.3], [3, "Charlie", 92.8]]))
```

### Real-World Use Cases

- **CLI reports**: Aligned tables, columnar output
- **Logging**: Consistent timestamp and message format
- **Data export**: CSV, TSV, and fixed-width formats
- **Monitoring dashboards**: Real-time formatted console updates
- **Financial reports**: Currency formatting with precision
- **Scientific output**: Significant figures, scientific notation
- **Configuration generators**: Templated config file output

### Common Mistakes

```python
# Mistake 1: Mismatched format spec
val = 3.14
# print(f"{val:d}")  # ValueError: Unknown format code 'd' for float
print(f"{val:.2f}")  # Correct

# Mistake 2: Confusing width and precision
print(f"{3.14159:.2f}")    # 3.14 (precision)
print(f"{3.14159:6.2f}")   # '  3.14' (width + precision)

# Mistake 3: Not escaping braces in f-strings
# print(f"Dict: {person}")  # Correct
# print(f"{a} {b}")         # Correct
print(f"{{literal braces}}")  # {literal braces}

# Mistake 4: Expressions with side effects in f-strings
# print(f"{some_list.pop()}")  # Avoid side effects!

# Mistake 5: Mixing format styles
print(f"Value: {val:.2f}")   # Good
print("Value: {:.2f}".format(val))  # Good
# print(f"Value: {0:.2f}".format(val))  # Don't mix!

# Mistake 6: Forgetting that % formatting only supports tuples
print("Values: %d %d" % (1, 2))  # Correct
print("Values: %d" % 1)          # Single value works
# print("Values: %d %d" % [1, 2])  # TypeError in older Python
```

### Best Practices

- Use f-strings for all new code (Python 3.6+)
- Use `.format()` for dynamic format strings (templates)
- Avoid `%` formatting in new code
- Use format specifiers for precision, alignment, bases
- Use `!r` in f-strings for repr debugging: `f"{obj!r}"`
- Use `__format__` for custom object formatting
- Use `datetime` format codes for date/time output
- Use `:n` for locale-aware number formatting
- Keep format strings short — extract complex formatting to functions

### Performance Considerations

F-strings are the fastest formatting method — they compile to optimized bytecode. `str.format()` is slower (Python-level parsing). `%` formatting is between them in speed but less flexible. For high-performance output in loops, build strings with f-strings and print once. Avoid `.format()` in tight loops where f-strings can be used instead.

### Interview Questions

1. What are the three string formatting methods in Python?
2. Why are f-strings preferred over `.format()` and `%`?
3. How do you align text (left, right, center) in formatted output?
4. How do you format numbers with leading zeros?
5. How do you format binary, octal, and hex values?
6. What is the `__format__` dunder method?
7. How do you use f-string expressions for calculations?
8. How do you escape braces in f-strings?
9. How do you format dates and times?

### Coding Challenges

```python
# Challenge 1: Formatted multiplication table
def multiplication_table(n):
    header = [""] + list(range(1, n+1))
    rows = [[i] + [i*j for j in range(1, n+1)] for i in range(1, n+1)]
    widths = [max(len(str(cell)) for cell in col) for col in zip(*rows)]
    
    for i, row in enumerate(rows):
        line = " | ".join(str(cell).rjust(w) for cell, w in zip(row, widths))
        print(line)
        if i == 0:
            print("-+-".join("-" * w for w in widths))

multiplication_table(5)

# Challenge 2: Pretty byte formatter
def format_bytes(bytes_val):
    for unit in ("B", "KB", "MB", "GB", "TB"):
        if bytes_val < 1024:
            return f"{bytes_val:.2f} {unit}"
        bytes_val /= 1024
    return f"{bytes_val:.2f} PB"

print(format_bytes(1234567890))  # 1.15 GB

# Challenge 3: Account number formatter
def format_account(num, blocks=4, sep="-"):
    s = str(num)
    return sep.join(s[i:i+blocks] for i in range(0, len(s), blocks))

print(format_account("1234567890123456"))  # 1234-5678-9012-3456

# Challenge 4: CSS-style color formatter
def rgb_to_hex(r, g, b):
    return f"#{r:02x}{g:02x}{b:02x}"

def hex_to_rgb(hex_str):
    hex_str = hex_str.lstrip("#")
    return tuple(int(hex_str[i:i+2], 16) for i in range(0, 6, 2))

print(rgb_to_hex(255, 128, 64))  # #ff8040
print(hex_to_rgb("#ff8040"))     # (255, 128, 64)

# Challenge 5: Dynamic progress string
def format_progress(current, total, bar_width=20):
    pct = current / total
    filled = int(bar_width * pct)
    bar = "=" * filled + " " * (bar_width - filled)
    return f"[{bar}] {pct:6.1%}"

for i in range(0, 101, 10):
    print(format_progress(i, 100), end="\r")
    import time; time.sleep(0.1)
print()
```

### Related Topics

- String methods (.zfill(), .ljust(), .rjust(), .center())
- f-strings (PEP 498)
- str.format() (PEP 3101)
- string.Template
- Format Specification Mini-Language
- locale module
- datetime.strftime()
