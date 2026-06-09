# Type Casting - int(), float(), str(), bool(), explicit conversion

## Introduction

Type casting (or type conversion) is the process of converting a value from one data type to another. Python supports two types of conversion: implicit (automatic conversion done by Python) and explicit (manual conversion using built-in functions like `int()`, `float()`, `str()`, `list()`, `tuple()`, `set()`, and `dict()`). Understanding type casting is crucial for working with different data types effectively.

## Why It Is Important

Type casting is important because:
- User input always comes as strings and needs conversion
- Mathematical operations require compatible numeric types
- Data serialization often requires type conversion
- Different APIs and libraries expect specific types
- Implicit conversion can cause subtle bugs if not understood
- It enables interoperability between different data types

## Syntax

```python
# Explicit type conversion
int_value = int(x)       # Convert to integer
float_value = float(x)   # Convert to float
str_value = str(x)       # Convert to string
bool_value = bool(x)     # Convert to boolean
list_value = list(x)     # Convert to list
tuple_value = tuple(x)   # Convert to tuple
set_value = set(x)       # Convert to set
dict_value = dict(x)     # Convert to dict

# Checking types
type(value)              # Returns the type
isinstance(value, type)  # Boolean check
```

## Examples

### Explicit Type Conversion

```python
# String to int
age = "30"
age_int = int(age)
print(f"age_int: {age_int}, type: {type(age_int)}")

# String to float
price = "19.99"
price_float = float(price)
print(f"price_float: {price_float}, type: {type(price_float)}")

# Number to string
count = 42
count_str = str(count)
print(f"count_str: '{count_str}', type: {type(count_str)}")

# Float to int (truncates decimal)
pi = 3.14159
pi_int = int(pi)
print(f"pi_int: {pi_int}")  # 3

# Int to float
num = 10
num_float = float(num)
print(f"num_float: {num_float}")  # 10.0

# String to list
word = "hello"
word_list = list(word)
print(f"word_list: {word_list}")  # ['h', 'e', 'l', 'l', 'o']

# List to tuple
fruits = ["apple", "banana", "cherry"]
fruits_tuple = tuple(fruits)
print(f"fruits_tuple: {fruits_tuple}")

# Tuple to list
coordinates = (10, 20)
coordinates_list = list(coordinates)
print(f"coordinates_list: {coordinates_list}")

# List to set (removes duplicates)
numbers = [1, 2, 2, 3, 3, 4]
numbers_set = set(numbers)
print(f"numbers_set: {numbers_set}")  # {1, 2, 3, 4}
```

### Implicit Type Conversion

```python
# Python automatically converts in many cases
# int + float -> float
result = 10 + 3.14
print(f"10 + 3.14 = {result}, type: {type(result)}")  # float

# int + bool -> int (True=1, False=0)
result = 10 + True
print(f"10 + True = {result}, type: {type(result)}")  # int

# Implicit conversion in comparisons
print(f"1 == True: {1 == True}")      # True
print(f"0 == False: {0 == False}")    # True
print(f"0 == '': {0 == ''}")          # False (no implicit str conversion)

# Division always returns float
print(f"10 / 3 = {10 / 3}, type: {type(10 / 3)}")

# Boolean conversions
print(f"bool(42): {bool(42)}")    # True
print(f"bool(0): {bool(0)}")      # False
print(f"bool(''): {bool('')}")    # False
print(f"bool('hello'): {bool('hello')}")  # True
```

## Beginner Examples

```python
# Converting user input
age_input = input("Enter your age: ")
age = int(age_input)  # Must convert from string
print(f"Next year you will be {age + 1}")

# Safe conversion with validation
def safe_int(value):
    try:
        return int(value)
    except (ValueError, TypeError):
        return 0

print(safe_int("42"))    # 42
print(safe_int("abc"))   # 0
print(safe_int(None))    # 0

# Common conversions in practice
num1 = float(input("First number: "))
num2 = float(input("Second number: "))
print(f"Sum: {num1 + num2}")

# Formatting with str conversion
total = 42.5
message = "Total is: " + str(total)  # Must convert to concatenate
print(message)

# Using f-strings handles conversion automatically
print(f"Total is: {total}")

# List conversion
text = "Python"
chars = list(text)
print(f"Chars: {chars}")  # ['P', 'y', 't', 'h', 'o', 'n']

# Tuple conversion
colors = ["red", "green", "blue"]
colors_tuple = tuple(colors)
print(f"Colors tuple: {colors_tuple}")
```

## Intermediate Examples

```python
# Type conversions with collections
# String to list of words
sentence = "The quick brown fox"
words = sentence.split()  # ['The', 'quick', 'brown', 'fox']
print(f"Words: {words}")

# List to dict (from pairs)
pairs = [("a", 1), ("b", 2), ("c", 3)]
d = dict(pairs)
print(f"Dict: {d}")

# Dict keys/values to list
d = {"name": "Alice", "age": 30, "city": "NYC"}
keys = list(d.keys())
values = list(d.values())
print(f"Keys: {keys}")
print(f"Values: {values}")

# Nested conversions
matrix_str = "1 2 3; 4 5 6; 7 8 9"
rows = matrix_str.split(";")
matrix = [list(map(int, row.split())) for row in rows]
print(f"Matrix: {matrix}")

# Converting between number bases
print(f"int('1010', 2): {int('1010', 2)}")    # 10 (binary to int)
print(f"int('FF', 16): {int('FF', 16)}")       # 255 (hex to int)
print(f"bin(10): {bin(10)}")                   # '0b1010'
print(f"hex(255): {hex(255)}")                 # '0xff'
print(f"oct(8): {oct(8)}")                     # '0o10'

# Handling None conversions
def convert_safe(value, target_type, default=None):
    try:
        if value is None:
            return default
        return target_type(value)
    except (ValueError, TypeError):
        return default

print(convert_safe("42", int))          # 42
print(convert_safe("abc", int))         # None
print(convert_safe(None, int, 0))       # 0
```

## Advanced Examples

```python
# Custom type conversion
from datetime import datetime

def str_to_date(date_string, formats=None):
    """Convert string to datetime with multiple format support."""
    if formats is None:
        formats = ["%Y-%m-%d", "%m/%d/%Y", "%d-%m-%Y", "%Y/%m/%d"]
    for fmt in formats:
        try:
            return datetime.strptime(date_string, fmt)
        except ValueError:
            continue
    raise ValueError(f"Could not parse date: {date_string}")

dates = ["2024-01-15", "01/15/2024", "15-01-2024"]
for d in dates:
    dt = str_to_date(d)
    print(f"'{d}' -> {dt.date()}")

# Converting complex types with type annotations
from typing import List, Dict, Any
import json

def convert_to_type(value: Any, target_type: type) -> Any:
    """Convert a value to the specified type."""
    if target_type == bool:
        if isinstance(value, str):
            return value.lower() in ("true", "yes", "1", "y")
        return bool(value)
    elif target_type == int:
        if isinstance(value, float):
            return int(value)
        return int(value)
    elif target_type == float:
        return float(value)
    elif target_type == str:
        return str(value)
    elif target_type == list:
        if isinstance(value, str):
            return json.loads(value)
        return list(value)
    elif target_type == dict:
        if isinstance(value, str):
            return json.loads(value)
        return dict(value)
    return value

print(convert_to_type("true", bool))      # True
print(convert_to_type("42.5", int))      # 42
print(convert_to_type("[1,2,3]", list))  # [1, 2, 3]
```

## Real-World Use Cases

- **User Input Processing**: Converting form submissions from strings to proper types
- **Data Parsing**: Converting CSV/JSON strings to Python data structures
- **Database Operations**: Converting between Python types and SQL types
- **API Development**: Serializing/deserializing request/response data
- **Configuration Files**: Converting config values to appropriate types
- **Data Cleaning**: Normalizing data types in datasets
- **Number System Conversions**: Working with binary, hex, octal representations

## Common Mistakes

```python
# Mistake 1: Converting without checking
# int("abc")  # ValueError: invalid literal for int()

# Mistake 2: Assuming int() truncates towards zero
print(int(3.9))    # 3
print(int(-3.9))   # -3 (truncates toward zero, not floor)

# Mistake 3: str() vs repr()
import datetime
now = datetime.datetime(2024, 1, 15)
print(str(now))   # '2024-01-15 00:00:00'
print(repr(now))  # 'datetime.datetime(2024, 1, 15, 0, 0)'

# Mistake 4: Trying to convert incompatible types
# dict([1, 2, 3])  # ValueError: dictionary update sequence element

# Mistake 5: Forgetting that bool('False') is True
print(bool("False"))  # True! (non-empty string)
# Correct:
print("False" == "true")  # Compare strings, or:
def str_to_bool(s):
    return s.lower() in ("true", "yes", "1")

# Mistake 6: Implicit string concatenation with +
result = "Count: " + 42  # TypeError
# Correct:
result = "Count: " + str(42)

# Mistake 7: int vs float conversion precision
print(int(0.9999999999999999))  # 0 (floating point precision)
print(int(0.99999999999999999))  # May print 1!
```

## Best Practices

- Always validate input before conversion, use try/except
- Use f-strings for automatic string conversion
- Prefer `int(x)` and `float(x)` for numeric conversion from strings
- Use `isinstance()` for type checking before conversion
- Use `dict()` with iterables of pairs
- Use `list()` and `tuple()` for sequence conversion
- Use `set()` for deduplication via conversion
- Handle `None` values explicitly in conversion functions
- Use `bin()`, `oct()`, `hex()` for base conversions
- Use `json.loads()`/`json.dumps()` for structured data
- Be aware of boolean conversion quirks (especially with strings)

## Interview Questions

1. What is the difference between implicit and explicit type conversion?
2. How do you convert a string to an integer safely?
3. What happens when you convert a float to an int?
4. How do you convert between different number bases?
5. Why is `bool("False")` == True?
6. How do you convert a list of tuples to a dictionary?
7. What is the difference between `str()` and `repr()`?
8. How do you handle type conversion in user input?
9. What are the limitations of `int()` conversion?
10. How do you convert between collection types?

## Coding Challenges

```python
# Challenge 1: Safe number conversion
def safe_convert(value, target_type, default=None):
    try:
        return target_type(value)
    except (ValueError, TypeError):
        return default

print(safe_convert("42", int))         # 42
print(safe_convert("abc", int))        # None
print(safe_convert("3.14", float))     # 3.14
print(safe_convert("true", bool))      # True (bool("true") is True)

# Challenge 2: CSV string to list of dicts
def csv_to_dicts(csv_string):
    lines = csv_string.strip().split("\n")
    if not lines:
        return []
    headers = lines[0].split(",")
    result = []
    for line in lines[1:]:
        values = line.split(",")
        row = {}
        for h, v in zip(headers, values):
            # Try to convert to numeric
            try:
                if "." in v:
                    row[h] = float(v)
                else:
                    row[h] = int(v)
            except ValueError:
                row[h] = v
        result.append(row)
    return result

csv = "name,age,score\nAlice,30,95.5\nBob,25,87.3\nCharlie,35,92.8"
data = csv_to_dicts(csv)
for d in data:
    print(f"{d['name']}: {d['age']} years, score {d['score']}")

# Challenge 3: Universal type converter
def universal_convert(value, target_type_str):
    type_map = {
        "int": int,
        "float": float,
        "str": str,
        "bool": bool,
        "list": lambda x: list(x) if not isinstance(x, str) else [x],
        "tuple": tuple,
        "set": set,
    }
    converter = type_map.get(target_type_str.lower())
    if converter:
        try:
            return converter(value)
        except (ValueError, TypeError):
            return None
    return None

print(universal_convert("42", "int"))       # 42
print(universal_convert("hello", "list"))   # ['hello']

# Challenge 4: Binary string to integer converter
def binary_to_int(binary_str):
    return int(binary_str, 2)

def int_to_binary(num):
    return bin(num)[2:]  # Remove '0b' prefix

print(binary_to_int("1010"))        # 10
print(int_to_binary(10))            # '1010'
print(binary_to_int("11111111"))    # 255

# Challenge 5: Data type normalizer
def normalize_types(data):
    """Convert all string numbers in a list to actual numbers."""
    result = []
    for item in data:
        if isinstance(item, str):
            # Try int first, then float
            try:
                result.append(int(item))
            except ValueError:
                try:
                    result.append(float(item))
                except ValueError:
                    # Check for boolean strings
                    if item.lower() in ("true", "false"):
                        result.append(item.lower() == "true")
                    else:
                        result.append(item)
        else:
            result.append(item)
    return result

mixed = ["42", "3.14", "hello", "true", 100, "false", None]
normalized = normalize_types(mixed)
for orig, norm in zip(mixed, normalized):
    print(f"  {orig!r:10} -> {norm!r:10} ({type(norm).__name__})")
```

## Summary

Type casting converts values between data types using built-in functions like `int()`, `float()`, `str()`, `list()`, `tuple()`, `set()`, and `dict()`. Python performs implicit conversion automatically in some contexts, but explicit conversion is often needed, especially for user input. Proper error handling during conversion prevents runtime errors.

## Related Topics

- Data Types
- User Input and Output
- Numbers and Math
- String Operations
- Collections (List, Tuple, Set, Dict)
- Error Handling (try/except)
- JSON Serialization
