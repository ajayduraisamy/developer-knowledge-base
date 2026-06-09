# Type Casting - int(), float(), str(), bool(), explicit conversion

## Introduction

Type casting (type conversion) transforms a value from one data type to another. Python supports implicit conversion (automatic widening by the interpreter) and explicit conversion using built-in functions like int(), float(), str(), bool(), list(), tuple(), set(), and dict(). Understanding both forms is critical for handling user input, data serialization, arithmetic operations, and interop between libraries.

## int() conversion

### What It Is

int(x, base=10) converts a number or string to an integer. It accepts ints, floats (truncates toward zero), strings (with optional base), bytes, and objects implementing __int__().

### Why It Is Important

Integer conversion is essential for processing user input, parsing configuration values, working with indices, performing exact arithmetic, and converting floating-point data to discrete values.

### How It Works Internally

For strings, int() calls PyLong_FromString(), which parses the C string, handles sign and base prefix detection. For floats, PyLong_FromDouble() truncates toward zero. For other objects, int() calls the __int__() slot if defined, else __index__().

### Syntax

```python
i = int("42")
i = int("1010", 2)    # 10 (binary)
i = int("FF", 16)     # 255 (hex)
i = int("0xFF", 0)    # 255 (auto-detect base)
i = int(3.14)         # 3 (truncates)
```

### Beginner Examples

```python
age = int("30")
print(age + 10)  # 40
print(int(3.99))   # 3
print(int(-3.99))  # -3
print(int(True))   # 1
print(int("1010", 2))  # 10
```

### Intermediate Examples

```python
def safe_int(value, default=0):
    try:
        return int(value)
    except (ValueError, TypeError):
        return default

class Score:
    def __init__(self, value):
        self.value = value
    def __int__(self):
        return int(self.value * 100)

print(int(Score(0.95)))  # 95
```

### Advanced Examples

```python
nums = list(map(int, ["1", "2", "3"]))
print(nums)  # [1, 2, 3]
huge = int("1" + "0" * 1000)
print(huge.bit_length())  # 3322 bits (no overflow)
```

### Real-World Use Cases

User input conversion, config parsing, CSV/JSON integer fields, ID normalization, base conversion.

### Common Mistakes

```python
# int("3.14")  # ValueError
print(int(-3.9))   # -3 (truncation toward zero)
import math
print(math.floor(-3.9))  # -4 (floor is different)
# int(None)  # TypeError
# int("")    # ValueError
```

### Best Practices

Wrap in try/except ValueError; use int(x, 0) for auto base detection; use map(int, seq) for bulk conversion.

### Performance Considerations

O(n) for string conversion (n = digits), O(1) for float. Python ints are arbitrary-precision (no overflow).

### Interview Questions

1. What happens with int("3.14")?
2. How does int() handle different bases?
3. What is the difference between int(-3.9) and math.floor(-3.9)?
4. Can int() overflow in Python?
5. What is the __int__ dunder method used for?

### Coding Challenges

```python
def roman_to_int(s):
    values = {"I": 1, "V": 5, "X": 10, "L": 50, "C": 100, "D": 500, "M": 1000}
    total, prev = 0, 0
    for c in reversed(s):
        curr = values[c]
        total += -curr if curr < prev else curr
        prev = curr
    return total

print(roman_to_int("XIV"))       # 14
print(roman_to_int("MCMXCIV"))   # 1994

def binary_to_int(s):
    result = 0
    for c in s:
        result = result * 2 + (c == "1")
    return result

print(binary_to_int("1010"))  # 10
```

### Related Topics

- float() conversion, str() conversion, math.floor/ceil, __int__/__index__ dunder methods

## float() conversion

### What It Is

float(x) converts to double-precision floating-point (64-bit IEEE 754). Accepts ints, strings, bytes, and objects with __float__().

### Why It Is Important

Float conversion is essential for scientific computing, user-supplied measurements, parsing decimal data, and any domain requiring fractional values.

### How It Works Internally

float() calls PyFloat_FromString() which uses strtod() for parsing, supporting scientific notation, inf, and NaN. For ints, PyFloat_FromDouble() converts the Python int to a C double (may lose precision for very large ints).

### Syntax

```python
f = float("3.14")
f = float("1e-3")      # 0.001
f = float("inf")
f = float("nan")
f = float(42)          # 42.0
f = float(True)        # 1.0
```

### Beginner Examples

```python
price = float("19.99")
print(price + 0.01)  # 20.0
print(float(10))     # 10.0
print(float("1e6"))  # 1000000.0
print(float("inf"))  # inf
print(float("nan"))  # nan
```

### Intermediate Examples

```python
def safe_float(value, default=0.0):
    try:
        return float(value)
    except (ValueError, TypeError):
        return default

class Temperature:
    def __init__(self, celsius):
        self.celsius = celsius
    def __float__(self):
        return float(self.celsius)

print(float(Temperature(23.5)))  # 23.5

data = ["1.5", "2.7", "3.14"]
floats = list(map(float, data))
print(floats)  # [1.5, 2.7, 3.14]
```

### Advanced Examples

```python
print(0.1 + 0.2)  # 0.30000000000000004 (IEEE 754)
print(0.1 + 0.2 == 0.3)  # False!
print(abs((0.1 + 0.2) - 0.3) < 1e-9)  # True (tolerance)

def clean_floats(dataset):
    if isinstance(dataset, list):
        return [clean_floats(item) for item in dataset]
    if isinstance(dataset, str):
        try:
            return float(dataset)
        except ValueError:
            return dataset
    return dataset

messy = ["3.14", "abc", "1e3"]
print(clean_floats(messy))  # [3.14, 'abc', 1000.0]
```

### Real-World Use Cases

Scientific measurements, config thresholds, JSON float fields, CSV price/weight columns.

### Common Mistakes

```python
print(0.1 + 0.2)  # 0.30000000000000004
print(0.1 + 0.2 == 0.3)  # False!
# float("$3.14")  # ValueError -- strip symbols first
print(float(10**1000))  # inf (overflow to infinity)
```

### Best Practices

Use Decimal for financial calculations; use math.isclose() for float equality.

### Performance Considerations

float() is O(1) for ints, O(n) for string length. Decimal is significantly slower but exact.

### Interview Questions

1. Why is 0.1 + 0.2 != 0.3?
2. How does Python represent floats internally?
3. What does float("inf") return?
4. What happens when converting a very large int to float?

### Coding Challenges

```python
def parse_currency(s):
    return float(s.replace("$", "").replace(",", ""))

print(parse_currency("$1,234.56"))  # 1234.56

def celsius_to_fahrenheit(c):
    return float(c) * 9/5 + 32

print(celsius_to_fahrenheit("100"))  # 212.0
```

### Related Topics

- int() conversion, decimal.Decimal, math.isclose(), IEEE 754

## str() conversion

### What It Is

str(object) returns a string representation. Calls __str__() first, falling back to __repr__().

### Why It Is Important

String conversion is the most common type conversion -- used for output, concatenation, logging, serialization.

### How It Works Internally

Checks for __str__() method; if not defined, uses __repr__(). For bytes, accepts encoding parameter.

### Syntax

```python
s = str(42)        # "42"
s = str(3.14)      # "3.14"
s = str([1, 2])   # "[1, 2]"
s = str(True)     # "True"
s = str(None)     # "None"
```

### Beginner Examples

```python
count = 42
message = "Count: " + str(count)
print(message)  # Count: 42

print(str(3.14))    # 3.14
print(str(True))    # True
print(str(None))    # None
print(str([1, 2]))  # [1, 2]

# Without str(), concatenation fails
# "Value: " + 42  # TypeError
```

### Intermediate Examples

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    def __str__(self):
        return f"{self.name}, {self.age}yo"
    def __repr__(self):
        return f"Person({self.name!r}, {self.age})"

p = Person('Alice', 30)
print(str(p))   # Alice, 30yo
print(repr(p))  # Person('Alice', 30)

import datetime
d = datetime.datetime(2024, 1, 15)
print(str(d))   # 2024-01-15 00:00:00
print(repr(d))  # datetime.datetime(2024, 1, 15, 0, 0)
```

### Advanced Examples

```python
data = b"hello"
s = str(data, "utf-8")
print(s)  # hello

class Report:
    def __init__(self, title, data):
        self.title = title
        self.data = data
    def __str__(self):
        lines = [f"Report: {self.title}"]
        for k, v in self.data.items():
            lines.append(f"  {k}: {v}")
        return "\n".join(lines)

r = Report("Sales", {"Q1": 100, "Q2": 200})
print(str(r))
```

### Real-World Use Cases

String formatting, logging, serialization, user-facing display, error messages.

### Common Mistakes

```python
s = "hello\nworld"
print(str(s))   # hello (newline) world
print(repr(s))  # 'hello\\nworld' (escaped)

# str() on bytes without encoding
print(str(b"\\xc3\\xa9"))  # b'\\xc3\\xa9'
print(str(b"\\xc3\\xa9", "utf-8"))  # Correct
```

### Best Practices

Implement __str__ for display, __repr__ for debugging; use f-strings over str() concatenation.

### Performance Considerations

str() is O(n) for the string length. Custom __str__() complexity depends on implementation.

### Interview Questions

1. What is the difference between __str__ and __repr__?
2. How do you convert bytes to a string?
3. What is the default str() behavior for objects?

### Coding Challenges

```python
class Color:
    def __init__(self, r, g, b):
        self.r, self.g, self.b = r, g, b
    def __str__(self):
        return f"rgb({self.r}, {self.g}, {self.b})"
    def __repr__(self):
        return f"Color({self.r}, {self.g}, {self.b})"

c = Color(255, 128, 64)
print(str(c))   # rgb(255, 128, 64)
print(repr(c))  # Color(255, 128, 64)

def str_join(items, sep=", "):
    return sep.join(str(item) for item in items)

print(str_join([1, "hello", 3.14]))  # 1, hello, 3.14
```

### Related Topics

- repr() built-in, __str__ vs __repr__, encode()/decode()

## bool() conversion

### What It Is

bool(x) converts to True or False. Falsy values: None, False, zero numerics, empty collections/strings. Everything else is True.

### Why It Is Important

Used in truthiness testing for conditionals, filtering, and logical operations. Understanding what converts to False prevents subtle bugs.

### How It Works Internally

CPython's PyObject_IsTrue(): checks None/False returns False; numeric returns x != 0; __len__() returns len != 0; __bool__() if defined; otherwise True.

### Syntax

```python
b = bool(0)        # False
b = bool(1)        # True
b = bool("")       # False
b = bool("hello")  # True
b = bool([])       # False
b = bool(None)     # False
```

### Beginner Examples

```python
print(bool(False))    # False
print(bool(0))        # False
print(bool(0.0))      # False
print(bool(""))       # False
print(bool([]))       # False
print(bool({}))       # False
print(bool(None))     # False
print(bool(1))        # True
print(bool(-1))       # True
print(bool([0]))      # True (non-empty list)
print(bool("False"))  # True! (non-empty string)
```

### Intermediate Examples

```python
print(bool("False"))  # True -- common gotcha!

def str_to_bool(s):
    return s.lower() in ("true", "yes", "1", "y")

print(str_to_bool("True"))   # True
print(str_to_bool("false"))  # False

class Account:
    def __init__(self, balance):
        self.balance = balance
    def __bool__(self):
        return self.balance > 0

print(bool(Account(100)))  # True
print(bool(Account(0)))    # False

# Using bool for filtering
items = [0, 1, "", "hello", None, [], [1]]
truthy = [x for x in items if bool(x)]
print(truthy)  # [1, 'hello', [1]]
```

### Advanced Examples

```python
class Team:
    def __init__(self, members):
        self.members = members
    def __len__(self):
        return len(self.members)

t1 = Team(['Alice', 'Bob'])
t2 = Team([])
print(bool(t1))  # True
print(bool(t2))  # False

# Numeric bool behavior
print(bool(0))      # False
print(bool(0.0))    # False
print(bool(1))      # True
print(bool(-1))     # True
```

### Real-World Use Cases

Conditional checks, data validation, filtering falsy values, flag parsing, object state checks.

### Common Mistakes

```python
print(bool("False"))  # True (non-empty string is always True)
# Use str_to_bool() for string parsing

print(bool([]))     # False
print(bool([[]]))   # True (non-empty list containing empty list)

# if bool(x): is same as if x:
# Prefer: if x:
```

### Best Practices

Use 'if x:' not 'if bool(x):'; implement __bool__ for custom truthiness; use explicit string parsing for boolean strings.

### Performance Considerations

bool() is extremely fast -- just a pointer check and optionally a method call. 'if x:' implicitly calls bool().

### Interview Questions

1. What values are falsy in Python?
2. Why is bool("False") True?
3. How is bool related to int?
4. What is the difference between __bool__ and __len__?

### Coding Challenges

```python
def safe_bool(value):
    if isinstance(value, str):
        return value.lower() in ("true", "yes", "1", "y")
    return bool(value)

print(safe_bool("True"))    # True
print(safe_bool("false"))   # False
print(safe_bool(1))         # True

def all_true(iterable):
    return all(iterable)

print(all_true([1, "hello", [1]]))   # True
print(all_true([1, "", [1]]))        # False
```

### Related Topics

- Truthiness, __bool__ vs __len__, all()/any() built-ins, short-circuit evaluation

## Explicit conversion

### What It Is

Explicit conversion (casting) uses built-in functions: list(), tuple(), set(), dict(), frozenset(), complex(), bytes(), bytearray().

### Why It Is Important

Python rarely performs implicit cross-type conversions beyond numeric widening. Strings must be explicitly parsed, collections must be explicitly converted.

### How It Works Internally

Each converter: checks if already target type; for strings, parses; for collections, iterates and constructs new collection; for custom types, calls appropriate dunder method.

### Syntax

```python
list('hello')       # ['h', 'e', 'l', 'l', 'o']
tuple([1, 2, 3])    # (1, 2, 3)
set([1, 2, 2, 3])   # {1, 2, 3}
dict([('a', 1)])    # {'a': 1}
frozenset([1, 2])
bytes([65, 66])     # b'AB'
complex(1, 2)       # (1+2j)
```

### Beginner Examples

```python
text = 'hello'
chars = list(text)
print(chars)  # ['h', 'e', 'l', 'l', 'o']

fruits = ['apple', 'banana']
fruits_t = tuple(fruits)
print(fruits_t)  # ('apple', 'banana')

numbers = [1, 2, 2, 3]
unique = set(numbers)
print(unique)  # {1, 2, 3}

pairs = [('a', 1), ('b', 2)]
d = dict(pairs)
print(d)  # {'a': 1, 'b': 2}
```

### Intermediate Examples

```python
matrix_str = "1 2 3; 4 5 6; 7 8 9"
rows = matrix_str.split(";")
matrix = [list(map(int, row.split())) for row in rows]
print(matrix)  # [[1, 2, 3], [4, 5, 6], [7, 8, 9]]

data = bytes([65, 66, 67])
print(data)  # b'ABC'

c = complex('3+4j')
print(c)  # (3+4j)
```

### Advanced Examples

```python
def convert(value, target_type, default=None):
    try:
        if target_type == int:
            return int(value)
        elif target_type == float:
            return float(value)
        elif target_type == str:
            return str(value)
        elif target_type == bool and isinstance(value, str):
            return value.lower() in ("true", "yes", "1", "y")
    except (ValueError, TypeError):
        return default

print(convert("42", int))       # 42
print(convert("true", bool))    # True

def infer_type(value):
    if value is None:
        return None
    try:
        return int(value) if "." not in value else float(value)
    except (ValueError, TypeError):
        return value.lower() in ("true", "false") if isinstance(value, str) else value
```

### Real-World Use Cases

CSV/JSON parsing, data cleaning, API response conversion, config parsing, ETL pipelines.

### Common Mistakes

```python
print(list('hello'))       # ['h', 'e', 'l', 'l', 'o']
print('hello'.split())     # ['hello'] -- different!
# set([[1, 2], [3, 4]])   # TypeError (unhashable)
# dict([1, 2, 3])          # ValueError (not pairs)
```

### Best Practices

Use explicit conversion; handle errors with try/except; use map(type_func, data) for bulk conversion.

### Performance Considerations

list()/tuple() on existing collection: O(n) shallow copy. set() deduplicates O(n). dict() on pairs O(n).

### Interview Questions

1. How do you convert a list of tuples to a dictionary?
2. Why is list('hello') different from 'hello'.split()?
3. What happens with dict() on a list of 2-element lists?
4. How do you safely convert a value to multiple types?

### Coding Challenges

```python
def convert_row(headers, values, types):
    return {h: types[i](v) for i, (h, v) in enumerate(zip(headers, values))}

headers = ["name", "age", "score"]
row = ["Alice", "30", "95.5"]
types = [str, int, float]
print(convert_row(headers, row, types))

def flatten_and_convert(nested):
    result = []
    for item in nested:
        if isinstance(item, (list, tuple)):
            result.extend(flatten_and_convert(item))
        else:
            result.append(item)
    return result

nested = [[1, 2], (3, "4"), [5, 6.0]]
flat = flatten_and_convert(nested)
print(flat)  # [1, 2, 3, '4', 5, 6.0]
```

### Related Topics

- int(), float(), str(), bool() conversions; list(), tuple(), set(), dict() constructors; type coercion vs conversion