# Strings - str() constructor, slicing, f-strings, methods

## Introduction

Strings (`str`) are immutable sequences of Unicode code points. They are one of the most commonly used types in Python, essential for text processing, user interaction, data serialization, and output formatting. Python provides rich string support including the `str()` constructor for type conversion, flexible slicing syntax, fast readable f-strings for formatting, and an extensive set of built-in methods for searching, transforming, splitting, joining, and validating strings.

## str() Constructor

### What It Is
The `str()` constructor creates a string representation of any Python object. It calls the object's `__str__()` method (or `__repr__()` as fallback). With no arguments, it returns an empty string.

### Why It Is Important
`str()` enables type conversion to string for concatenation, formatting, and display. Every Python object has a string representation, making `str()` universally applicable. Understanding `__str__` vs `__repr__` is crucial for debugging and logging.

### How It Works Internally
`str(obj)` calls `type(obj).__str__(obj)` via `PyObject_Str()`. If `__str__` is not defined, Python falls back to `__repr__`. If neither exists, it calls `object.__repr__()` which returns `<ClassName object at 0x...>`. For string arguments, `str(s)` returns the string itself (no copy). The constructor can also decode bytes (with encoding argument).

### Syntax
```python
# Basic usage
str()            # "" (empty string)
str(42)          # "42"
str(3.14)        # "3.14"
str(True)        # "True"
str([1, 2, 3])   # "[1, 2, 3]"

# With bytes and encoding
str(b"hello", "utf-8")  # "hello"
str(b"caf\xc3\xa9", "utf-8")  # "café"
```

### Beginner Examples
```python
# Converting numbers
age = 30
message = "You are " + str(age) + " years old"
print(message)  # You are 30 years old

# Converting booleans
print(str(True))   # "True"
print(str(False))  # "False"

# Converting collections
print(str([1, 2, 3]))      # "[1, 2, 3]"
print(str({"a": 1}))        # "{'a': 1}"
print(str((1, 2, 3)))       # "(1, 2, 3)"

# str() on custom objects
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    def __str__(self):
        return f"Person({self.name}, {self.age})"

p = Person("Alice", 30)
print(str(p))  # Person(Alice, 30)
```

### Intermediate Examples
```python
# __str__ vs __repr__
import datetime

today = datetime.date.today()
print(str(today))   # 2026-06-09 (human-readable)
print(repr(today))  # datetime.date(2026, 6, 9) (unambiguous)

# Custom __repr__ for debugging
class Point:
    def __init__(self, x, y):
        self.x, self.y = x, y
    
    def __repr__(self):
        return f"Point({self.x}, {self.y})"
    
    def __str__(self):
        return f"({self.x}, {self.y})"

p = Point(3, 4)
print(str(p))   # (3, 4)
print(repr(p))  # Point(3, 4)

# str() with collections containing custom objects
points = [Point(1, 2), Point(3, 4)]
print(str(points))  # [(3, 4), (3, 4)] — uses __str__? No, uses __repr__!
print(repr(points)) # [Point(1, 2), Point(3, 4)]

# Bytes decoding
data = b"Python is fun"
text = str(data, encoding="utf-8")
print(text)  # Python is fun

# str() for format specification
print(str(3.14159))  # "3.14159" (full precision)
```

### Advanced Examples
```python
# Smart string representation for complex objects
import json
from datetime import datetime

class LogEntry:
    def __init__(self, message: str, level: str = "INFO"):
        self.message = message
        self.level = level
        self.timestamp = datetime.utcnow()
    
    def __repr__(self):
        return f"LogEntry({self.message!r}, {self.level!r})"
    
    def __str__(self):
        return f"[{self.timestamp.isoformat()}] {self.level}: {self.message}"
    
    def to_json(self) -> str:
        return json.dumps({
            "message": self.message,
            "level": self.level,
            "timestamp": self.timestamp.isoformat(),
        })

entry = LogEntry("Server started", "INFO")
print(str(entry))    # [2026-06-09T...] INFO: Server started
print(repr(entry))   # LogEntry('Server started', 'INFO')
print(entry.to_json())  # {"message": "Server started", ...}

# Lazy string conversion for logging
import logging

class LazyStr:
    def __init__(self, func):
        self.func = func
    
    def __str__(self):
        return self.func()

def expensive_str():
    return "expensive computation"

log = logging.getLogger(__name__)
log.debug("Result: %s", LazyStr(expensive_str))  # Only evaluated if DEBUG enabled

# str() behavior with __str__ returning non-string
class IncorrectStr:
    def __str__(self):
        return 42  # Must return str!

# str(IncorrectStr())  # TypeError: __str__ returned non-string
```

### Real-World Use Cases
- **Logging**: Converting objects to log messages
- **Serialization**: Converting data to JSON or CSV strings
- **Error Messages**: Formatting user-facing error strings
- **Debugging**: Using `repr()` for unambiguous object representation
- **Data Export**: Converting database records to text formats

### Common Mistakes
```python
# Mistake 1: str() vs repr() confusion
class Temperature:
    def __init__(self, celsius):
        self.celsius = celsius

t = Temperature(25)
print(str(t))   # <__main__.Temperature object at 0x...> (not helpful)
print(repr(t))  # Same unhelpful output

# Fix: define __str__ and __repr__

# Mistake 2: Forgetting str() when concatenating
# age = 30
# print("Age: " + age)  # TypeError
# Correct:
print("Age: " + str(age))

# Mistake 3: str() on bytes without encoding
data = b"hello"
# print(str(data))  # "b'hello'" (not "hello")
# Correct:
print(str(data, "utf-8"))  # "hello"

# Mistake 4: Expecting str() to format
value = 3.14159
print(str(value))  # "3.14159" (full precision)
# Use f-string for formatting: f"{value:.2f}"
```

### Best Practices
- Define `__repr__` for all custom classes (debugging)
- Define `__str__` when a human-readable form differs from `__repr__`
- Use f-strings instead of `str()` + concatenation
- Use `repr()` for logging/debugging (unambiguous)
- Use `str(obj, encoding)` for bytes-to-str conversion
- Prefer f-strings over `str.format()` and `%` formatting

### Performance Considerations
- `str()` is very fast (C implementation)
- `str(obj)` on a string returns the string itself (no copy)
- `str()` is slower than f-string for simple number conversion
- Python 3.12+ has optimized str() for common types
- `str(big_list)` on large lists can be expensive (full serialization)

### Interview Questions
1. What is the difference between `str()` and `repr()`?
2. How does `str()` handle custom objects?
3. What happens if `__str__` is not defined?
4. How do you convert bytes to string using `str()`?
5. What does `str()` return when called with no arguments?
6. Can `__str__` return a non-string value?
7. How is `str()` used in the `print()` function?
8. What is the difference between `str([1,2,3])` and `repr([1,2,3])`?
9. How does `str()` interact with the `__str__` method?
10. What are the encoding considerations when using `str()` on bytes?

### Coding Challenges
```python
# Challenge 1: Robust str() wrapper
def safe_str(obj, fallback="<unprintable>"):
    try:
        return str(obj)
    except Exception:
        return fallback

# Test with problematic objects
class BadStr:
    def __str__(self):
        raise RuntimeError("Cannot convert!")

print(safe_str(BadStr()))  # <unprintable>

# Challenge 2: Recursive repr for nested structures
def deep_repr(obj, indent=0):
    prefix = " " * indent
    if isinstance(obj, dict):
        items = [f"{prefix}  {k!r}: {deep_repr(v, indent+2)}" for k, v in obj.items()]
        return "{\n" + ",\n".join(items) + f"\n{prefix}}}"
    elif isinstance(obj, (list, tuple)):
        items = [deep_repr(v, indent+2) for v in obj]
        return "[\n" + ",\n".join(items) + f"\n{prefix}]"
    else:
        return repr(obj)

nested = {"a": [1, {"b": 2}], "c": "hello"}
print(deep_repr(nested))
```

### Related Topics
- `repr()` Built-in Function
- `__str__` and `__repr__` Methods
- `__format__` Method
- String Formatting
- Bytes and Encoding

## Slicing

### What It Is
Slicing extracts a contiguous subsequence from a sequence (string, list, tuple) using the `[start:stop:step]` syntax. It returns a new sequence of the same type without modifying the original.

### Why It Is Important
Slicing is one of Python's most elegant features, enabling concise extraction of substrings, reversal, skipping elements, and copying sequences. It is widely used in text processing, data analysis, and algorithm implementation.

### How It Works Internally
String slicing is implemented by `PyUnicode_Substring()` (or `unicodeobject.c`'s slicing path). For positive steps, it creates a new string by copying characters from start to stop-1 with the given step. Negative steps work by iterating backward. Slicing handles out-of-range indices gracefully by clamping to the actual bounds.

### Syntax
```python
# Basic slicing
s[start:stop]    # From start to stop-1
s[start:]        # From start to end
s[:stop]         # From beginning to stop-1
s[:]             # Copy entire sequence
s[::step]        # Every step-th element

# With negative indices
s[-1]            # Last character
s[-3:]           # Last 3 characters
s[:-1]           # All but last
s[::-1]          # Reversed

# Step with start/stop
s[1:10:2]        # Every 2nd from index 1 to 9
```

### Beginner Examples
```python
text = "Python Programming"
print(f"String: '{text}'")

# Basic slicing
print(text[0:6])      # "Python"
print(text[7:18])     # "Programming"
print(text[:6])       # "Python" (from start)
print(text[7:])       # "Programming" (to end)
print(text[:])        # Full copy

# Negative indices
print(text[-1])       # "g" (last char)
print(text[-11:])     # "Programming" (last 11 chars)
print(text[:-8])      # "Python" (all but last 8)

# Step
print(text[::2])      # "Pto rgamn" (every 2nd)
print(text[1::2])     # "yhnPormig" (odd indices)
print(text[::-1])     # "gnimmargorP nohtyP" (reversed)

# Slicing with step and bounds
print(text[2:12:3])   # "t" + "o" + "r" + "m" = "torm"
```

### Intermediate Examples
```python
# Practical string slicing
def reverse_string(s: str) -> str:
    return s[::-1]

def is_palindrome(s: str) -> bool:
    cleaned = "".join(c.lower() for c in s if c.isalnum())
    return cleaned == cleaned[::-1]

def get_extension(filename: str) -> str:
    if "." in filename:
        return filename[filename.rindex(".") + 1:]
    return ""

def truncate(text: str, max_length: int, suffix: str = "...") -> str:
    if len(text) <= max_length:
        return text
    return text[:max_length - len(suffix)] + suffix

print(reverse_string("hello"))        # olleh
print(is_palindrome("Racecar"))       # True
print(get_extension("image.jpg"))     # jpg
print(truncate("Hello World", 8))     # Hello...

# Slicing for string manipulation
def remove_prefix(s: str, prefix: str) -> str:
    if s.startswith(prefix):
        return s[len(prefix):]
    return s

def remove_suffix(s: str, suffix: str) -> str:
    if s.endswith(suffix) and suffix:
        return s[:-len(suffix)]
    return s

# Python 3.9+ has removeprefix and removesuffix built-in:
print("HelloWorld".removeprefix("Hello"))  # World
print("HelloWorld".removesuffix("World"))  # Hello

# Slicing in assignments (for lists, not strings — strings are immutable)
items = [1, 2, 3, 4, 5]
items[1:3] = [10, 20]  # [1, 10, 20, 4, 5]
items[1:3] = []        # [1, 4, 5] (remove slice)
items[1:1] = [2, 3]    # [1, 2, 3, 4, 5] (insert at position)
```

### Advanced Examples
```python
# Custom slicing with __getitem__
class Sentence:
    def __init__(self, text: str):
        self.words = text.split()
    
    def __getitem__(self, key):
        if isinstance(key, slice):
            words = self.words[key]
            return Sentence(" ".join(words))
        return self.words[key]
    
    def __len__(self):
        return len(self.words)
    
    def __repr__(self):
        return f"Sentence({self.words!r})"

sent = Sentence("The quick brown fox jumps over the lazy dog")
print(sent[0])       # "The"
print(sent[1:4])     # Sentence(['quick', 'brown', 'fox'])
print(sent[::-1])    # Sentence(['dog', 'lazy', 'the', ...])

# Slice objects
slice_obj = slice(2, 8, 2)
print(slice_obj)            # slice(2, 8, 2)
print(slice_obj.start)      # 2
print(slice_obj.stop)       # 8
print(slice_obj.step)       # 2

# Using slice() with strings
text = "Python Programming"
s = slice(2, 12, 3)
print(text[s])  # "torm"

# Negative step edge cases
text = "abcdef"
print(text[-1:-4:-1])   # "fed"
print(text[-1:-4:1])    # "" (can't go backward with positive step)
print(text[-4:-1])      # "cde"

# Slicing for efficient string reversal comparison
def compare_reverse_methods(s: str) -> dict:
    import timeit
    return {
        "s[::-1]": timeit.timeit(lambda: s[::-1]),
        "''.join(reversed(s))": timeit.timeit(lambda: "".join(reversed(s))),
    }

# String slicing in data processing
def split_and_slice_csv(line: str, columns: slice) -> list[str]:
    """Extract specific columns from a CSV line."""
    return [line.split(",")[col] for col in range(
        columns.start or 0,
        columns.stop or len(line.split(",")),
        columns.step or 1
    )]

csv_line = "1,Alice,30,NYC,Engineer"
print(split_and_slice_csv(csv_line, slice(1, 4)))  # ['Alice', '30', 'NYC']
```

### Real-World Use Cases
- **Text Processing**: Extracting substrings, removing prefixes/suffixes
- **Data Cleaning**: Slicing fixed-width fields from log files
- **URL Parsing**: Extracting path components from URLs
- **File Handling**: Getting file extensions, trimming whitespace
- **Bioinformatics**: DNA/RNA sequence analysis with step slicing
- **Image Processing**: Slicing pixel rows from image data

### Common Mistakes
```python
# Mistake 1: Off-by-one with stop index
text = "Python"
print(text[0:5])  # "Pytho" (stops at index 5, not including index 5)

# Mistake 2: Assuming out-of-bounds raises IndexError
print(text[0:100])  # "Python" (clamps to length, no error)
print(text[100:])   # "" (empty string, no error)

# Mistake 3: Forgetting strings are immutable
text = "hello"
# text[0] = "H"  # TypeError
# Correct:
text = "H" + text[1:]

# Mistake 4: Confusing step direction
print("abcde"[5:0:-1])  # "edcb" (stops before index 0)
print("abcde"[5:0:-2])  # "ec"

# Mistake 5: Slice assignment with wrong length (only for lists)
# strings: must assign same-length string
# lists: can assign any iterable
```

### Best Practices
- Use `s[::-1]` for string reversal (fast, readable)
- Use `s[:n]` for first n characters
- Use `s[-n:]` for last n characters
- Use `s[1:-1]` to remove first and last character
- Prefer `removeprefix()`/`removesuffix()` (Python 3.9+) over manual slicing
- Avoid negative step with complex start/stop (hard to read)
- Use slice objects for reusable slicing patterns

### Performance Considerations
- String slicing always creates a new string (O(k) where k = slice length)
- `s[:]` creates a cheap reference copy (only for lists/tuples, not strings)
- For strings, `s[:]` creates a new string (immutability → safe aliasing means no copy in some Python implementations)
- Slicing with small steps is fast (C-level memcpy-like operation)
- `s[::-1]` is optimized in CPython (specialized reverse path)
- Avoid repeated slicing of large strings in loops (store the sliced result)

### Interview Questions
1. How does string slicing work with negative indices?
2. What happens if slice indices are out of bounds?
3. Can you slice with a step of 0? Why not?
4. How do you reverse a string using slicing?
5. What is the difference between `s[:]` and `s`?
6. How does slicing work for immutable vs mutable sequences?
7. What is a `slice` object and when would you use it?
8. How do you use slicing to extract every nth character?
9. What is the time complexity of string slicing?
10. How does slicing handle step values greater than 1?

### Coding Challenges
```python
# Challenge 1: Slice-based string rotation
def rotate_string(s: str, n: int) -> str:
    """Rotate string left by n positions."""
    if not s:
        return s
    n = n % len(s)
    return s[n:] + s[:n]

print(rotate_string("HelloWorld", 3))  # loWorldHel
print(rotate_string("HelloWorld", -2))  # ldHelloWor

# Challenge 2: Snake case to camel case using slicing
def snake_to_camel(name: str) -> str:
    parts = name.split("_")
    return parts[0] + "".join(p.capitalize() for p in parts[1:])

print(snake_to_camel("user_name"))       # userName
print(snake_to_camel("first_name_age"))  # firstNameAge

# Challenge 2: Split a string into chunks
def chunk_string(s: str, chunk_size: int) -> list[str]:
    return [s[i:i + chunk_size] for i in range(0, len(s), chunk_size)]

print(chunk_string("HelloWorld", 3))  # ['Hel', 'loW', 'orl', 'd']

# Challenge 3: Extract domain from email using slicing
def get_domain(email: str) -> str | None:
    at_index = email.find("@")
    if at_index == -1:
        return None
    return email[at_index + 1:]

print(get_domain("alice@example.com"))  # example.com
```

### Related Topics
- Indexing (0-based, negative)
- `slice` Built-in
- Sequence Protocol (`__getitem__`)
- `itertools.islice()`
- NumPy Array Slicing

## f-strings

### What It Is
f-strings (formatted string literals, Python 3.6+) embed expressions inside string literals using `{expression}` syntax. They are prefixed with `f` or `F` and provide the fastest, most readable string formatting in Python.

### Why It Is Important
f-strings have become the standard string formatting method in Python. They are faster than `%` formatting, `.format()`, and `string.Template`, more readable (expressions inline), and support full Python expressions including function calls and attribute access.

### How It Works Internally
At parse time, f-strings are converted into a sequence of string constants and `FORMAT_VALUE` bytecode instructions. The expressions inside `{}` are evaluated in the current scope and formatted using `__format__()`. Since Python 3.12, f-strings can be nested and span multiple lines without backslash continuation.

### Syntax
```python
# Basic f-string
name = "Alice"
print(f"Hello, {name}!")

# Expressions
print(f"2 + 2 = {2 + 2}")

# Format specifiers
print(f"{value:10.2f}")
print(f"{value:<20}")   # Left align
print(f"{value:>20}")   # Right align
print(f"{value:^20}")   # Center
print(f"{value:0>5d}")  # Zero-pad

# Calling functions
print(f"Upper: {name.upper()}")

# Dict access
person = {"name": "Alice"}
print(f"Name: {person['name']}")
```

### Beginner Examples
```python
name = "Alice"
age = 30
score = 95.5678

# Basic f-string
print(f"Name: {name}, Age: {age}")

# Format specifiers
print(f"Score: {score:.1f}")    # 95.6 (1 decimal)
print(f"Score: {score:.0f}")    # 96 (no decimal)
print(f"Percent: {score:.1%}")  # 9556.8% (interprets 95.5 as 9556%)

# Width and alignment
print(f"|{name:<10}|")   # |Alice     | (left align, width 10)
print(f"|{name:>10}|")   # |     Alice| (right align)
print(f"|{name:^10}|")   # |  Alice   | (center)

# Padding with character
print(f"{name:*^10}")    # **Alice***

# Number formatting
value = 1234567.89
print(f"{value:,.2f}")   # 1,234,567.89
print(f"{value:.2e}")    # 1.23e+06 (scientific)
```

### Intermediate Examples
```python
# Expressions in f-strings
import math

radius = 5
print(f"Area: {math.pi * radius ** 2:.2f}")

# Conditional expressions
age = 20
print(f"{'Adult' if age >= 18 else 'Minor'}")

# Dict and attribute access
user = {"name": "Bob", "role": "admin"}
print(f"User: {user['name']}, Role: {user['role']}")

# Datetime formatting
from datetime import datetime
now = datetime.now()
print(f"Date: {now:%Y-%m-%d %H:%M:%S}")
print(f"ISO: {now:%FT%T}")

# Multi-line f-strings
long_name = "John Christopher Smith"
formatted = (
    f"Name: {long_name}\n"
    f"Length: {len(long_name)}\n"
    f"Initials: {''.join(w[0] for w in long_name.split())}"
)
print(formatted)

# Nested f-strings (Python 3.12+)
precision = 3
value = 3.14159
print(f"Pi to {precision} decimals: {value:.{precision}f}")

# Debugging with f-strings (Python 3.8+)
x = 10
y = 20
print(f"{x=}, {y=}")              # x=10, y=20
print(f"{x + y=}")                 # x + y=30
print(f"{x=}, {y=}, {x + y=}")    # x=10, y=20, x + y=30
```

### Advanced Examples
```python
# Custom __format__ method
class Money:
    def __init__(self, amount: float, currency: str = "USD"):
        self.amount = amount
        self.currency = currency
    
    def __format__(self, format_spec: str) -> str:
        if format_spec == "sign":
            return f"{self.currency} {self.amount:+.2f}"
        return f"{self.currency} {self.amount:.2f}"

price = Money(49.99)
print(f"{price}")         # USD 49.99
print(f"{price:sign}")    # USD +49.99

# F-string with conversion flags
value = 42
print(f"{value!s}")   # str(value) = "42"
print(f"{value!r}")   # repr(value) = "42"
print(f"{value!a}")   # ascii(value) = "42"

# Lazy f-string evaluation with lambda
import logging
log = logging.getLogger(__name__)
log.debug(f"Expensive: {expensive_func()}")  # Always evaluated!

# Alternative with lazy:
class LazyFormat:
    def __init__(self, func):
        self.func = func
    def __str__(self):
        return self.func()

log.debug("%s", LazyFormat(lambda: f"Expensive: {expensive_func()}"))

# F-string in docstrings (PEP 257 — not recommended but possible)
def greet(name):
    """Say hello."""
    return f"Hello, {name}!"

# Large numbers formatting
big = 1234567890
print(f"{big:_}")     # 1_234_567_890
print(f"{big:,}")     # 1,234,567,890

# F-string for building SQL (use with caution — SQL injection risk!)
table = "users"
column = "name"
value = "Alice"
# print(f"SELECT {column} FROM {table} WHERE {column}='{value}'")
# Better: use parameterized queries

# Formatting with variable width
def format_table(data: list[list], headers: list[str]):
    col_widths = [
        max(len(str(row[i])) for row in data + [headers])
        for i in range(len(headers))
    ]
    header = " | ".join(f"{h:{w}}" for h, w in zip(headers, col_widths))
    separator = "-+-".join("-" * w for w in col_widths)
    rows = [
        " | ".join(f"{str(cell):{w}}" for cell, w in zip(row, col_widths))
        for row in data
    ]
    return "\n".join([header, separator] + rows)

data = [["Alice", 30, "NYC"], ["Bob", 25, "LA"], ["Charlie", 35, "Chicago"]]
print(format_table(data, ["Name", "Age", "City"]))
```

### Real-World Use Cases
- **Logging**: Formatted log messages with context
- **Report Generation**: Formatted numerical output
- **CLI Tools**: Progress bars, formatted tables
- **Data Serialization**: Building JSON/CSV strings
- **Debug Output**: Using `{var=}` for quick diagnostics (Python 3.8+)
- **Templates**: Simple templating for emails, notifications

### Common Mistakes
```python
# Mistake 1: Missing f prefix
name = "Alice"
print("Hello, {name}!")  # "Hello, {name}!" (literal, not f-string)

# Mistake 2: Braces in string
# Use {{ and }} for literal braces:
print(f"{{name}}: {name}")  # "{name}: Alice"

# Mistake 3: Backslash in expression
# print(f"Newline: \n")  # SyntaxError!
# Can't use backslash in f-string expression part
# Correct:
nl = "\n"
print(f"Newline: {nl}")

# Mistake 4: Using quotes inside f-string
person = {"name": "Alice"}
# print(f"Name: {person['name']}")  # OK with single quotes
# print(f"Name: {person["name"]}")  # SyntaxError!

# Mistake 5: Complex expressions in f-strings
# f"{lambda x: x * 2(5)}"  # SyntaxError
# Keep f-string expressions simple

# Mistake 6: Not using !r for repr
path = "C:\\Users\\name"
print(f"Path: {path!r}")   # 'C:\\Users\\name' (with quotes)
```

### Best Practices
- Use f-strings as the default formatting method (Python 3.6+)
- Use `{var=}` for debug output (Python 3.8+)
- Keep expressions simple (extract complex logic to variables)
- Use format specifiers for alignment, precision, and padding
- Prefer f-strings over `.format()` and `%` formatting
- Use `!r` for repr in debugging output
- Use `{{` and `}}` for literal braces in f-strings

### Performance Considerations
- f-strings are the fastest formatting method (parse-time compilation)
- ~1.5-2x faster than `.format()` and `%` formatting
- Python 3.12+ f-strings have improved parsing (faster for complex expressions)
- `{var=}` syntax has negligible overhead
- Nested f-strings (3.12+) have additional parsing cost
- f-strings with many expressions are still faster than alternatives

### Interview Questions
1. What Python version introduced f-strings?
2. What expressions can be used inside f-string curly braces?
3. How do you include literal braces in an f-string?
4. What is the `{var=}` syntax and when was it introduced?
5. How do you format numbers with thousands separators in f-strings?
6. What are the conversion flags `!s`, `!r`, and `!a`?
7. Can you use f-strings in logging calls? What's the caveat?
8. How do nested f-strings work (Python 3.12+)?
9. How do you use format specifiers for alignment?
10. How are f-strings different from template strings?

### Coding Challenges
```python
# Challenge 1: Dynamic table formatter
def format_row(data: list, widths: list[int]) -> str:
    return " | ".join(f"{str(val):{w}}" for val, w in zip(data, widths))

data = [["Alice", 30, 95000], ["Bob", 25, 72000], ["Charlie", 35, 110000]]
headers = ["Name", "Age", "Salary"]
widths = [max(len(str(row[i])) for row in data + [headers]) + 2 for i in range(len(headers))]

print(format_row(headers, widths))
print("-" * sum(widths))
for row in data:
    print(format_row(row, widths))

# Challenge 2: Custom formatting function
def sizeof_fmt(num: float, suffix: str = "B") -> str:
    for unit in ("", "K", "M", "G", "T", "P"):
        if abs(num) < 1024.0:
            return f"{num:3.1f}{unit}{suffix}"
        num /= 1024.0
    return f"{num:.1f}Pi{suffix}"

print(sizeof_fmt(1234567890))  # 1.1GB
print(sizeof_fmt(500))         # 500.0B
```

### Related Topics
- String Formatting Methods
- `__format__` Method
- Format Specification Mini-Language
- Template Strings
- `string.Formatter`

## String Methods

### What It Is
String methods are built-in functions on `str` objects that perform common text operations. They include methods for case conversion, searching, splitting, joining, trimming, padding, checking character types, and more. Strings are immutable — all methods return new strings.

### Why It Is Important
String methods handle the vast majority of text processing tasks without requiring regular expressions or external libraries. They are highly optimized (C implementation) and follow consistent naming patterns, making text manipulation concise and readable.

### How It Works Internally
Each string method is a C function in `unicodeobject.c`. Most methods create a new `PyUnicodeObject` by operating on the internal character array. Methods like `.upper()` and `.lower()` use Unicode case mapping tables. `.split()` and `.join()` are highly optimized — `.split()` scans for delimiters and builds substrings; `.join()` pre-calculates buffer size for single allocation. `.strip()` uses a lookup table for whitespace characters.

### Syntax
```python
# Case methods
s.upper()       # Uppercase
s.lower()       # Lowercase
s.title()       # Title Case
s.capitalize()  # Capitalize first letter
s.swapcase()    # Swap case
s.casefold()    # Aggressive lowercase (for comparison)

# Search methods
s.find(sub)     # First index (or -1)
s.index(sub)    # First index (or ValueError)
s.rfind(sub)    # Last index
s.rindex(sub)   # Last index (or ValueError)
s.count(sub)    # Occurrences
s.startswith(prefix)
s.endswith(suffix)

# Transform methods
s.replace(old, new)
s.strip()       # Trim whitespace
s.lstrip()
s.rstrip()

# Split/Join methods
s.split(sep)    # Split into list
s.rsplit(sep)
s.splitlines()  # Split at line breaks
sep.join(list)  # Join list into string

# Check methods
s.isdigit()
s.isalpha()
s.isalnum()
s.isspace()
s.islower()
s.isupper()
s.istitle()
```

### Beginner Examples
```python
text = "  Hello, Python World!  "

# Case conversion
print(text.upper())           # "  HELLO, PYTHON WORLD!  "
print(text.lower())           # "  hello, python world!  "
print(text.title())           # "  Hello, Python World!  "
print(text.capitalize())      # "  hello, python world!  "
print(text.swapcase())        # "  hELLO, pYTHON wORLD!  "

# Trimming
print(text.strip())           # "Hello, Python World!"
print(text.lstrip())          # "Hello, Python World!  "
print(text.rstrip())          # "  Hello, Python World!"

# Finding
sentence = "The quick brown fox"
print(sentence.find("quick"))  # 4
print(sentence.find("slow"))   # -1
print(sentence.index("brown")) # 10
# sentence.index("slow")       # ValueError

# Checking
print("Python3".isalnum())    # True
print("Python".isalpha())     # True
print("123".isdigit())        # True
print("hello".islower())      # True
print("HELLO".isupper())      # True
print("   ".isspace())        # True
```

### Intermediate Examples
```python
# Split and join
csv = "apple,banana,cherry,date"
fruits = csv.split(",")
print(fruits)  # ['apple', 'banana', 'cherry', 'date']

joined = " | ".join(fruits)
print(joined)  # "apple | banana | cherry | date"

# Split with maxsplit
data = "a:b:c:d:e"
print(data.split(":", 2))    # ['a', 'b', 'c:d:e']
print(data.rsplit(":", 2))   # ['a:b:c', 'd', 'e']

# Partition — split into three parts
print(data.partition(":"))   # ('a', ':', 'b:c:d:e')
print(data.rpartition(":"))  # ('a:b:c:d', ':', 'e')

# Replace
text = "Hello, World!"
print(text.replace("World", "Python"))  # Hello, Python!
print(text.replace("l", "L"))           # HeLLo, WorLd!
print(text.replace("l", "L", 1))        # HeLlo, World! (only first)

# Translate
trans = str.maketrans({"a": "@", "e": "3", "o": "0"})
print("hello world".translate(trans))  # "h3ll0 w0rld"

# String alignment
print("42".zfill(5))       # "00042"
print("hello".ljust(10))   # "hello     "
print("hello".rjust(10))   # "     hello"
print("hello".center(10))  # "  hello   "

# expandtabs
print("a\tb\tc".expandtabs(10))  # "a         b         c"
```

### Advanced Examples
```python
# Custom string method chaining
class EnhancedStr:
    def __init__(self, value: str):
        self.value = value
    
    def __getattr__(self, name):
        # Forward to str methods
        attr = getattr(self.value, name)
        if callable(attr):
            def wrapper(*args, **kwargs):
                result = attr(*args, **kwargs)
                if isinstance(result, str):
                    return EnhancedStr(result)
                return result
            return wrapper
        return attr
    
    def __repr__(self):
        return f"'{self.value}'"

s = EnhancedStr("  Hello, Python!  ")
result = s.strip().upper().replace("PYTHON", "World")
print(result)  # 'HELLO, WORLD!'

# Case-insensitive comparison
def case_insensitive_equal(a: str, b: str) -> bool:
    return a.casefold() == b.casefold()

# .casefold() is more aggressive than .lower() for German ß etc.
print(case_insensitive_equal("straße", "STRASSE"))  # True (casefold)
print("straße".lower() == "STRASSE".lower())        # False (lower not enough)

# Using str methods for validation
def validate_password(password: str) -> list[str]:
    errors = []
    if len(password) < 8:
        errors.append("Must be 8+ characters")
    if not any(c.isupper() for c in password):
        errors.append("Must have uppercase")
    if not any(c.islower() for c in password):
        errors.append("Must have lowercase")
    if not any(c.isdigit() for c in password):
        errors.append("Must have digit")
    if not any(c in "!@#$%^&*" for c in password):
        errors.append("Must have special char")
    return errors

# str.maketrans and translate for speed
def remove_punctuation(text: str) -> str:
    import string
    translator = str.maketrans("", "", string.punctuation)
    return text.translate(translator)

print(remove_punctuation("Hello, World! How's it going?"))
# "Hello World Hows it going"

# Efficient character counting
from collections import Counter

def char_frequency(text: str) -> Counter:
    return Counter(text.lower())

freq = char_frequency("Hello World")
print(freq.most_common(3))  # [('l', 3), ('o', 2), ('h', 1)]

# str methods with data cleaning
def clean_text(text: str) -> str:
    """Normalize whitespace and strip."""
    import re
    text = text.strip()
    text = re.sub(r'\s+', ' ', text)  # Collapse multiple spaces
    return text

print(repr(clean_text("  Hello    World!  ")))  # 'Hello World!'

# String method performance comparison
import timeit

text = "hello world python programming" * 1000
# Method chaining vs multiple operations
start = timeit.default_timer()
result = text.strip().upper().replace(" ", "_")
print(f"Chained: {timeit.default_timer() - start:.6f}s")
```

### Real-World Use Cases
- **Data Cleaning**: `.strip()`, `.replace()`, `.lower()` for normalizing text
- **CSV Parsing**: `.split(",")` for field extraction
- **URL/Path Parsing**: `.split("/")`, `.startswith("https")`
- **Validation**: `.isdigit()` for numeric checks, `.isalpha()` for names
- **Text Processing**: `.join()` for building strings from lists
- **Log Analysis**: `.find()`, `.count()`, `.startswith()` for pattern matching
- **User Input**: `.strip().lower()` for case-insensitive comparison

### Common Mistakes
```python
# Mistake 1: Forgetting strings are immutable
text = "HELLO"
text.lower()  # Returns new string, doesn't modify
print(text)  # "HELLO"
# Correct:
text = text.lower()

# Mistake 2: Using split() vs split(' ')
"hello   world".split()     # ['hello', 'world'] (any whitespace)
"hello   world".split(' ')  # ['hello', '', '', 'world'] (individual spaces)

# Mistake 3: find() vs index()
# find returns -1, index raises ValueError
text = "hello"
print(text.find("z"))   # -1
# print(text.index("z"))  # ValueError

# Mistake 4: Using in for substring check
# if text.find("sub") != -1:  # Works but not Pythonic
# Better:
if "sub" in text:
    pass

# Mistake 5: join() on non-string list
items = [1, 2, 3]
# ",".join(items)  # TypeError
# Correct:
",".join(str(x) for x in items)

# Mistake 6: Not handling encoding with .encode()/.decode()
```

### Best Practices
- Use `.casefold()` for case-insensitive comparison (more thorough than `.lower()`)
- Use `.join()` for string concatenation of lists (never `+=` in a loop)
- Use `.split()` without arguments to split on any whitespace
- Use `.partition()` instead of `.split(maxsplit=1)` for simpler code
- Use `.translate()` for batch character replacement (faster than chained `.replace()`)
- Use `.startswith()`/`.endswith()` with tuples for multiple prefixes/suffixes
- Use `str.isdigit()` for numeric checks (but beware of Unicode digits)
- Prefer `in` operator over `.find()` for existence checks

### Performance Considerations
- `.join()` is O(n) — always use it over `+=` in loops
- `.translate()` is faster than repeated `.replace()` calls
- `.split()` without arguments handles all whitespace and compresses runs
- `.count()` is O(n) — use `collections.Counter` for multiple character counts
- `.strip()`, `.lower()`, `.upper()` are fast C implementations
- `.startswith()`/`.endswith()` with tuples check multiple prefixes in one pass
- Python 3.12+ has optimized `.split()` and `.join()` implementations

### Interview Questions
1. What is the difference between `.find()` and `.index()`?
2. How do `.split()` and `.split(' ')` differ?
3. What is `.casefold()` and how is it different from `.lower()`?
4. How does `.join()` work and why is it faster than `+=`?
5. What does `.partition()` return and when would you use it?
6. How do you remove all occurrences of a character from a string?
7. What is `.translate()` and how does it compare to `.replace()`?
8. How do you check if a string contains only digits?
9. What is the difference between `.strip()`, `.lstrip()`, and `.rstrip()`?
10. How do you efficiently count character frequencies in a string?

### Coding Challenges
```python
# Challenge 1: String compressor using method chaining
def compress(s: str) -> str:
    if not s:
        return ""
    result = []
    count = 1
    for i in range(1, len(s)):
        if s[i] == s[i-1]:
            count += 1
        else:
            result.append(f"{s[i-1]}{count}")
            count = 1
    result.append(f"{s[-1]}{count}")
    compressed = "".join(result)
    return compressed if len(compressed) < len(s) else s

print(compress("aaabbbcc"))     # a3b3c2
print(compress("abc"))          # abc (not shorter)

# Challenge 2: Simple template engine using string methods
class SimpleTemplate:
    def __init__(self, template: str):
        self.template = template
    
    def render(self, **kwargs) -> str:
        result = self.template
        for key, value in kwargs.items():
            result = result.replace(f"{{{{ {key} }}}}", str(value))
        return result

t = SimpleTemplate("Hello, {{ name }}! You are {{ age }} years old.")
print(t.render(name="Alice", age=30))
# Hello, Alice! You are 30 years old.

# Challenge 3: Advanced URL parser
from urllib.parse import urlparse

def parse_url(url: str) -> dict:
    parsed = urlparse(url)
    return {
        "scheme": parsed.scheme,
        "hostname": parsed.hostname,
        "port": parsed.port,
        "path": parsed.path,
        "query": parsed.query,
        "fragment": parsed.fragment,
    }

print(parse_url("https://example.com:8080/path/to/page?key=val#section"))
```

### Related Topics
- Regular Expressions (`re` Module)
- String Formatting
- Unicode and Encoding
- `string` Module (constants like `string.digits`, `string.punctuation`)
- `collections.Counter`
- `textwrap` Module
