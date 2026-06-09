# Operators - Arithmetic (+, -, *, /, //, %, **), comparison, logical

## Introduction

Operators are special symbols or keywords that perform operations on values (operands). Python provides a rich set of operators including arithmetic, comparison, logical, assignment, bitwise, identity, and membership operators. Each operator has a specific precedence that determines the order of evaluation in expressions.

## Why It Is Important

Understanding operators is fundamental because:
- They form the building blocks of expressions and computations
- Operator precedence determines how complex expressions are evaluated
- Different operators serve different purposes (math, logic, comparison, etc.)
- Understanding short-circuit evaluation can optimize performance
- Identity and membership operators have unique Python-specific behavior

## Syntax

```python
# All operator types
result = operand1 operator operand2  # Binary operator
result = operator operand            # Unary operator

# Arithmetic
x + y, x - y, x * y, x / y, x // y, x % y, x ** y

# Comparison
x == y, x != y, x < y, x > y, x <= y, x >= y

# Logical
x and y, x or y, not x

# Assignment
x = y, x += y, x -= y, x *= y, x /= y, x //= y, x %= y, x **= y
x &= y, x |= y, x ^= y, x <<= y, x >>= y

# Bitwise
x & y, x | y, x ^ y, ~x, x << y, x >> y

# Identity
x is y, x is not y

# Membership
x in y, x not in y
```

## Examples

### Arithmetic Operators

```python
# Basic arithmetic
a, b = 15, 4
print(f"a = {a}, b = {b}")
print(f"Addition: a + b = {a + b}")           # 19
print(f"Subtraction: a - b = {a - b}")        # 11
print(f"Multiplication: a * b = {a * b}")     # 60
print(f"Division: a / b = {a / b}")           # 3.75
print(f"Floor Division: a // b = {a // b}")   # 3
print(f"Modulus: a % b = {a % b}")            # 3
print(f"Exponentiation: a ** b = {a ** b}")   # 50625

# Special cases
print(f"Division by zero raises ZeroDivisionError")
# print(10 / 0)  # ZeroDivisionError

print(f"Negative modulo: -7 % 3 = {-7 % 3}")  # 2
print(f"Floor division negative: -7 // 3 = {-7 // 3}")  # -3
```

### Comparison Operators

```python
# All comparison operators return Boolean values
a, b = 10, 20
print(f"a = {a}, b = {b}")
print(f"a == b: {a == b}")   # False
print(f"a != b: {a != b}")   # True
print(f"a < b: {a < b}")     # True
print(f"a > b: {a > b}")     # False
print(f"a <= b: {a <= b}")   # True
print(f"a >= b: {a >= b}")   # False

# Chained comparisons (Python-specific)
x = 15
print(f"10 < x < 20: {10 < x < 20}")      # True
print(f"5 < x < 10: {5 < x < 10}")        # False
print(f"10 <= x <= 20: {10 <= x <= 20}")  # True

# String comparison (lexicographic)
print(f"'apple' < 'banana': {'apple' < 'banana'}")  # True
print(f"'Hello' < 'hello': {'Hello' < 'hello'}")    # True (ASCII comparison)

# List comparison
print(f"[1, 2, 3] < [1, 2, 4]: {[1, 2, 3] < [1, 2, 4]}")  # True
```

## Beginner Examples

```python
# Simple calculator
num1 = float(input("Enter first number: "))
num2 = float(input("Enter second number: "))

print(f"Sum: {num1 + num2}")
print(f"Difference: {num1 - num2}")
print(f"Product: {num1 * num2}")
print(f"Quotient: {num1 / num2}")
print(f"Remainder: {num1 % num2}")
print(f"Power: {num1 ** num2}")

# Even or odd check
number = int(input("Enter a number: "))
if number % 2 == 0:
    print(f"{number} is even")
else:
    print(f"{number} is odd")

# Age verification
age = int(input("Enter your age: "))
if age >= 18:
    print("You are an adult")
else:
    print("You are a minor")

# Logical operators
has_permission = True
is_logged_in = True
is_admin = False

if is_logged_in and has_permission:
    print("Access granted")

if is_admin or has_permission:
    print("Can view resource")

if not is_admin:
    print("Limited access")

# Assignment operators
x = 10
x += 5   # x = x + 5
print(f"x += 5: {x}")    # 15
x -= 3   # x = x - 3
print(f"x -= 3: {x}")    # 12
x *= 2   # x = x * 2
print(f"x *= 2: {x}")    # 24
x /= 4   # x = x / 4
print(f"x /= 4: {x}")    # 6.0
x //= 2  # x = x // 2
print(f"x //= 2: {x}")   # 3.0
x %= 2   # x = x % 2
print(f"x %= 2: {x}")    # 1.0
x **= 3  # x = x ** 3
print(f"x **= 3: {x}")   # 1.0
```

## Intermediate Examples

```python
# Identity operators (is, is not)
a = [1, 2, 3]
b = [1, 2, 3]
c = a

print(f"a is c: {a is c}")          # True (same object)
print(f"a is b: {a is b}")          # False (different objects)
print(f"a == b: {a == b}")          # True (same content)

# Special cases with integers
x = 256
y = 256
print(f"x is y (256): {x is y}")    # True (cached)

x = 257
y = 257
print(f"x is y (257): {x is y}")    # False (not cached)

# None checking (always use is)
value = None
if value is None:
    print("Value is None")
if value is not None:
    print("Value is not None")

# Membership operators
fruits = ["apple", "banana", "cherry"]
print(f"'banana' in fruits: {'banana' in fruits}")   # True
print(f"'grape' not in fruits: {'grape' not in fruits}")  # True

# String membership
text = "Hello, World!"
print(f"'World' in text: {'World' in text}")    # True
print(f"'Python' in text: {'Python' in text}")  # False

# Dict membership checks keys
person = {"name": "Alice", "age": 30}
print(f"'name' in person: {'name' in person}")     # True
print(f"30 in person: {30 in person}")              # False (checks keys)

# Short-circuit evaluation
def expensive_function():
    print("Expensive function called!")
    return True

# If first condition is True, second is not evaluated (OR)
result = False or expensive_function()  # Expensive function IS called
result = True or expensive_function()  # Expensive function NOT called

# If first condition is False, second is not evaluated (AND)
result = True and expensive_function()  # Expensive function IS called
result = False and expensive_function()  # Expensive function NOT called

# Practical short-circuit: avoid errors
items = None
# items.append(1)  # AttributeError: 'NoneType' object has no attribute 'append'

if items is not None and len(items) > 0:
    print("Items has elements")
else:
    print("Items is None or empty")
```

## Advanced Examples

```python
# Bitwise operators (working with binary representations)
a = 0b1100  # 12 in binary
b = 0b1010  # 10 in binary

print(f"a = {a:04b} ({a})")
print(f"b = {b:04b} ({b})")
print(f"a & b = {a & b:04b} ({a & b})")    # AND: 1000 (8)
print(f"a | b = {a | b:04b} ({a | b})")    # OR: 1110 (14)
print(f"a ^ b = {a ^ b:04b} ({a ^ b})")    # XOR: 0110 (6)
print(f"~a = {~a} (signed: {~a & 0b1111:04b})")  # NOT
print(f"a << 1 = {a << 1:04b} ({a << 1})")  # Left shift: 11000 (24)
print(f"a >> 1 = {a >> 1:04b} ({a >> 1})")  # Right shift: 0110 (6)

# Bit flags example
READ = 0b001
WRITE = 0b010
EXECUTE = 0b100

permissions = READ | WRITE  # 0b011 (read + write)
print(f"Can read: {bool(permissions & READ)}")
print(f"Can write: {bool(permissions & WRITE)}")
print(f"Can execute: {bool(permissions & EXECUTE)}")

# Checking if a number is power of 2
def is_power_of_two(n):
    return n > 0 and (n & (n - 1)) == 0

print(f"is_power_of_two(16): {is_power_of_two(16)}")  # True
print(f"is_power_of_two(18): {is_power_of_two(18)}")  # False

# Ternary operator (conditional expression)
age = 20
status = "Adult" if age >= 18 else "Minor"
print(status)

# Nested ternary
score = 85
grade = "A" if score >= 90 else "B" if score >= 80 else "C" if score >= 70 else "D" if score >= 60 else "F"
print(f"Score {score} = Grade {grade}")

# Operator overloading
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)
    
    def __sub__(self, other):
        return Vector(self.x - other.x, self.y - other.y)
    
    def __mul__(self, scalar):
        return Vector(self.x * scalar, self.y * scalar)
    
    def __eq__(self, other):
        return self.x == other.x and self.y == other.y
    
    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

v1 = Vector(2, 3)
v2 = Vector(1, 4)
print(f"v1 + v2 = {v1 + v2}")  # Vector(3, 7)
print(f"v1 - v2 = {v1 - v2}")  # Vector(1, -1)
print(f"v1 * 3 = {v1 * 3}")    # Vector(6, 9)
print(f"v1 == v2: {v1 == v2}")  # False

# Operator precedence
# 1. ** (exponentiation)
# 2. ~, +, - (unary)
# 3. *, /, //, %
# 4. +, - (binary)
# 5. <<, >>
# 6. &
# 7. ^
# 8. |
# 9. ==, !=, <, >, <=, >=, is, in
# 10. not
# 11. and
# 12. or

result = 2 + 3 * 4 ** 2  # 2 + 3 * 16 = 2 + 48 = 50
print(f"2 + 3 * 4 ** 2 = {result}")

# Use parentheses for clarity
result = (2 + 3) * (4 ** 2)  # 5 * 16 = 80
print(f"(2 + 3) * (4 ** 2) = {result}")
```

## Real-World Use Cases

- **E-commerce**: Price calculations with arithmetic operators
- **Access Control**: Bitwise operators for permission systems
- **Data Filtering**: Comparison and logical operators in queries
- **Input Validation**: Membership and comparison operators for checking inputs
- **Game Development**: Collision detection using comparison operators
- **Configuration Systems**: Logical operators for feature flags
- **Search Functionality**: Membership operators for text matching

## Common Mistakes

```python
# Mistake 1: Using = instead of ==
x = 10
# if x = 20:  # SyntaxError
#     pass

# Mistake 2: Operator precedence confusion
result = 2 + 3 * 4  # 14, not 20
print(f"2 + 3 * 4 = {result}")  # 14

# Mistake 3: Float comparison with ==
a = 0.1 + 0.2
b = 0.3
print(f"a == b: {a == b}")  # False!

# Fix: use math.isclose
import math
print(f"math.isclose(a, b): {math.isclose(a, b)}")  # True

# Mistake 4: Using is for value comparison
a = 1000
b = 1000
print(f"a is b: {a is b}")  # False (implementation dependent)
print(f"a == b: {a == b}")  # True

# Mistake 5: Modifying while iterating
# numbers = [1, 2, 3, 4]
# for n in numbers:
#     if n % 2 == 0:
#         numbers.remove(n)  # Dangerous during iteration

# Mistake 6: Confusing 'and' with '&'
print(True and False)  # False (logical AND)
print(True & False)    # False (bitwise AND)
print(2 and 3)         # 3 (logical AND returns last truthy)
print(2 & 3)           # 2 (bitwise AND)
```

## Best Practices

- Use parentheses for clarity in complex expressions
- Use `is` for `None` comparisons, `==` for value comparisons
- Use `math.isclose()` for float equality checks
- Use `in` for membership tests instead of manual loops
- Use `and`/`or`/`not` for logical operations (not `&`/`|`/`~`)
- Leverage short-circuit evaluation for conditional execution
- Use augmented assignment operators (`+=`, `-=`) for conciseness
- Use chained comparisons (`a < b < c`) when appropriate
- Avoid modifying collections while iterating over them
- Use bitwise operators for low-level operations only
- Consider using `any()` and `all()` instead of chained `or`/`and`

## Interview Questions

1. What is the difference between `==` and `is`?
2. Explain short-circuit evaluation in Python.
3. What is operator precedence? Give an example of a common pitfall.
4. How does the modulo operator work with negative numbers?
5. What are bitwise operators and when would you use them?
6. Explain chained comparisons in Python.
7. What is the ternary operator in Python?
8. How do you overload operators in a custom class?
9. What is the difference between `and`/`or` and `&`/`|`?
10. Explain how the `in` operator works with different data types.

## Coding Challenges

1. Create a calculator that handles all arithmetic operations.
2. Write a function that checks if a number is a power of two using bitwise operations.
3. Create a permission system using bitwise flags.
4. Write a program that demonstrates operator overloading with a custom class.
5. Create a function that finds common elements between two lists using membership operators.

```python
# Challenge 1: Full Calculator
def calculator(a, b, operator):
    if operator == '+':
        return a + b
    elif operator == '-':
        return a - b
    elif operator == '*':
        return a * b
    elif operator == '/':
        if b == 0:
            return "Error: Division by zero"
        return a / b
    elif operator == '//':
        if b == 0:
            return "Error: Division by zero"
        return a // b
    elif operator == '%':
        if b == 0:
            return "Error: Modulo by zero"
        return a % b
    elif operator == '**':
        return a ** b
    else:
        return "Invalid operator"

print(calculator(10, 3, '+'))   # 13
print(calculator(10, 3, '/'))   # 3.333...
print(calculator(10, 3, '//'))  # 3
print(calculator(10, 3, '%'))   # 1
print(calculator(10, 3, '**'))  # 1000

# Challenge 2: Power of Two Check
def is_power_of_two_bitwise(n):
    return n > 0 and (n & (n - 1)) == 0

test_numbers = [1, 2, 4, 8, 16, 32, 18, 6, 64]
for num in test_numbers:
    print(f"{num}: {is_power_of_two_bitwise(num)}")

# Challenge 3: Permission System
class Permissions:
    NONE = 0b000
    READ = 0b001
    WRITE = 0b010
    EXECUTE = 0b100
    
    def __init__(self, perms=0):
        self.perms = perms
    
    def grant(self, permission):
        self.perms |= permission
    
    def revoke(self, permission):
        self.perms &= ~permission
    
    def has(self, permission):
        return bool(self.perms & permission)
    
    def __repr__(self):
        flags = []
        if self.has(self.READ): flags.append("R")
        if self.has(self.WRITE): flags.append("W")
        if self.has(self.EXECUTE): flags.append("X")
        return f"Permissions({''.join(flags) or 'NONE'})"

p = Permissions(Permissions.READ | Permissions.WRITE)
print(p)  # Permissions(RW)
print(f"Can read: {p.has(Permissions.READ)}")
p.grant(Permissions.EXECUTE)
print(p)  # Permissions(RWX)
p.revoke(Permissions.WRITE)
print(p)  # Permissions(RX)

# Challenge 4: Operator Overloading - Complex Number
class ComplexNumber:
    def __init__(self, real, imag):
        self.real = real
        self.imag = imag
    
    def __add__(self, other):
        return ComplexNumber(self.real + other.real, self.imag + other.imag)
    
    def __sub__(self, other):
        return ComplexNumber(self.real - other.real, self.imag - other.imag)
    
    def __mul__(self, other):
        real = self.real * other.real - self.imag * other.imag
        imag = self.real * other.imag + self.imag * other.real
        return ComplexNumber(real, imag)
    
    def __abs__(self):
        return (self.real ** 2 + self.imag ** 2) ** 0.5
    
    def __eq__(self, other):
        return self.real == other.real and self.imag == other.imag
    
    def __repr__(self):
        sign = '+' if self.imag >= 0 else '-'
        return f"{self.real} {sign} {abs(self.imag)}i"

c1 = ComplexNumber(3, 4)
c2 = ComplexNumber(1, -2)
print(f"c1 = {c1}")
print(f"c2 = {c2}")
print(f"c1 + c2 = {c1 + c2}")
print(f"c1 - c2 = {c1 - c2}")
print(f"c1 * c2 = {c1 * c2}")
print(f"|c1| = {abs(c1):.2f}")

# Challenge 5: Common Elements Finder
def common_elements(list1, list2):
    return [item for item in list1 if item in list2]

list_a = [1, 2, 3, 4, 5]
list_b = [3, 4, 5, 6, 7]
print(f"Common: {common_elements(list_a, list_b)}")  # [3, 4, 5]

def common_with_duplicates(list1, list2):
    result = []
    temp = list2.copy()
    for item in list1:
        if item in temp:
            result.append(item)
            temp.remove(item)
    return result

list_c = [1, 2, 2, 3, 4]
list_d = [2, 2, 3, 3, 5]
print(f"Common (with duplicates): {common_with_duplicates(list_c, list_d)}")  # [2, 2, 3]
```

## Summary

Python provides a comprehensive set of operators spanning arithmetic, comparison, logical, assignment, bitwise, identity, and membership operations. Understanding how each operator works, its precedence, and its interaction with different data types is essential for writing correct and efficient Python code. Special attention should be paid to operator precedence, short-circuit evaluation, the difference between `is` and `==`, and the limitations of float arithmetic.

## Related Topics

- Control Flow (Conditions)
- Boolean Logic
- Numbers and Math
- Operator Overloading
- Data Type Conversion
- Expressions and Statements
