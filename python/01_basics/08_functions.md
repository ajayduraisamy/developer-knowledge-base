# Functions - def, return, parameters, *args, **kwargs

## Introduction

Functions are reusable blocks of code that perform a specific task. They help organize code, reduce repetition, and make programs more modular and maintainable. Python functions are defined using the `def` keyword, can accept parameters and return values, and support advanced features like default arguments, keyword arguments, variable-length arguments (`*args`, `**kwargs`), docstrings, and type annotations.

## Why It Is Important

Functions are fundamental to programming because:
- They promote code reusability (DRY principle)
- They break complex problems into smaller, manageable pieces
- They improve code readability and organization
- They allow for abstraction (hiding implementation details)
- They enable testing individual components
- They support different argument passing styles for flexibility

## Syntax

```python
# Basic function
def function_name(parameters):
    """Docstring describing the function."""
    # Function body
    return value

# Function with default arguments
def function_name(param1, param2=default_value):
    return value

# Function with *args (variable positional arguments)
def function_name(*args):
    for arg in args:
        print(arg)

# Function with **kwargs (variable keyword arguments)
def function_name(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")

# Function with type annotations
def function_name(param1: str, param2: int) -> str:
    return f"{param1} {param2}"
```

## Examples

### Basic Functions

```python
# Simple function without parameters
def greet():
    """Print a greeting message."""
    print("Hello, World!")

greet()

# Function with parameters
def greet_person(name):
    """Greet a person by name."""
    print(f"Hello, {name}!")

greet_person("Alice")

# Function with return value
def add(a, b):
    """Return the sum of two numbers."""
    return a + b

result = add(5, 3)
print(f"5 + 3 = {result}")

# Function with multiple returns
def min_max(numbers):
    """Return both minimum and maximum of a list."""
    return min(numbers), max(numbers)

minimum, maximum = min_max([3, 1, 7, 2, 9, 4])
print(f"Min: {minimum}, Max: {maximum}")
```

### Default and Keyword Arguments

```python
# Default arguments
def power(base, exponent=2):
    """Raise base to exponent (default is square)."""
    return base ** exponent

print(power(5))        # 25 (5^2)
print(power(5, 3))     # 125 (5^3)

# Keyword arguments (order doesn't matter)
def create_profile(name, age, city="Unknown"):
    """Create a user profile string."""
    return f"{name}, {age} years old, from {city}"

print(create_profile("Alice", 30, "New York"))
print(create_profile(city="London", name="Bob", age=25))

# Mixing positional and keyword arguments
print(create_profile("Charlie", 35))  # Uses default city
```

## Beginner Examples

```python
# Temperature converter
def celsius_to_fahrenheit(celsius):
    """Convert Celsius to Fahrenheit."""
    return (celsius * 9/5) + 32

def fahrenheit_to_celsius(fahrenheit):
    """Convert Fahrenheit to Celsius."""
    return (fahrenheit - 32) * 5/9

print(f"25°C = {celsius_to_fahrenheit(25):.1f}°F")
print(f"77°F = {fahrenheit_to_celsius(77):.1f}°C")

# Area calculator
def circle_area(radius):
    """Calculate the area of a circle."""
    import math
    return math.pi * radius ** 2

def rectangle_area(length, width):
    """Calculate the area of a rectangle."""
    return length * width

def triangle_area(base, height):
    """Calculate the area of a triangle."""
    return 0.5 * base * height

print(f"Circle (r=5): {circle_area(5):.2f}")
print(f"Rectangle (4x6): {rectangle_area(4, 6)}")
print(f"Triangle (3x8): {triangle_area(3, 8)}")

# Simple calculator using functions
def calculate(num1, num2, operation):
    """Perform arithmetic operations."""
    if operation == "add":
        return num1 + num2
    elif operation == "subtract":
        return num1 - num2
    elif operation == "multiply":
        return num1 * num2
    elif operation == "divide":
        if num2 == 0:
            return "Error: Division by zero"
        return num1 / num2
    else:
        return "Invalid operation"

print(calculate(10, 5, "add"))      # 15
print(calculate(10, 5, "divide"))   # 2.0
```

## Intermediate Examples

```python
# *args - Variable positional arguments
def sum_all(*numbers):
    """Sum any number of arguments."""
    print(f"Received {len(numbers)} numbers: {numbers}")
    return sum(numbers)

print(sum_all(1, 2, 3))        # 6
print(sum_all(10, 20, 30, 40)) # 100

# **kwargs - Variable keyword arguments
def print_info(**info):
    """Print key-value pairs."""
    print("User Information:")
    for key, value in info.items():
        print(f"  {key}: {value}")

print_info(name="Alice", age=30, city="NYC", job="Engineer")

# Mixing *args and **kwargs
def advanced_function(a, b, *args, **kwargs):
    """Function that uses all argument types."""
    print(f"Positional: a={a}, b={b}")
    print(f"Additional positional (*args): {args}")
    print(f"Keyword arguments (**kwargs): {kwargs}")

advanced_function(1, 2, 3, 4, 5, name="Test", value=100)

# Functions as first-class objects
def apply_operation(func, a, b):
    """Apply a function to two arguments."""
    return func(a, b)

print(apply_operation(lambda x, y: x + y, 10, 5))  # 15
print(apply_operation(lambda x, y: x * y, 10, 5))  # 50

# Returning multiple values (as tuple)
def get_stats(numbers):
    """Calculate multiple statistics."""
    return (
        sum(numbers) / len(numbers),  # mean
        min(numbers),                  # min
        max(numbers),                  # max
        sorted(numbers)[len(numbers)//2]  # median
    )

mean, min_val, max_val, median = get_stats([1, 3, 5, 7, 9, 11])
print(f"Mean: {mean}, Min: {min_val}, Max: {max_val}, Median: {median}")
```

## Advanced Examples

```python
# Decorators
def timer(func):
    """Decorator that measures function execution time."""
    import time
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        end = time.perf_counter()
        print(f"{func.__name__} took {end - start:.6f}s")
        return result
    return wrapper

@timer
def slow_sum(n):
    """Sum numbers from 0 to n."""
    total = 0
    for i in range(n):
        total += i
    return total

result = slow_sum(10_000_000)
print(f"Result: {result}")

# Decorator with arguments
def repeat(times):
    """Decorator that repeats a function multiple times."""
    def decorator(func):
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def say_hello(name):
    """Greet someone."""
    print(f"Hello, {name}!")

say_hello("Alice")

# Lambda functions
square = lambda x: x ** 2
print(f"Square of 7: {square(7)}")

# Lambda with map, filter, sorted
numbers = [3, 7, 1, 9, 2, 8, 4, 6]
even_numbers = list(filter(lambda x: x % 2 == 0, numbers))
squared_numbers = list(map(lambda x: x ** 2, numbers))
sorted_by_parity = sorted(numbers, key=lambda x: (x % 2, x))

print(f"Even numbers: {even_numbers}")
print(f"Squared: {squared_numbers}")
print(f"Sorted by parity: {sorted_by_parity}")

# Recursive function
def factorial(n):
    """Calculate factorial recursively."""
    if n <= 1:
        return 1
    return n * factorial(n - 1)

print(f"Factorial 5: {factorial(5)}")  # 120

# Recursive directory traversal (simplified)
def flatten_list(nested):
    """Flatten a nested list recursively."""
    result = []
    for item in nested:
        if isinstance(item, list):
            result.extend(flatten_list(item))
        else:
            result.append(item)
    return result

nested = [1, [2, [3, 4], 5], 6, [7, 8]]
print(f"Flattened: {flatten_list(nested)}")

# Closure (function with captured state)
def make_multiplier(factor):
    """Create a function that multiplies by a given factor."""
    def multiply(x):
        return x * factor
    return multiply

double = make_multiplier(2)
triple = make_multiplier(3)
print(f"Double 5: {double(5)}")   # 10
print(f"Triple 5: {triple(5)}")   # 15

# Partial function application
from functools import partial

def power(base, exponent):
    return base ** exponent

square = partial(power, exponent=2)
cube = partial(power, exponent=3)
print(f"Square of 9: {square(9)}")   # 81
print(f"Cube of 4: {cube(4)}")       # 64
```

## Real-World Use Cases

- **Data Processing**: Functions for cleaning, transforming, analyzing data
- **Web APIs**: View functions that handle HTTP requests
- **Configuration**: Factory functions that create configured objects
- **Utilities**: Reusable helper functions (date formatting, validation, etc.)
- **Testing**: Test functions that verify code correctness
- **Plugins**: Functions as extensibility points in applications
- **Callbacks**: Functions passed to event handlers, sort keys, etc.

## Common Mistakes

```python
# Mistake 1: Mutable default arguments
def add_item(item, items=[]):  # Wrong!
    items.append(item)
    return items

print(add_item(1))  # [1]
print(add_item(2))  # [1, 2] - unexpected!

# Correct:
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items

# Mistake 2: Modifying a list passed as argument
def process_list(items):
    items.append("extra")  # Modifies the original list!

my_list = [1, 2, 3]
process_list(my_list)
print(my_list)  # [1, 2, 3, 'extra']

# Mistake 3: Forgetting to return a value
def square(n):
    n * n  # Missing return statement

print(square(5))  # None

# Mistake 4: Using mutable types as default for *args/**kwargs patterns
# Always use None as default for mutable values

# Mistake 5: Returning too many values without unpacking
def get_user():
    return "Alice", 30, "NYC"

user = get_user()  # Returns a tuple
print(user)  # ('Alice', 30, 'NYC')

# Mistake 6: Function name shadowing built-ins
def list():  # Shadows built-in list()
    return "my list"

print(list([1, 2, 3]))  # TypeError!
```

## Best Practices

- Functions should do one thing (Single Responsibility Principle)
- Keep functions short (ideally under 30 lines)
- Use descriptive names: `calculate_average()` not `calc_avg()`
- Include docstrings for all public functions
- Use type annotations for clarity
- Avoid mutable default arguments
- Prefer returning values over printing inside functions
- Use `*args` and `**kwargs` sparingly (can hide function signature)
- Use keyword arguments for optional parameters
- Don't modify arguments that should be immutable
- Keep the number of parameters small (under 5)
- Use early returns to reduce nesting
- Write pure functions when possible (no side effects)

## Interview Questions

1. What is the difference between `return` and `print` in a function?
2. Explain `*args` and `**kwargs` with examples.
3. What are default arguments? What's the pitfall with mutable defaults?
4. What is a decorator and how do you create one?
5. Explain closures and how they work in Python.
6. What is a lambda function? When would you use it?
7. How does Python handle passing arguments (by value or reference)?
8. What is recursion? Give an example.
9. What are docstrings and how are they used?
10. Explain the difference between positional and keyword arguments.

## Coding Challenges

1. Write a function that checks if a string is a palindrome.
2. Create a function that returns the nth Fibonacci number using recursion.
3. Write a decorator that logs function calls.
4. Create a function that accepts any number of keyword arguments and returns only those with string values.
5. Write a function that computes the GCD of two numbers using Euclid's algorithm.

```python
# Challenge 1: Palindrome Checker
def is_palindrome(text):
    """Check if a string is a palindrome."""
    cleaned = ''.join(c.lower() for c in text if c.isalnum())
    return cleaned == cleaned[::-1]

test_strings = ["racecar", "A man, a plan, a canal: Panama", "hello"]
for s in test_strings:
    print(f"'{s}': {is_palindrome(s)}")

# Challenge 2: Recursive Fibonacci
def fibonacci(n, memo=None):
    """Return the nth Fibonacci number using memoization."""
    if memo is None:
        memo = {}
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    memo[n] = fibonacci(n-1, memo) + fibonacci(n-2, memo)
    return memo[n]

for i in range(15):
    print(f"Fibonacci({i}) = {fibonacci(i)}")

# Challenge 3: Logging Decorator
def log_calls(func):
    """Decorator that logs function calls and results."""
    from functools import wraps
    @wraps(func)
    def wrapper(*args, **kwargs):
        args_repr = [repr(a) for a in args]
        kwargs_repr = [f"{k}={v!r}" for k, v in kwargs.items()]
        signature = ", ".join(args_repr + kwargs_repr)
        print(f"Calling: {func.__name__}({signature})")
        result = func(*args, **kwargs)
        print(f"Result: {result!r}")
        return result
    return wrapper

@log_calls
def multiply(a, b):
    """Multiply two numbers."""
    return a * b

multiply(3, 7)
multiply(10, b=5)

# Challenge 4: Filter String Values from **kwargs
def get_string_values(**kwargs):
    """Return only the string values from keyword arguments."""
    return {k: v for k, v in kwargs.items() if isinstance(v, str)}

result = get_string_values(
    name="Alice",
    age=30,
    city="New York",
    score=95.5,
    job="Engineer"
)
print(f"\nString values: {result}")

# Challenge 5: GCD Function
def gcd(a, b):
    """Compute GCD using Euclid's algorithm."""
    while b:
        a, b = b, a % b
    return a

def lcm(a, b):
    """Compute LCM using GCD."""
    return a * b // gcd(a, b)

pairs = [(48, 18), (56, 98), (17, 23)]
for a, b in pairs:
    print(f"GCD({a}, {b}) = {gcd(a, b)}, LCM({a}, {b}) = {lcm(a, b)}")
```

## Summary

Functions are reusable code blocks defined with `def` that accept parameters and return values. Python supports flexible argument passing (positional, keyword, default, *args, **kwargs), first-class functions, lambda expressions, decorators, closures, and recursion. Well-designed functions improve code organization, reusability, and testability. Following best practices like the Single Responsibility Principle, proper naming, and docstrings leads to maintainable code.

## Related Topics

- Variable Scope (LEGB Rule)
- Lambda Functions
- Decorators
- Generators and Yield
- Recursion
- Functions as First-Class Objects
- functools Module
- Type Annotations and Hints
