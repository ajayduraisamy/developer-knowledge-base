# Variables - Assigning values with =, dynamic typing, naming rules

## Introduction

Variables are names that refer to values stored in memory. In Python, variables are dynamically typed, meaning you don't need to declare their type explicitly — Python infers the type from the value assigned. This makes Python flexible and easy to use, but also requires attention to naming conventions and potential pitfalls.

Unlike many other languages, Python variables are actually references (or labels) attached to objects in memory. When you assign a value to a variable, you're creating a reference to an object.

## Why It Is Important

Understanding variables is fundamental because:
- They store and manage data throughout your program
- Dynamic typing makes Python quick to write but requires awareness
- Proper naming conventions improve code readability
- Understanding references vs values prevents bugs
- Type hints (introduced in Python 3.5+) help catch type-related issues early

## Syntax

```python
# Basic variable assignment
variable_name = value

# Multiple assignment
a = b = c = 0

# Tuple unpacking
x, y, z = 1, 2, 3

# Type hints (Python 3.5+)
name: str = "Alice"
age: int = 30
is_student: bool = False
scores: list[int] = [85, 92, 78]
```

## Examples

### Variable Assignment and Reassignment

```python
# Assigning values to variables
name = "Alice"
age = 25
height = 5.7
is_student = True

print(name, age, height, is_student)

# Reassigning variables
age = 26  # Variable now points to a new value
print(age)

# Dynamic typing - variable can change type
value = 42
print(f"value is {type(value)}")  # <class 'int'>
value = "forty-two"
print(f"value is {type(value)}")  # <class 'str'>
value = [4, 2]
print(f"value is {type(value)}")  # <class 'list'>
```

### Multiple Assignment

```python
# Assigning the same value to multiple variables
a = b = c = 100
print(a, b, c)  # 100 100 100

# Tuple unpacking
x, y, z = 10, 20, 30
print(x, y, z)  # 10 20 30

# Swapping variables (Pythonic way)
a, b = 5, 10
print(f"Before swap: a={a}, b={b}")
a, b = b, a
print(f"After swap: a={a}, b={b}")

# Unpacking with *
first, *middle, last = [1, 2, 3, 4, 5]
print(first)    # 1
print(middle)   # [2, 3, 4]
print(last)     # 5
```

### Constants Convention

```python
# Python doesn't have true constants, but by convention
# UPPERCASE names indicate constants
PI = 3.14159
MAX_CONNECTIONS = 100
DEFAULT_TIMEOUT = 30
API_KEY = "your-api-key-here"

print(PI, MAX_CONNECTIONS, DEFAULT_TIMEOUT)
```

## Beginner Examples

```python
# Variables for storing personal information
first_name = "John"
last_name = "Doe"
full_name = first_name + " " + last_name
age = 30
city = "New York"
occupation = "Engineer"

print(f"Name: {full_name}")
print(f"Age: {age}")
print(f"City: {city}")
print(f"Occupation: {occupation}")

# Simple calculator using variables
num1 = 15
num2 = 4
sum_result = num1 + num2
difference = num1 - num2
product = num1 * num2
quotient = num1 / num2
remainder = num1 % num2
power = num1 ** num2

print(f"Sum: {sum_result}")
print(f"Difference: {difference}")
print(f"Product: {product}")
print(f"Quotient: {quotient:.2f}")
print(f"Remainder: {remainder}")
print(f"Power: {power}")

# String variables
greeting = "Hello"
subject = "World"
message = f"{greeting}, {subject}!"
print(message)

# Boolean variables
is_raining = True
has_umbrella = False
should_go_out = not is_raining and has_umbrella
print(f"Should go out? {should_go_out}")
```

## Intermediate Examples

```python
# Variable references and mutability
# Immutable types (int, str, tuple)
x = 10
y = x  # y gets a copy of the reference
x = 20
print(f"x={x}, y={y}")  # x=20, y=10 (ints are immutable)

# Mutable types (list, dict, set)
list_a = [1, 2, 3]
list_b = list_a  # list_b references the same list
list_a.append(4)
print(f"list_a={list_a}")  # [1, 2, 3, 4]
print(f"list_b={list_b}")  # [1, 2, 3, 4] (same object modified)

# To copy a list (not just reference)
list_c = list_a.copy()
list_a.append(5)
print(f"list_a={list_a}")  # [1, 2, 3, 4, 5]
print(f"list_c={list_c}")  # [1, 2, 3, 4] (separate copy)

# Checking variable identity
a = [1, 2, 3]
b = [1, 2, 3]
c = a

print(a == b)   # True (same value)
print(a is b)   # False (different objects)
print(a is c)   # True (same object)

# Variable scope example
def demonstrate_scope():
    local_var = "I am local"
    print(local_var)

demonstrate_scope()
# print(local_var)  # NameError: name 'local_var' is not defined

# Global vs local
global_var = "I am global"

def test_scope():
    global_var = "I am local version"
    print(global_var)  # Local version

test_scope()
print(global_var)  # Global version (unchanged)
```

## Advanced Examples

```python
# Type hints with variables
from typing import Optional, Union, List, Dict, Tuple

# Basic type hints
name: str = "Alice"
age: int = 30
height: float = 5.7
is_active: bool = True

# Complex type hints
scores: List[int] = [85, 92, 78]
person: Dict[str, Union[str, int]] = {"name": "Bob", "age": 25}
coordinates: Tuple[float, float] = (40.7128, -74.0060)
maybe_value: Optional[int] = None

# Type hints don't enforce types at runtime
age: int = "thirty"  # This works (no runtime checking)

# But tools like mypy can catch these
# mypy will flag: error: Incompatible types in assignment

# Variable annotations with class-level constants
from dataclasses import dataclass
from typing import ClassVar

@dataclass
class Config:
    """Application configuration."""
    DEBUG: ClassVar[bool] = False
    VERSION: ClassVar[str] = "1.0.0"
    
    host: str = "localhost"
    port: int = 8080

config = Config()
print(config.host, config.port)

# Descriptors and properties
class Temperature:
    def __init__(self):
        self._celsius = 0
    
    @property
    def celsius(self):
        return self._celsius
    
    @celsius.setter
    def celsius(self, value):
        if value < -273.15:
            raise ValueError("Temperature below absolute zero")
        self._celsius = value
    
    @property
    def fahrenheit(self):
        return self._celsius * 9/5 + 32

temp = Temperature()
temp.celsius = 25
print(f"{temp.celsius}°C = {temp.fahrenheit}°F")
# temp.celsius = -300  # ValueError!

# Variable number of arguments
def calculate_average(*numbers: float) -> float:
    """Calculate average of variable number of arguments."""
    if not numbers:
        return 0.0
    return sum(numbers) / len(numbers)

print(calculate_average(1, 2, 3, 4, 5))  # 3.0
print(calculate_average(10, 20))  # 15.0

# Packing and unpacking dictionaries
def create_profile(**kwargs) -> Dict[str, str]:
    """Create a profile from keyword arguments."""
    return {k: str(v) for k, v in kwargs.items()}

profile = create_profile(name="Alice", age=30, city="NYC")
print(profile)

# Variable scope and closures
def make_counter():
    count = 0
    def counter():
        nonlocal count
        count += 1
        return count
    return counter

counter_a = make_counter()
counter_b = make_counter()

print(counter_a())  # 1
print(counter_a())  # 2
print(counter_b())  # 1 (separate count)
```

## Real-World Use Cases

- **Configuration Management**: Storing application settings in variables
- **Data Processing**: Storing intermediate results during computation
- **State Management**: Tracking application state with boolean flags
- **API Integration**: Storing API keys, endpoints, and responses
- **Database Operations**: Holding query results and connection parameters
- **User Input Handling**: Storing form data and user preferences

## Common Mistakes

```python
# Mistake 1: Using undefined variables
# print(undefined_var)  # NameError: name 'undefined_var' is not defined

# Mistake 2: Confusing assignment (=) with comparison (==)
if x = 5:  # SyntaxError: invalid syntax
    pass

# Correct:
x = 5
if x == 5:
    print("x is 5")

# Mistake 3: Mutable default arguments
def add_item(item, items=[]):  # Wrong! Default list is mutable
    items.append(item)
    return items

print(add_item(1))  # [1]
print(add_item(2))  # [1, 2]  # Unexpected!

# Correct way:
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items

# Mistake 4: Variable shadowing
max = 10  # Shadows built-in max() function
print(max([1, 2, 3]))  # TypeError: 'int' object is not callable

# Mistake 5: Assuming variable scope extends into loops
for i in range(5):
    pass
print(i)  # 4 (i is still accessible! Python doesn't have block scope)

# Mistake 6: Unintentional variable reuse
count = 10
# ... many lines later ...
count = [1, 2, 3]  # Overwrote the original meaning
```

## Best Practices

- Use descriptive, meaningful variable names: `user_age` instead of `u` or `x`
- Follow snake_case convention: `first_name`, `total_count`
- Use UPPERCASE for constants: `MAX_RETRIES = 3`
- Avoid single-character names except in loops: `for i in range(10)` is acceptable
- Don't shadow built-in names (`list`, `str`, `dict`, `max`, `min`)
- Add type hints for public interfaces and complex types
- Use `_` for throwaway variables: `for _ in range(10)`
- Keep variables close to where they're used
- Use `None` to represent "no value" rather than placeholder strings
- Declare variables as close to their first use as possible

## Interview Questions

1. How does Python handle variable assignment (pass-by-value vs pass-by-reference)?
2. What is the difference between mutable and immutable types?
3. Explain how variable scoping works in Python (LEGB rule).
4. What are type hints and how do they work?
5. How do you create constants in Python?
6. Explain tuple unpacking with examples.
7. What is variable shadowing and why should you avoid it?
8. How do you swap two variables in Python?
9. Explain the concept of "everything is an object" in Python.
10. What is the difference between `is` and `==` for variable comparison?

## Coding Challenges

1. Write a program that swaps two variables without using a temporary variable.
2. Create a function that accepts any number of arguments and returns their sum.
3. Write a program that demonstrates variable mutability with lists.
4. Implement a simple counter using closures and nonlocal variables.
5. Create a program that demonstrates tuple unpacking with the * operator.

```python
# Challenge 1: Swap without temp
def swap(a, b):
    a, b = b, a
    return a, b

x, y = 5, 10
x, y = swap(x, y)
print(f"x={x}, y={y}")  # x=10, y=5

# Challenge 2: Variable arguments sum
def sum_all(*args):
    return sum(args)

print(sum_all(1, 2, 3, 4, 5))  # 15
print(sum_all(10, 20, 30))     # 60

# Challenge 3: Mutability demonstration
def modify_list(lst):
    lst.append(99)
    lst[0] = 0

original = [1, 2, 3]
modify_list(original)
print(original)  # [0, 2, 3, 99]

def modify_int(n):
    n = 100

value = 50
modify_int(value)
print(value)  # 50 (unchanged - int is immutable)

# Challenge 4: Counter with closure
def create_counter():
    count = 0
    def increment():
        nonlocal count
        count += 1
        return count
    return increment

counter = create_counter()
print(counter())  # 1
print(counter())  # 2
print(counter())  # 3

# Challenge 5: Extended unpacking
def process_items(items):
    first, *middle, last = items
    print(f"First: {first}")
    print(f"Middle: {middle}")
    print(f"Last: {last}")

process_items([1, 2, 3, 4, 5])
process_items(["apple", "banana", "cherry", "date"])
```

## Summary

Variables in Python are dynamic references to objects in memory, not fixed memory locations as in statically-typed languages. They follow snake_case naming conventions, support multiple assignment and tuple unpacking, and can include type hints for better code clarity. Understanding the distinction between mutable and immutable types, variable scope, and proper naming conventions is essential for writing clean, bug-free Python code.

## Related Topics

- Python Data Types
- Python Operators
- Python Scope Rules (LEGB)
- Type Hints and Annotations
- Functions and Arguments
- Mutable vs Immutable Objects
