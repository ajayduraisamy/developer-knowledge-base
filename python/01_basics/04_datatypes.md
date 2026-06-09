# Data Types - int, float, complex, str, bool, NoneType

## Introduction

Python has several built-in data types that define the nature of values stored in variables. The main categories include numeric types (int, float, complex), boolean (bool), NoneType, sequences (str, list, tuple), mappings (dict), and sets (set, frozenset). Python automatically determines the type of a variable based on its value, and provides functions to check and convert between types.

## Why It Is Important

Understanding data types is crucial because:
- Each type supports different operations and methods
- Memory usage varies by type
- Type determines mutability (can the value change?)
- Type conversion (casting) is often needed for operations
- Type checking helps prevent bugs
- Performance characteristics differ between types

## Syntax

```python
# Checking types
type(value)           # Returns the type of the value
isinstance(value, type)  # Checks if value is an instance of type

# Type conversion
int(value)    # Convert to integer
float(value)  # Convert to float
str(value)    # Convert to string
bool(value)   # Convert to boolean
list(value)   # Convert to list
tuple(value)  # Convert to tuple
set(value)    # Convert to set
```

## Examples

### Numeric Types

```python
# Integers (int) - arbitrary precision
a = 42
b = -100
c = 0
d = 1_000_000  # Underscores improve readability
large = 2 ** 100  # Arbitrary precision!

print(f"a = {a}, type = {type(a)}")
print(f"large = {large}")
print(f"Integer size: {large.bit_length()} bits")

# Floating point (float)
pi = 3.14159
e = 2.71828
scientific = 1.5e-10  # 1.5 × 10⁻¹⁰
infinity = float('inf')
not_a_number = float('nan')

print(f"pi = {pi}, type = {type(pi)}")
print(f"scientific = {scientific}")
print(f"infinity = {infinity}")
print(f"is nan? {not_a_number != not_a_number}")  # NaN != NaN is True

# Complex numbers (complex)
z1 = 3 + 4j
z2 = complex(1, -2)
print(f"z1 = {z1}, real = {z1.real}, imag = {z1.imag}")
print(f"z1 conjugate = {z1.conjugate()}")
print(f"z1 magnitude = {abs(z1)}")
```

### Boolean Type

```python
# Boolean values
is_active = True
is_complete = False

# Booleans are actually integers (True=1, False=0)
print(f"True + True = {True + True}")  # 2
print(f"True * 10 = {True * 10}")     # 10
print(f"False + 5 = {False + 5}")     # 5

# Boolean conversion
print(f"bool(1) = {bool(1)}")
print(f"bool(0) = {bool(0)}")
print(f"bool('hello') = {bool('hello')}")
print(f"bool('') = {bool('')}")
print(f"bool([]) = {bool([])}")
print(f"bool([1, 2]) = {bool([1, 2])}")
print(f"bool(None) = {bool(None)}")
```

### NoneType

```python
# None represents the absence of a value
result = None
print(f"result = {result}, type = {type(result)}")

# Common uses of None
def find_user(user_id):
    if user_id == 1:
        return "Alice"
    return None  # User not found

user = find_user(42)
if user is None:
    print("User not found")

# Checking for None (always use 'is', not '==')
x = None
print(f"x is None: {x is None}")
print(f"x == None: {x == None}")  # Works but not idiomatic
```

## Beginner Examples

```python
# Exploring data types
name = "Alice"        # str
age = 30              # int
height = 5.7          # float
is_student = False    # bool
favorites = None      # NoneType

print(f"{name} is type {type(name)}")
print(f"{age} is type {type(age)}")
print(f"{height} is type {type(height)}")
print(f"{is_student} is type {type(is_student)}")
print(f"{favorites} is type {type(favorites)}")

# Type checking with isinstance
print(f"Is {name} a string? {isinstance(name, str)}")
print(f"Is {age} an integer? {isinstance(age, int)}")
print(f"Is {height} a float? {isinstance(height, float)}")
print(f"Is {is_student} a boolean? {isinstance(is_student, bool)}")

# Checking multiple types at once
print(f"Is {age} int or float? {isinstance(age, (int, float))}")

# Numeric operations with different types
int_val = 10
float_val = 3.14
result = int_val + float_val  # Implicit conversion: int -> float
print(f"{int_val} + {float_val} = {result} (type: {type(result)})")

# Division always returns float
print(f"10 / 3 = {10 / 3} (type: {type(10 / 3)})")
print(f"10 // 3 = {10 // 3} (type: {type(10 // 3)})")
```

## Intermediate Examples

```python
# Type hierarchy and isinstance checks
numbers = [42, 3.14, 2+3j, True, None, "hello", [1,2], (1,2), {"a": 1}]

for item in numbers:
    if isinstance(item, int):
        print(f"{item} is an integer")
    elif isinstance(item, float):
        print(f"{item} is a float")
    elif isinstance(item, complex):
        print(f"{item} is a complex number")
    elif isinstance(item, bool):
        print(f"{item} is a boolean")
    elif isinstance(item, str):
        print(f"{item} is a string")
    elif item is None:
        print(f"{item} is None")
    else:
        print(f"{item} is {type(item)}")

# Numeric type conversions
from decimal import Decimal, getcontext
from fractions import Fraction

# Decimal for precise arithmetic
getcontext().prec = 28
d1 = Decimal('0.1')
d2 = Decimal('0.2')
print(f"Decimal: {d1} + {d2} = {d1 + d2}")

# Compare with float
f1 = 0.1
f2 = 0.2
print(f"Float: {f1} + {f2} = {f1 + f2}")  # 0.30000000000000004

# Fractions
frac = Fraction(3, 4)
print(f"Fraction: {frac} = {float(frac)}")

# Type coercion
print(f"1 + 2.0 = {1 + 2.0}")        # float
print(f"True + 1 = {True + 1}")      # int (True=1)
print(f"False * 5 = {False * 5}")    # int (False=0)
print(f"1 + 2j + 3 = {1 + 2j + 3}")  # complex

# hashable types
print(f"hash(42) = {hash(42)}")
print(f"hash('hello') = {hash('hello')}")
print(f"hash((1, 2, 3)) = {hash((1, 2, 3))}")
# hash([1, 2, 3])  # TypeError: unhashable type: 'list'
```

## Advanced Examples

```python
# Abstract Base Classes for type checking
from collections.abc import Iterable, Sequence, MutableSequence

def process_items(items):
    if isinstance(items, MutableSequence):
        items.append("processed")
        return items
    elif isinstance(items, Sequence):
        return list(items) + ["processed"]
    elif isinstance(items, Iterable):
        return list(items) + ["processed"]
    else:
        raise TypeError("Items must be iterable")

print(process_items([1, 2, 3]))       # MutableSequence
print(process_items((1, 2, 3)))       # Sequence (immutable)
print(process_items({1, 2, 3}))       # Iterable (set)

# Custom type checking with type hints
from typing import TypeVar, Generic, Union

Number = Union[int, float]
T = TypeVar('T')

def double(value: Number) -> Number:
    return value * 2

print(double(5))     # 10
print(double(3.5))   # 7.0

# Numeric tower in Python
import numbers

def is_real_number(x):
    return isinstance(x, numbers.Real)

print(f"is_real_number(42): {is_real_number(42)}")
print(f"is_real_number(3.14): {is_real_number(3.14)}")
print(f"is_real_number(complex(1,2)): {is_real_number(complex(1,2))}")
print(f"is_real_number('hello'): {is_real_number('hello')}")

# sys.getsizeof for memory information
import sys

print(f"int size: {sys.getsizeof(42)} bytes")
print(f"float size: {sys.getsizeof(3.14)} bytes")
print(f"bool size: {sys.getsizeof(True)} bytes")
print(f"None size: {sys.getsizeof(None)} bytes")
print(f"string size: {sys.getsizeof('hello')} bytes")
print(f"small list size: {sys.getsizeof([1, 2, 3])} bytes")
```

## Real-World Use Cases

- **Financial Calculations**: Use Decimal instead of float for monetary values
- **Scientific Computing**: Complex numbers for physics/engineering calculations
- **Configuration Flags**: Boolean values for feature toggles
- **API Responses**: None to represent missing or null values
- **Data Validation**: isinstance() for runtime type checking
- **Machine Learning**: Converting between NumPy arrays and Python types

## Common Mistakes

```python
# Mistake 1: Float precision issues
print(0.1 + 0.2 == 0.3)  # False! (0.30000000000000004)
# Fix: Use math.isclose() or Decimal

import math
print(math.isclose(0.1 + 0.2, 0.3))  # True

# Mistake 2: Using is for value comparison
a = 256
b = 256
print(a is b)  # True (Python caches small integers)

c = 257
d = 257
print(c is d)  # False! (different objects)
print(c == d)  # True (correct comparison)

# Mistake 3: Not handling None properly
def get_data():
    return None

data = get_data()
# if data:  # This would be False for None, but also for 0, "", []
# Better:
if data is None:
    print("No data available")

# Mistake 4: Confusing type() with isinstance()
# type() doesn't account for inheritance
class MyInt(int):
    pass

x = MyInt(5)
print(type(x) is int)     # False (type is MyInt)
print(isinstance(x, int)) # True (MyInt is a subclass of int)

# Mistake 5: Assuming bool is not a subclass of int
print(isinstance(True, int))  # True!
print(True + True)  # 2
```

## Best Practices

- Use `isinstance()` instead of `type()` for type checking (supports inheritance)
- Use `is` for comparing with `None` (`x is None` not `x == None`)
- Prefer `Decimal` for financial/monetary calculations
- Be aware of float precision limitations
- Use `math.isclose()` for float comparisons
- Use type hints to document expected types
- Use `bool()` for truthiness checks
- Use `numbers.Number` ABCs for abstract numeric type checks
- Avoid mixing types in operations without explicit conversion
- Use underscores in large numeric literals: `1_000_000`

## Interview Questions

1. What are Python's built-in data types?
2. What is the difference between mutable and immutable types?
3. How does Python handle type conversion implicitly?
4. What is the difference between `type()` and `isinstance()`?
5. Explain the numeric type hierarchy in Python.
6. Why is `0.1 + 0.2 != 0.3` in Python?
7. What is None in Python and how do you check for it?
8. Are booleans a subtype of integers in Python?
9. How do you check if a variable is a number in Python?
10. What are the performance implications of different data types?

## Coding Challenges

1. Write a function that returns the type of a value as a string.
2. Create a function that safely converts a value to an integer.
3. Write a program that demonstrates float precision issues.
4. Create a function that categorizes values by their type.
5. Write a program that explores memory usage of different types.

```python
# Challenge 1: Type Name Function
def get_type_name(value):
    return type(value).__name__

print(get_type_name(42))       # 'int'
print(get_type_name("hello"))  # 'str'
print(get_type_name([1, 2]))   # 'list'

# Challenge 2: Safe Integer Conversion
def safe_int(value, default=0):
    try:
        return int(value)
    except (ValueError, TypeError):
        return default

print(safe_int("42"))       # 42
print(safe_int("abc"))      # 0 (default)
print(safe_int(None, -1))   # -1

# Challenge 3: Float Precision Demo
def demonstrate_precision():
    print("Float precision issues:")
    total = 0.0
    for _ in range(10):
        total += 0.1
    print(f"0.1 * 10 = {total}")
    print(f"Expected: 1.0")
    print(f"Equal to 1.0? {total == 1.0}")
    print(f"Close to 1.0? {math.isclose(total, 1.0)}")

import math
demonstrate_precision()

# Challenge 4: Type Categorizer
def categorize_values(values):
    categories = {}
    for value in values:
        type_name = type(value).__name__
        if type_name not in categories:
            categories[type_name] = []
        categories[type_name].append(value)
    return categories

values = [42, 3.14, "hello", True, None, [1, 2], (3, 4), {"a": 1}]
for type_name, items in categorize_values(values).items():
    print(f"{type_name}: {items}")

# Challenge 5: Memory Usage Explorer
def explore_memory():
    import sys
    examples = [
        42,
        3.141592653589793,
        True,
        None,
        "hello",
        "a" * 1000,
        [1, 2, 3],
        list(range(1000)),
        {"a": 1, "b": 2, "c": 3},
    ]
    
    print(f"{'Type':<20} {'Size (bytes)':<15} {'Value'}")
    print("-" * 60)
    for item in examples:
        print(f"{type(item).__name__:<20} {sys.getsizeof(item):<15} {str(item)[:30]}")

explore_memory()
```

## Summary

Python provides several built-in data types including numeric types (int, float, complex), booleans, NoneType, and collection types. Each type has specific characteristics regarding mutability, memory usage, and operations. Understanding these types, how to check them with `type()` and `isinstance()`, and how to convert between them is essential for effective Python programming. Special attention should be paid to float precision limitations and proper None checking.

## Related Topics

- Python Variables
- Type Casting and Conversion
- Numbers and Math Operations
- Strings
- Lists, Tuples, Dicts, and Sets
- Type Hints and Annotations
- Abstract Base Classes
